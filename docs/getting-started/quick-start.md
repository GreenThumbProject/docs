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

## 4. Check Sensor Data

### Latest Readings

```bash
curl http://localhost:8080/data/latest
```

Response:

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

### Historical Data

```bash
curl "http://localhost:8080/data?limit=100"
```

## 5. Monitor Logs

```bash
# All services
make logs

# Specific service
make logs-data_collection
make logs-api
```

## Common Commands

| Command | Description |
|---------|-------------|
| `make up` | Start all services |
| `make down` | Stop all services |
| `make logs` | View logs |
| `make rebuild` | Full rebuild |
| `make db-shell` | PostgreSQL shell |

## Troubleshooting

### Camera Not Working

```bash
# Check if camera is detected
ls -la /dev/video0

# Check container logs
docker compose logs data_collection
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
