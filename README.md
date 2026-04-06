# ai-toolkit

A shared toolkit of skills, commands, and prompts for AI agents — designed to work across **Claude** (Cowork / Claude Code) and **GitHub Copilot**, built for team sharing.

## Structure

```
ai-toolkit/
├── claude/          # Claude-specific skills, commands, and plugin config
├── copilot/         # GitHub Copilot instructions and prompt files
├── shared/          # Agent-agnostic prompts and conventions (reusable by both)
└── .github/         # GitHub repo config (PR templates, workflows)
```

## Claude

Skills live under `claude/skills/` — each skill has its own folder with a `SKILL.md` file. To install as a Claude plugin, see `claude/plugin.json`.

| Skill | Description |
|---|---|
| `java-code-reviewer` | Reviews Java code for quality, patterns, and best practices |
| `java-unit-test-generator` | Generates JUnit 5 / Mockito unit tests |
| `java-javadoc-writer` | Writes Javadoc for classes and methods |
| `daily-standup` | Pulls Jira tasks, Confluence mentions, and PR status |

Commands live under `claude/commands/` as markdown files usable as slash commands.

## Copilot

- `copilot/instructions/` — custom instructions files (`.github/copilot-instructions.md` style)
- `copilot/prompts/` — `.prompt.md` files for Copilot Chat custom prompts

## Shared

Agent-agnostic prompt templates and coding conventions under `shared/`. These are the single source of truth — both Claude and Copilot skills reference them.

## Contributing

1. Fork the repo
2. Add your skill/prompt under the relevant agent folder
3. Reference any reusable prompt logic from `shared/`
4. Open a PR — use the template in `.github/PULL_REQUEST_TEMPLATE.md`
