# Uptime Kuma - Monitoring & Alerting

**Uptime Kuma** is a powerful, self-hosted monitoring solution that tracks the uptime and health of your services. It provides a beautiful web dashboard, detailed status pages, and multi-channel alerting to keep you informed about your infrastructure's health.

## 📋 Quick Overview

| Property | Value |
|----------|-------|
| **Image** | `louislam/uptime-kuma:2` |
| **Web Port** | 3001 (mapped to host) |
| **Restart Policy** | Always |
| **Storage** | Persistent (`~/home/root/docker-kuma:/app/data`) |
| **Timezone** | UTC (configurable) |
| **Network** | Custom `kuma_network` |

---

## 🚀 Configuration Breakdown

### Service Definition
```yaml
uptime-kuma:
  image: louislam/uptime-kuma:2              # Version 2 (latest stable)
  container_name: uptime-kuma                # Fixed container name
  restart: always                            # Always restart if crashes
  ports:
    - "3001:3001"                            # Host:Container port mapping
```

### Port Mapping
```yaml
ports:
  - "3001:3001"
```
- **Host Port**: 3001 (your machine's IP)
- **Container Port**: 3001 (Uptime Kuma inside container)
- **Access**: http://localhost:3001 or http://[your-ip]:3001

### Persistent Storage
```yaml
volumes:
  - ~/home/root/docker-kuma:/app/data       # Persistent data directory
```

**What's stored**:
- ✅ Monitor configurations (all your monitoring setup)
- ✅ Historical data (uptime records, incident logs)
- ✅ User accounts (credentials and permissions)
- ✅ Alert configurations (notification settings)
- ✅ Status page settings

**Why it matters**:
- If container restarts, all data is preserved
- Multiple restarts don't lose configuration
- You can backup the `~/home/root/docker-kuma` directory

### Environment Variables
```yaml
environment:
  - TZ=UTC                                   # Timezone for timestamps
  - UMASK=0022                               # File permission mask (744)
```

**Timezone Options**:
```bash
# Common timezones:
TZ=America/New_York          # Eastern Time
TZ=America/Chicago           # Central Time
TZ=America/Denver            # Mountain Time
TZ=America/Los_Angeles       # Pacific Time
TZ=Europe/London             # GMT
TZ=Europe/Paris              # CET
TZ=Asia/Tokyo                # JST
TZ=Australia/Sydney          # AEDT
```

**UMASK Explanation**:
- `0022` = 755 permissions (rwxr-xr-x)
- More secure: `0077` = 700 (rwx------)
- More open: `0002` = 775 (rwxrwxr-x)

### Network Configuration
```yaml
networks:
  - kuma_network                             # Custom isolated network
```

**Benefits**:
- ✅ Isolated from other Docker networks
- ✅ Can communicate with AutoHeal
- ✅ Services can reference each other by hostname
- ✅ Only containers on this network can reach Uptime Kuma

### Health Check
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3001"]
  interval: 30s                              # Check every 30 seconds
  retries: 3                                 # Fail after 3 consecutive failures
  start_period: 10s                          # Wait 10s before first check
  timeout: 5s                                # Timeout if no response in 5s
```

**Health Check Details**:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `test` | `curl -f http://localhost:3001` | Try to connect to the dashboard |
| `interval` | 30s | Check every 30 seconds |
| `retries` | 3 | Mark unhealthy after 3 failures |
| `start_period` | 10s | Grace period for startup |
| `timeout` | 5s | Fail if no response within 5 seconds |

**Timeline Example**:
```
00:00 - Container starts
00:10 - First health check (start_period done)
00:40 - Second health check (interval: 30s)
01:10 - Third health check
01:40 - Fourth health check
       ... continues every 30 seconds
```

### Logging Configuration
```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"                         # Rotate logs at 10MB
```

**Benefits**:
- ✅ Limits disk space usage
- ✅ Prevents log files from consuming all storage
- ✅ Automatic rotation
- ✅ JSON format for structured logging

---

## 📊 How Uptime Kuma Works

### Core Functionality

```
┌─────────────────────────────────────────────────────┐
│         Uptime Kuma Monitoring System                │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. You Create Monitors                             │
│     └─ Add website URLs or services to monitor     │
│     └─ Configure check intervals (30s, 1m, 5m)    │
│     └─ Set timeout and retry policies              │
│                                                      │
│  2. Kuma Checks Service Health                      │
│     └─ Sends HTTP request to service               │
│     └─ Measures response time                       │
│     └─ Records UP or DOWN status                    │
│                                                      │
│  3. Kuma Tracks Uptime                              │
│     └─ Calculates availability percentage           │
│     └─ Records incident times                       │
│     └─ Maintains historical data                    │
│                                                      │
│  4. Kuma Sends Alerts                               │
│     └─ Triggers when service goes DOWN              │
│     └─ Sends to Email, Slack, Discord, etc.        │
│     └─ Sends recovery notification when UP          │
│                                                      │
│  5. Displays Dashboard                              │
│     └─ Real-time status of all monitors             │
│     └─ Historical uptime charts                     │
│     └─ Incident log                                 │
│     └─ Public status page (optional)                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Monitor Check Process

```
Every 30 seconds (interval):

1. Check Started
   └─ Timer begins

2. Send Request
   └─ HTTP GET to monitored service
   └─ Timeout: 5 seconds

3. Response Received
   └─ Check response code (200-299 = UP)
   └─ Record response time
   └─ Mark as UP

4. Data Stored
   └─ Timestamp recorded
   └─ Response time logged
   └─ Uptime percentage updated

5. Next Check Scheduled
   └─ Schedule for next interval (30s later)
   └─ Loop continues
```

**If service is DOWN**:
```
Multiple Failed Checks:
└─ Check 1: DOWN (timeout)
└─ Check 2: DOWN (connection refused)
└─ Check 3: DOWN (HTTP 500)
└─ Triggered: ALERT SENT
└─ Status: SERVICE DOWN
└─ Dashboard: Shows RED
└─ Alert: Email, Slack, Discord sent
```

---

## 🚀 Getting Started

### 1. Start Uptime Kuma
```bash
cd applications/health\ check
docker-compose -f uptime-kuma/compose.yml up -d
```

### 2. Access the Dashboard
- **Local**: http://localhost:3001
- **Remote**: http://[your-server-ip]:3001

### 3. Initial Setup
1. Set admin username/password
2. Configure your preferred timezone
3. Add your first monitor
4. Configure notification channels

---

## 📈 Creating Monitors

### Monitor Types

#### 1. HTTP Monitor
```
Monitor Type: HTTP(s)
URL: https://example.com
Method: GET
Expected Status Code: 200
Timeout: 5 seconds
Interval: 30 seconds
```

**Use for**:
- Websites
- APIs
- Web services
- Status endpoints

#### 2. TCP/UDP Monitor
```
Monitor Type: TCP
Hostname: example.com
Port: 3306 (MySQL), 5432 (PostgreSQL), etc.
```

**Use for**:
- Databases
- Message queues
- Custom services

#### 3. Ping Monitor
```
Monitor Type: Ping
Hostname: example.com
Timeout: 5 seconds
```

**Use for**:
- Server availability
- Network connectivity

### Adding a Monitor: Example

**Monitor your Uptime Kuma instance itself**:

```
Name: Uptime Kuma Self-Check
Type: HTTP
URL: http://localhost:3001
Method: GET
Expected Status Code: 200
Timeout: 5 seconds
Interval: 30 seconds
Retries: 0
```

This creates a feedback loop - Uptime Kuma checks itself!

---

## 🔔 Alert Notifications

### Supported Channels

| Channel | Setup | Best For |
|---------|-------|----------|
| **Email** | SMTP server config | Team notifications |
| **Slack** | Webhook URL | Real-time chat |
| **Discord** | Webhook URL | Team alerts |
| **Telegram** | Bot token | Mobile alerts |
| **PagerDuty** | API key | Incident escalation |
| **Webhook** | Custom URL | Custom integrations |

### Setting Up Slack Alerts

1. **Create Slack App**
   - Go to api.slack.com
   - Create New App
   - Enable Incoming Webhooks

2. **Get Webhook URL**
   - Copy the Webhook URL

3. **Add to Uptime Kuma**
   - Settings → Notification Channels
   - Select: Slack
   - Paste Webhook URL
   - Test notification
   - Save

4. **Assign to Monitor**
   - Edit your monitor
   - Select Slack notification
   - Save

### Alert Workflow

```
Service Goes DOWN
       │
       ▼
Health Check Fails (after retries)
       │
       ▼
Uptime Kuma Detects Failure
       │
       ▼
Alert Triggered for Assigned Channels
       │
       ├─▶ Email sent to ops@company.com
       ├─▶ Slack message in #alerts channel
       ├─▶ SMS via Telegram
       └─▶ PagerDuty incident created
       │
       ▼
Team Notified Immediately
```

---

## 📊 Status Pages

### Public Status Page

Uptime Kuma can generate a public status page showing service health:

```
Example: https://status.example.com

Shows:
├─ All monitored services
├─ Current status (UP/DOWN)
├─ 30-day uptime percentage
├─ Incident history
├─ Maintenance windows
└─ Subscribers can get updates
```

### Creating a Status Page

1. **Access Settings**
   - Dashboard → Status Page

2. **Create New Page**
   - Choose public URL slug
   - Add title and description

3. **Add Monitors**
   - Select which monitors to display
   - Group by category (optional)

4. **Publish**
   - Share public URL with team/customers
   - Optionally make it private

---

## 🔄 Integration with AutoHeal

### How They Work Together

```
Uptime Kuma Monitor
       │
       ├─▶ Checks service health
       ├─▶ Records status changes
       └─▶ Sends alerts
            │
            ▼
      Service DOWN Detected
            │
            ▼
      Alert sent to ops team
            │
            ▼
      AutoHeal (running separately)
      detects health check failure
            │
            ▼
      AutoHeal restarts container
            │
            ▼
      Service comes back UP
            │
            ▼
      Uptime Kuma detects recovery
            │
            ▼
      Recovery alert sent
            │
            ▼
      Incident logged and resolved
```

### Real-World Example: Service Crash at 3 AM

```
Timeline:
─────────────────────────────────────────

03:00:00 - Normal Operations
          Uptime Kuma: All services UP
          AutoHeal: All containers healthy

03:05:23 - Service Crashes (Memory Leak)
          My-Service container exits
          Status: DOWN

03:05:24 - AutoHeal Detects Failure
          Health check fails
          Logs: "Container unhealthy"

03:05:30 - AutoHeal Restarts Container
          Stops crashed container
          Starts fresh container
          Waits for health

03:05:42 - Service Comes Online
          Container responds to health check
          Status: HEALTHY

03:05:45 - Uptime Kuma Detects Change
          Service responding again
          Status: UP
          Logs incident

03:05:46 - Alert Notifications Sent
          Email: "My-Service recovered"
          Slack: "🟢 My-Service is back online"
          Uptime Kuma logs incident
          Historical data updated

03:06:00 - Dashboard Updated
          Shows 99.99% uptime (1 min downtime)
          Incident visible in history
          Alert acknowledged by admin

Result: Service recovered in <40 seconds
        Team notified automatically
        No manual intervention needed
        Incident logged for review
```

---

## 💾 Data Backup

### What to Backup
```bash
# Backup Uptime Kuma data
cp -r ~/home/root/docker-kuma /backup/uptime-kuma-backup-$(date +%Y%m%d)
```

### Backup Script
```bash
#!/bin/bash
# backup-kuma.sh

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/uptime-kuma-$DATE"

mkdir -p "$BACKUP_PATH"
cp -r ~/home/root/docker-kuma "$BACKUP_PATH/"

echo "Backup completed: $BACKUP_PATH"
```

### Restore from Backup
```bash
# Stop Uptime Kuma
docker-compose -f uptime-kuma/compose.yml down

# Restore data
rm -rf ~/home/root/docker-kuma
cp -r /backup/uptime-kuma-20260515/* ~/home/root/

# Start Uptime Kuma
docker-compose -f uptime-kuma/compose.yml up -d
```

---

## 🔍 Monitoring & Logs

### View Uptime Kuma Logs
```bash
# All logs
docker logs uptime-kuma

# Follow in real-time
docker logs -f uptime-kuma

# Last 50 lines with timestamps
docker logs -t --tail 50 uptime-kuma
```

### Log Examples
```
[2026-05-15T13:00:00] Starting Uptime Kuma
[2026-05-15T13:00:05] Database initialized
[2026-05-15T13:00:10] Loaded 5 monitors
[2026-05-15T13:00:30] Monitor check: example.com - UP (120ms)
[2026-05-15T13:01:00] Monitor check: example.com - UP (118ms)
[2026-05-15T13:05:00] Monitor check: myapp.com - DOWN (timeout)
[2026-05-15T13:05:00] Alert triggered: myapp.com is DOWN
[2026-05-15T13:05:01] Notification sent to Slack
[2026-05-15T13:05:30] Monitor check: myapp.com - UP (150ms)
[2026-05-15T13:05:30] Alert resolved: myapp.com is UP
```

---

## 📊 Performance Metrics

### Resource Usage
```bash
# Check container resource usage
docker stats uptime-kuma

# Example output:
CONTAINER ID  MEM USAGE   MEM %   CPU %   NET I/O
abc12345...   150MiB      2.5%    0.1%    2.3MB/1.2MB
```

### Typical Resource Requirements
| Component | Typical Usage |
|-----------|---------------|
| Memory | 100-200 MB |
| CPU | <1% (idle), <5% (checking) |
| Disk | 100MB+ (depends on history) |
| Network | <1 MB/day (depends on monitors) |

---

## 🛡️ Security Considerations

### Network Isolation
```yaml
networks:
  - kuma_network    # Only containers on this network can reach it
```

### Access Control
- ✅ Set strong admin password
- ✅ Use HTTPS for remote access (reverse proxy recommended)
- ✅ Limit port 3001 access to trusted networks
- ✅ Enable 2FA if available

### Data Security
- ✅ Backup data regularly
- ✅ Store credentials securely (use Docker secrets in production)
- ✅ Don't expose dashboard to the internet directly
- ✅ Use reverse proxy with SSL/TLS

### Reverse Proxy Example (Nginx)
```nginx
server {
    listen 443 ssl http2;
    server_name kuma.example.com;
    
    ssl_certificate /etc/letsencrypt/live/kuma.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/kuma.example.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 🆘 Troubleshooting

### Uptime Kuma Won't Start

**Check 1**: Port 3001 already in use
```bash
# Find what's using port 3001
lsof -i :3001

# Kill the process
kill -9 [PID]
```

**Check 2**: Volume permission issues
```bash
# Fix permissions
chmod -R 755 ~/home/root/docker-kuma

# Restart Uptime Kuma
docker-compose -f uptime-kuma/compose.yml restart uptime-kuma
```

**Check 3**: Inspect logs
```bash
docker logs uptime-kuma
```

### Monitors Not Checking

**Check 1**: Is the service URL reachable?
```bash
curl -v http://example.com
```

**Check 2**: Is network properly configured?
```bash
docker network inspect kuma_network
```

**Check 3**: Monitor settings
- Verify interval is not too long
- Check timeout is appropriate
- Verify target service is responding

### Alerts Not Sending

**Check 1**: Notification channel configured?
```bash
# Check in dashboard settings
Settings → Notification
```

**Check 2**: Monitor assigned to channel?
```bash
# Edit monitor
Edit Monitor → Notification Channels
```

**Check 3**: Test notification
```bash
# Send test notification in settings
Settings → [Channel] → Test
```

### High Memory Usage

**Check 1**: Prune old data
```bash
# Reduce historical data retention
Settings → Data → Prune data older than 30 days
```

**Check 2**: Reduce check frequency
```bash
# Increase interval for non-critical monitors
Edit Monitor → Interval: 5m (instead of 30s)
```

**Check 3**: Restart to clear cache
```bash
docker-compose -f uptime-kuma/compose.yml restart uptime-kuma
```

---

## 📚 Advanced Features

### Status Page Customization
- Customize colors and branding
- Add company logo and links
- Enable subscriber notifications
- Schedule maintenance windows

### Monitor Groups
- Organize monitors by service
- Group related services
- Better dashboard organization
- Easier incident management

### Incident Tracking
- Automatic incident creation
- Manual incident logging
- Duration tracking
- Impact assessment

### Heartbeat API
```bash
# Send heartbeat from your service
curl -X POST https://your-kuma.com/api/push/[monitor-key]

# This tells Kuma "I'm alive"
# Used for background jobs, cron tasks, etc.
```

---

## 🔄 Best Practices

✅ **DO:**
- Monitor critical services every 30 seconds
- Set up alerts for all production services
- Configure multiple notification channels
- Review incident logs weekly
- Test health checks regularly
- Back up configuration daily
- Monitor Uptime Kuma itself
- Document monitor purposes

❌ **DON'T:**
- Don't check too frequently (causes high load)
- Don't ignore repeated failures (indicates real problem)
- Don't skip backup strategy
- Don't expose dashboard to untrusted networks
- Don't store sensitive credentials in URLs
- Don't check all services at the same time (stagger intervals)

---

## 📖 Related Documentation

- **AutoHeal Integration**: See `../autoheal/README.md`
- **Main Health Check Stack**: See `../README.md`
- **Official Uptime Kuma**: https://uptime.kuma.pet/
- **Docker Documentation**: https://docs.docker.com/

---

**Documentation Version**: 2026-05-15
**Uptime Kuma Version**: 2
**Last Updated**: 2026-05-15
