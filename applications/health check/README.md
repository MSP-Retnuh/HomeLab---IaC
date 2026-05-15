# Health Check & Monitoring Stack

This folder contains Docker Compose configurations for a comprehensive health check and monitoring ecosystem. The stack combines **Uptime Kuma** (monitoring & alerting) with **AutoHeal** (automatic container recovery) to create a self-healing infrastructure.

## 📋 Services Overview

### 1. **Uptime Kuma** - Monitoring & Uptime Tracking
- **Purpose**: Monitors container health and tracks uptime
- **Port**: 3001
- **Features**: Web UI for dashboards, alerting, status pages
- **Folder**: `./uptime-kuma/`

### 2. **AutoHeal** - Automatic Container Recovery
- **Purpose**: Automatically restarts unhealthy containers
- **Features**: Continuous health checking, self-healing mechanism
- **Folder**: `./autoheal/`

---

## 🔄 How They Work Together

```
┌─────────────────────────────────────────────────────────────┐
│                   Health Check Stack                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────┐         ┌──────────────────────┐  │
│  │   Uptime Kuma        │         │    AutoHeal          │  │
│  │  (Port 3001)         │         │  (Monitors All)      │  │
│  │                      │         │                      │  │
│  │ • Monitors services  │────────▶│ • Watches health     │  │
│  │ • Sends alerts       │◀────────│ • Restarts failed    │  │
│  │ • Status dashboards  │         │ • Reports status     │  │
│  │ • Status pages       │         │ • Self-healing       │  │
│  └──────────────────────┘         └──────────────────────┘  │
│           │                                  │                │
│           ▼                                  ▼                │
│  ┌─────────────────────────────────────────────────┐         │
│  │     All Other Services (Docker Containers)      │         │
│  │                                                 │         │
│  │  • Web Apps  • Databases  • APIs  • Workers    │         │
│  └─────────────────────────────────────────────────┘         │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Workflow Example: Container Failure & Recovery

```
1. Service Container Crashes
   └─▶ Fails health check

2. AutoHeal Detects Failure
   └─▶ Restarts container automatically

3. Uptime Kuma Detects Change
   └─▶ Logs the incident
   └─▶ Sends alert notification (if configured)
   └─▶ Updates status dashboard

4. Container Recovers
   └─▶ Service restored
   └─▶ Uptime Kuma confirms recovery
   └─▶ Alert cleared
```

---

## 🚀 Quick Start

### 1. Start the Stack
```bash
cd applications/health\ check
docker-compose -f uptime-kuma/compose.yml up -d
docker-compose -f autoheal/compose.yml up -d
```

### 2. Access Uptime Kuma Dashboard
- Open: `http://localhost:3001`
- Set up monitoring for your services
- Configure alerts and status pages

### 3. Verify AutoHeal is Running
```bash
docker logs autoheal
```

---

## 📊 Real-World Scenario

**Scenario**: Your web application container crashes at 3 AM

**What happens**:
1. **3:00 AM** - Application container crashes
2. **3:00 AM** - AutoHeal detects the crash (via health check)
3. **3:00 AM** - AutoHeal automatically restarts the container
4. **3:01 AM** - Container is back online and healthy
5. **3:02 AM** - Uptime Kuma detects recovery
6. **3:02 AM** - You receive an alert about the incident and recovery
7. **No manual intervention needed** ✅

---

## 🔧 Configuration Notes

### Health Check Interval
- **Uptime Kuma**: 30-second health check intervals
- **AutoHeal**: Monitors all containers continuously
- **Retries**: 3 attempts before marking as failed

### Persistent Storage
- **Uptime Kuma Data**: `~/home/root/docker-kuma:/app/data`
- **Configured Monitoring**: Saved and preserved across restarts

### Network Isolation
- Services run on custom `kuma_network` for isolated communication
- Can integrate with other networks as needed

---

## 📝 File Structure

```
applications/health check/
├── README.md                    (This file)
├── uptime-kuma/
│   ├── README.md               (Uptime Kuma configuration & usage)
│   └── compose.yml             (Docker Compose config)
└── autoheal/
    ├── README.md               (AutoHeal configuration & usage)
    └── compose.yml             (Docker Compose config)
```

---

## ⚙️ Integration Points

### Uptime Kuma ↔ AutoHeal
- **AutoHeal** uses health checks (curl requests to port 3001)
- **Uptime Kuma** provides the monitoring UI and alerting
- **Together** they form a feedback loop for continuous health management

### Adding More Services to Monitor
1. Add health checks to your service compose files
2. Add service monitoring URLs to Uptime Kuma dashboard
3. AutoHeal will automatically watch all labeled containers

---

## 🛡️ Best Practices

- ✅ Always set restart policies (`restart: unless-stopped` or `restart: always`)
- ✅ Configure health checks with appropriate intervals
- ✅ Monitor logs from both Uptime Kuma and AutoHeal
- ✅ Set up alerts/notifications for critical services
- ✅ Use persistent storage for Uptime Kuma data
- ✅ Review AutoHeal restart logs regularly
- ✅ Test your health checks before production deployment

---

## 📖 For More Details

- See `./uptime-kuma/README.md` for Uptime Kuma configuration
- See `./autoheal/README.md` for AutoHeal setup and usage
- Check Docker logs: `docker logs [container-name]`

---

## 🆘 Troubleshooting

| Issue | Solution |
|-------|----------|
| Services not monitoring | Check if health check endpoints are accessible |
| AutoHeal not restarting containers | Verify `AUTOHEAL_CONTAINER_LABEL=all` is set |
| Uptime Kuma not responding | Check port 3001 and persistent storage permissions |
| High restart loops | Review and adjust health check intervals/timeouts |

---

**Last Updated**: 2026-05-15
