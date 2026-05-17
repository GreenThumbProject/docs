# Installation

GreenThumb can be deployed in two ways: using the **GreenthumbOS image** (recommended for production) or by installing manually on an existing Raspberry Pi OS.

## Option A — GreenthumbOS Image (Recommended)

!!! info "Planned for v1"
    GreenthumbOS is a pre-configured Raspberry Pi OS image with Docker, all system packages, and pre-pulled Docker images baked in. See `rasp5/resources/greenthumbos-plan.md` for the full specification.

1. Flash the GreenthumbOS image onto a ≥32 GB SD card using Balena Etcher or rpi-imager.
2. Mount the boot partition. Edit `/boot/greenthumb.env`:
   ```
   DEVICE_ID=5
   DEVICE_TOKEN=<token from cloud admin dashboard>
   ```
3. Edit `/boot/wpa_supplicant.conf` with your WiFi credentials (or use rpi-imager's Advanced Options).
4. Insert SD card, power on Pi. Wait ~90 s for first-boot setup.
5. SSH in and run:
   ```bash
   make up
   ```

That's it. No further configuration is required for a standard single-camera, single-sensor setup.

---

## Option B — Manual Installation

### Prerequisites

- Raspberry Pi 5 (4 GB+ RAM recommended)
- Raspberry Pi OS Lite 64-bit (Bookworm)
- ≥32 GB SD card
- Docker CE + Docker Compose plugin installed
- I2C and Camera interfaces enabled

### 1. Enable Hardware Interfaces

```bash
sudo raspi-config
# Interface Options → I2C → Enable
# Interface Options → Camera → Enable
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

#### Actuators (GPIO)

| Actuator | GPIO Pins | Purpose |
|----------|-----------|---------|
| RGB LED | 17, 27, 22 (R, G, B) | Grow lighting |
| Water Pump | 18 | Irrigation |

#### Camera

Connect a USB camera to any USB port (appears as `/dev/video0`).

### 3. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### 4. Clone the Repository

```bash
git clone https://github.com/henriquebrnetto/rasp5.git /opt/greenthumb
cd /opt/greenthumb
```

### 5. Configure Environment

```bash
cp .env.example .env
nano .env
```

Minimum required variables:

```env
DEVICE_ID=1
DEVICE_TOKEN=<token from cloud admin — see below>
DB_PASSWORD=choose_a_strong_password
```

See [Configuration](configuration.md) for all variables.

### 6. Provision the Device in the Cloud

Before the Pi can sync, you must register it in the cloud admin dashboard:

1. Log in to the cloud admin dashboard.
2. Navigate to **Devices → New Device** — fill in name and location.
3. Click **Rotate Token** next to the new device — copy the generated token.
4. Paste it into `DEVICE_TOKEN` in the Pi's `.env`.

### 7. Start Services

```bash
make up
```

This starts: `db`, `api`, `controller`, `local-dashboard`, `watchtower`.

### 8. Verify Installation

```bash
# Check running containers
docker compose ps

# Follow logs
make logs

# Test API
curl http://localhost:8080/state/

# Open local dashboard
# Navigate to http://<pi-ip> in your browser
```

## Cloud Deployment

The cloud stack lives in `cloud/`. See [Local Setup](../development/local-setup.md) for development, or the `cloud/k8s/` directory for Kubernetes manifests.

```bash
cd cloud
cp .env.example .env
docker compose up --build
# Admin dashboard: http://localhost:3000
# API docs:        http://localhost:8000/docs
```

## Next Steps

- [Configuration](configuration.md) — Full environment variable reference
- [Quick Start](quick-start.md) — Start collecting data
- [Tailscale Setup](tailscale-setup.md) — Enable remote access
