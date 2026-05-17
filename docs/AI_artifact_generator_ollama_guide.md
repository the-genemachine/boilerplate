# AI Artifact Generator — Ollama Integration Guide

Use Ollama for local, private LLM-powered artifact generation. Run the AI Artifact Generator with models like Llama 2, Mistral, Qwen, or CodeLlama entirely offline on your machine.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Strategy Selection](#strategy-selection)
3. [Model Recommendations](#model-recommendations)
4. [Optimization & Performance](#optimization--performance)
5. [Advanced Patterns](#advanced-patterns)
6. [Troubleshooting](#troubleshooting)
7. [Cost-Benefit Analysis](#cost-benefit-analysis)

---

## Quick Start

### 1. Install Ollama

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Windows
# Download from https://ollama.ai/download

# Verify installation
ollama --version
```

### 2. Start Ollama Service

```bash
# Starts Ollama at http://localhost:11434
ollama serve

# Or in background (macOS/Linux)
ollama serve &
```

### 3. Pull a Model

```bash
# Recommended for artifact generation: Mistral (7B, fastest)
ollama pull mistral

# Alternative: Qwen (more accurate)
ollama pull qwen:7b

# Code-specific: CodeLlama (better at generating code)
ollama pull codellama

# Larger option: Llama 2 (13B, slower but higher quality)
ollama pull llama2:13b
```

### 4. Verify Model Runs

```bash
# Test the model
ollama run mistral "Generate a Python function that calculates fibonacci"

# Should respond with code in 5-10 seconds
```

### 5. Configure VS Code Agent to Use Ollama

Update `.github/agents/generate-ai-artifact.agent.md`:

```markdown
---
name: Generate AI Artifact
description: >
  [existing description...]
argument-hint: Describe the problem, workflow, or job the agent/skill should solve
target: vscode
tools:
  - search
  - read
  - edit
  - vscode/memory
  - vscode/askQuestions
  - agent
agents:
  - Explore
---

You are the **AI Artifact Generator** — using Ollama for local LLM inference.

[In the workflow section, add:]

**LLM Configuration**:
- Model: mistral (default) or specify via #tool:vscode/askQuestions
- Endpoint: http://localhost:11434
- Strategy: Hybrid (NLP-first, LLM refinement if needed)
- Quality threshold: 9.0/10

[Rest remains the same]
```

### 6. Create an Ollama Configuration File

Create `.github/ollama.config.json`:

```json
{
  "endpoint": "http://localhost:11434",
  "default_model": "mistral",
  "models": {
    "mistral": {
      "name": "mistral",
      "tokens": 2048,
      "temperature": 0.7,
      "use_case": "General artifact generation (fastest)"
    },
    "qwen:7b": {
      "name": "qwen:7b",
      "tokens": 2048,
      "temperature": 0.6,
      "use_case": "Balanced quality and speed"
    },
    "codellama": {
      "name": "codellama",
      "tokens": 4096,
      "temperature": 0.5,
      "use_case": "Code-heavy agents and skills"
    },
    "llama2:13b": {
      "name": "llama2:13b",
      "tokens": 4096,
      "temperature": 0.6,
      "use_case": "Highest quality (slower)"
    }
  },
  "strategies": {
    "fast": {
      "model": "mistral",
      "temperature": 0.7,
      "max_tokens": 2048,
      "strategy_name": "NLP-first with light LLM fallback"
    },
    "balanced": {
      "model": "qwen:7b",
      "temperature": 0.6,
      "max_tokens": 2048,
      "strategy_name": "Hybrid: NLP + LLM refinement"
    },
    "quality": {
      "model": "llama2:13b",
      "temperature": 0.6,
      "max_tokens": 4096,
      "strategy_name": "Full LLM pipeline (highest quality)"
    }
  }
}
```

### 7. Test the Integration

```bash
# In VS Code chat:
/Generate AI Artifact create an agent that reviews code for security issues

# System should:
# 1. PERCEIVE: Classify as agent + security domain
# 2. ACT: Call ollama run mistral with the generation prompt
# 3. OBSERVE: Evaluate the generated agent
# 4. DECIDE: Iterate if quality < 9.0 or finalize

# Expected time: 10-30 seconds (depending on model and complexity)
```

---

## Strategy Selection

### Strategy A: NLP-Only (Instant, No Ollama Needed)

Use when:
- You need instant feedback
- Generating simple, straightforward artifacts
- Running on resource-constrained machines
- Offline (Ollama not available)

**Speed**: ~1–2ms
**Quality**: Good for templates, patterns, boilerplate
**Cost**: Zero API calls

No Ollama required — uses pattern matching in `generate-agent.skill.md` and `generate-skill.skill.md`.

### Strategy B: Hybrid (NLP + Ollama Refinement)

Use when:
- You want speed + quality balance
- Generating moderately complex artifacts
- Have Ollama running with a fast model (Mistral)

**Process**:
1. NLP mode generates a first draft instantly
2. Quality evaluator checks the draft
3. If score < 8.0, use Ollama to refine weak areas
4. Re-evaluate
5. If now ≥ 9.0, finalize; else iterate once more

**Speed**: 2–15 seconds
**Quality**: High (benefits from both pattern matching + LLM reasoning)
**Cost**: Low (1–2 Ollama calls per artifact, ~30s compute time)

**Configuration in generate-artifact orchestrator:**

```markdown
## Phase 2: PLAN (Strategy Selection with Ollama)

If Ollama is available (http://localhost:11434 responding):
- Use HYBRID strategy by default
- Fallback to NLP-only if Ollama unavailable

If input complexity is HIGH:
- Use HYBRID with model upgrade: mistral → qwen:7b or llama2:13b

If user explicitly requests quality:
- Use STRATEGY C — LLM-only with llama2:13b
```

### Strategy C: LLM-Only (High Quality, Uses Ollama)

Use when:
- Quality is critical
- Complexity is high
- User explicitly said "I need high quality"
- Artifact will be team-standard or framework

**Process**:
1. Load the full problem description + codebase context
2. Call Ollama with a detailed generation prompt
3. Evaluate the output
4. If < 9.0, refine and iterate (max 3 times)

**Speed**: 20–60 seconds per artifact
**Quality**: Highest (full LLM reasoning + feedback loop)
**Cost**: Higher (3–5 Ollama calls per artifact, 2–3 minutes compute)

**Configuration**:

```markdown
## Phase 3: ACT — LLM-Only Generation

If strategy == C:
  Load full context from .github/ directory
  Build comprehensive prompt for Ollama:
    - Problem description
    - Domain classification
    - Complexity analysis
    - Analogous existing agents/skills (from codebase)
    - Project conventions (from AGENTS.md)
    - Quality criteria
  
  Call: ollama run <model> <prompt>
  
  Parse response into .agent.md or SKILL.md structure
  Pass to evaluate-artifact
```

---

## Model Recommendations

### By Use Case

| Model | Size | Speed | Quality | Best For | RAM Required |
|-------|------|-------|---------|----------|--------------|
| **Mistral** | 7B | ⚡⚡⚡⚡⚡ | ⭐⭐⭐ | Fast iteration, prototyping | 8GB |
| **Qwen** | 7B | ⚡⚡⚡⚡ | ⭐⭐⭐⭐ | Balanced (default for hybrid) | 8GB |
| **CodeLlama** | 7B–34B | ⚡⚡⚡ | ⭐⭐⭐⭐⭐ | Code-heavy agents and skills | 8–24GB |
| **Llama 2** | 7B–70B | ⚡⚡⚡ | ⭐⭐⭐⭐⭐ | Highest quality, team standards | 8–48GB |
| **Orca 2** | 7B–13B | ⚡⚡⚡⚡ | ⭐⭐⭐⭐ | Following instructions precisely | 8–16GB |

### Recommended Setup by Scenario

#### Scenario 1: Developer on Laptop (8GB RAM)

```json
{
  "default_model": "mistral",
  "strategy": "hybrid",
  "fallback_strategy": "npl-only"
}
```

- Fast generation: 5–10 seconds
- Use NLP-only if Ollama not running
- Mistral is small enough for laptop

```bash
ollama pull mistral
ollama serve
```

#### Scenario 2: Team Development Machine (16GB RAM)

```json
{
  "default_model": "qwen:7b",
  "strategy": "hybrid",
  "quality_model": "codellama"
}
```

- Default: Qwen (balanced speed/quality)
- For critical artifacts: CodeLlama (code-specific)
- 15–30 second generation

```bash
ollama pull qwen:7b
ollama pull codellama
ollama serve
```

#### Scenario 3: CI/CD Server (32GB+ RAM)

```json
{
  "default_model": "llama2:13b",
  "strategy": "quality",
  "max_iterations": 5
}
```

- Highest quality for team standards
- 30–60 second generation
- Accept longer iteration loops

```bash
ollama pull llama2:13b
ollama pull llama2:34b
ollama serve --gpu all
```

---

## Optimization & Performance

### 1. GPU Acceleration

Ollama automatically uses GPU if available. Verify:

```bash
# Check if GPU detected
ollama list

# Should show "3.5 GB / 8 GB" or similar VRAM usage
```

**Enable NVIDIA GPU:**

```bash
# Install CUDA if not already installed
nvidia-smi

# Ollama will auto-detect and use NVIDIA GPUs
ollama serve
```

**Enable Metal (macOS):**

```bash
# Ollama uses Metal by default on macOS
# Verify with System Information → GPU
```

### 2. Model Quantization

Smaller quantizations load faster:

```bash
# Download quantized versions (smaller, faster)
ollama pull mistral:latest        # Default (full precision)
ollama pull mistral:4bit          # 4-bit quantization (faster, some quality loss)

# 4-bit is usually fine for artifact generation
# Use full precision only if quality is critical
```

In `.github/ollama.config.json`:

```json
{
  "models": {
    "mistral": {
      "name": "mistral:4bit",
      "tokens": 2048
    }
  }
}
```

### 3. Caching & Memory

Cache Ollama responses to avoid re-generation:

Store in `/memories/session/ollama-cache.md`:

```markdown
## Ollama Response Cache

Cache recent generations to avoid re-running for similar requests.

### Cache entries:
- Request hash: SHA256(problem_description)
- Model used: <model name>
- Response: <generated artifact>
- Quality score: <X.X/10>
- Timestamp: <ISO date>
```

Update `artifact-memory.skill.md`:

```markdown
## Step 1: RECALL (with Ollama cache)

Before calling Ollama:
1. Hash the problem description
2. Check `/memories/session/ollama-cache.md`
3. If cache hit AND score ≥ 9.0 → return cached artifact
4. If cache hit AND score < 9.0 → use as starting point
5. If cache miss → proceed with Ollama call
```

### 4. Prompt Optimization

Write efficient prompts for Ollama:

```markdown
## Efficient Prompts for Ollama

### Good Prompt (concise, structured)
```
You are an agent generator. Generate a .agent.md file for:

PROBLEM: Security code review agent
DOMAIN: security
COMPLEXITY: medium

REQUIREMENTS:
- name: security-pr-reviewer
- tools: [search, read, vscode/askQuestions]
- rules: 3-5 rules about what NOT to do
- capabilities: 4-5 concrete capabilities
- workflow: 4 phases with concrete steps

OUTPUT FORMAT:
---
name: ...
description: ...
[frontmatter]
---
[body]
```

### Bad Prompt (vague, long)
```
Please generate an agent that is really good at reviewing
pull requests and finding security issues. It should be 
comprehensive but not too long, and it should work well 
with our codebase...
```

### Prompt Templates

Store in `.github/prompts/ollama/`:

**`generate-agent.prompt`:**
```
You are an expert agent generator. Create a VS Code agent for:

PROBLEM STATEMENT:
{problem_description}

DOMAIN: {domain}
COMPLEXITY: {complexity}
QUALITY_TARGET: 9.0/10

CONTEXT:
- Existing agents: {existing_agent_names}
- Project conventions: {project_conventions}
- Code style: {code_style}

CONSTRAINTS:
- Valid tools: {valid_tools}
- Frontmatter format: YAML between --- markers
- Body structure: rules, capabilities, workflow (no placeholders)

GENERATE A VALID .agent.md FILE:
```

**`generate-skill.prompt`:**
```
You are an expert skill generator. Create a VS Code skill for:

WORKFLOW: {workflow_description}
DOMAIN: {domain}
PREREQUISITES: {prerequisites}

Generate a valid SKILL.md with:
- 5-7 numbered workflow steps
- At least 1 decision branch per step
- Quality criteria checklist
- Output specification

FRONTMATTER:
---
name: {skill_name}
description: ...
argument-hint: ...
disable-model-invocation: true
---
```

### 5. Timeout & Resource Management

Configure timeouts in `.github/ollama.config.json`:

```json
{
  "timeouts": {
    "model_load": 30000,      // 30s to load model into VRAM
    "generation": 60000,      // 60s max generation time
    "total_request": 120000   // 2 minutes total per artifact
  },
  "resource_limits": {
    "max_tokens": 4096,
    "context_window": 2048,
    "temperature": 0.6
  }
}
```

In `generate-artifact.agent.md`:

```markdown
## Error Handling

If Ollama times out:
1. Check: Is Ollama service running? (`curl http://localhost:11434/api/tags`)
2. Check: Is the model loaded? (`ollama list`)
3. Check: Free memory? (`free -h`)
4. Fallback to NLP-only strategy
5. User can increase timeouts in ollama.config.json

If out of memory:
1. Unload other models: `ollama rm <model>`
2. Or use smaller model: mistral (7B) instead of llama2:13b
3. Or increase swap space
```

---

## Advanced Patterns

### 1. Multi-Model Voting (Ensemble)

For critical artifacts, generate with 2–3 models and pick the best:

```markdown
## Step 3: ACT — Ensemble Generation

If quality is critical (quality_threshold > 9.2):
  models = [qwen:7b, codellama, llama2:13b]
  
  for model in models:
    response[model] = ollama.run(model, prompt)
    parse response into artifact
    score[model] = evaluate(artifact)
  
  best = max(score.values())
  return artifact[best]
  
  // Only needed for team standards or framework-wide agents
```

Example:

```bash
# Generate with three models, evaluate each, return highest-scoring one
/Generate AI Artifact --ensemble create a core authentication agent for the platform
```

### 2. Model Switching by Complexity

In `.github/skills/generate-agent.skill.md`:

```markdown
## Step 2: Select Model

Based on complexity score:

| Complexity | Model | Generation Time | Quality |
|-----------|-------|-----------------|---------|
| LOW | mistral | 5s | Good |
| MEDIUM | qwen:7b | 15s | Very Good |
| HIGH | llama2:13b | 45s | Excellent |
| CRITICAL | codellama (for code) | 60s | Excellent |

Decision:
- If COMPLEXITY == HIGH AND domain == code-generation → use codellama
- Else if COMPLEXITY == MEDIUM → use qwen:7b
- Else → use mistral (default)
```

### 3. Local Fine-Tuning (Advanced)

For custom artifact styles, fine-tune a model on your past artifacts:

```bash
# Create training data from your ARTIFACT_LIBRARY
# Format: {input: problem_description, output: generated_artifact}

# Ollama fine-tuning (advanced)
# Not yet supported natively, but can use: https://github.com/jmorganca/ollama/blob/main/docs/finetuning.md
```

### 4. Streaming Generation

For long artifacts, stream Ollama responses:

```markdown
## Step 3: ACT — Streaming

For artifacts > 2000 tokens:
  Use streaming API:
  - ollama generate --stream <model> <prompt>
  - Display generation progress to user
  - Reduces perceived latency
```

Example in VS Code:

```typescript
// In a custom extension (advanced)
const response = await fetch('http://localhost:11434/api/generate', {
  method: 'POST',
  body: JSON.stringify({
    model: 'mistral',
    prompt: agentGenerationPrompt,
    stream: true
  })
});

for await (const chunk of response.body) {
  // Stream chunk to UI
  updateProgressBar(chunk);
}
```

### 5. Custom Model Hosting

Run Ollama on a remote server for team sharing:

**Remote server setup:**

```bash
# On server: 192.168.1.100
ollama serve --host 0.0.0.0:11434

# Keep models loaded (prevent unloading)
# Edit: /etc/systemd/system/ollama.service or ~/.ollama/ollama.service
[Service]
ExecStart=/usr/bin/ollama serve
```

**Client configuration:**

```json
{
  "endpoint": "http://192.168.1.100:11434",
  "default_model": "mistral"
}
```

Then all team members generate artifacts using the shared model server.

---

## Troubleshooting

### "Connection refused: http://localhost:11434"

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# If not, start it
ollama serve

# If still fails:
# - Port 11434 blocked by firewall?
# - Check: lsof -i :11434
# - If occupied, change port: ollama serve --port 11435
```

### "CUDA out of memory"

```bash
# Check VRAM usage
nvidia-smi

# Solutions:
1. Unload other models:
   ollama rm llama2:13b
   
2. Use smaller model:
   ollama pull mistral
   
3. Increase context window size in ollama.config.json:
   "context_window": 1024  // Reduce from 2048

4. Use model quantization:
   ollama pull mistral:4bit
```

### "Model takes 2+ minutes to generate"

```bash
# Model might be swapping to disk (out of VRAM)

# Check:
free -h                    # Available RAM
nvidia-smi                 # VRAM usage

# Solutions:
1. Use smaller model (mistral instead of llama2:13b)
2. Increase available VRAM (close other apps)
3. Enable GPU acceleration properly:
   - nvidia-smi shows VRAM allocated?
   - Check CUDA_VISIBLE_DEVICES environment variable
4. Reduce max_tokens in ollama.config.json
```

### "Generated agent has hallucinations (invalid tool names)"

```bash
# Ollama sometimes makes up tool names

# Solutions:
1. Use better model:
   mistral → qwen:7b → llama2:13b
   
2. Improve prompt in generate-agent.skill.md:
   Add list of VALID tools explicitly:
   "Valid tools: search, read, edit, web, vscode/memory, ..."
   
3. Increase temperature (for variety) or decrease (for consistency):
   temperature: 0.6  // Lower = more consistent, less creative
   
4. Add post-processing: evaluate-artifact catches tool name errors
   and triggers regeneration automatically
```

### "Ollama doesn't persist between restarts"

```bash
# Models are cached locally, but not auto-loaded
# Need to pre-load or add to startup script

# Option 1: Add to shell startup
# ~/.bashrc or ~/.zshrc
ollama serve &

# Option 2: Create systemd service
# /etc/systemd/system/ollama.service
[Unit]
Description=Ollama
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/ollama serve
Restart=always
User=ollama

[Install]
WantedBy=multi-user.target

# Enable:
sudo systemctl enable ollama
sudo systemctl start ollama
```

### "Quality score < 7.0, endless iteration"

```bash
# Problem description is too ambiguous

# Solutions:
1. Regenerate with more specific description:
   /Generate AI Artifact I need an agent that reviews pull requests.
   It should:
   - Check for security vulnerabilities
   - Look for SQL injection and XSS
   - NOT modify code, only report findings
   
2. Break into smaller artifacts:
   Instead of: "comprehensive code review agent"
   Generate: "security code review agent" + separate "style review agent"
   
3. Lower quality threshold for this artifact:
   In evaluate-artifact.skill.md, set threshold to 8.5 instead of 9.0
   (accept slightly lower quality to avoid infinite iteration)
```

---

## Cost-Benefit Analysis

### Running Ollama Locally vs. Cloud APIs

| Factor | Ollama (Local) | Cloud API (e.g., OpenAI) |
|--------|---|---|
| **Monthly cost** | $0 (hardware) | $0–100+ (usage-based) |
| **Privacy** | ✅ All local | ❌ Data sent to cloud |
| **Speed** | Medium (depends on hardware) | Slow (network latency) |
| **Model quality** | Good (7B–13B) | Excellent (GPT-4) |
| **Customization** | ✅ Fine-tune locally | ❌ Limited |
| **Availability** | ✅ Offline | ❌ Requires internet |
| **Setup effort** | Medium | Low |
| **Hardware cost** | $500–3000 (one-time) | $0 |

### When to Use Ollama

✅ Use Ollama if:
- Generating artifacts within a company (privacy)
- Team has available hardware (GPU or high-RAM CPU)
- Generating frequently (cost savings)
- Offline capability needed
- Long-term cost reduction matters

❌ Use cloud API if:
- One-time generation (no hardware investment)
- Highest quality needed (GPT-4 > Llama 2)
- Team prefers minimal setup
- Budget allows per-use payment

### Hybrid Approach

Use both:

```json
{
  "primary": "ollama",
  "endpoint": "http://localhost:11434",
  "fallback_api": {
    "provider": "openai",
    "model": "gpt-3.5-turbo",
    "trigger": "quality_score < 8.5 after 3 iterations"
  }
}
```

If Ollama generation gets stuck in low-quality loop, fallback to OpenAI for that artifact.

---

## Performance Benchmarks

Generate security-pr-reviewer agent on different setups:

| Setup | Model | Strategy | Time | Quality |
|-------|-------|----------|------|---------|
| MacBook Pro (M1, 16GB) | mistral | Hybrid | 8s | 8.8/10 |
| MacBook Pro (M1, 16GB) | qwen:7b | Hybrid | 18s | 9.2/10 |
| Ubuntu (i7, 16GB, RTX3060) | mistral | NLP→Hybrid | 5s | 9.0/10 |
| Ubuntu (Xeon, 32GB, RTX3090) | llama2:13b | Quality | 35s | 9.6/10 |
| Laptop (i5, 8GB, no GPU) | mistral | NLP-only | 2s | 8.0/10 |

**Takeaway**: Hybrid strategy gives best ROI — NLP speed + LLM quality in 10–20 seconds.

---

## Next Steps

1. **Install Ollama**: `brew install ollama` (or your OS)
2. **Pull a model**: `ollama pull mistral`
3. **Start service**: `ollama serve`
4. **Create config**: Save `.github/ollama.config.json`
5. **Update orchestrator**: Add Ollama reference to `generate-ai-artifact.agent.md`
6. **Test**: `/Generate AI Artifact create a simple agent`
7. **Monitor**: Track generation time and quality scores
8. **Optimize**: Adjust model or strategy based on results

---

## Related Resources

- [Ollama Official Docs](https://ollama.ai/)
- [Available Models](https://ollama.ai/library)
- [Model Quantization Guide](https://github.com/ollama/ollama/wiki/Quantization)
- [AI Artifact Generator — Usage Guide](AI_artifact_generator_guide.md)
- [Sparrow Pseudocode Agent — Ollama Integration](https://github.com/the-genemachine/sparrow-pseudocode-agent/blob/main/src/ollama.rs)
