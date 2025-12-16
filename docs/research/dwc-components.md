# DWC Greenhouse Components

List of components for the physical Deep Water Culture (DWC) greenhouse.

!!! info "Work in Progress"
    Specific models will be added as components are acquired and tested.

## Controller

| Component | Model | Notes |
|-----------|-------|-------|
| Main Controller | Raspberry Pi 5 (4GB+) | Runs all services in Docker |

## Hydroponic System

| Component | Model | Notes |
|-----------|-------|-------|
| System Type | DWC (Deep Water Culture) | Roots submerged in nutrient solution |
| Reservoir | TBD | Holds nutrient solution |
| Net Pots | TBD | Holds plants above solution |
| Growing Medium | TBD | Clay pebbles, rockwool, etc. |

## Aeration

| Component | Model | Notes |
|-----------|-------|-------|
| Air Pump | TBD | Provides oxygen to roots |
| Air Stones | TBD | Distributes air bubbles |
| Air Tubing | TBD | Connects pump to stones |

## Sensors

| Category | Sensor Type | Model | Measurements |
|----------|-------------|-------|--------------|
| Environment | Temperature/Humidity | AHT10 | Temp, RH |
| Environment | Pressure | BMP280 | Atmospheric pressure |
| Environment | Light | TSL2561 | Lux |
| Solution | pH Sensor | TBD | Solution acidity |
| Solution | EC Sensor | TBD | Electrical conductivity |
| Solution | Water Temperature | TBD | Solution temperature |

## Actuators

| Category | Component | Model | Control |
|----------|-----------|-------|---------|
| Lighting | Full-spectrum LEDs | TBD | PWM via GPIO |
| Aeration | Air Pump | TBD | On/Off relay |
| Circulation | Water Pump | TBD | PWM via GPIO |

## Camera

| Component | Model | Notes |
|-----------|-------|-------|
| Camera | USB Webcam | Current development setup |
| Future | TBD | Higher resolution for CV |

## Structure

| Component | Model | Notes |
|-----------|-------|-------|
| Greenhouse Frame | TBD | Mini-greenhouse enclosure |
| Lighting Mount | TBD | LED panel mounting |

---

## Adding Components

To add a new component:

1. Update this table with the component details
2. For sensors: Follow the [Adding New Sensors](../components/sensors.md#adding-new-sensors) guide
3. For actuators: Similar process (documentation coming soon)
