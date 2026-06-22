# AGENTS.md — Backend (FastAPI)

> Cross-tool agent rules. Read by Antigravity, Cursor, Claude Code, and Codex.
> This file governs the **FastAPI repo only**. Any frontend or sibling service lives in its own repo with its own AGENTS.md.

---

## Tech Stack
- Language: Python 3.12+
- Framework: FastAPI (ASGI)
- Server: uvicorn (dev) / gunicorn + uvicorn workers (prod)
- Validation & schemas: Pydantic v2
- API docs: FastAPI built-in — Swagger UI at `/docs`, ReDoc at `/redoc`, schema at `/openapi.json`
- Database/driver: async driver (Motor for MongoDB / asyncpg / SQLAlchemy async) — match the project
- Auth: OAuth2 + JWT via FastAPI dependencies (adjust to project)
- Tooling: ruff (lint + format), mypy (types), pytest + httpx (async tests)
- Env management: pydantic-settings / env vars — secrets via env only

## Commands
- Install deps: `pip install -r requirements.txt`
- Run dev server: `uvicorn app.main:app --reload`  (adjust import path)
- Run tests: `pytest`
- Lint + format: `ruff check . --fix && ruff format .`
- Type check: `mypy .`
- Export OpenAPI schema (for consumers): `python -c "import json,app.main as m; print(json.dumps(m.app.openapi()))" > openapi.json`

---

## Code Quality
- Follow PEP 8. Format with ruff; do not hand-format.
- Type hints on every function signature (params + return). Run mypy before declaring a task done.
- **Async-first**: endpoints and I/O (DB, HTTP, queue) use `async def` and async clients. Never block the event loop with sync I/O inside async paths; offload CPU-bound work to a threadpool/worker.
- Keep functions small and single-purpose. Cyclomatic complexity target ≤ 10.
- Keep route files focused; split routers by domain (`app/routers/orders.py`) once a file passes ~400 lines.
- Business logic lives in services/repositories, NOT in route handlers. Keep handlers thin.
- No print() for logging — use the `logging` module / structured logging.
- DRY: reuse Pydantic models, dependencies, and router patterns; don't copy-paste endpoint logic.

## Security (non-negotiable)
- **Never hardcode secrets** (API keys, DB URIs, JWT signing keys). Env vars only.
- **Never read or print the contents of `.env`** files.
- All input is validated through **Pydantic models** — never trust raw request data.
- Enforce auth on every protected route via FastAPI dependencies (e.g. `Depends(get_current_user)`). Make public routes explicit and intentional.
- Use parameterized queries / the ORM driver correctly — never build queries via string concatenation (prevents injection, incl. NoSQL injection on MongoDB).
- Configure CORS via `CORSMiddleware` with an explicit allow-list — never `allow_origins=["*"]` in production.
- Rate-limit sensitive routes (login, password reset) — e.g. slowapi or gateway-level.
- Hash passwords with a vetted library (passlib/bcrypt) — never store or log plaintext.
- Validate file uploads (type, size); stream large uploads, don't load fully into memory.
- Never return internal exception details/stack traces to clients; return clean error responses.
- Set sensible limits (request body size, timeouts) and security headers.

## API Documentation (Swagger / OpenAPI) — required for every endpoint
- FastAPI generates the schema automatically — your job is to make it complete and accurate.
- Every route declares a `response_model` and typed request body (Pydantic) — no untyped dict responses.
- Add `summary`, `description`, and `tags` to every route; group related routes by tag.
- Provide `responses={...}` for non-200 outcomes (400/401/403/404/409/422/500) with their models.
- Use `Field(..., description=..., examples=...)` on Pydantic fields so docs are self-explanatory.
- Swagger UI must stay usable at `/docs`. Verify new endpoints render correctly there.
- When a contract changes, regenerate/export `openapi.json` so any consuming repo can sync types.

## Code Documentation
- Module-level docstring on every non-trivial module describing its responsibility.
- Docstrings on public service functions, dependencies, and complex logic (Google style).
- Explain *why*, not *what*, in inline comments.
- Keep `README.md` current: setup, env vars (names only, never values), run, test.
- Maintain `.env.example` listing every required env var by name with a placeholder.

## Testing
- Test every route (happy path + at least one failure/edge case) using `httpx.AsyncClient` / `TestClient`.
- Test auth: confirm unauthorized requests are rejected with the right status.
- Test Pydantic validation boundaries (422 cases).
- Use pytest + pytest-asyncio for async tests; mock external services (DB, OpenAI, queues).
- Keep coverage ≥ 80% on new/changed code. Tests + lint must pass before a task is done.

## Database & Data Safety
- **Ask for explicit approval before** destructive operations (dropping collections/tables, bulk deletes, data-transform migrations).
- Never run anything against a production datastore without confirmation.
- For SQL projects, manage schema via migrations (Alembic); never edit applied migrations.

## Git Conventions
- Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.
- Never commit directly to `main`. Branch pattern: `type/short-description`.
- One logical change per commit. Never commit `.env`, secrets, or local data dumps.

## Agent Behavior & Guardrails
- Before starting, list every file you intend to create or modify.
- If a change touches more than ~5 files, pause and confirm the plan first.
- Ask for approval before any destructive shell command (rm, DB drops, force push).
- When you change an API contract (path, request, or response shape), call it out explicitly in your summary so any **consuming frontend/service can be updated to match** — name the route, method, and response model. (If this service has no external consumer, ignore this line.)
- After a multi-step task, summarize what changed in 3–5 bullets and note any follow-up needed in consuming repos.

---

## Agent Skills (invoke when the task matches)
> Install targeted skills only — do not bulk-install. Use the right skill for the task below.

- **`backend-architect`** — invoke when designing a new service, router group, or major feature, or deciding service/repository boundaries.
- **`fastapi-pro`** — invoke for FastAPI-specific work: dependency injection, async patterns, Pydantic v2 models, router structure, background tasks.
- **`api-design-principles`** — invoke when adding/changing any endpoint, to keep paths, status codes, pagination, and response models consistent before finalizing the contract.
- **`security-auditor`** — **invoke before committing any change that touches auth, dependencies/Depends, user input, file uploads, or env/secret handling.** Treat its findings as blocking.
- **`python-pro`** (or a python scaffold skill) — invoke when creating new modules, to keep structure and typing consistent.
- **`full-stack-feature`** orchestration — invoke for any feature that spans this API + a frontend, so the endpoint and its consumer are built as one coherent flow.

### Skill guardrails
- Run `security-auditor` before declaring any auth/endpoint/input task complete.
- Don't let an irrelevant skill auto-activate hijack a task — if a skill fires that doesn't fit, ignore it and proceed as specified here.