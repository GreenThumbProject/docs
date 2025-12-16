# GreenThumb Project

**An AI-powered automated greenhouse ecosystem for optimal plant growth.**

<center><img src="assets/images/greenthumb-logo.png" alt="drawing" width="300"></center>

## What is GreenThumb?

GreenThumb is an open-source automated greenhouse project that combines IoT sensors, computer vision, and machine learning to create optimal growing environments for plants.

## Current Phase

!!! info "Research Phase"
    We are currently developing a **low-cost hydroponic platform** for cherry tomato cultivation as part of a 12-month research project (PIBITI).

## Key Features

- 🌡️ **Environmental Monitoring** - Temperature, humidity, pressure, and light intensity
- 📸 **Computer Vision** - Automated plant growth analysis via camera
- 🔄 **Automated Control** - PWM-controlled LEDs and water pumps
- 📊 **Data Collection** - Continuous sensor data with cloud sync
- 🐳 **Containerized** - Docker-based deployment on Raspberry Pi 5

## Quick Links

| Section | Description |
|---------|-------------|
| :material-rocket-launch: [**Getting Started**](getting-started/installation.md) | Set up your own GreenThumb greenhouse |
| :material-cog: [**Architecture**](architecture/overview.md) | Understand the system design |
| :material-code-tags: [**API Reference**](api/reference.md) | Explore the REST API endpoints |
| :material-github: [**Source Code**](architecture/repositories.md) | View the code on GitHub |

## Technology Stack

| Component | Technology |
|-----------|------------|
| Controller | Raspberry Pi 5 |
| Language | Python 3.11+ |
| Web Framework | FastAPI |
| Database | PostgreSQL 17 |
| ORM | SQLModel |
| Containers | Docker Compose |
| CI/CD | GitHub Actions |

## Contact

- **Developer**: Henrique Bucci R. Netto
- **GitHub**: [@henriquebrnetto](https://github.com/henriquebrnetto)
- **Organization**: [GreenThumbProject](https://github.com/GreenThumbProject)
