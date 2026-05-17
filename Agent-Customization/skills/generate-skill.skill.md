---
name: generate-skill
description: >
  ACT phase skill for generating SKILL.md files. Takes a structured classification
  (from analyze-request) and a workflow description and produces a complete, valid
  skill file with step-by-step instructions, decision branches, quality criteria,
  and output specification. USE FOR: generate skill file, write skill markdown,
  create skill template, build skill from classification, scaffold skill, workflow
  skill generation.
argument-hint: Workflow description + classification output from analyze-request
disable-model-invocation: true
---

# Generate Skill

## Purpose

Produces a complete, valid `SKILL.md` file from a workflow description and its
classification. Skills package repeatable multi-step workflows — this generator
ensures every skill has concrete steps, decision logic, and measurable quality criteria.
This is the ACT phase of the generator pipeline for skill artifacts.

---

## Step 1: Classify Skill Complexity

Determine the skill's internal structure:

| Signal | Tier |
|--------|------|
| Linear steps, single output, no branching | SIMPLE (3–4 steps) |
| Multiple phases with decision points | STANDARD (5–7 steps) |
| Sub-workflows, parallel paths, or external tool coordination | COMPLEX (7–10 steps) |

Tier drives step count and branching depth.

---

## Step 2: Extract Required Elements from Problem Description

Parse the input for these building blocks:

**Input**: What does the user provide to trigger this skill?
- Explicit: "given a PR", "given a file path", "given a problem description"
- Implicit: derive from context (code-review skill → implicit input is code/file)

**Output**: What artifact or state change does this skill produce?
- Explicit: "generates a test file", "produces a review report"
- Implicit: derive from domain (documentation skill → produces docs)

**Prerequisites**: What must be true before this skill runs?
- Access requirements (repo access, running server, etc.)
- Context requirements (existing code, config file, etc.)

**Steps**: Extract any explicit workflow hints from the description.
Look for: "first", "then", "after", "finally", numbered lists, phase names.

---

## Step 3: Build the Description Field

```
<What this skill produces in one sentence>. <When to invoke it>.
USE FOR: <trigger phrase 1>, <trigger phrase 2>, <trigger phrase 3>.
DO NOT USE FOR: <anti-trigger 1>.
```

---

## Step 4: Build Prerequisites Section

List 2–5 prerequisites. Categorize as:
- **Input**: What data or artifact the user must provide
- **Access**: What systems, repos, or services must be accessible  
- **State**: What must already be true in the workspace
- **Tools**: Which tools the skill relies on (for `disable-model-invocation: true` skills)

---

## Step 5: Build Workflow Steps

For each step, follow this pattern:

```markdown
### Step N: <Action Verb + Object>

<2–4 sentences describing WHAT to do and HOW. Be concrete. 
Reference specific tools, file paths, or commands where applicable.>

<Include a decision point if the step has multiple outcomes:>
- If <condition A> → <action / proceed to step N+1>
- If <condition B> → <alternative action / skip to step N+2>
- If <error condition> → <recovery action>
```

**Step naming by domain:**
| Domain | Typical Step Sequence |
|--------|-----------------------|
| `code-review` | Load Context → Identify Scope → Analyze → Score → Report |
| `testing` | Discover Patterns → Identify Gaps → Generate Tests → Validate → Report |
| `documentation` | Understand Code → Gather Context → Draft → Review → Finalize |
| `planning` | Discover → Clarify → Design → Validate → Document |
| `devops` | Verify Prerequisites → Configure → Execute → Monitor → Confirm |
| `security` | Enumerate Surface → Classify Threats → Assess → Report → Recommend |
| `code-generation` | Understand Requirements → Explore Patterns → Draft → Verify → Finalize |

---

## Step 6: Build Quality Criteria

Generate 4–7 measurable quality checks as a checklist. Each criterion must be:
- Binary (pass/fail, not subjective)
- Verifiable without running the code
- Specific to this skill's output

Examples of good quality criteria:
```
- [ ] All referenced files exist in the workspace
- [ ] Generated code follows the naming conventions in the codebase
- [ ] Every TODO in the original is addressed or explicitly deferred
- [ ] Output file is saved to the correct path
```

Examples of BAD quality criteria (too vague):
```
- [ ] Output is good quality  ← REJECT: not measurable
- [ ] Workflow was followed   ← REJECT: tautological
```

---

## Step 7: Build Output Section

Describe exactly what the skill produces:
- File created/modified: path and format
- State change: what is different after the skill runs
- Report/output: what gets shown to the user

---

## Step 8: Assemble the File

```markdown
---
name: <kebab-case-name>
description: >
  <Built in Step 3>
argument-hint: <One-line usage hint>
disable-model-invocation: true
---

# <Title Case Name>

## Purpose
<One paragraph: what problem this solves and when to use it>

## Prerequisites
- **Input**: <required input>
- **Access**: <required access>
- **State**: <required state>

## Workflow

### Step 1: <Action>
<content from Step 5>

### Step 2: <Action>
<content from Step 5>

[continue for all steps]

## Quality Criteria
- [ ] <criterion from Step 6>
- [ ] <criterion from Step 6>

## Output
<content from Step 7>
```

---

## Step 9: Path Selection

Skills are stored in a named subdirectory:
- **Workspace-scoped**: `.github/skills/<kebab-name>/SKILL.md`

If the directory doesn't exist, create it.

Skill name rules:
- All lowercase, hyphen-separated: `code-review`, `generate-tests`, `deploy-check`
- Should be a verb-noun pair describing the action + subject
- 2–4 words maximum

---

## Quality Pre-Check

Before passing to `evaluate-artifact`, self-check:
- [ ] All frontmatter keys are valid for SKILL.md
- [ ] No `<placeholder>` text remains
- [ ] `description` has USE FOR phrases
- [ ] Every workflow step has a concrete action verb
- [ ] At least one step has a decision branch (if/else)
- [ ] Quality criteria are measurable and binary
- [ ] Output section is specific (names files/reports/state changes)
