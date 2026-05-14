# Media Portal (Jellyfin + Jellyseerr + Jellyprobe)

Docker Compose configuration for a complete media streaming portal.

## Overview

Full-featured media streaming and management platform:
- **Jellyfin** - Self-hosted media server
- **Jellyseerr** - Media request management UI
- **Jellyprobe** - Health monitoring and status page

## Services Included

- **Jellyfin** - Streaming server
- **Jellyseerr** - Request and discovery interface
- **Jellyprobe** - Service monitoring
- **MariaDB** (optional) - Database backend

## Files

- `docker-compose.yml` - Service definitions
- `.env.example` - Environment variables template
- `config/` - Configuration directories

## Quick Start

1. Copy `.env.example` to `.env`
2. Configure media paths and credentials
3. Run: `docker-compose up -d`
4. Access at configured domain/port

## Service Ports

- **Jellyfin** - 8096 (default)
- **Jellyseerr** - 5055 (default)
- **Jellyprobe** - 8080 (default)

## Features

- Multiple user profiles
- Content recommendations
- Request management
- Health status monitoring
- Remote access support

## Integration

- Arr Stack provides media library
- Media Bridge enhances streaming
- Secure Vault manages credentials

## Documentation

See the main [README](../../README.md) for general setup instructions.
