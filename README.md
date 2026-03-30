# py_commons_per

A shared Python utility library for HashiCorp Vault secret loading and logging configuration. Designed to work consistently across local, Docker, and GitHub Actions environments.

## Installation

```bash
pip install py_commons_per
```

Or with uv:

```bash
uv pip install py_commons_per
```

**Requires Python >= 3.11**

## Modules

### `vault_secret_loader`

Loads secrets from HashiCorp Vault (KV v2) and injects them as environment variables.

**Required environment variables:**

| Variable | Description |
|---|---|
| `VAULT_ADDR` | Vault server URL (e.g. `http://localhost:8200`) |
| `VAULT_TOKEN` | Vault authentication token |
| `VAULT_SECRET_PATH` | Comma-separated secret paths (e.g. `secret/myapp,secret/shared`) |

```python
from py_commons_per.vault_secret_loader import load_secrets

load_secrets()
# Secrets are now available via os.environ or os.getenv(...)
```

**Environment setup examples:**

- **Local:** `export VAULT_ADDR=http://localhost:8200` in your shell
- **Docker:** set in `docker-compose.yml` under `environment:`
- **GitHub Actions:** set in GitHub Secrets

### `logging_setup`

Configures Python logging from a `logging_config.json` file at the calling project's root. Falls back to `basicConfig(level=INFO)` if the file is missing or invalid.

```python
from py_commons_per.logging_setup import setup_logging

setup_logging()                              # uses logging_config.json by default
setup_logging(config_file="my_logging.json") # custom config file name
```

**`logging_config.json` example:**

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "standard": {
      "format": "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
    }
  },
  "handlers": {
    "console": {
      "class": "logging.StreamHandler",
      "formatter": "standard"
    }
  },
  "root": {
    "level": "INFO",
    "handlers": ["console"]
  }
}
```

## Recommended Startup Pattern

Call these at application startup before any other imports that depend on secrets or logging:

```python
from py_commons_per.logging_setup import setup_logging
from py_commons_per.vault_secret_loader import load_secrets

setup_logging()  # configure logging first
load_secrets()   # inject Vault secrets into os.environ
```

## Building from Source

```bash
uv pip install -e .

# Build distribution
python -m build
```