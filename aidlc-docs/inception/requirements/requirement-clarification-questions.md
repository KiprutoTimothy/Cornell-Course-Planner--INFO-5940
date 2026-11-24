# Requirements Clarification Questions

**Stage**: INCEPTION - Requirements Analysis (follow-up to `requirement-verification-questions.md`)

The AI checked your 43 answers against each other, against `docs/vision.md`, and against the Security, Resiliency, and Property-Based Testing extensions you opted into (Q41 to Q43 = A). It found the issues below. Answer them the same way: put a letter after each `[Answer]:` tag, or choose the last option (Other) and describe your answer.

All other answers are accepted as given. Requirements will be written from them once this file is complete.

---

## Contradiction 1: Subject coverage vs. the vision and your capacity concern
You chose **all 170+ subjects** (Q7:C) and **4 years of roster history** (Q9:C), and wrote that you are "wary of how the previous data will be fetched and stored and if we have that capacity". The vision's out-of-scope table defers "All 170+ Cornell subjects" to Phase 2 because of "larger data load and evaluation effort". You also chose to pre-parse prerequisite text with the LLM at load time and have the team review it (Q22:C), using a **local model** (Q1:B). Across all subjects that means thousands of courses: hours of local-model time and far more parsed rules than 5 people can review.

### Clarification Question 1
What should the MVP load?

A) All 170+ subjects with 4 years of history, as answered. Update the vision's out-of-scope table and accept the load and review effort.

B) All 170+ subjects, but only the last 2 years of history (FA24 to SP27), to cut the data volume roughly in half

C) The 8-subject set (CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON) with 4 years of history. Make subjects a configuration list so more can be added later.

D) Build the pipeline for all subjects, but commit to the 8-subject set for the MVP evaluation; load more subjects only if time allows

E) Other (please describe after [Answer]: tag below)

[Answer]: D

### Clarification Question 2
Pre-parsing prerequisites with the LLM at load time (Q22:C) produces one stored rule per course. Which stored rules must a team member review?

A) Only the rules the parser flags as ambiguous, plus the courses in the 40 labeled evaluation cases

B) Every rule for courses in the 8-subject set (or the chosen core set); other subjects stay unreviewed and are labeled "unreviewed" in the app

C) None are reviewed by hand. Accuracy is measured only by the 40 labeled cases.

D) Other (please describe after [Answer]: tag below)

[Answer]: D. Easy cases are parsed by pattern matching. The LLM parses only complext text. Every LLM-parsed rule must pass automated checks: course-code cross check, catalog existence, keyword flags, and agreement between parses. Rules that fail are labeled 'unverified' in the app wit hthe original text shown, and no one reviews them by hand. Accurancy is measured on the 40 human-labeled cases.

---

## Contradiction 2: Local model, Docker, and the 90-second target
You chose a **free local model** (Q1:B), **Docker on any OS** (Q6:C), and the **90-second median target on the fastest laptop** (Q32:B). On macOS and on Windows laptops without a supported GPU setup, a model inside a Docker container runs on the CPU only, which can be several times slower than running it natively. The vision also says the technical environment must name a **fallback model**, and none is chosen.

### Clarification Question 3
Where does the local model run?

A) Ollama (or LM Studio) runs natively on each laptop, using the GPU or Apple Silicon where available. The app runs in Docker and calls it over localhost.

B) The model and the app both run in Docker Compose. Accept CPU-only speed on Mac and on most Windows laptops.

C) No Docker. Native Python install from a pinned lock file, and the model runs natively.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

### Clarification Question 4
What is the fallback model?

A) A smaller, faster local model (same runtime), used when the primary model is too slow on a given laptop

B) An optional hosted API model, used only when a key is present in `.env`

C) No fallback model. Update the vision's risk table.

D) Other (please describe after [Answer]: tag below)

[Answer]: D, a smaller local model by default, in the same runtime. If the course provides an API key, a hosted model is used when the key is present in .env. That keeps the app working offline and makes the key a bonus rather than a dependency.

---

## Contradiction 3: Planning Fall 2026 vs. counting Fall 2026 as complete
You chose to **allow the remainder of Fall 2026** in plans (Q10:B) and also to **count in-progress Fall 2026 courses as completed** (Q12:A). Today is November 24, 2026: Fall 2026 add deadlines have passed and the term ends in a few weeks, so a student cannot add Fall 2026 courses.

### Clarification Question 5
How should Fall 2026 appear?

A) Plans start in Spring 2027. Fall 2026 courses are entered as in-progress and count as completed (Q10 becomes A).

B) Fall 2026 appears in the plan as a fixed, read-only term showing the student's in-progress courses. The planner adds nothing to it.

C) Fall 2026 is open for new courses, because the app may be used by a student before the term starts (for example, if the app is reused in a later year, the "current term" is configurable)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Contradiction 4: Full session transcripts vs. Security rule SECURITY-03
You chose to **store full session transcripts locally** (Q36:B) and to **enforce all Security rules** (Q41:A). SECURITY-03 requires that "Sensitive data (passwords, tokens, PII) MUST NOT appear in log output". Transcripts contain the student's course history, program, year, and goals. SECURITY-01 also requires encryption at rest for every data store. That covers transcript files and the saved profile/plan JSON files (Q14:B).

### Clarification Question 6
How should session logging work?

A) Keep full transcripts, and record in the requirements that the app never asks for a name, NetID, or email. Course history alone is classified as non-identifying. Transcripts go to a git-ignored local folder.

B) Full transcripts only when a debug flag is turned on (Q36 becomes C). Default logs hold tool calls and timing only.

C) Developer logs only (tool calls and timing, no profile content) (Q36 becomes A)

D) Other (please describe after [Answer]: tag below)

[Answer]: C

### Clarification Question 7
How should SECURITY-01 (encryption at rest) be met for local files: saved profiles and plans, any transcripts, and the catalog store?

A) Require OS full-disk encryption (BitLocker or FileVault) on every team laptop and document it as the at-rest control. Public catalog data is documented as non-sensitive.

B) The app encrypts files that hold student data (profile, plan, transcripts) with a key from `.env`. The public catalog store is documented as a non-sensitive exception.

C) Both A and B

D) Remove the saved-profile feature (Q14 becomes A) and keep no student data on disk except the Markdown export the student chooses to download

E) Other (please describe after [Answer]: tag below)

[Answer]: D

---

## Ambiguity 1: Course levels for master's students
Q20 was answered "A, for the master's students, see 5000 level+ classes". Many courses central to MPS and MEng students are at the 4000 level (for example CS 4780 Machine Learning), and Journey 2 shows a master's student needing CS 2110 as a prerequisite.

### Clarification Question 8
What should master's students see by default?

A) 5000-level and above only. Lower levels appear only as missing prerequisites for a recommended course.

B) 4000-level and above. Lower levels appear only as missing prerequisites.

C) 5000-level and above ranked first, with 3000 and 4000-level courses also shown but ranked lower. Lower levels appear as missing prerequisites.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 2: Courses with uncertain offering terms
Q21 was answered "D, if getting the data from the public API class roster, it's usually organized by semester so this shouldn't be an issue". Rosters are organized by semester, but that does not settle future terms. A course may appear in the snapshot only once (for example in Fall 2023), or only in alternate years, and the plan has to guess whether it will be taught in Fall 2027.

### Clarification Question 9
How should the planner treat a course's offering pattern?

A) Place a course only in a term type (fall or spring) in which it was offered in the loaded history. If it has not been offered in the last 2 years, include it with a visible "uncertain" warning.

B) Same as A, but exclude courses not offered in the last 2 years

C) Use the roster's "when offered" text first and history second. Warn whenever the two disagree or both are missing.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 3: What happens to the plan after a profile or skill change
Q30 was answered "B, but change the profile only when mentioned specifically, no full re-plan". The vision's success criterion requires "at least three revisions in one session without losing earlier choices", but the answer does not say what happens to the existing plan after the student changes their profile or skills.

### Clarification Question 10
After a profile or skill change in chat, what happens to the plan?

A) Incremental update: courses already placed stay where they are; the agent re-checks prerequisites and credits, then adds, removes, or moves only the affected courses and explains each change

B) Incremental by default, plus a full re-plan only when the student asks for it explicitly (for example "start over")

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Ambiguity 4: Agent-generated evaluation labels
Q37 was answered "D, assign an agent to generate the evaluation data and reviewed by one team member". If an LLM writes the expected prerequisite verdicts and the app's LLM produces the verdicts being tested, the "90% agreement" metric may measure how much two models agree, not whether they are correct.

### Clarification Question 11
How are the expected answers for the 40 prerequisite cases set?

A) The agent drafts the 10 profiles and picks the 40 cases. A team member sets each expected verdict by reading the original prerequisite text without seeing any model's verdict.

B) The agent generates the cases and the verdicts. One team member reviews each verdict against the original text and corrects it.

C) Same as B, but the agent uses a different model from the app's model

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Ambiguity 5: Construction work split
Q39 lists each member's responsibilities for Inception and project deliverables (repository and AI-DLC, README and proposal, vision and requirements, tech environment and application design, user stories and units and QA). It does not say who builds which part of the code during Construction.

### Clarification Question 12
How should Construction work be split?

A) Decide during Units Generation, once the units are known (each member owns one or more units)

B) By layer now: data ingestion; prerequisite engine; agent and tools; planner and validator; UI and evaluation

C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Ambiguity 6: Supported careers for the pick list and cache
Q15:B (pick list of team-reviewed careers) and Q16:C (cache mappings after team review) need a starting list.

### Clarification Question 13
Which careers are on the reviewed list for the MVP?

A) The six in the vision: software engineer, data scientist, ML engineer, product manager, UX designer or researcher, quantitative analyst

B) The six above, plus the careers in the 10 evaluation profiles

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Resiliency Extension: Required Questions

You enabled the Resiliency Baseline (Q42:A). These questions are required during Requirements Analysis. The extension's other decisions (CI/CD tooling, rollback mechanism, deployment style, regional topology, resiliency testing approach) will be asked at NFR Requirements or NFR Design. Because the MVP runs only on local laptops, many resiliency rules are likely to be N/A. Your answers here set that scope.

## Clarification Question 14: RTO/RPO Goals and Disaster Recovery Strategy
What are your Recovery Time Objective (RTO) and Recovery Point Objective (RPO) goals? These determine the appropriate Disaster Recovery strategy and infrastructure redundancy level.

A) RPO/RTO: Hours. Backup & Restore strategy. Lowest cost ($). Data backed up, no services deployed. Redeploy from IaC and restore from backups on failure. Suitable for non-critical workloads.

B) RPO/RTO: 10s of minutes. Pilot Light strategy. Cost: $$. Data live, services idle. Infrastructure deployed but not running, scaled up on failover. Suitable for important workloads.

C) RPO/RTO: Minutes. Warm Standby strategy. Cost: $$$. Data live, services run at reduced capacity. Scaled up during failover. Suitable for business-critical applications.

D) RPO/RTO: Near real-time. Multi-site Active/Active strategy. Highest cost ($$$$). Data live, live services in multiple regions simultaneously. Suitable for mission-critical, zero-downtime requirements.

E) N/A. Single-region deployment is acceptable, no cross-region DR needed. Rely on multi-zone availability within one region.

X) Other (please describe after [Answer]: tag below)

**Note**: There is no cloud deployment. The only things to recover are the code and the catalog snapshot (both in git) and any local student files. Option X with "local app; recovery = re-clone the repo and rebuild the snapshot" is a valid answer.

[Answer]: X, local app with no deployment. Code and catalog snapshot live in git. Recovery means re-cloning the repo and rebuilding the snampshot from the README. No disaster-recovery infrastructure.

## Clarification Question 15: Change Management Process
How should production changes for this workload be governed? AI-DLC will conform the design to your answer rather than inventing a process.

A) Use our existing organizational change management process. Provide the name or tool (e.g., ServiceNow, Jira Change, internal CAB). AI-DLC will reference it and ensure deployable artifacts fit that process (change records, approval gates).

B) No formal process exists yet. AI-DLC should propose a lightweight change management process (change record + approval + rollback note) for the team to adopt.

C) N/A. This workload is exempt from formal change management (e.g., internal tooling). Document the exemption rationale.

X) Other (describe after [Answer]: tag below)

**Note**: You chose direct commits to main with review only at AI-DLC stage gates (Q40:B). The vision also says "changes go through AI-DLC".

[Answer]: X, No production deployment. The AI-DLC workflow is the change process: approvals happen at stage gates and are logged in audit.md, git history is the change record, and the rollback is git revert.

## Clarification Question 16: Incident Response Process
How are production incidents handled for this workload?

A) Use our existing incident response process. Provide the reference (e.g., PagerDuty runbooks, internal IR/on-call process). AI-DLC will align alerting and runbooks to it.

B) No formal process exists. AI-DLC should propose a lightweight incident response and Correction of Errors (COE) process for adoption.

X) Other (describe after [Answer]: tag below)

[Answer]: X, No production or outside users. Bugs are logged as Github issues and fixed through the AI-DLC workflow. When the app gives wrong advice, the issue records the cause and the case is added to the evaluation set so a regression is caught.
