# API Reference

The GreenThumb API provides REST endpoints for accessing sensor data and system information.

## Base URL

```
http://<raspberry-pi-ip>:8080
```

## Endpoints

### Home

#### `GET /`

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

### Video Stream

#### `GET /video`

Live MJPEG video stream from the camera.

**Response:**

- Content-Type: `multipart/x-mixed-replace; boundary=frame`
- Continuous JPEG frames

**Usage in HTML:**

```html
<img src="http://localhost:8080/video" alt="Live Feed">
```

!!! note
    Camera is opened on-demand and released when client disconnects. If camera is in use by `data_collection`, streaming will fail gracefully.

---

### Sensor Data

#### `GET /data`

Get the latest sensor measurements.

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

#### `GET /data/latest`

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
    },
    "Pressure": {
      "value": 1013.25,
      "unit": "hPa",
      "collected_at": "2025-12-16T02:30:00"
    },
    "Light": {
      "value": 450,
      "unit": "lux",
      "collected_at": "2025-12-16T02:30:00"
    }
  }
}
```

---

## Error Responses

### No Sensors Found

```json
{
  "error": "No sensors found for device 1"
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
