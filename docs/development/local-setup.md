# Local Development Setup

Set up a GreenThumb development environment on a Windows workstation. Workstation commands are for PowerShell. Commands that run on the Raspberry Pi are marked as such.

## Prerequisites

- Git
- Docker Desktop
- Python 3.11
- Node.js 20
- Access to the GreenThumbProject GitHub organization (the code repositories are private)

## Clone the repositories

Clone the repositories next to each other in one folder. The schema tooling expects `database` beside `rasp5` and `cloud`.

```powershell
mkdir greenthumb
cd greenthumb
git clone --recurse-submodules https://github.com/GreenThumbProject/rasp5.git
git clone --recurse-submodules https://github.com/GreenThumbProject/greenthumb-cloud.git cloud
git clone https://github.com/GreenThumbProject/database.git
git clone https://github.com/GreenThumbProject/docs.git
```

## Shared package

The shared `greenthumb` package (models, CRUD helpers, sync) lives in the `greenthumb-models` repository, which is a submodule of both `rasp5` and `cloud`. Edit it in `rasp5\greenthumb-models`. CI moves the cloud's copy to the same commit (see [CI/CD](ci-cd.md)).

## Node API tests

There is no mock mode: the node API runs only on a Pi with its hardware attached. On a workstation, run its test suite instead.

```powershell
cd rasp5\microcontroller-api
python -m venv .venv
.venv\Scripts\python.exe -m pip install -e "..\greenthumb-models[api,test]" -e ..\greenthumb-rpi5 -r requirements-test.txt loguru
.venv\Scripts\python.exe -m pytest
```

## Local dashboard

```powershell
cd rasp5\local-dashboard
npm ci
npm test
npm run dev
```

The dev server sends API calls to `http://localhost:8080`, so pages that need live data only work with a node API reachable there.

## Cloud stack (local development)

```powershell
cd cloud
Copy-Item .env.example .env
docker compose up --build
# Admin dashboard: http://localhost:443 (plain HTTP)
# API docs: http://localhost:8000/docs
# Landing page: http://localhost:8083
```

## Database schema

Schema changes are made only in the `database` repository: edit `schemas/` and add a dated file in `migrations/`. Then refresh the generated copy in `rasp5` or `cloud` from that folder:

```powershell
cd rasp5
cmd /c make schema-sync
```

## Documentation

```powershell
cd docs
python -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m mkdocs serve
.venv\Scripts\python.exe -m mkdocs build --strict
```

`mkdocs serve` previews the site at `http://localhost:8000`.

## On the Pi

On the Pi (Linux), update the node from the `deploy/` folder:

```bash
cd ~/Documents/greenthumb/rasp5/deploy
make pull
make up
```

!!! warning
    Never run the root `rasp5` Makefile on the Pi. It is for workstations, and `make dev-reset` wipes the database and photo volumes.

## Related

- [Contributing](contributing.md) - Code standards, branches, commit messages
- [CI/CD](ci-cd.md) - Automated builds and deployment
