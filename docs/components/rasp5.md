# Raspberry Pi 5 (rasp5)

The `rasp5/` directory contains everything that runs on each greenhouse node: Docker Compose services, the microcontroller API, the local React dashboard, hardware drivers, and shared Python packages.

## Repository Structure

```
rasp5/
├── greenthumb-models/          # Shared SQLModel + Pydantic package (Pi + Cloud)
├── greenthumb-rpi5/            # Pi hardware drivers package
├── greenthumb-core/            # Compatibility shim (re-exports above two)
├── microcontroller-api/        # FastAPI service — hardware control + REST API
│   ├── app.py                  # FastAPI application
│   ├── mock_app.py             # Mock app for testing without hardware
│   ├── routes/
│   │   ├── state_routes.py     # /state/ — Sense-Think-Act state endpoint
│   │   ├── data_routes.py      # /data/ — measurement history
│   │   ├── camera_routes.py    # /camera/ — photo capture
│   │   ├── video_routes.py     # /video/ — MJPEG stream
│   │   └── settings_routes.py  # /settings/ — dashboard settings + cultivation API
│   └── utils/
│       ├── lifespan.py         # Startup/shutdown + background task launch
│       ├── config_builder.py   # Builds DeviceConfig from local DB
│       ├── config_sync.py      # Applies cloud config to local DB
│       ├── db.py               # SQLAlchemy engine helpers
│       ├── db_utils.py         # Generic CRUD helpers
│       └── sync/
│           ├── client.py       # HTTP client for Pi→Cloud communication (SyncClient)
│           └── tasks.py        # sensor_persist, photo_capture, cloud_sync tasks
├── local-dashboard/            # React + Vite + Tailwind SPA for local display
│   ├── src/
│   │   ├── api/piApi.js        # TanStack Query hooks for all Pi API routes
│   │   └── pages/              # Dashboard, Graphs, Thresholds, Settings, Cultivation
│   └── vite.config.js          # Dev proxy → localhost:8080
├── microcontroller-api-client/ # Standalone controller (Sense-Think-Act loop)
├── db/
│   ├── 01_schema.sql           # Fallback DDL (tables also created by SQLAlchemy)
│   └── 02_seed.sql             # Seed data: units, variables, sensor/actuator models
├── resources/
│   └── implementation_plan.md  # Phase planning doc
├── compose.yaml                # Docker Compose for Pi deployment
├── Makefile
└── .env.example
```

## Services

### Database (`db`)

PostgreSQL 17 with automatic schema initialization.

- **Port**: 5432 (container-internal only)
- **Data**: Persisted in `postgres_data` named volume
- **Init**: `db/01_schema.sql` + `db/02_seed.sql` on first run

### API (`api`)

FastAPI microcontroller API — the central hub for all hardware and data access.

- **Port**: 8080
- **Docs**: `http://<pi-ip>:8080/docs`
- **Responsibilities**:
  - Read sensors in real time (via `DeviceManager`)
  - Command actuators with per-actuator locking
  - Serve MJPEG live video stream
  - Run three asyncio background tasks (sensor persist, photo capture, cloud sync)

#### Background Tasks

Three coroutines are started with `asyncio.create_task()` in the FastAPI lifespan and cancelled cleanly on shutdown:

| Task | Interval env var | Default | What it does |
|------|-----------------|---------|--------------|
| `sensor_persist_task` | `SENSOR_INTERVAL` | 300 s | Calls `dm.save_data()`, writes `Measurement` rows |
| `photo_capture_task` | `PHOTO_INTERVAL` | 14400 s (4 h) | Captures JPEG via camera actuator, creates `Photo` row, attempts immediate cloud upload |
| `cloud_sync_task` | `SYNC_INTERVAL` | 86400 s (24 h) | Pushes unsynced measurements + photos, pulls latest config, updates `last_seen_at` |

#### Startup Sequence (`utils/lifespan.py`)

1. Create DB engine + tables (`create_db_and_tables()`)
2. Create `SyncClient` (reads `CLOUD_API_URL`, `DEVICE_TOKEN` from env)
3. Try `sync_client.pull_config()` → if successful, call `apply_cloud_config_to_local_db()` to update local DB, record `last_config_sync` in `sync_metadata`
4. `build_config_from_db()` — assemble `DeviceConfig` DTO from (updated) local DB
5. `DeviceManager.init_from_config(config)` — instantiates all sensor/actuator drivers
6. Launch three background tasks (`sensor_persist`, `photo_capture`, `cloud_sync`)
7. **On shutdown**: cancel all tasks, `device_manager.shutdown()`, `engine.dispose()`

### Controller (`controller`)

Implements the Sense-Think-Act loop, calling the API every `CONTROL_INTERVAL` seconds (default 15 s):

1. `GET /state/` — fetch sensor readings + active thresholds + variable names
2. Evaluate each threshold; determine actuator commands
3. `POST /state/actuators/{id}/command` for each action
4. `POST /state/heartbeat` — reset safety-mode timer

If the heartbeat is missed for > 2× `CONTROL_INTERVAL`, the API enters **safety mode** (all actuators commanded to safe state).

### Local Dashboard (`local-dashboard`)

React SPA served by nginx on port 80, proxying API calls to `api:8080`.

| Page | Route | Description |
|------|-------|-------------|
| Dashboard | `/` | Live sensor cards (real variable names + units), sparklines, MJPEG camera feed, safety mode badge, active phase indicator |
| Graphs | `/graphs` | Historical charts (recharts), time range selector, variable filter using real names |
| Thresholds | `/thresholds` | Inline threshold editing, `is_dirty` pending badges |
| Settings | `/settings` | Device mode, sync status, manual sync trigger |
| Cultivation | `/cultivation` | Active cultivation view with phase timeline and advance-phase form; Begin cultivation form (species picker) and End cultivation button when no active cultivation |

### Watchtower

Polls Docker Hub every **1 hour** (`WATCHTOWER_POLL_INTERVAL: 3600`), pulls updated images, and restarts affected containers.

## Environment Variables

All variables come from `rasp5/.env` (copy from `.env.example`).

### Required

| Variable | Description |
|----------|-------------|
| `DEVICE_ID` | Device ID registered in the cloud admin dashboard |
| `DEVICE_TOKEN` | Bearer token for cloud sync — shown once at device creation |
| `DB_PASSWORD` | PostgreSQL password |

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_USER` | `greenthumb` | PostgreSQL user |
| `DB_NAME` | `greenthumb` | Database name |
| `DATABASE_URL` | auto-built | Full SQLAlchemy URL (overrides individual DB_* vars) |

### Cloud Sync

| Variable | Default | Description |
|----------|---------|-------------|
| `CLOUD_API_URL` | `https://api.greenthumb.io` | Cloud gateway base URL |
| `DEVICE_TOKEN` | — | Pi bearer token (also in Required above) |

### Background Task Intervals

| Variable | Default | Description |
|----------|---------|-------------|
| `SENSOR_INTERVAL` | 300 | Seconds between sensor persist cycles |
| `PHOTO_INTERVAL` | 14400 | Seconds between photo captures (4 h) |
| `SYNC_INTERVAL` | 86400 | Seconds between cloud sync cycles (24 h) |
| `PERSIST_INTERVAL` | 300 | Kept for backwards compatibility — use `SENSOR_INTERVAL` |

### Camera

| Variable | Default | Description |
|----------|---------|-------------|
| `CAMERA_SRC` | `/dev/video0` | Camera device path |
| `CAMERA_WIDTH` | 1280 | Capture width in pixels |
| `CAMERA_HEIGHT` | 720 | Capture height in pixels |
| `PHOTOS_DIR` | `/data/photos` | Local directory for captured photos |

### Controller

| Variable | Default | Description |
|----------|---------|-------------|
| `API_BASE_URL` | `http://api:8080` | microcontroller-api base URL (controller reads this) |
| `CONTROL_INTERVAL` | 15 | Sense-Think-Act loop cadence in seconds |
| `HYSTERESIS_PCT` | 0.1 | Threshold dead-band fraction (10%) |

### CI/CD

| Variable | Description |
|----------|-------------|
| `DOCKERHUB_USERNAME` | Docker Hub username for image pushes |

## Makefile Commands

| Command | Description |
|---------|-------------|
| `make up` | Start all services (`docker compose up -d`) |
| `make down` | Stop all services |
| `make logs` | Follow logs from all services |
| `make logs-api` | Follow API logs |
| `make update` | `git pull` + `docker compose pull` + `make up` |
| `make status` | Container health + last sync times |
| `make backup` | Dump local PostgreSQL to `/data/backups/` |
| `make db-shell` | Open psql shell |
| `make clean` | Remove all containers + volumes (destructive!) |

## Hardware Access

The `api` container has direct device access via Docker bind-mounts and `privileged: true`:

| Device | Path | Purpose |
|--------|------|---------|
| Camera | `/dev/video0` | USB camera (MJPEG stream + photo capture) |
| I2C bus | `/dev/i2c-1` | AHT10, BMP280, TSL2561 sensors |
| GPIO | `/dev/gpiochip0` | RGB LED, Water Pump actuators |

## CI/CD

On push to `main`, GitHub Actions builds ARM64 Docker images for `api` and `controller`, pushes them to Docker Hub. Watchtower on the Pi picks up the new images within 1 hour and restarts the affected containers.
