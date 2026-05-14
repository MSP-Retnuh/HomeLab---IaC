# Applications

Docker Compose configurations for various homelab applications and services.

## Application Folders

### 🌉 [Media Bridge](./DUMB)
Debrid Unlimited integration for seamless media streaming.

### 📺 [Arr Stack](./media-portal)
Complete media management suite (Sonarr, Radarr, Lidarr, Readarr, Prowlarr).

### 📹 [Media Portal](./DUMB)
Jellyfin media server with Jellyseerr request management and Jellyprobe monitoring.

### 🔐 [Secure Vault](./secure-vault)
Vaultwarden password manager with Nginx reverse proxy for secure access.

---

## General Usage

Each application folder contains:
- `docker-compose.yml` - Service definitions
- `.env.example` - Environment variables template
- `README.md` - Application-specific documentation

### Getting Started with Any Application

1. Navigate to the application folder
2. Copy `.env.example` to `.env`
3. Configure environment variables
4. Run: `docker-compose up -d`

## Network Architecture

Applications are designed to work together:
- **Arr Stack** ← Downloads media
- **Media Bridge** ← Provides streaming sources
- **Media Portal** ← Streams to users
- **Secure Vault** ← Manages credentials

## Documentation

For detailed setup instructions, see the main [README](../README.md).

---

**Tip:** Use `docker-compose logs -f` to monitor application logs.
