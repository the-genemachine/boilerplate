---
name: artifact-memory
description: >
  MEMORY phase for the AI artifact generator. Stores and retrieves patterns from
  previously generated agents and skills. Recognizes recurring problem signatures,
  tracks which strategies and templates produced the highest quality scores, and
  surfaces reuse opportunities before new generation begins. USE FOR: artifact memory,
  cache artifact, retrieve pattern, reuse template, pattern recognition, similar artifact,
  generation history, strategy performance.
argument-hint: Problem signature or domain to look up, or artifact path + score to store
disable-model-invocation: true
---

# Artifact Memory

## Purpose

Provides the MEMORY & LEARNING layer for the artifact generator pipeline — analogous
to `MemoryAgent` in sparrow-pseudocode-agent. Prevents redundant generation, surfaces
reuse opportunities, and tracks which strategies produce the best results per domain.

Memory is stored in VS Code's session and user memory scopes via `#tool:vscode/memory`.

---

## Memory Structure

### Session Memory (current conversation)
Path: `/memories/session/artifact-pipeline.md`

Tracks artifacts generated in the current session:
```
## Session Artifacts
| Timestamp | Type | Domain | Strategy | Score | Path |
|-----------|------|--------|----------|-------|------|
| <ISO time> | agent | code-review | A | 9.4 | .github/agents/... |
```

### User Memory (persistent across sessions)
Path: `/memories/artifact-patterns.md`

Tracks patterns that produced high-quality artifacts:
```
## High-Quality Patterns
### <domain>/<artifact-type>
- Strategy: <A|B|C>
- Template notes: <what worked>
- Common tools: <tool list>
- Score range: <min>–<max>

## Strategy Performance
| Domain | Strategy A avg | Strategy B avg | Strategy C avg | Recommended |
```

---

## Step 1: RECALL — Check for Similar Artifacts (pre-generation)

Called by the orchestrator BEFORE generating a new artifact.

### 1a. Compute Problem Signature

Extract a normalized signature from the problem description:
- Lowercase the description
- Remove stop words (the, a, an, is, to, for, that, with…)
- Extract the 5 most significant nouns and verbs
- Join as: `<domain>:<verb1>-<noun1>-<verb2>-<noun2>`

Example: "I need an agent that reviews PRs for security issues"
→ Signature: `security:review-pr-detect-vulnerability`

### 1b. Check Session Memory

Load `/memories/session/artifact-pipeline.md`.
- If a session artifact has a matching domain AND 3+ overlapping keywords → **HIT**
- Report: "Similar artifact generated this session: `<path>`. Reuse or generate new?"
- If user wants to reuse → return existing path, skip generation
- If user wants new → proceed with generation, note the variation

### 1c. Check User Memory

Load `/memories/artifact-patterns.md`.
- Look for matching domain + type combination in High-Quality Patterns
- If found: surface the recommended strategy and template notes as pre-generation hints
- These hints are passed as constraints to `generate-agent` or `generate-skill`

### 1d. Report Recall Outcome

```
MEMORY RECALL
─────────────────────────
Signature    : <computed signature>
Session hit  : <YES / NO>
Pattern hit  : <YES / NO>
Recommended  : Strategy <A|B|C> for domain <domain>
Hints        : <any template notes from past successes>
```

---

## Step 2: STORE — Save After Successful Generation (post-generation)

Called by the orchestrator AFTER a successful quality-gated artifact is finalized.

### 2a. Update Session Memory

Append to `/memories/session/artifact-pipeline.md`:
```
| <now> | <type> | <domain> | <strategy> | <score> | <path> |
```

If file doesn't exist yet, create it with the header row first.

### 2b. Update User Memory (only if score ≥ 9.0)

Load `/memories/artifact-patterns.md`.

If the domain/type combination already exists:
- Update the score range if this score is higher or lower
- Add any new template notes that contributed to quality

If new combination:
- Create a new entry under `## High-Quality Patterns`

Template notes to capture:
- Which tools were in the final `tools:` list
- Whether the workflow had 2, 3, 4, or 5 phases
- Any domain-specific rules that scored well in clarity checks

### 2c. Update Strategy Performance Table

Recalculate average score for the domain/strategy combination and update the table.

---

## Step 3: PATTERN RECOGNITION

During generation, actively look for these recurring patterns and apply known-good templates:

| Pattern Type | Signals | Recommended Template |
|-------------|---------|---------------------|
| **Code Reviewer** | "review", "analyze code", "check quality" | Read-only agent, search+read tools, structured report output |
| **Code Generator** | "generate", "scaffold", "create boilerplate" | edit tools required, explore-first workflow |
| **Test Writer** | "test", "spec", "coverage", "assertion" | explore+edit+testFailure tools |
| **Documentation Writer** | "document", "README", "explain code" | read-heavy, edit for output |
| **Pipeline Orchestrator** | "coordinate", "orchestrate", "manage workflow" | Strategy C, subagents |
| **Domain Expert** | single deep domain (security/data/devops) | Domain-specific rules, specialized tool set |
| **Research Assistant** | "find", "explore", "understand", "where is" | Read-only, web+search+memory tools |
| **Deployment Automator** | "deploy", "release", "CI/CD" | terminal tools, confirmation gates |

When a pattern is recognized:
- Report: "Recognized pattern: `<Pattern Type>`. Applying template."
- Skip strategy selection, use the recommended template directly
- Flag to `evaluate-artifact` that a known-good template was used (may allow lower iteration count)

---

## Step 4: LEARNING — Failure Analysis

When an artifact scores < 7.0 and is regenerated, record the failure:

Append to `/memories/artifact-patterns.md` under `## Failure Patterns`:
```
### <domain>/<type> — <ISO date>
- Strategy tried: <A|B|C>
- Score: <X.X>
- Top issues: <issue1>, <issue2>
- Lesson: <one sentence on what to avoid>
```

This prevents the same approach from being tried again for similar problems.

---

## Step 5: Output

After recall, return:
```
ARTIFACT MEMORY REPORT
──────────────────────
Mode: <RECALL | STORE | BOTH>
Signature: <computed>
Cache status: <HIT | MISS | STORED>
Strategy hint: <recommended strategy or "no hint">
Template notes: <any relevant notes or "none">
```
