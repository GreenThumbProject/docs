# GreenThumb Project

**An AI-powered automated greenhouse ecosystem for optimal plant growth.**

![GreenThumb](assets/images/logo-placeholder.png){ align=right width=200 }

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

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } **Getting Started**

    ---

    Set up your own GreenThumb greenhouse

    [:octicons-arrow-right-24: Installation](getting-started/installation.md)

-   :material-cog:{ .lg .middle } **Architecture**

    ---

    Understand the system design

    [:octicons-arrow-right-24: Overview](architecture/overview.md)

-   :material-code-tags:{ .lg .middle } **API Reference**

    ---

    Explore the REST API endpoints

    [:octicons-arrow-right-24: API Docs](api/reference.md)

-   :material-github:{ .lg .middle } **Source Code**

    ---

    View the code on GitHub

    [:octicons-arrow-right-24: Repositories](architecture/repositories.md)

</div>

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
