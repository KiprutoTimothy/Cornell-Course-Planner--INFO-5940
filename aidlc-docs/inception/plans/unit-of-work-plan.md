# Unit of Work Plan

**Stage**: INCEPTION - Units Generation, Part 1 (Planning)
**Owner role**: Member 5 (Erick: units of work and QA)
**Inputs**:
- `application-design.md` and its 4 detailed documents (approved)
- `stories.md` (30 stories, approved)
- `requirements.md` (approved, amended)
- `execution-plan.md` (Construction stages provisional)
- `docs/technical-environment.md` (binding)

**Outputs** (in `aidlc-docs/inception/application-design/`):
- `unit-of-work.md`
- `unit-of-work-dependency.md`
- `unit-of-work-story-map.md`

**Terminology**: the app is one deployable service (a modular monolith). Each **unit of work** is a group of modules and stories that one member owns through Construction. Units are not separate services.

---

## How to Answer
1. Fill in every `[Answer]:` tag with a letter, for example `[Answer]: A`. Add a short note after the letter if needed.
2. If you pick Other, describe what you want after the tag.
3. When every tag is filled in, reply "done" in chat.

---

## Starting Proposal (context for Question 1)
The design has 5 layers and the team has 5 members (B12:A, CQ12:A). The natural split is one unit per layer:

| Unit | Modules and components | Stories |
|---|---|---|
| U1 Catalog Data | `config`, `catalog` (C1 to C8), ingestion and index scripts | US-10.1, US-3.1 (search and grounding parts), US-10.4 (settings part) |
| U2 Prerequisite Engine | `prereq` (C9 to C14) | US-5.1, US-5.2, US-5.3 (chain part), US-1.4 (checklist rules) |
| U3 Planner | `planner` (C15 to C20) | US-1.3, US-4.1 (ranking), US-6.1, US-6.2, US-6.3 (fixer), US-7.1 (editor) |
| U4 Agent and Services | `agent` (C21 to C27) | US-2.1 to US-2.4, US-6.1 (graph), US-7.1 to US-7.4 |
| U5 Web and Evaluation | `web` (C28 to C32), `evaluation` (C33, C34) | US-1.1, US-1.2, US-8.1, US-9.1, US-10.2, US-10.3, US-11.1 to US-11.4 |

---

## Questions

## Question 1
How should the system be split into units?

A) 5 units, one per layer, as in the starting proposal above (Recommended: matches the module boundaries, so each owner works in their own directories)

B) 6 units: as above, but Evaluation (C33, C34) becomes its own unit, owned by the QA lead in addition to another unit

C) Vertical slices following the epics (for example profile, career, planning, revision), each cutting across all modules

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
The Ingestion Parser (C24, LLM parsing of complex prerequisite text at load time) is in the `agent` module, but it serves ingestion. Which unit owns it?

A) U1 Catalog Data: the ingestion owner runs the whole ingestion pipeline end to end, using U4's LLM Gateway

B) U2 Prerequisite Engine: the prerequisite owner owns everything about parsing rules, including the LLM half; U2's Parse Checks (C11) decide trust

C) U4 Agent and Services: all LLM code has one owner

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 3
Who owns each unit? Inception roles were: 1 GitHub and AI-DLC workflow; 2 README and product proposal; 3 vision and requirements; 4 technical environment and application design (Revanth); 5 user stories, units, and QA (Erick). The technical environment document says at least two members have LangGraph experience and lead agent design.

A) Assign by fit: the LangGraph-experienced members own U4 and U3; Member 4 (Revanth) owns U4 as design owner; Member 5 (Erick) owns U5, which includes evaluation and QA; Members 1 to 3 take U1 to U3 based on preference. Write the final names after the letter.

B) Other: write the assignment, for example "U1: name; U2: name; ..." after [Answer]: below

[Answer]: 1 GitHub and AI-DLC workflow; (Timothy) 2 README and product proposal (Addis); 3 vision and requirements ( Carey); 4 technical environment and application design (Revanth); 5 user stories, units, and QA (Erick)

## Question 4
The execution plan recommends measuring local model speed and tool-calling first, because NFR-1.1 (90 seconds) rests on an untested assumption. How should this risk check be handled?

A) A small Unit 0 "model spike", done first and timeboxed to 1 day: one structured-output call and one tool call through Ollama with `qwen3:8b`, timed on the team's fastest laptop. The result decides whether more logic moves into deterministic code before U4 is designed.

B) The first step of U4's Code Generation, with no separate unit

C) Skip it; measure during Build and Test

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
Several units depend on shared data shapes: Rule Model (C9), Verdict (C12), Planning Models (C15), and the service signatures (C27). How should these be fixed before parallel work starts?

A) A short "contracts" step at the start of Construction: Member 4 writes these models and signatures as code (types only, no logic), the team reviews them at one stage gate, and then all units build against them. Later changes go through the AI-DLC workflow.

B) Each owner defines their own models; integrate and reconcile later

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
How should units run through Construction?

A) In parallel: each owner runs their unit's own loop (Functional Design, then Code Generation) after the contracts step, with a stage gate per unit. Integration happens in Build and Test.

B) One unit at a time, in dependency order (U1, U2, U3, U4, U5), with the whole team on each unit

C) Parallel in two waves: wave 1 is U1, U2, U3 (deterministic core, no model needed) plus U5's web shell against fake services; wave 2 is U4, then the U5 integration and evaluation

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 7
NFR Requirements and NFR Design are mostly shared (security headers, logging, timeouts, retries, health checks). How should they run?

A) Once for the whole system, before the per-unit loops, owned by Member 4. Units add only unit-specific NFRs in their Functional Design.

B) Separately for each unit

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
How should the web unit (U5) work before the services (U4) exist?

A) Against fake services that implement the same signatures (C27) and return fixture data, so the UI, sessions, security middleware, and accessibility can be built and tested early

B) Wait for U4 to finish services first

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
The test fixture catalog (about 30 courses in `tests/fixtures/snapshot/`, per the technical environment) is used by every unit's tests. Who builds it, and when?

A) U1 builds it first, from real roster responses for the 8 core subjects, including courses that cover the evaluation and persona cases (for example CS 2110, CS 4780, MATH 2940, a course with "permission of instructor", a course offered only in fall)

B) Each unit builds its own small fixtures

C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 10
How should the code be organized?

A) Exactly the repository layout in `docs/technical-environment.md`: `src/course_planner/{catalog,prereq,planner,agent,web,evaluation}/`, tests mirrored in `tests/unit/`, `tests/property/`, `tests/integration/`, scripts in `scripts/`. Each unit owns its module directories and their test directories.

B) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 11
Where should the Should Have story US-8.1 (Markdown export) go, given the deadline?

A) In U5, scheduled last; dropped without affecting other units if time runs out

B) In U4 (ExportService), with U5 adding only the download route

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Follow-up Questions (from the answer review)

Questions 1, 2 and 4 to 11 are clear. Question 3 needs two follow-ups: the answer lists who held each **Inception role**, not who owns each **unit**, and its names differ from the README team table.

## Question F1
The Question 3 answer gives role 1 (GitHub and AI-DLC workflow) to Timothy and role 3 (vision and requirements) to Carey. The README team table (section 8) has the reverse: Carey Manwa is Member 1 and Timothy Kipruto is Member 3. Which is correct?

A) The README is correct: 1 Carey, 2 Addis, 3 Timothy, 4 Revanth, 5 Erick. Question 3's answer had roles 1 and 3 swapped.

B) The Question 3 answer is correct: 1 Timothy, 2 Addis, 3 Carey, 4 Revanth, 5 Erick. The README team table must be fixed (Member 2 owns the README).

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question F2
Who owns each Construction unit?

A) By fit with each member's Inception work:
- U1 Catalog Data: Addis (Member 2)
- U2 Prerequisite Engine: the vision and requirements owner (Member 3), who wrote the prerequisite requirements
- U3 Planner: the GitHub and AI-DLC workflow owner (Member 1), who keeps the workflow role as well
- U4 Agent and Services, plus the Unit 0 spike and the contracts step: Revanth (Member 4)
- U5 Web and Evaluation, plus the shared NFR stages' QA review: Erick (Member 5)

B) Other: write the assignment after [Answer]: below, as "U1: name; U2: name; U3: name; U4: name; U5: name"

[Answer]: A

---

## Execution Checklist (Part 2: Generation)
These steps run only after every answer is in, ambiguities are resolved, and the plan is approved. Each box is marked [x] as soon as its step is done.

- [x] 1. Define each unit: purpose, modules and components, owner, responsibilities, interfaces it provides and consumes, and definition of done
- [x] 2. Document the code organization strategy (greenfield): directory ownership per unit, test layout, scripts, fixtures (Question 10)
- [x] 3. Document any Unit 0 spike and contracts step (Questions 4 and 5)
- [x] 4. Document the Construction sequence and NFR handling (Questions 6 and 7), marked provisional per the execution plan
- [x] 5. Write `unit-of-work.md`
- [x] 6. Build the unit dependency matrix: which unit provides which interface to which, and what can be faked (Question 8)
- [x] 7. Write `unit-of-work-dependency.md`, with a Mermaid diagram and text alternative
- [x] 8. Assign every story (US-1.1 to US-11.4, including US-1.4) to a primary unit and list contributing units
- [x] 9. Write `unit-of-work-story-map.md`
- [x] 10. Validate: every story assigned; every component (C1 to C34) in exactly one unit; no dependency cycle between units; the deterministic core units do not depend on U4
- [x] 11. Run the extension compliance check (Security, Resiliency, PBT) for this stage
