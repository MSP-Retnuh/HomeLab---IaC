# Sonarr Configuration

## Overview

Sonarr is a PVR (Personal Video Recorder) for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will interface with clients and indexers to grab, sort, and rename them.

**Container Name:** sonarr  
**Image:** lscr.io/linuxserver/sonarr:latest  
**Port:** 8989  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
sonarr:
  image: lscr.io/linuxserver/sonarr:latest
  container_name: sonarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/sonarr/config:/config
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr:/mnt/sonarr:rslave
  ports:
    - "8989:8989"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8989"]
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

| Container Path | Host Path | Purpose | Flags |
|-----------------|-----------|---------|-------|
| /config | /root/docker/sonarr/config | Configuration and database | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | DUMB debrid mount | - |
| /mnt/sonarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr | Episode downloads | rslave |

## Ports

| Port | Purpose |
|------|----------|
| 8989 | Web UI access |

## Health Check

- **Test:** HTTP GET to http://localhost:8989
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Features

- TV series management
- Episode tracking
- Automatic download monitoring
- Quality and language profiles
- Custom formats
- Episode renaming/organization
- Calendar view
- RSS monitoring
- API integration
- Notification system

## Initial Setup

1. Access Sonarr at `http://localhost:8989`
2. Go to Settings → General
3. Configure:
   - Root folder: `/mnt/sonarr`
   - Library location
4. Add Indexers:
   - Settings → Indexers
   - Use Prowlarr (recommended)
5. Set Download Client:
   - Settings → Download Clients
   - Configure DUMB or your download client
6. Setup Media Management:
   - Naming conventions
   - Folder organization
   - Import settings

## Integration Points

### With Prowlarr (Recommended)

1. In Prowlarr: Indexers → Manage → Add Sonarr
2. Add Sonarr instance URL and API key
3. Prowlarr syncs indexers automatically

### With Download Clients

**DUMB Integration:**
- Sonarr monitors `/mnt/sonarr` downloads
- DUMB provides streaming sources
- Unpackerr extracts compressed files

**Manual Setup:**
- Settings → Download Clients → Add
- Configure your download client

### With Jellyseerr

Jellyseerr sends TV requests to Sonarr:
1. In Jellyseerr: Settings → Services → Sonarr
2. Add Sonarr instance
3. Configure quality profile
4. Set root folder

## Series Management

### Adding a Series

1. Click "+ Add Series"
2. Search for TV show
3. Select quality profile
4. Choose root folder
5. Click "Add"

### Season Types

- **Standard:** Regular seasons (Season 1, 2, etc.)
- **Anime:** Special anime numbering
- **Daily:** Daily shows with air date as episode number
- **Absolute:** Absolute episode numbering

## Quality Profiles

Configure quality tiers:
- 720p WebDL
- 1080p WebDL
- 1080p Remux
- 4K UHD

Custom Formats allow detailed control:
- Codec preferences (H.264, H.265)
- Source preferences (WEB-DL, BluRay)
- Release group preferences

## Root Folders

Organize series storage:
```
/mnt/sonarr/
├── Action/
├── Comedy/
├── Drama/
├── Sci-Fi/
└── Sports/
```

## Release Profiles

Control release selection priority:
- Set preferred release groups
- Define must-have keywords
- Exclude unwanted terms
- Configure release delays

## Configuration Files

Key files in `/root/docker/sonarr/config`:
- `config.xml` - Main configuration
- `sonarr.db` - Database
- `logs/` - Application logs

## Common Tasks

### Monitoring Downloads

1. Go to Activity → Queue
2. View current downloads
3. Check status and progress

### Viewing Calendar

1. Click "Calendar"
2. See upcoming releases
3. Plan viewing schedule

### Library View

1. Click "Series"
2. Filter by status
3. Click series for details

## Episode Monitoring

Control which episodes to download:
- **Monitored:** Will search for downloads
- **Unmonitored:** Ignored in searches
- **All Unmonitored:** Don't monitor any episodes
- **Latest Only:** Only monitor latest season

## Troubleshooting

### Episodes Not Downloading
- Verify indexers are configured
- Check Prowlarr integration
- Ensure download client available
- Review logs for errors

### Import Failures
- Check file naming conventions
- Verify folder permissions
- Review import settings
- Check logs: `docker logs -f sonarr`

### API Connection Issues
- Ensure Sonarr is running
- Verify API key is correct
- Check network connectivity
- Review firewall rules

## API Usage

Sonarr provides REST API:

```bash
curl -X GET http://localhost:8989/api/v3/series \
  -H "X-Api-Key: YOUR_API_KEY"
```

API Documentation: `http://localhost:8989/api`

## Monitoring

View logs:
```bash
docker logs -f sonarr
```

Access web interface:
```
http://localhost:8989
```

## Restart Service

```bash
docker restart sonarr
```

## Performance Optimization

- Adjust RSS sync interval
- Increase search timeout for slow indexers
- Limit concurrent downloads
- Monitor disk I/O and CPU

## Documentation

- Official: https://wiki.servarr.com/en/sonarr
- GitHub: https://github.com/Sonarr/Sonarr
- API: https://sonarr.tv/docs/api/
