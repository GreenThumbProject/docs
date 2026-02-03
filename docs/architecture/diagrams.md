# System Diagrams

Visual representations of the GreenThumb system architecture.

## Hardware Setup

```mermaid
graph TB
    subgraph "Raspberry Pi 5"
        RPI[Raspberry Pi 5]
        GPIO[GPIO/I2C]
        USB[USB Port]
    end
    
    subgraph "Sensors"
        AHT[AHT10<br/>Temp/Humidity]
        BMP[BMP280<br/>Pressure/Temp]
        TSL[TSL2561<br/>Light]
    end
    
    subgraph "Actuators"
        CAM[USB Camera]
        LED[RGB LED Strip]
        PUMP[Water Pump]
    end
    
    GPIO --> AHT
    GPIO --> BMP
    GPIO --> TSL
    USB --> CAM
    GPIO --> LED
    GPIO --> PUMP
```

## Docker Services

```mermaid
graph LR
    subgraph "Docker Network"
        DB[(PostgreSQL<br/>:5432)]
        API[microcontroller-api<br/>:8080]
        CTRL[Controller]
        WT[Watchtower]
    end
    
    subgraph "External"
        DH[Docker Hub]
        USER[User Browser]
    end
    
    CTRL -->|HTTP| API
    API --> DB
    USER --> API
    DH --> WT
    WT --> API
    WT --> CTRL
```

## Database Schema

```mermaid
erDiagram
    DEVICE ||--o{ DEVICE_SENSOR : has
    DEVICE ||--o{ DEVICE_ACTUATOR : has
    DEVICE ||--o{ CULTIVATION : hosts
    SENSOR_MODEL ||--o{ DEVICE_SENSOR : defines
    SENSOR_MODEL ||--o{ SENSOR_CAPABILITY : has
    ACTUATOR_MODEL ||--o{ DEVICE_ACTUATOR : defines
    VARIABLE ||--o{ SENSOR_CAPABILITY : measured_by
    VARIABLE ||--o{ MEASUREMENT : records
    DEVICE_SENSOR ||--o{ MEASUREMENT : produces
    PLANT_SPECIES ||--o{ CULTIVATION : grows
    CULTIVATION ||--o{ THRESHOLD : has
    UNIT ||--o{ VARIABLE : default_for
    
    DEVICE {
        int id_device PK
        string name
        string mac_address
        string location
        timestamp created_at
    }
    
    SENSOR_MODEL {
        int id_sensor_model PK
        string model_name
        string manufacturer
    }
    
    ACTUATOR_MODEL {
        int id_actuator_model PK
        string model_name
        string manufacturer
    }
    
    DEVICE_SENSOR {
        int id_device_sensor PK
        int id_device FK
        int id_sensor_model FK
        string port_address
        boolean is_active
    }
    
    DEVICE_ACTUATOR {
        int id_device_actuator PK
        int id_device FK
        int id_actuator_model FK
        json config
        boolean is_active
    }
    
    VARIABLE {
        int id_variable PK
        string name
        string description
        int default_unit_id FK
    }
    
    MEASUREMENT {
        int id_measurement PK
        timestamp collected_at
        float value
        int id_device_sensor FK
        int id_variable FK
    }
    
    THRESHOLD {
        int id_threshold PK
        int id_cultivation FK
        int id_variable FK
        float min_value
        float max_value
        float target_value
        int id_actuator_action FK
    }
    
    PLANT_SPECIES {
        int id_plant_species PK
        string name
        string scientific_name
    }
    
    CULTIVATION {
        int id_cultivation PK
        int id_device FK
        int id_plant_species FK
        timestamp start_date
        timestamp end_date
    }
```

## CI/CD Pipeline

```mermaid
graph LR
    subgraph "Developer"
        DEV[Push to main]
    end
    
    subgraph "GitHub Actions"
        BUILD[Build Docker Images]
        PUSH[Push to Docker Hub]
    end
    
    subgraph "Docker Hub"
        API_IMG[microcontroller-api:latest]
        CTRL_IMG[microcontroller-api-client:latest]
    end
    
    subgraph "Raspberry Pi"
        WT[Watchtower]
        CONTAINERS[Running Containers]
    end
    
    DEV --> BUILD
    BUILD --> PUSH
    PUSH --> API_IMG
    PUSH --> CTRL_IMG
    API_IMG --> WT
    CTRL_IMG --> WT
    WT --> CONTAINERS
```

## Control Loop (Sense-Think-Act)

```mermaid
sequenceDiagram
    participant CTRL as Controller
    participant API as microcontroller-api
    participant HW as Hardware
    participant DB as PostgreSQL
    
    Note over CTRL: Every 15 seconds
    
    rect rgb(200, 230, 200)
        Note over CTRL,API: SENSE
        CTRL->>API: GET /state/
        API->>HW: Read all sensors
        HW-->>API: Sensor values
        API->>DB: Load thresholds
        DB-->>API: Threshold data
        API-->>CTRL: SystemState
    end
    
    rect rgb(200, 200, 230)
        Note over CTRL: THINK
        Note over CTRL: Compare values vs thresholds
        Note over CTRL: Determine actions
    end
    
    rect rgb(230, 200, 200)
        Note over CTRL,HW: ACT
        CTRL->>API: POST /actuators/{id}/command
        API->>HW: Set actuator state
        HW-->>API: OK
        API-->>CTRL: OK
    end
    
    CTRL->>API: POST /state/heartbeat
```

## Future: Cloud Integration

```mermaid
graph TB
    subgraph "Greenhouse 1"
        RPI1[Raspberry Pi]
        DB1[(Local DB)]
    end
    
    subgraph "Greenhouse 2"
        RPI2[Raspberry Pi]
        DB2[(Local DB)]
    end
    
    subgraph "Cloud Services"
        SUPA[(Supabase<br/>PostgreSQL)]
        R2[Cloudflare R2<br/>Image Storage]
    end
    
    subgraph "Analytics"
        ML[ML Models]
        DASH[Dashboard]
    end
    
    RPI1 --> SUPA
    RPI1 --> R2
    RPI2 --> SUPA
    RPI2 --> R2
    SUPA --> ML
    R2 --> ML
    SUPA --> DASH
    ML --> DASH
```
