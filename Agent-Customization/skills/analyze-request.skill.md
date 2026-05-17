---
name: analyze-request
description: >
  PERCEIVE phase for the AI artifact generator. Classifies a problem description
  into artifact type (agent/skill/prompt) and domain (code-generation, testing,
  devops, documentation, planning, etc.). Computes confidence scores and flags
  ambiguous inputs for clarification. USE FOR: analyze request, classify input,
  detect artifact type, determine domain, confidence scoring.
argument-hint: The raw problem description to analyze
disable-model-invocation: true
---

# Analyze Request

## Purpose

Transforms a raw problem description into a structured classification that drives
the rest of the artifact generation pipeline. Mirrors the `InputAnalyzer` in
sparrow-pseudocode-agent: detect type → detect domain → score confidence → gate.

---

## Step 1: Artifact Type Detection

Score each type 0.0–1.0 based on signal presence in the input.

### Agent Signals (→ `.agent.md`)
| Signal | Weight |
|--------|--------|
| Contains role/persona language: "reviewer", "planner", "debugger", "assistant that…", "agent that…" | +0.4 |
| Describes ongoing availability: "whenever I…", "every time I…", "always…" | +0.3 |
| Mentions tool restrictions: "should only read", "must never modify" | +0.2 |
| Has a named identity: "a <name> agent" | +0.1 |

### Skill Signals (→ `SKILL.md`)
| Signal | Weight |
|--------|--------|
| Contains step-by-step language: "first… then… finally…", "workflow", "process", "checklist" | +0.4 |
| Describes a repeatable methodology: "whenever I need to X, I want to Y" | +0.3 |
| Has clear inputs and outputs: "given X, produce Y" | +0.2 |
| Multi-phase with decision branches | +0.1 |

### Prompt Signals (→ `.prompt.md`)
| Signal | Weight |
|--------|--------|
| Single focused task with clear parameters | +0.4 |
| Template-like language: "generate a X for Y", "create a Z that does W" | +0.4 |
| No ongoing loop or iteration described | +0.2 |

**Classification rule**: Pick the type with the highest score.
If top two types are within 0.15 of each other → **Ambiguous**, trigger confidence gate.

---

## Step 2: Domain Classification

Scan the description for domain keywords:

| Domain | Keywords |
|--------|---------|
| `code-generation` | write, generate, scaffold, refactor, implement, create code, boilerplate |
| `code-review` | review, analyze, check, audit, quality, lint, assess, critique |
| `testing` | test, spec, assertion, coverage, mock, fixture, unit test, integration test |
| `devops` | deploy, pipeline, CI/CD, Docker, Kubernetes, infrastructure, release, container |
| `documentation` | document, README, comment, docstring, API docs, explain, describe |
| `planning` | plan, design, architecture, research, outline, brainstorm, strategy |
| `data` | SQL, query, database, schema, migration, pipeline, ETL, dataset |
| `security` | security, vulnerability, threat, OWASP, auth, permission, exploit, sanitize |
| `exploration` | explore, find, search, navigate, understand, where is, how does |
| `multi-domain` | Spans 3+ domains above |
| `unknown` | No clear domain signal |

Assign the top domain with a confidence score (0.0–1.0).

---

## Step 3: Complexity Scoring

Estimate complexity to recommend a generation strategy:

| Signal | Complexity Score |
|--------|----------------|
| Single responsibility, clear scope | LOW (→ Strategy A) |
| 2–4 distinct phases or responsibilities | MEDIUM (→ Strategy B) |
| Independent parallel concerns OR needs specialized subagents | HIGH (→ Strategy C) |
| Explicitly mentions "orchestrate", "coordinate", or multiple agents | HIGH (→ Strategy C) |

---

## Step 4: Output

Return a structured classification:

```
ARTIFACT TYPE : <agent | skill | prompt>
TYPE CONFIDENCE : <0.0–1.0>
DOMAIN : <domain name>
DOMAIN CONFIDENCE : <0.0–1.0>
COMPLEXITY : <LOW | MEDIUM | HIGH>
STRATEGY : <A | B | C>
AMBIGUOUS : <YES | NO>
CLARIFICATION NEEDED : <null | one specific question to ask>
```

---

## Step 5: Confidence Gate

If `TYPE CONFIDENCE < 0.90` OR `DOMAIN CONFIDENCE < 0.90`:
- Set `AMBIGUOUS: YES`
- Formulate ONE focused question that resolves the ambiguity
- Do NOT generate multiple questions

Examples of good clarifying questions:
- "Should this run once when invoked (skill), or be available as a persistent assistant with a dedicated persona (agent)?"
- "Is the primary concern code quality review, or is it security vulnerability detection?"
- "Should this coordinate other agents as subagents, or operate independently?"
