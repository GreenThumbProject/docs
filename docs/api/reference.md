# API Reference

GreenThumb exposes two REST APIs: the **Pi API** (runs on each Raspberry Pi node) and the **Cloud API** (runs in the cloud, accessed via the Gateway).

---

## Pi API

**Base URL:** `http://<raspberry-pi-ip>:8080`

Interactive docs: `http://<pi-ip>:8080/docs` (Swagger), `/redoc`

### Endpoint Groups

| Prefix | Description |
|--------|-------------|
| `/state` | System state for the Sense-Think-Act loop |
| `/data` | Measurement history from local DB |
| `/video` | Live MJPEG camera stream |
| `/camera` | Photo capture |
| `/settings` | Dashboard settings and sync control |

---

### State Routes (`/state`)

#### `GET /state/`

Real-time system state used by the controller.

**Response:**

```json
{
  "sensors": {
    "1": [{"id_variable": 1, "value": 25.3}],
    "2": [{"id_variable": 2, "value": 65.1}]
  },
  "thresholds": [
    {
      "id_threshold": 1,
      "id_cultivation_phase": 2,
      "id_variable": 1,
      "min_value": 20.0,
      "max_value": 30.0,
      "target_value": 25.0
    }
  ],
  "safety_mode": false,
  "timestamp": "2026-04-13T10:00:00"
}
```

---

#### `POST /state/actuators/{id}/command`

Command an actuator (concurrent-safe, per-actuator lock).

**Path params:** `id` — `id_device_actuator`

**Body (RGB LED):**
```json
{"r": 255, "g": 128, "b": 0}
```

**Body (Water Pump):**
```json
{"duty_cycle": 100}
```

**Response:**
```json
{"status": "ok", "actuator_id": 1, "command": {"duty_cycle": 100}}
```

---

#### `POST /state/heartbeat`

Controller heartbeat — resets safety-mode timer.

**Response:**
```json
{"status": "ok", "safety_mode": false, "last_heartbeat": "2026-04-13T10:00:00"}
```

---

#### `GET /state/safety`

Current safety mode status.

**Response:**
```json
{"safety_mode": false, "last_heartbeat": "2026-04-13T10:00:00"}
```

---

### Data Routes (`/data`)

#### `GET /data/data`

Measurement history grouped by variable.

**Query params:** `device_id` (default: 1), `limit` (default: 10)

**Response:**
```json
{
  "device_id": 1,
  "data": {
    "Temperature": {
      "unit": "°C",
      "latest": 25.3,
      "latest_at": "2026-04-13T10:00:00",
      "history": [
        {"value": 25.3, "collected_at": "2026-04-13T10:00:00"},
        {"value": 25.1, "collected_at": "2026-04-13T09:55:00"}
      ]
    }
  }
}
```

---

#### `GET /data/latest`

Most recent value only for each variable.

**Query params:** `device_id` (default: 1)

---

### Video & Camera Routes

#### `GET /video/stream`

Live MJPEG stream.

- Content-Type: `multipart/x-mixed-replace; boundary=frame`
- HTML usage: `<img src="http://<pi-ip>:8080/video/stream">`

---

#### `POST /camera/capture`

Capture a photo immediately (outside the scheduled interval).

**Response:**
```json
{"status": "ok", "path": "/data/photos/20260413_100000_1_2.jpg"}
```

---

### Settings Routes (`/settings`)

#### `GET /settings/`

Device info, sync metadata, and pending item counts.

**Response:**
```json
{
  "device": {
    "id_device": 1,
    "name": "Greenhouse A",
    "mac_address": "aa:bb:cc:dd:ee:ff",
    "location": "Shed A",
    "device_mode": "MEDIUM",
    "is_dirty": false,
    "updated_at": "2026-05-14T10:00:00"
  },
  "safety_mode": false,
  "sync": {
    "last_config_sync": "2026-05-14T09:00:00",
    "last_sensor_persist": "2026-05-14T09:55:00",
    "last_data_push": "2026-05-14T09:55:00",
    "pending_measurements": 12,
    "pending_photos": 1
  }
}
```

---

#### `PATCH /settings/device-mode`

Update device operating mode (`LOW`, `MEDIUM`, `HIGH`).

**Body:**
```json
{"device_mode": "HIGH"}
```

---

#### `GET /settings/thresholds`

List all thresholds for the active cultivation phase.

**Response:** Array of threshold objects with `variable_name`, `phase_name`, `is_default_phase`, `is_dirty`.

---

#### `PATCH /settings/thresholds/{id}`

Update a threshold value. Sets `is_dirty=True` and updates `updated_at` for cloud sync.

**Body:**
```json
{"min_value": 18.0, "max_value": 28.0, "target_value": 23.0}
```

---

#### `GET /settings/cultivation`

Current cultivation with full phase history.

**Response:** Cultivation object with `phases` array, each with `is_current` flag.

---

#### `POST /settings/cultivation/advance-phase`

Close the current `CultivationPhase` (sets `ended_at`) and open the next growth phase.

**Body:**
```json
{"next_growth_phase_id": 3}
```

---

#### `POST /settings/thresholds`

Create a new threshold for the active cultivation.

**Body:**
```json
{"id_variable": 1, "id_growth_phase": 2, "min_value": 20.0, "max_value": 30.0, "target_value": 25.0}
```

**Response:**
```json
{"id_threshold": 7}
```

---

#### `GET /settings/growth-phases`

Return growth phases for the active cultivation's species, ordered by `phase_order`.

---

#### `GET /settings/variables`

Return all variables available for threshold creation.

---

#### `GET /settings/plant-species`

Return all plant species available for starting a cultivation.

---

#### `GET /settings/cultivation/template-sources`

Return distinct threshold template sources available for a species.

**Query params:** `id_plant_species` (int)

**Response:**
```json
{"sources": ["manual", "ml"]}
```

---

#### `POST /settings/cultivation/begin`

Start a new cultivation. Fails with 409 if one is already active. Auto-copies `SpeciesThreshold` templates as `Threshold` rows (`is_dirty=True`). Template source priority: `template_source` param → `ml` → `manual`.

**Body:**
```json
{"id_plant_species": 3, "notes": "Spring run", "template_source": "ml"}
```

**Response:**
```json
{
  "id_cultivation": 5,
  "id_plant_species": 3,
  "species_name": "Tomato",
  "id_cultivation_phase": 9,
  "id_growth_phase": 2,
  "started_at": "2026-05-14T08:00:00",
  "thresholds_copied": 8,
  "template_source_used": "ml"
}
```

---

#### `POST /settings/cultivation/end`

End the current active cultivation. Closes the open `CultivationPhase` and sets `end_date`.

**Response:**
```json
{"id_cultivation": 5, "ended_at": "2026-05-14T18:00:00"}
```

---

#### `POST /settings/sync`

Trigger an immediate cloud sync (fire-and-forget asyncio task). Returns immediately — the sync runs asynchronously in the background. Check `GET /settings/` for updated sync timestamps after a few seconds.

**Response:**
```json
{"status": "accepted", "message": "Sync triggered — check /settings/ for result"}
```

---

## Cloud API

The cloud API is accessed through the **Gateway** (port 80), which routes requests to the appropriate backend service.

**Base URL:** `http://<cloud-host>:80`  (or HTTPS in production)

Interactive docs: `http://<cloud-host>:8000/docs` (greenthumb-api direct access)

### Gateway Routing

| Path prefix | Backend service | Auth required |
|-------------|-----------------|---------------|
| `/auth/**` | auth-service :8081 | None (public login) |
| `/accounts/**` | account-service :8082 | JWT (user) |
| `/admin/**` | greenthumb-api :8000 | JWT (user) |
| `/sync/**` | greenthumb-api :8000 | Device token (Bearer) |

---

### Auth Routes (`/auth`)

#### `POST /auth/login`

Returns a JWT for a registered user.

**Body:**
```json
{"email": "admin@example.com", "password": "secret"}
```

**Response:**
```json
{"token": "eyJ...", "expires_in": 86400}
```

---

#### `GET /auth/validate`

Validate a JWT (called internally by greenthumb-api, not typically called directly).

**Headers:** `Authorization: Bearer <jwt>`

**Response:**
```json
{"id_user": "550e8400-e29b-41d4-a716-446655440000"}
```

---

### Admin Routes (`/admin`)

All `/admin/**` routes require `Authorization: Bearer <jwt>` from a logged-in user.

#### Device Management

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/admin/devices/` | List all devices |
| `POST` | `/admin/devices/` | Register a new device |
| `GET` | `/admin/devices/{id}` | Get device details |
| `PUT` | `/admin/devices/{id}` | Update device |
| `DELETE` | `/admin/devices/{id}` | Delete device |
| `POST` | `/admin/devices/{id}/token` | Rotate `device_token` — returns new token once |

#### Fleet View

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/admin/fleet/` | All devices with online/stale/offline status based on `updated_at` |

#### Cultivation & Species

Full CRUD is available for all entities in the cultivation tree:

| Resource | Prefix |
|----------|--------|
| Plant species | `/admin/plant-species` |
| Growth phases | `/admin/growth-phases` |
| Cultivations | `/admin/cultivations` |
| Cultivation phases | `/admin/cultivation-phases` |
| Thresholds | `/admin/thresholds` |

#### Hardware Catalog (read-only in admin dashboard)

| Resource | Prefix |
|----------|--------|
| Sensor models | `/admin/sensor-models` |
| Actuator models | `/admin/actuator-models` |
| Variables | `/admin/variables` |
| Units | `/admin/units` |

#### Device Configuration Catalog

Full CRUD for device-level hardware and threshold configuration:

| Resource | Prefix |
|----------|--------|
| Device sensors | `/admin/device-sensors` |
| Sensor capabilities | `/admin/sensor-capabilities` |
| Device actuators | `/admin/device-actuators` |
| Species thresholds | `/admin/species-thresholds` |
| Actuator logs | `/admin/actuator-logs` |

#### Data & Media

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/admin/measurements/` | Query measurements (filter by device, variable, limit) |
| `GET` | `/admin/photos/` | List photo metadata (filter by device, cultivation) |

---

### Sync Routes (`/sync`)

All `/sync/**` routes require `Authorization: Bearer <device_token>` where the token matches the requesting device's `device.device_token`.

#### `GET /sync/devices/{id}/config`

Pull the full `FullSyncPayload` for the device from the cloud database.

**Response:** `FullSyncPayload` JSON (same schema as built by `config_builder.py`):

```json
{
  "device": {"id_device": 1, "name": "Greenhouse A"},
  "sensors": [...],
  "actuators": [...],
  "active_phase": {
    "id_cultivation_phase": 2,
    "id_cultivation": 1,
    "thresholds": [...]
  }
}
```

---

#### `POST /sync/devices/{id}/measurements`

Bulk-insert measurements from the Pi. Skips malformed rows.

**Body:**
```json
[
  {"id_device_sensor": 1, "id_variable": 1, "value": 25.3, "collected_at": "2026-04-13T10:00:00"},
  {"id_device_sensor": 2, "id_variable": 2, "value": 65.1, "collected_at": "2026-04-13T10:00:00"}
]
```

**Response:**
```json
{"inserted": 2}
```

---

#### `POST /sync/devices/{id}/photos`

Receive a JPEG photo from the Pi, upload it to Supabase Storage, and store the metadata row. The Pi never holds Supabase credentials — the cloud API handles the upload.

**Content-Type:** `multipart/form-data`

**Form fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | binary (JPEG) | Yes | Raw photo bytes |
| `id_device_actuator` | int | Yes | Camera actuator ID |
| `captured_at` | string (ISO 8601) | Yes | Capture timestamp |
| `id_cultivation` | int | No | Active cultivation ID |
| `file_size_bytes` | int | No | File size hint |

**Response:**
```json
{"id_photo": 42, "cloud_url": "https://xyz.supabase.co/storage/v1/object/public/plant-photos/1/...", "status": "stored"}
```

`cloud_url` is `null` if Supabase credentials are not configured on the cloud side.

---

#### `PATCH /sync/devices/{id}/local-changes`

Push local threshold edits to the cloud. Uses **last-write-wins** via `updated_at` comparison.

**Body:**
```json
{
  "thresholds": [
    {"id_threshold": 1, "min_value": 18.0, "max_value": 28.0, "target_value": 23.0, "updated_at": "2026-04-13T09:50:00"}
  ]
}
```

---

## Error Responses

| Status | Body | When |
|--------|------|------|
| 401 | `{"detail": "Invalid or missing token"}` | Bad/missing auth on any protected route |
| 403 | `{"detail": "Device ID mismatch"}` | Device token used for wrong device ID |
| 404 | `{"detail": "Not found"}` | Resource doesn't exist |
| 422 | Validation error detail | Malformed request body |
| 500 | `{"detail": "Internal server error"}` | Unexpected server error |
