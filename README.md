# Cornell AI Course Planning Agent
 
**INFO 5940 Final Project · Team of 5 · Demo deadline: December 16, 2026**
 
A local web app that helps Cornell students choose courses based on the career they want. A student types a career goal in plain language and lists the courses they have completed. The agent then:
 
1. maps the goal to concrete skills
2. searches the Cornell course catalog for courses that teach those skills
3. checks each course's prerequisites against the student's history
4. lays the courses out across the student's remaining semesters
Every recommendation comes with a short explanation and a link to the official Class Roster listing. Students can revise the plan in chat and take it to their advisor.
 
> **Advisory only.** This tool does not check graduation requirements and is not an official degree audit. Always confirm your plan with your academic advisor.
 
**Project status:** Inception phase complete (AI-DLC v1.0.1). All Inception stages approved 2026-11-24; Construction starts in the final project. No application code yet. See [`aidlc-docs/aidlc-state.md`](aidlc-docs/aidlc-state.md).

**Coding agent:** Claude Code, following AI-DLC v1.0.1 (rules in [`CLAUDE.md`](CLAUDE.md) and [`.aidlc-rule-details/`](.aidlc-rule-details/)).
 
---
 
## Table of Contents
 
1. [The Problem](#1-the-problem)
2. [Why an LLM-Powered Agent](#2-why-an-llm-powered-agent)
3. [Target Users](#3-target-users)
4. [Planned LLM Integration](#4-planned-llm-integration)
5. [Project Scope](#5-project-scope)
6. [Success Metrics](#6-success-metrics)
7. [Risks and Mitigations](#7-risks-and-mitigations)
8. [Team](#8-team)
9. [Collaboration Plan](#9-collaboration-plan)
10. [Repository Structure](#10-repository-structure)
11. [Setup (Planned)](#11-setup-planned)
12. [Key Documents](#12-key-documents)
---
 
## 1. The Problem
 
Cornell offers thousands of courses across more than 170 subjects each semester. Students who know what job they want ("data scientist at a health company", "UX researcher", "ML engineer") have no tool that connects that goal to specific courses:
 
| Existing tool | Question it answers |
|---|---|
| Class Roster | What is offered this semester? |
| Requirement trackers | What counts toward my major? |
| Course review sites | Is this course good? |
| **Nothing** | **What should I take, and in what order, to be ready for this job?** |
 
Students piece this together from upperclassmen, LinkedIn profiles, and advisor meetings that are short and booked weeks out.
 
**Prerequisites make it harder.** They are published as free text, and each department writes them differently. Real examples from the Spring 2026 INFO roster:
 
- *"CS 1110 or CS 1112 and INFO 1300."* (unclear how "or" and "and" group)
- *"INFO 2950 or equivalent, AEM 2100, CS 1110."* (open-ended "or equivalent")
- *"Recommended prerequisite for programmers: CS 2110, or permission of instructor. Recommended prerequisite for designers: INFO 2450 ..."* (different rules depending on the student's background)
A student who misreads one of these finds out at enrollment or in week two of the course. A student who misses a prerequisite chain (for example CS 2110 → CS 3110 → a 4000-level systems course) can lose a full semester. That hurts most for one-year professional master's students.
 
---
 
## 2. Why an LLM-Powered Agent
 
The core tasks are **language tasks inside a reasoning loop**:
 
| Task | Why it needs an LLM |
|---|---|
| Career goal → skills | Requires world knowledge about jobs that no Cornell dataset contains |
| Skills → courses | Course descriptions use different words for the same skill ("statistical learning", "predictive modeling", "machine learning") |
| Reading prerequisite text | The text is inconsistent natural language with edge cases like "or equivalent" |
| Building a plan | Several dependent steps: search, check eligibility, check when a course is offered, search again when a course is blocked, then sequence. The agent decides which step to run next and uses tool results before it answers. |
 
**Where we do *not* trust the LLM.** Deterministic code handles anything that has to be exactly right: credit totals, prerequisite evaluation, ordering checks, and course-code validation. The LLM handles the parts that need language understanding and judgment, and it must ground every course it names in catalog data returned by a tool. Any course code not found in the loaded catalog is removed before it is shown.
 
---
 
## 3. Target Users
 
| User | Primary need |
|---|---|
| **Career-focused undergraduate** (sophomore/junior in CS, InfoSci, ORIE, Statistics, etc.) | Which electives build the skills for my target role, and when should I take them? |
| **Exploring undergraduate** (first-year/sophomore, undecided) | Compare 2–3 career paths and see the skills each one needs |
| **Professional master's student** (MPS/MEng, 2–3 semesters, often a non-Cornell undergrad) | Highest-value courses for the target role, and fast detection of prerequisite gaps |
| **Major switcher** (e.g., Biology → data science) | The shortest valid prerequisite chain into the courses they need |
 
Stakeholders who are not direct users: academic advisors (receive draft plans), INFO 5940 course staff (evaluate the product as an agentic LLM application), and the 5-person team.
 
---
 
## 4. Planned LLM Integration
 
### 4.1 Architecture
 
```
+-------------------------------------------------------------+
|  Web UI (server-rendered, FastAPI or Flask + HTML templates)|
|  Profile form  |  Career & skills  |  Plan view  |  Chat    |
+------------------------------+------------------------------+
                               |
+------------------------------v------------------------------+
|        Agent (Python, LangChain / LangGraph)                |
|        LLM decides the next step and calls tools            |
+----+-------------+---------------+---------------+----------+
     |             |               |               |
+----v-----+ +-----v------+ +------v-------+ +-----v---------+
| Skill    | | Hybrid     | | Prerequisite | | Planner +     |
| mapping  | | catalog    | | evaluator    | | validator +   |
| (cached  | | search     | | (determin-   | | fixer         |
| or live) | | (keyword + | | istic rules) | | (determin-    |
|          | | embeddings)| |              | | istic)        |
+----------+ +-----+------+ +------+-------+ +---------------+
                   |               |
+------------------v---------------v--------------------------+
|  Dated catalog snapshot (Class Roster API, 4 yrs history)   |
|  + prerequisite rules parsed at load time                   |
+-------------------------------------------------------------+
 
LLM: free local model via Ollama or LM Studio, running natively
on the host. The app runs in Docker and calls it over localhost.
```
 
### 4.2 Where the LLM is used
 
| Step | LLM role | Deterministic safeguard |
|---|---|---|
| **Career → skills** | Turns a free-text goal into 6–12 named skills, each with a one-line reason. Asks follow-up questions if the goal is too vague. | Six reviewed careers use cached mappings the team checked. The student can add, remove, or reword skills. |
| **Catalog search** | Writes search queries from the skill list | Hybrid search (keyword + local sentence-transformers embeddings) over the snapshot only |
| **Prerequisite parsing** (at load time) | Parses complex prerequisite text into structured rules. Simple patterns are parsed with regex instead. | Each LLM parse must pass: a course-code cross-check against the original text, a check that every code exists in the catalog, keyword flags, and agreement between two independent parses. Failures are stored as **unverified**. |
| **Prerequisite verdicts** | none | A deterministic evaluator returns **Met**, **Not met** (with the missing courses), or **Needs review** (permission of instructor, "or equivalent", recommended-only, background-specific, or unverified parse). The original text is always shown. |
| **Semester planning** | Proposes a placement across semesters | A validator checks prerequisite order, credit range, forbidden overlaps, unavailable terms, and the terms each course is usually offered. The LLM retries with the errors; a fixer repairs anything left. **No plan is shown unless it passes.** |
| **Explanations** | Writes a short paragraph per course naming the skills it covers | Every course code in the output is checked against the snapshot |
| **Conversational revision** | Handles edits ("swap INFO 3300 for something with less design", "I'm abroad in Fall 2027") and "why" questions | Updates change only the courses affected; everything is re-validated |
 
### 4.3 Technology stack (from Requirements Analysis)
 
| Layer | Choice |
|---|---|
| LLM | Free local model via **Ollama** or **LM Studio**; smaller local fallback model; hosted model only if a course-provided key is in `.env` |
| Agent framework | Python with **LangChain / LangGraph** |
| Web app | Server-rendered **FastAPI or Flask** with HTML templates |
| Search | Hybrid keyword + embeddings (**sentence-transformers**, local) |
| Data | **Cornell Class Roster API** (`classes.cornell.edu/api/2.0`), committed as a dated snapshot with a refresh script |
| Packaging | App in **Docker**; model runs natively on the host |
| Testing | Unit and **property-based tests** (Hypothesis expected) for all deterministic components |
 
---
 
## 5. Project Scope
 
### MVP features (by December 16, 2026)
 
| ID | Feature | Priority |
|---|---|---|
| F1 | Student profile intake: program, year, credit range (default 12–18), completed and in-progress courses, fixed courses, interests, topics to avoid, unavailable terms | Must |
| F2 | Career-based skill mapping, plus light comparison of 2–3 careers side by side | Must |
| F3 | Hybrid catalog search that returns only real courses | Must |
| F4 | Personalized, ranked recommendations with explanations, prerequisite status, and Class Roster links | Must |
| F5 | Prerequisite verification (Met / Not met / Needs review) | Must |
| F6 | Validated multi-semester plan | Must |
| F7 | Conversational revision | Must |
| F8 | Markdown export of the plan | Should |
| F9 | Advisory notice and official source links on every plan | Must |
 
### MVP boundaries
 
- **Subjects:** the ingestion pipeline supports any subject; evaluation and demo use 8 core subjects: CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON
- **Planning window:** Spring 2027 through Fall 2028 (up to 4 semesters). Fall 2026 courses count as in progress.
- **Course levels:** undergraduates see 1000–4000 level; master's students see 5000 level and above. Lower-level courses appear only as missing prerequisites.
- **Reviewed careers:** software engineer, data scientist, ML engineer, product manager, UX designer/researcher, quantitative analyst. Other goals are accepted and mapped live.
- **Privacy:** local only, no login, no name/NetID/email collected, nothing about the student saved to disk except an export they choose to download
### Out of scope for the MVP
 
Official degree audits and graduation requirement checks · enrollment or Student Center actions · Cornell SSO · transcript upload · section times and schedule conflicts · seat availability · course reviews and grade data · job-posting data · advisor accounts and sharing · saved profiles · AP/transfer credit rules · hosted or mobile deployment.
 
### Example journey
 
An Information Science junior who wants to be a *"data scientist at a hospital or health tech company"* enters her completed courses (CS 1110, CS 2110, INFO 1300, INFO 2950, MATH 1920, STSCI 2150). The agent maps 9 skills and marks 3 as already covered. She removes "deep learning" and adds "survey methods". She gets ranked recommendations and a 3-semester plan with one course flagged as uncertain for spring. She then types *"I'm abroad in Fall 2027"*. The agent moves the affected courses, re-checks prerequisites, and explains what shifted. Finally she exports the plan for her advisor.
 
---
 
## 6. Success Metrics
 
| Metric | Target | How it's measured |
|---|---|---|
| Fabricated courses | 0 across 10 evaluation profiles | Automated check |
| Prerequisite verdict accuracy | ≥ 90% agreement | 40 labeled cases, each reviewed by a team member |
| Plan validity | 100% of plans pass the validator | Automated |
| Explanation coverage | 100% name a mapped skill and show prerequisite status | Automated |
| Recommendation relevance | ≥ 80% of top-5 judged relevant | 10 profiles, 2 reviewers each |
| Time to first plan | < 90 seconds (median profile) | Timed runs on the team's fastest laptop |
| Perceived usefulness | ≥ 4 / 5 average | 5+ student testers outside the team |
 
One command runs all automated metrics and writes a dated results report.
 
---
 
## 7. Risks and Mitigations
 
| Risk | Mitigation |
|---|---|
| LLM names a course that doesn't exist | Courses come only from catalog tools; every code is validated before it is displayed; fabrications are logged |
| Prerequisite text is misread | Original text shown with every verdict; ambiguous cases → Needs review; accuracy measured on labeled cases |
| Course isn't offered in the planned term | Placement uses 4 years of offering history; uncertain courses carry a visible warning |
| Students treat the plan as official | Advisory notice on every plan and export |
| Local model too slow or weak for tool use | Timing measured early in Construction; smaller fallback model; more logic moved into deterministic code if needed |
| Class Roster API changes or goes down | App reads only from the committed snapshot at runtime |
| Prompt injection through course text or chat | Course-code validation and the plan validator are layered controls the LLM can't bypass |
| Scope grows beyond what 5 people can build | Out-of-scope list is binding; changes go through AI-DLC |
 
---
 
## 8. Team
 
| # | Name | NetID | Inception responsibilities |
|---|---|---|---|
| 1 | Timothy Kipruto | tkk24 | **GitHub setup and AI-DLC workflow:** create the repository and add members, install and configure AI-DLC v1.0.1, lead the shared workflow, maintain `audit.md` and `aidlc-state.md`, handle the final submission |
| 2 | Addis Sahle | as3734 | **README and product proposal:** problem statement, why an LLM agent fits, planned LLM integration and scope, team and collaboration plan |
| 3 | Carey Manwa | com34 | **Vision and requirements:** target users and objectives, MVP and out-of-scope features, open questions, lead Requirements Analysis |
| 4 | Venkata Sai Revanth Damisetty | vd259 | **Technical environment and application design:** Cornell data sources, tech stack and LLM, retrieval and recommendation design, agent tools, security requirements, testing framework, technical environment document and code examples |
| 5 | Erick Muriungi | ekm82 | **User stories, units of work, and QA:** user stories, units of work, evaluation profiles and test cases |
 
**Construction ownership** (set during AI-DLC Units Generation; see [`unit-of-work.md`](aidlc-docs/inception/application-design/unit-of-work.md)):

| Unit | Owner |
|---|---|
| U1 Catalog Data | Addis Sahle |
| U2 Prerequisite Engine | Carey Manwa |
| U3 Planner | Timothy Kipruto |
| U4 Agent and Services (plus the model spike and contracts step) | Venkata Sai Revanth Damisetty |
| U5 Web and Evaluation | Erick Muriungi |
 
---
 
## 9. Collaboration Plan
 
**Workflow: AI-DLC (v1.0.1).** The team follows the AI-Driven Development Life Cycle: Inception → Construction → Operations. Each stage ends at a **stage gate** where the whole team reviews the stage's documents and approves them before the next stage starts.
 
- Approvals and key decisions are logged in [`aidlc-docs/audit.md`](aidlc-docs/audit.md)
- Current progress is tracked in [`aidlc-docs/aidlc-state.md`](aidlc-docs/aidlc-state.md)
- Design questions are answered in shared question files (for example `requirement-verification-questions.md`), so every decision can be traced back to who chose what and why
**Version control**
- Shared GitHub repository; members commit directly to `main`
- Review happens at AI-DLC stage gates rather than per pull request
- The evaluation command is run before each stage approval to catch regressions
- Rollback is `git revert`; git history is the change record
- Secrets live only in a local `.env` created from `.env.example` (git-ignored)
**Communication and tracking**
- Team chat for day-to-day coordination
- Regular team syncs, plus a review meeting at each stage gate
- Bugs and wrong advice are tracked as GitHub issues and fixed through AI-DLC; each wrong-advice case is added to the evaluation set as a permanent regression check
**Timeline**
 
| Phase | Work | Target |
|---|---|---|
| Inception | Vision ✅, requirements ✅, user stories ✅, workflow planning ✅, application design ✅, units of work ✅ | Completed Nov 24, 2026 |
| Construction | Per-unit design and code, build and test, evaluation runs | Early Dec 2026 |
| Final | Evaluation report, demo video of both MVP journeys, submission | **Dec 16, 2026** |
 
---
 
## 10. Repository Structure
 
Current (Inception):
 
```
.
├── README.md                     # this file: product proposal and setup
├── CLAUDE.md                     # AI-DLC workflow rules for the coding agent
├── docs/
│   ├── vision.md                 # product vision and MVP definition
│   └── technical-environment.md  # tech stack, standards, example code
├── aidlc-docs/
│   ├── aidlc-state.md            # current stage and progress
│   ├── audit.md                  # log of decisions and approvals
│   └── inception/
│       ├── requirements/         # requirements + question files
│       ├── user-stories/         # stories and personas
│       ├── plans/                # execution plan and stage plans
│       └── application-design/   # components, services, units of work
└── .aidlc-rule-details/          # AI-DLC v1.0.1 rule files
```
 
Application code will be added at the repository root during Construction (never inside `aidlc-docs/`). Its layout will be set during Application Design.
 
---
 
## 11. Setup (Planned)
 
Exact commands will be finalized in the technical environment document. The planned flow:
 
1. Install **Docker** and a local model runtime (**Ollama** or **LM Studio**), then pull the primary and fallback models
2. Clone the repository
3. Copy `.env.example` to `.env` and set the model names (and a hosted key only if the course provides one)
4. Build and start the app with Docker; it reads the committed catalog snapshot
5. Open the app at `http://127.0.0.1:<port>`. The server is bound to localhost only.
6. (Optional) Run the refresh script to rebuild the catalog snapshot from the Class Roster API
7. Run the evaluation command to produce a results report
---
 
## 12. Key Documents
 
| Document | Contents |
|---|---|
| [`docs/vision.md`](docs/vision.md) | Problem, users, full vision, MVP scope, risks |
| [`docs/technical-environment.md`](docs/technical-environment.md) | Tech stack, models, security and testing standards, example code |
| [`aidlc-docs/inception/requirements/requirements.md`](aidlc-docs/inception/requirements/requirements.md) | Approved functional and non-functional requirements, with every decision traced |
| [`aidlc-docs/inception/user-stories/stories.md`](aidlc-docs/inception/user-stories/stories.md) and [`personas.md`](aidlc-docs/inception/user-stories/personas.md) | User stories with acceptance criteria, and personas |
| [`aidlc-docs/inception/plans/execution-plan.md`](aidlc-docs/inception/plans/execution-plan.md) | Which AI-DLC stages run, and the timeline |
| [`aidlc-docs/inception/application-design/application-design.md`](aidlc-docs/inception/application-design/application-design.md) | Components, services, and main flows |
| [`aidlc-docs/inception/application-design/unit-of-work.md`](aidlc-docs/inception/application-design/unit-of-work.md) | Units of work, owners, dependencies, and story map |
| [`aidlc-docs/aidlc-state.md`](aidlc-docs/aidlc-state.md) | Current AI-DLC stage |
| [`aidlc-docs/audit.md`](aidlc-docs/audit.md) | Decision and approval log |
