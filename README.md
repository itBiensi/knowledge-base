# Docmost Knowledge Base

Self-hosted Docmost deployment using Docker Compose. Docmost is a collaborative wiki and knowledge base software (similar to Notion or Confluence).

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Port 8889 available

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

3. **Start the services**
   ```bash
   docker-compose up -d
   ```

4. **Access Docmost**
   
   Open your browser and navigate to: **http://localhost:8889**

## Architecture

| Service | Image | Purpose |
|---------|-------|---------|
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

### Ports

- **External**: `8889`
- **Internal**: `3000`

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
- `docmost_data` - File uploads
- `db_data` - Database
- `redis_data` - Cache

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
