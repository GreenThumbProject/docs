# Actuators

GreenThumb uses GPIO-controlled actuators connected to the Raspberry Pi 5 for environmental control.

## Supported Actuators

| Actuator | Control | Purpose |
|----------|---------|---------|
| RGB LED | PWM (3 channels) | Grow lighting |
| Water Pump | PWM | Irrigation |
| Camera | USB | Monitoring (treated as actuator for control) |

## Hardware Connection

### RGB LED Strip

Connect to GPIO pins with PWM capability:

| Wire | GPIO Pin | Function |
|------|----------|----------|
| Red | GPIO 17 | Red channel PWM |
| Green | GPIO 27 | Green channel PWM |
| Blue | GPIO 22 | Blue channel PWM |
| Ground | GND | Common ground |

!!! warning "Power"
    LED strips typically need external 12V power supply. Use a transistor or MOSFET to switch the LED power with the Pi's 3.3V GPIO.

### Water Pump

| Wire | GPIO Pin | Function |
|------|----------|----------|
| Control | GPIO 18 | PWM signal |
| Ground | GND | Common ground |

!!! warning "Power"
    Pumps require external power. Use a relay or MOSFET controlled by the GPIO pin.

## Database Configuration

Actuators are registered in the database:

```sql
-- Actuator models
INSERT INTO actuator_model (model_name, manufacturer) VALUES
    ('RGBLED', 'Generic'),
    ('WaterPump', 'Generic'),
    ('Camera', 'Generic');

-- Device actuators (example)
INSERT INTO device_actuator (id_device, id_actuator_model, config, is_active) VALUES
    (1, 1, '{"pins": {"r": 17, "g": 27, "b": 22}}', true),
    (1, 2, '{"pin": 18}', true);
```

## API Control

Actuators are controlled via the `/state` API endpoints:

### Set RGB LED Color

```bash
curl -X POST "http://localhost:8080/state/actuators/1/command" \
  -H "Content-Type: application/json" \
  -d '{"r": 255, "g": 128, "b": 0}'
```

### Set Pump Duty Cycle

```bash
curl -X POST "http://localhost:8080/state/actuators/2/command" \
  -H "Content-Type: application/json" \
  -d '{"duty_cycle": 100}'
```

## Python Usage

```python
from greenthumb_core.rpi5 import RGBLED, WaterPump

# RGB LED
led = RGBLED(config={"pins": {"r": 17, "g": 27, "b": 22}})
led.init()
led.command({"r": 255, "g": 0, "b": 255})  # Purple
led.off()

# Water Pump
pump = WaterPump(config={"pin": 18})
pump.init()
pump.command({"duty_cycle": 50})  # 50% power
pump.off()
```

## Adding New Actuators

Adding a new actuator type requires three steps:

### 1. Connect the Hardware

Connect the actuator to appropriate GPIO pins with proper power management.

### 2. Register in Database

```sql
-- Register the actuator model
INSERT INTO actuator_model (model_name, manufacturer) VALUES
    ('NewActuator', 'Manufacturer');

-- Add to device
INSERT INTO device_actuator (id_device, id_actuator_model, config, is_active) VALUES
    (1, (SELECT id_actuator_model FROM actuator_model WHERE model_name = 'NewActuator'),
     '{"pin": 23}', true);
```

### 3. Create Actuator Class

Create a Python class in `greenthumb-core` that extends the `Actuator` base class:

```python
from dataclasses import dataclass
from greenthumb_core.rpi5.actuator import Actuator, register_actuator

@register_actuator("NewActuator")  # Must match database model_name
@dataclass
class NewActuator(Actuator):
    """Driver for new actuator type."""
    
    async def init(self):
        """Initialize actuator hardware."""
        pin = self.config.get("pin", 23)
        # Initialize GPIO, PWM, etc.
        
    async def command(self, payload: dict):
        """Execute command on actuator."""
        # Implement control logic
        
    async def off(self):
        """Turn off actuator (safety state)."""
        # Implement safe shutdown
```

!!! warning "Registration Required"
    The `@register_actuator("NewActuator")` decorator is **required** to register the actuator in the `ACTUATOR_REGISTRY`. The name must **exactly** match the `model_name` in the `actuator_model` database table.

### 4. Restart API

```bash
make restart-api
```

## Safety Mode

When the controller stops sending heartbeats, the API enters **safety mode**:

- All actuators are turned off using their `off()` method
- Prevents uncontrolled operation if controller crashes
- Can be reset by controller heartbeat

## Troubleshooting

### GPIO Permission Denied

```bash
# Add user to gpio group
sudo usermod -aG gpio $USER
# Re-login required
```

### In Docker

Containers need device access:

```yaml
devices:
  - "/dev/gpiochip0:/dev/gpiochip0"
  - "/dev/gpiochip4:/dev/gpiochip4"
privileged: true
```
