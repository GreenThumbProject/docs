# Installation

This guide covers installing the GreenThumb system on a Raspberry Pi 5.

## Prerequisites

- Raspberry Pi 5 (4GB+ RAM recommended)
- Raspberry Pi OS (64-bit)
- Docker and Docker Compose installed
- I2C enabled
- USB camera connected

## Hardware Setup

### Enable I2C

```bash
sudo raspi-config
# Navigate to: Interface Options > I2C > Enable
sudo reboot
```

### Connect Sensors

Connect the following sensors to the I2C bus:

| Sensor | Address | Measurements |
|--------|---------|--------------|
| AHT10 | 0x38 | Temperature, Humidity |
| BMP280 | 0x76 | Pressure, Temperature |
| TSL2561 | 0x39 | Light Intensity |

### Connect Actuators (Optional)

| Actuator | GPIO Pins | Purpose |
|----------|-----------|---------|
| RGB LED | 17, 27, 22 | Grow lighting |
| Water Pump | 18 | Irrigation |

## Software Installation

### 1. Clone the Repository

```bash
git clone https://github.com/GreenThumbProject/rasp5.git
cd rasp5
git submodule update --init --recursive
```

### 2. Configure Environment

```bash
cp .env.example .env
nano .env  # Edit with your values
```

Required environment variables:

| Variable | Description |
|----------|-------------|
| `DB_PASSWORD` | PostgreSQL password |
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `GH_PAT` | GitHub PAT for private repos |

### 3. Start Services

```bash
make up
```

This starts all services:

- PostgreSQL database
- Microcontroller API (port 8080)
- Controller (Sense-Think-Act loop)
- Watchtower (auto-updates)

### 4. Verify Installation

```bash
# Check running containers
docker compose ps

# View logs
make logs

# Access the dashboard
curl http://localhost:8080
```

## Makefile Quick Reference

| Command | Description |
|---------|-------------|
| `make up` | Start all services |
| `make down` | Stop all services |
| `make pull` | Pull latest images from Docker Hub |
| `make logs` | Follow logs from all services |
| `make logs-api` | Follow API logs only |
| `make logs-ctrl` | Follow controller logs only |
| `make db-shell` | Open PostgreSQL shell |
| `make rebuild` | Rebuild api + controller |
| `make clean` | Remove all (DANGER!) |

## Next Steps

- [Configuration](configuration.md) - Customize your setup
- [Quick Start](quick-start.md) - Start collecting data
