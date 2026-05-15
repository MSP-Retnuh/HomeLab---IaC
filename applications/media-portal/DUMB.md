# DUMB (Debrid Unlimited Media Bridge) Configuration

## Overview

DUMB (Debrid Unlimited Media Bridge) is a powerful media streaming solution that integrates with Debrid services to provide seamless access to vast media libraries through Jellyfin and other media servers.

**Container Name:** DUMB  
**Image:** iampuid0/dumb:latest  
**Ports:** 3005, 8096, 8282, 8182  
**User:** 1000:1000  
**Timezone:** EST  
**GitHub:** https://github.com/I-am-PUID-0/DUMB

## Container Configuration

```yaml
DUMB:
  container_name: DUMB
  image: iampuid0/dumb:latest
  stop_grace_period: 30s
  shm_size: 128mb
  stdin_open: true
  tty: true
  volumes:
    - /root/docker/DUMB/config:/config
    - /root/docker/DUMB/log:/log
    - /root/docker/DUMB/data:/data
    - /root/docker/DUMB/mnt/debrid:/mnt/debrid:rshared
  environment:
    - TZ=EST
    - PUID=1000
    - PGID=1000
  ports:
    - "3005:3005"
    - "8096:8096"
    - "8282:8282"
    - "8182:8182"
  devices:
    - /dev/fuse:/dev/fuse:rwm
    - /dev/dri:/dev/dri
  cap_add:
    - SYS_ADMIN
  security_opt:
    - apparmor:unconfined
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8096"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 1200s
  restart: unless-stopped
```

## Environment Variables

| Variable | Value | Description |
|----------|-------|-------------|
| TZ | EST | Timezone |
| PUID | 1000 | User ID for container process |
| PGID | 1000 | Group ID for container process |

## Volumes

| Container Path | Host Path | Purpose | Flags |
|-----------------|-----------|---------|-------|
| /config | /root/docker/DUMB/config | Configuration files | - |
| /log | /root/docker/DUMB/log | Application logs | - |
| /data | /root/docker/DUMB/data | Application data | - |
| /mnt/debrid | /root/docker/DUMB/mnt/debrid | Debrid mount (FUSE) | rshared |

## Ports

| Port | Purpose | Service |
|------|---------|----------|
| 3005 | API/Control | DUMB API |
| 8096 | Media Server | Jellyfin/Emby interface |
| 8282 | Cache/Metadata | Internal service |
| 8182 | Configuration | Admin interface |

## Special Requirements

### Devices

```yaml
devices:
  - /dev/fuse:/dev/fuse:rwm    # FUSE for mount operations
  - /dev/dri:/dev/dri          # GPU acceleration (optional)
```

### Capabilities

```yaml
cap_add:
  - SYS_ADMIN    # Required for FUSE mount

security_opt:
  - apparmor:unconfined    # Disable AppArmor restrictions
```

### Memory/Resources

```yaml
shm_size: 128mb    # Shared memory for FUSE operations
stop_grace_period: 30s    # Allow clean shutdown
```

## Health Check

- **Test:** HTTP GET to http://localhost:8096
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 1200 seconds (20 minutes - longer initial startup)

## Features

- Debrid Unlimited API integration
- Virtual filesystem via FUSE
- Media library caching
- Streaming optimization
- Multi-user support
- Integrated metadata
- Download management
- Cache intelligence

## Initial Setup

1. **Start container:** `docker-compose up -d DUMB`
2. **Wait for startup:** Full initialization takes ~20 minutes
3. **Access admin:** http://localhost:3005
4. **Configure Debrid:**
   - Enter Debrid API keys
   - Authorize account
   - Configure cache settings
5. **Access media server:** http://localhost:8096

## Configuration

### Debrid Integration

In admin panel (http://localhost:8182):
1. Settings → Debrid
2. Enter API keys for:
   - Real-Debrid
   - Alldebrid (optional)
3. Configure download preferences
4. Set cache behavior

### Mount Configuration

The FUSE mount at `/mnt/debrid` provides:
- Virtual file access to Debrid content
- Transparent caching
- Automatic cleanup

### Download Folders

DUMB creates download folders for Arr integration:
```
/mnt/debrid/decypharr_downloads/
├── sonarr/     # TV show downloads
└── radarr/     # Movie downloads
```

## Integration with Arr Stack

### Sonarr Integration

1. In Sonarr → Settings → Download Clients
2. Add custom script pointing to DUMB
3. Configure:
   - Category: Series
   - Download path: `/mnt/debrid/decypharr_downloads/sonarr`

### Radarr Integration

1. In Radarr → Settings → Download Clients
2. Add custom script pointing to DUMB
3. Configure:
   - Category: Movie
   - Download path: `/mnt/debrid/decypharr_downloads/radarr`

## Unpackerr Integration

Unpackerr monitors download folders:
```yaml
sonarr:
  - /mnt/debrid/decypharr_downloads/sonarr
radarr:
  - /mnt/debrid/decypharr_downloads/radarr
```

When downloads complete:
1. Files extracted automatically
2. Sonarr/Radarr notified
3. Content imported into library

## Jellyfin/Media Server Access

1. Add library location:
   - Path: `/mnt/debrid/decypharr_downloads/`
2. Configure scanner and matcher
3. Set metadata sources
4. Enable auto-refresh

## Performance Tuning

### Cache Settings

- Enable intelligent caching
- Set cache size appropriately
- Configure retention policies
- Monitor cache hits/misses

### Streaming Optimization

- Buffer size configuration
- Timeout settings
- Concurrent connection limits
- Quality preferences

## Storage Structure

```
/root/docker/DUMB/
├── config/         # Configuration files
├── log/            # Application logs
├── data/           # Application data
└── mnt/debrid/     # FUSE mount point
    ├── config/
    ├── cache/
    └── decypharr_downloads/
        ├── sonarr/
        └── radarr/
```

## Troubleshooting

### Mount Not Working
- Verify `/dev/fuse` access: `ls -l /dev/fuse`
- Check apparmor restrictions
- Review container logs: `docker logs -f DUMB`
- Ensure SYS_ADMIN capability present

### Connection to Debrid Failed
- Verify API keys are correct
- Check network connectivity
- Review Debrid account status
- Check logs for error messages

### High Memory Usage
- Adjust cache settings
- Reduce buffer size
- Check for memory leaks in logs
- Monitor with: `docker stats DUMB`

### Slow Performance
- Check network connectivity
- Monitor cache hit rate
- Review Debrid account limits
- Optimize buffer settings
- Consider GPU acceleration

## Monitoring

View logs:
```bash
docker logs -f DUMB
```

Check statistics:
```bash
docker stats DUMB
```

Access admin panel:
```
http://localhost:8182
```

## File Permissions

Ensure proper ownership:
```bash
chown -R 1000:1000 /root/docker/DUMB
chmod -R 755 /root/docker/DUMB
```

## Restart Service

```bash
docker restart DUMB
```

## Important Notes

⚠️ **FUSE Mount:** Requires elevated privileges  
⚠️ **Startup Time:** Initial startup takes ~20 minutes  
⚠️ **Debrid Account:** Required for functionality  
⚠️ **Storage:** Cache can grow significantly - monitor disk space  

## Documentation

- GitHub: https://github.com/I-am-PUID-0/DUMB
- Official Site: https://dumbarr.com
- Issues/Support: https://github.com/I-am-PUID-0/DUMB/issues
