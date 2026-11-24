# Personas: Cornell AI Course Planning Agent

**Stage**: INCEPTION - User Stories
**Sources**: `requirements.md` section 3, `docs/vision.md` MVP Journeys 1 and 2, `story-generation-plan.md` answers Q2:B, Q3:B, Q4:B
**Status**: Draft for review by Member 5 and Member 3, then team approval (Q12:B)

All personas are fictional planning examples, not real students. The sample profiles seed the 10 evaluation profiles (FR-10.1).

> **Course code check**: every sample code is in the 8-subject core set (CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON), except one code that is deliberately outside it (noted below). Each code still has to be confirmed in the catalog snapshot once ingestion runs in Construction. Any code not found there is replaced before the profile becomes an evaluation profile.

---

## Summary

| ID | Persona | Type | Planning rules that apply | MVP journey |
|---|---|---|---|---|
| P1 | Maya, career-focused undergraduate | Student | 1000 to 4000-level courses (FR-3.7); 3 semesters left; unavailable term added in chat | Journey 1 |
| P2 | Daniel, professional master's student | Student | 5000 level and above, plus lower-level missing prerequisites (FR-3.7); 2 semesters; prior work entered as Cornell equivalents | Journey 2 |
| P3 | Priya, exploring first-year | Student | 1000 to 4000-level courses; career comparison before planning (FR-2.6); vague-goal follow-up questions (FR-2.2); 4-semester limit (FR-6.1) | None (extra stories) |
| P4 | Sam, major switcher | Student | 1000 to 4000-level courses; shortest valid prerequisite chain; reachability ranking (FR-4.3) | None (extra stories) |
| P5 | Team evaluator | Team member (developer) | Builds the snapshot, reviews evaluation cases, runs the evaluation, reads the developer log, configures models | None (evaluation and operations stories) |

Stakeholders who are not personas (Q2:B): academic advisors, who receive exported plans, and INFO 5940 course staff, who grade the product. Their needs appear in the advisory-notice and export stories (FR-8, FR-9).

---

## P1: Maya, Career-Focused Undergraduate

- **Program and year**: Information Science major, junior
- **Career goal**: "Data scientist at a hospital or health tech company"
- **Main frustration**: She knows the job she wants but not which of hundreds of courses build the skills for it, or whether she can take them in the 3 semesters she has left.
- **What she needs from the app**: A grounded plan she can bring to her advisor, with a reason for each course and checked prerequisites.
- **Rules that apply**:
  - Undergraduate level filter: 1000 to 4000 (FR-3.7)
  - Plan horizon: Spring 2027 to Spring 2028 (FR-6.1)
  - Offering-term rules and "uncertain" warnings (FR-6.4)
  - Incremental revision when she adds an unavailable term (FR-7.4)

**Sample profile** (from vision MVP Journey 1, Q4:B)

| Field | Value |
|---|---|
| Student type | Undergraduate |
| Expected graduation | Spring 2028 (3 semesters left: SP27, FA27, SP28) |
| Credit range | 14 to 17 per semester |
| Completed courses | CS 1110, CS 2110, INFO 1300, INFO 2950, MATH 1920, STSCI 2150 |
| In-progress (Fall 2026) | None |
| Fixed courses | None |
| Unavailable terms | None at the start; Fall 2027 is added in chat (study abroad) |
| Interests | Health data, statistics |
| Skill edits | Removes "deep learning", adds "survey methods" |

---

## P2: Daniel, Professional Master's Student

- **Program and year**: MPS in Information Science, from a non-CS background (undergraduate degree in economics)
- **Career goal**: "Product manager for AI products"
- **Main frustration**: He has a short program and cannot afford to enroll in a course and then find out he lacks the prerequisites.
- **What he needs from the app**: Courses he can take now, a clear "Not met" with the missing courses when he cannot take one, and a reachable alternative.
- **Rules that apply**:
  - Master's level filter: 5000 level and above. Lower-level courses appear only as missing prerequisites (FR-3.7, CQ8:A).
  - Plan horizon: 2 semesters (FR-6.1)
  - Prior non-Cornell work is entered as Cornell equivalents (requirements section 7 assumption)

**Sample profile** (from vision MVP Journey 2, Q4:B)

| Field | Value |
|---|---|
| Student type | Master's |
| Expected graduation | Fall 2027 (2 semesters: SP27, FA27) |
| Credit range | 15 to 18 per semester |
| Completed courses (entered as equivalents) | STSCI 2150, CS 1110 |
| In-progress (Fall 2026) | None |
| Fixed courses | None |
| Unavailable terms | None |
| Interests | AI products, user research |
| Key moment | A target machine learning course needs CS 2110 and linear algebra; the verdict is "Not met" with options (US-5.3) |

---

## P3: Priya, Exploring First-Year

- **Program and year**: College of Arts and Sciences, first-year, no major yet
- **Career goal**: "I like biology and coding but don't know what job I want" (adapted from the vision's full-scope Journey 2)
- **Main frustration**: She can't compare career paths in terms of the courses she would actually take.
- **What she needs from the app**: Follow-up questions that turn a vague interest into 2 to 3 named careers, a side-by-side view of their skills, then a plan for the career she picks.
- **Rules that apply**:
  - Vague-goal follow-up questions (FR-2.2)
  - Light career comparison, skill lists only (FR-2.6, Q17:B)
  - Undergraduate level filter (FR-3.7)
  - 4-semester limit even though she graduates later (FR-6.1)
  - Warning for a course code outside the loaded catalog (FR-1.2)

**Sample profile** (new, Q4:B)

| Field | Value |
|---|---|
| Student type | Undergraduate |
| Expected graduation | Spring 2030 (plan limited to SP27, FA27, SP28, FA28) |
| Credit range | 12 to 18 (default) |
| Completed courses | None |
| In-progress (Fall 2026) | CS 1110, MATH 1110, BIOG 1440 |
| Fixed courses | None |
| Unavailable terms | None |
| Interests | Biology, coding |
| Careers she might compare | Data scientist, ML engineer, plus one live-generated career such as "bioinformatics scientist" (not on the reviewed list, CQ13:B) |

*BIOG 1440 is deliberately outside the 8-subject core set. It tests the "code not found in the loaded catalog" warning (FR-1.2).*

---

## P4: Sam, Major Switcher

- **Program and year**: Sophomore moving from an Economics focus toward quantitative finance
- **Career goal**: "Quantitative analyst"
- **Main frustration**: He started late on the math and programming prerequisites and doesn't know the shortest path to the upper-level courses his target career needs.
- **What he needs from the app**: Recommendations he can reach soonest ranked first, the full prerequisite chain for a target course, and an honest message if the courses cannot all fit in 4 semesters.
- **Rules that apply**:
  - Reachability ranking (FR-4.3)
  - Prerequisite chain view (FR-5.9)
  - "Plan does not fit" conflict and relaxation choices (FR-6.6)
  - Fixed course pinned to a term (FR-1.4)

**Sample profile** (new, Q4:B)

| Field | Value |
|---|---|
| Student type | Undergraduate |
| Expected graduation | Spring 2029 (plan limited to SP27, FA27, SP28, FA28) |
| Credit range | 15 to 18 per semester |
| Completed courses | ECON 1110, ECON 1120, MATH 1110, MATH 1120 |
| In-progress (Fall 2026) | MATH 1920, ECON 3030 |
| Fixed courses | MATH 2940 pinned to Spring 2027 |
| Unavailable terms | None |
| Interests | Finance, optimization |
| Topics to avoid | Hardware |

---

## P5: Team Evaluator (Developer Persona)

- **Who**: Any of the 5 team members, acting as the person who builds the catalog snapshot, reviews evaluation data, runs the evaluation, and diagnoses wrong advice
- **Goal**: Show with evidence that the app meets the success metrics (NFR-2) before each stage approval and the December 16, 2026 demo
- **Main frustration**: LLM output varies from run to run. Without fixed test data, a single command, and logs, nobody can tell whether a change made the app better or worse.
- **What they need from the app**:
  - A resumable ingestion script and a dated snapshot (FR-3.1 to FR-3.4)
  - Agent-generated evaluation cases they can review and correct, with the reviewer recorded (FR-10.2)
  - One command that writes a dated results report (FR-10.3)
  - A way to turn a wrong-advice incident into a permanent regression case (FR-10.4)
  - A developer log with tool calls and timing but no student data (NFR-4.2)
  - Model choice by configuration only (NFR-6.3)
- **Environment**: Their own laptop (Windows, macOS, or Linux), the app in Docker, and the model runtime running natively (NFR-6.1)
