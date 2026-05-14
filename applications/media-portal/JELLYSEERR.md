# Jellyseerr Configuration

## Overview

Jellyseerr is a free and open source software application for managing requests for your media library. It is a web application built on top of Jellyfin (and Emby) to enable your users to request new content and manage requests.

**Container Name:** jellyseerr  
**Image:** ghcr.io/fallenbagel/jellyseerr:latest  
**Port:** 5055  
**User:** 1000:1000

## Container Configuration

```yaml
jellyseerr:
  image: ghcr.io/fallenbagel/jellyseerr:latest
  init: true
  container_name: jellyseerr
  environment:
    - PUID=1000
    - PGID=1000
    - user=1000:1000
    - LOG_LEVEL=debug
    - TZ=New York
    - PORT=5055
  ports:
    - 5055:5055
  volumes:
    - /root/docker/overseerr/config:/app/config
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:5055"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  restart: unless-stopped
```

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| PUID | 1000 | User ID for container process |
| PGID | 1000 | Group ID for container process |
| LOG_LEVEL | debug | Logging level (debug, info, warn, error) |
| TZ | New York | Timezone |
| PORT | 5055 | Service port |

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /app/config | /root/docker/overseerr/config | Configuration and database storage |

## Ports

| Port | Purpose |
|------|----------|
| 5055 | Web UI access |

## Health Check

- **Test:** HTTP GET to http://localhost:5055
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Features

- User request management
- Content discovery and recommendations
- Integration with Jellyfin/Emby
- Request approval workflow
- Movie and TV show requests
- Real-time notifications

## Integration

Jellyseerr integrates with:
- **Jellyfin** - Media server (connects automatically after setup)
- **Sonarr** - TV show management
- **Radarr** - Movie management
- **Arr Stack** - Full automation pipeline

## First Time Setup

1. Access Jellyseerr at `http://localhost:5055`
2. Configure Jellyfin server connection
3. Link Sonarr and Radarr instances
4. Set up notification preferences
5. Configure approval settings

## Configuration File

Configuration is stored in `/root/docker/overseerr/config/`:
- `settings.json` - Main settings
- `logs/` - Application logs
- `db/` - Database files

## Common Issues

### Connection to Jellyfin Failed
- Ensure Jellyfin container is running
- Verify network connectivity between containers
- Check Jellyfin API key configuration

### Requests Not Reaching Arr Stack
- Verify Sonarr/Radarr API keys are correct
- Check network connectivity
- Ensure quality profiles are configured in Arr services

## Monitoring

View logs:
```bash
docker logs -f jellyseerr
```

Access web interface:
```
http://localhost:5055
```

## Restart Service

```bash
docker restart jellyseerr
```

## Documentation

- Official: https://docs.jellyseerr.dev/
- GitHub: https://github.com/fallenbagel/jellyseerr
