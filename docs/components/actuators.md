# Actuators

Actuators are drivers in the `greenthumb-rpi5` package. The driver class tracks what a load does. How it is wired is data on its `device_component` row: `interface` (relay or PWM line), `address` (the BCM pin) and `instance_config` (for example relay polarity). A relay line becomes an on/off output and a PWM line becomes a dimmable one, with no code change.

!!! note "v1 prototype"
    The v1 prototype is a single bench rig. Not every supported component is fitted, and probes and dosing pumps must be calibrated before automatic dosing is used.

!!! warning "Power"
    Mains and 12 V loads are switched through a relay board or MOSFET, never driven from a GPIO pin.

## Supported Actuators

| Class | Registry keys | Accepted fields |
|-------|---------------|-----------------|
| `SwitchedLoad` | `GROW_LIGHT_RELAY`, `EXHAUST_FAN`, `WATER_PUMP`, `AIR_PUMP` | `duration_s`; also `level` when wired to a PWM line. A load with no switched line (the USB air pump) accepts no fields. |
| `DosingPump` | `PERISTALTIC_PUMP` | As `SwitchedLoad`, plus `volume_ml` once the pump has a flow rate (`ml_per_s`) |
| `TwoPartDoser` | `TWO_PART_DOSER` | `volume_ml` only. A virtual actuator that doses two `DosingPump`s at a fixed ratio. |

Every command also carries `action` (`"on"` or `"off"`, required) and may carry `triggered_by`, `id_control_rule`, `reason` and `manual_override`. `GET /actuator/` lists each actuator with the fields its wiring accepts.

## Control

`POST /actuator/{id}/command`. On the Pi (Linux):

```bash
curl -X POST "http://localhost:8080/actuator/1/command" \
  -H "Content-Type: application/json" \
  -d '{"action": "on", "duration_s": 120}'
```

| Status | Meaning |
|--------|---------|
| 400 | A field is not supported by this actuator's wiring |
| 409 | The command was refused by a safety bound |
| 422 | Unknown field or invalid value |

`POST /actuator/{id}/release` hands an actuator under manual override back to the controller. See the [API Reference](../api/reference.md) for every route.

## Safety

- **Per-actuator bounds** on the `device_component` row: maximum on-time (`max_on_s`), cooldown (`min_off_s`), dose cap (`max_dose_ml`) and a rolling dose budget (`budget_ml` over `budget_window_s`). Automated commands meet every bound. Manual commands skip the cooldown and dose guards but are still clamped to `max_on_s`, unless they set `manual_override`.
- **Heartbeat watchdog:** the controller sends a heartbeat every cycle. If heartbeats stop, the API enters safety mode and turns every actuator off, except those flagged `fail_safe` (the air pump, which must keep the reservoir oxygenated). `PATCH /actuator/{id}/fail-safe` changes the flag.

## Adding a New Actuator

- **Another on/off load:** add its key to the `@register_component(...)` list on `SwitchedLoad` in `greenthumb_rpi5/actuator.py`, and add a `component_model` row with that `registry_key` in the `database` repository.
- **New behaviour:** subclass `Actuator`, implement `activate` and `deactivate`, and register it with `@register_component("MY_KEY")`.

Then add a `device_component` row for the node and restart the API. On the Pi (Linux):

```bash
cd deploy && make restart-api
```

## Hardware Access

The `api` container is given `/dev/i2c-1`, `/dev/video0` and the GPIO chip and runs privileged (see `rasp5/compose.yaml`).

### GPIO Permission Denied

On the Pi (Linux):

```bash
sudo usermod -aG gpio $USER
# Log out and back in
```
