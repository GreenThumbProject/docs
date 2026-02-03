# Raspberry Pi 5

The `rasp5` repository contains all code for deploying GreenThumb on a Raspberry Pi 5.

## Overview

This is a Docker Compose deployment consisting of:

- **PostgreSQL** - Local database
- **Microcontroller API** - REST API and hardware control
- **Controller** - Sense-Think-Act loop client
- **Watchtower** - Automatic updates

## Repository Structure

```
rasp5/
├── .github/workflows/      # CI/CD pipelines
├── microcontroller-api/    # API service (submodule)
│   ├── Dockerfile
│   ├── app.py
│   ├── routes/
│   └── requirements.txt
├── microcontroller-api-client/  # Controller scripts (submodule)
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── greenthumb-core/        # Shared library (submodule)
├── db/                     # Database schemas
│   ├── 01_schema.sql
│   └── 02_seed.sql
├── cron/                   # Scheduled tasks
├── compose.yaml            # Docker Compose config
├── Makefile                # Developer commands
├── .env.example            # Environment template
└── README.md
```

## Services

### Database (`db`)

PostgreSQL 17.6 with automatic schema initialization.

- **Port**: 5432 (internal only)
- **Data**: Persisted in `postgres_data` volume
- **Init scripts**: `db/01_schema.sql`, `db/02_seed.sql`

### API (`api`)

Microcontroller API server with centralized device management:

- REST endpoints for data access
- Hardware control (sensors and actuators)
- Live video streaming
- Static file serving
- Per-actuator locking for concurrent access

- **Port**: 8080
- **Docs**: `/docs` (Swagger), `/redoc`

### Controller (`controller`)

HTTP-based client implementing Sense-Think-Act loop:

- Fetches system state from API
- Evaluates thresholds
- Commands actuators
- Sends heartbeat to prevent safety mode

- **Control Interval**: 15 seconds (configurable)

### Watchtower

Automatically pulls and restarts updated containers from Docker Hub.

- **Poll interval**: 5 minutes
- **Cleanup**: Removes old images

## Makefile Commands

### Production (Pull Pre-built Images)

```bash
make up          # Start all services
make down        # Stop all services
make pull        # Pull latest images from Docker Hub
```

### Local Build (For Development)

```bash
make dev         # Quick build: api + controller
make rebuild     # Rebuild api + controller (uses cache)
make rebuild-all # Full rebuild with --no-cache
```

### Submodule Management

```bash
make update-greenthumb-core    # Update greenthumb-core submodule
make update-microcontroller-api # Update API submodule
make update-all                # Update all submodules
```

### Utilities

```bash
make logs        # Follow logs from all services
make logs-api    # Follow API logs only
make logs-ctrl   # Follow controller logs only
make db-shell    # Open psql shell to database
make clean       # Remove all containers, volumes, images (DANGER!)
make restart-%   # Restart specific service (e.g., make restart-api)
```

## Hardware Access

The containers have direct access to:

| Device | Path | Purpose |
|--------|------|---------|
| Camera | `/dev/video0` | USB camera |
| I2C | `/dev/i2c-1` | Sensor bus |
| GPIO | `/dev/gpiochip*` | Actuator control |

## Environment Variables

### Required

| Variable | Description |
|----------|-------------|
| `DB_PASSWORD` | PostgreSQL password |
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `GH_PAT` | GitHub PAT for private repos |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `DEVICE_ID` | 1 | Device ID in database |
| `CONTROL_INTERVAL` | 15 | Controller loop interval (seconds) |
| `PERSIST_INTERVAL` | 300 | Data persistence interval (seconds) |
| `CAMERA_SRC` | /dev/video0 | Camera device path |
| `CAMERA_WIDTH` | 1280 | Camera resolution width |
| `CAMERA_HEIGHT` | 720 | Camera resolution height |

## Dependencies

Both `api` and `controller` depend on `greenthumb-core`:

```txt
# requirements.txt
git+https://${GH_PAT}@github.com/GreenThumbProject/greenthumb-core.git
```

## CI/CD

On push to `main`:

1. GitHub Actions builds ARM64 Docker images
2. Pushes to Docker Hub
3. Watchtower pulls new images on Raspberry Pi
4. Containers restart with new code
