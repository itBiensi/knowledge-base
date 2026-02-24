# Docmost Deployment - Project Context

## Project Overview

This is a **Docmost** self-hosted deployment using Docker Compose. Docmost is a collaborative wiki and knowledge base software (similar to Notion or Confluence) that enables teams to create, organize, and share documentation.

### Architecture

The deployment consists of three services:

| Service | Image | Purpose |
|---------|-------|---------|
| `docmost` | `docmost/docmost:latest` | Main application (Node.js-based) |
| `db` | `postgres:16-alpine` | PostgreSQL database for persistent storage |
| `redis` | `redis:7-alpine` | Redis cache for session management and caching |

### Network Configuration

- **External Port**: `8889` (mapped from internal port `3000`)
- **Access URL**: `http://localhost:8889`

## Building and Running

### Start the Services

```bash
docker-compose up -d
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f docmost
docker-compose logs -f db
docker-compose logs -f redis
```

### Stop the Services

```bash
docker-compose down
```

### Restart Services

```bash
docker-compose restart
```

### Check Service Status

```bash
docker-compose ps
```

## Data Persistence

Data is stored in Docker volumes:

| Volume | Purpose |
|--------|---------|
| `docmost_data` | Application file uploads and storage |
| `db_data` | PostgreSQL database files |
| `redis_data` | Redis append-only file (AOF) |

## Configuration (.env)

### Key Configuration Variables

| Variable | Description |
|----------|-------------|
| `APP_URL` | Public URL for the application |
| `APP_SECRET` | 32+ character secret for session encryption |
| `DATABASE_URL` | PostgreSQL connection string |
| `DB_PASSWORD` | Database password (must match in both `DATABASE_URL` and `db` service) |
| `REDIS_URL` | Redis connection URL |
| `STORAGE_DRIVER` | File storage driver (`local` for local filesystem) |
| `FILE_UPLOAD_SIZE_LIMIT` | Maximum file upload size (default: `50mb`) |

### Email Configuration (SMTP)

Currently configured with MailerSend for outgoing emails (password resets, notifications):

- **Host**: `smtp.mailersend.net`
- **Port**: `587` (TLS)
- **Secure**: `true`

## Development Conventions

### Security Best Practices

1. **Secrets**: The `.env` file contains sensitive credentials. Never commit this file to version control.
2. **APP_SECRET**: Should be a cryptographically random string of at least 32 characters.
3. **Database Password**: Use strong, unique passwords for production deployments.
4. **SMTP Credentials**: Keep MailerSend credentials secure.

### Maintenance Commands

```bash
# Backup database
docker-compose exec db pg_dump -U docmost docmost > backup.sql

# Access database shell
docker-compose exec db psql -U docmost -d docmost

# Access Redis CLI
docker-compose exec redis redis-cli

# Access application shell
docker-compose exec docmost sh
```

### Troubleshooting

| Issue | Solution |
|-------|----------|
| App won't start | Check `docker-compose logs docmost` for errors |
| Database connection failed | Verify `DATABASE_URL` and `DB_PASSWORD` match |
| Port conflict | Change the port mapping in `docker-compose.yaml` (e.g., `"8890:3000"`) |
| Data lost after restart | Ensure volumes are properly mounted and not removed |

### Updating

```bash
# Pull latest image
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

## Database Backup and Restore

### Backup Database (Step-by-Step)

**Step 1: Create a backups directory (one-time setup)**
```bash
mkdir -p ./backups
```

**Step 2: Run the backup command**
```bash
docker-compose exec db pg_dump -U docmost docmost > ./backups/backup-$(date +%Y%m%d-%H%M%S).sql
```

**Step 3: Verify the backup was created**
```bash
ls -lh ./backups/
```

**Step 4: (Optional) Compress the backup**
```bash
gzip ./backups/backup-*.sql
```

### Restore Database (Step-by-Step)

**Step 1: Stop the application (to prevent write operations)**
```bash
docker-compose stop docmost
```

**Step 2: Identify the backup file to restore**
```bash
ls -lh ./backups/
```

**Step 3: Drop and recreate the database**
```bash
# Drop existing database
docker-compose exec db dropdb -U docmost docmost

# Recreate database
docker-compose exec db createdb -U docmost docmost
```

**Step 4: Restore from backup**
```bash
# For uncompressed backup
docker-compose exec -T db psql -U docmost docmost < ./backups/backup-YYYYMMDD-HHMMSS.sql

# For compressed backup (.gz)
gunzip -c ./backups/backup-YYYYMMDD-HHMMSS.sql.gz | docker-compose exec -T db psql -U docmost docmost
```

**Step 5: Restart the application**
```bash
docker-compose start docmost
```

**Step 6: Verify the restore**
```bash
# Check database tables
docker-compose exec db psql -U docmost docmost -c "\dt"

# Check row counts
docker-compose exec db psql -U docmost docmost -c "SELECT COUNT(*) FROM users;"
```

### Automated Backups (Optional)

Create a cron job for daily backups at 2 AM:

```bash
# Edit crontab
crontab -e

# Add this line (adjust path as needed)
0 2 * * * cd /Users/mac158/Documents/docmost && docker-compose exec db pg_dump -U docmost docmost > ./backups/backup-$(date +\%Y\%m\%d-\%H\%M\%S).sql
```

### Backup Best Practices

| Practice | Description |
|----------|-------------|
| **Frequency** | Daily backups recommended for active instances |
| **Retention** | Keep at least 7-14 days of backups |
| **Off-site** | Copy backups to cloud storage (S3, Google Drive, etc.) |
| **Test restores** | Periodically test restore procedure to verify backup integrity |
| **Pre-update** | Always backup before running updates |

## File Structure

```
docmost/
├── docker-compose.yaml    # Docker Compose configuration
├── .env                   # Environment variables (secrets)
├── QWEN.md                # This context file
└── docker-volumes/        # (Implicit) Docker-managed volumes
```

## Useful Links

- **Docmost Documentation**: https://docmost.com/docs/
- **GitHub Repository**: https://github.com/docmost/docmost
- **Local Access**: http://localhost:8889
