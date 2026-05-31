# FinAlly Comprehensive Codebase Review

## Review Summary

This workspace is a specification scaffold, not an implemented FinAlly
application. `planning/PLAN.md` is detailed enough to guide implementation, but
the required runtime layers are absent: backend, frontend, schema, tests,
Docker packaging, launch scripts, README, and agent handoff documents.

The immediate security issue is the populated `.env` file without a
`.gitignore`. No application behavior can be approved until implementation and
tests are added.

## Review Scope

### Files Reviewed

- [x] `.env`
- [x] `.claude/CLAUDE.md`
- [x] `.claude/settings.local.json`
- [x] `.claude/commands/doc-review.md`
- [x] `.claude/agents/codex-review.md`
- [x] `.claude/agents/reviewer.md`
- [x] `.claude/skills/cerebras/SKILL.md`
- [x] `planning/PLAN.md`

### Workspace Limitations

- [ ] `backend/`, `frontend/`, `db/`, and `test/` exist but contain no files.
- [ ] The workspace is not a Git repository, so commit history and tracked-file
      status cannot be inspected.
- [ ] `planning/HANDOFF.md`, `REVIEW.md`, `README.md`, `Dockerfile`,
      `docker-compose.yml`, `.gitignore`, and `.env.example` are absent.

## Immediate Blockers

Complete these items before dependent review or repository sharing:

- [ ] Protect populated credentials with a root `.gitignore`.
- [ ] Add a placeholder-only `.env.example`.
- [ ] Rotate credentials if the populated `.env` values were shared or
      committed elsewhere.
- [ ] Implement the backend contracts from `planning/PLAN.md` sections 6-9.
- [ ] Implement the frontend contracts from `planning/PLAN.md` section 10.
- [ ] Add the production `Dockerfile`.
- [ ] Add unit and Playwright test coverage from `planning/PLAN.md` section 12.

## Findings

### Blockers

- [ ] **Implement the backend**
  - **Path:** `backend/`
  - **Issue:** The directory is empty. Required FastAPI routes, SQLite schema,
    lifespan hooks, WAL configuration, atomic trading logic, cookie identity,
    SSE fan-out, market data providers, and LiteLLM integration do not exist.
  - **Action:** Implement the backend contracts in `planning/PLAN.md`
    sections 6-9.

- [ ] **Implement the frontend**
  - **Path:** `frontend/`
  - **Issue:** The directory is empty. There is no Next.js static-export
    application and no REST or SSE integration.
  - **Action:** Implement the frontend contracts in `planning/PLAN.md`
    section 10.

- [ ] **Add the production Dockerfile**
  - **Path:** `Dockerfile`
  - **Issue:** The required multi-stage Dockerfile is missing. The
    single-container build, static asset copy from `frontend/out` to
    `/app/static`, health check, and port `8000` startup contract cannot work.
  - **Action:** Add the Dockerfile described in `planning/PLAN.md` section 11.

- [ ] **Protect local credentials**
  - **Paths:** `.env`, `.gitignore`, `.env.example`
  - **Issue:** `.env` contains populated OpenRouter and Alpaca values, but
    `.gitignore` is missing. Initializing or copying the workspace into version
    control could expose secrets. The required `.env.example` is also missing.
  - **Action:** Add `.gitignore` before initializing Git, ignore `.env` and
    database files, add a placeholder-only `.env.example`, and rotate
    credentials if they were shared or committed elsewhere.

- [ ] **Add automated tests**
  - **Path:** `test/`
  - **Issue:** The directory is empty and there are no backend or frontend unit
    tests. Contract-critical behavior cannot be verified.
  - **Action:** Add the unit and Playwright coverage required by
    `planning/PLAN.md` section 12.

### Major Findings

- [ ] **Add the agent handoff document**
  - **Path:** `planning/HANDOFF.md`
  - **Issue:** Active implementation status and blockers cannot be
    communicated between agents.
  - **Action:** Add `planning/HANDOFF.md` and record the blockers from this
    review.

- [ ] **Add the shared reviewer log**
  - **Path:** `REVIEW.md`
  - **Issue:** The append-only reviewer log required by
    `.claude/agents/reviewer.md` is missing.
  - **Action:** Add `REVIEW.md` or explicitly adopt this report as the initial
    shared review log.

- [ ] **Add setup documentation**
  - **Path:** `README.md`
  - **Issue:** Users cannot discover the one-command launch flow, environment
    setup, test commands, or security warning for local credentials.
  - **Action:** Add a README covering setup, start/stop scripts, environment
    variables, and test execution.

- [ ] **Add launch scripts**
  - **Path:** `scripts/`
  - **Issue:** The required idempotent macOS/Linux and Windows start/stop
    scripts are absent.
  - **Action:** Add the scripts described in `planning/PLAN.md` section 11.

- [ ] **Add or defer the Docker Compose wrapper**
  - **Path:** `docker-compose.yml`
  - **Issue:** The optional convenience wrapper named in the required
    directory structure is missing.
  - **Action:** Add the wrapper or update `planning/PLAN.md` if intentionally
    deferred.

### Minor Findings

- [ ] **Preserve the empty database directory**
  - **Path:** `db/`
  - **Issue:** The directory lacks the planned `.gitkeep` and will disappear
    if committed while empty.
  - **Action:** Add `db/.gitkeep` and ignore `db/*.db`, WAL, and shared-memory
    files.

- [ ] **Document all environment variables**
  - **Path:** `.env`
  - **Issue:** `DATABASE_PATH`, `PRICE_STALE_AFTER_SECONDS`,
    `MARKET_DATA_MAX_SYMBOLS`, `OPENROUTER_MODEL`,
    `OPENROUTER_FALLBACK_MODEL`, and `OPENROUTER_TIMEOUT_SECONDS` are absent.
  - **Action:** Document all supported variables in `.env.example` and keep
    safe defaults in backend settings.

### Nitpick

- [ ] **Remove or document the empty Cerebras skill**
  - **Path:** `.claude/skills/cerebras/SKILL.md`
  - **Issue:** The empty skill file has no executable instructions and may
    confuse agents that discover it.
  - **Action:** Remove it or document the intended workflow.

## Contract Verification Checklist

### Failed

- [ ] **Hardcoded secret protection**
  - Populated credentials exist in `.env`, and `.gitignore` is absent.

- [ ] **Docker static export integration**
  - `Dockerfile` is absent.

### Blocked By Missing Implementation

- [ ] Correctness against `planning/PLAN.md`
- [ ] SQL injection protection through parameterized queries
- [ ] Visitor cookie attributes
- [ ] Rejection of client-supplied `user_id`
- [ ] Frontend/backend API contracts
- [ ] SSE cookie scoping and fan-out
- [ ] Integer micros for money and quantity
- [ ] SQLite WAL mode and busy timeout
- [ ] FastAPI lifespan startup and shutdown
- [ ] Market data provider interface
- [ ] LiteLLM integration through OpenRouter
- [ ] Strict Pydantic validation of LLM output
- [ ] Maximum 10 LLM actions per response

## Approved Planning Items

- [x] `planning/PLAN.md` defines the expected API routes, fixed-precision money
      representation, cookie-only visitor identity, SSE scoping, provider
      interface, SQLite lifecycle, Docker layout, LiteLLM call pattern, strict
      Pydantic validation, and 10-action limit clearly enough to implement.

- [x] `.claude/agents/reviewer.md` reinforces the key checks: parameterized
      SQL, cookie attributes, no client-supplied `user_id`, integer money
      values, WAL mode, lifespan hooks, provider conformance, LiteLLM usage,
      validated actions, and separate `action_results`.

- [x] `.env` stores credentials in environment variables rather than embedding
      them in the available documentation or assistant configuration files.

## Review Note

No source-level hardcoded secret, SQL interpolation, floating-point money use,
deprecated FastAPI startup event, or unscoped SSE handler was found because no
source files exist. This is not approval of those implementation requirements.
