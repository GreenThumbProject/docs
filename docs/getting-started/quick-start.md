# Quick Start

Get your GreenThumb greenhouse running in 5 minutes.

## Prerequisites

Complete the [Installation](installation.md) guide first.

## 1. Start the System

On the Pi (Linux):

```bash
cd ~/Documents/greenthumb/rasp5/deploy
make up
```

## 2. Access the Local Dashboard

Open your browser and navigate to:

```
http://<raspberry-pi-ip>
```

The local dashboard has four pages: Dashboard (live readings, history, camera), Actuators, Cultivation (control rules) and Settings (sync).

## 3. View Live Video

The MJPEG stream is embedded in the dashboard. Direct URL:

```
http://<raspberry-pi-ip>/camera/stream
```

## 4. Check System State

### Get Current State

On the Pi (Linux):

```bash
curl http://localhost:8080/state/
```

Response:

```json
{
  "sensors": {...},
  "sensor_values": {...},
  "variables": {...},
  "control_rules": [...],
  "actuators": [...],
  "effects": [...],
  "active_phase": {...},
  "safety_mode": false,
  "timezone": "...",
  "timestamp": "..."
}
```

### Get Latest Sensor Data

```bash
curl http://localhost:8080/data/latest
```

Returns a list with the latest measurement row for each active sensor.

## 5. Control Actuators

On the Pi (Linux):

```bash
# List actuators
curl http://localhost:8080/actuator/

# Turn one on for 30 seconds
curl -X POST http://localhost:8080/actuator/<id>/command \
  -H "Content-Type: application/json" \
  -d '{"action": "on", "duration_s": 30}'
```

A manual command holds the actuator until you release it:

```bash
curl -X POST http://localhost:8080/actuator/<id>/release
```

A 409 means a safety bound refused the command.

## 6. Monitor Logs

On the Pi (Linux), from `deploy/`:

```bash
# All services
make logs

# Specific services
make logs-api          # API logs
make logs-controller   # Controller logs
```

## Common Commands

Run from `deploy/` on the Pi.

| Command | Description |
|---------|-------------|
| `make up` | Start all services |
| `make down` | Stop all services |
| `make ps` | Service status |
| `make logs` | Follow all logs |
| `make logs-<svc>` | Follow one service (e.g. `make logs-controller`) |
| `make restart-<svc>` | Restart one service |
| `make pull` | Pull new images (does not restart api or controller) |
| `make promote` | Apply pulled api and controller images (relays may switch; do it with the rig in view) |
| `make deploy` | `git pull`, then up |
| `make status` | Pi health: temperature, throttling, load |
| `make db-shell` | PostgreSQL shell |
| `make verify` | Project and volume names, build versions, sync backlog |

## Trigger a Manual Cloud Sync

On the Pi (Linux):

```bash
curl -X POST http://localhost:8080/settings/sync
```

Or click **Sync Now** in the local dashboard Settings page.

## Troubleshooting

Run these on the Pi, from `~/Documents/greenthumb/rasp5/deploy`.

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
sudo i2cdetect -y 1

# Expected addresses:
# 0x38 - AHT10
# 0x39 - TSL2561
# 0x76 - BMP280
```

### Controller Not Connecting

```bash
# Check controller logs
make logs-controller

# Verify API is healthy
curl http://localhost:8080/
```

### Safety Mode Activated

If actuators turn off unexpectedly, the API has not heard from the controller for 60 s and has entered safety mode. Check `curl http://localhost:8080/state/safety`, then:

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
