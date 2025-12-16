# Configuration

This guide covers configuring the GreenThumb system.

## Environment Variables

All configuration is done through environment variables in the `.env` file.

### Required Variables

```env
# Database
DB_PASSWORD=your_secure_password

# Docker Hub (for CI/CD)
DOCKERHUB_USERNAME=your_username

# GitHub PAT (for greenthumb-core access)
GH_PAT=ghp_xxxxxxxxxxxxx
```

### Optional Variables (Future)

```env
# Supabase Cloud Database
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=your_supabase_key

# Cloudflare R2 Storage
CLOUDFLARE_R2_ENDPOINT=https://xxxxx.r2.cloudflarestorage.com
CLOUDFLARE_R2_ACCESS_KEY=your_access_key
CLOUDFLARE_R2_SECRET_KEY=your_secret_key
```

## Data Collection Intervals

Data collection intervals are configured in `data_collection/main.py`:

```python
SENSOR_INTERVAL = 30 * 60      # Sensor readings: every 30 minutes
PHOTO_INTERVAL = 4 * 60 * 60   # Photo capture: every 4 hours
CHECK_INTERVAL = 60            # Check loop: every minute
```

To change intervals, modify these values and rebuild the container:

```bash
make rebuild
```

## Database Configuration

The database schema is automatically initialized on first run from:

- `db/01_schema.sql` - Table definitions
- `db/02_seed.sql` - Initial data (sensors, variables, etc.)

### Accessing the Database

```bash
make db-shell
```

## Docker Compose Profiles

### Default Services

```bash
make up  # Starts: db, api, data_collection, watchtower
```

### With Cron Jobs

```bash
docker compose --profile cron up -d
```

## Next Steps

- [Quick Start](quick-start.md) - Start using the system
