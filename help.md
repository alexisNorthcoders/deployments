### Docker Compose Help

This project ships with a `docker-compose.yml` that runs three services:

- **postgres**: PostgreSQL 15 on port 5432, data persisted in volume `postgres_data`
- **taskmanager**: Spring Boot backend on port 8080, uploads in volume `uploads_data`
- **taskmanager-client**: Web client on port 3000

Run all commands from the project root: `/home/alexis/Projects/task-manager`.

### Prerequisites

- Docker Engine 24+ and Docker Compose (v2). Use `docker compose ...` (or `docker-compose ...`).
- Optional `.env` file in the project root to override defaults used by `docker-compose.yml`.

Example `.env` (adjust for production):

```bash
POSTGRES_DB=taskmanager
POSTGRES_USER=taskmanager
POSTGRES_PASSWORD=taskmanager123

SPRING_JPA_DDL_AUTO=update
JWT_SECRET=change-me-in-production
JWT_EXPIRATION=86400000
SERVER_PORT=8080
LOGGING_LEVEL=INFO
MAX_FILE_SIZE=10MB
MAX_REQUEST_SIZE=10MB
```

### Start the stack

```bash
docker compose up -d
# or with image pulling
docker compose up -d --pull always
```

Wait for healthchecks to pass:

```bash
docker compose ps
```

Open:

- Client: `http://localhost:3000`
- API: `http://localhost:8080`
- Health: `http://localhost:8080/actuator/health`

### View logs

```bash
# all services (follow, last 200 lines)
docker compose logs -f --tail=200

# single service
docker compose logs -f taskmanager
docker compose logs -f postgres
docker compose logs -f taskmanager-client
```

### Check status and inspect

```bash
docker compose ps
docker compose top               # show running processes per service
docker compose config            # render the effective compose config
```

### Stop, start, restart

```bash
# stop services but keep containers/volumes
docker compose stop

# start previously stopped containers
docker compose start

# restart all or one service
docker compose restart
docker compose restart taskmanager

# remove containers and network (data persists)
docker compose down

# remove containers, network, and named volumes (DATA LOSS!)
docker compose down -v
```

### Update containers to the latest images

```bash
# pull new images for all services
docker compose pull

# recreate containers with the new images
docker compose up -d

# update a single service
docker compose pull taskmanager-client && docker compose up -d taskmanager-client
```

### Execute commands inside containers

```bash
# open a shell
docker compose exec taskmanager sh
docker compose exec taskmanager-client sh

# psql into the database
docker compose exec postgres psql -U ${POSTGRES_USER:-taskmanager} -d ${POSTGRES_DB:-taskmanager}
```

### Data and volumes

- Database data: volume `postgres_data`
- File uploads: volume `uploads_data`

These survive `docker compose down`. To reset everything, run `docker compose down -v` (irreversible).

Backup the database (custom format):

```bash
docker compose exec postgres pg_dump -U ${POSTGRES_USER:-taskmanager} -d ${POSTGRES_DB:-taskmanager} -F c -f /tmp/backup.dump
container_id=$(docker compose ps -q postgres)
docker cp "$container_id:/tmp/backup.dump" ./backup.dump
```

### Rebuild backend locally (optional)

By default, the compose file uses the published image `alexiscreoulo/taskmanager:latest`. To run a locally built backend image from this repository:

```bash
# from project root where Dockerfile is located
docker build -t alexiscreoulo/taskmanager:latest .
docker compose up -d taskmanager
```

### Common issues

- **Port already in use**: Change the host port mapping in `docker-compose.yml` or stop the conflicting process.
- **Images not updating**: Run `docker compose pull` before `docker compose up -d`.
- **Reset to a clean state**: `docker compose down -v` (removes volumes; data loss).