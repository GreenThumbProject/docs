# Contributing

Thank you for your interest in contributing to GreenThumb!

## Getting Started

1. Fork the repository you want to contribute to
2. Clone your fork locally
3. Create a feature branch: `git checkout -b feature/your-feature`
4. Make your changes
5. Test thoroughly
6. Submit a pull request

## Code Style

### Python

- Follow [PEP 8](https://peps.python.org/pep-0008/)
- Use type hints for all functions
- Write docstrings for public functions

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

### SQL

- Use uppercase for SQL keywords
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

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

Examples:
```
feat(api): add endpoint for historical data
fix(sensors): handle AHT10 read timeout
docs(readme): update installation instructions
```

## Pull Requests

1. **Title**: Use conventional commit format
2. **Description**: Explain what and why
3. **Testing**: Describe how you tested
4. **Screenshots**: Include for UI changes

### PR Checklist

- [ ] Code follows style guidelines
- [ ] All tests pass
- [ ] Documentation updated if needed
- [ ] No sensitive data exposed

## Repository-Specific Guidelines

### rasp5

- Test on actual Raspberry Pi before submitting
- Docker build must succeed for ARM64
- Update `compose.yaml` if adding services

### greenthumb-core

- Maintain backwards compatibility
- Add tests for new features
- Update `pyproject.toml` if adding dependencies

### docs

- Preview locally with `mkdocs serve`
- Check all links work
- Keep language consistent (EN/PT as appropriate)

## Questions?

Open an issue in the relevant repository or contact the maintainer directly.
