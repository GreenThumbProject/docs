# Architecture Overview

GreenThumb uses a multi-repository, microservices architecture deployed on a Raspberry Pi 5, designed for horizontal scalability across multiple cultivation nodes.

## System Diagram

```mermaid
flowchart TB
    subgraph RPI["🍓 Raspberry Pi 5"]
        direction TB
        CTRL["🎮 controller"]
        API["🌐 microcontroller-api :8080"]
        DB[("🗄️ PostgreSQL")]
        CRON["⏰ cron"]
    end
    
    subgraph HW["🔌 Hardware"]
        CAM["📷 USB Camera"]
        SENSORS["🌡️ Sensors<br/>AHT10 · BMP280 · TSL2561"]
        ACTUATORS["💡 Actuators<br/>RGB LED · Water Pump"]
    end
    
    subgraph CLOUD["☁️ Cloud - Future"]
        SUPA[("Supabase")]
        R2["Cloudflare R2"]
    end
    
    CORE["📦 greenthumb-core"]
    
    %% Controller to API
    CTRL -->|HTTP /state| API
    
    %% API to Hardware
    API --> CAM
    API --> SENSORS
    API --> ACTUATORS
    
    %% Internal service connections
    API --> DB
    CRON --> DB
    
    %% Library
    CORE -.-> CTRL
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

### Microcontroller API

The `microcontroller-api` service is the central hub that:

- Manages all hardware (sensors and actuators)
- Provides REST API endpoints for data access
- Serves live video streaming from the camera
- Hosts the static dashboard
- Implements per-actuator locking for concurrent access

### Controller

The `controller` service (from `microcontroller-api-client` repo) implements the Sense-Think-Act loop:

- **Sense**: Fetches system state from `/state/` endpoint
- **Think**: Evaluates thresholds and determines actions
- **Act**: Commands actuators via `/state/actuators/{id}/command`
- **Heartbeat**: Signals liveness to prevent safety mode

This separation allows future replacement with ML agents.

### PostgreSQL

Local database storing:

- Device and sensor configurations
- Actuator definitions and states
- Plant species catalog
- Measurement history
- Cultivation records and thresholds

### Cron (Optional)

Scheduled tasks for:

- Cloud database sync (future)
- Local database cleanup
- Backup operations

## Data Flow

```mermaid
sequenceDiagram
    participant CTRL as Controller
    participant API as microcontroller-api
    participant HW as Hardware
    participant DB as PostgreSQL
    
    loop Every 15 seconds
        CTRL->>API: GET /state/
        API->>HW: Read sensors
        HW-->>API: Sensor values
        API->>DB: Load thresholds
        DB-->>API: Threshold data
        API-->>CTRL: {sensors, thresholds, safety_mode}
        
        Note over CTRL: Evaluate thresholds
        
        CTRL->>API: POST /state/actuators/{id}/command
        API->>HW: Set actuator state
        
        CTRL->>API: POST /state/heartbeat
    end
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
| Architecture | API-as-Device-Manager | Centralized hardware control, future ML agent support |

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

