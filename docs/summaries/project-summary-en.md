# Project Summary

This document provides an overview of the GreenThumb project.

## Vision

GreenThumb is a **controlled-environment plant production system** that combines a Raspberry Pi controller, environmental sensors, automation, and a cloud backend to run greenhouses reliably and collect consistent cultivation data.

## Current Status

### Origin

GreenThumb started as a 12-month PIBITI undergraduate research project at Insper (2025–2026). The research phase closed with the final report in August 2026; development of the system continues.

### What exists today

- Edge node on a Raspberry Pi 5 with Docker Compose (PostgreSQL, device API, controller, local dashboard, Watchtower)
- Sensor drivers: AHT10, BMP280, TSL2561, DS18B20, float switch, pH and TDS/EC probes, USB camera
- Relay-switched actuators: grow light, exhaust fan, water and air pumps, peristaltic dosing pumps
- Controller with threshold, schedule, interval and after-actuator rules, safety bounds and a heartbeat watchdog (safety mode)
- Periodic photos and a live camera stream
- Offline-first sync to a self-hosted cloud (PostgreSQL 17 + TimescaleDB; photos in Cloudflare R2)
- Cloud API, authentication services and an admin dashboard (not public yet)
- CI/CD: GitHub Actions → Docker Hub

### Next

- 🔄 Finishing the physical prototype
- 🔄 Calibrating the dosing pumps and the pH and TDS probes
- First cultivation run (cherry tomato)

### Planned

- **Computer Vision**: OpenCV for growth analysis
- **Machine Learning**: Growth prediction models

## Technology Stack

| Component | Technology |
|-----------|------------|
| Controller | Raspberry Pi 5 |
| Language | Python 3.11+ |
| Web Framework | FastAPI |
| Database | PostgreSQL 17 |
| ORM | SQLModel |
| Containers | Docker Compose |
| CI/CD | GitHub Actions → Docker Hub |
| Cloud services | FastAPI; Java (Spring Boot, Spring Cloud Gateway) |
| Dashboards | React |
| Cloud data | PostgreSQL 17 + TimescaleDB; Cloudflare R2 (photos) |
| Networking | WireGuard |

### Sensors

- **AHT10**: Temperature and humidity
- **BMP280**: Atmospheric pressure and temperature
- **TSL2561**: Light intensity
- **DS18B20**: Water temperature
- **Float switch**: Tank level (full or empty)
- **pH and TDS/EC probes**: Nutrient solution pH and conductivity
- **USB Camera**: Plant photos for computer vision

### Actuators

- **Grow light, exhaust fan, water and air pumps**: switched by relay
- **Peristaltic dosing pumps**: nutrient and pH dosing with safety bounds

## System Architecture

The system uses a microservices architecture with centralized device management:

```
Raspberry Pi 5
├── PostgreSQL (database)
├── microcontroller-api (API + hardware control)
├── controller (Sense-Think-Act loop client)
├── local-dashboard (React SPA)
└── watchtower (auto-updates)
```

All services run in Docker containers and share a common network.

A self-hosted cloud stack (API, gateway, authentication services, admin dashboard, PostgreSQL + TimescaleDB) receives the synced data. See [Cloud Backend](../components/cloud.md).

## Repository Organization

See [Repositories](../architecture/repositories.md). Most repositories are private; this documentation and the organisation profile are public.

## Data Collection

The system collects:

- **Sensor data**, logged periodically
- **Photos**, captured periodically for future computer-vision work

Data is stored on the node first and synced to the cloud when a connection is available.

## Long-term Goals

1. **Many nodes**: the cloud already registers and lists multiple devices; self-service onboarding is still to come.
2. **Improved Automation**: Refine environmental control and monitoring

## Contact

- **Developer**: Henrique Bucci R. Netto
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)
