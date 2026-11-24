# Vision: Cornell AI Course Planning Agent

## Executive Summary

The Cornell AI Course Planning Agent is a local web application that helps Cornell students choose courses based on the career they want. A student states a career goal in plain language, enters the courses they have completed, and the agent maps the goal to concrete skills, searches the Cornell course catalog for courses that teach those skills, checks each course's prerequisites against the student's history, and lays the courses out across the student's remaining semesters. Every recommendation comes with a short explanation and links back to the official course listing. The goal is to cut the hours students spend reading Class Roster pages and decoding prerequisite text, and to give them a plan they can take to their advisor.

## Business Context

### Problem Statement

Cornell offers thousands of courses across more than 170 subjects each semester. Students who know what job they want ("data scientist at a health company", "UX researcher", "ML engineer") have no tool that connects that goal to specific courses. Class Roster answers "what is offered this semester", requirement trackers answer "what counts toward my major", and course review sites answer "is this course good". None answers "what should I take, and in what order, to be ready for this job". Students piece this together from upperclassmen, LinkedIn profiles, and advisor meetings that are short and booked weeks out.

Prerequisites make this harder. They are published as free text, written differently by each department. Real examples from the Spring 2026 INFO roster:

- "CS 1110 or CS 1112 and INFO 1300." (ambiguous grouping of "or" and "and")
- "INFO 2950 or equivalent, AEM 2100, CS 1110." (open-ended "or equivalent")
- "Recommended prerequisite for programmers: CS 2110, or permission of instructor. Recommended prerequisite for designers: INFO 2450 ..." (different rules per student background)

A student who misreads one of these finds out at enrollment or in week two of the course. A student who misses a prerequisite chain (for example, needing CS 2110 before CS 3110 before a 4000-level systems course) can lose a full semester, which hurts most for one-year professional master's students.

### Why an LLM Agent

The core tasks are language tasks with a reasoning loop:

- Turning a vague career goal into a skill list requires world knowledge about jobs that no Cornell dataset contains.
- Matching skills to courses requires reading course descriptions and learning outcomes, which use different words for the same skill ("statistical learning", "predictive modeling", "machine learning").
- Interpreting prerequisite text requires parsing inconsistent natural language and judging edge cases such as "or equivalent".
- Building a plan requires several dependent steps: search, check eligibility, check when a course is offered, re-search when a course is blocked, then sequence. The LLM decides which step to run next and uses tool results before it answers.

A rules engine could handle the final sequencing, and the product will use deterministic code where it is more reliable (credit totals, ordering checks). The LLM handles the parts that need language understanding and judgment, and it must ground every course it names in catalog data returned by a tool.

### Target Users and Stakeholders

| User Type | Description | Primary Need |
|-----------|-------------|--------------|
| Career-focused undergraduate | Sophomore or junior in CS, Information Science, ORIE, Statistics, or a related major, with a target role in mind | Which electives build the skills for that role, and when to take them |
| Exploring undergraduate | First-year or sophomore who has not settled on a career or major | Compare two or three career paths and see what courses each one implies |
| Professional master's student | MPS or MEng student with two or three semesters and often a non-Cornell undergraduate background | Highest-value courses for the target role, and fast detection of prerequisite gaps from their prior degree |
| Major switcher | Student moving into a technical area from an unrelated major (for example, Biology to data science) | The shortest valid prerequisite chain into the courses they need |

Stakeholders who are not direct users in the MVP:

| Stakeholder | Interest |
|-------------|----------|
| Academic advisors | Students arrive at meetings with a draft plan and specific questions |
| INFO 5940 course staff | Evaluate the product as an agentic LLM application |
| Project team (5 members) | Deliver a working, demoable product by December 16, 2026 |

### Business Constraints

- The product runs locally on team laptops. No hosting, no user accounts.
- Course data comes only from public Cornell sources (Class Roster API and Courses of Study). No access to student records, Cornell SSO, or official degree audits.
- The product is advisory. It never claims a plan satisfies graduation requirements and always tells the student to confirm with their advisor.
- No paid data sources. LLM cost must fit a student budget or run on a free local model (provider and model are defined in the technical environment document).
- Construction must finish by the final project deadline, December 16, 2026.

### Success Metrics

| Metric | Current State | Target State | Measurement Method |
|--------|---------------|--------------|--------------------|
| Fabricated courses | Not measured | 0 recommended courses that are missing from the loaded catalog | Automated check on every plan in the evaluation set |
| Prerequisite verdict accuracy | Students read text by hand | At least 90% agreement with team-labeled verdicts | 40 hand-labeled (course, completed-courses) pairs |
| Recommendation relevance | No tool | At least 80% of top-5 recommendations judged relevant to the stated career | Team review of 10 evaluation profiles, 2 reviewers each |
| Plan validity | No tool | 100% of generated plans place every course after its prerequisites and inside the student's credit range | Automated plan validator |
| Explanation coverage | No tool | 100% of recommendations name at least one mapped skill and show prerequisite status | Automated check |
| Time to a first plan | Hours of manual research | Under 90 seconds on a team laptop | Timed runs on the evaluation profiles |
| Perceived usefulness | Not measured | Average at least 4 out of 5 | 5 or more student testers outside the team rate the plan |

## Full Scope Vision

### Product Vision Statement

Every Cornell student can describe where they want to go and get a semester-by-semester path there, built from real Cornell courses, with the reasoning shown, that they can refine in conversation and bring to their advisor.

### Feature Areas

#### Feature Area 1: Student Profile

- **Description**: What the agent knows about the student.
- **Key Capabilities**:
  - Program, college, year, and expected graduation term
  - Completed and in-progress courses
  - AP, transfer, and placement credit
  - Interests, disliked topics, and workload preferences
  - Saved profiles across sessions
- **User Value**: The student enters their history once and every answer is personalized.

#### Feature Area 2: Career and Skill Mapping

- **Description**: Translate a career goal into the skills and knowledge that role needs.
- **Key Capabilities**:
  - Free-text career goal to a ranked skill list
  - Comparison of multiple careers side by side
  - Skill gap view: skills already covered by completed courses versus skills still missing
  - Optional grounding in external taxonomies or job postings
- **User Value**: The student sees what the job requires in terms courses can teach.

#### Feature Area 3: Course Discovery and Recommendation

- **Description**: Find and rank courses that build the needed skills.
- **Key Capabilities**:
  - Search across all Cornell subjects by skill, topic, or keyword
  - Ranked recommendations with per-course explanations
  - Filters for level, credits, distribution category, and semester
  - Course quality signals such as reviews and workload
- **User Value**: The student finds relevant courses outside their home department that they would not have searched for.

#### Feature Area 4: Prerequisite and Eligibility Checking

- **Description**: Decide whether the student can take a course.
- **Key Capabilities**:
  - Interpretation of free-text prerequisites and corequisites
  - Full prerequisite chain for any target course
  - Forbidden-overlap detection (courses that cannot both count)
  - Handling of permission-of-instructor and "or equivalent" cases
- **User Value**: No surprises at enrollment.

#### Feature Area 5: Semester Planning

- **Description**: Sequence courses across remaining semesters.
- **Key Capabilities**:
  - Multi-semester plan that respects prerequisites, credit limits, and offering patterns
  - Conversational edits ("move CS 4780 to fall", "I want a lighter spring")
  - Section-level schedules with time-conflict checks
  - Degree requirement tracking for each major
- **User Value**: A concrete plan the student can act on and adjust.

#### Feature Area 6: Advisor Collaboration

- **Description**: Share plans with advisors.
- **Key Capabilities**:
  - Export to PDF or a shareable link
  - Advisor comments on a plan
- **User Value**: Advising meetings start from a draft instead of a blank page.

### Integration Points

- Cornell Class Roster API (`classes.cornell.edu/api/2.0`): course titles, descriptions, learning outcomes, credits, prerequisite text, forbidden overlaps, when offered, last terms offered, distribution categories
- Cornell Courses of Study (`courses.cornell.edu`): catalog descriptions and major requirements (full vision)
- LLM provider: as defined in the technical environment document
- Course review data and job posting data (full vision only)

### User Journeys (Full Vision)

#### Journey 1: Junior targeting machine learning engineering

1. Signs in with Cornell credentials; the profile loads from their degree audit.
2. Types "I want to be an ML engineer at a robotics company".
3. Sees mapped skills, which ones their completed courses already cover, and the gaps.
4. Gets a two-semester plan with sections chosen to avoid time conflicts and requirement progress shown.
5. Shares the plan with their advisor, who leaves comments.

**Outcome**: An advisor-reviewed plan tied to a career goal.

#### Journey 2: Exploring first-year

1. Enters "I like biology and coding but don't know what job I want".
2. The agent proposes three career paths and the courses each one implies.
3. The student compares the paths side by side and saves two of them.

**Outcome**: A grounded comparison of options before declaring a major.

### Scalability and Growth

- Expand from a subset of subjects to all Cornell subjects
- Add other universities with public course APIs
- Add graduate research tracks and PhD preparation paths

### Long-Term Roadmap

| Phase | Focus | Timeframe |
|-------|-------|-----------|
| MVP | Career mapping, grounded recommendations, prerequisite checks, multi-semester plan, chat revision, for a defined subject subset | December 16, 2026 |
| Phase 2 | All subjects, transcript upload, degree requirement tracking for a few majors, section schedules | After the course |
| Phase 3 | Advisor collaboration, reviews and workload signals, saved accounts | TBD |

## MVP Scope

### MVP Objective

Show that an agent can take a free-text career goal and a list of completed courses and return a valid, explained, multi-semester plan built only from real Cornell courses, with prerequisites checked.

### MVP Success Criteria

- [ ] For each of 10 evaluation profiles, the agent returns a plan with zero fabricated courses
- [ ] Prerequisite verdicts match team labels on at least 90% of 40 labeled cases
- [ ] Every plan passes the automated validator (prerequisite order and credit range)
- [ ] A student can revise a plan through chat at least three times in one session without losing earlier choices
- [ ] The full flow runs on every team member's laptop from the README instructions

### Features In Scope (MVP)

| ID | Feature | Description | Priority | Rationale for Inclusion |
|----|---------|-------------|----------|-------------------------|
| F1 | Student profile intake | Form and chat input for program, year, semesters remaining, credit range per semester (default 12 to 18), completed and in-progress courses (typed or pasted as a list of course codes), fixed courses the student already plans to take, interests, topics to avoid, and career courses per semester (default 2). Stored for the session only. | Must Have | Every other feature depends on knowing the student |
| F2 | Career-based skill mapping | Converts a free-text career goal into 6 to 12 named skills, each with a one-line reason. The student can add, remove, or reword skills before recommendations run. Marks skills already covered by completed courses. | Must Have | This is the link between careers and courses that no existing tool provides |
| F3 | Catalog search | Searches the loaded Cornell course data by skill, topic, or keyword, using course titles, descriptions, and learning outcomes. Returns only courses that exist in the loaded data. | Must Have | Grounds every recommendation in real courses |
| F4 | Personalized course recommendations | Ranked list of courses for the mapped skills. Each entry shows course code, title, credits, skills it covers, a 1 to 2 sentence reason, prerequisite status, and a link to the Class Roster page. Excludes completed courses and forbidden overlaps with completed courses. | Must Have | The main output students came for |
| F5 | Prerequisite verification | Interprets the free-text prerequisite and corequisite text for a course, compares it with the student's completed and planned courses, and returns one of: Met, Not met (with the missing courses), or Needs review (permission of instructor, "or equivalent", recommended-only). Always shows the original text. | Must Have | Prevents the most costly planning mistake |
| F6 | Semester planning | Places the student's fixed courses first, then fills remaining credits with recommended courses so that each course follows its prerequisites, each semester stays in the credit range, and courses are placed in terms they are typically offered (from "when offered" and "last terms offered" data). Flags any course whose offering term is uncertain. | Must Have | Turns a list into an actionable plan |
| F7 | Conversational revision | The student can ask follow-up questions and request changes ("swap INFO 3300 for something with less design", "I'm studying abroad in spring"). The agent updates the plan and re-checks prerequisites and credits. | Must Have | Plans are never right on the first pass; this is where the agent loop earns its place |
| F8 | Plan export | Download the current plan, skills, and explanations as a Markdown file. | Should Have | Students need something to bring to their advisor |
| F9 | Advisory notice and sources | Every plan states that it is advisory and must be confirmed with an advisor, and links each course to its official listing. | Must Have | Protects students from over-trusting the output |

### Features Explicitly Out of Scope (MVP)

| Feature | Reason for Deferral | Target Phase |
|---------|---------------------|--------------|
| Official degree audit or graduation requirement checking | Requirements differ by college and major and are not in a clean public dataset; wrong answers here carry real cost | Phase 2 (a few majors) |
| Course enrollment, registration, or waitlist actions in Student Center | Requires Cornell authentication and is outside a planning tool's role | Not planned |
| Cornell SSO login and pulling records from student systems | Requires institutional access and FERPA handling | Phase 3 |
| Transcript PDF upload and parsing | Manual course entry is enough to validate the core idea | Phase 2 |
| Section-level scheduling and meeting-time conflict checks | Meeting times are only known for the current roster, and plans span future semesters | Phase 2 |
| Seat availability and real-time enrollment data | Changes hourly; not needed for multi-semester planning | Not planned |
| Course reviews, ratings, grade distributions, workload estimates | Separate data sources with licensing and accuracy questions | Phase 3 |
| Live job-posting or salary data | Scraping and API access add cost and terms-of-service risk | Phase 3 |
| Advisor accounts, sharing links, and comments | Requires hosting and multi-user accounts | Phase 3 |
| Saved profiles across sessions and user accounts | MVP runs locally for a single user | Phase 2 |
| All 170+ Cornell subjects | Larger data load and evaluation effort; a subject subset proves the idea | Phase 2 |
| AP, transfer, and placement credit rules | Rules vary by department; students can list the Cornell equivalent course as completed | Phase 2 |
| Mobile app and hosted deployment | Assignment requires local execution only | Not planned |
| Other universities | Out of the project's purpose | Not planned |

### MVP User Journeys

#### Journey 1: Junior targeting data science in health care

1. Opens the local app and enters: Information Science major, junior, 3 semesters left, 14 to 17 credits, completed CS 1110, CS 2110, INFO 1300, INFO 2950, MATH 1920, STSCI 2150.
2. Types "data scientist at a hospital or health tech company".
3. The agent shows 9 skills (statistical modeling, machine learning, causal inference, SQL and data engineering, data visualization, health data privacy, and others), with 3 already marked covered.
4. The student removes "deep learning" and adds "survey methods".
5. The agent returns ranked recommendations, each with skills covered, reason, prerequisite status, and link.
6. The agent places 7 courses across Spring 2027, Fall 2027, and Spring 2028, and flags one course whose spring offering is uncertain.
7. The student types "I'm abroad in Fall 2027, no Cornell courses that term". The agent moves courses, re-checks prerequisites, and explains which course had to shift to Spring 2028.
8. The student exports the plan as Markdown.

**Outcome**: A three-semester plan with reasons and prerequisite checks, ready for an advisor meeting.
**Limitation vs Full Vision**: Courses are entered by hand, no requirement tracking, no section times, no sharing.

#### Journey 2: Professional master's student from a non-CS background

1. Enters: MPS in Information Science, 2 semesters, 15 to 18 credits, undergraduate in economics, completed (equivalent) intro statistics and one Python course, listed as STSCI 2150 and CS 1110.
2. Types "product manager for AI products".
3. The agent maps skills, recommends courses, and finds that a target machine learning course needs CS 2110 and linear algebra.
4. The agent marks the course "Not met", names the missing courses, and offers two options: a reachable alternative course this spring, or taking the prerequisite now and the target course next semester.
5. The student picks the alternative and asks why each course matters for product management. The agent answers per course, citing the mapped skills.

**Outcome**: The student avoids enrolling in a course they cannot take and gets a two-semester plan that fits their timeline.
**Limitation vs Full Vision**: No transcript import; prior coursework is mapped to Cornell equivalents by the student.

### MVP Constraints and Assumptions

- **Assumption**: The Class Roster API remains public and returns prerequisite text, learning outcomes, and offering history for the chosen subjects. **Risk if wrong**: Fall back to a stored snapshot taken before Construction.
- **Assumption**: The chosen LLM can follow tool-calling instructions and produce structured output reliably enough for plan generation. **Risk if wrong**: Move more logic into deterministic code and use the LLM only for mapping and explanations.
- **Assumption**: Past offering patterns ("when offered", last terms offered) predict future offerings well enough for planning. **Risk if wrong**: Plans may place a course in a term it is not taught; mitigated by flagging uncertain terms.
- **Assumption**: Students can list completed courses as Cornell course codes. **Risk if wrong**: Master's students with outside degrees need help mapping; handled in conversation for the MVP.
- **Accepted Limitation**: The MVP covers 8 subjects (CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, ECON) and up to 4 future semesters starting Spring 2027.
- **Accepted Limitation**: Recommendations are advisory and do not check major requirements.
- **Accepted Limitation**: One student per session, no persistence after the app is closed (except the exported file).

### MVP Definition of Done

- [ ] F1 to F7 and F9 implemented and tested; F8 implemented if time allows
- [ ] Evaluation set of 10 student profiles and 40 labeled prerequisite cases committed to the repository
- [ ] All MVP success criteria measured and results recorded
- [ ] App runs locally on every team member's laptop from the README
- [ ] No API keys or secrets in the repository
- [ ] Demo video shows both MVP user journeys end to end
- [ ] Whole team has reviewed and approved the final plan output on the demo profiles

## Risks and Dependencies

### Key Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| LLM names a course that does not exist or misstates its details | High | High | Agent may only cite courses returned by catalog tools; every course code in output is validated against the loaded data before display |
| Prerequisite text is misread (for example "A or B and C") | Medium | High | Show original text with every verdict; use "Needs review" when grouping is ambiguous; measure against labeled cases |
| Course not offered in the term the plan assumes | Medium | Medium | Use offering history; flag uncertain placements; advisory notice |
| Students treat the plan as official | Medium | High | Advisory notice on every plan; no claims about graduation requirements |
| Career-to-skill mapping is generic or biased toward common roles | Medium | Medium | Student can edit skills before recommendations; team reviews mappings for the evaluation careers |
| Local model too slow or weak for multi-step tool use | Medium | Medium | Measure early in Construction; tech environment document names a fallback model |
| Class Roster API changes, rate-limits, or goes down | Low | High | Store a dated snapshot of the subject subset; app reads from the snapshot |
| Scope grows past what 5 people can build by December 16 | Medium | High | Out-of-scope table is binding for the MVP; changes go through AI-DLC |

### External Dependencies

- Cornell Class Roster API: Cornell University Registrar, public, no key required, available as of November 2026 (rosters through SP27 published)
- Cornell Courses of Study: Cornell University, public, used for reference only in the MVP
- LLM provider and model: see the technical environment document

### Open Questions and Proposed Answers

These are the known uncertainties for Requirements Analysis. Each has the team's proposed answer for the prototype. Requirements Analysis should confirm or challenge each one; any answer can change during Construction if testing shows a better option.

1. **Subject coverage.** Which subjects does the MVP load?
   - **Proposed answer**: CS, INFO, ORIE, STSCI, MATH, ECE, BTRY, and ECON. These cover the six supported careers (question 7) and keep the dataset small enough to load, search, and evaluate in the time available. The subject list is a configuration value so more subjects can be added without code changes.
2. **Semesters covered.** Which terms can a plan include, and how many?
   - **Proposed answer**: Plans start at Spring 2027 (the next roster) and cover up to 4 future semesters for every student. Master's students set a smaller number of remaining semesters in their profile. Fall 2026 is treated as in progress (question 6).
3. **Undergraduate versus graduate courses.** Which course levels does each student see?
   - **Proposed answer**: Undergraduates see 1000 to 5000-level courses, with 5000-level courses marked "graduate level, check enrollment rules". Master's students see 3000-level and above. Any lower-level course can still appear when it is a prerequisite that unlocks a recommended course.
4. **Prerequisite interpretation timing.** Interpret prerequisite text once at data load, or every time a course is checked?
   - **Proposed answer**: Once, at data load. The LLM converts each course's prerequisite text into a structured rule stored next to the original text. Checking a student against a stored rule is deterministic code. This makes rules reviewable by the team, makes checks fast and repeatable, and makes them testable against the labeled cases.
5. **Ambiguous prerequisite rules.** How are unclear rules classified?
   - **Proposed answer**:
     - "Permission of instructor", alone or as an alternative the student needs: Needs review.
     - "Or equivalent": Met if the student completed the named course, otherwise Needs review.
     - "Recommended" prerequisites: do not block. Verdict is based on required prerequisites only, with the recommendation shown as a note.
     - Background-specific rules (for example programmers versus designers): Needs review, with each path shown.
     - Text where the grouping of "and" and "or" is unclear: Needs review.
6. **Corequisites and in-progress courses.** Do in-progress courses count?
   - **Proposed answer**: Yes. In-progress Fall 2026 courses count as completed for any later term, and the plan notes "assumes you pass". A corequisite is met if it is completed, in progress, or placed in the same semester of the plan.
7. **Career goal input.** Free text, a pick list, or both?
   - **Proposed answer**: Both. The student can type any goal, and the form suggests six supported careers: software engineer, data scientist, ML engineer, product manager, UX designer or researcher, and quantitative analyst. Goals outside those six are accepted and marked "skill mapping not reviewed by the team".
8. **Source of skill mappings.** Where do career-to-skill mappings come from?
   - **Proposed answer**: The LLM generates the mapping. Mappings for the six supported careers are generated once, reviewed and edited by the team, and stored in the repository. Other careers are generated live for the session. The student can edit any mapping (F2).
9. **Ranking factors.** What decides the order of recommendations?
   - **Proposed answer**, in order of weight:
     1. Number of uncovered mapped skills the course teaches
     2. Eligibility: Met first, then reachable within the plan, then Needs review
     3. Number of other recommended courses it unlocks
     4. Match with the student's stated interests
     - Courses matching a topic the student wants to avoid drop to the bottom. Credits do not affect ranking; they are handled by the planner.
10. **Number of recommendations.** How many courses does a plan include?
    - **Proposed answer**: The recommendation list shows the top 10. The plan places 2 career courses per semester by default, adjustable from 1 to 4 by the student.
11. **Required courses the student already has to take.** Can the student enter fixed courses?
    - **Proposed answer**: Yes. The student can list fixed courses (for example known major requirements) and optionally pin each to a semester. The planner counts their credits first and fills only the remaining credits with career courses. Fixed courses are not checked against major requirements.
12. **Offering-term uncertainty.** What happens to courses with unclear offering patterns?
    - **Proposed answer**: A course offered in at least one of the last 4 terms but with no clear fall or spring pattern is included with a warning. A course not offered in the last 4 terms is excluded, since it is likely no longer taught.
13. **Session memory.** Session-only, or saved to disk?
    - **Proposed answer**: Session-only for the MVP. The Markdown export (F8) is the way to keep a plan. Saved profiles move to Phase 2.
14. **Evaluation data.** Who builds the evaluation set, and how is it labeled?
    - **Proposed answer**: The QA and user stories owner builds the 10 evaluation profiles from the personas. The vision and requirements owner labels the 40 prerequisite cases, and a second team member checks every label. Cases where the two disagree are labeled Needs review.
15. **Explanation length and tone.** How much does the agent explain?
    - **Proposed answer**: 1 to 2 plain sentences per course that name the skills it covers. Each plan has a collapsible "How this plan was built" section listing the tool calls the agent made, so students and graders can see the agent's steps.
