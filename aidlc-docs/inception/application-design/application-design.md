# Application Design: Cornell AI Course Planning Agent

**Stage**: INCEPTION - Application Design
**Owner**: Member 4 (Revanth)
**Date**: 2026-11-24
**Status**: Draft for team approval
**Detailed documents**: [`components.md`](components.md), [`component-methods.md`](component-methods.md), [`services.md`](services.md), [`component-dependency.md`](component-dependency.md)

This document consolidates the four detailed documents. Where they go deeper, they are the reference.

---

## 1. Design Decisions

| Topic | Decision | Source |
|---|---|---|
| Structure | Modular monolith: one package `src/course_planner/`, one FastAPI process, one Docker Compose service; ingestion, index build, and evaluation are scripts | B1:A, tech env |
| First plan | LangGraph state graph with fixed nodes: map skills, search, rank, check prerequisites, propose plan, validate, retry or fix, explain | B2:C |
| Chat revision and "why" | One agent node in the graph, with `InMemorySaver` keyed by session ID | B2:C |
| Agent tools | Coarse, read-only: `search_courses`, `get_course`, `check_prerequisites`, `get_prerequisite_chain`, `validate_plan`, `explain_course`, plus proposal tools `propose_plan_change` and `propose_profile_change` | B3:A, LLM06 |
| Applying changes | The agent only proposes. The deterministic Plan Editor applies, the Validator checks, the Fixer repairs | B4:A |
| Catalog storage | SQLite with FTS5 for keyword search, plus a local Chroma collection for embeddings (FastEmbed) | B5:A |
| What is committed | Raw roster JSON and `prereq_rules.json`; SQLite and Chroma are git-ignored and rebuilt by `scripts/build_index.py` | B6:C |
| Session state | Server memory keyed by a random cookie ID; 60-minute idle timeout | B7:A |
| UI updates | HTMX fragments, vendored, no hand-written JavaScript | B8:A |
| LLM access | One LLM Gateway in `agent`; `catalog`, `prereq`, `planner` never import it | B9:A |
| Evaluation access | In process, through the same service layer as the web app | B10:A |
| Reviewed skill mappings | Committed JSON file | B11:A |
| Component split | By layer: data, rules, intelligence, interface, quality | B12:A |
| Ranking | Deterministic, in `planner` (C19) | Plan decision |
| Ingestion LLM parse | LLM call in `agent` (C24); trust checks in `prereq` (C11) | Plan decision |
| CR-1 verdict label | "Met (student-confirmed)" is Met plus a `student_confirmed` marker; still 3 statuses | A4:A, plan decision |

## 2. Components

| Layer | Module | Components |
|---|---|---|
| Shared | `config` | C1 Settings |
| Data | `catalog` | C2 Roster Client, C3 Snapshot Store, C4 Index Builder, C5 Catalog Store, C6 Course Codes, C7 Hybrid Search, C8 Grounding Check |
| Rules | `prereq` | C9 Rule Model, C10 Pattern Parser, C11 Parse Checks, C12 Prerequisite Evaluator, C13 Chain Builder, C14 Checklist Builder |
| Rules | `planner` | C15 Planning Models, C16 Offering and Credit Rules, C17 Plan Validator, C18 Plan Fixer, C19 Recommendation Ranker, C20 Plan Editor |
| Intelligence | `agent` | C21 LLM Gateway, C22 Prompt Library, C23 Skill Mapper, C24 Ingestion Parser, C25 Agent Tools, C26 Planning Graph, C27 Services |
| Interface | `web` | C28 Session Store, C29 Security Middleware, C30 Routes and Templates, C31 Health Checks, C32 Error Handler |
| Quality | `evaluation` | C33 Evaluation Cases, C34 Evaluation Runner |

Responsibilities are in `components.md`; signatures in `component-methods.md`.

## 3. Services and Flows
Six runtime services (Profile, Career, Planning, Revision, Export, Health) plus two script services (Ingestion, Evaluation). Five flows are defined in `services.md`:
1. **First plan**: search, rank, check prerequisites, propose, ground, validate, retry or fix, explain; conflict goes to "which constraint to relax".
2. **Chat revision**: the agent reads and proposes; the Plan Editor applies to a copy; the Validator checks; the answer is grounded.
3. **Prerequisite checklist (CR-1)**: build checklist, confirm, add codes or store clauses, re-evaluate affected courses, incremental plan re-check.
4. **Ingestion and index build**.
5. **Evaluation**.

## 4. Change Request CR-1 in the Design

| Requirement | Where it is handled |
|---|---|
| FR-1.9.1 When the checklist is offered | `ProfileService.get_checklist`; offered after `set_courses`, `set_constraints`, and on plan courses with a verdict that is not Met (Flow 3) |
| FR-1.9.2 Items (codes and clauses) | C14 Checklist Builder; `ReviewFlag.clause_id` in C9 |
| FR-1.9.3 Checked codes count everywhere, even outside the catalog | `ConfirmedPrerequisites.codes_from_checklist`; `StudentProfile.taken()` includes them |
| FR-1.9.4 Clauses stay with their course | `ConfirmedPrerequisites.clauses[course]`, passed to `evaluate(..., confirmed_clauses=...)` for that course only |
| FR-1.9.5 Unchecked = not done | Evaluator default |
| FR-1.9.6 Session only, in export | Session Store; ExportService |
| FR-1.9.7 Review and remove | `ProfileService.list_confirmed`, `confirm_prerequisites(unchecked=...)` |
| FR-1.9.8 Incremental re-check, outside first-plan time | Flow 3 uses Plan Editor and Validator, not the first-plan graph |
| FR-5.7 Unverified rules | Checklist lists codes found in text; verdict stays "Needs review (unverified parse)" |
| FR-5.11 "Met (student-confirmed)" | `Verdict.student_confirmed`; label rendered by the template |

## 5. Story-to-Component Map

| Story | Main components | Service |
|---|---|---|
| US-1.1 Academic profile | C15, C28, C30 | ProfileService |
| US-1.2 Courses taken | C6, C5, C15 | ProfileService |
| US-1.3 Constraints and preferences | C15, C16 | ProfileService |
| US-1.4 Confirm prerequisites (CR-1) | C14, C12, C20, C17, C15 | ProfileService |
| US-2.1 Career to skills | C23, C21, C22 | CareerService |
| US-2.2 Vague goal | C23 | CareerService |
| US-2.3 Edit skills | C15 | CareerService |
| US-2.4 Compare careers | C23 | CareerService |
| US-3.1 Real courses at my level | C7, C8, C6, C22 | PlanningService |
| US-4.1 Ranked recommendations | C19, C7, C12 | PlanningService |
| US-5.1 Prerequisite verdict | C12, C9, C5, C16 | PlanningService |
| US-5.2 Needs review | C12, C11, C14 | PlanningService |
| US-5.3 Options when not met | C13, C19, C26 | PlanningService |
| US-6.1 Valid plan | C26, C17, C18, C16, C8 | PlanningService |
| US-6.2 Uncertain offering | C16, C5 | PlanningService |
| US-6.3 Plan does not fit | C18, C26 | PlanningService |
| US-7.1 Edit plan in chat | C26, C25, C20, C17, C29 | RevisionService |
| US-7.2 Ask why | C26, C25, C8 | RevisionService |
| US-7.3 Profile change in chat | C25, C20 | RevisionService |
| US-7.4 Start over | C26 | RevisionService |
| US-8.1 Markdown export (S) | C30 | ExportService |
| US-9.1 Advisory notice and links | C30, C5 | PlanningService |
| US-10.1 Catalog snapshot | C2, C3, C4, C10, C11, C24 | IngestionService |
| US-10.2 Review evaluation cases | C33 | EvaluationService |
| US-10.3 Run evaluation | C34 | EvaluationService |
| US-10.4 Setup, models, logs | C1, C21, C31 | HealthService |
| US-11.1 Keyboard and screen reader | C30 | (templates) |
| US-11.2 Data privacy | C28, C29 | (all) |
| US-11.3 Clear errors | C32, C31, C21 | HealthService |
| US-11.4 Input safety | C29, C30, C8, C22, C21 | (all) |

All 30 stories map to at least one component.

## 6. Workload Criticality (RESILIENCY-01)
Summary (full table in `components.md`):
- **Critical**: Catalog Store and its index, Evaluator, Validator, Fixer, Grounding Check. Without them no plan or verdict can be shown safely.
- **High**: LLM Gateway with Ollama, Hybrid Search, Session Store, Security Middleware, Routes. The app degrades with a clear message.
- **Medium**: Skill Mapper, Ranker, Checklist Builder, Plan Editor, Health Checks, Error Handler.
- **Low**: ingestion components, evaluation components, Markdown export.

There is no revenue or regulatory impact (local course project, advisory output). The impact recorded is to students using the app and to the team's evidence for the demo.

## 7. Validation

| Check | Result |
|---|---|
| Deterministic components separate from LLM components (NFR-7.1) | Pass. `catalog`, `prereq`, `planner` hold every deterministic component (C2 to C20) and never import `agent` or LLM libraries. The only LLM code is in `agent` (C21 to C26). |
| Validator cannot be bypassed (FR-6.7) | Pass. Every path that returns a plan goes through a service that calls C17: Flow 1 (step 6), Flow 2 (step 6), Flow 3 (step 5). Agent tools cannot modify the plan; they only propose. |
| Grounding cannot be bypassed (FR-4.5, NFR-4.8) | Pass. Flow 1 grounds the proposal and the explanations; Flow 2 grounds proposals and answer text; search results come only from SQLite-checked hits. |
| Student data never on disk (FR-1.7) | Pass. Only Session Store and `InMemorySaver` hold it; logs exclude content; export is streamed. |
| Every story mapped | Pass. 30 of 30 (section 5). |
| Mermaid diagrams | Validated: alphanumeric node IDs, quoted labels, quoted edge labels; each diagram has a text alternative. |

## 8. Extension Compliance (Application Design)

| Extension | Rule | Status | Notes |
|---|---|---|---|
| Security | SECURITY-03 Logging | Compliant | Structured JSON logs without student content (C21, services) |
| Security | SECURITY-04 Headers | Compliant | C29 |
| Security | SECURITY-05 Input validation | Compliant | Pydantic forms with `extra="forbid"` (C30), C6 code format, 64 KB body limit (C29) |
| Security | SECURITY-07 Network | Compliant | Port published on 127.0.0.1 only; outbound URLs only from Settings |
| Security | SECURITY-08 Access control | Compliant | Session isolation by cookie ID (C28); CSRF on every POST (C29) |
| Security | SECURITY-09 Hardening | Compliant | C32 generic errors; no debug mode; non-root container user |
| Security | SECURITY-10 Supply chain | Compliant | `uv.lock`, pip-audit in CI, pinned base image |
| Security | SECURITY-11 Secure design | Compliant | Read-only tools, proposal-only changes, Validator and Grounding as layered controls; rate limit in C29 |
| Security | SECURITY-12 Credentials | Compliant | Only `LLM_API_KEY`, from `.env` through C1 |
| Security | SECURITY-13 Integrity | Compliant | Pydantic loading of snapshot and LLM output; no pickle; vendored HTMX with checksum |
| Security | SECURITY-15 Exceptions | Compliant | C32; services keep state on failure |
| Security | SECURITY-01, -02, -06, -14 | N/A | Unchanged from requirements section 10 (no stored sensitive data, no intermediaries, no IAM, no deployed alerting) |
| Resiliency | RESILIENCY-01 Criticality | **Compliant (completed here)** | Section 6 and `components.md` |
| Resiliency | RESILIENCY-06 Health checks | Compliant | C31 shallow and deep checks; startup embedding check |
| Resiliency | RESILIENCY-10 Timeouts and degradation | Compliant | C21 (60 s, 3 attempts), C2 (10 s); degraded messages in services |
| Resiliency | RESILIENCY-04, -14 | N/A at this stage | Still deferred to NFR Requirements and NFR Design |
| Resiliency | Others | Unchanged | As in requirements section 10 |
| PBT | PBT-01 to PBT-10 | N/A at this stage | Enforced from Functional Design. Pure components with properties to define there: C6, C7 `rrf_merge`, C9 round trip, C10, C12 (including a new CR-1 property: confirming clauses never turns Met into Not met or Needs review), C14, C16, C17, C18, C19, C20. |

No blocking findings.

## 9. Open Items

| Item | Owner | Why |
|---|---|---|
| Update Example 2 in `docs/technical-environment.md`: add `confirmed_clauses` to `evaluate`, `student_confirmed` to `Verdict`, and `clause_id` to `ReviewFlag` | Member 4 (technical environment owner) | That document says examples must change in the same commit as the standard they show. CR-1 changes the evaluator's interface. |
| Add the CR-1 property to the PBT list in `docs/technical-environment.md` | Member 4 | Keeps the binding list of required invariants complete |
| Import-boundary test (module rules in `component-dependency.md`) | NFR Design | Enforces NFR-7.1 automatically |
| Final route paths and form fields | Functional Design (web unit) | Paths in `component-methods.md` are indicative |
