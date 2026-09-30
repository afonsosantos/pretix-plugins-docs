# Development

Both plugins share the same layout: `uv` + `pyproject.toml`, pytest against a real pretix install,
ruff, and GitHub Actions publishing to PyPI via trusted publishing.

## Setup

1. Have a working [pretix development setup](https://docs.pretix.eu/en/latest/development/setup.html).
2. Clone the plugin and, with pretix's virtualenv active, install it editable:

    ```bash
    pip install -e .        # or: uv sync --extra test
    make                    # compile translations
    ```

3. Restart the pretix dev server (plugins are read at startup, not on autoreload) and enable the
   plugin under the event's **Settings → Plugins**.

## Common tasks

```bash
make test                 # run the test suite
make lint                 # ruff check + format check
make format               # ruff fix + format
make translate            # extract strings into locale/*/django.po
make compile-translations # .po → .mo
make bump-version 1.2.3   # then: uv lock
```

## Releasing

Publishing a GitHub release runs lint and tests, builds, and publishes to PyPI.

## Repositories

- [afonsosantos/pretix-eupago](https://github.com/afonsosantos/pretix-eupago)
- [afonsosantos/pretix-pt-invoicing](https://github.com/afonsosantos/pretix-pt-invoicing)
