# Instructions templates

Reusable VS Code / Copilot **File Instructions** (`.instructions.md`) for projects that do investigative or OSINT documentation.

| File | Use when |
|------|----------|
| `investigative-docs.instructions.md` | Research / leak / OSINT markdown — claim hygiene, quotes, ground-truth diff, completion fiction |

## Install into a project

```bash
mkdir -p /path/to/project/.github/instructions
cp instructions/investigative-docs.instructions.md \
  /path/to/project/.github/instructions/
# Optional: narrow applyTo globs for that repo
```

Also install hub Continue rules (includes `04-narrative-claim-hygiene`):

```bash
/home/gene/projects/ai-efficiency/scripts/setup_project.sh --no-venv
```

Domain pins (debate cast, party-specific dates) stay in the project’s thin `AGENTS.md`, not in this generic template.
