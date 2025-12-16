# Project Summary

This document provides a comprehensive overview of the GreenThumb project.

## Vision

GreenThumb aims to create an **AI-powered automated greenhouse ecosystem** that establishes optimal growing environments for any plant species, maximizing growth efficiency while minimizing resource consumption.

## Current Status

### Research Phase (2025)

We are developing a **low-cost hydroponic platform** for cherry tomato cultivation as part of a 12-month PIBITI research project at the university.

**Key Objectives:**

1. Build a mini-greenhouse with automated environmental control
2. Implement continuous monitoring (pH, EC, temperature, humidity, light)
3. Develop computer vision for plant growth analysis
4. Validate the system through a complete cultivation cycle

### Completed Work

- ✅ Raspberry Pi 5 deployment with Docker Compose
- ✅ I2C sensor integration (AHT10, BMP280, TSL2561)
- ✅ PostgreSQL database for data storage
- ✅ FastAPI REST API with live video streaming
- ✅ GitHub Actions CI/CD pipeline
- ✅ Automatic container updates via Watchtower
- ✅ Shared library (`greenthumb-core`)

### In Progress

- 🔄 Physical greenhouse construction
- 🔄 pH and EC sensor integration
- 🔄 LED and pump PWM control
- 🔄 Cloud sync (Supabase + Cloudflare R2)

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

### Planned

- **Supabase**: Cloud PostgreSQL database
- **Cloudflare R2**: Image storage
- **Computer Vision**: OpenCV for growth analysis
- **Machine Learning**: Growth prediction models

## System Architecture

The system uses a microservices architecture:

```
Raspberry Pi 5
├── PostgreSQL (database)
├── FastAPI (API + dashboard)
├── data_collection (sensor readings + photos)
├── cron (scheduled tasks)
└── watchtower (auto-updates)
```

All services run in Docker containers and share a common network.

## Repository Organization

| Repository | Purpose |
|------------|---------|
| `greenthumb-core` | Shared Python library |
| `rasp5` | Raspberry Pi 5 deployment |
| `database` | Database schemas |
| `cron` | Scheduled tasks |
| `docs` | This documentation |
| `research` | Research papers |

## Data Collection

The system collects:

- **Sensor data** every 30 minutes
- **Photos** every 4 hours

Data is stored locally and will sync to cloud storage daily.

## Long-term Goals

1. **Multiple Greenhouses**: Scale to manage multiple units
2. **ML Optimization**: Use collected data to optimize growing conditions
3. **Commercial Product**: Develop a reproducible, sellable greenhouse kit
4. **Open Source**: Share knowledge and tools with the community

## Contact

- **Developer**: Henrique Bucci R. Netto
- **Email**: henriquebrn@al.insper.edu.br
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)

---

*Last updated: December 2025*
