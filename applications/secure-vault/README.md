# Secure Vault (Vaultwarden + Nginx Proxy)

Docker Compose configuration for Vaultwarden password manager with Nginx reverse proxy for secure access.

## Overview

A self-hosted password management solution with secure proxy routing:
- **Vaultwarden** - Unofficial Bitwarden-compatible password manager
- **Nginx** - Reverse proxy for secure routing and SSL termination

## Services Included

- **Vaultwarden** - Password vault backend
- **Nginx** - Reverse proxy and SSL termination
- **PostgreSQL** - Database backend (optional)

## Files

- `docker-compose.yml` - Service definitions
- `.env.example` - Environment variables template
- `nginx/` - Nginx configuration files

## Quick Start

1. Copy `.env.example` to `.env`
2. Configure domain and SSL certificates
3. Run: `docker-compose up -d`
4. Access Vaultwarden through Nginx proxy

## Security Features

- SSL/TLS encryption
- Reverse proxy isolation
- Environment-based secrets management

## Documentation

See the main [README](../../README.md) for general setup instructions.
