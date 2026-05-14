# Radarr Configuration

## Overview

Radarr is a movie collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new movies and will interface with clients and indexers to grab, sort, and rename them.

**Container Name:** radarr  
**Image:** lscr.io/linuxserver/radarr:latest  
**Port:** 7878  
**User:** 1000:1000  
**Timezone:** EST

## Container Configuration

```yaml
radarr:
  image: lscr.io/linuxserver/radarr:latest
  container_name: radarr
  environment:
    - PUID=1000
    - PGID=1000
    - TZ=EST
  volumes:
    - /root/docker/radarr/config:/config
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr:/mnt/radarr:rslave
  ports:
    - "7878:7878"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:7878"]
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
| /config | /root/docker/radarr/config | Configuration and database | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | DUMB debrid mount | - |
| /mnt/radarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr | Movie downloads | rslave |

## Ports

| Port | Purpose |
|------|----------|
| 7878 | Web UI access |

## Health Check

- **Test:** HTTP GET to http://localhost:7878
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Features

- Movie database management
- Automatic download monitoring
- Quality and language management
- Custom formats support
- API integration
- Notification system
- Import/Rename automation
- Multiple root folders

## Initial Setup

1. Access Radarr at `http://localhost:7878`
2. Go to Settings → General
3. Set up:
   - Root folder: `/mnt/radarr`
   - Library location
4. Configure Indexers:
   - Settings → Indexers → Add Indexer
   - Use Prowlarr (recommended)
5. Set Download Client:
   - Settings → Download Clients
   - Configure DUMB or your download client
6. Configure Media Management:
   - Naming conventions
   - File/Folder organization
   - Import settings

## Integration Points

### With Prowlarr (Recommended)

1. In Prowlarr: Indexers → Manage → Add Radarr
2. Add your Radarr instance URL and API key
3. Prowlarr automatically syncs indexers

### With Download Clients

**DUMB Integration:**
- Radarr monitors `/mnt/radarr` downloads
- DUMB provides streaming sources
- Unpackerr extracts any compressed files

**Manual Client Setup:**
- Settings → Download Clients → Add
- Type: Your download client
- Configure connection details

### With Jellyseerr

Jellyseerr sends movie requests to Radarr:
1. In Jellyseerr: Settings → Services → Radarr
2. Add Radarr instance
3. Configure quality profile
4. Set root folder

## Quality Profiles

Create quality profiles for different media types:
- HD (1080p)
- 4K (2160p)
- Web-DL
- Remux

Custom Formats allow fine-grained control over releases.

## Root Folders

Define where movies are stored:
```
/mnt/radarr/
├── Action/
├── Comedy/
├── Drama/
└── Horror/
```

## Configuration Files

Key files in `/root/docker/radarr/config`:
- `config.xml` - Main configuration
- `radarr.db` - Database
- `logs/` - Application logs

## Common Tasks

### Adding a Movie
1. Click "+ Add Movies"
2. Search for movie
3. Select quality profile
4. Choose root folder
5. Click "Add"

### Monitoring Downloads
1. Go to Activity → Queue
2. View current downloads
3. Check status and ETA

### Viewing Library
1. Click "Movies"
2. Filter and sort as needed
3. Click movie for details

## Troubleshooting

### Movies Not Downloading
- Verify indexers are configured
- Check Prowlarr integration
- Ensure download client is available
- Review logs for errors

### Import Failures
- Check file naming conventions
- Verify folder permissions
- Review import settings
- Check logs: `docker logs -f radarr`

### API Connection Issues
- Ensure Radarr is running
- Verify API key is correct
- Check network connectivity
- Review firewall settings

## API Usage

Radarr provides REST API:

```bash
curl -X GET http://localhost:7878/api/v3/movie \
  -H "X-Api-Key: YOUR_API_KEY"
```

API Documentation available at: `http://localhost:7878/api`

## Monitoring

View logs:
```bash
docker logs -f radarr
```

Access web interface:
```
http://localhost:7878
```

## Restart Service

```bash
docker restart radarr
```

## Performance Tuning

- Increase indexer timeout if searches fail
- Adjust interval between jobs
- Limit concurrent downloads
- Monitor disk I/O

## Documentation

- Official: https://wiki.servarr.com/en/radarr
- GitHub: https://github.com/Radarr/Radarr
- API: https://radarr.video/docs/api/
