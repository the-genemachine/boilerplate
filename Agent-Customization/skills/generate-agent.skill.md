---
name: generate-agent
description: >
  ACT phase skill for generating .agent.md files. Takes a structured classification
  (from analyze-request) and a problem description and produces a complete, valid
  agent file using domain-appropriate templates. USE FOR: generate agent file,
  write agent markdown, create agent template, build agent from classification,
  agent code generation, scaffold agent.
argument-hint: Problem description + classification output from analyze-request
disable-model-invocation: true
---

# Generate Agent

## Purpose

Produces a complete, valid `.agent.md` file from a problem description and its
classification. Applies domain-specific templates and tool selection heuristics
to generate high-quality agents on the first attempt. This is the ACT phase
of the generator pipeline.

---

## Step 1: Select Generation Strategy

Based on the `COMPLEXITY` from `analyze-request`:

| Complexity | Strategy | What to generate |
|-----------|----------|-----------------|
| LOW | A — Single agent | One `.agent.md`, standalone |
| MEDIUM | B — Agent + skills | One `.agent.md` with skill references |
| HIGH | C — Multi-agent | One coordinator `.agent.md` + subagent stubs |

For Strategy C, generate the coordinator first, then queue subagent generation.

---

## Step 2: Select Tools Based on Domain

Use this mapping to populate the `tools:` frontmatter. Only include tools the agent
will actually USE — never add "just in case":

| Domain | Core Tools | Optional (add if workflow requires) |
|--------|-----------|--------------------------------------|
| `code-generation` | `search`, `read`, `edit` | `execute/getTerminalOutput` |
| `code-review` | `search`, `read`, `vscode/askQuestions` | `web` |
| `testing` | `search`, `read`, `edit`, `execute/getTerminalOutput` | `execute/testFailure` |
| `devops` | `search`, `read`, `edit`, `execute/getTerminalOutput` | `web` |
| `documentation` | `search`, `read`, `edit` | `web` |
| `planning` | `search`, `read`, `web`, `vscode/memory` | `vscode/askQuestions`, `agent` |
| `data` | `search`, `read`, `edit` | `execute/getTerminalOutput` |
| `security` | `search`, `read`, `vscode/askQuestions` | `web` |
| `exploration` | `search`, `read`, `web`, `vscode/memory` | `execute/getTerminalOutput` |
| `multi-domain` | `search`, `read`, `edit`, `web` | `vscode/askQuestions`, `agent` |

Always add `vscode/askQuestions` if the agent needs to handle ambiguous inputs.

---

## Step 3: Build the Description Field

The `description` is the agent's discovery surface. Write it as:

```
<Primary job statement>. <Key capability 1>. <Key capability 2>.
USE FOR: <trigger phrase 1>, <trigger phrase 2>, <trigger phrase 3>,
<trigger phrase 4>, <trigger phrase 5>.
DO NOT USE FOR: <anti-trigger 1>, <anti-trigger 2>.
```

Rules for description quality:
- Front-load with what the agent DOES (not what it is)
- Include 5–8 USE FOR trigger phrases matching how a user would ask for it
- Include 2–3 DO NOT USE FOR phrases to avoid collisions with other agents
- Keep under 5 sentences total

---

## Step 4: Build the Rules Section

Generate 5–8 rules. Balance DO and DON'T:

**Always include:**
- One "NEVER modify/write/execute" rule if domain is read-heavy (review, exploration, planning)
- One "use `#tool:vscode/askQuestions` to clarify before acting" rule
- One rule about scope boundaries (what this agent handles vs. defers)

**Domain-specific rules:**
| Domain | Recommended Rules |
|--------|--------------------|
| `code-generation` | Never generate code without understanding the existing patterns first |
| `code-review` | Never suggest changes that aren't backed by a specific reason |
| `testing` | Always run existing tests before modifying test files |
| `security` | Never suggest security changes without explaining the threat model |
| `planning` | Never start implementation — plans are for others to execute |
| `devops` | Always confirm destructive operations before executing |

---

## Step 5: Build the Capabilities Section

Generate 4–7 capabilities. Each should be:
- Named with a bold heading: `**Capability Name**`
- Followed by a colon and one-sentence description
- Concrete and specific to the domain (not generic)

Pattern: `**<Action noun>**: <What it does and how>`

---

## Step 6: Build the Workflow Section

Generate 2–5 workflow phases. Each phase must have:
1. A numbered heading (`## 1. Phase Name`)
2. 2–4 concrete, actionable steps (not vague guidance)
3. At least one decision point ("If X → Y, else → Z") per complex phase

**Phase count by complexity:**
- LOW: 2 phases (Understand → Execute)
- MEDIUM: 3–4 phases (Understand → Research → Execute → Validate)
- HIGH: 4–5 phases (full pipeline with subagent coordination)

---

## Step 7: Assemble the File

Compose the final file using this structure:

```markdown
---
name: <Name in Title Case>
description: >
  <Built in Step 3>
argument-hint: <One-line usage hint>
target: vscode
tools: [<built in Step 2>]
agents: [<subagent names if Strategy C>]
---

You are a <ROLE> — <one-sentence identity>.

Your job: <primary mission statement>.

<rules>
<rules from Step 4>
</rules>

<capabilities>
<capabilities from Step 5>
</capabilities>

<workflow>
<workflow from Step 6>
</workflow>
```

---

## Step 8: Path Selection

Determine save path:
- **Workspace-scoped** (default): `.github/agents/<kebab-name>.agent.md`
- **User-level** (if user said "personal" or "for all projects"): `{{VSCODE_USER_PROMPTS_FOLDER}}/<kebab-name>.agent.md`

If the workspace doesn't have a `.github/agents/` directory yet, create it.

---

## Quality Pre-Check

Before passing to `evaluate-artifact`, self-check:
- [ ] All frontmatter keys are valid
- [ ] No tool names are fabricated
- [ ] No `<placeholder>` text remains
- [ ] `description` has USE FOR phrases
- [ ] `<rules>` has at least one DO NOT rule
- [ ] Workflow has at least 2 phases with concrete steps
