# Architecture Templates

Two templates for documenting architectural decisions and proposals.

## When to use each

| Template | Use when |
|---|---|
| **ADR** | A decision has already been made (or is about to be made within one team) and you need to record *what* was decided and *why*. Lightweight, takes ~15 minutes. |
| **RFC** | A change affects more than one team, requires buy-in from Product / QA / Security / Ops, or carries significant risk. Use RFC to align *before* building. Takes longer but prevents surprises. |

**If in doubt:** start with an ADR. Upgrade to an RFC if you find yourself needing sign-off from outside your immediate team.

## File locations

| Document | Location in repo |
|---|---|
| ADR | `docs/adr/NNNN-kebab-case-title.md` — numbered sequentially per repo |
| RFC | `rfc/NNNN-kebab-case-title.md` — or publish to Confluence for cross-team review |

## Skills that use these templates

- `/adr` → `adr-write` skill — interviews you and fills in `adr-template.md`
- `/rfc` → `rfc-write` skill — interviews you and fills in `rfc-template.md`
