# Story Generation Plan

**Stage**: INCEPTION - User Stories, Part 1 (Planning)
**Owner role**: Team member 5 (User Stories, Units of Work and QA)
**Inputs**:
- `aidlc-docs/inception/requirements/requirements.md` (approved)
- `docs/vision.md` (MVP Journeys 1 and 2)
- `aidlc-docs/inception/plans/user-stories-assessment.md`

**Outputs**:
- `aidlc-docs/inception/user-stories/personas.md`
- `aidlc-docs/inception/user-stories/stories.md`

---

## How to Answer
1. Fill in every `[Answer]:` tag in the **Questions** section below with a letter, for example `[Answer]: B`.
2. If you pick the last option (Other), describe what you want after the tag.
3. You can add a short note after a letter, for example `[Answer]: A, but keep it under 10 stories`.
4. When every tag is filled in, reply "done" in chat.

The answers become the decision record in `audit.md`. Generation does not start until every answer is in and any ambiguity is resolved.

---

## Story Breakdown Approaches (context for Question 1)

| Approach | How stories are grouped | Benefit | Trade-off |
|---|---|---|---|
| User Journey-Based | By the steps of MVP Journeys 1 and 2 | Mirrors the demo video and journey tests (NFR-7.3) | Shared steps (search, prerequisite checks) are split across journeys; some FRs, like career comparison, fit neither journey |
| Feature-Based | By FR-1 to FR-10 | Clean traceability to requirements; easy mapping to units | Loses the end-to-end student view |
| Persona-Based | By the 4 student personas | Shows the differences between undergraduate and master's rules | Many stories repeat across personas |
| Domain-Based | By area: profile, career and skills, catalog, prerequisites, planning, revision, evaluation | Lines up with likely Construction units | Less natural for acceptance testing by students |
| Epic-Based | Epics = feature areas (F1 to F9 plus evaluation); stories inside each epic, each tagged with the journeys and personas it serves | Combines traceability with the journey view; tags let journey tests pick their stories | Needs consistent tagging |

---

## Questions

## Question 1
How should stories be organized?

A) Epic-Based hybrid: one epic per feature area (F1 to F9 plus Evaluation). Stories inside each epic are tagged with the persona(s) and the MVP journey step(s) they serve. (Recommended: keeps FR traceability for Units Generation and the journey view for the demo.)

B) User Journey-Based: stories follow Journey 1 and Journey 2 step by step, plus a section for stories outside both journeys

C) Feature-Based: one section per FR group (FR-1 to FR-10), no journey tags

D) Persona-Based: one section per persona

E) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 2
How many personas should `personas.md` define?

A) The 4 student personas from requirements section 3 (career-focused undergraduate, exploring undergraduate, professional master's student, major switcher)

B) The 4 student personas plus a **team evaluator / developer** persona, for the evaluation harness (FR-10), catalog refresh (FR-3.1), and developer log stories

C) Option B plus an **academic advisor** persona as a secondary, non-user stakeholder who reads exported plans (FR-8, FR-9)

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 3
How detailed should each persona be?

A) Short: name, program and year, career goal, key planning rules that apply, main frustration (about half a page each)

B) Full: option A plus a sample profile (completed courses as real Cornell codes, credit range, terms left, unavailable terms). The sample profiles can seed the 10 evaluation profiles (FR-10.1).

C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 4
Should the persona sample profiles reuse the two MVP journey students exactly (Information Science junior targeting health data science; MPS Information Science student with an economics background targeting AI product management)?

A) Yes: those two are the career-focused undergraduate and professional master's personas, word for word from the vision

B) Yes for those two, with new students written for the exploring undergraduate and major switcher

C) No: write all personas fresh and keep the journey students only as story examples

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 5
What size should stories be?

A) Small: one observable behavior per story (for example "see 'Not met' with missing courses listed"). Expect about 40 to 60 stories.

B) Medium: one student task per story (for example "check whether I can take a recommended course"). Expect about 20 to 30 stories.

C) Large: about one story per FR group, broken down later in Construction. Expect about 10 to 12 stories.

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 6
What format should acceptance criteria use?

A) Given/When/Then scenarios. These map one-to-one onto example-based tests (NFR-7.3).

B) Checklist of testable statements

C) Given/When/Then for the main and error paths, plus a short checklist for display details (fields shown, labels, links)

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 7
How should non-functional requirements (performance, security, accessibility, logging) appear in the stories?

A) Only as acceptance criteria inside the student stories they affect (for example "the plan appears within 90 seconds" inside the first-plan story)

B) As a separate "Cross-cutting quality" epic of stories (for example "as a student, I can use the whole app by keyboard")

C) Both: cross-cutting stories for app-wide rules (WCAG 2.1 AA, no student data on disk, localhost only, generic errors), with story-specific NFR criteria added where they apply

D) Leave NFRs out of stories; they stay in `requirements.md` and are handled in the NFR stages

E) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 8
Should error and edge behavior be written as separate stories or as extra acceptance criteria on the main stories? Examples: vague career goal (FR-2.2), plan does not fit (FR-6.6), course not offered recently (FR-6.4), unknown course code (FR-1.2), model runtime unavailable (NFR-3.2), LLM retries exhausted (NFR-3.1).

A) Separate stories for edge cases the student acts on (vague goal, plan does not fit, unmet prerequisite with options); extra acceptance criteria for the rest

B) Always extra acceptance criteria on the main story

C) Always separate stories

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 9
Should stories carry a priority label?

A) Yes: Must Have or Should Have, copied from the FR they trace to (F8 export and anything beyond the 8-subject core set are Should Have)

B) Yes: MoSCoW labels decided per story, which may differ from the FR priority

C) No: priority is decided in Workflow Planning and Units Generation

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 10
Should stories include a rough size estimate, to help split work in Units Generation?

A) Yes: T-shirt size (S, M, L) per story

B) Yes: story points (1, 2, 3, 5, 8)

C) No estimates at this stage

D) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 11
How should the LLM-specific behavior be stated in acceptance criteria, since model output varies from run to run? Examples: skill mapping, explanations, "why" answers.

A) Criteria test only checkable properties (for example "6 to 12 skills, each with a one-line reason"; "every explanation names at least one mapped skill"; "every course code exists in the snapshot"), never exact wording

B) Option A plus a reviewer-judged criterion where the metric is human-scored (for example relevance of top-5 recommendations, NFR-2.5)

C) Other (please describe after [Answer]: tag below)

[Answer]: 

## Question 12
Who reviews the generated stories before the team approves them?

A) Team member 5 (stories owner) reviews first, then the whole team approves at the stage gate

B) Team member 5 plus team member 3 (requirements owner), to check traceability to FRs, then the whole team approves

C) The whole team reviews together at the stage gate

D) Other (please describe after [Answer]: tag below)

[Answer]: 

---

## Execution Checklist (Part 2: Generation)
These steps run only after every answer above is filled in, any ambiguity is resolved, and the team approves this plan. Each box is marked [x] as soon as its step is done.

### Personas
- [ ] 1. Draft personas at the agreed count and detail (Questions 2 to 4)
- [ ] 2. For each student persona, list the planning rules that apply: level filter (FR-3.7), horizon (FR-6.1), comparison (FR-2.6), reachability ranking (FR-4.3)
- [ ] 3. Check that every sample course code exists in the 8-subject core set (CQ1:D) and is labeled as a planning example, not a real student's record
- [ ] 4. Write `aidlc-docs/inception/user-stories/personas.md`

### Stories
- [ ] 5. Set up the story structure agreed in Question 1, with story IDs (for example US-2.3) and a "Traces to" line citing FR and NFR IDs
- [ ] 6. Write stories for Student Profile Intake (FR-1)
- [ ] 7. Write stories for Career and Skill Mapping, including comparison (FR-2)
- [ ] 8. Write stories for Catalog Search and Recommendations (FR-3, FR-4)
- [ ] 9. Write stories for Prerequisite Verification (FR-5)
- [ ] 10. Write stories for Semester Planning (FR-6)
- [ ] 11. Write stories for Conversational Revision (FR-7)
- [ ] 12. Write stories for Plan Export (FR-8, Should Have) and Advisory Notice and Sources (FR-9)
- [ ] 13. Write stories for the Evaluation Harness and catalog refresh (FR-10, FR-3.1), if a team persona is included (Question 2)
- [ ] 14. Add NFR coverage as agreed in Question 7
- [ ] 15. Add edge and error behavior as agreed in Question 8
- [ ] 16. Add priority and size labels as agreed in Questions 9 and 10
- [ ] 17. Write `aidlc-docs/inception/user-stories/stories.md`

### Verification
- [ ] 18. Check every story against INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable)
- [ ] 19. Check that every story has acceptance criteria in the agreed format (Questions 6 and 11)
- [ ] 20. Build a persona-to-story map in `stories.md`
- [ ] 21. Build a coverage table: every Must Have FR sub-requirement maps to at least one story; list any gaps
- [ ] 22. Check that MVP Journeys 1 and 2 can be traced end to end through stories, step by step
- [ ] 23. Validate Markdown tables and any diagrams per `common/content-validation.md`
- [ ] 24. Run the extension compliance check (Security, Resiliency, PBT) for this stage and record it
