# Components

**Stage**: INCEPTION - Application Design
**Sources**: `requirements.md` (amended by CR-1 and C3:A), `stories.md`, `application-design-plan.md` (answers A1 to C3), `docs/technical-environment.md` (binding)

## Architecture in One Paragraph
A **modular monolith** (B1:A): one Python 3.12 package, `src/course_planner/`, running as one FastAPI process in one Docker Compose service. Ingestion, index build, and evaluation are command-line scripts in `scripts/` that use the same package. The package has seven modules with fixed import rules (technical environment, Module boundaries). The **deterministic core** (`catalog`, `prereq`, `planner`) never imports `agent`, LangChain, LangGraph, or OpenAI clients (NFR-7.1, B9:A). The `agent` module owns every LLM call, the LangGraph graph (B2:C), the read-only tools (B3:A), and the service layer that `web` and `evaluation` both call (B10:A).

## Module Overview

| Module | Layer (B12:A) | Components |
|---|---|---|
| `config` | Shared | C1 Settings |
| `catalog` | Data | C2 Roster Client, C3 Snapshot Store, C4 Index Builder, C5 Catalog Store, C6 Course Codes, C7 Hybrid Search, C8 Grounding Check |
| `prereq` | Rules | C9 Rule Model, C10 Pattern Parser, C11 Parse Checks, C12 Prerequisite Evaluator, C13 Chain Builder, C14 Checklist Builder |
| `planner` | Rules | C15 Planning Models, C16 Offering and Credit Rules, C17 Plan Validator, C18 Plan Fixer, C19 Recommendation Ranker, C20 Plan Editor |
| `agent` | Intelligence | C21 LLM Gateway, C22 Prompt Library, C23 Skill Mapper, C24 Ingestion Parser, C25 Agent Tools, C26 Planning Graph, C27 Services |
| `web` | Interface | C28 Session Store, C29 Security Middleware, C30 Routes and Templates, C31 Health Checks, C32 Error Handler |
| `evaluation` | Quality | C33 Evaluation Cases, C34 Evaluation Runner |

---

## `config`

### C1 Settings
- **Purpose**: One typed configuration object, read from `.env` at startup.
- **Responsibilities**:
  - Load and validate every setting in the technical environment's Configuration table (model names, fallback switch, thinking switches, embedding model, subjects, log level)
  - Expose the active model (primary or fallback, NFR-6.3)
  - Fail at startup with a clear message on an invalid value
- **Interface**: `Settings` object; `get_settings()`
- **Stories**: US-10.4

---

## `catalog` (Data layer)

### C2 Roster Client
- **Purpose**: Fetch raw roster data from the Class Roster API. Used only by the ingestion script, never at request time (NFR-3.4).
- **Responsibilities**: list terms and subjects; fetch course data per term and subject; throttle to at most 1 request per second; 10-second timeout; retry with a fixed attempt count; resume from what is already on disk (FR-3.4)
- **Interface**: `RosterClient`
- **Stories**: US-10.1

### C3 Snapshot Store
- **Purpose**: Read and write the committed source of truth: `data/snapshot/rosters/{TERM}/{SUBJECT}.json` and `data/snapshot/prereq_rules.json` (B6:C).
- **Responsibilities**: save raw responses; load them as Pydantic records; save and load parsed rules with `verified` status and original text
- **Interface**: `SnapshotStore`
- **Stories**: US-10.1

### C4 Index Builder
- **Purpose**: Build the git-ignored runtime indexes from the snapshot with one command, `scripts/build_index.py` (B5:A, B6:C).
- **Responsibilities**: create `data/index/catalog.sqlite3` (courses, offerings, rules, FTS5 table); embed course text with FastEmbed and load the Chroma collection `courses`; record `embedding_model` in the collection metadata
- **Interface**: `IndexBuilder`
- **Stories**: US-10.1, US-10.4

### C5 Catalog Store
- **Purpose**: Read-only access to courses, offering history, and parsed rules in SQLite at runtime.
- **Responsibilities**: look up a course; check that a code exists; return offering terms; return a course's rule and original prerequisite text; list forbidden overlaps; parameterized queries only
- **Interface**: `CatalogStore`
- **Stories**: US-3.1, US-4.1, US-5.1, US-6.2, US-9.1

### C6 Course Codes
- **Purpose**: Normalize and validate course codes everywhere they enter the system.
- **Responsibilities**: normalize case and spacing; enforce `^[A-Z]{2,5} \d{4}$`; parse pasted lists (at most 80 codes); extract codes from free text (used by Parse Checks and the Checklist Builder); derive course level
- **Interface**: `normalize_course_code`, `normalize_course_codes`, `extract_codes`, `course_level`
- **Stories**: US-1.2, US-1.4

### C7 Hybrid Search
- **Purpose**: Find catalog courses for a skill (FR-3.5).
- **Responsibilities**: FTS5 BM25 keyword search; Chroma vector search with FastEmbed query embeddings; Reciprocal Rank Fusion merge; level filter by student type (FR-3.7); re-check every vector hit against SQLite
- **Interface**: `HybridSearch`
- **Stories**: US-3.1, US-4.1

### C8 Grounding Check
- **Purpose**: Guarantee that no course code outside the snapshot reaches the student (FR-4.5, NFR-4.8).
- **Responsibilities**: check every course code in any LLM-derived output; drop unknown codes; emit a fabrication event to the log
- **Interface**: `GroundingCheck`
- **Stories**: US-3.1, US-11.4

---

## `prereq` (Rules layer)

### C9 Rule Model
- **Purpose**: The stored shape of a parsed prerequisite or corequisite rule.
- **Responsibilities**: Pydantic models `CourseReq`, `ReviewFlag`, `AllOf`, `AnyOf` (technical environment, Example 2); `StoredRule` with course code, kind (prerequisite or corequisite), original text, rule, `verified` flag, parse method (pattern or LLM), and assumption notes (FR-5.6); lossless JSON round trip
- **Interface**: Pydantic models
- **Stories**: US-5.1, US-10.1

### C10 Pattern Parser
- **Purpose**: Parse simple prerequisite text deterministically (FR-5.2).
- **Responsibilities**: recognize single course, "A or B", "A and B", comma lists, and the documented default "and/or" reading (FR-5.6); return no result when the text is not a known pattern, so it goes to the LLM
- **Interface**: `parse_simple`
- **Stories**: US-5.1, US-10.1

### C11 Parse Checks
- **Purpose**: Decide whether an LLM-parsed rule can be trusted (FR-5.3).
- **Responsibilities**: course-code cross check against the original text; catalog existence of each code; keyword flags ("equivalent", "permission", "recommended", "or"/"and" grouping) must appear as ReviewFlags or groups; agreement between two independent parses. Any failure gives `verified=False`.
- **Interface**: `check_llm_parse`
- **Stories**: US-5.2, US-10.1

### C12 Prerequisite Evaluator
- **Purpose**: Turn a stored rule plus the student's history into a verdict, with no I/O and no LLM (FR-5.4).
- **Responsibilities**: return Met, Not met (with missing courses), or Needs review (with reasons); always carry the original text; unverified rules give "Needs review (unverified parse)" (FR-5.7); treat ReviewFlags the student confirmed for this course as satisfied and mark the verdict `student_confirmed` (CR-1, FR-5.11); deterministic tie-breaking
- **Interface**: `evaluate`
- **Stories**: US-5.1, US-5.2, US-5.3, US-1.4

### C13 Chain Builder
- **Purpose**: Build the full prerequisite chain for a target course (FR-5.9).
- **Responsibilities**: walk rules recursively down to completed courses or courses without prerequisites; detect cycles; return a tree with the shortest missing path marked
- **Interface**: `build_chain`
- **Stories**: US-5.3

### C14 Checklist Builder *(CR-1)*
- **Purpose**: Turn a course's rule into the checklist the student sees (FR-1.9.2).
- **Responsibilities**: list every course code and every non-course clause as separate items with stable item IDs; mark items already satisfied; for unverified rules, list the codes found in the original text only (FR-5.7); flag codes outside the loaded catalog (C2:A)
- **Interface**: `build_checklist`
- **Stories**: US-1.4

---

## `planner` (Rules layer)

### C15 Planning Models
- **Purpose**: Shared, validated data shapes for profile and plan.
- **Responsibilities**: `StudentProfile` (type, program, credit range, completed, in-progress, fixed courses, unavailable terms, interests, topics to avoid, skills, and the CR-1 `ConfirmedPrerequisites`); `Term`; `Plan`; `PlannedCourse`; `Violation`; `PlanChange` (structured edit); `ProfileChange`
- **Interface**: Pydantic models
- **Stories**: US-1.1 to US-1.4, US-6.1, US-7.1, US-7.3

### C16 Offering and Credit Rules
- **Purpose**: The term-level rules the validator and fixer share.
- **Responsibilities**: plan horizon Spring 2027 to at most Fall 2028 (FR-6.1); offering-term check and "uncertain" flag (FR-6.4); credit totals per term; forbidden-overlap check (FR-5.10)
- **Interface**: `plan_terms`, `offering_status`, `term_credits`, `overlaps`
- **Stories**: US-6.1, US-6.2

### C17 Plan Validator
- **Purpose**: The single gate every plan passes before display (FR-6.5, FR-6.7).
- **Responsibilities**: check prerequisite order (using C12 with the courses of earlier terms), corequisites, credit range, forbidden overlaps, unavailable terms, and offering terms; return a list of violations
- **Interface**: `validate_plan`
- **Stories**: US-6.1, US-7.1, US-1.4

### C18 Plan Fixer
- **Purpose**: Repair any violations left after the LLM retries (FR-6.3).
- **Responsibilities**: move or drop courses deterministically until the validator passes; report what it changed; report an unfixable conflict for FR-6.6 instead of returning an invalid plan
- **Interface**: `fix_plan`
- **Stories**: US-6.1, US-6.3

### C19 Recommendation Ranker
- **Purpose**: Rank candidate courses deterministically (FR-4.3; decision recorded in the plan).
- **Responsibilities**: score by skill coverage from search hits plus prerequisite reachability (number of missing courses); lower the rank for topics to avoid; exclude completed and in-progress courses and their forbidden overlaps (FR-4.4)
- **Interface**: `rank_candidates`
- **Stories**: US-4.1, US-5.3

### C20 Plan Editor
- **Purpose**: Apply a structured change to a plan or profile deterministically (B4:A).
- **Responsibilities**: apply a `PlanChange` (swap, move, add, remove, change credits, block term) or a `ProfileChange`, including CR-1 confirmations and removals; keep unaffected courses in place (FR-7.4); return the changed plan and a list of what changed, for validation
- **Interface**: `apply_plan_change`, `apply_profile_change`
- **Stories**: US-7.1, US-7.3, US-1.4

---

## `agent` (Intelligence layer)

### C21 LLM Gateway
- **Purpose**: The only place the app talks to a language model (B9:A).
- **Responsibilities**: build the OpenAI-compatible client from Settings (Ollama, fallback, or hosted with `LLM_API_KEY`); thinking-mode switch; 60-second timeout; 3 attempts with `tenacity`; structured output validated against a Pydantic model (invalid output counts as a failed attempt); timing and retry logs with no content
- **Interface**: `LLMGateway`
- **Stories**: US-10.4, US-11.3

### C22 Prompt Library
- **Purpose**: Versioned prompt text files in `agent/prompts/`.
- **Responsibilities**: load prompts by name and version; wrap catalog text and student text as delimited data, never as instructions (LLM01)
- **Interface**: `load_prompt`
- **Stories**: US-11.4

### C23 Skill Mapper
- **Purpose**: Career goal to 6 to 12 skills (FR-2).
- **Responsibilities**: load the committed reviewed mappings JSON (B11:A); generate a live mapping when no reviewed one exists, labeled "AI-generated"; detect a vague goal and produce guided follow-up questions (FR-2.2); compare 2 to 3 careers (FR-2.6); mark skills covered by completed courses (FR-2.7); provide the command that drafts mappings for team review
- **Interface**: `SkillMapper`
- **Stories**: US-2.1, US-2.2, US-2.4

### C24 Ingestion Parser
- **Purpose**: LLM parsing of complex prerequisite text at load time (FR-5.2).
- **Responsibilities**: produce two independent structured parses per text, with thinking mode on; hand both to C11 Parse Checks (which lives in `prereq`)
- **Interface**: `IngestionParser`
- **Stories**: US-10.1

### C25 Agent Tools
- **Purpose**: The tools the revision agent node may call (B3:A).
- **Responsibilities**: `search_courses`, `get_course`, `check_prerequisites`, `get_prerequisite_chain`, `validate_plan`, `explain_course`, and the two proposal tools `propose_plan_change` and `propose_profile_change`. Every tool is read-only (LLM06). Proposal tools return a structured change and never modify state. Session-specific limits (level range, profile) are bound in a factory (technical environment, Example 4).
- **Interface**: `make_tools(session_context)`
- **Stories**: US-7.1 to US-7.4

### C26 Planning Graph
- **Purpose**: LangGraph state graph (B2:C).
- **Responsibilities**: fixed nodes for the first plan: map skills, search, rank, check prerequisites, propose plan, validate, retry or fix, explain; one agent node for chat revision and "why" questions; conditional edges for "plan does not fit" (FR-6.6) and for apply-and-validate after a proposal; `InMemorySaver` with `thread_id` = session ID; `recursion_limit=25`
- **Interface**: `build_planning_graph`, `build_revision_graph`
- **Stories**: US-6.1, US-6.3, US-7.1 to US-7.4

### C27 Services
- **Purpose**: The service layer that `web` and `evaluation` call (B10:A). Details in `services.md`.
- **Responsibilities**: one entry point per student task; owns the order of grounding check, validation, and fixer; returns view models for templates
- **Interface**: `CareerService`, `ProfileService`, `PlanningService`, `RevisionService`, `ExportService`, `HealthService`
- **Stories**: all student stories

---

## `web` (Interface layer)

### C28 Session Store
- **Purpose**: Hold each student's state in memory only (B7:A, FR-1.7).
- **Responsibilities**: create a 256-bit session ID cookie (`HttpOnly`, `SameSite=Strict`); map ID to `SessionState` (profile, skills, plan, CSRF token, rate-limit bucket); 60-minute idle timeout; nothing written to disk
- **Interface**: `SessionStore`, `get_session`
- **Stories**: US-11.2

### C29 Security Middleware
- **Purpose**: App-wide request protection.
- **Responsibilities**: security headers (NFR-4.4); request body limit of 64 KB; CSRF check on every POST; 10 chat requests per minute per session (NFR-4.9); request ID assignment for logs
- **Interface**: middleware plus `verify_csrf`
- **Stories**: US-11.4, US-7.1

### C30 Routes and Templates
- **Purpose**: Server-rendered pages and HTMX fragments (B8:A).
- **Responsibilities**: Pydantic form models with `extra="forbid"`; page routes and fragment routes (kebab-case paths); Jinja2 autoescaping; `aria-live` chat region; advisory notice on every plan view (FR-9.1); Class Roster links (FR-9.2); Markdown export as a streamed download
- **Interface**: FastAPI routers
- **Stories**: all student stories; US-11.1

### C31 Health Checks
- **Purpose**: Shallow and deep health (RESILIENCY-06, NFR-3.2).
- **Responsibilities**: `GET /health`; `GET /health/deep` checks Ollama, the SQLite file, the Chroma collection, and the embedding-model match; startup check that refuses to start on an embedding mismatch
- **Interface**: routes calling `HealthService`
- **Stories**: US-11.3, US-10.4

### C32 Error Handler
- **Purpose**: Safe failure everywhere (SECURITY-15).
- **Responsibilities**: global exception handler; generic `partials/error.html` with 4xx or 5xx; no stack traces; log with request ID, error type, and module; keep session state intact (NFR-3.1)
- **Interface**: FastAPI exception handlers
- **Stories**: US-11.3

---

## `evaluation` (Quality layer)

### C33 Evaluation Cases
- **Purpose**: Load and validate the committed evaluation data in `evaluation/`.
- **Responsibilities**: 10 profiles and 40 prerequisite cases as JSON with Pydantic models; reviewer recorded per case (FR-10.2); regression flag (FR-10.4); core-subject check. Cases are generated outside the app by an AI assistant and reviewed by a team member (Q37:D, CQ11:B).
- **Interface**: `load_cases`
- **Stories**: US-10.2

### C34 Evaluation Runner
- **Purpose**: One command for all automated metrics (FR-10.3).
- **Responsibilities**: run every profile through `PlanningService` in process (B10:A) and every prerequisite case through the evaluator; compute fabricated course count, verdict agreement, plan validity, explanation coverage, and time to first plan; flag unreviewed cases; write `evaluation/reports/YYYY-MM-DD.md`
- **Interface**: `scripts/run_evaluation.py` calling `run_evaluation`
- **Stories**: US-10.3

---

## Workload Criticality (RESILIENCY-01)

The product has one deployable workload, the web app container, plus offline scripts and one external runtime dependency (Ollama). The classification below is per component, as the rule requires. "Unavailable" means the component fails or is missing while a student uses the app. There is no revenue or regulatory impact: this is a local course project, and the plan is advisory only.

| Component | Criticality | Impact if unavailable | Upstream dependencies | Downstream dependents |
|---|---|---|---|---|
| C5 Catalog Store, C4 Index Builder output | Critical | No course can be looked up; the app refuses to plan. Recovery: run `build_index.py` (CQ14). | Snapshot (C3), SQLite file | Search, evaluator, validator, grounding, services |
| C12 Evaluator, C17 Validator, C18 Fixer | Critical | No plan can be shown (FR-6.7); prerequisite verdicts missing | Catalog Store, Rule Model | Planning Graph, services, evaluation |
| C8 Grounding Check | Critical | Fabricated courses could reach students (NFR-2.1); output is withheld instead | Catalog Store | Services |
| C21 LLM Gateway plus Ollama | High | No skill mapping, first plan, or chat. App shows "model unavailable" with fix steps; inputs kept (NFR-3.1, NFR-3.2) | Ollama on host, Settings | Skill Mapper, Planning Graph, Ingestion Parser |
| C7 Hybrid Search (Chroma, FastEmbed) | High | No recommendations. Deep health check reports it; startup refuses on embedding mismatch | Index files, FastEmbed model in the image | Planning Graph, Agent Tools |
| C28 Session Store, C29 Security Middleware, C30 Routes | High | App unusable; restarting clears sessions (intended) | Settings | Students |
| C23 Skill Mapper, C19 Ranker, C14 Checklist Builder, C20 Plan Editor | Medium | One feature degrades (for example no checklist or no chat edits); the plan view still works | Gateway, Catalog, prereq | Services |
| C31 Health Checks, C32 Error Handler | Medium | Failures are less clear to the student and developer | All | Students, P5 |
| C2 Roster Client, C24 Ingestion Parser, C3 Snapshot Store writes | Low | Snapshot cannot be refreshed; the app keeps running on the committed snapshot (NFR-3.4) | Class Roster API, Ollama | Index Builder |
| C33 Evaluation Cases, C34 Evaluation Runner | Low | No metrics report; no effect on students | Services, prereq | Team (P5) |
| C30 Markdown export (US-8.1) | Low | Student cannot download; plan still visible | Services | Students, advisors |
