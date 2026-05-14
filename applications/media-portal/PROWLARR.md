# Prowlarr Configuration

## Overview

Prowlarr is a indexer manager/proxy built on the popular arr net base stack to integrate with your various PVR apps. It supports management of both Torrent Trackers and Usenet Indexers.

**Container Name:** prowlarr  
**Image:** lscr.io/linuxserver/prowlarr:latest  
**Port:** 9696  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
prowlarr:
  image: lscr.io/linuxserver/prowlarr:latest
  container_name: prowlarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/prowlarr/config:/config
  ports:
    - "9696:9696"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9696"]
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
| TZ | EST | Timezone |

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /config | /root/docker/prowlarr/config | Configuration and database |

## Ports

| Port | Purpose |
|------|----------|
| 9696 | Web UI access |

## Health Check

- **Test:** HTTP GET to http://localhost:9696
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Features

- Centralized indexer management
- Torrent tracker support
- Usenet indexer support
- Automatic sync with Sonarr/Radarr
- Cloudflare/DDoS-GUARD bypass (with FlareSolverr)
- Test indexer connectivity
- Search aggregation
- Custom indexers support
- API integration

## Initial Setup

1. Access Prowlarr at `http://localhost:9696`
2. Go to Settings → General
3. Configure:
   - API Key (note this)
   - Authentication if needed
4. Add Indexers:
   - Indexers → Add Indexer
   - Search and configure each type
5. Configure Cloudflare Bypass:
   - Settings → Services → FlareSolverr
   - Add FlareSolverr URL: `http://flaresolverr:8191`
6. Sync with Apps:
   - Settings → Apps → Add App
   - Add Sonarr and Radarr instances

## Adding Indexers

### Torrent Trackers

1. Indexers → Add Indexer → Select tracker
2. Configure:
   - URL
   - API Key/Login
   - Priority
   - Categories
3. Test connection
4. Save

### Usenet Indexers

1. Indexers → Add Indexer → Select indexer
2. Configure:
   - API Key
   - Server settings
   - Categories
3. Test connection
4. Save

## App Integration

### Sync with Sonarr

1. Settings → Apps → Add App
2. Select "Sonarr"
3. Configure:
   - URL: `http://sonarr:8989`
   - API Key (from Sonarr)
   - Categories to sync
4. Save

### Sync with Radarr

1. Settings → Apps → Add App
2. Select "Radarr"
3. Configure:
   - URL: `http://radarr:7878`
   - API Key (from Radarr)
   - Categories to sync
4. Save

## Cloudflare Protection Bypass

Enable FlareSolverr for Cloudflare-protected sites:

1. Settings → Services → FlareSolverr
2. Enter URL: `http://flaresolverr:8191`
3. Test connection
4. Indexers using FlareSolverr marked with indicator

## Search Functionality

### Direct Search

1. Top menu: Indexers search bar
2. Enter search term
3. Select indexers to search
4. View results from multiple sources

### App-Initiated Search

- Sonarr/Radarr search via Prowlarr
- Automatic category mapping
- Results returned to originating app

## Configuration Files

Key files in `/root/docker/prowlarr/config`:
- `config.xml` - Main configuration
- `prowlarr.db` - Database
- `logs/` - Application logs
- `indexers/` - Indexer definitions

## Priority and Categories

Set indexer priority for search order:
- **Priority:** Lower number = higher priority
- **Categories:** Map search categories to indexer categories

Example:
```
Movie - Maps to: 2000 (standard), 2010 (3D)
TV - Maps to: 5000 (standard), 5010 (anime)
```

## Testing Indexers

1. Indexers page
2. Click test icon on indexer
3. Verify "Success" response
4. Check response time

## Common Issues

### Indexer Connection Failed
- Verify indexer credentials
- Check site URL
- Review network connectivity
- Check logs: `docker logs -f prowlarr`

### Cloudflare Bypass Not Working
- Ensure FlareSolverr is running
- Verify FlareSolverr URL is correct
- Check FlareSolverr logs
- Try restarting FlareSolverr

### Sync Not Working with Sonarr/Radarr
- Verify app URLs are correct
- Check API keys are valid
- Ensure network connectivity
- Review sync logs

### No Search Results
- Verify at least one indexer is configured
- Test indexer connectivity
- Check search term validity
- Review indexer categories

## API Usage

Prowlarr provides REST API:

```bash
curl -X GET http://localhost:9696/api/v1/indexer \
  -H "X-Api-Key: YOUR_API_KEY"
```

API Documentation: `http://localhost:9696/api`

## Monitoring

View logs:
```bash
docker logs -f prowlarr
```

Access web interface:
```
http://localhost:9696
```

## Restart Service

```bash
docker restart prowlarr
```

## Best Practices

- Test all indexers regularly
- Use appropriate categories
- Set sensible priority order
- Monitor API usage
- Keep indexer list updated
- Test new indexers before relying on them

## Documentation

- Official: https://wiki.servarr.com/prowlarr
- GitHub: https://github.com/Prowlarr/Prowlarr
- API: https://prowlarr.com/docs/api/
