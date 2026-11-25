# Requirements: Cornell AI Course Planning Agent

**Stage**: INCEPTION - Requirements Analysis (Comprehensive depth)
**Sources**: `docs/vision.md`, `requirement-verification-questions.md` (Q1 to Q43), `requirement-clarification-questions.md` (CQ1 to CQ16)
**Date**: 2026-11-24
**Status**: Approved 2026-11-24. Amended 2026-11-24 by change request CR-1 and by the technical environment alignment (C3:A); see section 12.

---

## 1. Intent Analysis

| Item | Value |
|---|---|
| User request | "Using AI-DLC, we want to start our new project. Read the vison.md file. we are a team of 5 working on completing the inception phase of our project." |
| Request type | New Project (greenfield) |
| Scope estimate | System-wide: a new multi-component application (catalog ingestion, search, prerequisite engine, agent, planner, web UI, evaluation) |
| Complexity estimate | Complex: LLM agent with tool use, free-text prerequisite interpretation, constrained multi-semester planning, measurable evaluation targets |
| Requirements depth | Comprehensive |
| Team and deadline | 5 members; working demo by December 16, 2026 |

### Summary
A local web app that takes a student's profile and a free-text career goal and returns a multi-semester plan built only from real Cornell courses. It maps the goal to skills, searches the catalog, checks each course's prerequisites, places courses across Spring 2027 to Fall 2028, explains each choice, and lets the student revise the plan in chat. It runs on a free local LLM, and deterministic code checks every plan.

---

## 2. Decisions Taken From the Question Files

This table is the traceability index. Requirement IDs in later sections cite the decision IDs (Q = first question file, CQ = clarification file).

| Topic | Decision | Source |
|---|---|---|
| Primary LLM | Free local model through Ollama, running natively on each laptop (GPU or Apple Silicon where available). Model names are in `docs/technical-environment.md`. | Q1:B, CQ3:A, C3:A |
| Fallback LLM | A smaller local model in the same runtime by default. A hosted model is used only if a course-provided key is present in `.env`. | CQ4:D |
| Model selection on failure | No automatic switch on errors. Retry, then show a clear error. Primary or fallback model is chosen by configuration. | Q33:A, CQ4:D |
| Backend | Python with LangChain or LangGraph | Q2:B |
| UI | Server-rendered web app (for example FastAPI or Flask with HTML templates) | Q3:D |
| Search | Hybrid keyword plus embeddings, results merged | Q4:C |
| Embeddings | Local embedding model (for example sentence-transformers). A hosted embedding API may be used later if confirmed. | Q5:A |
| Runtime packaging | App runs in Docker on any OS; the model runs natively on the host and is called over localhost | Q6:C, CQ3:A |
| Subject coverage | Ingestion pipeline supports all 170+ subjects. MVP evaluation and demo commit to the 8-subject core set: CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON. More subjects load only if time allows. | Q7:C, CQ1:D |
| Catalog source | Committed dated snapshot; a refresh script any member can run | Q8:B |
| Roster history | 4 years of rosters (about FA22 to SP27) | Q9:C |
| Planning horizon | Plans start in Spring 2027; at most 4 future semesters for every student (SP27, FA27, SP28, FA28) | CQ5:A, Q11:B |
| In-progress courses | Fall 2026 courses are entered as in-progress and count as completed for all later terms | Q12:A, CQ5:A |
| Fixed courses | Student can pin known required courses to a term; they count toward that term's credits | Q13:A |
| Persistence | Session only. No student data is written to disk except a Markdown export the student chooses to download. | CQ7:D (replaces Q14:B) |
| Career input | Free text plus a pick list of reviewed careers; the agent asks guided questions when the goal is unclear | Q15:B |
| Skill mappings | LLM-generated, cached after team review; careers without a cached mapping use live generation | Q16:C |
| Reviewed careers | Software engineer, data scientist, ML engineer, product manager, UX designer or researcher, quantitative analyst, plus the careers in the 10 evaluation profiles | CQ13:B |
| Career comparison | In MVP, light: 2 to 3 careers with skill lists side by side, no full plans | Q17:B |
| Ranking | Skill coverage plus prerequisite reachability | Q18:B |
| Plan size | Career courses fill the credit range after fixed courses are placed | Q19:C |
| Course levels | Undergraduates: 1000 to 4000 level. Master's: 5000 level and above. Lower levels appear for either group only as missing prerequisites of a recommended course. | Q20:A, CQ8:A |
| Offering terms | Place a course only in a term type (fall or spring) it was offered in within the loaded history. A course not offered in the last 2 years is included with a visible "uncertain" warning. | CQ9:A |
| Prerequisite parsing | At load time. Simple text is parsed by pattern matching; complex text by the LLM. LLM parses must pass automated checks; failures are labeled "unverified". No manual review. | Q22:C, CQ2:D |
| Ambiguous prerequisites | "Or equivalent", permission of instructor, "recommended", and background-specific rules are always "Needs review" | Q23:A |
| "And/or" grouping | Documented default reading, flagged as an assumption in the verdict | Q24:B |
| Forbidden overlaps | Checked against completed courses and among courses in the same plan | Q25:B |
| Plan construction | LLM proposes; deterministic validator checks; LLM retries with the errors; deterministic fixer repairs remaining violations | Q26:C |
| Plan does not fit | Ask the student which constraint to relax | Q27:B |
| Explanations | Short paragraph per course | Q28:C |
| Tool-call visibility | Not shown to students; written to a developer log only | Q29:C |
| Conversational revision | Plan edits, "why" questions, and profile or skill changes. Profile changes only when the student states them explicitly. Plan updates are incremental; a full re-plan only on explicit request. | Q30:B, CQ10:B |
| Markdown export (F8) | Optional ("Should Have"); built if time allows | Q31:B |
| 90-second target | Median evaluation profile on the team's fastest laptop | Q32:B |
| LLM failure handling | Fixed number of retries, then a clear error; student inputs are kept | Q33:A |
| Secrets | `.env` created from a committed `.env.example`; `.env` is git-ignored | Q34:A |
| Accessibility | WCAG 2.1 AA for custom UI elements | Q35:B |
| Logging | Developer logs of tool calls and timing only; no profile content | CQ6:C (replaces Q36:B) |
| Evaluation data | An agent generates the 10 profiles and 40 prerequisite cases with expected verdicts; one team member reviews each verdict against the original text and corrects it | Q37:D, CQ11:B |
| Evaluation runner | One command runs all automated metrics and writes a results report | Q38:A |
| Team roles (Inception) | 1 GitHub and AI-DLC workflow; 2 README and proposal; 3 vision and requirements; 4 technical environment and application design; 5 user stories, units, and QA | Q39:D |
| Construction split | Decided in Units Generation; each member owns one or more units | CQ12:A |
| Version control | Direct commits to main; team review at AI-DLC stage gates | Q40:B |
| Extensions | Security Baseline: on. Resiliency Baseline: on. Property-Based Testing: on, full enforcement. | Q41:A, Q42:A, Q43:A |
| Recovery targets | Local app, no deployment. Recovery = re-clone the repository and rebuild the snapshot from the README. No disaster-recovery infrastructure. | CQ14:X |
| Change management | The AI-DLC workflow: approvals at stage gates logged in `audit.md`; git history is the change record; rollback is `git revert` | CQ15:X |
| Incident response | Bugs are GitHub issues fixed through AI-DLC. Wrong advice is recorded with its cause, and the case is added to the evaluation set as a regression check. | CQ16:X |

---

## 3. Users

| Persona | Planning rules that apply |
|---|---|
| Career-focused undergraduate | 1000 to 4000-level courses; up to 4 semesters |
| Exploring undergraduate | Uses career comparison (FR-2.6) before choosing one career to plan |
| Professional master's student | 5000-level and above, plus lower-level missing prerequisites; often enters non-Cornell work as Cornell equivalents |
| Major switcher | Needs the shortest valid prerequisite chain; relies on reachability ranking (FR-4.3) |

Stakeholders who are not users: academic advisors (receive exported plans), INFO 5940 staff (evaluate the product), the 5-person team.

---

## 4. Functional Requirements

Priority: **M** = Must Have, **S** = Should Have. Every functional requirement traces to a vision feature (F1 to F9).

### FR-1 Student Profile Intake (F1, M)
- **FR-1.1** The student enters program, college, year (undergraduate or master's), expected graduation term, and credit range per semester (default 12 to 18), through a form or chat.
- **FR-1.2** The student enters completed and in-progress courses as Cornell course codes, typed or pasted as a list. The app normalizes the codes (case, spacing) and warns about codes not found in the loaded catalog. A typed code not found in the loaded catalog does not satisfy prerequisites. Codes checked off in a prerequisite checklist follow FR-1.9 instead (C2:A).
- **FR-1.3** In-progress Fall 2026 courses count as completed for planning Spring 2027 onward (Q12:A).
- **FR-1.4** The student can enter fixed courses and pin each to a planned term; they count toward that term's credit total (Q13:A).
- **FR-1.5** The student can enter interests and topics to avoid.
- **FR-1.6** The student can mark one or more planned terms as unavailable (for example study abroad).
- **FR-1.7** The profile lives in memory for the session only, with a 60-minute idle timeout. Nothing about the student is written to disk (CQ7:D, B7:A).
- **FR-1.8** The app never asks for name, NetID, email, or any login.
- **FR-1.9** *(CR-1)* **Prerequisite checklist.** The app offers a course's prerequisite and corequisite items as a checklist, so the student can confirm what they have already done.
  - **FR-1.9.1** The checklist is offered for each course the student enters as in-progress or fixed, and, after the first plan is shown, for each recommended or planned course whose verdict is not Met (A1:B, C1:A).
  - **FR-1.9.2** Items are every course code in the rule, plus every non-course clause ("or equivalent", permission of instructor, "a course in linear algebra", background statements) as a separate item (A2:B).
  - **FR-1.9.3** A checked course code is added to the student's completed courses and satisfies that code wherever it appears in any course's prerequisites (A3:A, A5:B). This holds even when the code is outside the loaded catalog; such a code is shown with a "not in loaded catalog" note (C2:A).
  - **FR-1.9.4** A checked non-course clause is stored for that course only. The same clause text in another course is not treated as confirmed (A5:B).
  - **FR-1.9.5** Unchecked items count as not done, and verdicts follow the normal rules (A6:A).
  - **FR-1.9.6** Confirmations are part of the session profile only, and they appear in the Markdown export if it is built (A7:A, FR-1.7).
  - **FR-1.9.7** The profile shows a "Prerequisites I've confirmed" list where any item can be removed (A8:A).
  - **FR-1.9.8** Confirming or removing an item re-checks verdicts and placements incrementally (FR-7.4). It does not rebuild the plan, and it is not counted in the first-plan time (C1:A, NFR-1.1).

### FR-2 Career and Skill Mapping (F2, M)
- **FR-2.1** The student enters a career goal as free text or picks it from the reviewed-career list (Q15:B, CQ13:B).
- **FR-2.2** If the goal is too vague to map (for example "something with computers"), the agent asks guided follow-up questions before mapping (Q15).
- **FR-2.3** The goal maps to 6 to 12 named skills, each with a one-line reason.
- **FR-2.4** For a reviewed career, the cached team-reviewed mapping is used. Otherwise the LLM generates the mapping live (Q16:C). The UI shows which source was used.
- **FR-2.5** The student can add, remove, or reword skills before recommendations run.
- **FR-2.6** Career comparison: the student can enter 2 to 3 careers and see their skill lists side by side, then pick one to plan (Q17:B). No full plans are built for the comparison.
- **FR-2.7** Skills already covered by completed courses are marked "covered".

### FR-3 Catalog Data and Search (F3, M)
- **FR-3.1** An ingestion script fetches course data from the Class Roster API for a configured list of subjects and terms and writes a dated snapshot (Q8:B).
- **FR-3.2** The ingestion pipeline works for any Cornell subject. The default configuration is the 8-subject core set and 4 years of rosters (CQ1:D, Q9:C).
- **FR-3.3** The snapshot keeps, per course: code, title, description, learning outcomes, credits, prerequisite and corequisite text, forbidden overlaps, "when offered" text, terms offered in the loaded history, distribution categories, and the Class Roster URL.
- **FR-3.4** The ingestion script respects the API's published request-rate guidance and can resume after an interruption.
- **FR-3.5** Search is hybrid: keyword matches and embedding similarity, merged into one ranked list (Q4:C, Q5:A).
- **FR-3.6** Search returns only courses present in the loaded snapshot.
- **FR-3.7** Level filtering by student type, as in the Q20 and CQ8 decisions: undergraduates see 1000 to 4000-level courses, master's students see 5000 level and above. A lower-level course appears only as a missing prerequisite of a recommended course.

### FR-4 Personalized Recommendations (F4, M)
- **FR-4.1** The app produces a ranked list of courses for the mapped skills.
- **FR-4.2** Each entry shows: course code, title, credits, skills it covers, a short-paragraph explanation (Q28:C), prerequisite status (FR-5), offering-term status (FR-6.4), and a link to the Class Roster page.
- **FR-4.3** Ranking uses skill coverage plus prerequisite reachability: courses the student can take sooner rank higher (Q18:B). Topics to avoid lower the rank.
- **FR-4.4** Completed and in-progress courses are excluded, along with forbidden overlaps with them.
- **FR-4.5** Every course code in any agent output is checked against the snapshot before it is displayed. A code not in the snapshot is removed and logged as a fabrication event (vision risk 1).

### FR-5 Prerequisite Verification (F5, M)
- **FR-5.1** Prerequisite and corequisite text is parsed into structured rules at load time and stored with the snapshot (Q22:C).
- **FR-5.2** Text that matches known simple patterns (single course, "A or B", "A and B", comma lists) is parsed by deterministic pattern matching. All other text is parsed by the LLM (CQ2:D).
- **FR-5.3** Each LLM-parsed rule must pass all automated checks: (a) course-code cross check (every code in the rule appears in the original text, and every code in the text appears in the rule); (b) catalog existence of each referenced code; (c) keyword flags for "equivalent", "permission", "recommended", "or" and "and" grouping; (d) agreement between two independent parses. A rule that fails any check is stored as **unverified** (CQ2:D).
- **FR-5.4** A deterministic evaluator applies the stored rule to the student's completed, in-progress, and earlier-planned courses. It returns one of:
  - **Met**
  - **Not met**, listing the missing courses
  - **Needs review**, with the reason
- **FR-5.5** "Or equivalent", permission of instructor, "recommended", and background-specific clauses produce **Needs review** (Q23:A), unless the student has confirmed that clause for that course in the checklist (FR-1.9.4, FR-5.11).
- **FR-5.6** Ambiguous "and/or" grouping uses this documented default reading: commas and semicolons separate ANDed groups, and within a group "and" binds tighter than "or". The verdict notes that an assumption was used (Q24:B).
- **FR-5.7** A verdict from an **unverified** rule is shown as **Needs review (unverified parse)** (confirmed at Requirements approval). Its checklist lists the course codes found in the original text. Checking them adds them to completed courses (FR-1.9.3), but the verdict stays **Needs review (unverified parse)**, because the rule's logic is not trusted.
- **FR-5.8** Every verdict shows the original prerequisite text.
- **FR-5.9** The app can show the full prerequisite chain for any target course.
- **FR-5.10** Forbidden overlaps are detected against completed courses and among courses inside the same plan (Q25:B).
- **FR-5.11** *(CR-1)* The evaluator receives the student's confirmed clauses for the course being checked. A confirmed clause counts as satisfied. A **Met** verdict that depends on at least one confirmed clause is shown as **Met (student-confirmed)** (A4:A). The verdict status stays one of the three types (Met, Not met, Needs review); "student-confirmed" is a marker on Met.

### FR-6 Semester Planning (F6, M)
- **FR-6.1** Plans cover Spring 2027 through at most Fall 2028 (4 semesters), limited by the student's expected graduation term and unavailable terms (CQ5:A, Q11:B, FR-1.6).
- **FR-6.2** Fixed courses are placed first. Career courses then fill each term up to the credit range (Q13:A, Q19:C).
- **FR-6.3** The LLM proposes a placement. A deterministic validator checks it. If the check fails, the LLM receives the errors and retries a fixed number of times. A deterministic fixer then repairs any remaining violations (Q26:C).
- **FR-6.4** A course is placed only in a term type (fall or spring) in which it was offered in the loaded history. A course not offered in the last 2 years carries a visible "uncertain" warning (CQ9:A).
- **FR-6.5** The validator enforces:
  - every course is placed after its prerequisites, and corequisites are in the same or an earlier term
  - each term's credits are within the student's range
  - no forbidden overlaps
  - no course in an unavailable term
  - offering-term rules (FR-6.4)
- **FR-6.6** If the chosen courses cannot all fit, the agent explains the conflict and asks which constraint to relax: credit range, number of courses, or an added semester within the 4-semester limit (Q27:B).
- **FR-6.7** No plan is shown to the student unless it passes the validator.

### FR-7 Conversational Revision (F7, M)
- **FR-7.1** The student can request plan edits in chat: swap, move, add, or remove a course; change credits; block a term.
- **FR-7.2** The student can ask "why" questions about any course or placement. Answers cite the mapped skills and catalog data.
- **FR-7.3** The student can change profile fields or the skill list in chat. The agent changes the profile only when the student states the change explicitly (Q30).
- **FR-7.4** Updates are incremental: courses already placed stay in place, the agent re-checks prerequisites, credits, and offering terms, changes only the affected courses, and explains each change (CQ10:B).
- **FR-7.5** A full re-plan happens only when the student asks for one explicitly (for example "start over") (CQ10:B).
- **FR-7.6** At least 3 revisions in one session keep earlier student choices (vision MVP criterion).

### FR-8 Plan Export (F8, S)
- **FR-8.1** The student can download the current plan, skills, explanations, prerequisite verdicts, and the advisory notice, and the student's confirmed prerequisites (FR-1.9.6) as a Markdown file (Q31:B, built if time allows).

### FR-9 Advisory Notice and Sources (F9, M)
- **FR-9.1** Every plan view and every export states that the plan is advisory, does not check graduation requirements, and must be confirmed with an advisor.
- **FR-9.2** Every course links to its official Class Roster listing.

### FR-10 Evaluation Harness (supports vision success metrics)
- **FR-10.1** The repository contains 10 evaluation profiles and 40 prerequisite cases. All of them use courses in the 8-subject core set (CQ1:D).
- **FR-10.2** An agent generates the profiles, the cases, and the expected verdicts. One team member reviews each expected verdict against the original prerequisite text and corrects it. Each case records who reviewed it (Q37, CQ11:B).
- **FR-10.3** One command runs all automated metrics and writes a dated results report (Q38:A):
  - fabricated course count
  - prerequisite verdict agreement
  - plan validity
  - explanation coverage
  - time to first plan
- **FR-10.4** Cases added after a reported wrong-advice incident are kept as permanent regression cases (CQ16).

---

## 5. Non-Functional Requirements

### NFR-1 Performance
- **NFR-1.1** Time to a first plan is under 90 seconds on the median evaluation profile, on the team's fastest laptop, using the primary model (Q32:B).
- **NFR-1.2** Every LLM and HTTP call has an explicit timeout: 60 seconds per LLM call and 10 seconds per Class Roster request (RESILIENCY-10, C3:A).

### NFR-2 Accuracy and Grounding
- **NFR-2.1** 0 fabricated courses across the 10 evaluation profiles.
- **NFR-2.2** Prerequisite verdicts agree with the reviewed labels on at least 90% of the 40 cases.
- **NFR-2.3** 100% of generated plans pass the validator.
- **NFR-2.4** 100% of recommendations name at least one mapped skill and show prerequisite status.
- **NFR-2.5** At least 80% of top-5 recommendations are judged relevant (10 profiles, 2 reviewers each).
- **NFR-2.6** Perceived usefulness averages at least 4 out of 5 from at least 5 student testers outside the team.

### NFR-3 Reliability and Error Handling
- **NFR-3.1** LLM calls are retried a fixed 3 attempts (C3:A). After that the app shows a clear, generic error message and keeps the student's inputs and current plan (Q33:A, SECURITY-15).
- **NFR-3.2** If the model runtime is unreachable, the app says so at startup and in the UI, and shows how to fix it (RESILIENCY-06, shallow and deep checks).
- **NFR-3.3** A global error handler catches unhandled exceptions, logs them, and returns a safe response with no stack trace (SECURITY-09, SECURITY-15).
- **NFR-3.4** The app reads only the local snapshot at runtime. It does not depend on the Class Roster API being up (vision risk: API changes).

### NFR-4 Security (Security Baseline enabled)
- **NFR-4.1** Secrets live only in `.env`, created from a committed `.env.example`. No keys in code or in the repository (Q34:A, SECURITY-12).
- **NFR-4.2** No student data is persisted. Logs hold tool names, timing, and errors only, with a timestamp, request ID, level, and message. Logs contain no profile content, no secrets, and no tokens (CQ6:C, SECURITY-03).
- **NFR-4.3** All form, chat, and API inputs are validated with a schema that enforces type, maximum length, and allowlisted formats (for example the course-code pattern). Payload size limits are set (request body at most 64 KB; per-field limits in `docs/technical-environment.md`, Input Validation). User text is escaped before rendering (SECURITY-05, C3:A).
- **NFR-4.4** HTML responses set Content-Security-Policy (`default-src 'self'`, no `unsafe-inline` without a documented reason), `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Referrer-Policy: strict-origin-when-cross-origin` (SECURITY-04).
- **NFR-4.5** The server binds to `127.0.0.1` only. CORS allows only the app's own origin (SECURITY-07, SECURITY-08).
- **NFR-4.6** Dependencies are pinned in a lock file. Docker base images use pinned tags, not `latest`. `pip-audit` runs in GitHub Actions CI on every push, and Dependabot alerts are enabled (SECURITY-10, C3:A).
- **NFR-4.7** Text returned by the LLM and data loaded from the snapshot are parsed with schema validation, never with unsafe deserialization. Front-end assets are served locally, or with SRI hashes if loaded from a CDN (SECURITY-13).
- **NFR-4.8** Prompt-injection misuse case: text in course descriptions or student chat must not be able to make the agent name non-catalog courses or skip validation. The FR-4.5 code check and the FR-6.7 validator are the layered controls (SECURITY-11).
- **NFR-4.9** Chat requests are rate-limited to 10 per minute per session (C3:A), so a runaway loop cannot overload the local model (SECURITY-11).

### NFR-5 Usability and Accessibility
- **NFR-5.1** Custom UI elements meet WCAG 2.1 AA: contrast, labels, focus order, and full keyboard operation (Q35:B).
- **NFR-5.2** Verdicts and warnings are shown in text, not by color alone.

### NFR-6 Portability and Setup
- **NFR-6.1** The app runs in Docker on the team's macOS (Apple Silicon) and Windows 10/11 laptops, including laptops without a GPU (C3:A). The model runtime runs natively on the host and the app reaches it over localhost (Q6:C, CQ3:A).
- **NFR-6.2** The full flow runs on all 5 team laptops by following the README.
- **NFR-6.3** Primary model, fallback model, and the optional hosted key are set by configuration with no code change (CQ4:D).

### NFR-7 Maintainability and Testability
- **NFR-7.1** Deterministic components are separate from LLM components and unit-testable without a model:
  - prerequisite pattern parser
  - rule evaluator
  - plan validator
  - plan fixer
  - credit arithmetic
  - course-code checks
- **NFR-7.2** Property-based tests (full PBT enforcement) cover the deterministic components. Example invariants:
  - every valid plan places each course after its prerequisites
  - each term stays within its credit range
  - the fixer's output always passes the validator
  - rule serialization round-trips
- **NFR-7.3** Example-based tests pin the two MVP user journeys and the labeled prerequisite cases (PBT-10).

### NFR-8 Cost
- **NFR-8.1** The MVP runs at zero cost on local models. A hosted model is used only if a course-provided key is present.

---

## 6. Constraints
- Local execution only. No hosting, no user accounts, no Cornell SSO.
- Public Cornell data only: Class Roster API, with Courses of Study for reference.
- Advisory only. No claims about graduation requirements.
- Hard deadline: December 16, 2026.
- Changes after approval go through the AI-DLC workflow (CQ15).

## 7. Assumptions
- The Class Roster API stays public and returns 4 years of rosters for the core subjects. If it does not, the committed snapshot is used.
- A local model that runs on the team's fastest laptop can follow tool-calling and structured-output instructions well enough to meet NFR-1.1 and NFR-2. If it cannot, logic moves into deterministic code (vision assumption).
- Offering history in 4 years of rosters predicts future fall and spring offerings well enough. Courses outside that pattern are flagged.
- Students can express prior coursework as Cornell course codes.
- FR-5.7 (unverified parse shown as "Needs review") is an AI interpretation of CQ2. The team should confirm it during review.

## 8. Out of Scope (MVP)
Same as the vision's out-of-scope table, with these changes from the answers:
- **All 170+ subjects**: the pipeline supports them, but MVP evaluation and the demo use the 8-subject core set. Other subjects are loaded only if time allows (CQ1:D).
- **Saved profiles**: no save or load of profiles or plans to disk. Session only (CQ7:D).
- **Planning the rest of Fall 2026**: excluded; plans start in Spring 2027 (CQ5:A).
- **Multi-career comparison with full plans**: excluded; only skill-list comparison is in scope (Q17:B).
- **Showing tool calls to students**: excluded; tool calls go to the developer log only (Q29:C).

## 9. Risks Introduced by the Decisions

| Risk | Source | Mitigation |
|---|---|---|
| Master's students do not see 3000 and 4000-level courses that suit their career (for example CS 4780) unless those courses are prerequisites | CQ8:A | Accepted by the team. Check during evaluation; revisit if relevance scores for master's profiles fall below target. |
| Agent-generated expected verdicts may share the app model's blind spots | CQ11:B | Reviewer checks each verdict against the original text; reviewer is recorded per case. |
| Local model too slow for the 90-second target | Q1:B, Q32:B | Native runtime (CQ3:A), smaller fallback model, timing measured early in Construction |
| Unverified prerequisite parses are never reviewed by hand | CQ2:D | Shown as "Needs review (unverified parse)" with the original text; accuracy measured on the 40 cases |
| Without save, a student loses work when the app closes | CQ7:D | F8 export, if built, is the way to keep a plan |
| Direct commits to main with no per-commit review | Q40:B | Stage-gate reviews; evaluation command run before each stage approval |

## 10. Extension Configuration and Requirements-Stage Compliance

### Security Baseline (enabled)
| Rule | Status | Rationale |
|---|---|---|
| SECURITY-01 Encryption at rest/in transit | Compliant (scoped) | No student data is persisted (FR-1.7). The only store is the public catalog snapshot and search index, documented as non-sensitive public data. The model is called over localhost, and a hosted model would be called over HTTPS. |
| SECURITY-02 Access logging on intermediaries | N/A | No load balancer, API gateway, or CDN |
| SECURITY-03 Application logging | Compliant | NFR-4.2. Centralized log service adapted to a single local log file, because there is no hosting. |
| SECURITY-04 HTTP security headers | Compliant (scoped) | NFR-4.4. HSTS is N/A because the app is served over HTTP on localhost only. |
| SECURITY-05 Input validation | Compliant | NFR-4.3 |
| SECURITY-06 Least-privilege IAM | N/A | No cloud identities or IAM policies |
| SECURITY-07 Network configuration | Compliant | NFR-4.5: bound to localhost |
| SECURITY-08 Application access control | Compliant (scoped) | Single local user with no accounts. CORS restricted (NFR-4.5). No resource IDs are shared across users. |
| SECURITY-09 Hardening | Compliant | NFR-3.3; no default credentials; no sample endpoints |
| SECURITY-10 Supply chain | Compliant | NFR-4.6. SBOM applies to production deployments only, so N/A. |
| SECURITY-11 Secure design | Compliant | NFR-4.8 misuse case; NFR-4.9 rate limiting |
| SECURITY-12 Authentication and credentials | Compliant (scoped) | No authentication in the MVP. No hardcoded secrets (NFR-4.1). |
| SECURITY-13 Integrity | Compliant | NFR-4.7. GitHub Actions CI installs from the committed lock file (`uv sync --frozen`); HTMX is vendored with a recorded checksum (C3:A). |
| SECURITY-14 Alerting and monitoring | N/A | No deployed service or authentication events. Local log only. |
| SECURITY-15 Exception handling | Compliant | NFR-3.1, NFR-3.3 |

### Resiliency Baseline (enabled)
| Rule | Status | Rationale |
|---|---|---|
| RESILIENCY-01 Workload criticality | Deferred to Application Design | Components are identified there |
| RESILIENCY-02 Availability and recovery targets | Compliant | CQ14: local app; recovery = re-clone and rebuild snapshot; no SLA or DR infrastructure, justified by the absence of deployment and users |
| RESILIENCY-03 Change management | Compliant | CQ15: AI-DLC stage gates, `audit.md`, git history, `git revert` |
| RESILIENCY-04 Deployment and rollback | Deferred to NFR Requirements | CI/CD, rollback, and deployment-style questions are asked there |
| RESILIENCY-05 Monitoring | Compliant (scoped) | Local structured log with timing (NFR-4.2). Dashboards N/A with no deployment. |
| RESILIENCY-06 Health checks | Compliant | NFR-3.2 |
| RESILIENCY-07 Resiliency monitoring | N/A | No deployed infrastructure |
| RESILIENCY-08 Multi-zone/region | N/A | Local only; consistent with CQ14 |
| RESILIENCY-09 Auto-scaling | N/A | Single local user |
| RESILIENCY-10 Timeouts and degradation | Compliant | NFR-1.2, NFR-3.1, NFR-3.4 |
| RESILIENCY-11 to 13 DR, backup, failover | Compliant (scoped) | No student data to back up. Code and snapshot are in git. Recovery steps go in the README (CQ14). |
| RESILIENCY-14 Resiliency testing | Deferred to NFR Design | Required question asked there |
| RESILIENCY-15 Incident response | Compliant | CQ16: GitHub issues, AI-DLC fixes, regression cases (FR-10.4) |

### Property-Based Testing (enabled, full)
| Rule | Status | Rationale |
|---|---|---|
| PBT-01 to PBT-10 | N/A at Requirements | Enforced from Functional Design onward. NFR-7.2 records the candidate properties. Hypothesis is the expected framework for Python (decided at NFR Requirements, PBT-09). |

## 11. Recommended Updates to `docs/vision.md`
These are not made automatically; the vision owner (team role 3) decides.
- Out-of-scope table: change "All 170+ subjects" to "pipeline supports all subjects; evaluation on the 8-subject core set".
- MVP feature table: add light career comparison (Q17:B).
- Risk table: name the fallback model (CQ4:D).
- Open Questions: mark all 15 as resolved, pointing to this document.
- Add the technical environment document the vision refers to (team role 4). Section 2 above has its content.

## 12. Change Log

| Date | Change | Source | Requirements affected |
|---|---|---|---|
| 2026-11-24 | **CR-1: prerequisite checklist stored in the profile.** Students confirm a course's prerequisite items; confirmed course codes carry over to every course, confirmed clauses stay with that course. | Workflow Planning approval message; `application-design-plan.md` A1 to A8, C1, C2 | FR-1.2, new FR-1.9, FR-5.5, FR-5.7, new FR-5.11, FR-8.1 |
| 2026-11-24 | **Technical environment alignment.** `docs/technical-environment.md` is binding where it differs: Ollama only; macOS and Windows machines only; GitHub Actions CI with `pip-audit` and Dependabot; 3 retries; 60 s LLM and 10 s roster timeouts; 10 chat requests per minute; 60-minute session idle timeout; input limits. | `application-design-plan.md` C3:A | Section 2 (Primary LLM), FR-1.7, NFR-1.2, NFR-3.1, NFR-4.3, NFR-4.6, NFR-4.9, NFR-6.1, section 10 (SECURITY-13) |
