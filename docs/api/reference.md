# API Reference

The GreenThumb API provides REST endpoints for accessing sensor data, controlling actuators, and system management.

## Base URL

```
http://<raspberry-pi-ip>:8080
```

## Endpoints Overview

| Category | Prefix | Description |
|----------|--------|-------------|
| Home | `/` | Dashboard and status |
| State | `/state` | System state and actuator control |
| Data | `/data` | Sensor measurements |
| Video | `/video` | Live camera stream |
| CRUD | Various | Auto-generated model endpoints |

---

## Home

### `GET /`

Returns the dashboard or API status.

**Response:**

- If `static/index.html` exists: Returns the HTML dashboard
- Otherwise: Returns JSON status

```json
{
  "message": "GreenThumb API is running"
}
```

---

## State Routes

The `/state` endpoints provide centralized system state and actuator control for the controller's Sense-Think-Act loop.

### `GET /state/`

Get sensors + thresholds in a single response.

**Response:**

```json
{
  "sensors": {
    "1": [{"id_variable": 1, "value": 25.3}, ...],
    "2": [{"id_variable": 2, "value": 65.2}, ...]
  },
  "thresholds": [
    {
      "id_threshold": 1,
      "id_cultivation": 1,
      "id_variable": 1,
      "min_value": 20.0,
      "max_value": 30.0,
      "target_value": 25.0,
      "id_actuator_action": 1
    }
  ],
  "safety_mode": false,
  "timestamp": "2026-02-03T12:00:00"
}
```

---

### `POST /state/actuators/{id}/command`

Send command to actuator (concurrent-safe with per-actuator locking).

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | int | Device actuator ID |

**Request Body:**

For RGB LED:
```json
{
  "r": 255,
  "g": 128,
  "b": 0
}
```

For Water Pump:
```json
{
  "duty_cycle": 100
}
```

**Response:**

```json
{
  "status": "ok",
  "actuator_id": 1,
  "command": {"duty_cycle": 100}
}
```

---

### `POST /state/heartbeat`

Controller heartbeat to prevent safety mode.

**Response:**

```json
{
  "status": "ok",
  "safety_mode": false,
  "last_heartbeat": "2026-02-03T12:00:00"
}
```

---

### `GET /state/safety`

Get current safety mode status.

**Response:**

```json
{
  "safety_mode": false,
  "last_heartbeat": "2026-02-03T12:00:00"
}
```

---

## Data Routes

### `GET /data/data`

Get latest sensor measurements from the database.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `device_id` | int | 1 | Device ID to query |
| `limit` | int | 10 | Number of measurements per variable |

**Response:**

```json
{
  "device_id": 1,
  "data": {
    "Temperature": {
      "unit": "°C",
      "latest": 25.3,
      "latest_at": "2025-12-16T02:30:00",
      "history": [
        {"value": 25.3, "collected_at": "2025-12-16T02:30:00"},
        {"value": 25.1, "collected_at": "2025-12-16T02:00:00"}
      ]
    },
    "Humidity": {
      "unit": "%",
      "latest": 65.2,
      "latest_at": "2025-12-16T02:30:00",
      "history": [...]
    }
  }
}
```

---

### `GET /data/latest`

Get only the most recent measurement for each variable.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `device_id` | int | 1 | Device ID to query |

**Response:**

```json
{
  "device_id": 1,
  "data": {
    "Temperature": {
      "value": 25.3,
      "unit": "°C",
      "collected_at": "2025-12-16T02:30:00"
    },
    "Humidity": {
      "value": 65.2,
      "unit": "%",
      "collected_at": "2025-12-16T02:30:00"
    }
  }
}
```

---

## Video Stream

### `GET /video`

Live MJPEG video stream from the camera.

**Response:**

- Content-Type: `multipart/x-mixed-replace; boundary=frame`
- Continuous JPEG frames

**Usage in HTML:**

```html
<img src="http://localhost:8080/video" alt="Live Feed">
```

!!! note
    Camera is opened on-demand and released when client disconnects. If camera is in use, streaming will fail gracefully.

---

## CRUD Routes

Auto-generated endpoints for all database models using `greenthumb_core.api.make_crud_router()`.

### Available Resources

| Resource | Prefix | Operations |
|----------|--------|------------|
| Device | `/device` | GET, POST, PUT, DELETE |
| Plant Species | `/plant_species` | GET, POST, PUT, DELETE |
| Cultivation | `/cultivation` | GET, POST, PUT, DELETE |
| Sensor Model | `/sensor_model` | GET, POST, PUT, DELETE |
| Device Sensor | `/device_sensor` | GET, POST, PUT, DELETE |
| Unit | `/unit` | GET, POST, PUT, DELETE |
| Variable | `/variable` | GET, POST, PUT, DELETE |
| Measurement | `/measurement` | GET, POST, PUT, DELETE |
| Sensor Capability | `/sensor_capability` | GET, POST, PUT, DELETE |

### Standard CRUD Operations

Each resource supports:

- `GET /{prefix}/` - List all
- `GET /{prefix}/{id}` - Get by ID
- `POST /{prefix}/` - Create new
- `PUT /{prefix}/{id}` - Update
- `DELETE /{prefix}/{id}` - Delete

---

## Error Responses

### No Sensors Found

```json
{
  "error": "No sensors found for device 1"
}
```

### Actuator Not Found

```json
{
  "detail": "Actuator 99 not found"
}
```

### Internal Server Error

```json
{
  "detail": "Internal server error"
}
```

---

## OpenAPI Documentation

Interactive API documentation is available at:

- **Swagger UI**: `http://<ip>:8080/docs`
- **ReDoc**: `http://<ip>:8080/redoc`
- **OpenAPI JSON**: `http://<ip>:8080/openapi.json`
