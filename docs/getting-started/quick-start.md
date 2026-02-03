# Quick Start

Get your GreenThumb greenhouse running in 5 minutes.

## Prerequisites

Complete the [Installation](installation.md) guide first.

## 1. Start the System

```bash
cd rasp5
make up
```

## 2. Access the Dashboard

Open your browser and navigate to:

```
http://<raspberry-pi-ip>:8080
```

Or from the Raspberry Pi itself:

```
http://localhost:8080
```

## 3. View Live Video

The camera feed is available at:

```
http://<raspberry-pi-ip>:8080/video
```

## 4. Check System State

### Get Current State (Sensors + Thresholds)

```bash
curl http://localhost:8080/state/
```

Response:

```json
{
  "sensors": {...},
  "thresholds": [...],
  "safety_mode": false,
  "timestamp": "2026-02-03T12:00:00"
}
```

### Get Latest Sensor Data

```bash
curl http://localhost:8080/data/latest
```

Response:

```json
{
  "device_id": 1,
  "data": {
    "Temperature": {"value": 25.3, "unit": "°C", "collected_at": "..."},
    "Humidity": {"value": 65.2, "unit": "%", "collected_at": "..."}
  }
}
```

## 5. Control Actuators

### Set RGB LED Color

```bash
curl -X POST "http://localhost:8080/state/actuators/1/command" \
  -H "Content-Type: application/json" \
  -d '{"r": 255, "g": 0, "b": 128}'
```

### Set Pump Duty Cycle

```bash
curl -X POST "http://localhost:8080/state/actuators/2/command" \
  -H "Content-Type: application/json" \
  -d '{"duty_cycle": 50}'
```

## 6. Monitor Logs

```bash
# All services
make logs

# Specific services
make logs-api    # API logs
make logs-ctrl   # Controller logs
```

## Common Commands

| Command | Description |
|---------|-------------|
| `make up` | Start all services |
| `make down` | Stop all services |
| `make logs` | View all logs |
| `make logs-api` | View API logs |
| `make logs-ctrl` | View controller logs |
| `make rebuild` | Rebuild services |
| `make db-shell` | PostgreSQL shell |

## Troubleshooting

### Camera Not Working

```bash
# Check if camera is detected
ls -la /dev/video0

# Check container logs
make logs-api
```

### Sensors Not Reading

```bash
# Check I2C devices
i2cdetect -y 1

# Expected addresses:
# 0x38 - AHT10
# 0x39 - TSL2561
# 0x76 - BMP280
```

### Controller Not Connecting

```bash
# Check controller logs
make logs-ctrl

# Verify API is healthy
curl http://localhost:8080/
```

### Safety Mode Activated

If actuators turn off unexpectedly, the controller may have crashed:

```bash
# Check controller status
docker compose ps controller

# Restart controller
make restart-controller
```

### Database Connection Issues

```bash
# Check database container
docker compose logs db

# Restart database
docker compose restart db
```

## Next Steps

- [Architecture Overview](../architecture/overview.md)
- [API Reference](../api/reference.md)
- [Actuators](../components/actuators.md)
