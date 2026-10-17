# Raspberry Pi nginx

The nginx config for `alexisraspberry.duckdns.org` and the LAN. The folders
mirror `/etc/nginx`: each file here is installed at the same path there.

| File | What it does |
|---|---|
| `sites-available/alexisraspberry.duckdns.org` | The server blocks. Public (443): the two games, the portfolio contact form (`/contact`, port 5555), the contentful app (`/contentful`, port 4000) and the chess trainer (`/chess-trainer/`); everything else is 404. LAN (port 80 default server): the games, the chess trainer, monitor-canvas and the agent-runner office dashboard. |
| `snippets/pi-games.conf` | `/zigzag/` and `/kings-and-pigs/`. Each serves only its page and the files it loads; anything else under its path, dotfiles included, is not found. Proxies `/zigzag/score` to go-server (go-server-dev, port 8080), rate-limited. |
| `snippets/pi-chess-trainer.conf` | `/chess-trainer/`, proxied to the PM2 app on port 3100. LAN addresses get in directly; anyone else needs the basic-auth login. |
| `snippets/pi-lan.conf` | `/monitor-canvas/`, LAN only. |
| `snippets/security-headers.conf` | HSTS, nosniff and Referrer-Policy. Included at server level and again in every location with its own `add_header`. |
| `conf.d/limits.conf` | `server_tokens off`, and the rate-limit zones for the public forms (only POSTs count). |
| `conf.d/extra-mime-types.conf` | `.mjs`, `.wav` and `.ttf`, which nginx's `mime.types` lacks. |

Not here: `/etc/nginx/chess-trainer.htpasswd` (the chess trainer's login, see below), `snippets/agent-runner-office.conf`, which comes from the agent-runner
repo (`docs/nginx/office.conf`), and the certbot-managed files under
`/etc/letsencrypt`.

`nginx.conf` itself is the Debian default except `client_max_body_size 1m;`.

## Installing a change

```bash
cd raspberrypi/nginx
sudo cp sites-available/alexisraspberry.duckdns.org /etc/nginx/sites-available/
sudo cp snippets/*.conf /etc/nginx/snippets/
sudo cp conf.d/*.conf /etc/nginx/conf.d/
sudo nginx -t && sudo systemctl reload nginx
```

## Adding a file to a game

A game's files are allowed by folder and type: see the `location ~` lines in
`snippets/pi-games.conf`. A new folder or file type needs adding there, or it
will be not found.

## The chess trainer's login

Outside the LAN, `/chess-trainer/` asks for the basic-auth login in `/etc/nginx/chess-trainer.htpasswd`,
which is not in this repo. Without that file, requests from outside the LAN fail with a 500. To create it, or to change the password:

```bash
printf 'alexis:%s\n' "$(openssl passwd -apr1)" | sudo tee /etc/nginx/chess-trainer.htpasswd >/dev/null
sudo chown root:www-data /etc/nginx/chess-trainer.htpasswd
sudo chmod 640 /etc/nginx/chess-trainer.htpasswd
```

The app itself must run with `BASE_PATH=/chess-trainer` (its
`ecosystem.config.cjs` sets it), or its links and assets point at the root.
