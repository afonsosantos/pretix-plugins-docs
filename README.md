# pretix-plugins-docs

MkDocs site for [pretix-eupago](https://github.com/afonsosantos/pretix-eupago) and
[pretix-pt-invoicing](https://github.com/afonsosantos/pretix-pt-invoicing).

```bash
uv run mkdocs serve   # http://127.0.0.1:8000
uv run mkdocs build --strict
```

Pushes to `main` deploy to GitHub Pages via `.github/workflows/docs.yml`.
