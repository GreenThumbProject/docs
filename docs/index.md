# GreenThumb Project

**A scalable platform for controlled environment plant production and high-fidelity data collection.**

<center><img src="assets/images/greenthumb-logo.png" alt="drawing" width="300"></center>

## What is GreenThumb?

GreenThumb is a distributed system for controlled environment plant production, designed with a focus on **massive data collection and horizontal scalability**. Each cultivation unit acts as a modular node in a data collection cluster, enabling total control of environmental variables and generation of high volumes of standardized phenotypic data.

## Current Phase

!!! info "Research Phase"
    We are developing a **scalable controlled environment plant production system** as part of a 12-month research project (PIBITI), with the goal of establishing a foundation for ML-based crop optimization.

## Key Features

- 🌡️ **Environmental Monitoring** - Temperature, humidity, pressure, and light intensity
- 📸 **Computer Vision** - Automated plant growth analysis via camera
- 🔄 **Automated Control** - PWM-controlled LEDs and water pumps
- 📊 **Data Collection** - Continuous sensor data with cloud sync
- 🐳 **Containerized** - Docker-based deployment on Raspberry Pi 5
- 🔀 **Horizontal Scalability** - Modular nodes for data collection clusters

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
