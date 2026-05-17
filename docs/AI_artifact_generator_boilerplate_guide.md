# AI Artifact Generator — Boilerplate Integration Guide

Use the AI Artifact Generator as a foundation for any VS Code-based project or team. This guide covers setup, customization, extension, and team adoption patterns.

---

## Table of Contents

1. [Quick Setup](#quick-setup)
2. [Customizing for Your Project](#customizing-for-your-project)
3. [Domain-Specific Extensions](#domain-specific-extensions)
4. [Managing Your Artifact Library](#managing-your-artifact-library)
5. [Team Adoption](#team-adoption)
6. [Integration with Existing Workflows](#integration-with-existing-workflows)
7. [Extending Pattern Recognition](#extending-pattern-recognition)
8. [Maintenance & Versioning](#maintenance--versioning)

---

## Quick Setup

### 1. Copy the Boilerplate

```bash
# Copy the Agent-Customization folder into your project
cp -r /path/to/boilerplate/Agent-Customization /your-project/.github/

# Verify structure
ls -la /your-project/.github/
# agents/
# skills/
# instructions/
# mcp-servers/
# plugins/
# prompts/
```

### 2. Verify Core Components

Ensure these files exist:

```
.github/agents/
  ├── ask.agent.md                    # Read-only query agent (keep as-is)
  ├── explore.agent.md                # Exploration subagent (keep as-is)
  ├── plan.agent.md                   # Planning agent (keep as-is)
  └── generate-ai-artifact.agent.md   # Main orchestrator (customization point)

.github/skills/
  ├── analyze-request.skill.md        # Input classification (core)
  ├── artifact-memory.skill.md        # Memory layer (core)
  ├── evaluate-artifact.skill.md      # Quality gating (core)
  ├── generate-agent.skill.md         # Agent template builder (core)
  ├── generate-skill.skill.md         # Skill template builder (core)
  └── [project-specific skills]       # Add your own here
```

### 3. Create a Project-Level Instructions File

Create `.github/AGENTS.md` to document your team's conventions:

```markdown
# AI Agents for [Your Project]

## Project Context

- **Tech stack**: <your stack>
- **Build command**: <your build command>
- **Test command**: <your test command>
- **Key directories**: src/, tests/, docs/

## Agent Usage

### `/Generate AI Artifact`
Generates new agents and skills for this project. Use when:
- Need a new specialized agent for a workflow
- Want to automate a repeatable process
- Extend the codebase with AI-assisted features

### `/Ask`
Ask questions about the codebase without modifying anything.

### `/Plan`
Create detailed plans before implementation.

## Project-Specific Agents

[List any custom agents you've generated for this project]

## Patterns & Conventions

- Code style: <link to CONTRIBUTING.md or style guide>
- Architecture: <link to ARCHITECTURE.md or design docs>
- Testing approach: <your testing philosophy>
```

---

## Customizing for Your Project

### 1. Update the Orchestrator Agent

Edit `.github/agents/generate-ai-artifact.agent.md` to add project-specific context:

```markdown
---
name: Generate AI Artifact
description: >
  [Keep existing description]
target: vscode
tools: [search, read, edit, vscode/memory, vscode/askQuestions, agent]
agents: [Explore]
---

You are the **AI Artifact Generator** for [Your Project Name].

Your job: transform a problem description into a production-quality agent or skill
tailored to [Your Project]'s tech stack and conventions.

<rules>
- [Keep existing rules]
- Generate artifacts that follow [Your Project] naming conventions
- Ensure all generated code respects [Your Project] architecture
- Always reference [Your Project]'s style guide and patterns
- For [Your Domain], add domain-specific concerns (e.g., "Always include security checks for web agents")
</rules>

[Rest of the file remains the same]
```

### 2. Customize Pattern Recognition

Add project-specific patterns to `analyze-request.skill.md`:

```markdown
## Step 2: Domain Classification

[Keep existing domains, then add:]

| Domain | Keywords |
|--------|---------|
| `<your-domain-1>` | keyword1, keyword2, keyword3 |
| `<your-domain-2>` | keyword1, keyword2, keyword3 |
```

### 3. Customize Tool Lists

Update `generate-agent.skill.md` to reflect your team's available tools:

```markdown
## Step 2: Select Tools Based on Domain

[Keep existing table, customize the "Optional" column based on:]
- Your project's MCP servers
- Custom tools available to your team
- Team preferences (e.g., "we prefer LSP over CLI")
```

---

## Domain-Specific Extensions

### Example: Web Development Project

Create a specialized agent for your stack. Start with the boilerplate, then customize:

#### 1. Create a Web-Specific Agent Generator

Save as `.github/agents/web-agent-generator.agent.md`:

```markdown
---
name: Web Agent Generator
description: >
  Specialized agent generator for web development. Generates web-specific agents
  (API handlers, frontend components, middleware, auth guards) with framework-aware
  best practices. USE FOR: generate web agent, create api handler, build frontend agent,
  middleware generator, auth agent.
argument-hint: Describe the web component or handler to automate
target: vscode
tools: [search, read, edit, web, vscode/askQuestions, agent]
agents: [Explore]
---

You are a **WEB AGENT GENERATOR** specialized in [Your Framework: React/Vue/Angular/FastAPI/Django].

Your job: generate web-specific agents that follow [Your Framework] patterns and [Your Project]'s architecture.

<rules>
- Always reference [Your Project]'s component library and design patterns
- Generated agents must integrate with [Your API framework] conventions
- Include security best practices: CSRF tokens, input validation, XSS prevention
- Follow [Your Framework]'s performance recommendations
- Defer to the main Generate AI Artifact for non-web concerns
</rules>

<capabilities>
You can help with:
- **API handler agents**: Generate agents that build REST/GraphQL endpoints
- **Frontend component agents**: Create component generators for your UI framework
- **Middleware agents**: Generate auth, logging, rate-limiting middleware
- **Security agents**: Generate agents that implement auth flows and access control
- **Testing agents**: Generate integration test and E2E test agents
</capabilities>

<workflow>
## 1. Understand Web Scope
Determine which layer (API, frontend, middleware).

## 2. Analyze Framework Patterns
Search the codebase for similar components to match style.

## 3. Generate Web-Specific Agent
Use framework idioms and [Your Project]'s patterns.

## 4. Validate Against Conventions
Check the generated agent against [Your Project]'s style guide.
</workflow>
```

#### 2. Create Web-Specific Skills

Save as `.github/skills/generate-web-component/SKILL.md`:

```markdown
---
name: generate-web-component
description: >
  Generates web components following [Your Framework] best practices.
  USE FOR: create component, generate web component, scaffold UI component,
  build form component, create layout component.
argument-hint: Component name and expected behavior
disable-model-invocation: true
---

# Generate Web Component

## Purpose

Scaffolds [Your Framework] components that integrate with your design system,
state management, and testing infrastructure.

## Prerequisites
- **Context**: Design system tokens and component library location
- **Access**: Component examples in src/components/
- **State**: Storybook running (if applicable)

## Workflow

### Step 1: Determine Component Type
- Container / Presentational?
- Form / Display?
- Custom / Using design system?

### Step 2: Generate Component File
Use your framework's component structure:
- [Framework-specific boilerplate]
- Props interface
- Default export

### Step 3: Add Tests
Generate unit test file matching [Your Testing Framework]:
- Props validation tests
- Behavior tests
- A11y tests (if applicable)

### Step 4: Add to Storybook (if applicable)
Generate a `.stories.tsx` file with:
- Basic story
- Interactive variant with controls
- Accessibility annotations

## Quality Criteria
- [ ] Component follows [Your Project]'s file structure
- [ ] Exports match your barrel file conventions
- [ ] Props are typed (TypeScript) or documented
- [ ] Tests exist and pass
- [ ] Storybook story is created and renders

## Output
New component file at `src/components/<ComponentName>/<ComponentName>.{tsx|jsx}`
with accompanying test and story files.
```

### Example: Backend/Data Project

Create a data-specific agent generator at `.github/agents/data-agent-generator.agent.md`:

```markdown
---
name: Data Agent Generator
description: >
  Specialized agent generator for data engineering. Generates data pipeline agents,
  SQL analyzers, ETL orchestrators, and data quality checkers. USE FOR: generate data
  agent, create pipeline agent, sql analyzer, etl orchestrator, data quality agent.
argument-hint: Describe the data workflow or pipeline
target: vscode
tools: [search, read, edit, vscode/askQuestions, agent]
agents: [Explore]
---

You are a **DATA AGENT GENERATOR** for [Your Data Stack: dbt/Airflow/Spark/Pandas].

<rules>
- Generated agents must respect [Your Data Warehouse] schema conventions
- Include data lineage and impact analysis
- Always include data quality checks
- Follow [Your Data Stack] best practices for performance
</rules>

[Continue similar to web example]
```

---

## Managing Your Artifact Library

### 1. Organize Generated Artifacts

Create a catalog in `.github/ARTIFACT_LIBRARY.md`:

```markdown
# Artifact Library

## Generated Agents

### Code Review
- `security-pr-reviewer` — Security vulnerability analysis
- `performance-analyzer` — Performance bottleneck detection
- `code-style-checker` — Code quality and style compliance

### Development
- `test-generator` — Unit test scaffolding
- `component-generator` — Web component generation
- `doc-writer` — Documentation generation

## Generated Skills

### Testing
- `generate-unit-tests` — Pytest test generation
- `generate-integration-tests` — Multi-module test generation

### Documentation
- `generate-api-docs` — API documentation from docstrings

## Template History

| Date | Artifact | Strategy | Score | Notes |
|------|----------|----------|-------|-------|
| 2026-05-15 | security-pr-reviewer | B | 9.2 | High-quality security template |
```

### 2. Version Control

Commit generated artifacts to git:

```bash
# .gitignore — exclude only generated outputs, not the artifacts themselves
node_modules/
dist/
build/
.DS_Store

# But INCLUDE these:
!.github/agents/
!.github/skills/
!.github/ARTIFACT_LIBRARY.md
```

### 3. Create a Review Checklist

Before committing new artifacts, `.github/ARTIFACT_CHECKLIST.md`:

```markdown
# Artifact Review Checklist

Before merging a new agent or skill:

## Frontmatter
- [ ] `name` is descriptive and kebab-case
- [ ] `description` has USE FOR triggers
- [ ] `tools` list is minimal (not "just in case")
- [ ] All tool names are valid

## Content
- [ ] No `<placeholder>` text remains
- [ ] Workflow has 2+ phases with concrete steps
- [ ] Rules say DO and DON'T (not just one)
- [ ] Capabilities are specific, not generic

## Quality
- [ ] Generated by artifact generator and scored ≥ 9.0
- [ ] Manually reviewed by team lead
- [ ] Tested with real use case
- [ ] Documented in ARTIFACT_LIBRARY.md

## Integration
- [ ] Follows project naming conventions
- [ ] References existing agents/skills correctly
- [ ] All generated code snippets match your stack
```

---

## Team Adoption

### 1. Kick-Off: Introduce the System

Share this slide deck outline with your team:

```
1. Problem: How do we automate routine AI assistance?
2. Solution: AI Artifact Generator (PERCEIVE → PLAN → ACT → OBSERVE → DECIDE)
3. What it generates: Agents and skills
4. When to use it: [Show 3 use cases from your domain]
5. How to invoke: /Generate AI Artifact [problem description]
6. Quality gates: 3-axis scoring, 9.0/10 threshold
7. Team process: Generate → Review → Merge → Document
```

### 2. Training Workshop

Run a 60-minute workshop:

**Part 1: Concepts (15 min)**
- Show the pipeline architecture
- Demo a quick example (/Generate AI Artifact create a code reviewer)
- Explain quality gates

**Part 2: Hands-on (30 min)**
- Pair programming: Generate an artifact together
- Walk through quality evaluation
- Show how to edit and customize

**Part 3: Team Workflow (15 min)**
- Artifact library and catalog
- Code review checklist
- When to generate vs. manually write

### 3. Onboarding Template

Create `GETTING_STARTED.md` for new team members:

```markdown
# Getting Started with AI Agents

## What You Can Do

Use `/Generate AI Artifact` to create agents and skills that automate your work:

### Examples
- Generate a code review agent: `/Generate AI Artifact create an agent that reviews pull requests for security bugs`
- Generate a test generator: `/Generate AI Artifact create a skill that generates unit tests for Python functions`
- Generate a documentation writer: `/Generate AI Artifact create an agent that writes API documentation`

## Your First Artifact

1. Open VS Code
2. Open the chat (Ctrl+L or Cmd+L)
3. Type: `/Generate AI Artifact [your problem]`
4. Watch the pipeline run (PERCEIVE → ACT → OBSERVE)
5. The artifact is saved to `.github/agents/` or `.github/skills/`

## If It Doesn't Work

- Check that you're in [Your Project] workspace
- Ensure `.github/agents/` and `.github/skills/` exist
- Run: `cat .github/agents/ask.agent.md` to verify setup

## Next Steps

- Read [ARTIFACT_LIBRARY.md](.github/ARTIFACT_LIBRARY.md) to see what's available
- Review [Agent-Customization Guide](.github/Agent-Customization/README.md)
- Generate your first artifact and share with the team
```

---

## Integration with Existing Workflows

### 1. GitHub Actions

Trigger artifact generation from PRs:

Create `.github/workflows/artifact-generation.yml`:

```yaml
name: Generate AI Artifacts

on:
  workflow_dispatch:
    inputs:
      description:
        description: 'Problem description for artifact generation'
        required: true

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Generate artifact
        run: |
          # Invoke the generator via CLI (requires API integration)
          # For now, this is manual — PR with new artifact + evaluation
          echo "Artifact request: ${{ github.event.inputs.description }}"
          echo "Generate in VS Code, then submit PR"
```

### 2. Code Review Integration

Add a comment to your PR template (`.github/pull_request_template.md`):

```markdown
## AI-Assisted Review

Need an AI agent to review this PR?

```
/security-pr-reviewer [mention what to focus on]
/performance-analyzer [if performance-sensitive]
```

Available agents: [Link to ARTIFACT_LIBRARY.md]
```

### 3. Documentation Generation

Use a documentation skill in your build:

```bash
# In your Makefile or build script
docs:
	# Generate API docs using the skill
	code --command "vscode.executeCommand('agent.invoke', 'generate-api-docs')"
```

### 4. Pre-Commit Hooks

Validate generated artifacts before commit (`.githooks/pre-commit`):

```bash
#!/bin/bash
# Verify all .agent.md and SKILL.md files have required frontmatter

for file in $(git diff --cached --name-only | grep -E '\.agent\.md|SKILL\.md'); do
  if ! grep -q "^name:" "$file"; then
    echo "Error: $file missing 'name:' in frontmatter"
    exit 1
  fi
  if ! grep -q "^description:" "$file"; then
    echo "Error: $file missing 'description:' in frontmatter"
    exit 1
  fi
done
exit 0
```

Configure in your repo:

```bash
git config core.hooksPath .githooks
```

---

## Extending Pattern Recognition

### 1. Add Custom Patterns

Update `.github/skills/artifact-memory.skill.md` with your own patterns:

```markdown
## Pattern Recognition

Add these to the pattern table:

| Pattern Type | Signals | Recommended Template |
|---|---|---|
| **Your Pattern 1** | keywords that trigger it | Template to use |
| **Your Pattern 2** | keywords | Template |
```

### 2. Create Domain-Specific Memory Files

Store domain-specific knowledge in `/memories/`:

Create `/memories/repo/web-patterns.md`:

```markdown
# Web Development Patterns

## High-Quality Agent Templates
- API handler agents: Use REST-first pattern with error handling
- Frontend agents: Always include props validation and tests
- Middleware agents: Include logging and timing instrumentation

## Tools We Always Use
- For frontend: edit, search, web
- For backend: edit, search, execute/getTerminalOutput

## Performance Baselines
- Agent generation: < 10 seconds expected
- Skill generation: < 15 seconds expected
- Quality evaluation: < 5 seconds expected
```

### 3. Create Reusable Skill Templates

Save common workflows as templates in `.github/skills/templates/`:

```
.github/skills/templates/
  ├── api-handler-template.md
  ├── test-generator-template.md
  ├── doc-writer-template.md
  └── code-analyzer-template.md
```

Then reference in `generate-agent.skill.md`:

```markdown
## Recognized Templates

If the domain is API development → use api-handler-template
If the domain is testing → use test-generator-template
```

---

## Maintenance & Versioning

### 1. Quarterly Artifact Review

Create a process to review generated artifacts:

```markdown
# Q2 2026 Artifact Review

Date: May 15, 2026
Reviewers: [Team members]

## Artifacts to Review
- [ ] `security-pr-reviewer` — Still solving the right problem?
- [ ] `test-generator` — Still accurate for current test suite?
- [ ] `documentation-writer` — Still matches current doc style?

## Template Updates Needed
- [ ] Update `generate-agent.skill.md` for new tool availability
- [ ] Update `artifact-memory.skill.md` for new patterns recognized
- [ ] Update pattern weights based on success rates

## New Artifacts to Generate
- [ ] Data pipeline orchestrator (new domain)
- [ ] Mobile app component generator (expanding scope)
```

### 2. Version Control Artifacts

Add version tags to high-value artifacts:

```markdown
---
name: Security PR Reviewer
version: 1.0.0
created: 2026-05-15
updated: 2026-05-15
quality-score: 9.2
---
```

### 3. Track Success Metrics

Store in `ARTIFACT_METRICS.md`:

```markdown
# Artifact Success Metrics

## Usage Tracking

| Artifact | Invocations (this month) | Avg Satisfaction | Issues Reported |
|----------|--------------------------|------------------|-----------------|
| security-pr-reviewer | 24 | 4.5/5 | 1 (tool name error) |
| test-generator | 18 | 4.8/5 | 0 |
| doc-writer | 12 | 4.2/5 | 2 (formatting) |

## Improvement Backlog
1. Update `security-pr-reviewer` to include OWASP A07:2021
2. Extend `test-generator` to support async/await patterns
3. Improve `doc-writer` for API endpoint documentation
```

### 4. Deprecation Policy

Mark outdated artifacts:

```markdown
---
name: Old API Handler
status: DEPRECATED
successor: api-handler-v2
deprecated-date: 2026-05-15
removal-date: 2026-08-15
migration-guide: See MIGRATION.md#api-handler
---
```

---

## Example: Onboarding a New Project Type

### Scenario: Your Team Wants to Add Mobile App Development

**Step 1: Assess Needs**
```
What problems do mobile developers face?
- Component boilerplate
- Navigation setup
- State management plumbing
- Testing setup
```

**Step 2: Create Custom Domain**

Update `analyze-request.skill.md`:
```markdown
| Domain | Keywords |
|--------|---------|
| `mobile-development` | React Native, Flutter, Swift, Kotlin, component, screen, navigation |
```

**Step 3: Create Specialized Agent**

Generate `.github/agents/mobile-agent-generator.agent.md` for mobile-specific concerns.

**Step 4: Create Supporting Skills**

Generate skills:
- `.github/skills/generate-mobile-component/SKILL.md`
- `.github/skills/setup-navigation/SKILL.md`
- `.github/skills/generate-mobile-test/SKILL.md`

**Step 5: Document and Train**

Add to `ARTIFACT_LIBRARY.md`:
```markdown
## Mobile Development

- `mobile-agent-generator` — Creates mobile-specific agents
- `generate-mobile-component` — Scaffolds React Native components
- `setup-navigation` — Configures React Navigation
```

Train the team on when to use each new artifact.

**Step 6: Monitor and Iterate**

Track artifact usage for 1 month, then refine based on feedback.

---

## Troubleshooting Setup Issues

### "The orchestrator agent isn't found"

Check:
```bash
# Verify file exists
test -f .github/agents/generate-ai-artifact.agent.md && echo "✓ Found"

# Verify frontmatter
head -5 .github/agents/generate-ai-artifact.agent.md | grep "name:"

# Check VS Code can access it
code --list-extensions | grep copilot
```

### "Skills aren't being discovered"

Ensure the skill `description` has clear trigger phrases:

```markdown
description: >
  What this skill does.
  USE FOR: trigger1, trigger2, trigger3.
```

### "Generated agents reference non-existent tools"

This is a hallucination. Validate in `.github/skills/evaluate-artifact.skill.md`:

```markdown
### Valid tool names (reference list):
search, read, edit, web, vscode/memory, vscode/askQuestions, ...
```

Add any new tools available in your setup.

---

## Next Steps

1. **Copy the boilerplate** to your project
2. **Customize** `.github/agents/generate-ai-artifact.agent.md` with your context
3. **Create** `.github/AGENTS.md` documenting your conventions
4. **Run** `/Generate AI Artifact` to generate your first domain-specific agent
5. **Document** results in `.github/ARTIFACT_LIBRARY.md`
6. **Train** your team on usage
7. **Monitor** metrics and iterate quarterly

---

## Related Resources

- [AI Artifact Generator — Usage Guide](AI_artifact_generator_guide.md)
- [Agent Customization — Full Documentation](../Agent-Customization/skills/agent-customization.skill.md)
- [Sparrow Pseudocode Agent — Inspiration](https://github.com/the-genemachine/sparrow-pseudocode-agent)
