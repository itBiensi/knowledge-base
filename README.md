# Docmost Knowledge Base

Self-hosted Docmost deployment using Docker Compose with Caddy reverse proxy for automatic HTTPS. Docmost is a collaborative wiki and knowledge base software (similar to Notion or Confluence).

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Domain name pointing to your server (for production)
- Ports 80 and 443 available (for Caddy)

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/itBiensi/knowledge-base.git
   cd knowledge-base
   ```

2. **Configure environment variables**
   ```bash
   cp .env.example .env
   ```

   Edit `.env` and update:
   - `APP_SECRET` - Generate a random 32+ character string
   - `DB_PASSWORD` - Set a strong database password
   - SMTP settings (optional, for email notifications)

3. **Configure domain (Production)**

   Edit `Caddyfile` and replace `your-domain.com` with your actual domain:
   ```caddy
   your-domain.com {
       reverse_proxy docmost:3000
   }
   ```

   Update `.env`:
   ```bash
   APP_URL=https://your-domain.com
   ```

4. **Start the services**
   ```bash
   docker-compose up -d
   ```

5. **Access Docmost**

   - **Local**: http://localhost:8889
   - **Production**: https://your-domain.com

## Architecture

| Service | Image | Purpose |
|---------|-------|---------|
| `caddy` | `caddy:2-alpine` | Reverse proxy with auto HTTPS |
| `docmost` | `docmost/docmost:latest` | Main application |
| `db` | `postgres:16-alpine` | PostgreSQL database |
| `redis` | `redis:7-alpine` | Redis cache |

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `APP_URL` | Public URL | `http://localhost:8889` |
| `PORT` | Internal port | `3000` |
| `APP_SECRET` | Session encryption secret | *(required)* |
| `DATABASE_URL` | PostgreSQL connection | *(required)* |
| `REDIS_URL` | Redis connection | `redis://redis:6379` |
| `SMTP_HOST` | SMTP server | *(optional)* |
| `SMTP_PORT` | SMTP port | `587` |
| `SMTP_USERNAME` | SMTP username | *(optional)* |
| `SMTP_PASSWORD` | SMTP password | *(optional)* |

### Caddy Domain Configuration

Edit `Caddyfile` to configure your domain:

```caddy
# Production (with automatic HTTPS)
your-domain.com {
    reverse_proxy docmost:3000
}

# Or for local testing
localhost:8889 {
    reverse_proxy docmost:3000
}
```

Caddy automatically obtains and renews SSL certificates from Let's Encrypt.

### Ports

| Port | Service | Description |
|------|---------|-------------|
| `80` | Caddy | HTTP (redirects to HTTPS) |
| `443` | Caddy | HTTPS |
| `8889` | Docmost | Direct access (local only) |

## Common Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Restart services
docker-compose restart

# Check status
docker-compose ps
```

## Database Backup

### Backup
```bash
mkdir -p ./backups
docker-compose exec db pg_dump -U docmost docmost > ./backups/backup-$(date +%Y%m%d-%H%M%S).sql
```

### Restore
```bash
# Stop application
docker-compose stop docmost

# Drop and recreate database
docker-compose exec db dropdb -U docmost docmost
docker-compose exec db createdb -U docmost docmost

# Restore from backup
docker-compose exec -T db psql -U docmost docmost < ./backups/backup-YYYYMMDD-HHMMSS.sql

# Restart application
docker-compose start docmost
```

## Troubleshooting

### Email not sending
- Check SMTP credentials in `.env`
- For port 2525, set `SMTP_SECURE=false`
- For port 587, set `SMTP_SECURE=true`

### Container won't start
```bash
# Check logs
docker-compose logs docmost

# Recreate container
docker-compose up -d --force-recreate docmost
```

### Database connection error
- Verify `DATABASE_URL` and `DB_PASSWORD` match
- Ensure db container is healthy: `docker-compose ps`

## Data Persistence

Data is stored in Docker volumes:
- `caddy_data` - Caddy SSL certificates
- `caddy_config` - Caddy configuration
- `docmost_data` - File uploads
- `db_data` - Database
- `redis_data` - Cache

## Production Deployment

### DNS Configuration

Point your domain to your server:

| Type | Name | Value |
|------|------|-------|
| `A` | `@` | Your server IP |
| `A` | `knowledge` (optional) | Your server IP |

### Firewall Settings

Ensure these ports are open:
- **80/tcp** - HTTP (for SSL certificate issuance)
- **443/tcp** - HTTPS (secure access)

### Caddy SSL Certificates

Caddy automatically:
- Obtains SSL certificates from Let's Encrypt
- Renews certificates before expiration
- Stores certificates in `caddy_data` volume

## Security Notes

- Never commit `.env` to version control
- Use strong, unique passwords
- Change default `APP_SECRET` before production
- Keep Docker images updated

## Links

- [Docmost Documentation](https://docmost.com/docs/)
- [Docmost GitHub](https://github.com/docmost/docmost)
- [Local Access](http://localhost:8889)

## License

This deployment configuration is provided as-is. Docmost has its own licensing terms.
