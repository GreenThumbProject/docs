# Sensors

GreenThumb uses I2C sensors connected to the Raspberry Pi 5 for environmental monitoring.

!!! tip "Looking for Actuators?"
    For output devices like LEDs and pumps, see the [Actuators](actuators.md) page.

## Supported Sensors

| Sensor | Address | Measurements |
|--------|---------|--------------|
| AHT10 | 0x38 | Temperature, Humidity |
| BMP280 | 0x76 | Pressure, Temperature |
| TSL2561 | 0x39 | Light Intensity |

## Hardware Connection

All sensors connect to the I2C1 bus:

| Pin | Function |
|-----|----------|
| Pin 3 | SDA (Data) |
| Pin 5 | SCL (Clock) |
| Pin 1 | 3.3V Power |
| Pin 6 | Ground |

```
Raspberry Pi 5         Sensors
    ┌─────┐           ┌─────────┐
    │ 3.3V├───────────┤ VCC     │
    │ GND ├───────────┤ GND     │
    │ SDA ├───────────┤ SDA     │
    │ SCL ├───────────┤ SCL     │
    └─────┘           └─────────┘
```

## Enable I2C

```bash
sudo raspi-config
# Interface Options > I2C > Enable
sudo reboot
```

Verify sensors are detected:

```bash
i2cdetect -y 1
```

Expected output:

```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- 38 39 -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- 76 --                         
```

## Sensor Details

### AHT10

**Address**: 0x38

**Measurements**:
- Temperature: -40°C to 85°C (±0.3°C accuracy)
- Humidity: 0-100% RH (±2% accuracy)

**Usage**:

```python
from greenthumb_core.rpi5 import AHT10

sensor = AHT10()
temperature, humidity = sensor.read()
print(f"Temperature: {temperature:.1f}°C")
print(f"Humidity: {humidity:.1f}%")
```

### BMP280

**Address**: 0x76

**Measurements**:
- Pressure: 300-1100 hPa (±1 hPa accuracy)
- Temperature: -40°C to 85°C (±1°C accuracy)

**Usage**:

```python
from greenthumb_core.rpi5 import BMP280

sensor = BMP280()
pressure, temperature = sensor.read()
print(f"Pressure: {pressure:.1f} hPa")
print(f"Temperature: {temperature:.1f}°C")
```

### TSL2561

**Address**: 0x39

**Measurements**:
- Light: 0.1 to 40,000 lux

**Usage**:

```python
from greenthumb_core.rpi5 import TSL2561

sensor = TSL2561()
lux = sensor.read()
print(f"Light: {lux} lux")
```

## Database Configuration

Sensors are registered in the database:

```sql
-- Sensor models
INSERT INTO sensor_model (model_name, manufacturer) VALUES
    ('AHT10', 'Aosong'),
    ('BMP280', 'Bosch'),
    ('TSL2561', 'TAOS');

-- Sensor capabilities
INSERT INTO sensor_capability (id_sensor_model, id_variable) VALUES
    (1, 1), -- AHT10 -> Temperature
    (1, 2), -- AHT10 -> Humidity
    (2, 3), -- BMP280 -> Pressure
    (2, 1), -- BMP280 -> Temperature
    (3, 4); -- TSL2561 -> Light
```

## Planned Sensors

| Sensor | Measurements | Status |
|--------|--------------|--------|
| pH Sensor | Solution pH | Planned |
| EC Sensor | Electrical Conductivity | Planned |
| CO2 Sensor | Carbon Dioxide | Future |
| O2 Sensor | Oxygen | Future |

## Adding New Sensors

Adding a new sensor to the GreenThumb system requires three steps:

### 1. Connect the Hardware

Connect the sensor to the Raspberry Pi's I2C bus (SDA/SCL pins) or other appropriate interface.

### 2. Register in Database

Add the sensor model to the database:

```sql
-- Register the sensor model
INSERT INTO sensor_model (model_name, manufacturer) VALUES
    ('NewSensorModel', 'Manufacturer');

-- Define what the sensor measures
INSERT INTO sensor_capability (id_sensor_model, id_variable) VALUES
    ((SELECT id_sensor_model FROM sensor_model WHERE model_name = 'NewSensorModel'),
     (SELECT id_variable FROM variable WHERE name = 'your_variable'));
```

### 3. Create Sensor Class

Create a Python class in `greenthumb-core` that extends the `Sensor` base class and register it:

```python
from dataclasses import dataclass
from greenthumb_core.rpi5.sensor import Sensor, register_sensor

@register_sensor("NewSensorModel")  # Name must match database model_name
@dataclass
class NewSensorModel(Sensor):
    """Driver for NewSensorModel sensor."""
    
    def init(self, session):
        """Initialize sensor hardware and capabilities."""
        # Initialize sensor-specific hardware
        self.obj = some_library.SensorDriver(self.i2c, address=self.address)
        return super().init(session)
    
    def read_data(self) -> dict:
        """Read sensor values and return as dict."""
        return {
            "measurement_name": self.obj.read_value(),
        }
```

!!! warning "Registration Required"
    The `@register_sensor("NewSensorModel")` decorator is **required** to register the sensor in the `SENSOR_REGISTRY`. The name must **exactly** match the `model_name` in the `sensor_model` database table.

### 4. Restart Data Collection

After adding the sensor, restart the data collection container:

```bash
docker compose restart data_collection
```

The system will automatically detect and start using the new sensor.

!!! tip "Actuators (Coming Soon)"
    The same pattern will be used for actuators, allowing easy addition of new output devices like pumps, LEDs, and valves.

## Troubleshooting

### Sensor Not Detected

1. Check wiring connections
2. Verify power (3.3V, not 5V!)
3. Check I2C is enabled: `ls /dev/i2c*`
4. Try different I2C address if sensor has options

### Permission Denied

```bash
# Add user to i2c group
sudo usermod -aG i2c $USER
# Re-login required
```

### In Docker

Containers need device access:

```yaml
devices:
  - "/dev/i2c-1:/dev/i2c-1"
```
