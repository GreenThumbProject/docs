# Configuration

On the Pi, configuration lives in `rasp5/deploy/.env` (the repository root keeps a symlink to it). Timing and control settings live in the database (below). Cloud configuration lives in the cloud repository's `.env`.

## Pi configuration (`rasp5/deploy/.env`)

### Required

| Variable | Description |
|----------|-------------|
| `DEVICE_ID` | Device ID registered in the cloud admin dashboard |
| `DEVICE_TOKEN` | Bearer token for cloud sync, generated with **Rotate token** on the device page of the cloud admin dashboard |
| `CLOUD_API_URL` | Sync endpoint the node pushes to over the VPN |

!!! note
    If `CLOUD_API_URL` or `DEVICE_TOKEN` is empty, the node keeps measuring and controlling, but nothing syncs and the logs do not say so.

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_USER` | `greenthumb` | PostgreSQL user |
| `DB_PASSWORD` | `greenthumb` (change it) | PostgreSQL password |
| `DB_NAME` | `greenthumb` | Database name |
| `DATABASE_URL` | — | Built by compose from the `DB_*` values; a value in `.env` is ignored |

### Camera and photos

| Variable | Default | Description |
|----------|---------|-------------|
| `PHOTOS_DIR` | `/data/photos` | Local directory for captured photos |

Camera source, resolution and frame rate are set per device in the database.

### Other

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_LEVEL` | `INFO` | api and controller log level |
| `RESTORE_ON_BOOT` | `true` | Re-apply schedule-driven loads at startup |
| `IMAGE_TAG` | `prod` | Pin a release by its short commit SHA |

### Timing and control (stored on the device, not in .env)

| Field | Default | Description |
|-------|---------|-------------|
| `sensor_interval_s` | 300 | Seconds between sensor persist cycles |
| `photo_interval_s` | 14400 | Seconds between scheduled photos (4 h) |
| `sync_interval_s` | 3600 | Seconds between cloud sync cycles |
| `control_interval_s` | 15 | Seconds between controller loops |
| `hysteresis_pct` | 0.10 | Threshold hysteresis band (10%) |
| `heartbeat_timeout_s` | 60 | Seconds without a controller heartbeat before safety mode |

Change them in the cloud admin dashboard (device settings) or with `PATCH /settings/runtime` on the node. No restart is needed.

---

## Control rules

Each cultivation has control rules, tied to a growth phase:

- **THRESHOLD**: keep a variable in a range.
- **SCHEDULE**: on at a set time for N hours.
- **INTERVAL**: run every N minutes.
- **AFTER_ACTUATOR**: run after another load.

They are copied from the species' rule templates when a cultivation begins. Edit them on the node's local dashboard (Cultivation) or in the cloud admin dashboard (Cultivations); edits made on the node are pushed to the cloud on the next sync.

---

## How the node gets its configuration

At startup the node pulls its configuration from the cloud (`GET /sync/devices/{id}/config`) and writes it into its local database, then starts from the local database. If the cloud is unreachable it starts from what the local database already holds; a freshly wiped node holds nothing, so set the device up in the cloud first.

Changes made on the node are marked dirty and pushed back with `POST /sync/devices/{id}/state`.

---

## Rotating the device token

1. Cloud admin dashboard → device page → **Rotate token** (the old token stops working at once).
2. On the Pi, set `DEVICE_TOKEN` in `deploy/.env`.
3. Recreate the `api` container. On the Pi (Linux), from `deploy/`:

    ```bash
    docker compose up -d api
    ```

!!! warning
    A plain restart does not re-read `.env`. Recreating `api` releases the GPIO lines, so do it with the rig in view.

---

## Cloud configuration

Set in the cloud repository's `.env`. Values are not listed here.

| Variable | Description |
|----------|-------------|
| `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` | Postgres connection parts; compose builds both connection strings from them |
| `JWT_SECRET` | Read by auth-service only |
| `INTERNAL_SERVICE_TOKEN` | Shared by auth-service and account-service; must match |
| `DEV_AUTH_BYPASS` | Local development only |
| `R2_*` | Photo storage; unset = photos kept without a cloud URL |
| `CORS_ORIGINS` | Allowed browser origins (production) |
