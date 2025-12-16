# Architecture Overview

GreenThumb uses a multi-repository, microservices architecture deployed on a Raspberry Pi 5.

## System Diagram

```mermaid
graph TB
    subgraph "Raspberry Pi 5"
        subgraph "Docker Compose"
            DC["data_collection"]
            API["fastapi (port 8080)"]
            DB[(PostgreSQL)]
            CRON["cron"]
        end
        
        subgraph "Hardware"
            CAM[USB Camera]
            I2C[I2C Bus]
            AHT[AHT10]
            BMP[BMP280]
            TSL[TSL2561]
        end
    end
    
    subgraph "Shared Libraries"
        CORE["greenthumb-core"]
    end
    
    subgraph "Cloud (Future)"
        SUPA[(Supabase)]
        R2[Cloudflare R2]
    end
    
    DC --> DB
    DC --> CAM
    DC --> I2C
    I2C --> AHT
    I2C --> BMP
    I2C --> TSL
    API --> DB
    CRON --> DB
    CRON -.-> SUPA
    CRON -.-> R2
    DC -.-> CORE
    API -.-> CORE
```

## Services

### Data Collection

The `data_collection` service runs continuously, collecting:

- **Sensor data** every 30 minutes
- **Photos** every 4 hours

Data is stored in the local PostgreSQL database.

### FastAPI

The `api` service provides:

- REST API endpoints for data access
- Live video streaming from the camera
- Static dashboard

### PostgreSQL

Local database storing:

- Device and sensor configurations
- Plant species catalog
- Measurement history
- Cultivation records

### Cron (Optional)

Scheduled tasks for:

- Cloud database sync (future)
- Local database cleanup
- Backup operations

## Data Flow

```mermaid
sequenceDiagram
    participant S as Sensors
    participant DC as data_collection
    participant DB as PostgreSQL
    participant API as FastAPI
    participant UI as Dashboard
    
    loop Every 30 minutes
        S->>DC: Read sensor values
        DC->>DB: Store measurements
    end
    
    UI->>API: GET /data/latest
    API->>DB: Query measurements
    DB-->>API: Return data
    API-->>UI: JSON response
```

## Technology Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Controller | Raspberry Pi 5 | Powerful, runs Linux, Python support |
| Database | PostgreSQL | Robust, good for time-series data |
| ORM | SQLModel | Combines Pydantic + SQLAlchemy |
| Containers | Docker | Easy deployment, reproducibility |
| CI/CD | GitHub Actions | Automatic builds on push |

## Future Architecture

```mermaid
graph LR
    subgraph "Multiple Greenhouses"
        GH1[Greenhouse 1]
        GH2[Greenhouse 2]
        GH3[Greenhouse N]
    end
    
    subgraph "Cloud"
        SUPA[(Supabase)]
        R2[Cloudflare R2]
        ML[ML Service]
    end
    
    GH1 --> SUPA
    GH2 --> SUPA
    GH3 --> SUPA
    GH1 --> R2
    GH2 --> R2
    GH3 --> R2
    SUPA --> ML
    R2 --> ML
```
