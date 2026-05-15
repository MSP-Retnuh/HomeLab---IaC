# AutoHeal - Automatic Container Recovery

**AutoHeal** is a Docker container auto-healing tool that monitors the health of your containers and automatically restarts unhealthy ones. It creates a self-healing Docker infrastructure by continuously watching container health and taking corrective action.

## 📋 Quick Overview

| Property | Value |
|----------|-------|
| **Image** | `willfarrell/autoheal:latest` |
| **Container Restart Policy** | Always |
| **Container Label Monitor** | All (`all`) |
| **Socket Mount** | `/var/run/docker.sock` (Docker daemon access) |
| **Network** | Host (shared) |

---

## 🚀 Configuration Breakdown

### Service Definition
```yaml
autoheal:
  image: willfarrell/autoheal:latest                    # Latest stable version
  container_name: autoheal                              # Fixed container name
  restart: always                                       # Keep running at all times
```

### Environment Variables
```yaml
environment:
  - AUTOHEAL_CONTAINER_LABEL=all                       # Monitor ALL containers
```

**Label Options**:
- `all` - Monitor every container (default)
- `autoheal=true` - Monitor only containers with this label
- `autoheal=false` - Skip monitoring for labeled containers

### Docker Socket Mount
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock         # Access Docker daemon
```

**Why this is needed**:
- ✅ AutoHeal needs to read container status
- ✅ Can restart containers via Docker API
- ✅ Can inspect container health checks
- ✅ Can access container logs

**Security Note**: This gives AutoHeal full Docker access. In production, consider:
- Running in a restricted network
- Using Docker context/scope restrictions
- Auditing restart events

---

## 🔄 How AutoHeal Works

### Container Health Check Detection

```
Docker Health Check States
       │
       ├─ HEALTHY (Green)
       │  └─ Container is working correctly
       │
       ├─ UNHEALTHY (Red)
       │  └─ Health check failed
       │  └─ AutoHeal will restart
       │
       └─ STARTING
          └─ Container still initializing
          └─ AutoHeal waits for start_period
```

### Self-Healing Workflow

```
1. Container is Running
   └─ Health check runs every interval (30s for Uptime Kuma)

2. Health Check Fails (3 times)
   └─ Container marked as UNHEALTHY
   └─ AutoHeal detects status change

3. AutoHeal Decides to Restart
   └─ Checks if restart is allowed
   └─ Logs the unhealthy status
   └─ Triggers container restart

4. Container Restarts
   └─ Old container stopped
   └─ New container started
   └─ Waits for start_period (10s for Uptime Kuma)

5. Health Check Resumes
   └─ New health checks begin
   └─ If passes → HEALTHY
   └─ If fails again → Restart again

6. Service Restored
   └─ Uptime Kuma records the incident
   └─ Alert sent to notify ops team
   └─ Monitoring resumes normally
```

### Real-World Example: Uptime Kuma Crash

```
Timeline:
────────────────────────────────────────────────────────────

13:00:00 - Normal Operation
          Uptime Kuma running, monitoring services

13:05:23 - Memory Leak Causes Crash
          Uptime Kuma container crashes
          Status: UNHEALTHY

13:05:30 - AutoHeal Detects Crash
          Receives health check failure
          Logs: "Container unhealthy after 3 retries"

13:05:35 - AutoHeal Restarts Container
          Stops crashed container
          Starts new Uptime Kuma container
          Waits 10 seconds (start_period)

13:05:48 - Health Check Resumes
          New container responds to health check
          Status: HEALTHY

13:05:50 - Uptime Kuma Comes Online
          Dashboard accessible again
          Resumes monitoring

13:05:55 - Notification Sent
          Alert system triggered
          "Uptime Kuma recovered from crash"

13:06:00 - Back to Normal
          Service fully recovered
          No manual intervention needed
```

---

## 📊 Integration with Uptime Kuma

### How They Work Together

```
┌──────────────────────────────────────────────────────────────┐
│                  Self-Healing Monitoring Loop                 │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  Container Running                                             │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────────────────────────────────┐          │
│  │  Health Check (curl to http://localhost:3001)   │          │
│  │  Run every 30 seconds                           │          │
│  └─────────────────────────────────────────────────┘          │
│       │                        │                               │
│       ▼ (SUCCESS)             ▼ (FAILURE)                     │
│  ┌─────────┐            ┌──────────────┐                      │
│  │ HEALTHY │            │  Fail Count  │                      │
│  │ Status  │            │  increases   │                      │
│  └─────────┘            │  (max: 3)    │                      │
│       │                 └──────────────┘                       │
│       │                        │                               │
│       │                        ▼                               │
│       │                 ┌─────────────────────┐               │
│       │                 │ 3 Failures Reached? │               │
│       │                 │ Container Unhealthy │               │
│       │                 └────────────┬────────┘               │
│       │                              │                        │
│       │                              ▼                        │
│       │                 ┌──────────────────────┐              │
│       │                 │   AutoHeal Detects   │              │
│       │                 │   Unhealthy Status   │              │
│       │                 └────────────┬─────────┘              │
│       │                              │                        │
│       │                              ▼                        │
│       │                 ┌──────────────────────┐              │
│       │                 │ Restart Container    │              │
│       │                 │ Stop + Start         │              │
│       │                 └────────────┬─────────┘              │
│       │                              │                        │
│       │                 Wait start_period (10s)               │
│       │                              │                        │
│       │                              ▼                        │
│       │                 ┌──────────────────────┐              │
│       └────────────────▶│  Resume Health Checks│              │
│                         │  (Loop back to top)  │              │
│                         └──────────────────────┘              │
│                                                                │
│  Meanwhile - Uptime Kuma Dashboard:                            │
│  ├─ Detects service went DOWN                                │
│  ├─ Records incident timestamp                               │
│  ├─ Detects service came UP                                  │
│  ├─ Records recovery timestamp                               │
│  ├─ Sends alert to configured channels                       │
│  └─ Updates status page                                      │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

### Notification Flow

```
AutoHeal Restarts Container
         │
         ▼
Uptime Kuma Detects State Change
         │
         ├─ Service went DOWN → sends alert
         │
         └─ Service came UP → sends recovery notification
                    │
                    ▼
              Channels Configured:
              ├─ Email
              ├─ Slack
              ├─ Discord
              ├─ Telegram
              └─ etc.
                    │
                    ▼
              Alert Delivered to Team
```

---

## 🔍 Container Health Check States

AutoHeal monitors these health states:

| State | Description | AutoHeal Action |
|-------|-------------|-----------------|
| **HEALTHY** ✅ | Container is working | Do nothing, continue monitoring |
| **UNHEALTHY** ❌ | Health check failed 3 times | **Restart container** |
| **STARTING** ⏳ | Container initializing | Wait for `start_period`, then check |
| **NONE** ⏹️ | No health check defined | Monitor using restart policy |

### Health Check Definition (from Uptime Kuma)

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3001"]  # The command to run
  interval: 30s                                          # Check every 30 seconds
  retries: 3                                             # Fail after 3 consecutive failures
  start_period: 10s                                      # Wait 10s after startup before checking
  timeout: 5s                                            # Timeout if no response in 5 seconds
```

**Example Timeline**:
```
00:00 - Container starts
00:10 - First health check (start_period elapsed)
00:40 - Second health check (interval: 30s)
00:45 - FAILS (timeout: 5s)
01:10 - Third health check
01:15 - FAILS again
01:40 - Fourth health check
01:45 - FAILS AGAIN (3 consecutive failures = UNHEALTHY)
       └─ AutoHeal triggers restart
```

---

## 🚀 Setup & Configuration

### 1. Start AutoHeal
```bash
docker-compose -f applications/health\ check/autoheal/compose.yml up -d
```

### 2. Verify It's Running
```bash
# Check container status
docker ps | grep autoheal

# View logs
docker logs autoheal

# Should see output like:
# 2026-05-15 13:00:00 - Monitoring all containers for health
```

### 3. Test the Auto-Healing

**Test by crashing a container**:
```bash
# Find a container to test (or use Uptime Kuma)
docker ps

# Kill the container to simulate a crash
docker kill [container-id]

# Watch AutoHeal restart it
docker logs -f autoheal

# Check it restarted
docker ps | grep [container-name]
```

---

## 📋 Advanced Configuration

### Monitor Only Specific Containers

Instead of `AUTOHEAL_CONTAINER_LABEL=all`, use:

```yaml
environment:
  - AUTOHEAL_CONTAINER_LABEL=autoheal
```

Then add labels to specific containers:

```yaml
services:
  my-app:
    image: myapp:latest
    labels:
      autoheal: "true"           # This container will be monitored
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
```

### Environment Variables

```yaml
environment:
  - AUTOHEAL_CONTAINER_LABEL=all             # Which containers to monitor
  - AUTOHEAL_INTERVAL=5                      # Check interval (seconds)
  - AUTOHEAL_START_PERIOD=0                  # Grace period after startup
  - AUTOHEAL_DEFAULT_STOP_TIMEOUT=10         # Timeout when stopping containers
  - DOCKER_SOCK=/var/run/docker.sock         # Path to Docker socket
  - WEBHOOK_URL=                             # Optional webhook for notifications
```

---

## 📊 Monitoring & Logging

### View AutoHeal Logs
```bash
# All logs
docker logs autoheal

# Follow in real-time
docker logs -f autoheal

# Last 50 lines
docker logs --tail 50 autoheal

# With timestamps
docker logs -t autoheal
```

### Log Output Examples

```
# Container restarted
[2026-05-15 13:05:23] Container 'uptime-kuma' found unhealthy
[2026-05-15 13:05:23] Restarting container 'uptime-kuma'
[2026-05-15 13:05:25] Container 'uptime-kuma' restarted successfully

# Container monitoring
[2026-05-15 13:00:00] Starting container health monitoring
[2026-05-15 13:00:00] Found 5 containers to monitor
[2026-05-15 13:00:05] All containers healthy
```

### Check AutoHeal Itself

Since AutoHeal has `restart: always`, Docker will restart it if it crashes:

```bash
# Check AutoHeal container status
docker inspect autoheal | grep -A 5 State

# View restart count
docker inspect autoheal | grep RestartCount
```

---

## 🚨 Failure Scenarios & Recovery

### Scenario 1: Service Memory Leak
```
Problem: Container uses more memory over time until it crashes
Detection: Health check fails
Recovery: AutoHeal restarts container with fresh memory
Result: Service recovered without manual intervention
```

### Scenario 2: Database Connection Pool Exhaustion
```
Problem: Service can't connect to database anymore
Detection: Health check to /health endpoint fails
Recovery: AutoHeal restarts, reconnects to database
Result: Connection pool reset, service healthy again
```

### Scenario 3: Deadlock in Application
```
Problem: Application locks up, stops responding
Detection: Health check timeout (5 seconds)
Recovery: AutoHeal kills and restarts container
Result: Fresh state, application responsive again
```

### Scenario 4: Cascading Failures
```
Problem: Multiple services fail at once
Detection: AutoHeal restarts each independently
Recovery: Services come back online in sequence
Result: Full recovery without single point of failure
```

---

## 🔐 Security Considerations

### Docker Socket Access
- ⚠️ AutoHeal needs `/var/run/docker.sock` to manage containers
- ⚠️ This gives it full Docker API access
- ✅ Only run AutoHeal on trusted infrastructure
- ✅ Limit to host network (not exposed to internet)
- ✅ Monitor AutoHeal logs for unexpected restarts

### Audit Restart Events
```bash
# Check Docker events
docker events --filter type=container --filter event=start

# Or check AutoHeal logs for restart history
docker logs autoheal | grep "restarted"
```

### Prevent Restart Loops
AutoHeal has safeguards:
- ✅ Won't restart if `restart: unless-stopped` policy allows manual stop
- ✅ Won't restart on every failure (respects restart cooldown)
- ✅ Can exclude containers with `autoheal: false` label

---

## 🆘 Troubleshooting

### AutoHeal Not Restarting Containers

**Check 1**: Is AutoHeal running?
```bash
docker ps | grep autoheal
```

**Check 2**: Are containers being monitored?
```bash
docker logs autoheal | head -20
```

**Check 3**: Do containers have health checks?
```bash
docker inspect [container-name] | grep -A 5 Healthcheck
```

**Check 4**: Is the health check failing?
```bash
docker inspect [container-name] | grep -A 3 State
```

### Containers Restarting Too Frequently

**Check 1**: Review health check sensitivity
- Increase timeout if false positives
- Example: `timeout: 10s` (was 5s)

**Check 2**: Check for actual service issues
```bash
docker logs [container-name]
```

**Check 3**: Review restart limits
- Consider max restart attempts
- Add restart delay if needed

### Docker Socket Permission Issues

```bash
# Check socket access
ls -l /var/run/docker.sock

# If permission denied, check user
docker inspect autoheal | grep User
```

---

## 📈 Metrics & Monitoring

### Track Restart Events
```bash
# Count restarts for a container
docker inspect [container-name] | grep RestartCount

# View restart history
docker logs [container-name] | grep -i restart
```

### Monitor AutoHeal Health
```bash
# Check if AutoHeal itself is healthy
docker inspect autoheal

# Resource usage
docker stats autoheal

# Check for errors in logs
docker logs autoheal | grep -i error
```

---

## 🔄 Best Practices

✅ **DO:**
- Always set health checks on containers
- Use appropriate health check intervals (30-60s)
- Monitor AutoHeal logs for patterns
- Test health checks before production
- Have Uptime Kuma alerts configured
- Document health check logic
- Review restart events weekly

❌ **DON'T:**
- Don't ignore repeated restarts (indicates real problem)
- Don't set health checks too frequently (high CPU)
- Don't assume AutoHeal will fix all problems
- Don't run without monitoring the monitoring system
- Don't skip persistent storage backup
- Don't expose Docker socket to untrusted networks

---

## 📚 Related Documentation

- **Uptime Kuma Integration**: See `../uptime-kuma/README.md`
- **Main Health Check Stack**: See `../README.md`
- **Docker Health Checks**: https://docs.docker.com/engine/reference/builder/#healthcheck
- **AutoHeal Project**: https://github.com/willfarrell/docker-autoheal

---

## 💡 Common Use Cases

### Use Case 1: Zero-Downtime Infrastructure
```
AutoHeal + Uptime Kuma + Load Balancer
= Automatic failover and recovery
```

### Use Case 2: Development Environment Stability
```
Developers push code
Container crashes
AutoHeal restarts immediately
Developers notified via alert
```

### Use Case 3: Production Resilience
```
3 AM: Service crashes due to memory leak
AutoHeal restarts within seconds
Uptime Kuma records and alerts ops
Next morning: Team reviews logs
```

### Use Case 4: Multi-Service Monitoring
```
5 microservices deployed
Each has health check
AutoHeal monitors all
Uptime Kuma aggregates status
Public status page shows overall health
```

---

**Documentation Version**: 2026-05-15
**AutoHeal Version**: Latest
**For more info**: https://github.com/willfarrell/docker-autoheal
