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

## Software Installation

### 1. Clone the Repository

```bash
git clone https://github.com/GreenThumbProject/rasp5.git
cd rasp5
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
- FastAPI server (port 8080)
- Data collection service
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

## Next Steps

- [Configuration](configuration.md) - Customize your setup
- [Quick Start](quick-start.md) - Start collecting data
