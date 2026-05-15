# Cloudflare Tunnel Watchdog - Setup Guide

## Overview

This guide walks you through setting up an automated Cloudflare tunnel health monitor that detects and restarts unhealthy containers. This addresses **Cloudflare Tunnel Error 1033** (connection issues) by maintaining persistent tunnel connectivity.

---

## Step 1: Install the Script

### 1.1 Create the Script File

Copy the watchdog script to `/usr/local/bin` for system-wide access:

```bash
sudo nano /usr/local/bin/cf-tunnel-watchdog.sh
```

### 1.2 Paste the Script Content

```bash
#!/bin/bash
# ============================================================
# cf-tunnel-watchdog.sh
# Checks Cloudflare tunnel status via API and restarts
# the matching Docker container if inactive or down.
# ============================================================

ACCOUNT_ID="Cloudflare Account ID"
API_TOKEN="token_here"
LOG_FILE="/var/log/cf-tunnel-watchdog.log"

# Tunnel ID <-> Docker container name
declare -A TUNNELS
TUNNELS["id"]="cloudflared"
TUNNELS["id"]="cloudflared2"
TUNNELS["id"]="cloudflared3"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ℹ️  $1" | tee -a "$LOG_FILE"
}

for TUNNEL_ID in "${!TUNNELS[@]}"; do
    CONTAINER="${TUNNELS[$TUNNEL_ID]}"

    # Query Cloudflare API for tunnel status
    RESPONSE=$(curl --silent --fail \
        -H "Authorization: Bearer $API_TOKEN" \
        -H "Content-Type: application/json" \
        "https://api.cloudflare.com/client/v4/accounts/$ACCOUNT_ID/cfd_tunnel/$TUNNEL_ID")

    if [ $? -ne 0 ]; then
        log "API request failed for $CONTAINER ($TUNNEL_ID). Skipping."
        continue
    fi

    STATUS=$(echo "$RESPONSE" | grep -o '"status":"[^"]*"' | head -1 | cut -d'"' -f4)

    log "$CONTAINER → tunnel status: $STATUS"

    if [ "$STATUS" = "inactive" ] || [ "$STATUS" = "down" ]; then
        log "WARNING: $CONTAINER is $STATUS. Restarting container..."
        docker restart "$CONTAINER"
        [ $? -eq 0 ] && log "✅ $CONTAINER restarted successfully." || log "❌ Failed to restart $CONTAINER."
    fi

done
```

### 1.3 Make the Script Executable

```bash
sudo chmod +x /usr/local/bin/cf-tunnel-watchdog.sh
```

---

## Step 2: Configure the Script

Edit the script to add your credentials:

```bash
sudo nano /usr/local/bin/cf-tunnel-watchdog.sh
```

Replace the following placeholders:

- **`ACCOUNT_ID`**: Your Cloudflare Account ID
- **`API_TOKEN`**: Your Cloudflare API token
- **`TUNNELS` array**: Map your actual tunnel IDs to Docker container names

### 2.1 Getting Your Credentials

#### Find Account ID:
```bash
# Via Cloudflare Dashboard: Settings > General > Account ID
```

#### Create API Token:
1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Account > Tokens & Keys > Create API Token
3. Use template: **Edit Cloudflare Tunnel**
4. Grant permissions to your tunnels

#### Find Tunnel IDs:
```bash
# Via Cloudflare CLI or Dashboard
cloudflare-cli tunnel list

# Or check your Docker container logs
docker logs cloudflared | grep -i "tunnel"
```

---

## Step 3: Set Up the Log File

Create and configure the log file:

```bash
# Create log file with proper permissions
sudo touch /var/log/cf-tunnel-watchdog.log
sudo chmod 644 /var/log/cf-tunnel-watchdog.log

# Optional: Set up log rotation
sudo nano /etc/logrotate.d/cf-tunnel-watchdog
```

Add this to `/etc/logrotate.d/cf-tunnel-watchdog`:

```
/var/log/cf-tunnel-watchdog.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
}
```

---

## Step 4: Test the Script

Run the script manually to verify it works:

```bash
sudo /usr/local/bin/cf-tunnel-watchdog.sh
```

### Expected Output:

```
[2026-05-15 17:17:40] ℹ️  cloudflared2 → tunnel status: healthy
[2026-05-15 17:17:40] ℹ️  cloudflared → tunnel status: healthy
[2026-05-15 17:17:40] ℹ️  cloudflared3 → tunnel status: healthy
```

If you see **inactive** or **down** statuses, the script will automatically restart the container and log:

```
[2026-05-15 17:17:45] ℹ️  WARNING: cloudflared is inactive. Restarting container...
[2026-05-15 17:17:46] ℹ️  ✅ cloudflared restarted successfully.
```

---

## Step 5: Schedule with Cron

### 5.1 Edit the Crontab

```bash
sudo crontab -e
```

### 5.2 Add Cron Job

Choose an interval based on your needs:

#### **Every 5 minutes** (aggressive monitoring):
```cron
*/5 * * * * /usr/local/bin/cf-tunnel-watchdog.sh
```

#### **Every 15 minutes** (balanced):
```cron
*/15 * * * * /usr/local/bin/cf-tunnel-watchdog.sh
```

#### **Every hour** (light monitoring):
```cron
0 * * * * /usr/local/bin/cf-tunnel-watchdog.sh
```

#### **Custom: Run at specific times** (e.g., every 10 minutes):
```cron
*/10 * * * * /usr/local/bin/cf-tunnel-watchdog.sh >> /var/log/cf-tunnel-watchdog-cron.log 2>&1
```

### 5.3 Verify Cron Installation

```bash
sudo crontab -l
```

You should see your entry listed.

---

## Step 6: Monitor Logs

### View Real-Time Logs:

```bash
sudo tail -f /var/log/cf-tunnel-watchdog.log
```

### Search for Errors or Restarts:

```bash
# Find all restart events
sudo grep "Restarting container" /var/log/cf-tunnel-watchdog.log

# Find all status checks
sudo grep "tunnel status" /var/log/cf-tunnel-watchdog.log

# Find failed API requests
sudo grep "API request failed" /var/log/cf-tunnel-watchdog.log
```

### Example Log Output:

```
[2026-05-15 17:17:40] ℹ️  cloudflared2 → tunnel status: healthy
[2026-05-15 17:17:40] ℹ️  cloudflared → tunnel status: healthy
[2026-05-15 17:17:40] ℹ️  cloudflared3 → tunnel status: healthy
[2026-05-15 17:22:40] ℹ️  cloudflared → tunnel status: inactive
[2026-05-15 17:22:40] ℹ️  WARNING: cloudflared is inactive. Restarting container...
[2026-05-15 17:22:41] ℹ️  ✅ cloudflared restarted successfully.
[2026-05-15 17:22:45] ℹ️  cloudflared → tunnel status: healthy
```

---

## Troubleshooting

### Issue: Script shows "API request failed"

**Solution:**
- Verify `ACCOUNT_ID` and `API_TOKEN` are correct
- Check API token permissions include tunnel access
- Ensure the machine can reach `api.cloudflare.com`

```bash
curl -I https://api.cloudflare.com/client/v4/accounts
```

### Issue: Docker container names don't match

**Solution:**
List all containers and update the `TUNNELS` array:

```bash
docker ps --format "{{.Names}}"
```

### Issue: Cron job not running

**Solution:**
Check if cron is enabled and running:

```bash
sudo systemctl status cron
sudo systemctl start cron
```

Check cron logs:

```bash
sudo grep CRON /var/log/syslog
```

### Issue: Permission denied when running script

**Solution:**
```bash
sudo chmod +x /usr/local/bin/cf-tunnel-watchdog.sh
```

---

## Addressing Cloudflare Error 1033

**Error 1033** typically indicates:
- Tunnel is disconnected or inactive
- Connection issues between daemon and Cloudflare edge
- Container crash or restart loop

### How This Script Mitigates Error 1033:

1. **Continuous Health Checks**: Monitors tunnel status every 5-15 minutes
2. **Automatic Recovery**: Restarts unhealthy containers immediately
3. **Logging & Visibility**: Tracks all status changes for debugging
4. **Zero-Downtime Monitoring**: Uses API instead of container logs

---

## Advanced Configuration

### Email Alerts on Container Restart

```bash
# Install mail utilities
sudo apt-get install mailutils

# Modify the log function to send email
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ℹ️  $1" | tee -a "$LOG_FILE"
    if [[ "$1" == *"Restarting"* ]]; then
        echo "$1" | mail -s "Tunnel Watchdog Alert" admin@yourdomain.com
    fi
}
```

### Webhook Integration

Send alerts to Discord, Slack, or Teams:

```bash
log_with_webhook() {
    local message="$1"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ℹ️  $message" >> "$LOG_FILE"
    
    # Send to Discord webhook
    curl -X POST "YOUR_WEBHOOK_URL" \
        -H "Content-Type: application/json" \
        -d "{\"content\":\"$message\"}"
}
```

---

## Summary

| Step | Command |
|------|---------|
| Install | `sudo cp /path/to/script /usr/local/bin/cf-tunnel-watchdog.sh && sudo chmod +x /usr/local/bin/cf-tunnel-watchdog.sh` |
| Configure | `sudo nano /usr/local/bin/cf-tunnel-watchdog.sh` (add credentials) |
| Test | `sudo /usr/local/bin/cf-tunnel-watchdog.sh` |
| Schedule | `sudo crontab -e` (add cron entry) |
| Monitor | `sudo tail -f /var/log/cf-tunnel-watchdog.log` |

---

## References

- [Cloudflare Tunnel API Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/)
- [Cloudflare Error 1033](https://developers.cloudflare.com/support/cloudflare-tunnel/)
- [Cron Job Guide](https://linux.die.net/man/5/crontab)
