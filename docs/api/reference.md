# API Reference

GreenThumb exposes two REST APIs, both built with FastAPI: the **Pi API** (runs on each Raspberry Pi node) and the **Cloud API** (runs in the cloud, behind the gateway).

Each API's Swagger UI (`/docs`) and OpenAPI schema (`/openapi.json`) are the authoritative reference for request and response bodies. This page is an index of the endpoints.

---

## Pi API

**Base URL:** `http://<pi-address>:8080` · Swagger: `http://<pi-address>:8080/docs`

!!! note "Network access"
    The node's dashboard and API are meant for a trusted local network or the VPN. Never port-forward them.

### State

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/state/` | System state for the controller's Sense-Think-Act loop |
| `POST` | `/state/heartbeat` | Controller heartbeat; keeps safety mode from engaging |
| `GET` | `/state/safety` | Safety mode status and last heartbeat time |

### Actuators

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/actuator/` | Every actuator with its capabilities, state and provenance |
| `POST` | `/actuator/{id}/command` | Command an actuator (refused with 409 when a safety bound blocks it) |
| `POST` | `/actuator/by-name/{name}/command` | Command an actuator by name |
| `POST` | `/actuator/{id}/release` | Drop the manual-override mark ("return to automatic") |
| `PATCH` | `/actuator/{id}/fail-safe` | Set whether the actuator keeps running through a safety-mode trip |

### Data

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/data` | Measurement history from the local database |
| `GET` | `/data/latest` | Latest measurement per active sensor component |
| `POST` | `/data/current_values` | Save the current sensor readings |

### Camera

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/camera/capture` | Capture a photo |
| `GET` | `/camera/stream` | Live MJPEG camera stream |

### Settings

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/settings` | Device settings |
| `PATCH` | `/settings/device-mode` | Change the device operating mode |
| `GET` / `POST` | `/settings/control-rules` | List or create control rules |
| `PATCH` | `/settings/control-rules/{id}` | Edit a control rule |
| `GET` | `/settings/growth-phases` | Growth phases |
| `GET` | `/settings/variables` | Variables |
| `GET` | `/settings/plant-species` | Plant species |
| `GET` | `/settings/components` | Components configured on the device, with calibration and liveness |
| `POST` | `/settings/components/{id}/calibration` | Calibrate a component |
| `GET` | `/settings/cultivation` | Active cultivation with its phase history |
| `GET` | `/settings/cultivation/template-sources` | Template sources available for a species |
| `POST` | `/settings/cultivation/begin` | Begin a cultivation |
| `POST` | `/settings/cultivation/end` | End the active cultivation |
| `POST` | `/settings/cultivation/advance-phase` | Advance to the next growth phase |
| `POST` | `/settings/sync` | Trigger a full cloud sync now |
| `GET` / `PATCH` | `/settings/runtime` | Per-device runtime settings (task intervals, heartbeat timeout) |
| `POST` | `/settings/reinit-drivers` | Re-run the driver bring-up from the local database |

### Meta and generic CRUD

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/` | Local dashboard, or API status when no dashboard is bundled |
| `GET` | `/version` | Build running on the device |

Generic CRUD routes (`GET /`, `GET /{id}`, `POST /`, `PUT /{id}`, `DELETE /{id}`) exist for `/device`, `/plant_species`, `/cultivation`, `/component_model`, `/device_component`, `/unit`, `/variable`, `/measurement` and `/component_capability`.

---

## Cloud API

The cloud API is not publicly reachable yet. For local development, run the cloud stack and open `http://localhost:8000/docs`.

### Gateway routing

| Path | Service |
|------|---------|
| `/auth/**` | auth-service |
| `/admin/**`, `/sync/**`, `/` | greenthumb-api |

account-service is not routed by the gateway; only auth-service calls it.

### Auth (`/auth`)

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/auth/login` | Returns `{token}` and sets the HttpOnly `access_token` cookie |
| `POST` | `/auth/logout` | Clears the `access_token` cookie |
| `POST` | `/auth/register` | Create a user account (admin only) |
| `GET` | `/auth/validate` | Internal, called by greenthumb-api: returns `{id_user, role}` |

### Admin (`/admin`)

Every `/admin` route needs a logged-in user: the `access_token` cookie, or `Authorization: Bearer <jwt>`. Routes marked **admin** also need the `admin` role.

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/admin/me` | Caller's user id and current role |
| `GET` | `/admin/version` | Cloud build, including the sync contract version |
| `GET` / `POST` | `/admin/devices` | List or create devices |
| `GET` / `PATCH` | `/admin/devices/{id}` | Read or edit a device |
| `DELETE` | `/admin/devices/{id}` | Delete a device (**admin**) |
| `POST` | `/admin/devices/{id}/token` | Generate a new device token; the old one stops working (**admin**) |
| `POST` | `/admin/devices/epoch/bump` | Make every device re-push its local data on the next sync (**admin**) |
| `GET` | `/admin/users`, `/admin/users/{id_user}` | List or read users (**admin**) |
| `PATCH` | `/admin/users/{id_user}` | Edit a user (**admin**) |
| `GET` | `/admin/alerts` | Fleet alerts (**admin**) |
| `POST` | `/admin/alerts/{id_system_alert}/resolve` | Resolve an alert (**admin**) |
| `GET` | `/admin/cultivations`, `/admin/cultivations/{id}` | List or read cultivations |
| `PATCH` | `/admin/cultivations/{id}` | Edit a cultivation |
| `GET` | `/admin/cultivation-phases`, `/admin/cultivation-phases/{id}` | Cultivation phases (read-only) |
| `GET` | `/admin/control-rule-logs` | Control-rule log (read-only) |
| `GET` | `/admin/actuator-logs/daily` | Daily actuator-log rollup |
| `GET` | `/admin/device-components/effects`, `/admin/device-components/{id_device_component}/effects` | Resolved actuator effects |

**Catalog CRUD** (`GET` for any logged-in user; `POST`, `PUT`, `DELETE` **admin**): `/admin/units`, `/admin/variables`, `/admin/plant-species`, `/admin/growth-phases`, `/admin/component-models`, `/admin/device-models`, `/admin/component-capabilities`, `/admin/species-rules`, `/admin/substances`, `/admin/substance-effects`, `/admin/component-model-effects`, `/admin/actuator-effects`.

**Owner-scoped CRUD** (a user sees only their own rows; an admin sees all): `/admin/properties`, `/admin/containers`, `/admin/device-components`, `/admin/control-rules`, `/admin/measurements`, `/admin/photos`, `/admin/actuator-logs`, `/admin/calibration-logs`.

CRUD routes follow the pattern `GET` (list), `GET /{id}`, `POST`, `PUT /{id}`, `DELETE /{id}`.

### Sync (`/sync`)

Called by the Pi. Every route needs `Authorization: Bearer <device_token>`; the token must match the device's `device_token`.

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/sync/devices/{id}/config` | Pull the shared catalog and the device configuration |
| `POST` | `/sync/devices/{id}/ping` | Liveness ping |
| `POST` | `/sync/devices/{id}/measurements` | Push measurements |
| `POST` | `/sync/devices/{id}/actuator-logs` | Push actuator logs |
| `POST` | `/sync/devices/{id}/calibration-logs` | Push calibration logs |
| `POST` | `/sync/devices/{id}/photos` | Push one photo (multipart); the API uploads it to R2 |
| `POST` | `/sync/devices/{id}/photos/batch` | Push several photos; returns 207 with a per-photo result |
| `POST` | `/sync/devices/{id}/state` | Push device-owned state (cultivations, phases, control rules) |

---

## Error responses

| Status | Meaning |
|--------|---------|
| `401` | Missing or invalid user or device token |
| `403` | Role not allowed, or the row belongs to another owner |
| `404` | Not found |
| `409` | Sync contract mismatch (sync), or command refused by a safety bound (Pi) |
| `422` | Validation error |
| `502` | Upstream failure (auth-service, R2) |
| `500` | Unexpected error |

---

Index generated from the routers on 2026-10-01; regenerate from `/openapi.json`.
