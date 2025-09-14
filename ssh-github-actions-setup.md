# Setting up SSH Keys for GitHub Actions Deployment

This guide explains how to create SSH keys and configure them for automated deployment to your VPS using GitHub Actions.

## Overview

GitHub Actions can automatically deploy your application to a VPS by connecting via SSH. This requires:
1. An SSH key pair (private + public key)
2. The public key added to your VPS
3. The private key stored as a GitHub secret

## Step 1: Generate SSH Key Pair

On your VPS, run the following commands to generate a new SSH key pair:

```bash
# Generate a new SSH key pair
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/github_actions_deploy -N ""
```

This creates:
- `~/.ssh/github_actions_deploy` (private key)
- `~/.ssh/github_actions_deploy.pub` (public key)

## Step 2: Display the Private Key

Copy the private key content to add to GitHub secrets:

```bash
cat ~/.ssh/github_actions_deploy
```

The output should look like:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACD... (long string of characters)
-----END OPENSSH PRIVATE KEY-----
```

**Copy this entire output** - you'll need it for the GitHub secret.

## Step 3: Display the Public Key

View the public key to add to your VPS:

```bash
cat ~/.ssh/github_actions_deploy.pub
```

The output should look like:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG... github-actions-deploy
```

## Step 4: Add Public Key to VPS

Add the public key to your VPS's authorized keys:

```bash
cat ~/.ssh/github_actions_deploy.pub >> ~/.ssh/authorized_keys
```

## Step 5: Set Proper Permissions

Ensure the SSH files have correct permissions:

```bash
chmod 600 ~/.ssh/github_actions_deploy
chmod 644 ~/.ssh/github_actions_deploy.pub
chmod 600 ~/.ssh/authorized_keys
```

## Step 6: Configure GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add these three secrets:

### VPS_HOST
- **Name**: `VPS_HOST`
- **Value**: Your VPS IP address (e.g., `95.123.456.789`) or domain name
- **Note**: Use only the IP/domain, no `http://` prefix

### VPS_USER
- **Name**: `VPS_USER`
- **Value**: Your VPS username (usually `root` or your user account)

### VPS_SSH_KEY
- **Name**: `VPS_SSH_KEY`
- **Value**: The entire private key content from Step 2 (including `-----BEGIN` and `-----END` lines)

## Step 7: Test the Connection

You can test the SSH connection from your local machine:

```bash
ssh -i ~/.ssh/github_actions_deploy your-username@your-vps-ip
```

## Example GitHub Actions Workflow

Here's an example workflow that uses these secrets:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy on VPS via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /path/to/your/project
            docker compose pull
            docker compose up -d
```

## Security Best Practices

1. **Use a dedicated key**: Create a separate SSH key specifically for GitHub Actions
2. **Restrict permissions**: The SSH key should only have access to what's needed for deployment
3. **Regular rotation**: Consider rotating SSH keys periodically
4. **Monitor access**: Check your VPS logs to monitor SSH access

## Troubleshooting

### Permission Denied
- Verify the public key is in `~/.ssh/authorized_keys`
- Check file permissions (should be 600 for private key, 644 for public key)
- Ensure the SSH service is running: `sudo systemctl status ssh`

### Connection Refused
- Verify the VPS IP address and port (default is 22)
- Check if SSH is enabled on your VPS
- Ensure firewall allows SSH connections

### Key Format Issues
- Make sure the private key includes the `-----BEGIN` and `-----END` lines
- Verify there are no extra spaces or characters when copying

## Alternative: Using Existing SSH Key

If you already have an SSH key on your VPS:

```bash
# Display existing private key
cat ~/.ssh/id_rsa
# OR
cat ~/.ssh/id_ed25519

# Display existing public key
cat ~/.ssh/id_rsa.pub
# OR
cat ~/.ssh/id_ed25519.pub
```

Use the private key content as the `VPS_SSH_KEY` secret, and ensure the public key is in `~/.ssh/authorized_keys`.
