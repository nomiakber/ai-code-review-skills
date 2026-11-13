# Profile — python

- **Source**: `.py`
- **Exclusions**: `venv/`, `.venv/`, `__pycache__/`, build/dist artifacts, generated code, and paths ignored by the project VCS ignore files.
- **Project files**: `pyproject.toml` and/or `requirements*.txt` / `Pipfile`, pytest / ruff / mypy (or equivalent) configs, to learn Python version, FastAPI detection, and lint/type-check/test commands.
- **Extra prep**: detect FastAPI. Treat the project as FastAPI when dependencies list `fastapi` or in-scope code constructs `FastAPI(` / `APIRouter`. When FastAPI is not detected, skip FastAPI-only Map items, the FastAPI criteria category, and the FastAPI lens.
- **Tools**: the project's lint, type-check, and test commands when present (for example ruff, mypy, pytest).
- **Map**:
  - Entrypoints (CLI, `__main__`, app factory, scripts).
  - Package layout and public modules used in scope.
  - I/O and side effects: files, DB, network, subprocess.
  - Concurrency model used in scope (async, threads, processes).
  - When FastAPI is detected: routes, dependency injection, Pydantic models, middleware, background tasks, security dependencies.
- **Extra scope**: imported modules that define the in-scope contract; do not audit entire third-party frameworks.
- **Cross-slice**: duplication, layering, circular imports, dual write paths.
- **Lenses**:
  1. Input validation, injection, and secrets handling.
  2. Correctness and error paths: partial failure, swallowed exceptions, wrong contracts.
  3. Resource and async safety: leaks, blocking calls on event loops, cancellation gaps with concrete cost.
  4. FastAPI deps/security (only when FastAPI detected): dependency overrides, auth gaps, response model leakage, unsafe CORS.
- **Example**:

  > `api/users.py:77` (`update_user`) builds SQL with an f-string from `user_id`, so a crafted id can alter the query. Use a parameterized query or the project's query API. Keep the response shape unchanged. Verify with tests for valid update, missing user, and hostile `user_id` values.
