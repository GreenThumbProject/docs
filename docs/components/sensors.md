# Sensors

Sensors are drivers in the `greenthumb-rpi5` package. The node picks the driver for each wired part at runtime, from the `registry_key` of its catalog row (`component_model`).

!!! note "v1 prototype"
    The v1 prototype is a single bench rig. Not every supported component is fitted, and probes and dosing pumps must be calibrated before automatic dosing is used.

!!! tip "Looking for Actuators?"
    For lights, fans and pumps, see the [Actuators](actuators.md) page.

## Supported Sensors

| Component | Registry key | Interface | Measures |
|-----------|--------------|-----------|----------|
| TSL2561 | `TSL2561` | I2C | Light intensity, broadband, infrared |
| BMP280 | `BMP280` | I2C | Temperature, pressure |
| AHT10 | `AHT10` | I2C | Temperature, humidity |
| DS18B20 | `DS18B20` | 1-Wire | Water temperature |
| Float switch | `FLOAT_SWITCH` | GPIO | Reservoir level (on/off) |
| pH probe | `PH_PROBE` | Analog, through an ADS1115 | pH (plus the raw probe voltage) |
| TDS probe | `TDS_PROBE` | Analog, through an ADS1115 | Total dissolved solids, electrical conductivity (plus the raw probe voltage) |
| USB webcam | `WEBCAM_REDRAGON_HITMAN` | USB | Photos (no measurements) |

What each model measures is data: one `component_capability` row per variable in the catalog seed of the `database` repository.

## Wiring Is Data

How a part is wired to a node is a `device_component` row, not code: its `interface`, its `address` (I2C address, BCM pin or camera index), its parent component (an ADS1115 for the analog probes) and its `instance_config`. Rewiring a part means editing that row; the driver stays the same.

## Calibration

The pH and TDS probes convert volts to a reading with the coefficients from their newest `calibration_log` row. An uncalibrated probe reports no value. The raw probe voltage is still recorded, so readings can be recomputed once a calibration exists.

## Enable the Interfaces

On the Pi (Linux):

```bash
sudo raspi-config
# Interface Options > I2C > Enable
# Interface Options > 1-Wire > Enable (for the DS18B20)
sudo reboot
```

Check that the I2C sensors answer:

```bash
i2cdetect -y 1
```

The `api` container is given `/dev/i2c-1`, `/dev/video0` and the GPIO chip and runs privileged (see `rasp5/compose.yaml`).

## Adding a New Sensor

1. **Write the driver.** In `greenthumb_rpi5/sensor.py`, subclass `Sensor`, implement `_init_hardware` and `read_data`, and register it under a new key:

    ```python
    from greenthumb_rpi5.component import Sensor, register_component

    @register_component("MY_KEY")
    class MySensor(Sensor):
        def _init_hardware(self):
            # Open the hardware. Shared handles such as the I2C bus come from self.bus.
            ...

        def read_data(self, **_) -> dict:
            # One key per capability: the variable name, lower case, spaces as underscores.
            return {"temperature": ...}
    ```

2. **Add it to the catalog.** In the `database` repository, add a `component_model` row with `registry_key = 'MY_KEY'` and one `component_capability` row per variable it measures.
3. **Wire it.** Add a `device_component` row for the node.
4. **Restart the API.** On the Pi (Linux):

    ```bash
    cd deploy && make restart-api
    ```

## Troubleshooting

### Sensor Not Detected

1. Check the wiring and power (3.3V, not 5V, for the I2C sensors).
2. Check that I2C is enabled: `ls /dev/i2c*`.
3. Check the `address` on the part's `device_component` row.

### Permission Denied

On the Pi (Linux):

```bash
sudo usermod -aG i2c $USER
# Log out and back in
```
