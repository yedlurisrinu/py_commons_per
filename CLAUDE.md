# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (uses uv)
uv pip install -e .

# Build distribution package
python -m build

# Install hvac directly
uv pip install "hvac>=2.1.0,<3.0.0"
```

No test runner, linter, or formatter is configured in this project.

## Architecture

`py_commons_per` is a small shared utility library with two modules:

### `vault_secret_loader.py`
Loads secrets from HashiCorp Vault (KV v2) into `os.environ`. Requires three env vars:
- `VAULT_ADDR` — Vault server URL
- `VAULT_TOKEN` — authentication token
- `VAULT_SECRET_PATH` — comma-separated list of secret paths (e.g. `secret/myapp,secret/shared`)

Paths are read from the `secret/` mount point; the `secret/` prefix in each path is stripped before querying. All key/value pairs from each secret are injected as environment variables.

### `logging_setup.py`
Configures Python logging from a `logging_config.json` file expected at the project root of the *calling* application. Falls back to `basicConfig(level=INFO)` if the file is missing or malformed.

## Usage Pattern

Consuming projects call these at application startup before any other imports that need secrets or logging:

```python
from py_commons_per.logging_setup import setup_logging
from py_commons_per.vault_secret_loader import load_secrets

setup_logging()   # configure logging first
load_secrets()    # inject Vault secrets into os.environ
```