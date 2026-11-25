# Unit of Work Story Map

**Stage**: INCEPTION - Units Generation

Each story has one **primary unit**, whose owner is responsible for its acceptance criteria passing, and any **contributing units**, which provide parts it needs. Priority is from `stories.md` (M = Must Have, S = Should Have).

## Story Map

| Story | Title | Priority | Primary unit | Contributing units | Wave |
|---|---|---|---|---|---|
| US-1.1 | Enter my academic profile | M | U5 | U3 (profile model), U4 (ProfileService) | 1 (fake services), 2 |
| US-1.2 | Enter completed and in-progress courses | M | U5 | U1 (course codes), U3, U4 | 1, 2 |
| US-1.3 | Fixed courses, unavailable terms, interests | M | U3 | U4, U5 | 1 |
| US-1.4 | Confirm the prerequisites I've already done (CR-1) | M | U2 | U3 (editor, validator), U4 (ProfileService), U5 (checklist UI) | 1 (rules), 2 (flow) |
| US-2.1 | Map my career goal to skills | M | U4 | U5 | 2 |
| US-2.2 | Help turning a vague goal into a career | M | U4 | U5 | 2 |
| US-2.3 | Edit my skill list | M | U4 | U3 (model), U5 | 2 |
| US-2.4 | Compare the skills of 2 to 3 careers | M | U4 | U5 | 2 |
| US-3.1 | See only real courses at my level | M | U1 | U4 (grounding in flows), U5 | 1 |
| US-4.1 | Ranked recommendations with reasons | M | U3 | U1 (search), U2 (verdicts), U4 (explanations), U5 (display) | 1 (ranker), 2 |
| US-5.1 | Whether I meet a course's prerequisites | M | U2 | U1 (catalog), U5 (display) | 1 |
| US-5.2 | "Needs review" when a prerequisite can't be checked | M | U2 | U5 | 1 |
| US-5.3 | Options when I don't meet a prerequisite | M | U2 | U3 (ranker), U4 (options in the flow), U5 | 1 (chain), 2 |
| US-6.1 | A valid multi-semester plan | M | U3 | U4 (planning graph, 90-second target), U5 | 1 (validator, fixer), 2 |
| US-6.2 | Uncertain offering terms | M | U3 | U1 (offering history), U5 | 1 |
| US-6.3 | Decide what to relax when courses don't fit | M | U3 | U4 (conflict flow), U5 | 1 (fixer conflict), 2 |
| US-7.1 | Change my plan in chat | M | U4 | U3 (editor), U5 (rate limit, chat UI) | 2 |
| US-7.2 | Ask why a course is in my plan | M | U4 | U5 | 2 |
| US-7.3 | Update my profile or skills in chat | M | U4 | U3 (editor) | 2 |
| US-7.4 | Start over with a full re-plan | M | U4 | U5 | 2 |
| US-8.1 | Download my plan as Markdown | **S** | U5 | U3 (plan model) | Last; dropped first |
| US-9.1 | Advisory notice and official links | M | U5 | U1 (roster URLs) | 1 |
| US-10.1 | Build and refresh the catalog snapshot | M | U1 | U2 (parser, checks), U4 (Ingestion Parser) | 1 (pattern rules), 2 (LLM rules) |
| US-10.2 | Review the generated evaluation cases | M | U5 | U2 (case format matches verdicts) | 1 |
| US-10.3 | Run the full evaluation with one command | M | U5 | U4 (services), U2 (evaluator) | 2 |
| US-10.4 | Set up the app, choose models, diagnose problems | M | U4 | U1 (Settings), U5 (Docker, health) | 0 (spike), 2 |
| US-11.1 | Keyboard and screen reader | M | U5 | — | 1 |
| US-11.2 | My data stays private | M | U5 | U4 (no content in logs) | 1, 2 |
| US-11.3 | Clear message when something goes wrong | M | U5 | U4 (gateway retries, timeouts) | 1, 2 |
| US-11.4 | Protection from malformed or malicious input | M | U5 | U1 (grounding), U4 (prompts, schema validation) | 1, 2 |

## Primary Stories per Unit

| Unit | Owner | Primary stories | Count |
|---|---|---|---|
| U1 Catalog Data | Addis | US-3.1, US-10.1 | 2 |
| U2 Prerequisite Engine | Carey | US-1.4, US-5.1, US-5.2, US-5.3 | 4 |
| U3 Planner | Timothy | US-1.3, US-4.1, US-6.1, US-6.2, US-6.3 | 5 |
| U4 Agent and Services | Revanth | US-2.1, US-2.2, US-2.3, US-2.4, US-7.1, US-7.2, US-7.3, US-7.4, US-10.4 | 9 |
| U5 Web and Evaluation | Erick | US-1.1, US-1.2, US-8.1, US-9.1, US-10.2, US-10.3, US-11.1, US-11.2, US-11.3, US-11.4 | 10 |
| **Total** | | | **30** |

## Component Allocation Check

| Unit | Components |
|---|---|
| U1 | C1, C2, C3, C4, C5, C6, C7, C8 |
| U2 | C9, C10, C11, C12, C13, C14 |
| U3 | C15, C16, C17, C18, C19, C20 |
| U4 | C21, C22, C23, C24, C25, C26, C27 (except ExportService) |
| U5 | C28, C29, C30, C31, C32, C33, C34, and ExportService (part of C27, moved by Q11:A) |

All 34 components are allocated. Each is in exactly one unit; the only split is C27, where ExportService goes to U5 by decision Q11:A.

## MVP Journey Coverage by Unit
- **Journey 1** (Maya, health data science): U5 (profile, export) → U4 (skills) → U1 (search) → U3 (ranking, plan, uncertain offering) → U2 (verdicts) → U4 (chat move for Fall 2027) → U5 (export).
- **Journey 2** (Daniel, AI product management): U5 (profile) → U4 (skills) → U1 and U3 (recommendations) → U2 (Not met, chain) → U4 (options, "why" answers).

Both journeys need every unit, so the journey tests (NFR-7.3) run at integration checkpoint 4 (`unit-of-work-dependency.md`).
