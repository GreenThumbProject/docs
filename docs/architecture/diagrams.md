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
    
    subgraph "Peripherals"
        CAM[USB Camera]
        LED[LED Strip<br/>- Future -]
        PUMP[Water Pump<br/>- Future -]
    end
    
    GPIO --> AHT
    GPIO --> BMP
    GPIO --> TSL
    USB --> CAM
    GPIO -.-> LED
    GPIO -.-> PUMP
```

## Docker Services

```mermaid
graph LR
    subgraph "Docker Network"
        DB[(PostgreSQL<br/>:5432)]
        API[FastAPI<br/>:8080]
        DC[Data Collection]
        WT[Watchtower]
    end
    
    subgraph "External"
        DH[Docker Hub]
        USER[User Browser]
    end
    
    DC --> DB
    API --> DB
    USER --> API
    DH --> WT
    WT --> API
    WT --> DC
```

## Database Schema

```mermaid
erDiagram
    DEVICE ||--o{ DEVICE_SENSOR : has
    DEVICE ||--o{ CULTIVATION : hosts
    SENSOR_MODEL ||--o{ DEVICE_SENSOR : defines
    SENSOR_MODEL ||--o{ SENSOR_CAPABILITY : has
    VARIABLE ||--o{ SENSOR_CAPABILITY : measured_by
    VARIABLE ||--o{ MEASUREMENT : records
    DEVICE_SENSOR ||--o{ MEASUREMENT : produces
    PLANT_SPECIES ||--o{ CULTIVATION : grows
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
    
    DEVICE_SENSOR {
        int id_device_sensor PK
        int id_device FK
        int id_sensor_model FK
        string port_address
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
        API_IMG[greenthumb-api:latest]
        DC_IMG[greenthumb-data-collection:latest]
    end
    
    subgraph "Raspberry Pi"
        WT[Watchtower]
        CONTAINERS[Running Containers]
    end
    
    DEV --> BUILD
    BUILD --> PUSH
    PUSH --> API_IMG
    PUSH --> DC_IMG
    API_IMG --> WT
    DC_IMG --> WT
    WT --> CONTAINERS
```

## Data Collection Flow

```mermaid
sequenceDiagram
    participant S as Sensors
    participant DC as data_collection
    participant DB as PostgreSQL
    participant LOG as Log File
    
    Note over DC: Every 30 minutes
    DC->>S: Read AHT10
    S-->>DC: Temperature, Humidity
    DC->>S: Read BMP280
    S-->>DC: Pressure, Temperature
    DC->>S: Read TSL2561
    S-->>DC: Light Intensity
    DC->>DB: INSERT measurements
    DC->>LOG: Append log entry
    
    Note over DC: Every 4 hours
    DC->>S: Capture Photo
    S-->>DC: Image data
    DC->>LOG: Append photo entry
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
