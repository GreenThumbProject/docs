## Context

1. `rasp5/greenthumb-models/src/greenthumb_models/models.py` — all SQLModel table definitions
2. `rasp5/greenthumb-models/src/greenthumb_models/sync_schemas.py` — sync Pydantic schemas
3. `rasp5/microcontroller-api/background_tasks.py` — three background task coroutines
4. `rasp5/microcontroller-api/lifespan.py` — FastAPI lifespan (startup/shutdown)
5. `rasp5/microcontroller-api/sync_client.py` — Pi→Cloud HTTP client
6. `rasp5/microcontroller-api/routes/settings_routes.py` — /settings/ routes
7. `rasp5/microcontroller-api/routes/state_routes.py` — /state/ routes
8. `cloud/greenthumb-api/routes/admin/router.py` — admin routes
9. `cloud/greenthumb-api/routes/sync/sync_routes.py` — sync routes
10. `cloud/compose.yaml` — cloud Docker Compose
11. `rasp5/compose.yaml` — Pi Docker Compose
12. `rasp5/.env.example` — all environment variables
13. `rasp5/local-dashboard/src/api/piApi.js` — Pi dashboard API hooks
14. `cloud/admin-dashboard/src/api/cloudApi.js` — cloud dashboard API hooks

---

## Prompt

You are updating the MkDocs documentation for GreenThumb, an automated greenhouse system.

GreenThumb is a **Cloud + Pi hybrid**: each Raspberry Pi 5 runs a self-contained greenhouse controller (FastAPI + PostgreSQL + React dashboard + hardware drivers), and a shared cloud backend aggregates data from all Pi nodes and serves a fleet admin dashboard.

### Current documentation files that need updating

Update each of the following files. Do not change the Research, Project Summaries, or Development sections unless code changes directly affect them. Do not invent features or endpoints — only document what is present in the provided source files.

Analyze the current version of the docs repo and update them to the current version of the code.

Add diagrams and tables if needed to better illustrate the ideas.

### Rules

1. **Only document what exists in the source files.** Do not add hypothetical future features.
2. **Keep Mermaid diagrams accurate** — every service and connection in the diagram must correspond to a real service in compose.yaml.
3. **API endpoint paths must match** the route definitions exactly (prefix + path).
4. **Environment variables must match** `.env.example` exactly — don't add or remove variables.
5. **Do not modify** Research pages, Project Summary pages, or anything in `docs/research/` or `docs/summaries/`.
6. **Table column names** in `docs/components/database.md` must match the SQLModel field names (e.g. `id_cultivation_phase`, not `cultivation_phase_id`).
7. **Auth model** — users authenticate with JWT (issued by auth-service Java), devices authenticate with `device_token` Bearer token (validated against the `device` table in PostgreSQL).

### Output format

Follow the same format as the current documentation files.
