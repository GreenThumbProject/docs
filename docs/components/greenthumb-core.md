# greenthumb-core

The shared Python library containing database models, utilities, hardware interfaces, and API utilities.

## Overview

`greenthumb-core` is a private Python package that provides:

- SQLModel database definitions
- Database engine and session management
- Raspberry Pi 5 hardware interfaces
- Sensor drivers (AHT10, BMP280, TSL2561)
- Actuator drivers (RGB LED, Water Pump)
- Camera utilities
- FastAPI CRUD router generation

## Installation

```bash
# With GitHub PAT
pip install git+https://${GH_PAT}@github.com/GreenThumbProject/greenthumb-core.git

# With Raspberry Pi hardware support
pip install "git+https://${GH_PAT}@github.com/GreenThumbProject/greenthumb-core.git[rpi5]"
```

## Package Structure

```
greenthumb_core/
├── api/                # FastAPI utilities
│   └── crud.py         # CRUD router generation
│
├── db/                 # Database utilities
│   ├── __init__.py
│   ├── engine.py       # SQLModel engine setup
│   └── lifespan.py     # FastAPI lifespan context
│
├── models/             # SQLModel definitions
│   ├── __init__.py
│   └── models.py       # All table definitions
│
└── rpi5/               # Raspberry Pi 5 hardware
    ├── __init__.py
    ├── actuator.py     # Actuator base class + registry
    ├── camera.py       # USB camera interface
    ├── camera_manager.py # Camera lifecycle management
    ├── device.py       # DeviceManager class
    └── sensor.py       # Sensor base class + registry
```

## Database Utilities

### Engine Setup

```python
from greenthumb_core.db import get_engine, create_db_and_tables

engine = get_engine(echo=False)
create_db_and_tables()
```

### FastAPI Integration

```python
from greenthumb_core.db import lifespan, get_session
from fastapi import FastAPI, Depends
from sqlmodel import Session

app = FastAPI(lifespan=lifespan)

@app.get("/data")
def get_data(session: Session = Depends(get_session)):
    # session is automatically managed
    pass
```

## API Utilities

### CRUD Router Generation

Automatically generate full CRUD endpoints for any SQLModel:

```python
from greenthumb_core.api import make_crud_router
from greenthumb_core.models import Device, DeviceCreate, DeviceRead, DeviceUpdate

device_router = make_crud_router(
    Device, 
    DeviceCreate, 
    DeviceRead, 
    DeviceUpdate, 
    "/device", 
    tags=["Device"]
)

app.include_router(device_router)
```

This creates:
- `GET /device/` - List all
- `GET /device/{id}` - Get by ID
- `POST /device/` - Create new
- `PUT /device/{id}` - Update
- `DELETE /device/{id}` - Delete

## Models

All database tables are defined as SQLModel classes:

```python
from greenthumb_core.models import (
    Device,
    SensorModel,
    DeviceSensor,
    ActuatorModel,
    DeviceActuator,
    Variable,
    Measurement,
    PlantSpecies,
    Cultivation,
    Threshold,
    Unit
)
```

### Key Relationships

- `Device` has many `DeviceSensor` and `DeviceActuator`
- `DeviceSensor` belongs to `Device` and `SensorModel`
- `DeviceActuator` belongs to `Device` and `ActuatorModel`
- `Measurement` belongs to `DeviceSensor` and `Variable`
- `Cultivation` has many `Threshold`
- `Variable` has a `default_unit`

## Hardware Interfaces

### Camera

```python
from greenthumb_core.rpi5 import Camera

camera = Camera('/dev/video0', width=1280, height=720)
frame = camera.capture()  # Returns numpy array
camera.save('photo.jpg')
```

### Sensors

```python
from greenthumb_core.rpi5 import AHT10, BMP280, TSL2561

aht = AHT10()
temp, humidity = aht.read()

bmp = BMP280()
pressure, temp2 = bmp.read()

tsl = TSL2561()
lux = tsl.read()
```

### Actuators

```python
from greenthumb_core.rpi5 import RGBLED, WaterPump

# RGB LED with PWM
led = RGBLED(config={"pins": {"r": 17, "g": 27, "b": 22}})
led.command({"r": 255, "g": 128, "b": 0})

# Water Pump with PWM
pump = WaterPump(config={"pin": 18})
pump.command({"duty_cycle": 100})
```

### DeviceManager

High-level interface for managing a greenhouse device:

```python
from greenthumb_core.rpi5 import DeviceManager, get_device_manager
from sqlmodel import Session

# Used with FastAPI Depends
dm = get_device_manager()

with Session(engine) as session:
    dm.init(session)  # Load sensors and actuators from DB
    data = dm.collect_data(session)  # Read all sensors
    await dm.command_actuator(actuator_id, {"duty_cycle": 100})
    dm.heartbeat()  # Reset safety mode timer
```

## Configuration

The library uses environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | - | PostgreSQL connection string |
| `DEVICE_ID` | 1 | Device ID to load from database |

## Dependencies

Core:
- `sqlmodel>=0.0.27`
- `sqlalchemy>=2.0.0`
- `psycopg2-binary>=2.9.0`

Raspberry Pi (`[rpi5]`):
- `adafruit-blinka`
- `adafruit-circuitpython-ahtx0`
- `adafruit-circuitpython-bmp280`
- `adafruit-circuitpython-tsl2561`
- `opencv-python-headless`
- `rpi-lgpio`
