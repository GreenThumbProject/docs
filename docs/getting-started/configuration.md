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

### Controller Variables

```env
# API connection
API_BASE_URL=http://api:8080

# Control loop timing
CONTROL_INTERVAL=15      # Seconds between control cycles
PERSIST_INTERVAL=300     # Seconds between data persistence
```

### Camera Variables

```env
CAMERA_SRC=/dev/video0
CAMERA_WIDTH=1280
CAMERA_HEIGHT=720
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

## Control Loop Configuration

The controller loop timing is configured via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `CONTROL_INTERVAL` | 15 | Seconds between Sense-Think-Act cycles |
| `PERSIST_INTERVAL` | 300 | Seconds between data persistence (future) |

To change intervals, update `.env` and restart:

```bash
make restart-controller
```

## Database Configuration

The database schema is automatically initialized on first run from:

- `db/01_schema.sql` - Table definitions
- `db/02_seed.sql` - Initial data (sensors, variables, etc.)

### Accessing the Database

```bash
make db-shell
```

## Threshold Configuration

Thresholds are configured per cultivation in the database:

```sql
INSERT INTO threshold (id_cultivation, id_variable, min_value, max_value, target_value, id_actuator_action)
VALUES
    (1, 1, 20.0, 30.0, 25.0, 1),  -- Temperature: 20-30°C, target 25°C
    (1, 2, 60.0, 80.0, 70.0, 2);  -- Humidity: 60-80%, target 70%
```

The controller evaluates these thresholds and triggers actuator actions.

## Docker Compose Profiles

### Default Services

```bash
make up  # Starts: db, api, controller, watchtower
```

### With Cron Jobs

```bash
docker compose --profile cron up -d
```

## Next Steps

- [Quick Start](quick-start.md) - Start using the system
