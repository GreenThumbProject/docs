# greenthumb-core

The shared Python library containing database models, utilities, and hardware interfaces.

## Overview

`greenthumb-core` is a private Python package that provides:

- SQLModel database definitions
- Database engine and session management
- Raspberry Pi 5 hardware interfaces
- Sensor drivers (AHT10, BMP280, TSL2561)
- Camera utilities

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
    ├── camera.py       # USB camera interface
    ├── device.py       # DeviceManager class
    └── sensors.py      # I2C sensor drivers
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

## Models

All database tables are defined as SQLModel classes:

```python
from greenthumb_core.models import (
    Device,
    SensorModel,
    DeviceSensor,
    Variable,
    Measurement,
    PlantSpecies,
    Cultivation,
    Unit
)
```

### Key Relationships

- `Device` has many `DeviceSensor`
- `DeviceSensor` belongs to `Device` and `SensorModel`
- `Measurement` belongs to `DeviceSensor` and `Variable`
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

### DeviceManager

High-level interface for managing a greenhouse device:

```python
from greenthumb_core.rpi5 import DeviceManager, Camera
from sqlmodel import Session

camera = Camera('/dev/video0', 1280, 720)
device = DeviceManager(id=1, camera=camera)

with Session(engine) as session:
    device.init(session)  # Load sensors from DB
    data = device.collect_data(session)  # Read all sensors
```

## Configuration

The library uses environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | - | PostgreSQL connection string |

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
