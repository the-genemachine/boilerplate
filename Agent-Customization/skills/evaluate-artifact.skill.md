---
name: evaluate-artifact
description: >
  OBSERVE phase for the AI artifact generator. Performs a 3-axis quality evaluation
  of a generated .agent.md or SKILL.md: Completeness, Correctness, Clarity.
  Detects hallucinations, structural gaps, and invalid frontmatter. Returns a scored
  report with actionable improvement suggestions. USE FOR: evaluate agent quality,
  score skill quality, validate artifact, check agent completeness, detect hallucination,
  quality check agent, quality check skill.
argument-hint: Path to the generated artifact file to evaluate
disable-model-invocation: true
---

# Evaluate Artifact

## Purpose

Scores a generated `.agent.md` or `SKILL.md` on three quality axes — mirroring
the `QualityEvaluator` in sparrow-pseudocode-agent. Acts as the OBSERVE phase
gate that determines whether to accept, refine, or regenerate.

**Overall score formula**:
`(completeness × 0.4 + correctness × 0.4 + clarity × 0.2) × 10`

Quality threshold: **9.0 / 10** to finalize without iteration.

---

## Step 1: Read the Artifact

Read the full contents of the artifact file. Parse:
- Frontmatter (everything between `---` markers)
- Body sections (headings, rules, workflows, capabilities)

---

## Step 2: Completeness Check (0.0–1.0)

Does the artifact cover the full scope of the original request?

### For `.agent.md` files:
| Check | Score Impact |
|-------|-------------|
| `name` present and descriptive | +0.05 |
| `description` present and ≥ 3 sentences | +0.10 |
| `description` contains USE FOR trigger phrases | +0.10 |
| `tools` list present and non-empty | +0.10 |
| Agent identity/role statement in body | +0.10 |
| `<rules>` section with ≥ 3 constraints | +0.15 |
| `<capabilities>` section with ≥ 3 items | +0.15 |
| `<workflow>` section with ≥ 2 phases | +0.15 |
| Handles ambiguous/edge-case input | +0.10 |

### For `SKILL.md` files:
| Check | Score Impact |
|-------|-------------|
| `name` present and kebab-case | +0.05 |
| `description` present and ≥ 3 sentences | +0.10 |
| Purpose section present | +0.10 |
| Prerequisites section present | +0.10 |
| ≥ 3 workflow steps with concrete actions | +0.20 |
| Decision branches (if/else) present in workflow | +0.15 |
| Quality criteria checklist present | +0.15 |
| Output section describes the artifact/result | +0.15 |

---

## Step 3: Correctness Check (0.0–1.0)

Is the logic and content factually valid?

### Hallucination Detection (auto-deduct per violation):
| Violation | Deduction |
|-----------|----------|
| Tool name not in the valid VS Code copilot tools list | -0.20 |
| Agent name in `agents:` that doesn't exist in workspace | -0.15 |
| Frontmatter key that isn't valid for VS Code agents | -0.15 |
| Workflow step that requires a tool not declared in `tools:` | -0.15 |
| Reference to a file path that doesn't exist | -0.10 |
| Contradictory rules (rule A says X, rule B says not X) | -0.10 |

### Valid tool names (reference list):
```
search, read, edit, web, vscode/memory, vscode/askQuestions,
github/issue_read, github.vscode-pull-request-github/issue_fetch,
github.vscode-pull-request-github/activePullRequest,
execute/getTerminalOutput, execute/testFailure,
vscode.mermaid-chat-features/renderMermaidDiagram, agent
```

### Valid frontmatter keys for `.agent.md`:
```
name, description, argument-hint, target, model, tools, agents,
user-invocable, disable-model-invocation, hooks, handoffs
```

### Valid frontmatter keys for `SKILL.md`:
```
name, description, argument-hint, user-invocable, disable-model-invocation
```

### Logic Checks:
- If `disable-model-invocation: true` → body must not rely on LLM judgment (instructions must be deterministic steps)
- If `agents: []` → body must not reference subagents
- If `tools` doesn't include `edit` → body must not instruct file modifications

---

## Step 4: Clarity Check (0.0–1.0)

Are the instructions unambiguous and actionable?

| Check | Score Impact |
|-------|-------------|
| No vague directives ("handle appropriately", "deal with it", "as needed") | +0.20 |
| Each workflow step has a concrete action verb | +0.20 |
| Rules say what to do AND what not to do | +0.20 |
| Description trigger phrases match the actual capabilities | +0.20 |
| Example prompts or argument-hint present | +0.10 |
| No placeholder text left unfilled (`<...>`, `TODO`, `...`) | +0.10 |

---

## Step 5: Issue Catalog

For each deduction, record a specific issue:

```
ISSUE [severity: HIGH|MEDIUM|LOW]
Type: <Completeness|Correctness|Clarity>
Location: <section or field name>
Problem: <specific description>
Fix: <concrete suggested change>
```

---

## Step 6: Output Report

```
═══════════════════════════════════════
 ARTIFACT QUALITY REPORT
═══════════════════════════════════════
File: <path>
Type: <agent | skill>
Domain: <domain>

SCORES
  Completeness : X.XX / 1.00
  Correctness  : X.XX / 1.00
  Clarity      : X.XX / 1.00
  ─────────────────────────────────
  Overall      : X.X / 10.0

VERDICT: <PASS ≥9.0 | REFINE 7.0-8.9 | REGENERATE <7.0>

ISSUES FOUND: <count>
  [HIGH]   <issue 1>
  [MEDIUM] <issue 2>
  [LOW]    <issue 3>

IMPROVEMENTS NEEDED:
  1. <Specific actionable fix>
  2. <Specific actionable fix>
═══════════════════════════════════════
```

---

## Step 7: Iteration Guidance

Based on verdict:

- **PASS**: Return report, trigger finalization
- **REFINE**: Apply HIGH and MEDIUM issues only. Re-evaluate. Max 1 re-evaluation cycle.
- **REGENERATE**: Discard artifact. Return to ACT phase with a different strategy. Include the top 3 issues as constraints for the next generation attempt.
