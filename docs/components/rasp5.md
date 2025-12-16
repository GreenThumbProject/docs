# Raspberry Pi 5

The `rasp5` repository contains all code for deploying GreenThumb on a Raspberry Pi 5.

## Overview

This is a Docker Compose deployment consisting of:

- **PostgreSQL** - Local database
- **FastAPI** - REST API and dashboard
- **Data Collection** - Sensor readings and photos
- **Watchtower** - Automatic updates

## Repository Structure

```
rasp5/
├── .github/workflows/      # CI/CD pipelines
├── data_collection/        # Sensor collection service
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── db/                     # Database schemas
│   ├── 01_schema.sql
│   └── 02_seed.sql
├── fastapi/                # API service
│   ├── Dockerfile
│   ├── app.py
│   ├── static/
│   └── requirements.txt
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

FastAPI server with:

- REST endpoints for data access
- Live video streaming
- Static file serving

- **Port**: 8080
- **Docs**: `/docs` (Swagger), `/redoc`

### Data Collection (`data_collection`)

Background service that:

- Reads sensors every 30 minutes
- Captures photos every 4 hours
- Stores data in PostgreSQL
- Writes to log file

### Watchtower

Automatically pulls and restarts updated containers from Docker Hub.

- **Poll interval**: 5 minutes
- **Cleanup**: Removes old images

## Make Commands

```bash
make up        # Start all services
make down      # Stop all services
make logs      # View logs
make rebuild   # Full rebuild
make db-shell  # PostgreSQL shell
make restart-api  # Restart specific service
```

## Hardware Access

The containers have direct access to:

| Device | Path | Purpose |
|--------|------|---------|
| Camera | `/dev/video0` | USB camera |
| I2C | `/dev/i2c-1` | Sensor bus |
| GPIO | `/dev/gpiochip*` | Future actuators |

## Dependencies

Both `api` and `data_collection` depend on `greenthumb-core`:

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
