# Nginx Proxy Manager

## Overview
This directory contains the Docker Compose configuration for **Nginx Proxy Manager**, a reverse proxy solution for managing and routing traffic to your applications.

## Configuration

### Docker Compose File
The `docker-compose.yml` defines the following service:

**Service: `app` (nginx)**
- **Image**: `docker.io/jc21/nginx-proxy-manager:latest`
- **Container Name**: `nginx`
- **Restart Policy**: `unless-stopped`

### Ports
- **80**: HTTP traffic
- **81**: Nginx Proxy Manager Admin Panel
- **443**: HTTPS/SSL traffic

### Volumes
- `./data:/data` - Persistent configuration and certificate data
- `./letsencrypt:/etc/letsencrypt` - Let's Encrypt certificate storage

## Usage

### Starting the Service
```bash
docker-compose up -d
```

### Accessing the Admin Panel
Navigate to `http://localhost:81` in your browser to access the Nginx Proxy Manager admin interface.

### Default Credentials
Upon first access, you'll be prompted to create an admin account.

## Important Notes
- The service automatically restarts unless manually stopped
- All configuration data is persisted in the `./data` directory
- SSL certificates are managed and stored in `./letsencrypt`
- Ensure ports 80, 81, and 443 are available on the host system

## Stopping the Service
```bash
docker-compose down
```

## Documentation
For more information, visit: https://github.com/jc21/nginx-proxy-manager
