# Application Design Plan

**Stage**: INCEPTION - Application Design
**Owner role**: Member 4 (technical environment and application design)
**Inputs**:
- `requirements.md` (approved)
- `stories.md` and `personas.md` (approved)
- `execution-plan.md` (approved, including change request CR-1)

**Outputs** (in `aidlc-docs/inception/application-design/`):
- `components.md`
- `component-methods.md`
- `services.md`
- `component-dependency.md`
- `application-design.md`

**Scope reminder**: this stage defines components, their responsibilities, interfaces, and dependencies. Detailed business rules (for example the exact "and/or" parsing or the fixer algorithm) belong to Functional Design in Construction.

---

## How to Answer
1. Fill in every `[Answer]:` tag with a letter, for example `[Answer]: B`. Add a short note after the letter if needed.
2. If you pick Other, describe what you want after the tag.
3. Section A settles the details of change request CR-1. Section B is the design questions.
4. When every tag is filled in, reply "done" in chat.

---

# Section A: Change Request CR-1 (Prerequisite Checklist)

**Request**: when a student is taking a class that has prerequisites, show its prerequisite list and let the student select the ones they have done. Store the selections in the profile, so that when the planner meets another class with the same prerequisite or corequisite, it is already checked off.

## Question A1
When should the app show a course's prerequisite checklist?

A) When the student enters a course as in-progress (Fall 2026) or as a fixed (pinned) course

B) Option A, plus whenever a recommended or planned course has prerequisites not already satisfied by the student's entered courses (the student can confirm what they've done before the verdict is final)

C) Only when the student opens a course and chooses "check my prerequisites"

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question A2
Which prerequisite items appear in the checklist?

A) Only course-code prerequisites and corequisites (for example CS 2110, MATH 2210)

B) Course codes plus non-course clauses as separate items (for example "or equivalent", "permission of instructor", "a course in linear algebra")

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question A3
If the student checks off a **course code** (for example CS 2110), what happens?

A) The course is added to their completed courses, exactly as if they had entered it in the profile (FR-1.2)

B) It is stored separately as a "student-confirmed prerequisite". It satisfies prerequisites but is not treated as a completed course elsewhere (for example it still shows as a recommendation only if relevant)

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question A4
If the student checks off a **non-course clause** (for example "or equivalent", "permission of instructor"), how is the verdict shown? Today those clauses always give "Needs review" (FR-5.5).

A) The verdict becomes **Met (student-confirmed)**, a label that shows it rests on the student's word

B) The verdict stays **Needs review**, with a note that the student says it is satisfied

C) Not applicable (Question A2 answered A: non-course clauses are not in the checklist)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question A5
How should "the same prerequisite" be matched when another course comes up?

A) Exact match only: the same course code, or the same normalized clause text (for example "permission of instructor" matches for every course; "a course in linear algebra" matches only the identical phrase)

B) Course codes match exactly; non-course clauses match only within the same course (permission from one instructor does not carry to another course)

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question A6
What happens with items the student leaves unchecked?

A) They count as not done, and the verdict follows the normal rules (Not met or Needs review)

B) They count as unknown, and the app asks again the next time a course needs them

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question A7
The profile is currently session only, with nothing written to disk (CQ7:D, FR-1.7). Should the checklist selections follow the same rule?

A) Yes: stored in the session profile only, and included in the Markdown export if it is built

B) No: saving them to disk changes the earlier persistence decision (please describe after [Answer]: tag below)

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question A8
Can the student review and undo their checked-off items?

A) Yes: a "Prerequisites I've confirmed" list in the profile, where any item can be removed; affected verdicts and placements are re-checked incrementally (FR-7.4)

B) Yes, but only through chat (for example "I haven't actually taken MATH 2210")

C) No: selections stay for the session

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

# Section B: Application Design

## Question B1
What overall structure should the application have?

A) Modular monolith: one Python package with a module per component, run as one web app process in Docker. Ingestion and evaluation are command-line entry points in the same package. (Recommended: fits a local app, one container, five owners working on separate modules.)

B) Separate services: web app, agent service, and search service as separate processes or containers talking over HTTP

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question B2
How should the first plan be produced (profile and goal in, plan out)?

A) Fixed pipeline with LLM calls at set steps: map skills, search, rank, check prerequisites, propose placement, validate, retry or fix. The tool-calling agent is used only for chat revision and "why" questions. (Recommended: predictable, easier to time against the 90-second target and to test.)

B) A single tool-calling agent (for example LangGraph) for everything, deciding which tools to call and in what order

C) A LangGraph state graph with fixed nodes for the first plan and an agent node for chat revision

D) Other (please describe after [Answer]: tag below)

[Answer]: C. A LangGraph state graph: fixed nodes for map skills, search, check prerequisites, propose plan, validate, and retry/fix; one agent node for chat revision and "why" questions (per docs/technical-environment.md)

## Question B3
How fine-grained should the agent's tools be (for chat revision and "why" questions)?

A) Coarse, task-level tools: `search_courses`, `get_course`, `check_prerequisites`, `get_prerequisite_chain`, `propose_plan_change`, `validate_plan`, `explain_course`, `update_profile`

B) Fine-grained tools over raw data (for example `get_prereq_text`, `get_offering_terms`, `get_credits`), with the agent assembling the answer

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question B4
Where should plan changes requested in chat be applied?

A) The agent only proposes a structured change (for example "move CS 4780 to SP28"). Deterministic code applies it, validates it, and runs the fixer. The agent never edits the plan directly. (Recommended: keeps FR-6.7 and NFR-4.8 enforceable in one place.)

B) The agent edits the plan object directly, then the validator runs

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question B5
How should the catalog snapshot and search index be stored?

A) SQLite database for courses, parsed rules, and offering history, plus a local vector index file (for example FAISS or Chroma) for embeddings, all built by the ingestion command

B) JSON files for courses and rules, plus a local vector index file

C) SQLite for everything, including embeddings (for example the sqlite-vec extension)

D) Other (please describe after [Answer]: tag below)

[Answer]: A. SQLite (with an FTS5 table for keyword search) plus a local Chroma persistent collection for embeddings; no FAISS (per docs/technical-environment.md)

## Question B6
Should the large snapshot files be committed to git? (Q8:B says the snapshot is committed.)

A) Commit the raw API responses and the built database. The vector index is rebuilt locally by a command.

B) Commit everything, including the vector index

C) Commit only the raw API responses; every member builds the database and index locally

D) Other (please describe after [Answer]: tag below)

[Answer]: C. Commit the raw Class Roster responses and the parsed rules (`data/snapshot/prereq_rules.json`). `catalog.sqlite3` and `chroma/` are git-ignored and rebuilt by one command, `scripts/build_index.py` (per docs/technical-environment.md)

## Question B7
How should session state (profile, skills, plan, checklist selections) be held? Nothing is written to disk.

A) In server memory, keyed by a random session ID in an HTTP-only cookie, cleared when the app stops or after an idle timeout

B) In the browser (signed cookie or hidden form fields) and sent with every request

C) Other (please describe after [Answer]: tag below)

[Answer]: A. 60-minute idle timeout

## Question B8
How should the server-rendered UI update the page (form, chat, plan view)?

A) HTMX for partial page updates (served locally, no inline scripts, compatible with the CSP in NFR-4.4)

B) Full page reloads only, with no JavaScript

C) Small hand-written JavaScript files for the chat and plan view

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question B9
How should the app talk to the LLM?

A) One LLM gateway component that every other component uses. It reads the primary, fallback, and hosted settings from configuration, and it owns timeouts, retries, structured-output parsing, and timing logs.

B) Each component creates its own LangChain model client

C) Other (please describe after [Answer]: tag below)

[Answer]: A. The gateway lives in the `agent` module; `catalog`, `prereq`, and `planner` never import it (NFR-7.1)

## Question B10
How should the evaluation harness run the app?

A) In-process: it calls the same service layer the web app calls, with no HTTP. Timing is measured around the service call.

B) Over HTTP against a running app, as a student would use it

C) In-process for accuracy metrics, plus one HTTP run for the time-to-first-plan metric

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question B11
Where should the team-reviewed career-to-skill mappings (FR-2.4) live?

A) A YAML or JSON file committed in the repository; a command generates draft mappings, and a team member reviews and commits them

B) Inside the snapshot database, generated during ingestion

C) Other (please describe after [Answer]: tag below)

[Answer]: A. As a JSON file (docs/technical-environment.md allows YAML only for Docker Compose and CI)

## Question B12
How should components be split for ownership, given each member owns one or more units later?

A) By layer, each with a clear interface: data (ingestion, snapshot, search), rules (prerequisites, validator, fixer), intelligence (LLM gateway, skill mapper, recommender, agent), interface (web UI, sessions), quality (evaluation harness)

B) By feature, following the epics (E1 to E11), with shared data and LLM modules

C) Decide in Units Generation; this stage only defines components

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

# Section C: Follow-up Questions (from the answer review)

The answers to Sections A and B are complete. Three points still need a decision before generation: two gaps in the CR-1 answers, and one set of differences between `requirements.md` and `docs/technical-environment.md`. The technical environment document asks that such differences be settled through the workflow, not chosen silently.

## Question C1
A1:B shows a checklist whenever a recommended or planned course has prerequisites the student hasn't already covered. When does that happen relative to building the first plan?

A) Plan first. The first plan uses only the courses the student entered. Each course with an unmet prerequisite shows a "Confirm prerequisites" checklist, and confirming items re-checks verdicts and placements incrementally (FR-7.4). The 90-second first-plan timing is unaffected.

B) Ask first. After recommendations are ranked and before placement, the student sees one combined checklist of every distinct unmet item across the recommended courses. The plan is built after they submit it, and the 90-second timing excludes the time the student spends on the checklist.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question C2
A3:A adds a checked-off course code to completed courses. Some prerequisites name courses outside the loaded subjects (for example PHYS 1112 when PHYS isn't loaded). FR-1.2 says such codes are kept with a warning but do **not** satisfy prerequisites. What happens if the student checks one off in a prerequisite checklist?

A) It counts. A checked-off code taken from the course's own prerequisite text satisfies that prerequisite everywhere, even if the code isn't in the loaded catalog. It is shown with a "not in loaded catalog" note. FR-1.2 stays as it is for codes typed into the profile.

B) It doesn't count. Only codes in the loaded catalog can be checked off. Others are shown read-only with an explanation, and the verdict stays Not met.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question C3
`docs/technical-environment.md` differs from the approved `requirements.md` in these points:
- the model runtime is Ollama only, not "Ollama or LM Studio" (requirements section 2)
- the team machines are macOS and Windows only, not Linux (NFR-6.1)
- GitHub Actions CI with `pip-audit` and Dependabot exists, whereas requirements say "No CI pipeline" (section 10, SECURITY-13)
- it adds specific values: 3 LLM retries, 60-second LLM timeout, 10-second Class Roster timeout, 10 chat requests per minute per session, a 60-minute session idle timeout, and the input limits

Which document wins?

A) The technical environment document wins. Update `requirements.md` to match, with a change note citing this question.

B) The requirements win. The technical environment owner updates `docs/technical-environment.md`.

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Design decisions taken without a question
These follow directly from the answers and the technical environment document. They are listed so the team can object at the stage gate:
- **Agent tools stay read-only (LLM06).** Answer B3:A listed `update_profile` and `propose_plan_change`. Following B4:A, both become proposal tools (`propose_profile_change`, `propose_plan_change`). They return a structured change, and deterministic service code applies it after validation. No tool writes state.
- **"Met (student-confirmed)" (A4:A) keeps the three statuses.** The verdict status stays Met, Not met, or Needs review (as in the technical environment's evaluator example). A `student_confirmed` flag and reason mark a Met that rests on the student's checklist, and the UI shows the label "Met (student-confirmed)". The 40 evaluation cases and the existing property-based invariants stay valid.
- **Ranking is deterministic and lives in `planner`.** The B2:C node list has no ranking step. Ranking uses skill coverage from search hits plus reachability from `prereq` (FR-4.3), so it goes in `planner`, which may import `prereq` and `catalog`, and is run by a graph node.
- **Ingestion-time LLM parsing.** The LLM call lives in `agent` (B9). The automated checks on each parse (FR-5.3) live in `prereq`, which never imports `agent`. `scripts/ingest_roster.py` connects the two.

---

## Execution Checklist (Generation)
These steps run only after every answer is in, ambiguities are resolved, and the team approves. Each box is marked [x] as soon as its step is done.

### Change request CR-1
- [ ] 1. Add FR-1.9 and FR-5.11 to `requirements.md` (and adjust FR-5.5 and FR-5.7 if Question A4 needs it), with a change note citing CR-1. Apply the C3 decision to `requirements.md` or record it for the technical environment owner.
- [ ] 2. Add story US-1.4 to `stories.md`, add criteria to US-5.1 and US-5.2, and update the persona map and coverage table

### Design artifacts
- [ ] 3. Identify components and their responsibilities, including the CR-1 checklist in the profile; write `components.md`
- [ ] 4. Define method signatures with inputs and outputs for each component (no detailed business rules); write `component-methods.md`
- [ ] 5. Define the service layer and its orchestration: first-plan flow, chat revision flow, checklist flow, ingestion flow, evaluation flow; write `services.md`
- [ ] 6. Define the dependency matrix, communication patterns, and data flow diagrams; write `component-dependency.md`
- [ ] 7. Classify each component's criticality and the impact of its being unavailable (RESILIENCY-01)
- [ ] 8. Map every story (US-1.1 to US-11.4, plus US-1.4) to at least one component
- [ ] 9. Write `application-design.md`, which consolidates steps 3 to 8

### Validation
- [ ] 10. Check that deterministic components are separate from LLM components (NFR-7.1)
- [ ] 11. Check that the validator and the course-code check cannot be bypassed by any path (FR-6.7, FR-4.5, NFR-4.8)
- [ ] 12. Validate Mermaid diagrams, with text alternatives (`common/content-validation.md`)
- [ ] 13. Run the extension compliance check (Security, Resiliency, PBT) for this stage
