# Technical Environment: Cornell AI Course Planning Agent

This document defines the technical tooling, standards, constraints, and preferences for building the Cornell AI Course Planning Agent described in `docs/vision.md`. It is the binding technical reference for design, code generation, and testing. Where this document and a generated artifact disagree, raise the conflict through the AI-DLC workflow instead of choosing silently.

---

## Project Technical Summary

- **Project Name**: Cornell AI Course Planning Agent
- **Project Type**: Greenfield
- **Primary Runtime Environment**: Local only, on team laptops. No hosting, no user accounts.
- **Cloud Provider**: None
- **Target Deployment Model**: One Docker Compose service (the web app) on each laptop. The LLM runs natively on the host in Ollama, and the app calls it over the Docker host gateway.
- **Team Size**: 5
- **Team Machines**: macOS on Apple Silicon and Windows 10/11 (no Linux). Minimum 16 GB RAM on every laptop. Windows laptops may have no GPU, in which case Ollama runs on the CPU.
- **Team Experience**: All members are comfortable in Python. At least two members have built LangChain or LangGraph applications and lead agent design. The rest work on the deterministic modules, ingestion, the UI, and tests.

---

## Programming Languages

### Required Languages

| Language | Version | Purpose | Rationale |
|----------|---------|---------|-----------|
| Python | 3.12.x | All application code: ingestion, search, prerequisite engine, planner, agent, web app, evaluation runner, tests | Team expertise. The LLM, embedding, and web libraries below all support 3.12. |
| HTML (Jinja2 templates) | HTML5 | Server-rendered pages and HTMX fragments | Server-rendered UI with no build step |
| CSS | CSS3 | Styling, served as local static files | Must comply with the Content Security Policy: no inline styles |
| SQL (SQLite dialect) | SQLite 3.40+ with FTS5 | Catalog tables and keyword search | Ships with Python; no database server |

### Permitted Languages

| Language | Conditions for Use |
|----------|-------------------|
| YAML | Docker Compose file and GitHub Actions workflows only |
| TOML | `pyproject.toml` only |
| Bash / PowerShell | One-line helper commands documented in the README. All real logic stays in Python scripts so it runs on both macOS and Windows. |

### Prohibited Languages

| Language | Reason |
|----------|--------|
| Hand-written JavaScript or TypeScript | The UI uses HTMX attributes only. A JS build step adds setup risk on 5 laptops, and inline scripts break the Content Security Policy. |
| Any second backend language (Node.js, Go, Java) | One-language codebase for a 5-person team with a fixed deadline |
| Jupyter notebooks in `src/` | Untestable and unreviewable in diffs. Exploration notebooks may live only in a git-ignored `scratch/` folder. |

---

## Frameworks and Libraries

### Required Frameworks

| Framework/Library | Version | Domain | Rationale |
|-------------------|---------|--------|-----------|
| uv | 0.9.x or later | Package and environment management | One tool for Python install, virtual env, and lock file. `pyproject.toml` plus a committed `uv.lock`. |
| FastAPI | 0.115.x or later | Web framework and routing | Async routes, Pydantic validation of form input, dependency injection for session and catalog |
| Uvicorn | 0.32.x or later | ASGI server | Standard FastAPI server |
| Jinja2 | 3.1.x | Server-side HTML templates | Autoescaping on by default |
| HTMX | 2.0.x, vendored into `static/vendor/` | Partial page updates for chat, plan edits, and forms | No JS build. Served locally, never from a CDN. |
| Pydantic | 2.x | All structured data: forms, profile, LLM outputs, prerequisite rules, snapshot records | One validation layer for user input and model output |
| pydantic-settings | 2.x | Configuration from `.env` | One typed `Settings` object, validated at startup |
| LangGraph | 1.x | Agent orchestration | An explicit state graph for map skills, search, check prerequisites, propose plan, validate, retry, and fix |
| langchain-core | 1.x | Tool definitions and message types | Used by LangGraph; `@tool` decorator for agent tools |
| langchain-openai | 1.x | Chat model client | One OpenAI-compatible client for local Ollama (`/v1`) and any hosted provider. Switching is configuration only. |
| Ollama | 0.12.x or later, installed natively on the host | Local LLM runtime | Model pulls by CLI, tool calling, OpenAI-compatible API |
| FastEmbed | 0.7.x | Local text embeddings (`BAAI/bge-small-en-v1.5`, 384 dimensions) | ONNX runtime, no PyTorch. Model files are baked into the Docker image so the app works offline. |
| chromadb | 1.x | Vector store for course embeddings (persistent, on disk) | Metadata filters by level, subject, and term. Familiar to the team. |
| httpx | 0.28.x | HTTP client for the Class Roster ingestion script and health checks | Async-capable with explicit timeouts |
| pytest | 8.x | Test runner | Standard Python test runner |
| Hypothesis | 6.x | Property-based testing | Mandatory for all deterministic components |
| pytest-cov | 6.x | Coverage measurement | Enforces the coverage gates in Testing Requirements |
| ruff | 0.8.x or later | Linting and formatting | Replaces black, flake8, and isort |
| mypy | 1.x | Static type checking | Strict on deterministic modules |
| pip-audit | 2.x | Dependency vulnerability scanning | Runs in CI |

### Models

| Role | Model | Runtime | Notes |
|------|-------|---------|-------|
| Primary LLM | `qwen3:8b` (4-bit, about 5 GB) | Ollama on the host | Tool calling and structured output. Thinking mode **off** for interactive calls. |
| Fallback LLM | `qwen3:4b` (4-bit, about 2.5 GB) | Ollama on the host | Same family, so prompts carry over. Selected by configuration, not switched automatically on errors. |
| Prerequisite parser (offline ingestion) | `qwen3:8b` with thinking mode **on** | Ollama on the host | Runs only during ingestion, where accuracy matters and latency does not |
| Optional hosted LLM | Any OpenAI-compatible endpoint | Provider's API over HTTPS | Used only when `LLM_API_KEY` is present in `.env`. Set by `LLM_BASE_URL`, `LLM_MODEL`, `LLM_API_KEY`. No provider is hard-coded. |
| Embeddings | `BAAI/bge-small-en-v1.5` via FastEmbed | Inside the app container | One embedding model per index (see Data Patterns) |

Qwen3 thinking mode is controlled by the `LLM_THINKING_INTERACTIVE` and `LLM_THINKING_INGESTION` settings, implemented with Qwen3's documented `/think` and `/no_think` soft switch in the system prompt.

### Preferred Libraries

| Library | Purpose | Use When |
|---------|---------|----------|
| Python `sqlite3` (standard library) | Catalog store and FTS5 keyword index | All catalog reads and writes |
| Python `logging` (standard library) with a JSON formatter | Developer logs | All logging. No third-party logging library. |
| Python `secrets` (standard library) | Session IDs and CSRF tokens | Any random token |
| `tenacity` | Retry with a fixed attempt count and backoff | LLM calls and Class Roster API calls |
| `langgraph.checkpoint.memory.InMemorySaver` | Conversation state for chat revision | Only this checkpointer. State lives in memory for the session. |

### Prohibited Libraries

| Library | Reason | Alternative |
|---------|--------|-------------|
| `eval()`, `exec()`, `compile()` | Arbitrary code execution from LLM or catalog text | Pydantic-validated structured output and the deterministic rule evaluator |
| `pickle`, `joblib`, `shelve`, `dill` | Unsafe deserialization of stored data | JSON with Pydantic models; SQLite |
| `requests` | Blocks the async event loop; no default timeout | `httpx` with explicit timeouts |
| Flask, Django | The project uses one web framework | FastAPI |
| Streamlit, Gradio, Chainlit | The UI is a server-rendered web app | FastAPI + Jinja2 + HTMX |
| LangChain `AgentExecutor`, `initialize_agent` | Legacy agent APIs with hidden control flow | LangGraph state graph |
| `sentence-transformers`, `torch` | PyTorch adds over 1 GB to the Docker image and slows builds on mixed ARM and x86 laptops | FastEmbed (ONNX) |
| Chroma's default embedding function | A different model from `bge-small-en-v1.5`, so queries and documents would be embedded inconsistently | Embeddings computed by FastEmbed and passed to Chroma explicitly |
| `pandas` in application code | Not needed for per-request logic; heavy import | Standard library and Pydantic |
| CDN-loaded JavaScript or CSS | Breaks the Content Security Policy and needs network access at demo time | Vendored files in `static/` |
| black, flake8, isort | Replaced by one tool | ruff |
| pip, poetry, pipenv | One package manager and lock file | uv |
| Hosted vector databases (Pinecone, Weaviate Cloud, etc.) | Cost, external dependency, local-only product | Local Chroma |
| SQLite or Postgres LangGraph checkpointers | Would write student session data to disk | `InMemorySaver` |

### Library Approval Process

- To add a dependency, open a GitHub issue with the purpose, license (MIT, BSD, or Apache 2.0 only), last release date, and installed size.
- The technical environment owner and one other member approve it in the issue.
- The change is made through the AI-DLC workflow so it is recorded in `aidlc-docs/audit.md`. This document and `uv.lock` are updated in the same commit.

---

## Cloud Environment

### Cloud Provider

- **Primary Provider**: None. The product runs locally on team laptops.
- **Account Structure**: Not applicable
- **Regions**: Not applicable

### How the Product Runs

- **Web app**: a local FastAPI app in Docker Compose, served at `http://127.0.0.1:8000`. Inside the container, Uvicorn binds `0.0.0.0` so Docker can forward traffic. Compose publishes the port as `127.0.0.1:8000:8000`, so the app is reachable only from the laptop itself.
- **Model runtime**: Ollama runs natively on the host so Apple Silicon Macs use the GPU. The container reaches it at `http://host.docker.internal:11434/v1`. Docker Desktop provides this name on macOS and Windows; `extra_hosts` keeps it working if a Linux machine is added.
- **Data at runtime**: the app reads only the local catalog snapshot (SQLite and Chroma). It never calls the Class Roster API while serving a student.
- **Development tooling**: GitHub (repository) and GitHub Actions (CI, Dependabot) are development services. They are not part of the product runtime.

```yaml
# docker-compose.yml (shape to follow)
services:
  app:
    build: .
    ports:
      - "127.0.0.1:8000:8000"
    env_file: .env
    environment:
      LLM_BASE_URL: ${LLM_BASE_URL:-http://host.docker.internal:11434/v1}
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - ./data/index:/app/data/index
```

- **Base image**: `python:3.12.x-slim-bookworm`, pinned to an exact patch tag, multi-architecture (arm64 for Macs, amd64 for Windows). The app runs as a non-root user. The FastEmbed model is downloaded at build time into `/opt/models`.

### Service Allow List

| Service | Approved Use Cases | Constraints |
|---------|-------------------|-------------|
| Cornell Class Roster API (`https://classes.cornell.edu/api/2.0/`) | Ingestion and refresh script only | Never called at request time. Throttled to the API's published request-rate guidance (at most 1 request per second). Explicit timeouts. Resumable. |
| Ollama on the host (`host.docker.internal:11434`) | Chat model calls | The default and required runtime |
| Hosted OpenAI-compatible LLM endpoint | Optional chat model calls | Only when `LLM_API_KEY` is set. HTTPS only. |
| Hosted embedding API | Optional replacement for FastEmbed | Only when a key is set, and only after a full index rebuild with that model (see Data Patterns) |
| GitHub and GitHub Actions | Source control, CI, Dependabot | Development only |

### Service Disallow List

| Service | Reason | Alternative |
|---------|--------|-------------|
| Any cloud hosting (AWS, Azure, GCP, Vercel, Render, Hugging Face Spaces) | The product runs locally only | Docker Compose on each laptop |
| Managed databases | No server needed for read-only catalog data | SQLite and local Chroma |
| Hosted vector databases | Cost and external dependency | Local Chroma |
| Telemetry or analytics services, including Chroma's anonymized telemetry | Outbound calls from a local, privacy-preserving app | Telemetry disabled (`anonymized_telemetry=False`) |
| Web scraping of Cornell pages or job sites | Terms-of-service risk; out of scope in the vision | Class Roster API only |

### Service Approval Process

- Any new external service needs a GitHub issue covering purpose, cost (must be free), the data sent, and the offline behaviour.
- Team approval is required at a workflow stage gate, and this document is updated.

---

## Preferred Technologies and Patterns

### Architecture Patterns

| Pattern | When to Use | When Not to Use |
|---------|-------------|-----------------|
| Modular monolith (one Python package) | The whole application | Do not split into separate services or containers |
| Deterministic core, LLM at the edges | Prerequisite evaluation, plan validation, plan fixing, credit arithmetic, offering-term checks, course-code checks | Do not ask the LLM to decide anything code can compute |
| Propose, validate, retry, fix | Plan generation and revision: the LLM proposes, a deterministic validator checks, the LLM retries with the errors a fixed number of times, then a deterministic fixer repairs what remains | Single-shot LLM plans shown without validation |
| Offline batch ingestion | Fetching rosters, parsing prerequisites, building indexes | Anything in the request path |

**Module boundaries** (`src/course_planner/`):

| Module | Responsibility | May import |
|--------|----------------|-----------|
| `catalog` | Roster ingestion, snapshot loading, SQLite store, Chroma index, hybrid search | `config` |
| `prereq` | Pattern parser, LLM parse validation checks, rule model, deterministic evaluator | `catalog` (read only) |
| `planner` | Validator, fixer, credit and offering-term rules | `prereq`, `catalog` |
| `agent` | LangGraph graph, tools, prompts, LLM client | `catalog`, `prereq`, `planner`, `config` |
| `web` | FastAPI routes, templates, session store, security middleware | `agent`, `planner`, `catalog`, `config` |
| `evaluation` | One-command evaluation runner and report | all modules |
| `config` | `Settings` (pydantic-settings) | none |

`catalog`, `prereq`, and `planner` must never import `agent`, `langchain_*`, `langgraph`, or `openai`. This keeps them testable with no model running.

### Repository Layout

```
.
├── src/course_planner/{catalog,prereq,planner,agent,web,evaluation}/
├── src/course_planner/config.py
├── src/course_planner/web/templates/      # Jinja2 pages and partials
├── src/course_planner/web/static/         # CSS, vendored htmx.min.js
├── tests/unit/  tests/property/  tests/integration/
├── scripts/                               # ingest_roster.py, build_index.py, run_evaluation.py
├── data/snapshot/                         # committed: raw roster JSON + parsed rules
├── data/index/                            # git-ignored: catalog.sqlite3, chroma/
├── evaluation/                            # committed: profiles, prerequisite cases, reports
├── docs/                                  # vision.md, technical-environment.md
├── aidlc-docs/                            # AI-DLC artifacts only, never code
├── pyproject.toml  uv.lock  Dockerfile  docker-compose.yml  .env.example
```

### API Design Standards

- **Style**: server-rendered HTML. Page routes return full pages. HTMX routes return HTML fragments from `templates/partials/`. There is no public JSON API. The evaluation runner calls Python functions directly.
- **Health**: `GET /health` returns `{"status": "ok"}` (shallow). `GET /health/deep` checks the Ollama endpoint, the SQLite file, the Chroma collection, and the embedding-model match, and reports each one.
- **Methods**: `GET` for reads and `POST` for any change. Every `POST` carries a CSRF token.
- **Naming**: kebab-case URL paths (`/plan/move-course`), snake_case form fields and template variables.
- **Error format**: users see a generic message fragment (`partials/error.html`) with HTTP 4xx or 5xx and no stack trace. The log gets the request ID, error type, and module.

### Data Patterns

- **Source of truth (committed)**: `data/snapshot/rosters/{TERM}/{SUBJECT}.json`, the raw Class Roster responses for 4 years of terms and the 8 core subjects. Also `data/snapshot/prereq_rules.json`, the parsed rules with `verified` or `unverified` status and the original text.
- **Built locally (git-ignored)**: `data/index/catalog.sqlite3` (courses, offerings, rules, and an FTS5 table) and `data/index/chroma/`. One command, `uv run python scripts/build_index.py`, rebuilds both from the snapshot.
- **Keyword search**: SQLite FTS5 with BM25 over title, description, and learning outcomes.
- **Vector search**: Chroma collection `courses` with `{"hnsw:space": "cosine"}`, ID = course code, and metadata `subject`, `level`, `credits_min`, `credits_max`, `terms_offered`. Embeddings are always computed by FastEmbed and passed in explicitly.
- **Hybrid merge**: Reciprocal Rank Fusion over the two ranked lists, `score = Σ 1 / (60 + rank)`. Ranks are used, not raw scores.
- **One embedding model per index**: the Chroma collection metadata stores `embedding_model`. At startup the app compares it with `EMBEDDING_MODEL`. On a mismatch it refuses to start and tells the user to run `build_index.py`.
- **Grounding**: every course code in any LLM output is checked against SQLite before display. Codes not in the snapshot are removed and logged as fabrication events.
- **Student data**: held only in memory (see Session State). It is never written to SQLite, Chroma, logs, or files. The only output is a Markdown export the student chooses to download.

### Session State

- A random session ID (`secrets.token_urlsafe(32)`) is kept in a cookie with `HttpOnly`, `SameSite=Strict`, and `Path=/`.
- Profile, skills, and plan live in a server-side in-memory dictionary keyed by that ID, with a 60-minute idle timeout.
- LangGraph conversation state uses `InMemorySaver`, with `thread_id` = session ID.
- Restarting the app clears all sessions. This is intended.

### LLM Interaction Patterns

- Every LLM output that code consumes (skill map, parsed prerequisite rule, proposed plan, plan edit) comes from tool calls or structured output and is validated against a Pydantic model. Invalid output counts as a failed attempt.
- **Retries**: a fixed 3 attempts per call using `tenacity`, then a clear generic error. The student's inputs and current plan are kept.
- **Timeouts**: 60 seconds per LLM call and 10 seconds per Class Roster request.
- Prompts live in `src/course_planner/agent/prompts/` as versioned text files, not inline strings.

### Configuration

- All settings come from `.env` through one `Settings` class.
- `.env.example` is committed and lists every key with a safe default. `.env` is git-ignored.

| Setting | Default |
|---------|---------|
| `LLM_BASE_URL` | `http://host.docker.internal:11434/v1` |
| `LLM_MODEL` | `qwen3:8b` |
| `LLM_FALLBACK_MODEL` | `qwen3:4b` |
| `LLM_USE_FALLBACK` | `false` |
| `LLM_API_KEY` | empty (Ollama ignores it) |
| `LLM_THINKING_INTERACTIVE` | `false` |
| `LLM_THINKING_INGESTION` | `true` |
| `EMBEDDING_MODEL` | `BAAI/bge-small-en-v1.5` |
| `CATALOG_SUBJECTS` | `CS,INFO,ORIE,STSCI,MATH,ECE,BTRY,ECON` |
| `LOG_LEVEL` | `INFO` |

### Logging

- Standard library `logging` with a JSON formatter, written to `logs/app.log` (git-ignored) and stdout.
- Each entry has a timestamp, level, request ID, module, event, and duration in ms.
- Logged: tool names, LLM model name, latency, retry counts, validation failures, fabrication events.
- Never logged: profile content, course histories, chat text, career goals, API keys.

### Messaging and Events

- Not applicable. One process, synchronous request and response. Long-running work (ingestion, index build) runs as separate scripts, not background workers.

### Frontend Patterns

- **Component library**: none. Semantic HTML with one local stylesheet (`static/app.css`).
- **Interactivity**: HTMX attributes (`hx-post`, `hx-target`, `hx-swap`) only. No inline `<script>` or `style=`.
- **Accessibility**: WCAG 2.1 AA for custom elements:
  - every input has a `<label>`
  - visible focus states and full keyboard operation
  - verdicts and warnings shown as text, not color alone
  - chat updates announced through an `aria-live="polite"` region
- **Build tool**: none

---

## Security Requirements

### Authentication and Authorization

- **Authentication method**: none. A single local user with no accounts, logins, NetIDs, or Cornell SSO. The app never asks for name, NetID, or email.
- **Authorization model**: not applicable beyond session isolation. A session can only read its own state, looked up by its own cookie.
- **Session management**: random 256-bit session ID in an `HttpOnly`, `SameSite=Strict` cookie; 60-minute idle timeout; memory only.

### Data Protection

- **Data classification**:
  - Catalog snapshot and indexes: Public.
  - Student profile, plan, and chat: Confidential, kept in memory only.
- **Encryption at rest**: not applicable. No student data is stored, and catalog data is public.
- **Encryption in transit**: the app is served over HTTP on `127.0.0.1` only and is never exposed to the network. Calls to a hosted LLM use HTTPS (TLS 1.2+). Ollama traffic stays on the machine.
- **PII handling**: course history and career goals never reach logs or disk (see Logging). The Markdown export is generated on request and streamed to the browser, never saved on the server.

### Network Security

- Uvicorn binds `0.0.0.0` inside the container only. The host port is published on `127.0.0.1` only.
- CORS is not enabled, so only same-origin requests are allowed.
- HTTP security headers on every response:
  - `Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; frame-ancestors 'none'`
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Referrer-Policy: strict-origin-when-cross-origin`

### Input Validation

| Input | Limit |
|-------|-------|
| Chat message | at most 2,000 characters |
| Career goal | at most 300 characters |
| Interests, topics to avoid | at most 500 characters each |
| Completed or fixed course list | at most 80 codes; each must match `^[A-Z]{2,5} \d{4}$` after normalization (uppercase, single space) |
| Credit range | integers, minimum 1, maximum 30, min ≤ max |
| Request body | at most 64 KB |

- Every form is a Pydantic model with `extra="forbid"`.
- Jinja2 autoescaping stays on. `|safe` is prohibited on user or LLM text.
- **Rate limiting**: an in-process limiter allows 10 chat requests per minute per session. This stops runaway loops from flooding the local model.
- **CSRF**: a per-session token stored server-side, embedded in every form, and compared with `secrets.compare_digest`.

### Secrets Management

- **Secrets storage**: `.env` only, created by each member from the committed `.env.example`. `.gitignore` already excludes `.env` and `.env.*` but keeps `.env.example`.
- **Prohibited practices**:
  - no keys in code, templates, tests, Dockerfiles, compose files, or commits
  - no secrets in logs or error messages
  - no keys baked into Docker images (`.env` is passed at runtime with `env_file`)
- The only secret is the optional `LLM_API_KEY`. The app runs fully without it.

### Compliance Requirements

- **Standards**: none specific (course project; public data; no stored student records).
- **Audit logging**: development decisions are recorded in `aidlc-docs/audit.md`, and git history is the change record.
- **Vulnerability scanning**: `pip-audit` in CI on every push; GitHub Dependabot alerts enabled.

### Dependency Security

- **Dependency scanning**: `pip-audit` against the locked dependency set on every push; Dependabot weekly.
- **License policy**: MIT, BSD, and Apache 2.0 allowed. GPL and AGPL prohibited.
- **Update policy**: a critical or high vulnerability in a direct dependency is fixed before the next workflow stage approval. Docker base image tags are bumped by a reviewed commit, never `latest`.

### Security Compliance Framework

- **Frameworks chosen**: OWASP Top 10 (2021) and OWASP Top 10 for LLM Applications (2025).
- **Rationale**: the product is a web application (OWASP Top 10), and its core risks come from an LLM agent reading untrusted catalog text and student chat (OWASP LLM Top 10).

#### OWASP Top 10 (2021)

| Category | How the Project Addresses It |
|----------|------------------------------|
| A01 Broken Access Control | No accounts. State is isolated per session by an unguessable cookie ID. No resource IDs are exposed across sessions. The app is reachable only from `127.0.0.1`. |
| A02 Cryptographic Failures | No stored sensitive data. Hosted LLM calls use HTTPS. Tokens come from `secrets`. |
| A03 Injection | SQLite queries are parameterized only (no string formatting). FTS5 query terms are escaped. Jinja2 autoescaping is on. No `eval` or `exec`. |
| A04 Insecure Design | Deterministic validator gates every plan. Grounding check on every course code. Misuse case: prompt injection (see LLM01). |
| A05 Security Misconfiguration | Security headers, no debug mode in Docker, no default credentials, no sample routes, non-root container user. |
| A06 Vulnerable and Outdated Components | `uv.lock` pinning, `pip-audit` in CI, Dependabot, pinned base image tag. |
| A07 Identification and Authentication Failures | Not applicable: no authentication. Session IDs are 256-bit random values in `HttpOnly`, `SameSite=Strict` cookies. |
| A08 Software and Data Integrity Failures | Snapshot loaded through Pydantic models. No pickle. HTMX vendored with its checksum recorded in `static/vendor/README.md`. |
| A09 Security Logging and Monitoring Failures | Structured JSON logs of tool calls, retries, validation failures, and fabrication events, with no personal content. |
| A10 Server-Side Request Forgery | Outbound URLs come only from `Settings` (LLM endpoint) and the fixed Class Roster base URL. Users and the LLM can never supply a URL to fetch. |

#### OWASP Top 10 for LLM Applications (2025)

| Category | How the Project Addresses It |
|----------|------------------------------|
| LLM01 Prompt Injection | Catalog text and student chat are passed as clearly delimited data, never as instructions. Injected text cannot add courses (grounding check) or skip checks (deterministic validator). Tools are read-only. |
| LLM02 Sensitive Information Disclosure | No secrets or other students' data are ever in context. Profile content is excluded from logs. |
| LLM03 Supply Chain | Models pulled by exact tag from the Ollama library. Embedding model pinned by name and baked into the image. Python dependencies locked and audited. |
| LLM04 Data and Model Poisoning | Catalog data comes only from the official Class Roster API. LLM-parsed prerequisite rules must pass automated checks or are marked `unverified`. |
| LLM05 Improper Output Handling | All consumed output is validated by Pydantic. LLM text is autoescaped when rendered. No output is executed or used in SQL. |
| LLM06 Excessive Agency | Tools only search, read, and validate. No tool writes files, sends network requests, or changes the catalog. |
| LLM07 System Prompt Leakage | Prompts contain no secrets or access rules. Security is enforced in code, not in prompts. |
| LLM08 Vector and Embedding Weaknesses | Index built only from the committed snapshot. One embedding model per index, checked at startup. Vector hits are re-checked against SQLite. |
| LLM09 Misinformation | Grounding check, prerequisite verdicts with the original text shown, an "advisory only" notice on every plan, and official course links. Accuracy measured by the evaluation runner. |
| LLM10 Unbounded Consumption | Per-session rate limit, 60-second call timeout, 3-attempt retry cap, bounded graph recursion (`recursion_limit=25`), and input length limits. |

---

## Testing Requirements

### Test Strategy Overview

| Test Type | Required | Coverage Target | Tooling |
|-----------|----------|----------------|---------|
| Unit tests | Yes | 90% line coverage on `catalog`, `prereq`, `planner`; 70% overall | pytest, pytest-cov |
| Property-based tests | Yes, for every deterministic component | Every invariant listed below | Hypothesis |
| Integration tests | Yes | Every route, including security headers and the error handler | pytest + FastAPI `TestClient`, fake chat model |
| Real-model tests | Conditional (manual, marked) | Agent tool-calling smoke checks | pytest marker `llm` against local Ollama |
| End-to-end browser tests | No | Not applicable. The demo video covers the two MVP journeys. | None |
| Product evaluation | Yes | Vision success metrics | `uv run python scripts/run_evaluation.py` writes `evaluation/reports/YYYY-MM-DD.md` |
| Static checks | Yes | Whole codebase | ruff, mypy |

### Unit Testing Standards

- **Coverage minimum**: 90% line coverage for `src/course_planner/{catalog,prereq,planner}`, checked with `coverage report --include=... --fail-under=90`. 70% overall (`--cov-fail-under=70`).
- **Mocking policy**:
  - Mock only the LLM (fake chat model with scripted tool calls) and the network (`httpx.MockTransport`).
  - Never mock the prerequisite evaluator, validator, fixer, or SQLite. Tests use a small fixture catalog built in a temporary directory.
- **Naming**: `test_<unit>_<behaviour>`, for example `test_evaluate_any_of_met_when_one_branch_met`.
- **Location**: separate tree mirroring the package: `tests/unit/prereq/test_evaluator.py`, `tests/property/prereq/test_evaluator_properties.py`.

### Property-Based Testing Standards

Hypothesis is mandatory for every deterministic component. Required invariants:

- **Prerequisite evaluator**:
  - having taken every course named in a rule never gives `Not met`
  - missing courses are always a subset of the courses the rule names
  - adding completed courses never turns `Met` into `Not met`
  - confirming clauses never turns `Met` into `Not met` or `Needs review` (CR-1, FR-1.9)
  - a verdict is marked `student_confirmed` only when at least one clause was confirmed (CR-1, FR-5.11)
- **Checklist builder** (CR-1): the items are exactly the course codes and non-course clauses named in the rule, with no duplicates (FR-1.9.2).
- **Plan validator and fixer**:
  - the fixer's output always passes the validator
  - every placed course comes after its prerequisites
  - every term stays within its credit range
  - no course is placed in an unavailable term
- **Rule model**: JSON round-trip (`Rule → JSON → Rule`) is lossless.
- **Course-code normalizer**: idempotent; output always matches `^[A-Z]{2,5} \d{4}$` or raises.
- **Hybrid merge**: output contains only IDs from the inputs, with no duplicates.

Hypothesis profiles: `ci` (`max_examples=200`, `derandomize=True`) and `dev` (`max_examples=50`), selected by `HYPOTHESIS_PROFILE`.

### Integration Testing Standards

- **Scope**: FastAPI routes through `TestClient`, with the real SQLite and Chroma built from a fixture snapshot of about 30 courses, and the fake chat model.
- **Environment**: local, no Docker and no Ollama needed.
- **Data management**: fixture snapshot in `tests/fixtures/snapshot/`; indexes built into `tmp_path` per test session.

### Real-Model Testing Standards

- Marked `@pytest.mark.llm`. Excluded by default (`addopts = "-m 'not llm'"`).
- Run by hand before agent-related stage approvals with `uv run pytest -m llm` while Ollama is running.

### Performance Standards

- **Target**: time to a first plan under 90 seconds for the median evaluation profile, on the team's fastest laptop, using the primary model.
- **Measurement**: the evaluation runner records per-profile timing and tool-call counts in its report.

### CI/CD Testing Gates

The team commits directly to `main`. GitHub Actions (`.github/workflows/ci.yml`) runs on every push:

| Pipeline Stage | Required Checks | Failure Action |
|---------------|---------------|----------------|
| Every push to `main` | `uv sync --frozen`; `ruff check`; `ruff format --check`; `mypy`; `pytest -m "not llm"` with coverage gates; `pip-audit` | Red build is fixed before the next workflow stage approval. No new feature work lands on a red build. |
| Before each Construction stage approval | Full CI green, plus `pytest -m llm` and the evaluation runner on one laptop with Ollama | Approval is withheld until the results are recorded |
| Deployment | Not applicable: no deployment | — |

---

## Example and Template Code Guidance

### Purpose of Example Code

These examples are the canonical patterns for this project. Generated code must follow their structure, naming, validation, error handling, and testing style rather than inventing alternatives. Examples live in this document, and the first unit to implement each pattern places the working version at the path shown.

### Example 1: Endpoint (FastAPI route returning an HTMX fragment)

`src/course_planner/web/routes/profile.py`

```python
from typing import Annotated

from fastapi import APIRouter, Depends, Form, Request
from fastapi.responses import HTMLResponse
from pydantic import BaseModel, ConfigDict, Field

from course_planner.catalog.codes import InvalidCourseCodeError, normalize_course_codes
from course_planner.catalog.store import CatalogStore, get_catalog
from course_planner.web.security import verify_csrf
from course_planner.web.session import SessionState, get_session
from course_planner.web.templating import templates

router = APIRouter()


class CompletedCoursesForm(BaseModel):
    model_config = ConfigDict(extra="forbid")

    csrf_token: str = Field(min_length=32, max_length=128)
    courses: str = Field(max_length=1_200)


@router.post("/profile/completed-courses", response_class=HTMLResponse)
async def update_completed_courses(
    request: Request,
    form: Annotated[CompletedCoursesForm, Form()],
    session: Annotated[SessionState, Depends(get_session)],
    catalog: Annotated[CatalogStore, Depends(get_catalog)],
) -> HTMLResponse:
    verify_csrf(session, form.csrf_token)
    try:
        codes = normalize_course_codes(form.courses, max_codes=80)
    except InvalidCourseCodeError as exc:
        return templates.TemplateResponse(
            request,
            "partials/error.html",
            {"message": exc.public_message},
            status_code=422,
        )

    unknown = [code for code in codes if not catalog.has_course(code)]
    session.profile.completed = [code for code in codes if code not in unknown]
    return templates.TemplateResponse(
        request,
        "partials/completed_courses.html",
        {"completed": session.profile.completed, "unknown": unknown},
    )
```

Rules shown:
- a Pydantic form model with `extra="forbid"` and length limits
- CSRF check first
- domain errors become a generic 422 fragment
- unknown codes are reported, not silently dropped
- student data goes only into the in-memory session

### Example 2: Function (deterministic prerequisite evaluator)

`src/course_planner/prereq/evaluator.py`

```python
from enum import StrEnum
from typing import Annotated, Literal, NamedTuple

from pydantic import BaseModel, Field


class Status(StrEnum):
    MET = "Met"
    NOT_MET = "Not met"
    NEEDS_REVIEW = "Needs review"


class CourseReq(BaseModel):
    kind: Literal["course"] = "course"
    code: str


class ReviewFlag(BaseModel):
    """A clause code cannot decide, e.g. 'permission of instructor' or 'or equivalent'."""

    kind: Literal["review"] = "review"
    reason: str
    clause_id: str  # CR-1: the checklist item the student can confirm for this course


class AllOf(BaseModel):
    kind: Literal["all"] = "all"
    items: list["Rule"] = Field(min_length=1)


class AnyOf(BaseModel):
    kind: Literal["any"] = "any"
    items: list["Rule"] = Field(min_length=1)


Rule = Annotated[CourseReq | ReviewFlag | AllOf | AnyOf, Field(discriminator="kind")]
AllOf.model_rebuild()
AnyOf.model_rebuild()


class Verdict(BaseModel):
    status: Status
    missing: frozenset[str] = frozenset()
    reasons: tuple[str, ...] = ()
    original_text: str
    student_confirmed: bool = False  # CR-1: Met that relies on a confirmed clause


class _Result(NamedTuple):
    status: Status
    missing: frozenset[str] = frozenset()
    reasons: tuple[str, ...] = ()
    confirmed: bool = False  # Met only because of a student-confirmed clause


def evaluate(
    rule: Rule | None,
    taken: frozenset[str],
    original_text: str,
    verified: bool = True,
    confirmed_clauses: frozenset[str] = frozenset(),
) -> Verdict:
    """Evaluate a stored prerequisite rule against completed and earlier-planned courses.

    Args:
        rule: Parsed rule, or None when the course has no prerequisites.
        taken: Normalized codes of completed, in-progress, and earlier-term courses.
        original_text: Catalog prerequisite text, always returned for display.
        verified: False when the LLM parse failed the automated checks.
        confirmed_clauses: Clause IDs the student confirmed for this course only (FR-1.9.4).

    Returns:
        Verdict with status, missing courses, reasons, and the student-confirmed marker.
    """
    if not verified:
        return Verdict(
            status=Status.NEEDS_REVIEW,
            reasons=("unverified parse",),
            original_text=original_text,
        )
    if rule is None:
        return Verdict(status=Status.MET, original_text=original_text)
    result = _evaluate(rule, taken, confirmed_clauses)
    return Verdict(
        status=result.status,
        missing=result.missing,
        reasons=result.reasons,
        original_text=original_text,
        student_confirmed=result.confirmed,
    )


def _evaluate(rule: Rule, taken: frozenset[str], confirmed_clauses: frozenset[str]) -> _Result:
    if isinstance(rule, CourseReq):
        if rule.code in taken:
            return _Result(Status.MET)
        return _Result(Status.NOT_MET, frozenset({rule.code}))
    if isinstance(rule, ReviewFlag):
        if rule.clause_id in confirmed_clauses:
            return _Result(Status.MET, confirmed=True)
        return _Result(Status.NEEDS_REVIEW, reasons=(rule.reason,))

    results = [_evaluate(item, taken, confirmed_clauses) for item in rule.items]
    reasons = tuple(reason for result in results for reason in result.reasons)
    met = [result for result in results if result.status is Status.MET]

    if isinstance(rule, AllOf):
        not_met = [result.missing for result in results if result.status is Status.NOT_MET]
        if not_met:
            return _Result(Status.NOT_MET, frozenset().union(*not_met), reasons)
        if any(result.status is Status.NEEDS_REVIEW for result in results):
            return _Result(Status.NEEDS_REVIEW, reasons=reasons)
        return _Result(Status.MET, confirmed=any(result.confirmed for result in met))

    if met:
        # Prefer a branch the student met without relying on a confirmed clause.
        return _Result(Status.MET, confirmed=all(result.confirmed for result in met))
    if any(result.status is Status.NEEDS_REVIEW for result in results):
        return _Result(Status.NEEDS_REVIEW, reasons=reasons)
    cheapest = min((result.missing for result in results), key=lambda m: (len(m), sorted(m)))
    return _Result(Status.NOT_MET, cheapest, reasons)
```

Rules shown:
- a pure function with no I/O and no LLM
- Pydantic models for stored data
- full type hints and a Google-style docstring
- the original text always travels with the verdict
- deterministic tie-breaking
- confirmed clauses apply only to the course being checked; the caller passes that course's clause IDs (CR-1, FR-1.9.4)
- "Met (student-confirmed)" is a marker on Met, not a fourth status (CR-1, FR-5.11)
- an unverified parse stays "Needs review" even when clauses are confirmed (FR-5.7)

### Example 3: Test (example-based and property-based)

`tests/unit/prereq/test_evaluator.py` and `tests/property/prereq/test_evaluator_properties.py`

```python
from hypothesis import given
from hypothesis import strategies as st

from course_planner.prereq.evaluator import (
    AllOf,
    AnyOf,
    CourseReq,
    ReviewFlag,
    Rule,
    Status,
    evaluate,
)

TEXT = "CS 2110 and (MATH 1920 or MATH 2220)."
RULE = AllOf(
    items=[
        CourseReq(code="CS 2110"),
        AnyOf(items=[CourseReq(code="MATH 1920"), CourseReq(code="MATH 2220")]),
    ]
)


def test_evaluate_met_when_one_or_branch_taken() -> None:
    verdict = evaluate(RULE, frozenset({"CS 2110", "MATH 2220"}), TEXT)
    assert verdict.status is Status.MET
    assert verdict.original_text == TEXT


def test_evaluate_not_met_lists_missing_course() -> None:
    verdict = evaluate(RULE, frozenset({"MATH 1920"}), TEXT)
    assert verdict.status is Status.NOT_MET
    assert verdict.missing == frozenset({"CS 2110"})


def test_evaluate_unverified_parse_needs_review() -> None:
    verdict = evaluate(RULE, frozenset({"CS 2110", "MATH 1920"}), TEXT, verified=False)
    assert verdict.status is Status.NEEDS_REVIEW
    assert "unverified parse" in verdict.reasons


POI_TEXT = "CS 2110 or permission of instructor."
POI_RULE = AnyOf(
    items=[
        CourseReq(code="CS 2110"),
        ReviewFlag(reason="permission of instructor", clause_id="poi"),
    ]
)


def test_confirmed_clause_gives_student_confirmed_met() -> None:
    verdict = evaluate(POI_RULE, frozenset(), POI_TEXT, confirmed_clauses=frozenset({"poi"}))
    assert verdict.status is Status.MET
    assert verdict.student_confirmed


def test_course_branch_is_preferred_over_confirmed_clause() -> None:
    verdict = evaluate(POI_RULE, frozenset({"CS 2110"}), POI_TEXT, confirmed_clauses=frozenset({"poi"}))
    assert verdict.status is Status.MET
    assert not verdict.student_confirmed


codes = st.from_regex(r"[A-Z]{2,4} [1-6][0-9]{3}", fullmatch=True)
clause_ids = st.sampled_from(["poi", "equivalent", "linear-algebra"])
leaves = codes.map(lambda c: CourseReq(code=c)) | clause_ids.map(
    lambda c: ReviewFlag(reason=c, clause_id=c)
)
rules: st.SearchStrategy[Rule] = st.recursive(
    leaves,
    lambda children: st.lists(children, min_size=1, max_size=3).flatmap(
        lambda items: st.sampled_from([AllOf(items=items), AnyOf(items=items)])
    ),
    max_leaves=12,
)


def mentioned(rule: Rule) -> frozenset[str]:
    if isinstance(rule, CourseReq):
        return frozenset({rule.code})
    if isinstance(rule, ReviewFlag):
        return frozenset()
    return frozenset().union(*(mentioned(item) for item in rule.items))


@given(rule=rules)
def test_taking_every_named_course_is_never_not_met(rule: Rule) -> None:
    assert evaluate(rule, mentioned(rule), "text").status is not Status.NOT_MET


@given(rule=rules, taken=st.frozensets(codes, max_size=6))
def test_missing_courses_are_named_and_not_taken(rule: Rule, taken: frozenset[str]) -> None:
    verdict = evaluate(rule, taken, "text")
    assert verdict.missing <= mentioned(rule)
    assert verdict.missing.isdisjoint(taken)


@given(rule=rules, taken=st.frozensets(codes, max_size=6), extra=st.frozensets(codes, max_size=6))
def test_more_courses_never_turn_met_into_not_met(
    rule: Rule, taken: frozenset[str], extra: frozenset[str]
) -> None:
    if evaluate(rule, taken, "text").status is Status.MET:
        assert evaluate(rule, taken | extra, "text").status is Status.MET


@given(rule=rules, taken=st.frozensets(codes, max_size=6), confirmed=st.frozensets(clause_ids))
def test_confirming_clauses_never_breaks_met(
    rule: Rule, taken: frozenset[str], confirmed: frozenset[str]
) -> None:
    if evaluate(rule, taken, "text").status is Status.MET:
        assert evaluate(rule, taken, "text", confirmed_clauses=confirmed).status is Status.MET


@given(rule=rules, taken=st.frozensets(codes, max_size=6))
def test_no_confirmed_clauses_is_never_student_confirmed(rule: Rule, taken: frozenset[str]) -> None:
    assert not evaluate(rule, taken, "text").student_confirmed
```

Rules shown:
- example tests pin known catalog cases
- Hypothesis properties state the invariants
- no mocks of the evaluator
- test names say the behaviour

### Example 4: Agent Tool (grounded catalog search)

`src/course_planner/agent/tools.py`

```python
from langchain_core.tools import BaseTool, tool

from course_planner.catalog.search import HybridSearch


def make_search_courses_tool(search: HybridSearch, min_level: int, max_level: int) -> BaseTool:
    """Build the catalog search tool bound to this session's level range."""

    @tool
    def search_courses(skill: str, limit: int = 8) -> list[dict[str, str | int]]:
        """Search the loaded Cornell catalog for courses that teach a skill.

        Returns only courses that exist in the local snapshot.
        """
        hits = search.query(
            skill[:200],
            top_k=max(1, min(limit, 15)),
            min_level=min_level,
            max_level=max_level,
        )
        return [
            {"code": h.code, "title": h.title, "credits": h.credits_max, "url": h.roster_url}
            for h in hits
        ]

    return search_courses
```

Rules shown:
- tools are read-only
- inputs are clamped
- results come only from the snapshot
- session-specific limits are bound in a factory, not passed by the LLM

### Example 5: Configuration

`src/course_planner/config.py`

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", extra="ignore")

    llm_base_url: str = "http://host.docker.internal:11434/v1"
    llm_model: str = "qwen3:8b"
    llm_fallback_model: str = "qwen3:4b"
    llm_use_fallback: bool = False
    llm_api_key: str = "ollama"
    llm_thinking_interactive: bool = False
    llm_thinking_ingestion: bool = True
    embedding_model: str = "BAAI/bge-small-en-v1.5"
    catalog_subjects: str = "CS,INFO,ORIE,STSCI,MATH,ECE,BTRY,ECON"
    log_level: str = "INFO"

    @property
    def active_model(self) -> str:
        return self.llm_fallback_model if self.llm_use_fallback else self.llm_model
```

### How AI-DLC Uses These Examples

1. **Read the examples first.** Before generating code for a unit, read the examples relevant to it.
2. **Follow the established patterns.** Match the structure, naming, validation, error handling, and test style shown.
3. **Do not invent alternatives.** If an example covers a pattern, use it unless it clearly does not apply, and raise any exception through the workflow.
4. **Reference examples in plans.** Code Generation plans name the example each step follows.

### Maintaining Example Code

- When a standard in this document changes, update the matching example in the same commit.
- If a pattern is replaced, mark the old example "Deprecated" and point to its replacement.
