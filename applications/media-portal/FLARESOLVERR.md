# FlareSolverr Configuration

## Overview

FlareSolverr is a proxy server to bypass Cloudflare protection on web requests. It's essential for Prowlarr and Indexers to access sites protected by Cloudflare.

**Container Name:** flaresolverr  
**Image:** ghcr.io/flaresolverr/flaresolverr:latest  
**Port:** 8191  
**Region:** Europe/London

## Container Configuration

```yaml
flaresolverr:
  image: ghcr.io/flaresolverr/flaresolverr:latest
  container_name: flaresolverr
  environment:
    - LOG_LEVEL=${LOG_LEVEL:-info}
    - LOG_FILE=${LOG_FILE:-none}
    - LOG_HTML=${LOG_HTML:-false}
    - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
    - TZ=Europe/London
  ports:
    - "8191:8191"
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:8191"]
    interval: 30s
    timeout: 10s
    retries: 3
    start_period: 60s
  volumes:
    - /var/lib/flaresolver:/config
  restart: unless-stopped
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| LOG_LEVEL | info | Log verbosity (trace, debug, info, warn, error) |
| LOG_FILE | none | Log output file path (none = stdout) |
| LOG_HTML | false | HTML logging format |
| CAPTCHA_SOLVER | none | CAPTCHA solving service (none, harvester, imagetyperz) |
| TZ | Europe/London | Timezone |

## Ports

| Port | Purpose |
|------|----------|
| 8191 | FlareSolverr API endpoint |

## Health Check

- **Test:** HTTP GET to http://localhost:8191
- **Interval:** 30 seconds
- **Timeout:** 10 seconds
- **Retries:** 3
- **Start Period:** 60 seconds

## Volumes

| Container Path | Host Path | Purpose |
|-----------------|-----------|----------|
| /config | /var/lib/flaresolver | Configuration and cache |

## Features

- Cloudflare bypass
- CAPTCHA solving (optional)
- Request proxying
- Cookie management
- JavaScript rendering
- Headless browser operation

## Integration

FlareSolverr integrates with:
- **Prowlarr** - Indexer access
- **Sonarr** - Search functionality
- **Radarr** - Search functionality
- **Custom Indexers** - Any Cloudflare-protected site

## Configuration

### Basic Setup

1. FlareSolverr runs at `http://localhost:8191`
2. Configure in Prowlarr:
   - Settings → Download Clients → FlareSolverr
   - Set URL: `http://flaresolverr:8191`

### CAPTCHA Solving (Optional)

For CAPTCHA support, set `CAPTCHA_SOLVER`:

```bash
CAPTCHA_SOLVER=harvester
# or
CAPTCHA_SOLVER=imagetyperz
```

Provide API keys in configuration if using paid services.

## Troubleshooting

### Connection Refused
- Ensure FlareSolverr is running: `docker ps | grep flaresolverr`
- Check port mapping: `docker port flaresolverr`
- Verify network connectivity

### Cloudflare Bypass Not Working
- Check FlareSolverr logs: `docker logs flaresolverr`
- Verify TZ environment variable
- May need to wait for browser initialization
- Try restarting: `docker restart flaresolverr`

### High Memory Usage
- Normal behavior (uses headless browser)
- Chromium requires significant resources
- Monitor: `docker stats flaresolverr`

## API Testing

Test FlareSolverr API:

```bash
curl -X POST http://localhost:8191/v1 \
  -H "Content-Type: application/json" \
  -d '{
    "cmd": "request.get",
    "url": "https://www.example.com",
    "maxTimeout": 60000
  }'
```

## Monitoring

View logs:
```bash
docker logs -f flaresolverr
```

Check statistics:
```bash
docker stats flaresolverr
```

## Performance Notes

- First request may take 10-30 seconds (browser startup)
- Subsequent requests are faster (connection reuse)
- Memory usage: 300-500MB typical
- CPU usage: minimal when idle

## Restart Service

```bash
docker restart flaresolverr
```

## Important Notes

⚠️ **Resource Intensive:** Runs a full Chromium browser  
⚠️ **Rate Limiting:** Respect target site rate limits  
⚠️ **Legal:** Verify compliance with site ToS  

## Documentation

- GitHub: https://github.com/flaresolverr/flaresolverr
- API Docs: https://flaresolverr.com/
