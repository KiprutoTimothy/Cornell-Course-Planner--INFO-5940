# Requirements Verification Questions

**Project**: Cornell AI Course Planning Agent
**Source**: `docs/vision.md`
**Stage**: INCEPTION - Requirements Analysis (Comprehensive depth)

## How to answer

- Fill in the letter of your choice after each `[Answer]:` tag.
- If no option fits, choose the last option (Other) and describe your answer after the tag.
- This is a team decision. Discuss as a group, or assign sections to people and review together before submitting.
- A **Note** under a question gives context from the vision document or a trade-off. It is not a decision. The team decides.
- Tell the AI when every question is answered ("done"). The AI will check the answers for contradictions and may send a short follow-up file.

## Sections

1. Technical Environment (the vision refers to a technical environment document that is not in the repository)
2. Catalog Data and Coverage
3. Student Profile and Planning Horizon
4. Career and Skill Mapping
5. Recommendations and Ranking
6. Prerequisite Verification
7. Semester Planning
8. Conversation, Explanations, and Export
9. Non-Functional Requirements
10. Evaluation and Team Process
11. Extension Opt-Ins

---

# Section 1: Technical Environment

## Question 1
Which LLM provider and model will the MVP use as its primary model?

**Note**: The vision says provider and model are "defined in the technical environment document", but that document is not in the repository. The model must support reliable tool calling and fit a student budget.

A) A hosted commercial API (for example Anthropic Claude or OpenAI) paid with a course- or team-provided key

B) A free local model run through Ollama or LM Studio (for example a Llama, Qwen, or Mistral model)

C) A hosted API as primary, with a local model as the documented fallback

D) A course-provided LLM endpoint or proxy (supplied by INFO 5940 staff)

E) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2
What backend language and agent approach will the team use?

A) Python with a lightweight agent loop written by the team on top of the provider SDK (direct tool-calling)

B) Python with an agent framework (for example LangChain or LangGraph)

C) Python with another agent framework (for example LlamaIndex, OpenAI Agents SDK, or Pydantic AI)

D) TypeScript or JavaScript (Node.js) with a provider SDK or framework

E) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3
What should the local web interface be built with?

**Note**: The product runs locally on team laptops. Simpler frameworks reduce setup risk on 5 different machines.

A) Streamlit (Python only; chat and form widgets built in)

B) Gradio or Chainlit (Python only; chat-focused)

C) Separate frontend (for example React or Next.js) with a Python or Node API backend

D) Server-rendered web app (for example FastAPI or Flask with HTML templates)

E) Other (please describe after [Answer]: tag below)

[Answer]: D

## Question 4
How should catalog search (F3) find courses that match a skill?

**Note**: The vision notes that course text uses different words for the same skill ("statistical learning", "predictive modeling", "machine learning"). Keyword search alone may miss these.

A) Keyword search only (for example SQLite full-text search or BM25)

B) Semantic search with embeddings stored in a local vector store (for example Chroma, FAISS, or LanceDB)

C) Hybrid: keyword plus embeddings, with results merged

D) Load the course subset into the LLM context directly and let the model choose (no search index)

E) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 5
If semantic search is used, where do embeddings come from?

A) A local embedding model (for example sentence-transformers), at no cost

B) The hosted LLM provider's embedding API

C) Not applicable (no embeddings, based on Question 4)

D) Other (please describe after [Answer]: tag below)

[Answer]: A, will confirm if there's an embedding API later 

## Question 6
What minimum laptop setup must the app support, given the 5 team laptops?

A) Windows, macOS, and Linux, without a GPU

B) Windows and macOS, without a GPU

C) Any OS, run through Docker for a consistent setup

D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

# Section 2: Catalog Data and Coverage

## Question 7
Which subjects does the MVP load? (Vision open question: Subject coverage)

A) CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON (8 subjects)

B) All subjects in the Computing and Information Science and Engineering colleges

C) All 170+ Cornell subjects

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 8
How should the app get catalog data at runtime?

**Note**: The vision's risk table says "Store a dated snapshot of the subject subset; app reads from the snapshot."

A) A committed snapshot only: a script fetches from the Class Roster API once, and the app reads only the snapshot

B) Snapshot by default, plus a refresh script any team member can run to update it

C) Live API calls at runtime, with the snapshot as a fallback

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 9
Which roster terms should the snapshot include? Offering history drives the "typically offered" logic.

A) Only the latest published rosters (FA26 and SP27)

B) The last 2 years of rosters (roughly FA24 through SP27), to infer offering patterns

C) The last 4 years of rosters, for a fuller offering history

D) Other (please describe after [Answer]: tag below)

[Answer]: C, but wary of how the previous data will be fetched and stored and if we have that capacity at the moment

---

# Section 3: Student Profile and Planning Horizon

## Question 10
Which semesters can a plan cover? (Vision open question: Semesters covered)

A) Spring 2027 onward only. Fall 2026 is treated as in progress.

B) Allow the remainder of Fall 2026 as well as later terms

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 11
What is the maximum number of future semesters in a plan?

A) 4 for undergraduates, 3 for master's students

B) A single limit of 4 semesters for everyone

C) A single limit of 3 semesters for everyone

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12
Do in-progress Fall 2026 courses count as completed for Spring 2027 planning? (Vision open question: Corequisites and in-progress courses)

A) Yes. In-progress courses count as completed for any later term.

B) They count, but each verdict that depends on one is flagged "assuming you pass [course]"

C) No. Only finished courses count.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 13
Can a student enter fixed courses they already plan to take, so the planner fills only the remaining credits? (Vision open question: Required courses)

A) Yes. Fixed courses are pinned to a term the student picks and count toward that term's credits.

B) Yes, but the student only lists them and the planner chooses the term

C) No. The MVP plans career courses only, and the student adjusts the credit range by hand.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 14
Is session-only memory enough? (Vision open question: Session memory)

A) Session only. Nothing is saved when the app closes except the Markdown export.

B) Also allow save and load of the profile (and the current plan) to a local JSON file

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 4: Career and Skill Mapping

## Question 15
How does the student enter a career goal? (Vision open question: Career goal input)

A) Free text only, for any career

B) Free text, plus a pick list of team-reviewed careers as shortcuts. Any career is still allowed.

C) Pick list only, limited to careers the team has reviewed

D) Other (please describe after [Answer]: tag below)

[Answer]: B plus guided questions for any unclear career goals 

## Question 16
Where do skill mappings come from? (Vision open question: Source of skill mappings)

A) LLM-generated every time

B) Team-curated mappings for supported careers, with LLM generation for any other career

C) LLM-generated, then cached after team review. Uncached careers fall back to live generation.

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 17
The vision lists "Comparison of multiple careers side by side" (Exploring undergraduate persona, full-vision Journey 2) but does not list it in the MVP feature table. Is it in the MVP?

A) No. One career goal per plan in the MVP; comparison is deferred.

B) Yes, but light: the student can enter 2 to 3 careers and see their skill lists side by side, with no full plans

C) Yes, full: a separate plan per career, compared side by side

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 5: Recommendations and Ranking

## Question 18
What drives ranking beyond skill coverage? (Vision open question: Ranking factors)

A) Skill coverage only (number and importance of mapped skills covered)

B) Skill coverage plus prerequisite reachability (courses the student can take sooner rank higher)

C) A weighted score of skill coverage, reachability, student interests and avoided topics, and course level fit

D) The LLM ranks the search results and explains its order, with no fixed scoring formula

E) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 19
How many career-related courses should a plan contain by default? (Vision open question: Number of recommendations)

A) A fixed default (for example 6 to 8), which the student can change

B) A target number of career courses per semester, set by the student (for example 1 to 3)

C) As many as fit within the credit range after fixed courses are placed

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 20
Which course levels should each student type see? (Vision open question: Undergraduate versus graduate courses)

A) Filter by career level by default (undergraduates see 1000 to 4000 level; master's students see 3000 level and above), but include lower-level courses when they are missing prerequisites

B) Show all levels to everyone. Ranking prefers the student's level.

C) Undergraduates see 1000 to 5000 level; master's students see all levels

D) Other (please describe after [Answer]: tag below)

[Answer]: A, for the master's students, see 5000 level+ classes

## Question 21
What happens to a course with no "when offered" data and no recent offering history? (Vision open question: Offering-term uncertainty)

A) Exclude it from recommendations

B) Include it with a visible warning, and never place it in a specific term without a flag

C) Include it only when the student asks for it by name

D) Other (please describe after [Answer]: tag below)

[Answer]: D, if getting the data from the public API class roster, it's usually organized by semester so this shouldn't be an issue

---

# Section 6: Prerequisite Verification

## Question 22
When is prerequisite text interpreted? (Vision open question: Prerequisite interpretation timing)

A) Once, at data load: the LLM parses each course's text into a structured rule that is stored, reviewed by the team, and evaluated by deterministic code

B) Live, every time a course is checked

C) Pre-parsed and stored, with live LLM interpretation only for cases marked ambiguous at load

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 23
How are ambiguous prerequisite rules classified? (Vision open question: Ambiguous prerequisite rules)

A) Always "Needs review" for "or equivalent", "permission of instructor", "recommended", and background-specific rules

B) "Needs review" by default, but "Met" when the student has completed one of the named courses and only an "or equivalent" clause is unresolved

C) "Recommended" rules do not block. They are shown as advisory ("Met" with a note), and all other ambiguous cases are "Needs review".

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 24
How should ambiguous "and/or" grouping (for example "CS 1110 or CS 1112 and INFO 1300") be handled?

A) Mark it "Needs review" and show both readings

B) Use a documented default reading ("and" binds tighter than "or", or comma-separated groups are ANDed), and flag it as an assumption

C) Treat it as "Met" only if the student satisfies both possible readings; otherwise "Needs review"

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 25
Is forbidden-overlap detection in the MVP?

**Note**: F4 says recommendations exclude "forbidden overlaps with completed courses", but Feature Area 4 also lists overlap detection among *planned* courses.

A) Overlaps with completed courses only (as F4 says)

B) Overlaps with completed courses and among courses inside the same plan

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 7: Semester Planning

## Question 26
Which part of the system builds the semester plan?

**Note**: The vision says deterministic code is used "where it is more reliable (credit totals, ordering checks)".

A) Deterministic planner code places courses. The LLM only chooses which courses and explains them.

B) The LLM proposes the placement, and a deterministic validator checks it. On failure the LLM gets the errors and retries.

C) The LLM proposes, the validator checks, and a deterministic fixer repairs violations if the LLM's retries fail

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 27
If the planner cannot fit every chosen course (credits, prerequisites, or offering terms), what should it do?

A) Drop the lowest-ranked courses and list them as "did not fit", with the reason

B) Ask the student which constraint to relax (credits, number of courses, or adding a semester)

C) Use A by default and offer B as a follow-up in the chat

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 8: Conversation, Explanations, and Export

## Question 28
How long should each course explanation be? (Vision open question: Explanation length and tone)

A) One sentence naming the mapped skills covered

B) One to two sentences (as in F4), plus the prerequisite status line

C) A short paragraph per course

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 29
Should the app show which tools the agent called? (Vision open question: Explanation length and tone)

A) Yes, always visible as a step log next to the chat

B) Yes, in a collapsed "How I built this plan" panel

C) No for students; tool calls go only to a developer log file

D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 30
What should F7 (conversational revision) support in the MVP?

A) Plan edits only (swap, move, remove, or add courses; change credits; block a term) plus "why" questions about the plan

B) A plus changes to the profile and skill list mid-conversation, with a full re-plan

C) A plus B plus general advising questions not tied to the plan (for example "is CS 4780 hard?"), answered only from catalog data

D) Other (please describe after [Answer]: tag below)

[Answer]: B, but change the profile only when mentioned specifically, no full re-plan

## Question 31
Is F8 (Markdown export) required or optional for the MVP?

**Note**: The vision lists F8 as "Should Have" and "if time allows", but MVP Journey 1 ends with a Markdown export.

A) Required. Journey 1 depends on it, and it is small.

B) Optional. Build it only if time allows; the demo can end without it.

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 9: Non-Functional Requirements

## Question 32
How should "Time to a first plan: under 90 seconds" be applied?

A) A hard requirement on all 10 evaluation profiles, on every team laptop

B) A target on the median evaluation profile, on the team's fastest laptop

C) Required with the primary model only; the local fallback model may be slower

D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 33
How should the app behave when the LLM call fails (network error, rate limit, invalid tool call, or invalid structured output)?

A) Retry a fixed number of times, then show a clear error and keep the student's inputs

B) A, plus switch automatically to the fallback model

C) A, plus a degraded mode that still shows catalog search results and prerequisite verdicts without the LLM

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 34
How should the API key be managed? (Definition of Done: "No API keys or secrets in the repository")

A) A `.env` file that each member creates from a committed `.env.example` (`.gitignore` already excludes `.env`)

B) OS environment variables only, documented in the README

C) Not applicable (local model only, no key)

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 35
What accessibility and usability level should the UI meet?

A) Basic: readable layout, keyboard-usable controls, and the framework's default accessibility

B) Target WCAG 2.1 AA for custom UI elements (contrast, labels, focus order)

C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 36
What should be logged during a session?

**Note**: Profiles hold course history, which is personal but not from official student records.

A) Developer logs of tool calls and LLM timing only, with no student profile content, stored locally

B) Full session transcripts stored locally, for debugging and evaluation

C) Full transcripts only when a debug flag is on; minimal logging by default

D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 10: Evaluation and Team Process

## Question 37
Who builds the evaluation data, and how is it reviewed? (Vision open question: Evaluation data)

A) Split evenly: each of the 5 members writes 2 profiles and 8 prerequisite cases; each label is checked by a second member

B) Split evenly with a single labeler per item (no second review)

C) One or two members own all evaluation data; the full team reviews it once

D) Other (please describe after [Answer]: tag below)

[Answer]: D, assign an agent to generate the evaluation data and reviewed by one team member

## Question 38
Should the evaluation checks (fabricated courses, plan validity, explanation coverage, prerequisite accuracy) run as an automated script in the repository?

A) Yes. One command runs all automated metrics and writes a results report.

B) Yes, as part of the unit/integration test suite (pytest or similar)

C) No. The automated checks run inside the app; the team records results by hand.

D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 39
How will the 5 members split Construction work? This guides how the system is divided into units of work later.

A) By feature or layer: data ingestion; prerequisite engine; agent and tools; planner and validator; UI and evaluation

B) In pairs across features, with one member as integrator and reviewer

C) Decide during Units Generation based on the design

D) Other (please describe after [Answer]: tag below)

[Answer]: D, already split the work amongst us. For our Cornell AI Course Planning Agent, we can split the work among the five team members as follows:
1. GitHub Setup and AI-DLC Workflow
    •    Create the GitHub repository and add all team members.


    •    Install and configure AI-DLC v1.0.1.


    •    Lead the shared AI-DLC workflow.


    •    Ensure audit and state files are properly maintained.


    •    Handle the final submission.


2. README and Product Proposal
    •    Describe the problem students face when selecting courses based on their career goals.


    •    Explain why an LLM-powered agent is appropriate for solving this problem.


    •    Describe the planned LLM integration and project scope.


    •    Document team members, NetIDs and the collaboration plan.


3. Vision Document and Requirements
    •    Define the target users and product objectives.
      •    Identify core MVP features, including personalized course recommendations, career-based skill mapping, prerequisite verification and semester planning.


    •    Define out-of-scope features.


    •    Identify open questions and lead requirements analysis.


4. Technical Environment and Application Design
    •    Identify Cornell course data sources.


    •    Determine the technology stack, LLM provider and model.


    •    Design the course retrieval and recommendation mechanisms.


    •    Define the agent's tools, security requirements and testing framework.


     •    Prepare the required technical environment document and code examples.


5. User Stories, Units of Work and Quality Assurance
    •    Develop student personas with different academic backgrounds and career goals.


    •    Lead user story development and define acceptance criteria.


    •    Lead the breakdown of the product into manageable development units.


    •    Review dependencies between units.


    •    Verify that all deliverables meet the assignment rubric.


   

## Question 40
Which version control and review workflow will the team use for AI-DLC artifacts and code?

A) Feature branches with pull requests on GitHub; at least one reviewer approves each PR

B) Commit directly to main, with team review at AI-DLC stage gates only

C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

# Section 11: Extension Opt-Ins

## Question 41: Security Extensions
Should security extension rules be enforced for this project?

**Note**: The project is a local, single-user MVP with no accounts or hosting, but it handles an API key and student course history.

A) Yes. Enforce all SECURITY rules as blocking constraints (recommended for production-grade applications).

B) No. Skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects).

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 42: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices** for building resilient systems, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability. It covers 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early. It is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft of your resiliency posture** to build on and validate, not a finished, production-certified result.

A) Yes. Apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live).

B) No. Skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability).

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 43: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

**Note**: The prerequisite rule evaluator and the plan validator are deterministic logic with clear invariants (for example, every course is placed after its prerequisites, and every term stays within its credit range).

A) Yes. Enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components).

B) Partial. Enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity).

C) No. Skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic).

X) Other (please describe after [Answer]: tag below)

[Answer]: A
