# Research Overview

GreenThumb is developed as part of a PIBITI research project.

## Project Information

**Title**: Scalable Controlled Environment Plant Production System for High-Fidelity Data Collection

**Institution**: Insper Instituto de Ensino e Pesquisa

**Duration**: August 2025 - August 2026 (12 months)

**Researcher**: Henrique Bucci R. Netto

## Abstract

The development of machine learning models for crop optimization faces a fundamental challenge in the scarcity of standardized, high-quality phenotypic datasets. This work proposes the development of a distributed system for controlled environment plant production, shifting the focus from isolated physical infrastructure to an architecture oriented toward massive data collection and horizontal scalability.

The main objective is to consolidate a platform where each cultivation unit acts as a modular node in a data collection cluster, enabling total control of environmental variables and the generation of high volumes of standardized data.

## Objectives

### General Objective

Develop a distributed system for controlled environment plant production oriented toward massive data collection and horizontal scalability, establishing a foundation for ML-based crop optimization.

### Specific Objectives

1. Consolidate a platform where each cultivation unit acts as a modular node in a data collection cluster
2. Enable total control of environmental variables through sensor instrumentation and automation
3. Generate high volumes of standardized phenotypic data for ML model training
4. Implement container orchestration ensuring software environment parity across nodes
5. Develop a computer vision system for continuous plant monitoring
6. Validate the platform through a physical prototype at the end of the project

## Methodology

### Software Architecture

- **Microservices orchestration** via containers on single-board computers
- **Software environment parity** across nodes for consistent data collection
- **CI/CD pipeline** for continuous integration and deployment
- **PostgreSQL database** for structured data storage
- **RESTful API** with centralized device management pattern

### Hardware

- Raspberry Pi 5 for control and data processing
- Environmental sensors (temperature, humidity, pressure, light)
- USB camera for computer vision
- Actuators (LEDs, water pumps) with PWM control
- Mini-greenhouse prototype for validation

### Experimental Design

1. **Months 1-4**: Software architecture development and CI/CD pipeline
2. **Months 5-6**: Sensor integration and automation validation
3. **Month 7**: Computer vision implementation
4. **Months 8-9**: Database structuring and horizontal expansion
5. **Months 10-11**: Prototype construction and system validation
6. **Month 12**: Results consolidation and final report

## Partial Results

- ✅ Software architecture with CI/CD pipeline (GitHub Actions + Docker Hub + Watchtower)
- ✅ Validation of automation and remote access prototype
- ✅ Database structuring for horizontal expansion
- ✅ Centralized API-as-Device-Manager pattern implementation
- ✅ Controller client with Sense-Think-Act loop
- ✅ Actuator system (RGB LED, water pump)

## Expected Outcomes

- Robust datasets for training computer vision and growth prediction models
- Platform enabling horizontal expansion for data collection clusters
- Foundation for future ML agent replacement of manual control
- Physical prototype validating the system operation
- Technical documentation for replication
- Research paper for publication

## Related Work

Key references informing this research:

- Boonnam et al. (2017) - Optimal Plant Growth in Smart Farm Hydroponics
- Nesteruk et al. (2020) - Plant Growth Prediction through Embedded Sensing
- Nnadi & Idachaba (2018) - Sustainable IoT Greenhouse Prototype

See [References](references.md) for complete bibliography.
