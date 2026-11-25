# Units of Work

**Stage**: INCEPTION - Units Generation
**Owner**: Member 5 (Erick)
**Sources**: `unit-of-work-plan.md` (answers Q1 to Q11, F1, F2), `application-design.md`, `stories.md`, `docs/technical-environment.md`
**Status**: Draft for team approval. The Construction schedule in section 5 is **provisional**: the team will review the Construction plan before Construction starts (Workflow Planning approval).

The application is **one deployable service** (a modular monolith). A unit of work is a group of modules, components, and stories that one member owns through Construction (Q1:A). Units are not separate services.

---

## 1. Team

| # | Member | Inception role | Construction ownership |
|---|---|---|---|
| 1 | Timothy | GitHub and AI-DLC workflow | U3 Planner; keeps the AI-DLC workflow role (`audit.md`, `aidlc-state.md`, final submission) |
| 2 | Addis | README and product proposal | U1 Catalog Data |
| 3 | Carey | Vision and requirements | U2 Prerequisite Engine |
| 4 | Revanth | Technical environment and application design | U4 Agent and Services; Unit 0 model spike; contracts step; shared NFR stages |
| 5 | Erick | User stories, units, and QA | U5 Web and Evaluation; QA review of the shared NFR stages; final rubric check |

Names and numbering follow F1:B. README section 8 currently lists Carey as Member 1 and Timothy as Member 3 and must be corrected by its owner (Member 2) (see section 7).

---

## 2. Units

### U1 Catalog Data — Owner: Addis
- **Purpose**: Provide the course data every other unit reads: the committed snapshot, the runtime indexes, search, and course-code checks.
- **Modules**: `config`, `catalog`
- **Components**: C1 Settings, C2 Roster Client, C3 Snapshot Store, C4 Index Builder, C5 Catalog Store, C6 Course Codes, C7 Hybrid Search, C8 Grounding Check
- **Scripts**: `scripts/ingest_roster.py` (fetch and save; calls U2's Pattern Parser and Parse Checks and U4's Ingestion Parser for rules), `scripts/build_index.py`
- **Also owns**: the shared test fixture catalog in `tests/fixtures/snapshot/` (Q9:A): about 30 real courses from the 8 core subjects, covering the persona and evaluation cases (for example CS 2110, CS 4780, MATH 2940, a course with "permission of instructor", a course offered only in fall)
- **Provides**: `CatalogStore`, `HybridSearch`, `GroundingCheck`, course-code functions, `Settings`, the fixture catalog, the built indexes
- **Consumes**: U2 parser and checks, U4 Ingestion Parser (script level only)
- **Definition of done**:
  - Snapshot of 8 subjects and 4 years committed; `build_index.py` rebuilds SQLite, FTS5, and Chroma in one command
  - Fixture catalog committed and used by U2, U3, U4, U5 tests
  - Property tests for course-code normalization and `rrf_merge`; 90% line coverage on `catalog`
  - Startup embedding-model check works

### U2 Prerequisite Engine — Owner: Carey
- **Purpose**: Turn prerequisite text into trusted rules and turn rules plus a student's history into verdicts, chains, and checklists. No LLM.
- **Module**: `prereq`
- **Components**: C9 Rule Model, C10 Pattern Parser, C11 Parse Checks, C12 Prerequisite Evaluator, C13 Chain Builder, C14 Checklist Builder (CR-1)
- **Provides**: `Rule` models, `StoredRule`, `parse_simple`, `check_llm_parse`, `evaluate` and `Verdict`, `build_chain`, `build_checklist`
- **Consumes**: U1 `CatalogStore` (read only), U1 course-code functions
- **Definition of done**:
  - All Hypothesis invariants in the technical environment for the evaluator and rule model pass, plus the CR-1 property (confirming clauses never turns Met into Not met or Needs review)
  - Verdict agreement measured on the 40 labeled cases (target at least 90%, NFR-2.2) once U5's runner exists
  - 90% line coverage on `prereq`

### U3 Planner — Owner: Timothy
- **Purpose**: The deterministic planning core: models, term rules, ranking, validation, fixing, and applying edits.
- **Module**: `planner`
- **Components**: C15 Planning Models, C16 Offering and Credit Rules, C17 Plan Validator, C18 Plan Fixer, C19 Recommendation Ranker, C20 Plan Editor
- **Provides**: `StudentProfile`, `Plan`, `PlanChange`, `ProfileChange`, `validate_plan`, `fix_plan`, `rank_candidates`, `apply_plan_change`, `apply_profile_change`, term and credit functions
- **Consumes**: U2 evaluator and chain builder, U1 `CatalogStore`
- **Definition of done**:
  - Hypothesis invariants pass: fixer output always passes the validator; prerequisites before dependents; credits within range; nothing in an unavailable term
  - Incremental edits keep unaffected courses in place (FR-7.4)
  - 90% line coverage on `planner`

### U4 Agent and Services — Owner: Revanth
- **Purpose**: All LLM work and the service layer that the web app and evaluation call.
- **Module**: `agent` (including `agent/services/` and `agent/prompts/`)
- **Components**: C21 LLM Gateway, C22 Prompt Library, C23 Skill Mapper, C24 Ingestion Parser (Q2:C), C25 Agent Tools, C26 Planning Graph, C27 Services (except ExportService, see U5)
- **Also owns**:
  - **Unit 0 model spike** (Q4:A), before wave 1
  - **Contracts step** (Q5:A), before wave 1
  - Shared NFR Requirements and NFR Design stages (Q7:A)
  - The committed reviewed skill mappings JSON and the command that drafts them (B11:A)
- **Provides**: `ProfileService`, `CareerService`, `PlanningService`, `RevisionService`, `HealthService`, `IngestionParser`, the planning and revision graphs
- **Consumes**: U1, U2, U3 public interfaces
- **Definition of done**:
  - First plan in under 90 seconds on the median evaluation profile, fastest laptop, primary model (NFR-1.1)
  - 0 fabricated courses across the 10 evaluation profiles (NFR-2.1)
  - Agent tools are read-only; proposals are applied only by U3's Plan Editor and validated
  - Integration tests with a fake chat model pass; `pytest -m llm` smoke checks pass against Ollama

### U5 Web and Evaluation — Owner: Erick
- **Purpose**: The student-facing app and the team's evidence that it meets its targets.
- **Modules**: `web`, `evaluation`
- **Components**: C28 Session Store, C29 Security Middleware, C30 Routes and Templates, C31 Health Checks, C32 Error Handler, C33 Evaluation Cases, C34 Evaluation Runner, and ExportService (Q11:A: built last and dropped first)
- **Scripts**: `scripts/run_evaluation.py`
- **Also owns**: fake services implementing the C27 signatures with fixture data (Q8:A); the 10 evaluation profiles and 40 prerequisite cases with reviewer records (FR-10.2); QA review of the shared NFR stages; the final rubric check
- **Provides**: the running app, the evaluation report, the Dockerfile and `docker-compose.yml` (with U4 for model settings)
- **Consumes**: U4 services (fake until wave 2), U3 models, U1 `Settings`
- **Definition of done**:
  - Every route has integration tests, including security headers, CSRF, body limit, rate limit, and the error handler
  - WCAG 2.1 AA checks on custom elements (NFR-5.1)
  - One command writes the dated evaluation report with all 5 metrics (FR-10.3)
  - Both MVP journeys run end to end against the real services

---

## 3. Code Organization Strategy (Greenfield)

Follows the repository layout in `docs/technical-environment.md` exactly (Q10:A). Each unit owns its directories and their test directories.

```text
.
+-- src/course_planner/
|   +-- config.py                 U1
|   +-- catalog/                  U1
|   +-- prereq/                   U2
|   +-- planner/                  U3
|   +-- agent/                    U4  (services/, prompts/, tools.py, graph.py, gateway.py)
|   +-- web/                      U5  (routes/, templates/, static/)
|   +-- evaluation/               U5
+-- tests/
|   +-- unit/<module>/            owner of <module>
|   +-- property/<module>/        owner of <module>
|   +-- integration/              U5 (routes); U4 (graph with fake model)
|   +-- fixtures/snapshot/        U1
+-- scripts/
|   +-- ingest_roster.py          U1 (calls U2 and U4)
|   +-- build_index.py            U1
|   +-- run_evaluation.py         U5
+-- data/snapshot/                U1 (committed)
+-- data/index/                   built locally, git-ignored
+-- evaluation/                   U5 (profiles, cases, reports)
+-- pyproject.toml, uv.lock       shared; changes through the library approval process
+-- Dockerfile, docker-compose.yml, .env.example    U5 with U4
+-- .github/workflows/ci.yml      U3 owner (workflow role)
```

**Rules**
- Directories under `src/course_planner/` change only through their owner, or with the owner's review at a stage gate.
- Module import rules in `component-dependency.md` are enforced by an import-boundary test (specified in the shared NFR Design).
- `uv.lock` and dependency changes follow the technical environment's library approval process.

---

## 4. Pre-Wave Steps

### Unit 0: Model Spike (Q4:A) — Revanth, timeboxed to 1 day
- One structured-output call and one tool call through Ollama with `qwen3:8b`, thinking off, timed on the team's fastest laptop.
- Record: median latency per call, tool-call success rate over 20 runs, structured-output validity rate.
- **Decision rule**: if the measured time for the expected number of first-plan LLM calls exceeds the 90-second budget, U4's Functional Design moves more steps into deterministic code (requirements section 7 assumption) before wave 2 starts.
- Result recorded in `audit.md` and in U4's Functional Design.

### Contracts Step (Q5:A) — Revanth, reviewed by all
- Types only, no logic: C9 Rule Model, C12 `Verdict` and `Status`, C15 Planning Models, C27 service signatures and view models.
- Reviewed by the whole team at one stage gate. After that, the owning unit (U2, U3, U4) maintains each contract, and changes go through the AI-DLC workflow.
- Also updates Example 2 in `docs/technical-environment.md` for CR-1 (open item from Application Design).

---

## 5. Construction Sequence (provisional)

Two parallel waves (Q6:C). Shared NFR stages run once (Q7:A). Each unit runs its own Functional Design and Code Generation with its own stage gate.

| Step | Dates (provisional) | Work | Who |
|---|---|---|---|
| 0 | Nov 29 | Unit 0 model spike | Revanth |
| 1 | Nov 29 to Nov 30 | Contracts step and team review | Revanth, all |
| 2 | Nov 30 to Dec 1 | Shared NFR Requirements and NFR Design (system-wide), with QA review | Revanth, Erick |
| 3 | Dec 1 to Dec 6 | **Wave 1**: U1, U2, U3 (deterministic, no model needed); U5 web shell against fake services; U5 evaluation data | Addis, Carey, Timothy, Erick |
| 3a | Dec 1 to Dec 6 | U4 Functional Design and early Code Generation (gateway, skill mapper, ingestion parser) in parallel | Revanth |
| 4 | Dec 6 to Dec 10 | **Wave 2**: U4 graph and services on the real U1 to U3 code; U5 switches from fake to real services; LLM rule parsing completes the snapshot | Revanth, Erick, all |
| 5 | Dec 10 to Dec 11 | U5 Markdown export if time allows | Erick |
| 6 | Dec 11 to Dec 14 | Build and Test: evaluation runs, student testers, demo video | All |
| 7 | Dec 15 to Dec 16 | Buffer, rubric check, submission | Erick, Timothy |

**NFR handling (Q7:A)**: system-wide NFRs (headers, logging, timeouts, retries, health, input limits, import boundaries, PBT framework) are decided once in step 2. Each unit's Functional Design adds only unit-specific NFRs.

---

## 6. Workload Balance Notes
- U1 owns only 2 primary stories, but carries the ingestion pipeline, the indexes, and the fixture catalog that every unit's tests need in week 1.
- U4 owns 9 primary stories and the two pre-wave steps. The other LangGraph-experienced member should pair with Revanth on the Planning Graph in wave 2.
- U5 owns 10 primary stories, many of them cross-cutting acceptance criteria (security, accessibility, errors) that are thin in code once the middleware exists.

## 7. Open Items

| Item | Owner | Source |
|---|---|---|
| Correct README section 8: Member 1 is Timothy (GitHub and AI-DLC workflow), Member 3 is Carey (vision and requirements) | Addis (Member 2) | F1:B |
| Update `docs/technical-environment.md` Example 2 and the PBT list for CR-1 | Revanth | Application Design open items |
| Name the second LangGraph-experienced member who pairs on U4 | Team | Section 6 |

## 8. Validation and Extension Compliance

| Check | Result |
|---|---|
| Every story assigned to a primary unit | Pass: 30 of 30 (`unit-of-work-story-map.md`) |
| Every component in exactly one unit | Pass: C1 to C34; ExportService moved from C27 to U5 by Q11:A |
| No dependency cycle between units | Pass at module level; one script-level schedule dependency, mitigated (`unit-of-work-dependency.md`) |
| Deterministic core units (U1 to U3) do not depend on U4 | Pass: only `scripts/ingest_roster.py` calls U4, and it is not module code |

| Extension | Status at this stage | Notes |
|---|---|---|
| Security Baseline | Compliant | No new design surface. Security components stay in U5 (C28, C29, C32) and U1 (C8 grounding); shared security NFRs are decided once in the shared NFR stages (Q7:A). |
| Resiliency Baseline | Compliant | RESILIENCY-01 criticality from Application Design carries over: the Critical components sit in U1, U2, U3. RESILIENCY-04 and -14 remain deferred to the shared NFR Requirements and NFR Design stages. |
| Property-Based Testing | N/A at this stage | Each unit's definition of done names its required properties; PBT rules are enforced from Functional Design. |

No blocking findings.
