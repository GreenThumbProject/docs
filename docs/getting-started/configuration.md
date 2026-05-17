# Configuration

All Pi configuration is done through environment variables in `rasp5/.env`. Cloud configuration lives in `cloud/.env`.

## Pi Configuration (`rasp5/.env`)

### Required

| Variable | Description |
|----------|-------------|
| `DEVICE_ID` | Device ID registered in the cloud admin dashboard |
| `DEVICE_TOKEN` | Bearer token for cloud sync — generated via **Devices → Rotate Token** |
| `DB_PASSWORD` | PostgreSQL password (local Pi database) |

### Cloud Sync

| Variable | Default | Description |
|----------|---------|-------------|
| `CLOUD_API_URL` | — | Cloud gateway base URL, e.g. `https://api.greenthumb.io` |

If `CLOUD_API_URL` is not set, cloud sync and config pull are skipped and the Pi operates in fully offline mode.

### Background Task Intervals

| Variable | Default | Description |
|----------|---------|-------------|
| `SENSOR_INTERVAL` | 300 | Seconds between sensor persist cycles |
| `PHOTO_INTERVAL` | 14400 | Seconds between scheduled photo captures (4 h) |
| `SYNC_INTERVAL` | 86400 | Seconds between cloud sync cycles |
| `CONTROL_INTERVAL` | 15 | Seconds between Sense-Think-Act loops |
| `HYSTERESIS_PCT` | 0.10 | Threshold hysteresis band (10%) |

### Camera

| Variable | Default | Description |
|----------|---------|-------------|
| `CAMERA_SRC` | `/dev/video0` | Camera device path |
| `CAMERA_WIDTH` | 1280 | Capture width in pixels |
| `CAMERA_HEIGHT` | 720 | Capture height in pixels |
| `PHOTOS_DIR` | `/data/photos` | Local directory for captured photos |

### Database

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_USER` | `root` | PostgreSQL user |
| `DB_NAME` | `greenthumb` | Database name |
| `DATABASE_URL` | auto-built | Full SQLAlchemy URL (overrides individual DB_* vars) |

### Tailscale (optional)

| Variable | Description |
|----------|-------------|
| `TAILSCALE_AUTHKEY` | If set, `tailscale up --authkey` runs on first boot (GreenthumbOS only) |

---

## GreenthumbOS Boot Config (`/boot/greenthumb.env`)

When using the GreenthumbOS image, the user edits `/boot/greenthumb.env` on the FAT32 boot partition (readable from any OS without special tools). The `greenthumb-init.service` merges it over `.env.example` defaults on first boot.

```env
# Minimum required
DEVICE_ID=5
DEVICE_TOKEN=abc123xyz...

# WiFi (alternative to wpa_supplicant.conf)
WIFI_SSID=MyNetwork
WIFI_PASSWORD=secret

# Cloud API
CLOUD_API_URL=https://api.greenthumb.io

# Tailscale (optional — enables remote SSH)
TAILSCALE_AUTHKEY=tskey-auth-...

# Hardware overrides (optional)
CAMERA_SRC=/dev/video1
SENSOR_INTERVAL=600
```

### Token Rotation

When `DEVICE_TOKEN` needs to be changed:
1. Generate a new token: cloud admin dashboard → **Devices → Rotate Token**.
2. Edit `/boot/greenthumb.env` with the new token.
3. Run `make update` — the init merge logic propagates the token to `.env`.
4. Restart the API container: `docker compose restart api`.

---

## Threshold Configuration

Thresholds are configured per `cultivation_phase` in the database. You can edit them through:

1. **Local dashboard** (`http://<pi-ip>/thresholds`) — inline editing; changes are marked `is_dirty=True` and synced to cloud on next sync cycle.
2. **Cloud admin dashboard** (`http://<cloud-host>/cultivations`) — full CRUD via the `ThresholdPanel` component.
3. **Direct DB** (for initial seed):
   ```sql
   INSERT INTO threshold (id_cultivation_phase, id_variable, min_value, max_value, target_value)
   VALUES (1, 1, 20.0, 30.0, 25.0);  -- Temperature: 20-30°C
   ```

---

## Cloud Configuration (`cloud/.env`)

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_USER` | `postgres` | Cloud PostgreSQL user |
| `DB_PASSWORD` | `password` | Cloud PostgreSQL password |
| `DB_NAME` | `greenthumb` | Cloud database name |
| `JWT_SECRET` | `change-me-in-production` | JWT signing secret (auth-service) |
| `DEV_AUTH_BYPASS` | `false` | Set to `true` to skip JWT validation in local dev |

---

## Three-Tier Configuration Model

Device configuration flows through three levels, with higher tiers overriding lower ones:

```
Tier 0 — Compile-time defaults (code / .env.example)
   ↓ overridden by
Tier 1 — Local DB (device_sensor, device_actuator, threshold, cultivation tables)
   ↓ overridden by
Tier 2 — Cloud config pull (GET /sync/devices/{id}/config on startup)
```

On startup the API tries to pull the full `DeviceConfig` from the cloud. If the cloud is unreachable, it falls back to building `DeviceConfig` from the local DB. Either way, `DeviceManager.init_from_config(config)` receives the same schema.

The `SyncClient` also pushes **local changes back to the cloud** (`PATCH /sync/devices/{id}/local-changes`) so edits made offline through the local dashboard don't get overwritten on next config pull.

---

## Docker Compose Profiles

```bash
# Standard deployment
make up

# Development (rebuild from source)
docker compose up --build

# Cloud development with auth bypass
DEV_AUTH_BYPASS=true docker compose -f cloud/compose.yaml up --build
```
