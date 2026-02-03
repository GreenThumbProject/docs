# Local Development Setup

Set up a local development environment for GreenThumb.

## Prerequisites

- Python 3.11+
- Docker and Docker Compose
- Git
- (Optional) Raspberry Pi 5 for hardware testing

## Clone Repositories

```bash
# Create workspace
mkdir greenthumb && cd greenthumb

# Clone repositories
git clone https://github.com/GreenThumbProject/rasp5.git
git clone https://github.com/GreenThumbProject/greenthumb-core.git
git clone https://github.com/GreenThumbProject/docs.git
```

## greenthumb-core Development

### Setup

```bash
cd greenthumb-core
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# .venv\Scripts\activate    # Windows

pip install -e .
```

### With Hardware Support

```bash
pip install -e ".[rpi5]"
```

!!! note
    Hardware dependencies only work on Raspberry Pi with I2C enabled.

### Testing

```python
# Test database utilities
from greenthumb_core.db import get_engine
engine = get_engine()
print("Database connected!")

# Test models
from greenthumb_core.models import Device
print(Device.__tablename__)
```

## rasp5 Development

### Setup

```bash
cd rasp5
cp .env.example .env
# Edit .env with your values
```

### Run with Docker

```bash
make up        # Start services
make logs      # View logs
make down      # Stop services
```

### Run API Locally (without sensors)

```bash
cd microcontroller-api
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Need greenthumb-core
pip install git+https://${GH_PAT}@github.com/GreenThumbProject/greenthumb-core.git

# Start PostgreSQL
docker compose up -d db

# Run API
uvicorn app:app --reload --port 8080
```

## docs Development

### Setup

```bash
cd docs
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Preview

```bash
mkdocs serve
# Open http://localhost:8000
```

### Build

```bash
mkdocs build
# Output in site/
```

## Testing on Raspberry Pi

### SSH Setup

```bash
# Copy SSH key
ssh-copy-id pi@<raspberry-pi-ip>

# Connect
ssh pi@<raspberry-pi-ip>
```

### Deploy for Testing

```bash
# On Raspberry Pi
git clone https://github.com/GreenThumbProject/rasp5.git
cd rasp5
make up
```

### View Logs

```bash
make logs
# Or specific service
docker compose logs -f controller
```

## IDE Setup

### VS Code Extensions

- Python
- Pylance
- Docker
- YAML
- Markdown All in One

### settings.json

```json
{
    "python.defaultInterpreterPath": ".venv/bin/python",
    "python.formatting.provider": "black",
    "editor.formatOnSave": true,
    "[python]": {
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        }
    }
}
```

## Environment Variables

Create `.env` files as needed:

```bash
# rasp5/.env
DB_PASSWORD=dev_password
DOCKERHUB_USERNAME=your_username
GH_PAT=ghp_xxxxx

# For local testing without Docker
DATABASE_URL=postgresql://greenthumb:dev_password@localhost:5432/greenthumb
```
