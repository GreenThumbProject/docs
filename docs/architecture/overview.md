# Architecture Overview

GreenThumb uses a multi-repository, microservices architecture deployed on a Raspberry Pi 5.

## System Diagram

```mermaid
flowchart TB
    subgraph RPI["🍓 Raspberry Pi 5"]
        direction TB
        DC["📊 data_collection"]
        API["🌐 fastapi :8080"]
        DB[("🗄️ PostgreSQL")]
        CRON["⏰ cron"]
    end
    
    subgraph HW["🔌 Hardware"]
        CAM["📷 USB Camera"]
        SENSORS["🌡️ Sensors<br/>AHT10 · BMP280 · TSL2561"]
    end
    
    subgraph CLOUD["☁️ Cloud - Future"]
        SUPA[("Supabase")]
        R2["Cloudflare R2"]
    end
    
    CORE["📦 greenthumb-core"]
    
    %% Hardware to Services
    CAM --> DC
    SENSORS --> DC
    
    %% Internal service connections
    DC --> DB
    API --> DB
    CRON --> DB
    
    %% Library
    CORE -.-> DC
    CORE -.-> API
    
    %% Future cloud
    CRON -.-> SUPA
    CRON -.-> R2
```

## Remote Access

### Current Setup: Tailscale VPN

The Raspberry Pi runs [Tailscale](https://tailscale.com) to provide:

- **Always-on IP address** - Access the Pi from anywhere without port forwarding
- **Secure development** - SSH and dashboard access from any location
- **CI/CD integration** - Deploy updates remotely via GitHub Actions

See [Tailscale Setup](../getting-started/tailscale-setup.md) for configuration details.

### Future Vision

For end users, we plan to enable remote greenhouse access without requiring VPN knowledge:

- Client accesses their greenhouse via web dashboard
- System handles secure connectivity transparently
- No Tailscale or VPN configuration needed by the user

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
| Remote Access | Tailscale | Secure, easy VPN for development |

## Future Architecture

```mermaid
graph LR
    subgraph Greenhouses["Multiple Greenhouses"]
        GH1[Greenhouse 1]
        GH2[Greenhouse 2]
        GH3[Greenhouse N]
    end
    
    subgraph Cloud["Cloud"]
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

