# Database

GreenThumb runs two kinds of PostgreSQL database: one local database on each Raspberry Pi node, and one cloud database for the whole fleet.

| Tier | Engine | Purpose |
|------|--------|---------|
| **Pi** | PostgreSQL 17, in the node's Compose stack | Readings, actuator and rule logs, cultivations and control rules, photo metadata, and a copy of the catalog the node needs |
| **Cloud** | PostgreSQL 17 + TimescaleDB, self-hosted, reachable only over the VPN | Fleet history, users, device registry, the shared catalog |

The Pi does not need the cloud to run. It works against its local database and syncs with the cloud whenever it can reach it.

## Source of truth

The schema is defined in SQL in the `database` repository:

| File | Contents |
|------|----------|
| `schemas/base/01_schema.sql` | Shared base: enums, every shared table, indexes, triggers. Both tiers run it first. |
| `schemas/rasp/02_schema.sql` | Pi overlay: sync flags and the `sync_metadata` table |
| `schemas/cloud/02_schema.sql` | Cloud overlay: user credentials and roles, fleet columns on `device`, TimescaleDB hypertables, `cloud_meta`, `system_alert` |
| `schemas/cloud/02_seed.sql` | Catalog seed (cloud only) |

`make schema-sync` copies these files into `rasp5/db/` and `cloud/db/`, which each Postgres container mounts into `/docker-entrypoint-initdb.d/`. Never edit those copies. On a Windows workstation, from `rasp5/` or `cloud/`:

```powershell
cmd /c make schema-sync
```

The Pi gets no seed file: it fills its catalog from the cloud on the first sync.

The Python models in `greenthumb.models` (see [Python Packages](greenthumb-models.md)) mirror the SQL. Their `create_all()` call at startup is only a safety net; the SQL files create the schema.

## Tables

The shared base defines 25 tables, used by both tiers:

`app_user`, `property`, `unit`, `variable`, `plant_species`, `growth_phase`, `species_rule`, `device_model`, `component_model`, `component_capability`, `component_model_effect`, `substance`, `substance_effect`, `device`, `container`, `device_component`, `calibration_log`, `cultivation`, `actuator_effect`, `control_rule`, `control_rule_log`, `measurement`, `cultivation_phase`, `photo`, `actuator_log`

| Tier | Extra tables | Total |
|------|--------------|-------|
| Pi | `sync_metadata` | 26 |
| Cloud | `cloud_meta`, `system_alert` | 27 |

On the cloud, `measurement`, `actuator_log` and `control_rule_log` are TimescaleDB hypertables, and the `actuator_log_daily` continuous aggregate feeds the admin dashboard.

## Sync state on the Pi

The Pi overlay adds columns that track what still has to reach the cloud:

| Column | Tables | Meaning |
|--------|--------|---------|
| `is_synced` | `measurement`, `photo`, `actuator_log`, `control_rule_log`, `calibration_log` | `FALSE` until the row has been pushed |
| `is_dirty` | `device`, `cultivation`, `cultivation_phase`, `control_rule` | `TRUE` while a local edit has not been pushed |
| `pending_init` | `device_component` | Added while the Pi was running; its hardware is set up only after a restart or rescan |

The `sync_metadata` table is a key-value store. Its keys are `last_config_sync`, `last_sensor_persist`, `last_data_push`, `last_cloud_seen` and `last_epoch`.

## Migrations

The schema files run only once, against an empty database. Changes to a live database go in a new dated file, `database/migrations/YYYY-MM-DD_topic.sql`, applied by hand with `psql`.

On the cloud VM (Linux), from `cloud/deploy/`:

```bash
docker compose exec -T db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -v ON_ERROR_STOP=1' < ../../database/migrations/<file>.sql
```

## Accessing the database

On the Pi (Linux), from `deploy/`:

```bash
make db-shell   # psql shell inside the db container
```

## Device token

`device.device_token` is the secret a Pi uses for cloud sync. It is:

- **Generated** by the cloud when the device is created, and shown once in that response
- **Rotatable by an admin via `POST /admin/devices/{id}/token`**, which returns the new token once and invalidates the old one
- **Never returned** by the list, get or update routes
- **Sent by the Pi** as `Authorization: Bearer <token>` on every `/sync/**` call
- **Stored in the Pi's `.env` as `DEVICE_TOKEN`**
