# Vaultwarden

## Overview
This directory contains the Docker Compose configuration for **Vaultwarden**, a lightweight, open-source password manager and secrets storage solution. Vaultwarden is a Rust implementation of the Bitwarden API.

## Configuration

### Docker Compose File
The `docker-compose.yml` defines the following service:

**Service: `vaultwarden`**
- **Image**: `vaultwarden/server:latest`
- **Container Name**: `vaultwarden`
- **Restart Policy**: `unless-stopped`

### Environment Variables
- **DOMAIN**: `https://vw.domain.tld` - Set this to your actual Vaultwarden domain

### Volumes
- `./vault/vw-data/:/data/` - Persistent data storage for vault database and attachments

### Ports
- **8005**: Maps to port 80 inside the container (web interface)

## Usage

### Starting the Service
```bash
docker-compose up -d
```

### Accessing Vaultwarden
Navigate to `http://localhost:8005` in your browser to access the Vaultwarden web interface.

### Initial Setup
1. Access the web interface at your configured domain
2. Create your account
3. Log in and start managing your passwords and secrets

## Configuration

### Update Domain
Before deploying, update the `DOMAIN` environment variable in `docker-compose.yml` to match your actual domain:
```yaml
environment:
  DOMAIN: "https://your-actual-domain.com"
```

### Behind Nginx Proxy Manager
If using the Nginx Proxy Manager from this directory, configure a proxy host to forward traffic from your domain to `vaultwarden:80` or `localhost:8005`.

## Backup & Recovery

### Backing Up Your Vault
```bash
# Backup the entire data directory
cp -r ./vault/vw-data ./vault/vw-data.backup
```

### Restoring from Backup
```bash
# Stop the service
docker-compose down

# Restore the backup
rm -rf ./vault/vw-data
cp -r ./vault/vw-data.backup ./vault/vw-data

# Restart the service
docker-compose up -d
```

## Important Notes
- All vault data is persisted in `./vault/vw-data/`
- The service automatically restarts unless manually stopped
- Use a strong admin password
- Configure HTTPS/SSL through the Nginx Proxy Manager for secure access
- Regularly back up your vault data

## Stopping the Service
```bash
docker-compose down
```

## Documentation
For more information, visit: https://github.com/dani-garcia/vaultwarden
