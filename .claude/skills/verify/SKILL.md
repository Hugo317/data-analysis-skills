---
name: verify
description: Run the project's checks for skill-builder — dependency sync, linting, tests, and a smoke run of the CLI. Use after making changes, before committing, or when asked to verify that the project still works.
---

# Verify

Run these in order from the project root. Report failures with the actual output; do not
summarize a failure as a pass.

1. `uv sync` — dependencies in step with `pyproject.toml`.
2. Lint, if configured (a `[tool.ruff]` section in `pyproject.toml` or a `ruff.toml`):
   `uv run ruff check .` and `uv run ruff format --check .`
3. Tests, if a `tests/` directory exists: `uv run pytest`
4. Smoke test the entry point: `uv run skill-builder`

Skip a step only when its tooling genuinely isn't set up yet, and say which steps you
skipped and why. Never substitute a bare `python`, `pip`, or `pytest` for the `uv run`
form — that bypasses the project venv.
