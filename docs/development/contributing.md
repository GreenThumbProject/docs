# Code Standards

Code style guidelines for GreenThumb development.

!!! note "Private Research Project"
    This is a private research project. These standards are maintained for future team collaboration and code consistency.

## Python

### Style Guide

- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Use type hints for all functions
- Write docstrings for public functions and classes
- Use functions and classes for reusable code

### Example

```python
def calculate_average(values: list[float]) -> float:
    """Calculate the average of a list of values.
    
    Args:
        values: List of numeric values
        
    Returns:
        The arithmetic mean of the values
        
    Raises:
        ValueError: If the list is empty
    """
    if not values:
        raise ValueError("Cannot calculate average of empty list")
    return sum(values) / len(values)
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Functions | `snake_case` | `get_sensor_data()` |
| Classes | `PascalCase` | `SensorModel` |
| Constants | `UPPER_SNAKE` | `MAX_RETRIES` |
| Variables | `snake_case` | `sensor_reading` |

## SQL

- Use UPPERCASE for SQL keywords
- Use lowercase for table/column names
- Use underscores for multi-word names

```sql
CREATE TABLE IF NOT EXISTS device_sensor (
    id_device_sensor SERIAL PRIMARY KEY,
    id_device INTEGER NOT NULL REFERENCES device(id_device)
);
```

## Commit Messages

Use conventional commit format:

```
type(scope): description

[optional body]
```

### Types

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `refactor` | Code refactoring |
| `test` | Adding tests |
| `chore` | Maintenance tasks |

### Examples

```
feat(api): add endpoint for historical data
fix(sensors): handle AHT10 read timeout
docs(readme): update installation instructions
```

## Project Structure

- Keep related code together in modules
- Use clear, descriptive file names
- Separate concerns (data access, business logic, API)

## Related

- [Local Setup](local-setup.md) - Development environment
- [CI/CD](ci-cd.md) - Automated builds and deployment
