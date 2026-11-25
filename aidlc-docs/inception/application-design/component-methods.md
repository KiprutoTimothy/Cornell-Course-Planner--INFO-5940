# Component Methods

**Stage**: INCEPTION - Application Design

Signatures are Python 3.12 with full type hints, following the technical environment's examples. They define inputs, outputs, and purpose only. **Detailed business rules are defined in Functional Design (per unit, Construction).** Types in `PascalCase` are Pydantic models unless noted. `CourseCode` is a `str` that has passed C6 normalization.

---

## C1 Settings (`config`)

```python
class Settings(BaseSettings):
    # fields as in docs/technical-environment.md, Example 5
    @property
    def active_model(self) -> str: ...

def get_settings() -> Settings: ...
```

---

## `catalog`

### C2 Roster Client
```python
class RosterClient:
    def __init__(self, base_url: str, http: httpx.AsyncClient, min_interval_s: float = 1.0) -> None: ...
    async def list_terms(self) -> list[str]: ...
    async def fetch_subject(self, term: str, subject: str) -> RawRosterResponse: ...
```
- **Purpose**: throttled, timed-out, retried fetches for ingestion only.

### C3 Snapshot Store
```python
class SnapshotStore:
    def __init__(self, root: Path) -> None: ...
    def has_roster(self, term: str, subject: str) -> bool: ...          # resume support
    def save_roster(self, term: str, subject: str, raw: RawRosterResponse) -> None: ...
    def iter_courses(self) -> Iterator[CourseRecord]: ...
    def load_rules(self) -> list[StoredRule]: ...
    def save_rules(self, rules: list[StoredRule]) -> None: ...
```

### C4 Index Builder
```python
class IndexBuilder:
    def __init__(self, snapshot: SnapshotStore, index_dir: Path, embedder: TextEmbedding, embedding_model: str) -> None: ...
    def build(self) -> IndexBuildSummary: ...        # SQLite + FTS5 + Chroma; summary counts per FR-3 / US-10.1
```

### C5 Catalog Store
```python
class CatalogStore:
    def __init__(self, db_path: Path) -> None: ...
    def has_course(self, code: CourseCode) -> bool: ...
    def get_course(self, code: CourseCode) -> Course | None: ...
    def get_courses(self, codes: Iterable[CourseCode]) -> dict[CourseCode, Course]: ...
    def offering_terms(self, code: CourseCode) -> list[str]: ...          # e.g. ["FA23", "FA24", "SP25"]
    def get_rule(self, code: CourseCode, kind: RuleKind) -> StoredRule | None: ...
    def forbidden_overlaps(self, code: CourseCode) -> frozenset[CourseCode]: ...
    def subjects(self) -> frozenset[str]: ...

def get_catalog() -> CatalogStore: ...                                  # FastAPI dependency
```

### C6 Course Codes
```python
class InvalidCourseCodeError(ValueError):
    public_message: str

def normalize_course_code(raw: str) -> CourseCode: ...
def normalize_course_codes(raw_list: str, max_codes: int = 80) -> list[CourseCode]: ...
def extract_codes(text: str) -> list[CourseCode]: ...                  # codes mentioned in free text, in order
def course_level(code: CourseCode) -> int: ...                         # 1000, 2000, ..., 6000
```

### C7 Hybrid Search
```python
class HybridSearch:
    def __init__(self, catalog: CatalogStore, collection: chromadb.Collection, embedder: TextEmbedding) -> None: ...
    def query(self, text: str, top_k: int, min_level: int, max_level: int) -> list[SearchHit]: ...

def rrf_merge(*ranked: Sequence[CourseCode], k: int = 60) -> list[CourseCode]: ...  # pure; property-tested
```

### C8 Grounding Check
```python
class GroundingCheck:
    def __init__(self, catalog: CatalogStore) -> None: ...
    def filter_codes(self, codes: Iterable[str], source: str) -> GroundingResult: ...   # kept, dropped; logs fabrication events
```

---

## `prereq`

### C9 Rule Model
```python
class CourseReq(BaseModel): kind: Literal["course"]; code: CourseCode
class ReviewFlag(BaseModel): kind: Literal["review"]; reason: str; clause_id: str      # clause_id added for CR-1
class AllOf(BaseModel): kind: Literal["all"]; items: list[Rule]
class AnyOf(BaseModel): kind: Literal["any"]; items: list[Rule]
Rule = Annotated[CourseReq | ReviewFlag | AllOf | AnyOf, Field(discriminator="kind")]

class StoredRule(BaseModel):
    course: CourseCode
    kind: RuleKind                    # "prerequisite" | "corequisite"
    original_text: str
    rule: Rule | None
    verified: bool
    method: Literal["pattern", "llm"]
    assumptions: tuple[str, ...]      # FR-5.6 default "and/or" reading notes
```

### C10 Pattern Parser
```python
def parse_simple(text: str) -> Rule | None: ...                       # None = not a known pattern; send to LLM
```

### C11 Parse Checks
```python
def check_llm_parse(text: str, first: Rule, second: Rule, known_subjects: frozenset[str],
                    catalog_has: Callable[[CourseCode], bool]) -> ParseCheckResult: ...
# ParseCheckResult: verified: bool, failed_checks: tuple[str, ...]
```

### C12 Prerequisite Evaluator
```python
class Status(StrEnum): MET = "Met"; NOT_MET = "Not met"; NEEDS_REVIEW = "Needs review"

class Verdict(BaseModel):
    status: Status
    missing: frozenset[CourseCode]
    reasons: tuple[str, ...]
    original_text: str
    student_confirmed: bool = False   # CR-1: Met that relies on a confirmed clause
    assumptions: tuple[str, ...] = ()

def evaluate(rule: Rule | None, taken: frozenset[CourseCode], original_text: str,
             verified: bool = True, confirmed_clauses: frozenset[str] = frozenset()) -> Verdict: ...
```
- **Change from the technical environment's Example 2**: adds `confirmed_clauses` (clause IDs the student confirmed for *this* course, FR-1.9.4) and `Verdict.student_confirmed`. The example must be updated in `docs/technical-environment.md` by its owner (see `application-design.md`, Open Items).

### C13 Chain Builder
```python
def build_chain(target: CourseCode, taken: frozenset[CourseCode],
                rule_for: Callable[[CourseCode], StoredRule | None], max_depth: int = 6) -> PrereqChain: ...
```

### C14 Checklist Builder *(CR-1)*
```python
def build_checklist(stored: StoredRule, taken: frozenset[CourseCode],
                    confirmed_clauses: frozenset[str], loaded_subjects: frozenset[str]) -> Checklist: ...
# Checklist.items: list[ChecklistItem(item_id, kind: "course" | "clause", label, satisfied, outside_catalog)]
```

---

## `planner`

### C15 Planning Models
```python
class ConfirmedPrerequisites(BaseModel):          # CR-1
    clauses: dict[CourseCode, frozenset[str]]       # course -> confirmed clause IDs
    codes_from_checklist: frozenset[CourseCode]     # codes added via a checklist (for the review list and export)

class StudentProfile(BaseModel):
    student_type: Literal["undergraduate", "masters"]
    program: str; college: str; year: str
    graduation_term: str
    credit_min: int; credit_max: int
    completed: list[CourseCode]; in_progress: list[CourseCode]
    unknown_codes: list[str]                        # typed codes not in catalog (FR-1.2)
    fixed: dict[CourseCode, str]                    # course -> term
    unavailable_terms: frozenset[str]
    interests: str; avoid_topics: str
    skills: list[Skill]
    confirmed: ConfirmedPrerequisites

    def taken(self) -> frozenset[CourseCode]: ...   # completed + in-progress + checklist codes

class Plan(BaseModel): terms: dict[str, list[PlannedCourse]]
class PlannedCourse(BaseModel): code: CourseCode; credits: int; skills: list[str]; explanation: str
                                 verdict: Verdict; offering: OfferingStatus; fixed: bool
class Violation(BaseModel): kind: ViolationKind; course: CourseCode | None; term: str | None; detail: str
class PlanChange(BaseModel): action: Literal["swap", "move", "add", "remove", "set_credits", "block_term"]; ...
class ProfileChange(BaseModel): field: ProfileField; value: ...   # includes confirm_items / unconfirm_items (CR-1)
```

### C16 Offering and Credit Rules
```python
def plan_terms(graduation_term: str, unavailable: frozenset[str]) -> list[str]: ...     # SP27..FA28 max
def offering_status(code: CourseCode, term: str, history: list[str]) -> OfferingStatus: ...  # ok | wrong_term | uncertain
def term_credits(courses: Iterable[PlannedCourse]) -> int: ...
def overlaps(codes: Iterable[CourseCode], taken: frozenset[CourseCode],
             overlap_of: Callable[[CourseCode], frozenset[CourseCode]]) -> list[tuple[CourseCode, CourseCode]]: ...
```

### C17 Plan Validator
```python
def validate_plan(plan: Plan, profile: StudentProfile, catalog: CatalogStore) -> list[Violation]: ...   # empty = valid
```

### C18 Plan Fixer
```python
def fix_plan(plan: Plan, violations: list[Violation], profile: StudentProfile,
             catalog: CatalogStore) -> FixResult: ...
# FixResult: plan: Plan | None, changes: list[str], conflict: PlanConflict | None   (conflict -> FR-6.6)
```

### C19 Recommendation Ranker
```python
def rank_candidates(hits_by_skill: dict[str, list[SearchHit]], profile: StudentProfile,
                    catalog: CatalogStore) -> list[Recommendation]: ...
```

### C20 Plan Editor
```python
def apply_plan_change(plan: Plan, change: PlanChange, profile: StudentProfile) -> EditResult: ...
def apply_profile_change(profile: StudentProfile, change: ProfileChange) -> StudentProfile: ...
# EditResult: plan: Plan, changed: list[str]
```

---

## `agent`

### C21 LLM Gateway
```python
class LLMGateway:
    def __init__(self, settings: Settings) -> None: ...
    async def structured(self, prompt: Prompt, data: dict[str, str], schema: type[T],
                         thinking: bool = False) -> T: ...      # 60 s timeout, 3 attempts, Pydantic-validated
    def chat_model(self) -> BaseChatModel: ...                  # for the LangGraph agent node
    async def ping(self) -> bool: ...                           # deep health
```

### C22 Prompt Library
```python
def load_prompt(name: str, version: str = "v1") -> Prompt: ...
```

### C23 Skill Mapper
```python
class SkillMapper:
    def __init__(self, gateway: LLMGateway, reviewed_path: Path) -> None: ...
    def reviewed_careers(self) -> list[str]: ...
    async def assess_goal(self, goal: str) -> GoalAssessment: ...          # specific | vague + follow-up questions
    async def map_skills(self, career: str, completed: frozenset[CourseCode]) -> SkillMap: ...   # source: reviewed | ai_generated
    async def compare(self, careers: list[str]) -> CareerComparison: ...    # 2 to 3 careers
    async def draft_reviewed_mapping(self, career: str) -> SkillMap: ...    # used by a team command
```

### C24 Ingestion Parser
```python
class IngestionParser:
    def __init__(self, gateway: LLMGateway) -> None: ...
    async def parse_twice(self, text: str) -> tuple[Rule, Rule]: ...       # two independent parses, thinking on
```

### C25 Agent Tools
```python
def make_tools(ctx: ToolContext) -> list[BaseTool]: ...
# ToolContext: search, catalog, profile snapshot (read-only), current plan (read-only), level range
# tools: search_courses(skill, limit) | get_course(code) | check_prerequisites(code)
#        get_prerequisite_chain(code) | validate_plan() | explain_course(code)
#        propose_plan_change(change: PlanChange) -> PlanChange          (returns, never applies)
#        propose_profile_change(change: ProfileChange) -> ProfileChange (returns, never applies)
```

### C26 Planning Graph
```python
def build_planning_graph(deps: GraphDeps) -> CompiledStateGraph: ...    # first plan, fixed nodes
def build_revision_graph(deps: GraphDeps, checkpointer: InMemorySaver) -> CompiledStateGraph: ...
# PlanningState: profile, skills, hits_by_skill, recommendations, proposal, violations, attempts, plan, conflict
```

### C27 Services
Methods are listed in `services.md`.

---

## `web`

### C28 Session Store
```python
class SessionState: profile: StudentProfile; plan: Plan | None; csrf_token: str; last_seen: float; limiter: TokenBucket
class SessionStore:
    def __init__(self, idle_timeout_s: int = 3600) -> None: ...
    def create(self) -> tuple[str, SessionState]: ...
    def get(self, session_id: str) -> SessionState | None: ...
    def purge_idle(self) -> int: ...

def get_session(request: Request) -> SessionState: ...                  # FastAPI dependency
```

### C29 Security Middleware
```python
class SecurityHeadersMiddleware: ...
class BodySizeLimitMiddleware: ...                                      # 64 KB
class RequestIdMiddleware: ...
def verify_csrf(session: SessionState, token: str) -> None: ...          # secrets.compare_digest
def check_chat_rate(session: SessionState) -> None: ...                  # 10 per minute
```

### C30 Routes (indicative; final paths in Functional Design of the web unit)
| Method and path | Purpose | Calls |
|---|---|---|
| `GET /` | Main page | — |
| `POST /profile` | Save academic profile (US-1.1) | `ProfileService.update_profile` |
| `POST /profile/completed-courses` | Courses taken (US-1.2) | `ProfileService.set_courses` |
| `POST /profile/constraints` | Fixed courses, unavailable terms, interests (US-1.3) | `ProfileService.set_constraints` |
| `GET /prereq/checklist/{code}` | Checklist fragment (US-1.4) | `ProfileService.get_checklist` |
| `POST /prereq/confirm` | Confirm or remove items (US-1.4) | `ProfileService.confirm_prerequisites` |
| `POST /career/goal` | Map goal or ask follow-ups (US-2.1, US-2.2) | `CareerService.submit_goal` |
| `POST /career/skills` | Edit skills (US-2.3) | `CareerService.edit_skills` |
| `POST /career/compare` | Compare careers (US-2.4) | `CareerService.compare` |
| `POST /plan` | First plan (US-4.1 to US-6.3) | `PlanningService.create_plan` |
| `POST /plan/relax-constraint` | Answer a "does not fit" question (US-6.3) | `PlanningService.resolve_conflict` |
| `POST /chat` | Revision, "why", start over (US-7.x) | `RevisionService.handle_message` |
| `GET /prereq/chain/{code}` | Prerequisite chain (US-5.3) | `PlanningService.prerequisite_chain` |
| `GET /plan/export` | Markdown download (US-8.1) | `ExportService.export_markdown` |
| `GET /health`, `GET /health/deep` | Health (US-11.3) | `HealthService` |

### C31 Health Checks and C32 Error Handler
```python
def register_health_routes(app: FastAPI) -> None: ...
def register_error_handlers(app: FastAPI) -> None: ...
```

---

## `evaluation`

### C33 Evaluation Cases
```python
def load_cases(root: Path) -> EvaluationSet: ...    # profiles, prereq cases; validates reviewer, core subjects
```

### C34 Evaluation Runner
```python
async def run_evaluation(cases: EvaluationSet, services: Services, catalog: CatalogStore) -> EvaluationReport: ...
def write_report(report: EvaluationReport, out_dir: Path) -> Path: ...   # evaluation/reports/YYYY-MM-DD.md
```
