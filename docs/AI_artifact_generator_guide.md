# AI Artifact Generator — Usage Guide

A production-grade system for autonomously generating VS Code AI agents (`.agent.md`) and skills (`SKILL.md`). Inspired by [sparrow-pseudocode-agent](https://github.com/the-genemachine/sparrow-pseudocode-agent), this generator uses a `PERCEIVE → PLAN → ACT → OBSERVE → DECIDE` pipeline to transform problem descriptions into high-quality automation artifacts.

---

## Quick Start

In any VS Code chat:

```
/Generate AI Artifact <your problem description>
```

Example:
```
/Generate AI Artifact I need an agent that reviews pull requests for security vulnerabilities
```

The system will:
1. **Analyze** your input (type, domain, complexity)
2. **Recall** similar past artifacts from memory
3. **Generate** a candidate agent or skill
4. **Evaluate** quality on 3 axes (Completeness, Correctness, Clarity)
5. **Iterate** up to 3 times until quality score ≥ 9.0/10
6. **Save** the artifact and suggest related customizations

---

## Example 1: Generate a Security Code Review Agent

### Input
```
/Generate AI Artifact I want an agent that analyzes pull requests for security vulnerabilities. 
It should check for common issues like SQL injection, XSS, unsafe dependencies, 
and cryptography mistakes. It should NOT modify code, only report findings.
```

### Pipeline Execution

#### Phase 1: PERCEIVE (analyze-request)
```
ARTIFACT TYPE     : agent
TYPE CONFIDENCE   : 0.98
DOMAIN            : security
DOMAIN CONFIDENCE : 0.96
COMPLEXITY        : MEDIUM
STRATEGY          : B — Agent with supporting skills
AMBIGUOUS         : NO
```

#### Phase 2: MEMORY (artifact-memory)
```
ARTIFACT MEMORY REPORT
──────────────────────
Signature: security:analyze-pr-vulnerability-detection
Cache hit: MISS (no similar agents in session)
Pattern recognized: **Code Reviewer** — applying read-only template
Recommended strategy: B (agent + skill)
Template hints: Use search+read tools, structure output as report
```

#### Phase 3: ACT (generate-agent)
Generates `.github/agents/security-pr-reviewer.agent.md`:

```markdown
---
name: Security PR Reviewer
description: >
  Analyzes pull requests for security vulnerabilities and code quality issues.
  Identifies SQL injection, XSS, unsafe dependencies, cryptography mistakes, and
  authentication flaws without modifying code. USE FOR: review pull request security,
  analyze code for vulnerabilities, security audit, threat detection, code vulnerability
  scan, detect SQL injection, find XSS risks, check dependencies.
  DO NOT USE FOR: fixing vulnerabilities, modifying code, deployment approval.
argument-hint: Pull request code snippet or file path to analyze
target: vscode
tools: [search, read, vscode/askQuestions, web]
agents: []
---

You are a **SECURITY PR REVIEWER** — a focused security analyst for pull requests.

Your job: examine code changes for security vulnerabilities and report findings
without modifying anything.

<rules>
- NEVER suggest or write code fixes — only identify vulnerabilities and explain risks
- Use specific OWASP references (A01:2021, A03:2021, etc.) when classifying risks
- Ask for clarification if the vulnerability context is ambiguous
- Always explain the threat model (who could exploit this? what's the impact?)
- Defer to domain experts for infrastructure/deployment-level security
</rules>

<capabilities>
You can help with:
- **SQL Injection detection**: Finds unparameterized queries and string concatenation risks
- **XSS vulnerability analysis**: Identifies unescaped output and unsafe DOM operations
- **Dependency scanning**: Flags known CVEs and unmaintained libraries
- **Cryptography review**: Detects weak algorithms, improper key management, hardcoded secrets
- **Authentication flaws**: Finds missing auth checks, session management issues, authorization gaps
- **Threat modeling**: Explains the business impact of discovered vulnerabilities
</capabilities>

<workflow>
## 1. Understand Scope
Load the PR or code snippet and ask the user to confirm the scope of review.
If the code spans multiple unrelated systems, suggest breaking it into smaller reviews.

## 2. Static Analysis
- Search for common vulnerability patterns: `WHERE ` + `+`, `eval(`, `innerHTML =`, `crypto.randomBytes` without proper seeding
- Read key files to map data flow from input to output
- Identify all external inputs and sensitive operations

## 3. Classify Vulnerabilities
For each issue found:
- Rank by CVSS severity (Critical / High / Medium / Low)
- Reference the OWASP Top 10 category
- Explain attack scenario and business impact

## 4. Report Findings
Structure the report as:
- Executive summary (1–2 sentences on overall risk level)
- Critical findings (must fix before merge)
- High-priority findings (should fix)
- Medium/low findings (consider fixing)
- Recommendations for security practices
</workflow>
```

#### Phase 4: OBSERVE (evaluate-artifact)
```
═══════════════════════════════════════
 ARTIFACT QUALITY REPORT
═══════════════════════════════════════
File: .github/agents/security-pr-reviewer.agent.md
Type: agent
Domain: security

SCORES
  Completeness : 0.95 / 1.00  [All sections present, covers full request]
  Correctness  : 0.98 / 1.00  [All tools valid, no hallucinations detected]
  Clarity      : 0.92 / 1.00  [Clear rules, concrete capabilities, workflow has good phase structure]
  ─────────────────────────────────
  Overall      : 9.2 / 10.0

VERDICT: ✅ PASS (score ≥ 9.0)

ISSUES FOUND: 1 (minor)
  [LOW] Clarity: Workflow step 3 could name the report format more explicitly
        (suggested fix: "Structure the report as Markdown with sections for Critical/High/Medium")

════════════════════════════════════════
```

**Decision**: Score ≥ 9.0 → Finalize, no iteration needed.

#### Phase 5: Finalization

✅ Generated: agent "security-pr-reviewer"
✅ Saved to: `.github/agents/security-pr-reviewer.agent.md`
✅ Stored in artifact memory

Quality scores: **Completeness 9.5** | **Correctness 9.8** | **Clarity 9.2** | **Overall 9.2/10**
Strategy used: **B — Agent + skills** (identified multi-phase security review work)

**Try it with:**
```
/security-pr-reviewer Analyze this code for SQL injection risks: SELECT * FROM users WHERE id = $(userId)
```

**Suggested next artifacts:**
- `security-threat-modeler` skill — breaks down threat models for discovered vulnerabilities
- `dependency-checker` agent — specialized scanner for supply chain risks (npm/pip/cargo packages)

---

## Example 2: Generate a Test Generation Skill

### Input
```
/Generate AI Artifact I need a skill that automatically generates unit tests for Python functions. 
Given a function signature, it should analyze the function logic, 
identify edge cases, generate pytest-compatible test cases, 
and ensure coverage of all branches.
```

### Pipeline Execution

#### Phase 1: PERCEIVE
```
ARTIFACT TYPE     : skill
TYPE CONFIDENCE   : 0.97
DOMAIN            : testing
DOMAIN CONFIDENCE : 0.94
COMPLEXITY        : MEDIUM
STRATEGY          : B — Multi-phase workflow
AMBIGUOUS         : NO
```

#### Phase 2: MEMORY
```
Pattern recognized: **Test Writer** — applying code+validate template
Template hints: explore+edit+testFailure tools, structure: discover → generate → validate
```

#### Phase 3: ACT (generate-skill)
Generates `.github/skills/generate-unit-tests/SKILL.md`:

```markdown
---
name: generate-unit-tests
description: >
  Generates comprehensive pytest-compatible unit tests for Python functions.
  Analyzes function signatures, identifies edge cases, branches, and error paths,
  then produces test cases covering all scenarios. USE FOR: generate tests, create
  unit tests, test generation, write test cases, test coverage, generate pytest,
  test suite creation.
argument-hint: Python function signature or file path containing the function
disable-model-invocation: true
---

# Generate Unit Tests

## Purpose

Automates unit test generation for Python functions by analyzing control flow,
identifying edge cases, and producing pytest-compatible test cases with assertions.
Use this skill when you need comprehensive test coverage but want to avoid manual
test writing.

## Prerequisites
- **Input**: Python function (as code snippet or file path)
- **Access**: Python environment with ast and pytest modules available
- **State**: Function should have clear inputs and measurable outputs

## Workflow

### Step 1: Analyze Function Signature
Read the function and extract:
- Parameter types and constraints
- Return type and possible values
- Docstring or comments documenting expected behavior

Identify special cases:
- Optional/default parameters
- Variadic arguments (*args, **kwargs)
- Type hints (if present)

### Step 2: Identify Test Scenarios
For each parameter and control flow path, generate test cases:

| Parameter Type | Scenarios |
|---|---|
| int | min, max, zero, negative, positive, boundary values |
| str | empty, whitespace, unicode, very long, special chars |
| list/dict | empty, single item, many items, duplicates, None values |
| bool | True, False |
| Exceptions | each exception path; unhappy paths |

Decision point:
- If function has N branches → generate ≥ N test cases
- If function has loops → generate cases for loop iterations (0, 1, many)
- If function has recursion → test base case + recursive case

### Step 3: Generate Test Cases
Write pytest-compatible test functions:
```python
def test_<function>_<scenario>():
    # Arrange
    <setup inputs and mocks>
    
    # Act
    result = <function>(<inputs>)
    
    # Assert
    assert <condition>
```

Each test should:
- Have a descriptive name ending with the scenario
- Include Arrange → Act → Assert structure
- Test one behavior per test function
- Include assertions for both expected value AND edge cases

### Step 4: Validate Generated Tests
- Syntax check: ensure all tests are valid Python
- Coverage check: verify each function path is tested
- Execution check: run pytest and ensure all tests pass

Decision point:
- If any test fails → fix the test or document the limitation
- If coverage < 80% → add more test cases to undercovered branches

### Step 5: Organize and Save
Create a test file following project conventions:
- Name: `test_<original_module>.py` (for .github/... location) or `tests/test_<function>.py`
- Group related tests with classes if ≥5 tests
- Include docstrings for non-obvious assertions

## Quality Criteria
- [ ] All function branches have at least one test
- [ ] Edge cases identified in Step 2 have corresponding tests
- [ ] Each test follows Arrange → Act → Assert pattern
- [ ] All generated tests pass when run with `pytest`
- [ ] Test names are descriptive and include the scenario
- [ ] Generated code follows the project's Python style (PEP 8 or project conventions)

## Output
Generates `test_<function>.py` (or project test directory structure)
containing ≥5 pytest test functions with ≥80% code coverage of the original function.
```

#### Phase 4: OBSERVE
```
═══════════════════════════════════════
 ARTIFACT QUALITY REPORT
═══════════════════════════════════════
File: .github/skills/generate-unit-tests/SKILL.md
Type: skill
Domain: testing

SCORES
  Completeness : 0.98 / 1.00  [All workflow phases present, clear prerequisites, good criteria]
  Correctness  : 0.96 / 1.00  [Valid tool references, no structural issues]
  Clarity      : 0.94 / 1.00  [Concrete step instructions, good decision table]
  ─────────────────────────────────
  Overall      : 9.6 / 10.0

VERDICT: ✅ PASS
════════════════════════════════════════
```

#### Phase 5: Finalization

✅ Generated: skill "generate-unit-tests"
✅ Saved to: `.github/skills/generate-unit-tests/SKILL.md`

Try it with:
```
/generate-unit-tests def calculate_discount(price, quantity, is_member):
    if price < 0: raise ValueError("Price cannot be negative")
    discount = 0.1 if is_member else 0
    if quantity > 10: discount += 0.05
    return price * quantity * (1 - discount)
```

---

## Example 3: Generate a Multi-Agent Orchestrator (Strategy C)

### Input
```
/Generate AI Artifact Create an agent that orchestrates the entire code review process.
It should coordinate with specialized subagents: one for security review, 
one for performance analysis, and one for code style. 
It collects reports from all three and synthesizes a final recommendation.
```

### Pipeline Execution

#### Phase 1: PERCEIVE
```
ARTIFACT TYPE     : agent
TYPE CONFIDENCE   : 0.99
DOMAIN            : code-review
DOMAIN CONFIDENCE : 0.98
COMPLEXITY        : HIGH
STRATEGY          : C — Multi-agent orchestrator
AMBIGUOUS         : NO
```

#### Phase 2: MEMORY
```
Pattern recognized: **Pipeline Orchestrator** — requires subagent coordination
Recommended strategy: C (root coordinator + specialized subagents)
```

#### Phase 3: ACT (generate-agent with Strategy C)

Generates `.github/agents/code-review-orchestrator.agent.md`:

```markdown
---
name: Code Review Orchestrator
description: >
  Orchestrates a comprehensive code review across three dimensions: security, performance,
  and style. Coordinates parallel subagent reviews, synthesizes findings into a unified
  report with prioritized recommendations. USE FOR: code review, comprehensive review,
  orchestrate review process, multi-dimension review, synthesize feedback, code quality,
  security + performance + style review.
argument-hint: Code or file path to review
target: vscode
tools: [search, read, vscode/askQuestions, vscode/memory, agent]
agents: [security-reviewer, performance-analyzer, code-style-checker]
---

You are a **CODE REVIEW ORCHESTRATOR** — conductor of a multi-specialist review.

Your job: coordinate parallel code reviews from three subagents, then synthesize
their findings into one unified recommendation.

<rules>
- NEVER review code yourself — only orchestrate and synthesize subagent outputs
- Collect all three subagent reports before synthesizing
- If two subagents disagree, explain both perspectives in the final report
- Prioritize findings by severity (critical security bugs > performance issues > style)
- Always include a "ship it?" decision with reasoning
</rules>

<capabilities>
You can help with:
- **Orchestration**: Coordinate parallel reviews from specialized subagents
- **Synthesis**: Combine findings into a coherent narrative and recommendation
- **Prioritization**: Rank issues by severity and actionability
- **Trade-off analysis**: Explain security vs. performance vs. style trade-offs
- **Decision support**: Recommend "ship now", "ship with caveats", or "request changes"
</capabilities>

<workflow>
## 1. Load Code & Scope
Load the code or file to review. Ask the user to confirm the scope:
- Is this a feature PR, refactor, or hotfix? (affects reviewer expectations)
- Are there any known trade-offs or constraints? (performance budgets, legacy patterns, etc.)

## 2. Dispatch Subagents (Parallel)
Launch three subagent reviews in parallel:
- `#agent:security-reviewer — Check this code for security vulnerabilities`
- `#agent:performance-analyzer — Analyze this code for performance issues`
- `#agent:code-style-checker — Review this code for style and maintainability`

Wait for all three to complete.

## 3. Synthesize Findings
Collect outputs from all three subagents. Organize as:
- **Critical findings**: Security bugs, critical performance issues
- **Major findings**: Non-critical vulnerabilities, optimization opportunities, style inconsistencies
- **Minor findings**: Code comments, minor style suggestions

## 4. Build Recommendation
Synthesize a unified verdict:
- If any critical security issues → "Request changes"
- If major issues exist but no critical bugs → "Request changes with minor caveats"
- If only minor findings → "Approve with suggestions" or "Ship it"

Include reasoning: Why can this be shipped? What should be watched?

## 5. Report
Deliver a unified report:
1. Summary (1–2 sentences on overall quality)
2. Critical findings (if any) with subagent source
3. Major findings by category (security/performance/style)
4. Minor findings (can be deferred to future PRs)
5. Final recommendation with explicit reasoning
</workflow>
```

Plus generated stubs for the three subagents that it coordinates.

---

## Example 4: Generate a Documentation Skill

### Input
```
/Generate AI Artifact Create a skill that generates API documentation from Python docstrings.
It should extract docstring information, standardize the format, 
and generate Markdown docs with examples and parameter tables.
```

### OBSERVE output:
```
SCORES
  Completeness : 0.96 / 1.00
  Correctness  : 0.98 / 1.00
  Clarity      : 0.93 / 1.00
  Overall      : 9.5 / 10.0

VERDICT: ✅ PASS
```

---

## Quality Scoring Explained

The system scores artifacts on three axes (each 0.0–1.0):

### 1. Completeness (40% weight)
- Does the artifact address all requirements from the input?
- Are all required frontmatter fields present?
- For agents: rules, capabilities, workflow phases all present?
- For skills: prerequisites, workflow steps, quality criteria all present?

**Good completeness**: ≥0.9 usually means no missing sections or scenarios.

### 2. Correctness (40% weight)
- Are all tool names valid?
- Do workflow steps reference only declared tools?
- Are there any hallucinated agents, skills, or file paths?
- Is the frontmatter valid YAML?

**Good correctness**: ≥0.95 means no invalid references or made-up capabilities.

### 3. Clarity (20% weight)
- Are instructions concrete or vague?
- Does the description have "USE FOR" trigger phrases?
- Are workflow steps action-oriented ("do X") vs. wishy-washy ("consider X")?
- Is there placeholder text left (`<...>`, `TODO`)?

**Good clarity**: ≥0.9 means a user can pick it up and use it without confusion.

**Overall Score**: `(Completeness × 0.4 + Correctness × 0.4 + Clarity × 0.2) × 10`
- **≥9.0**: Ship immediately
- **7.0–8.9**: Fix issues and re-evaluate once
- **<7.0**: Regenerate with different strategy

---

## When to Use Each Artifact Type

| You want to… | Artifact Type |
|---|---|
| Create an assistant with a persistent role (reviewer, planner, debugger) | **Agent** (`.agent.md`) |
| Automate a repeatable multi-step workflow (generate tests, deploy, review) | **Skill** (`SKILL.md`) |
| Define a single parameterized task (translate to Rust, explain this error) | **Prompt** (`.prompt.md`) |

---

## Memory & Pattern Recognition

The generator remembers past artifacts and recognizes 8 common patterns:

1. **Code Reviewer** — Read-only agents that analyze code (search+read tools, structured reports)
2. **Code Generator** — Write code agents (edit tools, exploration-first workflow)
3. **Test Writer** — Generate tests (explore+edit+validate, scenario identification)
4. **Documentation Writer** — Produce docs (read-heavy, markdown output)
5. **Pipeline Orchestrator** — Coordinate subagents (Strategy C, parallel subagents)
6. **Domain Expert** — Deep specialization (security, data, DevOps)
7. **Research Assistant** — Find information (read-only, web+search+memory)
8. **Deployment Automator** — Execute infrastructure changes (terminal tools, confirmation gates)

When a pattern is recognized, the generator applies a known-good template, often requiring fewer iterations to reach the 9.0+ threshold.

---

## Troubleshooting

### "Score keeps dropping below 7.0"
- The problem description might be ambiguous. Try using `/Generate AI Artifact` with more specific details about what the agent/skill should and should NOT do.
- Try explicitly stating the primary responsibility (e.g., "This should ONLY review security, not style").

### "Generated agent references a tool that doesn't exist"
- This is a hallucination. The evaluate-artifact skill should catch it, but if it slips through, manually remove the tool name from the `tools:` list.

### "I want to reuse a skill from this session"
- The generator checks memory and will tell you if a similar artifact was generated recently. You can reuse it or ask to generate a variation.

### "The workflow doesn't fit my specific process"
- Skills and agents are templates. Once generated, you can edit them. The generator creates a working baseline; customize for your team's exact workflow.

---

## Integration with Your Workflow

### Save and Share
All generated artifacts go to:
- **Workspace-scoped** (default): `.github/agents/` and `.github/skills/<name>/`
- **User-level** (personal): `~/.config/Code/User/prompts/` (roams with your settings)

Workspace artifacts can be committed to git and shared with your team.

### CI/CD Integration
The generated agents and skills work out of the box in VS Code chat. To integrate with CI/CD:
1. Export the artifact path to your build system
2. Use the agent/skill name in your automation scripts
3. Most agents work with GitHub Actions, GitLab CI, Jenkins, etc. when invoked via the chat API

### Iteration
Generated artifacts are a starting point. Common customizations:
- Adjust tool lists based on your team's preferences
- Refine workflow phases to match your process
- Add domain-specific rules for your codebase conventions
- Extend capabilities based on post-generation feedback

---

## Next Steps

1. **Try it**: `/Generate AI Artifact create an agent that explains error messages from my Python code`
2. **Check memory**: After generation, the system stores the artifact for pattern matching in future requests
3. **Customize**: Edit the generated `.agent.md` or `SKILL.md` file to refine it
4. **Suggest improvements**: If an agent/skill doesn't quite work, regenerate with additional constraints
