# Installation

GreenThumb can be deployed in two ways: using the **GreenthumbOS image** (planned) or by installing manually on an existing Raspberry Pi OS.

## Option A — GreenthumbOS image (planned)

A ready-to-flash Raspberry Pi OS image with Docker and the GreenThumb services preinstalled is planned but not available yet. Use Option B.

---

## Option B — Manual Installation

### Prerequisites

- Raspberry Pi 5 (4 GB+ RAM recommended)
- Raspberry Pi OS Lite 64-bit
- ≥32 GB SD card
- Docker CE + Docker Compose plugin installed
- I2C enabled (and 1-Wire if you use a DS18B20 water-temperature probe)

### 1. Enable Hardware Interfaces

On the Pi (Linux):

```bash
sudo raspi-config
# Interface Options → I2C → Enable
sudo reboot
```

Verify I2C:
```bash
sudo i2cdetect -y 1
# Should show sensor addresses (0x38 = AHT10, 0x76 = BMP280, 0x39 = TSL2561)
```

### 2. Connect Hardware

#### Sensors (I2C bus)

| Sensor | I2C Address | Measurements |
|--------|------------|--------------|
| AHT10 | 0x38 | Temperature, Humidity |
| BMP280 | 0x76 | Pressure, Temperature |
| TSL2561 | 0x39 | Light Intensity |

These are the usual factory addresses; the real ones are stored per device in the database.

#### Actuators (GPIO)

| Actuator | GPIO Pins | Purpose |
|----------|-----------|---------|
| Relay bank (8 channels, active-LOW) | BCM 17, 27, 22, 5, 6, 26, 23, 24 | Grow light, fan, water and air pumps, dosing pumps |

Pin assignments are stored per device in the database, not in code.

#### Camera

Connect a USB camera to any USB port (appears as `/dev/video0`).

### 3. Install Docker

On the Pi (Linux):

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 4. Clone the Repository

On the Pi (Linux):

```bash
git clone --recurse-submodules https://github.com/GreenThumbProject/rasp5.git ~/Documents/greenthumb/rasp5
```

The code repositories are private; you need access to the GreenThumbProject organization.

### 5. Configure Environment

On the Pi (Linux):

```bash
cd ~/Documents/greenthumb/rasp5/deploy
cp ../.env.example .env && chmod 600 .env && nano .env
ln -s deploy/.env ../.env
```

Minimum required variables:

```env
DEVICE_ID=1
DEVICE_TOKEN=<token from cloud admin — see below>
DB_PASSWORD=choose_a_strong_password
CLOUD_API_URL=<your sync endpoint>
```

See [Configuration](configuration.md) for all variables.

### 6. Provision the Device in the Cloud

Before the Pi can sync, you must register it in the cloud admin dashboard:

1. Log in to the cloud admin dashboard.
2. On **Fleet**, open the property and click **Add device**.
3. Open the new device and click **Rotate token** (administrators only). Copy the token; it is shown once.
4. Paste it into `DEVICE_TOKEN` in the Pi's `.env`.

!!! note
    The admin dashboard is reachable only over the VPN (see [Remote Access](vpn-setup.md)).

### 7. Start Services

On the Pi (Linux):

```bash
cd ~/Documents/greenthumb/rasp5/deploy
make pull
make up
```

This starts: `db`, `api`, `controller`, `local-dashboard`, `watchtower`.

### 8. Verify Installation

On the Pi, from `deploy/`:

```bash
# Check running containers
make ps

# Follow logs
make logs

# Test API
curl http://localhost:8080/state/

# Open local dashboard
# Navigate to http://<raspberry-pi-ip> in your browser
```

## Cloud stack (local development)

See [Local Setup](../development/local-setup.md) for the full development setup.

```powershell
git clone --recurse-submodules https://github.com/GreenThumbProject/greenthumb-cloud.git cloud
cd cloud
cp .env.example .env
docker compose up --build
# Admin dashboard: http://localhost:443 (plain HTTP)
# API docs:        http://localhost:8000/docs
# Landing page:    http://localhost:8083
```

## Next Steps

- [Configuration](configuration.md) — Full environment variable reference
- [Quick Start](quick-start.md) — Start collecting data
- [Remote Access (WireGuard)](vpn-setup.md) — Reach the node from anywhere, without exposing it
