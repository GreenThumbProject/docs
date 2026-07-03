# Project Summary

This document provides a comprehensive overview of the GreenThumb project.

## Vision

GreenThumb is a **controlled-environment plant production system** that combines a Raspberry Pi controller, environmental sensors, automation, and a cloud backend to run greenhouses reliably and collect consistent cultivation data.

## Current Status

### Research Phase (2025-2026)

The system is being developed as part of a 12-month PIBITI undergraduate research project at Insper.

**Key Objectives:**

1. Build a reliable controlled-environment cultivation platform
2. Enable control of environmental variables through sensors and automation
3. Collect consistent, well-structured cultivation data
4. Validate the system with a physical prototype

### Completed Work

- ✅ Raspberry Pi 5 deployment with Docker Compose
- ✅ CI/CD pipeline (GitHub Actions → Docker Hub → Watchtower)
- ✅ I2C sensor integration (AHT10, BMP280, TSL2561)
- ✅ PostgreSQL database for data storage
- ✅ FastAPI REST API with centralized device management
- ✅ Live video streaming
- ✅ Shared library (`greenthumb-core`)
- ✅ Controller client with Sense-Think-Act loop
- ✅ Actuator system (RGB LED, water pump)
- ✅ Safety mode and heartbeat mechanism

### In Progress

- 🔄 Physical greenhouse prototype construction
- 🔄 pH and EC sensor integration
- 🔄 Cloud sync (Supabase + Cloudflare R2)
- 🔄 Computer vision for growth analysis

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

### Sensors

- **AHT10**: Temperature and humidity
- **BMP280**: Atmospheric pressure and temperature
- **TSL2561**: Light intensity
- **USB Camera**: Plant photos for computer vision

### Actuators

- **RGB LED**: PWM-controlled lighting
- **Water Pump**: PWM-controlled irrigation

### Planned

- **Supabase**: Cloud PostgreSQL database
- **Cloudflare R2**: Image storage
- **Computer Vision**: OpenCV for growth analysis
- **Machine Learning**: Growth prediction models

## System Architecture

The system uses a microservices architecture with centralized device management:

```
Raspberry Pi 5
├── PostgreSQL (database)
├── microcontroller-api (API + hardware control)
├── controller (Sense-Think-Act loop client)
├── cron (scheduled tasks)
└── watchtower (auto-updates)
```

All services run in Docker containers and share a common network.

## Repository Organization

| Repository | Purpose |
|------------|---------|
| `greenthumb-core` | Shared Python library |
| `rasp5` | Raspberry Pi 5 deployment |
| `microcontroller-api-client` | Controller scripts |
| `database` | Database schemas |
| `cron` | Scheduled tasks |
| `docs` | This documentation |
| `research` | Research papers |

## Data Collection

The system collects:

- **Sensor data** via API on-demand
- **Photos** for computer vision analysis

Data is stored locally and will sync to cloud storage for ML training.

## Long-term Goals

1. **Multiple Greenhouses**: Support managing more than one cultivation unit
2. **Improved Automation**: Refine environmental control and monitoring

## Contact

- **Developer**: Henrique Bucci R. Netto
- **Email**: henriquebrn@al.insper.edu.br
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)

---

*Last updated: February 2026*
