# Python Packages

GreenThumb's shared Python code is two packages:

| Package | Repository | Used by |
|---------|------------|---------|
| `greenthumb` | `greenthumb-models` | The node API (`microcontroller-api`) and the cloud API (`greenthumb-api`) |
| `greenthumb_rpi5` | `greenthumb-rpi5` | The node API only. Hardware drivers, no database session. |

`greenthumb_rpi5` depends on `greenthumb`. The controller (`microcontroller-api-client`) installs neither: it talks to the node API over HTTP.

---

## `greenthumb`

| Module | Contents |
|--------|----------|
| `greenthumb.models` | SQLModel tables and their Create/Read/Update schemas, split by tier (`tier0` users and properties, `tier1` catalog, `tier2` device configuration and cultivations, `tier3` operational data, `pi_only`, `enums`). Columns that exist on only one tier are selected by the `IS_CLOUD` environment variable. |
| `greenthumb.sync` | The Pi ↔ cloud sync: `schemas` (DTOs such as `DeviceConfig` and `ComponentConfig`), `payload` (builds the sync payload from the cloud database), `apply` (applies a payload to a session), `contract` (the sync wire contract version) |
| `greenthumb.crud` | Database helpers (below) |
| `greenthumb.api` | `make_crud_router` (below) |
| `greenthumb.db` | The engine and the `get_session` dependency |
| `greenthumb.config` | Typed service configuration |
| `greenthumb.errors` | `NotFoundError`, `ValidationError` and the transaction decorators |
| `greenthumb.error_handlers` | `install_error_handlers` (below) |
| `greenthumb.templates` | Default JSONB skeletons for `component_model.specs` and `device_component.instance_config` |

`greenthumb.api` and `greenthumb.error_handlers` need FastAPI, which comes with the `api` extra.

### `greenthumb.crud`

Use these helpers instead of raw SQLAlchemy or hand-rolled CRUD routes.

```python
from greenthumb.crud import get, get_by_id, save, update, delete, upsert
from greenthumb.db import get_session
```

| Function | Signature | Use for |
|----------|-----------|---------|
| `get` | `get(session, Model, join=None, filters=None, distinct=False, order_by=None, one_or_none=False, limit=None)` | SELECT with optional filter dict, ordering, and limit |
| `get_by_id` | `get_by_id(session, Model, id)` | Fetch a single row by primary key (tuple for composite PKs) |
| `save` | `save(session, obj_or_list)` | INSERT one or many model instances; returns the saved object(s) |
| `update` | `update(session, Model, id, vals)` | UPDATE a row by PK from a Pydantic/SQLModel payload; raises `NotFoundError` if missing |
| `delete` | `delete(session, Model, id)` | DELETE a row by PK; raises `NotFoundError` if missing |
| `upsert` | `upsert(session, Model, data, *, policy="always", pk=None, filters=None, allow=None, preserve=None, flush=False)` | Insert-or-update under a write policy (below) |

!!! warning "Pass `filters=` by keyword"
    `join` is the third positional argument of `get`, not `filters`. A filter dict passed positionally is read as a join and fails with SQLAlchemy's *"Join target ... expected, got 'somefield'"*, which does not look like an argument-order mistake.

Filter dict syntax: `{"field": value}` for equality, `{"field__gt": value}` for comparisons (`lt`, `lte`, `gt`, `gte`, `ne`, `in`, `contains`, `like`).

`upsert` write policies:

| Policy | Behaviour | Shortcut |
|--------|-----------|----------|
| `"always"` | Incoming data wins | `upsert_simple` |
| `"absent"` | Insert only; an existing row is left alone | `insert_if_missing` |
| `"newer"` | Last write wins, by `updated_at` | `upsert_lww` |

### `greenthumb.api`

`make_crud_router(Model, CreateSchema, ReadSchema, UpdateSchema, prefix, tags=None)` returns an `APIRouter` with the standard routes and handles commits and errors.

```python
from greenthumb.api import make_crud_router
from greenthumb.models import PlantSpecies, PlantSpeciesCreate, PlantSpeciesRead, PlantSpeciesUpdate

router.include_router(
    make_crud_router(
        PlantSpecies, PlantSpeciesCreate, PlantSpeciesRead, PlantSpeciesUpdate,
        "/plant-species", tags=["Plant Species"],
    )
)

# GET    /plant-species         list
# GET    /plant-species/{id}    get one
# POST   /plant-species         create
# PUT    /plant-species/{id}    update
# DELETE /plant-species/{id}    delete
```

Models with a composite primary key get `GET` and `DELETE` on `/{pk1}/{pk2}`. Override a route only when it needs custom validation.

### `greenthumb.error_handlers`

The app must call `greenthumb.error_handlers.install_error_handlers(app)` once, or a missing row returns 500 instead of 404.

```python
from fastapi import FastAPI
from greenthumb.error_handlers import install_error_handlers

app = FastAPI()
install_error_handlers(app)
```

---

## `greenthumb_rpi5`

| Module | Contents |
|--------|----------|
| `component` | The `Component`, `Sensor` and `Actuator` base classes and the driver registry (`@register_component`) |
| `sensor` | Sensor drivers (see [Sensors](sensors.md)) |
| `actuator` | Actuator drivers (see [Actuators](actuators.md)) |
| `bus` | Shared hardware handles (the I2C bus, ADS1115 chips) and the GPIO outputs |
| `device` | `DeviceManager`: builds every driver from a `DeviceConfig`, reads sensors, runs actuator commands, takes the controller heartbeat and enters safety mode |
| `schedule` | Time-window maths for schedule rules |
