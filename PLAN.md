# ai-toolkit — Development Plan

> Synthesised from three sources:
> 1. `claude_skills_proposal.html` — architect pain points
> 2. `sales-center` backend — real hexagonal conventions, OpenAPI rules, testing patterns
> 3. `mattpocock/skills` — structural best practices for writing high-quality agent skills

---

## Best Practices Learned from opensource/skills

These rules apply to every skill in this repo:

| Rule | Detail |
|---|---|
| **SKILL.md ≤ 100 lines** | Split anything longer into SKILL.md + REFERENCE.md + EXAMPLES.md |
| **Descriptions trigger loading** | Max 1024 chars. Must include "Use when [specific triggers]". This is the only thing Claude reads before deciding to load a skill. |
| **Action-oriented names** | Prefer short, verb-first kebab-case: `java-tdd` not `java-unit-test-generator`, `hexagonal-scaffold` not `hexagonal-scaffold-generator` |
| **Scripts for determinism** | Add `scripts/` when the same code would be generated repeatedly (validation, file transforms) |
| **Reference files for depth** | Complex skills (like `tdd`) have 5 supporting reference files — mocking, tests, refactoring, deep-modules, interface-design |
| **One-question-at-a-time interviews** | Skills that elicit user input ask one question at a time with a recommended answer |
| **Parallel sub-agents for design** | Skills exploring multiple options spawn parallel agents to produce radically different alternatives |
| **GitHub/Jira issues as output** | Skills that plan work create actual tickets, not just markdown notes |
| **Never couple to file paths** | Issue bodies, ADRs, RFCs reference behaviors and contracts — not line numbers or filenames |

---

## Context: What the Real Project Taught Us

The sales-center codebase reveals very specific, non-obvious conventions that generic AI skills get wrong. Every skill that generates or reviews code must encode these:

| Area | Convention |
|---|---|
| Java version | Java 25 |
| Architecture | Strict hexagonal: `domain/` is pure, `infrastructure/` wires everything |
| Naming | `*Facade` (input port), `*Provider` (output port), `*Controller` (REST adapter), `*Service` (domain impl) |
| Mappers | MapStruct with `@BeanMapping(ignoreByDefault = true)` + explicit `@Mapping` per field |
| Time | Always inject `Clock`; never `Instant.now()` directly |
| Domain models | Java records with `@Builder(toBuilder = true)` |
| Tests (unit) | JUnit 5 + Mockito + AssertJ + **Instancio**; `@DisplayName`; Given/When/Then |
| Tests (IT) | Extend `AbstractIT`; Testcontainers + Wiremock; `ClockFreezeConfig.NOW` |
| Assertions | `usingRecursiveComparison().ignoringAllOverriddenEquals()` — never chain `getX()` |
| OpenAPI | `internal/` vs `external/`; path pattern `/api/v1/{domain}/{resource}[:{action}]`; single-quoted YAML |
| Architecture test | `HexagonalArchitectureTest` exists — generated code must not break it |

---

## Full Skill Inventory (Revised)

### Claude Skills — 22 skills, 4 categories

#### Category 1: Code Generation

| Skill name | Trigger phrase | Pain point | Reference files? |
|---|---|---|---|
| `java-tdd` | "write tests", "TDD", "red-green", "test first" | Inconsistent test quality; wrong assertion style; no Instancio | Yes — `conventions.md`, `mocking.md`, `assertions.md` |
| `java-it` | "integration test", "testcontainers", "IT test", "AbstractIT" | IT scaffolding is slow; infrastructure wiring is repetitive | Yes — `conventions.md`, `setup.md` |
| `hexagonal-scaffold` | "scaffold", "new feature", "new service", "boilerplate" | Full feature (model → port → service → adapter → mapper → test) takes hours | Yes — `layers.md`, `naming.md` |
| `openapi-write` | "openapi spec", "add endpoint", "write spec", "YAML" | Specs violate path/quoting/naming conventions; code gen breaks | Yes — `conventions.md` |
| `mapstruct-mapper` | "mapper", "mapstruct", "DTO mapping", "map to domain" | Forgetting `@BeanMapping(ignoreByDefault = true)`; fields silently dropped | No |
| `java-javadoc` | "javadoc", "document this class", "write docs" | APIs are undocumented | No |

#### Category 2: Code Review & Audit

| Skill name | Trigger phrase | Pain point | Reference files? |
|---|---|---|---|
| `java-review` | "review this", "code review", "check this code", "PR review" | Reviews miss hexagonal violations, Clock injection, MapStruct issues | Yes — `checklist.md` |
| `hexagonal-audit` | "hexagonal audit", "architecture review", "check domain purity" | Architecture tests catch violations late; domain leakage is subtle | Yes — `violations.md` |
| `openapi-review` | "review spec", "openapi review", "check my YAML" | Specs have wrong path patterns, quoting, tag naming | No |
| `clock-audit` | "clock injection", "find Instant.now", "time bug" | Known gap: not all services have Clock injection; hard to find manually | No — script instead |

#### Category 3: Design & Planning

| Skill name | Trigger phrase | Pain point | Reference files? |
|---|---|---|---|
| `grill-feature` | "grill me", "stress-test this design", "interview me about" | Designs ship with unresolved decisions and implicit assumptions | No |
| `design-port` | "design interface", "design port", "design it twice", "multiple options" | First interface idea is rarely best; no systematic exploration | Yes — `dependency-types.md` |
| `hexagonal-feature-design` | "design this feature", "how should I structure", "feature design" | Jumping to code before the domain model is clear | No |
| `adr-write` | "ADR", "architecture decision", "document this decision" | Architecture decisions undocumented; knowledge siloed | Yes — `template.md` |
| `c4-diagram` | "C4 diagram", "system diagram", "architecture diagram", "Mermaid" | No visual system documentation; context lost for new joiners | No |
| `rfc-write` | "RFC", "proposal", "technical proposal", "request for comments" | Cross-team changes go ahead without structured alignment | Yes — `template.md` |

#### Category 4: Workflow & Team

| Skill name | Trigger phrase | Pain point | Reference files? |
|---|---|---|---|
| `triage-bug` | "triage this bug", "investigate", "find root cause", "file a bug" | Bug reports lack root cause; fix plans miss TDD structure | No |
| `feature-to-jira` | "create tickets", "break into issues", "Jira tickets", "vertical slices" | Features land as monolithic tickets; no vertical slices | No |
| `refactor-plan` | "refactor plan", "safe refactor", "tiny commits", "incremental refactor" | Refactors are too big; CI breaks mid-way | No |
| `onboarding-kit` | "onboarding", "new joiner", "team onboarding", "generate onboarding" | New joiners take months to become productive | No |
| `tech-debt-radar` | "tech debt", "debt backlog", "prioritise debt", "impact effort" | Technical debt invisible to management | No |
| `daily-standup` | "standup", "daily briefing", "what's on my plate" | Morning context-switching overhead | No |

---

### Claude Commands (slash commands)

| Command | Invokes | One-liner |
|---|---|---|
| `/java-tdd` | `java-tdd` skill | Write a TDD cycle for selected Java code |
| `/java-it` | `java-it` skill | Scaffold an integration test for this endpoint |
| `/scaffold` | `hexagonal-scaffold` skill | Generate a full hexagonal feature from a description |
| `/review` | `java-review` skill | Code review through hexagonal/SOLID/Spring Boot lens |
| `/audit` | `hexagonal-audit` skill | Audit this service for hexagonal violations |
| `/openapi` | `openapi-write` skill | Write or extend an OpenAPI spec |
| `/mapper` | `mapstruct-mapper` skill | Generate a MapStruct mapper |
| `/adr` | `adr-write` skill | Write an Architecture Decision Record |
| `/rfc` | `rfc-write` skill | Write a Request for Comments document |
| `/grill` | `grill-feature` skill | Get interviewed about this design until it's airtight |
| `/triage` | `triage-bug` skill | Investigate a bug and create a Jira issue with TDD fix plan |
| `/to-jira` | `feature-to-jira` skill | Convert a feature description into vertical-slice Jira tickets |

---

### Copilot Assets

| File | Purpose |
|---|---|
| `instructions/java-conventions.md` | Java 25, hexagonal naming, Instancio, Clock injection, MapStruct — update for real project |
| `instructions/openapi-conventions.md` | Mirror `backend-openapi.instructions.md` from sales-center |
| `instructions/testing-conventions.md` | Mirror `backend-tests.instructions.md` from sales-center |
| `instructions/hexagonal-architecture.md` | Package layout, layer boundaries, dependency rules |
| `prompts/java-tdd.prompt.md` | TDD cycle for selected code |
| `prompts/integration-test.prompt.md` | AbstractIT subclass for given endpoint |
| `prompts/openapi-operation.prompt.md` | Add a new operation to an existing OpenAPI spec |
| `prompts/code-review.prompt.md` | Exists — update with hexagonal/Instancio lens |
| `prompts/unit-test.prompt.md` | Exists — update with Instancio and assertions patterns |

---

### Shared Prompts (agent-agnostic)

| File | Notes |
|---|---|
| `shared/prompts/java/java-code-review.md` | Exists — update with real project conventions |
| `shared/prompts/java/refactor-clean-arch.md` | Exists — update with sales-center package naming |
| `shared/prompts/java/hexagonal-feature-design.md` | New |
| `shared/prompts/java/openapi-review.md` | New |
| `shared/prompts/java/explain-to-junior.md` | Exists — good as-is |
| `shared/prompts/architecture/adr-template.md` | New — raw template used by `adr-write` skill |
| `shared/prompts/architecture/rfc-template.md` | New — raw template used by `rfc-write` skill |
| `shared/conventions/java-style-guide.md` | Exists — update for Java 25 + Instancio + all project patterns |

---

## Revised Directory Structure

```
ai-toolkit/
├── claude/
│   ├── skills/
│   │   ├── java-tdd/
│   │   │   ├── SKILL.md
│   │   │   ├── REFERENCE-conventions.md      ← Instancio, AssertJ, @DisplayName patterns
│   │   │   ├── REFERENCE-mocking.md          ← when/how to mock; spy vs mock
│   │   │   └── REFERENCE-assertions.md       ← recursive comparison, collection assertions
│   │   ├── java-it/
│   │   │   ├── SKILL.md
│   │   │   ├── REFERENCE-conventions.md      ← AbstractIT, Testcontainers, WireMock setup
│   │   │   └── REFERENCE-setup.md            ← ClockFreezeConfig, TestcontainersConfig
│   │   ├── hexagonal-scaffold/
│   │   │   ├── SKILL.md
│   │   │   ├── REFERENCE-layers.md           ← domain → port → service → adapter → mapper
│   │   │   └── REFERENCE-naming.md           ← Facade/Provider/Controller/Service
│   │   ├── java-review/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-checklist.md        ← full review checklist by layer
│   │   ├── hexagonal-audit/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-violations.md       ← common violations catalogue
│   │   ├── openapi-write/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-conventions.md      ← path patterns, quoting, tags, operationId
│   │   ├── mapstruct-mapper/
│   │   │   └── SKILL.md
│   │   ├── java-javadoc/
│   │   │   └── SKILL.md
│   │   ├── openapi-review/
│   │   │   └── SKILL.md
│   │   ├── clock-audit/
│   │   │   ├── SKILL.md
│   │   │   └── scripts/find-clock-violations.sh
│   │   ├── grill-feature/
│   │   │   └── SKILL.md
│   │   ├── design-port/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-dependency-types.md ← in-process / local-sub / remote / external
│   │   ├── hexagonal-feature-design/
│   │   │   └── SKILL.md
│   │   ├── adr-write/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-template.md
│   │   ├── c4-diagram/
│   │   │   └── SKILL.md
│   │   ├── rfc-write/
│   │   │   ├── SKILL.md
│   │   │   └── REFERENCE-template.md
│   │   ├── triage-bug/
│   │   │   └── SKILL.md
│   │   ├── feature-to-jira/
│   │   │   └── SKILL.md
│   │   ├── refactor-plan/
│   │   │   └── SKILL.md
│   │   ├── onboarding-kit/
│   │   │   └── SKILL.md
│   │   ├── tech-debt-radar/
│   │   │   └── SKILL.md
│   │   └── daily-standup/
│   │       └── SKILL.md                      ← exists, good
│   └── commands/
│       ├── java-tdd.md
│       ├── java-it.md
│       ├── scaffold.md
│       ├── review.md
│       ├── audit.md
│       ├── openapi.md
│       ├── mapper.md
│       ├── adr.md
│       ├── rfc.md
│       ├── grill.md
│       ├── triage.md
│       └── to-jira.md
│
├── copilot/
│   ├── instructions/
│   │   ├── java-conventions.md               ← update for Java 25 + project patterns
│   │   ├── openapi-conventions.md            ← new
│   │   ├── testing-conventions.md            ← new
│   │   └── hexagonal-architecture.md         ← new
│   └── prompts/
│       ├── java-tdd.prompt.md                ← new
│       ├── integration-test.prompt.md        ← new
│       ├── openapi-operation.prompt.md       ← new
│       ├── code-review.prompt.md             ← update
│       └── unit-test.prompt.md               ← update
│
├── shared/
│   ├── conventions/
│   │   └── java-style-guide.md               ← update for Java 25 + all patterns
│   └── prompts/
│       ├── java/
│       │   ├── java-code-review.md           ← update
│       │   ├── refactor-clean-arch.md        ← update
│       │   ├── hexagonal-feature-design.md   ← new
│       │   ├── openapi-review.md             ← new
│       │   └── explain-to-junior.md          ← good as-is
│       └── architecture/
│           ├── adr-template.md               ← new
│           └── rfc-template.md               ← new
│
├── tests/                                    ← Phase 4
│   └── {skill-name}/
│       ├── input.md
│       ├── expected-output-notes.md
│       └── checklist.md
│
├── PLAN.md
├── README.md
├── LICENSE
└── .gitignore
```

---

## Prioritised Backlog

### Phase 1 — Foundation (Week 1–2)
Fix daily pain; establish the conventions that all other skills reference.

1. **`shared/conventions/java-style-guide.md`** — update for Java 25, Instancio, Clock, MapStruct
2. **`shared/prompts/architecture/adr-template.md`** and **`rfc-template.md`** — raw templates
3. **`java-tdd`** — most-used; encode Instancio + AssertJ assertions + `@DisplayName` + vertical slices; split into 3 reference files
4. **`openapi-write`** — encode path patterns, quoting, tag naming; REFERENCE-conventions.md
5. **`mapstruct-mapper`** — short skill; encode `@BeanMapping(ignoreByDefault = true)` pattern
6. **Update `copilot/instructions/java-conventions.md`** — align with Java 25, real project patterns

### Phase 2 — Architecture Core (Week 3–4)

7. **`hexagonal-scaffold`** — generate domain → port → service → adapter → mapper → unit test; 2 reference files
8. **`java-it`** — IT scaffolding with AbstractIT/Testcontainers/Wiremock; 2 reference files
9. **`java-review`** — add hexagonal, Clock, Instancio, MapStruct lenses; REFERENCE-checklist.md
10. **`hexagonal-audit`** — specific violation patterns (domain leakage, wrong naming, Spring in domain); REFERENCE-violations.md
11. **`adr-write`** — ADR generator with your personal style; uses shared template
12. **`c4-diagram`** — Mermaid C4 output; reference sales-center architecture as seed
13. **New copilot instructions** — `openapi-conventions.md`, `testing-conventions.md`, `hexagonal-architecture.md`

### Phase 3 — Workflow Skills (Week 5–6)

14. **`grill-feature`** — one-question-at-a-time interview; recommended answers; adapted from `grill-me`
15. **`design-port`** — parallel sub-agents, 3 radically different interface designs; REFERENCE-dependency-types.md
16. **`triage-bug`** — codebase exploration → root cause → Jira issue with TDD fix plan
17. **`feature-to-jira`** — vertical-slice Jira ticket creation; tracer-bullet approach
18. **`refactor-plan`** — tiny-commit refactor planning; Martin Fowler style; files as Jira issue
19. **`rfc-write`** — full RFC with cross-team sign-off sections
20. **`openapi-review`** — check spec against convention checklist

### Phase 4 — Completeness (Week 7+)

21. **`onboarding-kit`** — seeds from sales-center glossary; generates in your voice
22. **`tech-debt-radar`** — interactive HTML artifact; impact × effort scoring
23. **`clock-audit`** — script-based scanner for `Instant.now()` violations; `scripts/find-clock-violations.sh`
24. **`java-javadoc`** — update with project conventions
25. **Shared prompt refinements** — `java-code-review.md`, `refactor-clean-arch.md`, `hexagonal-feature-design.md`
26. **Test harness** (`tests/`) — input + expected-output-notes + checklist per skill

---

## Testing & Validation Protocol

### SKILL.md structural gate (all skills, before any other testing)
- [ ] Frontmatter has `name` and `description`
- [ ] Description includes "Use when [triggers]" and is under 1024 chars
- [ ] SKILL.md is under 100 lines — split if longer
- [ ] Complex content moved to REFERENCE-*.md files
- [ ] Quick start or minimal example present
- [ ] No time-sensitive information

### Code-generation skills (`java-tdd`, `java-it`, `hexagonal-scaffold`, `openapi-write`, `mapstruct-mapper`)
1. **Smoke test** — run against a simple input; verify output compiles
2. **Convention test** — check against `shared/conventions/java-style-guide.md` checklist
3. **Real project test** — run against a real sales-center entity (e.g. `Profile`, `ProspectTour`) and diff against existing implementation
4. **Architecture gate** — generated code must not break `HexagonalArchitectureTest`
5. **Edge case** — complex entity with nested records, multiple output ports

### Review & audit skills (`java-review`, `hexagonal-audit`, `openapi-review`)
1. **True positive** — feed intentionally broken code; verify all issues are flagged with correct severity
2. **True negative** — feed correct code; verify no false positives
3. **Specificity** — feedback references project naming conventions, not generic Java advice
4. **Severity calibration** — Critical vs Suggestion split is reasonable

### Planning & document skills (`adr-write`, `rfc-write`, `grill-feature`, `triage-bug`, `feature-to-jira`)
1. **Completeness** — all required sections present
2. **Tone** — output reads like something you'd actually send to a team
3. **Durability** — no file paths or line numbers in output; references behaviors and contracts only
4. **Jira integration** — tickets are actually created via MCP; not just markdown

### Diagram skills (`c4-diagram`, `tech-debt-radar`)
1. **Render test** — paste Mermaid output into https://mermaid.live; no errors
2. **Accuracy** — nodes and relationships match actual codebase
3. **Diff test** — small code change → re-run → only affected block changes

---

## Progress Tracker

| Skill / Asset | Phase | Status |
|---|---|---|
| `shared/conventions/java-style-guide.md` update | 1 | ⬜ TODO |
| `shared/prompts/architecture/adr-template.md` | 1 | ⬜ TODO |
| `shared/prompts/architecture/rfc-template.md` | 1 | ⬜ TODO |
| `java-tdd` | 1 | ⬜ TODO |
| `openapi-write` | 1 | ⬜ TODO |
| `mapstruct-mapper` | 1 | ⬜ TODO |
| `copilot/instructions/java-conventions.md` update | 1 | ⬜ TODO |
| `hexagonal-scaffold` | 2 | ⬜ TODO |
| `java-it` | 2 | ⬜ TODO |
| `java-review` update | 2 | ⬜ TODO |
| `hexagonal-audit` | 2 | ⬜ TODO |
| `adr-write` | 2 | ⬜ TODO |
| `c4-diagram` | 2 | ⬜ TODO |
| Copilot instructions: openapi, testing, hexagonal | 2 | ⬜ TODO |
| `grill-feature` | 3 | ⬜ TODO |
| `design-port` | 3 | ⬜ TODO |
| `triage-bug` | 3 | ⬜ TODO |
| `feature-to-jira` | 3 | ⬜ TODO |
| `refactor-plan` | 3 | ⬜ TODO |
| `rfc-write` | 3 | ⬜ TODO |
| `openapi-review` | 3 | ⬜ TODO |
| `onboarding-kit` | 4 | ⬜ TODO |
| `tech-debt-radar` | 4 | ⬜ TODO |
| `clock-audit` + script | 4 | ⬜ TODO |
| `java-javadoc` update | 4 | ⬜ TODO |
| Shared prompt refinements | 4 | ⬜ TODO |
| Test harness (`tests/`) | 4 | ⬜ TODO |
