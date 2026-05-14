# Unpackerr Configuration

## Overview

Unpackerr automatically extracts downloaded files in your download folders. It's designed to work with Sonarr, Radarr, and other Arr applications to automatically extract archives after downloads complete.

**Container Name:** unpackerr  
**Image:** golift/unpackerr  
**User:** 1000:1000  
**Health Check Port:** 7878

## Container Configuration

```yaml
unpackerr:
  image: golift/unpackerr
  container_name: unpackerr
  volumes:
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr:/mnt/sonarr
    - /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr:/mnt/radarr
    - /root/docker/unpackerr/config:/config
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:7878"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  restart: always
  user: 1000:1000
  environment:
    # See configuration section below
```

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /mnt/sonarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/sonarr | TV show downloads |
| /mnt/radarr | /root/docker/DUMB/mnt/debrid/decypharr_downloads/radarr | Movie downloads |
| /config | /root/docker/unpackerr/config | Configuration files |

## Health Check

- **Test:** HTTP GET to http://localhost:7878
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Environment Variables

Configure via environment variables or `/config/unpackerr.yml`:

```yaml
un:
  # Sonarr settings
  sonarr:
    - url: http://sonarr:8989
      api_key: YOUR_SONARR_API_KEY
  
  # Radarr settings
  radarr:
    - url: http://radarr:7878
      api_key: YOUR_RADARR_API_KEY
  
  # Extraction settings
  interval: 2s
  start_delay: 1m
  retry_delay: 5m
  max_retries: 3
  parallel: 1
  file_mode: 0644
  dir_mode: 0755
```

## Features

- Automatic archive extraction
- Supports RAR, ZIP, 7z, TAR formats
- Integration with Sonarr and Radarr
- Configurable retry logic
- File permission management
- Real-time extraction monitoring

## Integration

Unpackerr integrates with:
- **Sonarr** - Extracts TV show downloads
- **Radarr** - Extracts movie downloads
- **DUMB** - Monitors debrid download folders

## Configuration

Edit `/root/docker/unpackerr/config/unpackerr.yml`:

```yaml
# Extraction settings
interval: 2s              # Check interval
retry_delay: 5m           # Retry failed extractions
max_retries: 3            # Maximum retry attempts
parallel: 1               # Parallel extractions

# Sonarr configuration
sonarr:
  - url: http://sonarr:8989
    api_key: ${SONARR_API_KEY}
    paths:
      - /mnt/sonarr

# Radarr configuration
radarr:
  - url: http://radarr:7878
    api_key: ${RADARR_API_KEY}
    paths:
      - /mnt/radarr
```

## File Permissions

Set in configuration:
- `file_mode: 0644` - File permissions
- `dir_mode: 0755` - Directory permissions
- `user: 1000:1000` - Container user

## Supported Formats

- RAR (`.rar`)
- ZIP (`.zip`)
- 7-Zip (`.7z`)
- TAR (`.tar`, `.tar.gz`, `.tar.bz2`)
- ISO (`.iso`)

## Common Issues

### Extractions Not Starting
- Check Sonarr/Radarr API keys
- Verify folder permissions
- Review logs: `docker logs -f unpackerr`

### Permission Denied Errors
- Ensure UID/GID (1000:1000) owns directories
- Set correct permissions: `chmod 755 /root/docker/unpackerr`

### Failed Extractions
- Check available disk space
- Verify archive file integrity
- Review error logs

## Monitoring

View logs:
```bash
docker logs -f unpackerr
```

Check extraction status via Sonarr/Radarr:
- View History tab for completion status
- Check Import status

## Restart Service

```bash
docker restart unpackerr
```

## Documentation

- GitHub: https://github.com/unpackerr/unpackerr
- Configuration: https://unpackerr.zip/docs/
