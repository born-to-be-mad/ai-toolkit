# ai-toolkit — Development Plan

> Synthesised from: `claude_skills_proposal.html` (architect pain points) + deep analysis of `sales-center` backend (hexagonal arch, OpenAPI-first, testing conventions, `.github` instructions).

---

## Context: What the Real Project Taught Us

The sales-center codebase reveals very specific, non-obvious conventions that generic AI skills get wrong. Skills must encode these precisely:

| Area | Key facts |
|---|---|
| Java version | Java 25 (not 17 or 21) |
| Architecture | Strict hexagonal: `domain/` is pure, `infrastructure/` wires everything |
| Naming | `*Facade` (input port), `*Provider` (output port), `*Controller` (REST adapter), `*Service` (domain impl) |
| Mappers | MapStruct with `@BeanMapping(ignoreByDefault = true)` + explicit `@Mapping` per field |
| Time | Always inject `Clock`; never call `Instant.now()` directly |
| Domain models | Java records with `@Builder(toBuilder = true)` |
| Tests (unit) | JUnit 5 + Mockito + AssertJ + **Instancio** (not manual builders); Given/When/Then |
| Tests (IT) | Extend `AbstractIT`; Testcontainers + Wiremock; `ClockFreezeConfig.NOW` |
| Assertions | `usingRecursiveComparison().ignoringAllOverriddenEquals()` — avoid `getX()` chains |
| OpenAPI | OpenAPI-first; `internal/` vs `external/`; specific path pattern `/api/v1/{domain}/{resource}[:{action}]` |
| YAML quoting | Single-quoted strings; single-quoted HTTP status codes |
| Architecture test | `HexagonalArchitectureTest` exists — skills must not generate code that breaks it |

---

## Full Skill Inventory

### Claude Skills (16 total)

#### Tier 1 — Core Java (highest daily value)

| ID | Skill | Pain point addressed |
|---|---|---|
| `java-unit-test-generator` | Generate JUnit 5 unit tests (Instancio, AssertJ, Given/When/Then) | Inconsistent test quality; missing edge cases |
| `java-integration-test-generator` | Generate IT tests with AbstractIT, Testcontainers, Wiremock | IT tests are slow to write; infrastructure wiring is repetitive |
| `java-code-reviewer` | Review Java code through hexagonal/SOLID/Spring Boot lens | Inconsistent review culture |
| `hexagonal-scaffold-generator` | Generate full feature scaffold (domain model → port → service → adapter → mapper → test) | Boilerplate slows down new feature delivery |
| `openapi-spec-writer` | Write OpenAPI YAML following project path conventions, quoting rules, operationId naming | Specs are inconsistent; naming violations break code generation |
| `mapstruct-mapper-generator` | Generate MapStruct mappers with `@BeanMapping(ignoreByDefault = true)` and explicit `@Mapping` | Mappers frequently miss `ignoreByDefault`; fields silently dropped |

#### Tier 2 — Architecture & Design

| ID | Skill | Pain point addressed |
|---|---|---|
| `adr-writer` | Generate Architecture Decision Records (status, context, options, rationale, consequences) | Lack of documented architecture decisions |
| `c4-diagram-generator` | Generate C4 diagrams (Context → Container → Component) as Mermaid with DDD annotations | No visual system documentation |
| `hexagonal-arch-reviewer` | Specifically review for hexagonal violations (wrong dependencies, domain leakage, naming) | Architecture tests catch violations late; reviews miss subtle leaks |
| `bff-api-dependency-matrix-updater` | Update `bff-svc/docs/api-dependency-matrix.md` Mermaid diagram from source code | Diagram drifts from reality after every change |

#### Tier 3 — Team & Leadership

| ID | Skill | Pain point addressed |
|---|---|---|
| `rfc-template-engine` | Turn a rough idea into a structured RFC (problem, solution, alternatives, risks, sign-off) | Unclear technical roadmaps; no structured cross-team alignment |
| `tech-debt-radar` | Score tech debt items by impact × effort; group by domain; produce prioritised backlog | Technical debt invisible to management |
| `onboarding-kit-generator` | Generate onboarding doc: architecture overview, conventions, "how we review", "how we decide" | New joiners take months to become productive |
| `java-javadoc-writer` | Write Javadoc for classes and methods following project conventions | APIs are undocumented |
| `daily-standup` | Pull Jira tasks, Confluence mentions, PR status into a standup-ready briefing | Morning context-switching overhead |
| `clock-injection-auditor` | Scan codebase for `Instant.now()` / `LocalDate.now()` calls that bypass Clock injection | Known instability: not all services have Clock injection implemented |

### Copilot Assets (5 total)

| ID | Asset | Notes |
|---|---|---|
| `instructions/java-conventions.md` | Updated conventions for Java 25, Instancio, MapStruct, Clock | Current file in `shared/` needs to match real project |
| `instructions/openapi-conventions.md` | Copilot-specific OpenAPI instructions matching `backend-openapi.instructions.md` | Mirror from .github |
| `instructions/testing-conventions.md` | Copilot-specific test instructions matching `backend-tests.instructions.md` | Mirror from .github |
| `prompts/integration-test.prompt.md` | Generate IT test scaffolding for a given endpoint | High-value; IT tests are the slowest to write |
| `prompts/openapi-operation.prompt.md` | Add a new operation to an existing OpenAPI spec following all conventions | Common task, error-prone without guidance |

### Shared Prompts (7 total)

| ID | Prompt | Notes |
|---|---|---|
| `java/hexagonal-feature-design.md` | Design a new feature end-to-end in hexagonal terms (domain model → ports → adapters) | Agent-agnostic |
| `java/java-code-review.md` | Full code review (already created, needs project-specific enhancement) | Update with real conventions |
| `java/refactor-clean-arch.md` | Refactor towards hexagonal (already created) | Needs sales-center package naming |
| `java/openapi-review.md` | Review an OpenAPI spec for convention violations | New |
| `java/explain-to-junior.md` | Explain code to a junior developer (already created) | Good as-is |
| `architecture/adr-template.md` | Raw ADR template (agent-agnostic) | Used by adr-writer skill |
| `architecture/rfc-template.md` | Raw RFC template (agent-agnostic) | Used by rfc-template-engine skill |

---

## Prioritised Development Backlog

### Phase 1 — Immediate Value (Week 1–2)
These fix daily pain and encode real project patterns. Start here.

1. **Update `shared/conventions/java-style-guide.md`** — make it Java 25, add Instancio, Clock injection, MapStruct rules
2. **`java-unit-test-generator`** — most-used skill; encode Instancio + AssertJ + `@DisplayName` + Given/When/Then
3. **`openapi-spec-writer`** — encode path patterns, YAML quoting, tag naming, operationId conventions
4. **`mapstruct-mapper-generator`** — encode `@BeanMapping(ignoreByDefault = true)` and explicit mappings
5. **`java-code-reviewer`** (update) — add hexagonal, Clock injection, Instancio, MapStruct lenses to existing skill

### Phase 2 — Architecture Depth (Week 3–4)

6. **`hexagonal-scaffold-generator`** — generate full feature: model → Facade → Service → Provider → Controller → Mapper → Unit Test
7. **`java-integration-test-generator`** — generate AbstractIT subclass with Testcontainers + Wiremock wiring
8. **`hexagonal-arch-reviewer`** — check specifically for domain layer leakage, wrong naming, missing patterns
9. **`adr-writer`** — encode your personal ADR style
10. **`c4-diagram-generator`** — Mermaid C4 output; seed with sales-center architecture as reference

### Phase 3 — Team Leadership (Week 5–6)

11. **`rfc-template-engine`** — produce full RFC with stakeholder sign-off sections
12. **`onboarding-kit-generator`** — generate onboarding doc; seed with sales-center terminology glossary
13. **`bff-api-dependency-matrix-updater`** — port the existing `.github/prompts/update-api-dependency-matrix.prompt.md` into a Claude skill
14. **`tech-debt-radar`** — interactive HTML artifact; impact × effort scoring

### Phase 4 — Tooling & Polish (Week 7+)

15. **`clock-injection-auditor`** — scanner for `Instant.now()` violations
16. **`java-javadoc-writer`** (update) — align with real conventions
17. **Copilot assets** — sync all Copilot instructions with real project patterns
18. **Shared prompt refinement** — update `java-code-review.md` and `refactor-clean-arch.md` with project specifics

---

## Testing & Validation Approach

Each skill must be validated before it's considered done. Use this protocol:

### For code-generation skills (`hexagonal-scaffold-generator`, `java-unit-test-generator`, `mapstruct-mapper-generator`, `openapi-spec-writer`)

1. **Smoke test** — run the skill against a simple input; verify the output compiles
2. **Convention test** — manually check all generated code against the convention checklist in `shared/conventions/java-style-guide.md`
3. **Real project test** — run the skill against an actual sales-center domain (e.g., `Profile`, `Tour`) and compare to existing implementation
4. **Architecture test gate** — generated code must not break `HexagonalArchitectureTest` when dropped into prospect-svc
5. **Edge case test** — run against a complex input (e.g., entity with nested records, multiple output ports)

### For review skills (`java-code-reviewer`, `hexagonal-arch-reviewer`, `openapi-spec-writer`)

1. **True positive test** — feed intentionally broken code (wrong naming, domain leaking Spring, direct `Instant.now()`) and verify all issues are flagged
2. **True negative test** — feed correct code and verify no false positives
3. **Specificity test** — verify feedback references the project's naming conventions, not generic Java advice
4. **Severity calibration** — verify Critical vs Suggestion split is reasonable

### For document-generation skills (`adr-writer`, `rfc-template-engine`, `onboarding-kit-generator`)

1. **Completeness test** — all required sections present in output
2. **Tone test** — output reads like something you would actually send to a team
3. **Seed test** — run with a real sales-center decision/topic and verify output is usable without heavy editing

### For diagram skills (`c4-diagram-generator`, `bff-api-dependency-matrix-updater`)

1. **Render test** — paste Mermaid output into https://mermaid.live and verify it renders without errors
2. **Accuracy test** — verify nodes and relationships match the actual codebase
3. **Diff test** — make a small change to the source code and re-run; verify only the affected diagram block changes

### CI / Regression (Phase 4+)

- Add a `tests/` directory to the repository
- Each skill gets a `tests/{skill-name}/` folder with:
  - `input.md` — sample input
  - `expected-output-notes.md` — what a correct output must contain (not exact match)
  - `checklist.md` — the pass/fail criteria used during manual validation
- Track which skills have been validated in `PLAN.md` status column

---

## Repository Changes Required

Update `ai-toolkit/` structure to match this plan:

```
ai-toolkit/
├── claude/
│   ├── skills/
│   │   ├── java-unit-test-generator/       ← update SKILL.md (Phase 1)
│   │   ├── java-integration-test-generator/ ← new (Phase 2)
│   │   ├── java-code-reviewer/              ← update SKILL.md (Phase 1)
│   │   ├── java-javadoc-writer/             ← update SKILL.md (Phase 4)
│   │   ├── hexagonal-scaffold-generator/    ← new (Phase 2)
│   │   ├── hexagonal-arch-reviewer/         ← new (Phase 2)
│   │   ├── openapi-spec-writer/             ← new (Phase 1)
│   │   ├── mapstruct-mapper-generator/      ← new (Phase 1)
│   │   ├── adr-writer/                      ← new (Phase 2)
│   │   ├── c4-diagram-generator/            ← new (Phase 2)
│   │   ├── rfc-template-engine/             ← new (Phase 3)
│   │   ├── onboarding-kit-generator/        ← new (Phase 3)
│   │   ├── bff-api-dep-matrix-updater/      ← new (Phase 3)
│   │   ├── tech-debt-radar/                 ← new (Phase 3)
│   │   ├── clock-injection-auditor/         ← new (Phase 4)
│   │   └── daily-standup/                   ← exists, good
│   └── commands/
│       ├── review-pr.md                     ← exists
│       ├── gen-tests.md                     ← exists
│       ├── gen-it.md                        ← new (Phase 2)
│       ├── scaffold-feature.md              ← new (Phase 2)
│       ├── write-openapi.md                 ← new (Phase 1)
│       └── write-adr.md                     ← new (Phase 2)
│
├── copilot/
│   ├── instructions/
│   │   ├── java-conventions.md              ← update (Phase 1)
│   │   ├── openapi-conventions.md           ← new (Phase 2)
│   │   └── testing-conventions.md           ← new (Phase 2)
│   └── prompts/
│       ├── unit-test.prompt.md              ← exists
│       ├── code-review.prompt.md            ← exists
│       ├── integration-test.prompt.md       ← new (Phase 2)
│       └── openapi-operation.prompt.md      ← new (Phase 2)
│
├── shared/
│   ├── conventions/
│   │   └── java-style-guide.md              ← update for Java 25 + Instancio (Phase 1)
│   └── prompts/
│       ├── java/
│       │   ├── java-code-review.md          ← update with real conventions
│       │   ├── refactor-clean-arch.md       ← update with real package naming
│       │   ├── hexagonal-feature-design.md  ← new (Phase 2)
│       │   └── openapi-review.md            ← new (Phase 2)
│       ├── architecture/
│       │   ├── adr-template.md              ← new (Phase 2)
│       │   └── rfc-template.md              ← new (Phase 3)
│       └── general/
│           └── explain-to-junior.md         ← exists, good
│
├── tests/                                   ← new (Phase 4)
│   └── {skill-name}/
│       ├── input.md
│       ├── expected-output-notes.md
│       └── checklist.md
│
├── PLAN.md                                  ← this file
├── README.md
├── LICENSE
└── .gitignore
```

---

## Progress Tracker

| Skill / Asset | Phase | Status |
|---|---|---|
| `shared/conventions/java-style-guide.md` update | 1 | ⬜ TODO |
| `java-unit-test-generator` update | 1 | ⬜ TODO |
| `openapi-spec-writer` | 1 | ⬜ TODO |
| `mapstruct-mapper-generator` | 1 | ⬜ TODO |
| `java-code-reviewer` update | 1 | ⬜ TODO |
| `hexagonal-scaffold-generator` | 2 | ⬜ TODO |
| `java-integration-test-generator` | 2 | ⬜ TODO |
| `hexagonal-arch-reviewer` | 2 | ⬜ TODO |
| `adr-writer` | 2 | ⬜ TODO |
| `c4-diagram-generator` | 2 | ⬜ TODO |
| `rfc-template-engine` | 3 | ⬜ TODO |
| `onboarding-kit-generator` | 3 | ⬜ TODO |
| `bff-api-dep-matrix-updater` | 3 | ⬜ TODO |
| `tech-debt-radar` | 3 | ⬜ TODO |
| `clock-injection-auditor` | 4 | ⬜ TODO |
| `java-javadoc-writer` update | 4 | ⬜ TODO |
| Copilot instructions update | 4 | ⬜ TODO |
| Shared prompt refinement | 4 | ⬜ TODO |
| Test harness (`tests/`) | 4 | ⬜ TODO |
