# GreenThumb Project

**A scalable platform for controlled environment plant production and high-fidelity data collection.**

<center><img src="assets/images/greenthumb-logo.png" alt="drawing" width="300"></center>

## What is GreenThumb?

GreenThumb is a distributed system for controlled environment plant production, designed with a focus on **massive data collection and horizontal scalability**. Each cultivation unit acts as a modular node in a data collection cluster, enabling total control of environmental variables and generation of high volumes of standardized phenotypic data.

## Origin

GreenThumb began as a 12-month PIBITI undergraduate research project, with the goal of building a foundation for ML-based crop optimization.

## Key Features

- 🌡️ **Environmental Monitoring** - Air temperature, humidity, pressure and light; water temperature, level, pH and TDS
- 📸 **Plant Imaging** - Scheduled photos from a USB camera (image analysis is future work)
- 🔄 **Automated Control** - Relay-switched grow light, fan and pumps, driven by threshold, schedule and interval rules
- 📊 **Data Collection** - Continuous sensor data with cloud sync
- 🐳 **Containerized** - Docker-based deployment on Raspberry Pi 5
- 🔀 **Horizontal Scalability** - Modular nodes for data collection clusters

## Quick Links

| Section | Description |
|---------|-------------|
| :material-rocket-launch: [**Getting Started**](getting-started/installation.md) | Set up your own GreenThumb greenhouse |
| :material-cog: [**Architecture**](architecture/overview.md) | Understand the system design |
| :material-code-tags: [**API Reference**](api/reference.md) | Explore the REST API endpoints |
| :material-github: [**Source Code**](architecture/repositories.md) | Repository map |

## Technology Stack

| Component | Technology |
|-----------|------------|
| Controller | Raspberry Pi 5 |
| Language | Python 3.11+ |
| Web Framework | FastAPI |
| Database | PostgreSQL 17 |
| ORM | SQLModel |
| Cloud database | PostgreSQL 17 + TimescaleDB |
| Auth and gateway | Java 21, Spring Boot |
| Dashboards | React |
| Containers | Docker Compose |
| CI/CD | GitHub Actions |

## Contact

- **Developer**: Henrique Bucci R. Netto
- **GitHub**: [@henriquebrnetto](https://github.com/henriquebrnetto)
- **Organization**: [GreenThumbProject](https://github.com/GreenThumbProject)
