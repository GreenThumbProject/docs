# Microcontroller API Client

The `microcontroller-api-client` repository contains HTTP-based client scripts for greenhouse control.

## Overview

This repository houses scripts that communicate with the `microcontroller-api` via HTTP to implement control loops. The primary script is `controller`, which implements the Sense-Think-Act pattern.

## Repository Structure

```
microcontroller-api-client/
├── Dockerfile
├── main.py           # Controller script
├── requirements.txt
└── logs/             # Runtime logs
```

## Controller Script

The `controller` script implements a continuous Sense-Think-Act loop:

```mermaid
flowchart LR
    SENSE[1. SENSE<br/>GET /state/] --> THINK[2. THINK<br/>Evaluate thresholds]
    THINK --> ACT[3. ACT<br/>POST /actuators/{id}/command]
    ACT --> HEARTBEAT[4. HEARTBEAT<br/>POST /state/heartbeat]
    HEARTBEAT --> WAIT[Wait 15s]
    WAIT --> SENSE
```

### Sense

Fetches the complete system state from the API:

```python
response = await client.get("/state/")
state = response.json()
# Returns: {sensors, thresholds, safety_mode, timestamp}
```

### Think

Compares sensor values against cultivation thresholds:

```python
actions = evaluate_thresholds(state)
# Returns: [{id: actuator_id, payload: {...}, reason: "..."}]
```

Logic:
- Value < `min_value` → Activate actuator (duty_cycle: 100)
- Value > `max_value` → Deactivate actuator (duty_cycle: 0)

### Act

Executes actuator commands concurrently:

```python
for action in actions:
    await client.post(f"/state/actuators/{action['id']}/command", json=action['payload'])
```

### Heartbeat

Signals liveness to prevent safety mode:

```python
await client.post("/state/heartbeat")
```

## Configuration

Environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `API_BASE_URL` | `http://api:8080` | Base URL of the microcontroller API |
| `CONTROL_INTERVAL` | `15` | Seconds between control loop iterations |
| `PERSIST_INTERVAL` | `300` | Seconds between data persistence (future) |

## Running

### With Docker Compose

The controller runs automatically as part of the `rasp5` deployment:

```bash
cd rasp5
make up
```

### Standalone

```bash
cd microcontroller-api-client
pip install -r requirements.txt
export API_BASE_URL=http://localhost:8080
python main.py
```

### Logs

```bash
# Via Docker
make logs-ctrl

# Or directly
docker compose logs -f controller
```

## Safety Mode Integration

The controller's heartbeat prevents the API from entering safety mode:

1. Controller sends `POST /state/heartbeat` every cycle
2. API tracks `last_heartbeat` timestamp
3. If no heartbeat for configurable timeout → safety mode activates
4. Safety mode turns off all actuators
5. Next heartbeat resets safety mode

This ensures actuators don't run indefinitely if the controller crashes.

## Future: ML Agent Replacement

The architecture separates control logic from hardware access to enable:

1. **Current**: Rule-based threshold evaluation
2. **Future**: Replace `evaluate_thresholds()` with ML model inference
3. **Future**: Multiple control scripts for different strategies

The same API interface works for both rule-based and ML-based controllers.

## Adding New Control Scripts

Create new scripts in this repository that:

1. Fetch state from `/state/`
2. Implement custom decision logic
3. Command actuators via `/state/actuators/{id}/command`
4. Send heartbeats to maintain liveness

Example use cases:
- Time-based scheduling
- Multi-zone control
- ML-based optimization
- A/B testing different strategies
