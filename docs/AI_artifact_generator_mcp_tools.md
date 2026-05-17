# AI Artifact Generator — MCP Servers & Tool Calling Guide

Extend the AI Artifact Generator with **Model Context Protocol (MCP) servers** for domain-specific integrations and custom tool calling. Add database queries, API lookups, codebase analysis, and specialized generation workflows without modifying core agents.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [MCP Server Architecture](#mcp-server-architecture)
3. [Creating Custom MCP Servers](#creating-custom-mcp-servers)
4. [Tool Calling Patterns](#tool-calling-patterns)
5. [Domain-Specific Examples](#domain-specific-examples)
6. [Integration with the Artifact Generator](#integration-with-the-artifact-generator)
7. [Team MCP Server Setup](#team-mcp-server-setup)
8. [Troubleshooting](#troubleshooting)

---

## Quick Start

### 1. Understand the Architecture

The AI Artifact Generator uses **tools** to accomplish tasks:

```
┌─────────────────────────────────────────────┐
│  /Generate AI Artifact                      │
│  (Main Orchestrator Agent)                  │
└────────────────┬────────────────────────────┘
                 │
        ┌────────┼────────┐
        │        │        │
        ↓        ↓        ↓
    ┌───────┐ ┌──────┐ ┌──────────┐
    │search │ │read  │ │edit      │  <- Built-in tools
    │ tool  │ │ tool │ │ tool     │
    └───────┘ └──────┘ └──────────┘
        ↓        ↓        ↓
    ┌──────────────────────────────────┐
    │ MCP Servers (custom integrations)│
    ├──────────────────────────────────┤
    │ • Codebase Analysis Server       │
    │ • Database Schema Server         │
    │ • API Documentation Server       │
    │ • Pattern Recognition Server     │
    │ • Custom Domain Tools            │
    └──────────────────────────────────┘
```

### 2. Register an MCP Server

Create `.github/mcp-servers/codebase-analyzer.json`:

```json
{
  "name": "codebase-analyzer",
  "description": "Analyzes codebase patterns, architecture, and conventions",
  "command": "python",
  "args": [".github/mcp-servers/codebase_analyzer.py"],
  "tools": [
    {
      "name": "find_similar_agents",
      "description": "Find existing agents similar to what the user wants",
      "input_schema": {
        "type": "object",
        "properties": {
          "domain": { "type": "string", "description": "Domain (security, testing, etc.)" },
          "keywords": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["domain"]
      }
    },
    {
      "name": "analyze_code_patterns",
      "description": "Extract common code patterns from the project",
      "input_schema": {
        "type": "object",
        "properties": {
          "language": { "type": "string", "description": "Programming language" },
          "pattern_type": { "type": "string", "description": "Type of pattern to find" }
        },
        "required": ["language"]
      }
    }
  ]
}
```

### 3. Enable the MCP Server in the Agent

Update `.github/agents/generate-ai-artifact.agent.md`:

```markdown
---
name: Generate AI Artifact
description: [...]
target: vscode
tools: [search, read, edit, vscode/memory, vscode/askQuestions, agent]
mcp-servers:
  - codebase-analyzer
  - database-schema
  - api-docs
---

[Rest of agent content]
```

### 4. The Agent Now Has New Capabilities

When generating an artifact, the agent can:

```
1. Use codebase-analyzer to find similar existing agents
2. Use find_similar_agents tool to avoid duplication
3. Use analyze_code_patterns to understand your project style
4. Generate an artifact that matches your conventions
```

---

## MCP Server Architecture

### What is MCP?

**Model Context Protocol (MCP)** allows agents to call external tools and services:

```
Agent                      MCP Server
  │                            │
  ├─ Call tool ──────────────→ │
  │                            │
  │                    Process request
  │                            │
  │ ← Return result ───────────┤
  │                            │
```

### MCP Server Components

Every MCP server has:

1. **Name & Description** — What it does
2. **Tools** — Functions the agent can call
3. **Input Schema** — Required/optional parameters
4. **Implementation** — Python, Node.js, Rust, etc.

### Standard Tool Lifecycle

```
1. Agent needs information → calls MCP tool
2. Tool receives parameters
3. Tool executes logic (query DB, analyze code, call API)
4. Tool returns structured result
5. Agent uses result in generation or decision-making
```

---

## Creating Custom MCP Servers

### Example 1: Codebase Analyzer MCP Server

**File: `.github/mcp-servers/codebase_analyzer.py`**

```python
"""
Codebase Analyzer MCP Server

Provides tools for analyzing project structure, patterns, and conventions.
"""

import json
import subprocess
from pathlib import Path
from typing import Any

class CodebaseAnalyzerServer:
    def __init__(self):
        self.project_root = Path.cwd()
    
    def find_similar_agents(self, domain: str, keywords: list[str]) -> dict:
        """Find existing agents similar to what the user wants."""
        agents_dir = self.project_root / ".github" / "agents"
        results = []
        
        for agent_file in agents_dir.glob("*.agent.md"):
            with open(agent_file) as f:
                content = f.read()
            
            # Check if domain matches
            if domain.lower() in content.lower():
                results.append({
                    "name": agent_file.stem,
                    "path": str(agent_file),
                    "relevance": "high" if any(kw in content for kw in keywords) else "medium"
                })
        
        return {
            "similar_agents": results,
            "count": len(results),
            "recommendation": "Create a variation" if results else "Generate new agent"
        }
    
    def analyze_code_patterns(self, language: str, pattern_type: str) -> dict:
        """Extract common code patterns from the project."""
        src_dir = self.project_root / "src"
        patterns = {
            "naming_conventions": self._extract_naming_conventions(language),
            "error_handling": self._extract_error_patterns(language),
            "testing_patterns": self._extract_test_patterns(language),
            "documentation_style": self._extract_doc_style(language)
        }
        
        return {
            "language": language,
            "patterns": patterns[pattern_type] if pattern_type in patterns else patterns,
            "examples": self._get_example_files(language, 3)
        }
    
    def _extract_naming_conventions(self, language: str) -> dict:
        """Extract naming patterns from source files."""
        # Simplified: look for common patterns
        return {
            "function_names": "snake_case",
            "class_names": "PascalCase",
            "constants": "UPPER_SNAKE_CASE",
            "private_vars": "_underscore_prefix"
        }
    
    def _extract_error_patterns(self, language: str) -> list:
        """Find error handling patterns."""
        patterns = []
        # In real implementation: parse AST and identify error patterns
        if language == "python":
            patterns = ["try/except blocks", "custom exceptions", "logging"]
        return patterns
    
    def _extract_test_patterns(self, language: str) -> list:
        """Find testing patterns."""
        # Look for test files and extract patterns
        test_dir = self.project_root / "tests"
        if test_dir.exists():
            return ["pytest", "assertions", "fixtures", "mocking"]
        return []
    
    def _extract_doc_style(self, language: str) -> str:
        """Determine documentation style."""
        # Check for docstrings, comments, README
        return "docstrings" if language == "python" else "JSDoc/comments"
    
    def _get_example_files(self, language: str, count: int) -> list:
        """Get example files in the language."""
        extensions = {
            "python": ".py",
            "javascript": ".js",
            "typescript": ".ts",
            "java": ".java",
            "rust": ".rs"
        }
        
        ext = extensions.get(language, "")
        src_dir = self.project_root / "src"
        
        if src_dir.exists():
            files = list(src_dir.rglob(f"*{ext}"))[:count]
            return [str(f.relative_to(self.project_root)) for f in files]
        return []
    
    def handle_tool_call(self, tool_name: str, params: dict) -> Any:
        """Route tool calls to appropriate handler."""
        if tool_name == "find_similar_agents":
            return self.find_similar_agents(
                domain=params.get("domain"),
                keywords=params.get("keywords", [])
            )
        elif tool_name == "analyze_code_patterns":
            return self.analyze_code_patterns(
                language=params.get("language"),
                pattern_type=params.get("pattern_type")
            )
        else:
            return {"error": f"Unknown tool: {tool_name}"}

# Main entry point for MCP server
if __name__ == "__main__":
    server = CodebaseAnalyzerServer()
    
    # Read stdin for tool calls (MCP protocol)
    import sys
    for line in sys.stdin:
        try:
            request = json.loads(line)
            tool_name = request.get("tool")
            params = request.get("params", {})
            
            result = server.handle_tool_call(tool_name, params)
            print(json.dumps({"success": True, "result": result}))
        except Exception as e:
            print(json.dumps({"success": False, "error": str(e)}))
```

### Example 2: Database Schema MCP Server

**File: `.github/mcp-servers/database_schema.py`**

```python
"""
Database Schema MCP Server

Provides tools for querying database schema and generating schema-aware agents.
"""

import json
import subprocess
from typing import Any

class DatabaseSchemaServer:
    def __init__(self):
        self.db_url = "postgresql://localhost/mydb"  # Configure as needed
    
    def get_table_schema(self, table_name: str) -> dict:
        """Get column definitions for a table."""
        # In real implementation: query actual database
        return {
            "table": table_name,
            "columns": [
                {"name": "id", "type": "uuid", "nullable": False, "primary_key": True},
                {"name": "email", "type": "varchar", "nullable": False, "unique": True},
                {"name": "created_at", "type": "timestamp", "nullable": False}
            ]
        }
    
    def list_tables(self) -> dict:
        """List all tables in the database."""
        # In real implementation: query INFORMATION_SCHEMA
        return {
            "tables": ["users", "posts", "comments", "tags"],
            "count": 4
        }
    
    def get_relationships(self, table_name: str) -> dict:
        """Get foreign key relationships."""
        relationships = {
            "posts": [
                {"target_table": "users", "foreign_key": "user_id", "type": "many_to_one"}
            ],
            "comments": [
                {"target_table": "posts", "foreign_key": "post_id", "type": "many_to_one"},
                {"target_table": "users", "foreign_key": "user_id", "type": "many_to_one"}
            ]
        }
        return relationships.get(table_name, {})
    
    def generate_migration_template(self, operation: str, table_name: str) -> str:
        """Generate a SQL migration template."""
        if operation == "create":
            return f"""
-- Create table {table_name}
CREATE TABLE {table_name} (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Create index on created_at
CREATE INDEX idx_{table_name}_created_at ON {table_name}(created_at);
"""
        elif operation == "alter":
            return f"ALTER TABLE {table_name} ADD COLUMN new_column VARCHAR(255);"
        
        return ""
    
    def handle_tool_call(self, tool_name: str, params: dict) -> Any:
        """Route tool calls."""
        if tool_name == "get_table_schema":
            return self.get_table_schema(params.get("table_name"))
        elif tool_name == "list_tables":
            return self.list_tables()
        elif tool_name == "get_relationships":
            return self.get_relationships(params.get("table_name"))
        elif tool_name == "generate_migration_template":
            return {"migration": self.generate_migration_template(
                params.get("operation"), params.get("table_name")
            )}
        else:
            return {"error": f"Unknown tool: {tool_name}"}
```

### Example 3: API Documentation MCP Server

**File: `.github/mcp-servers/api_docs.py`**

```python
"""
API Documentation MCP Server

Provides tools for extracting and analyzing API documentation.
"""

import json
import yaml
from pathlib import Path
from typing import Any

class ApiDocsServer:
    def __init__(self):
        self.openapi_files = list(Path.cwd().glob("docs/openapi/*.yaml"))
    
    def get_endpoint_info(self, endpoint_path: str) -> dict:
        """Get details about a specific API endpoint."""
        # In real implementation: parse OpenAPI spec
        return {
            "path": endpoint_path,
            "methods": ["GET", "POST"],
            "parameters": [
                {"name": "id", "in": "path", "required": True, "schema": {"type": "string"}},
                {"name": "limit", "in": "query", "required": False, "schema": {"type": "integer"}}
            ],
            "responses": {
                "200": {"description": "Success"},
                "404": {"description": "Not found"}
            }
        }
    
    def list_endpoints(self, tag: str = None) -> dict:
        """List all API endpoints, optionally filtered by tag."""
        # In real implementation: query OpenAPI spec
        endpoints = [
            "/users",
            "/users/{id}",
            "/posts",
            "/posts/{id}/comments"
        ]
        return {"endpoints": endpoints, "count": len(endpoints)}
    
    def get_auth_requirements(self) -> dict:
        """Get authentication scheme for APIs."""
        return {
            "type": "bearer",
            "scheme": "jwt",
            "bearer_format": "JWT"
        }
    
    def handle_tool_call(self, tool_name: str, params: dict) -> Any:
        """Route tool calls."""
        if tool_name == "get_endpoint_info":
            return self.get_endpoint_info(params.get("endpoint_path"))
        elif tool_name == "list_endpoints":
            return self.list_endpoints(params.get("tag"))
        elif tool_name == "get_auth_requirements":
            return self.get_auth_requirements()
        else:
            return {"error": f"Unknown tool: {tool_name}"}
```

---

## Tool Calling Patterns

### Pattern 1: Tool Chaining

The agent calls multiple tools sequentially:

```
1. find_similar_agents(domain="security") 
   → Returns: security-pr-reviewer exists

2. analyze_code_patterns(language="python")
   → Returns: Your code uses type hints + docstrings

3. [Decision] Domain and patterns are known
   → Generate security agent with your patterns
```

### Pattern 2: Conditional Tool Calling

The agent decides which tools to call based on input:

```markdown
## Phase 1: PERCEIVE

If user input is CODE:
  → Call analyze_code_patterns tool
  → Extract patterns from the code
Else if user input is PROBLEM DESCRIPTION:
  → Call find_similar_agents tool
  → Check for existing solutions
Else:
  → Ask for clarification
```

### Pattern 3: Fallback Patterns

If a tool fails, use fallback:

```markdown
## Tool Error Handling

Try:
  1. Call codebase_analyzer.find_similar_agents()
  
Catch network/permission error:
  2. Fallback to NLP-based pattern matching
  3. Use generic templates instead
  
Return:
  Best result (tool or fallback)
```

### Pattern 4: Parallel Tool Calls

The agent calls multiple tools at once:

```
Parallel:
  - get_table_schema("users")
  - get_relationships("users")
  - get_endpoint_info("/users")

Wait for all to complete, then:
  → Generate comprehensive data agent
```

---

## Domain-Specific Examples

### Example 1: Web Development MCP Server

**File: `.github/mcp-servers/web_dev.json`**

```json
{
  "name": "web-dev-tools",
  "description": "Web development specific tools",
  "command": "python",
  "args": [".github/mcp-servers/web_dev.py"],
  "tools": [
    {
      "name": "get_framework_conventions",
      "description": "Get conventions for React/Vue/Angular/FastAPI",
      "input_schema": {
        "type": "object",
        "properties": {
          "framework": { "type": "string", "enum": ["react", "vue", "angular", "fastapi", "django"] }
        },
        "required": ["framework"]
      }
    },
    {
      "name": "list_components",
      "description": "List existing UI components in the project",
      "input_schema": {
        "type": "object",
        "properties": {
          "component_type": { "type": "string", "enum": ["button", "form", "modal", "layout"] }
        }
      }
    },
    {
      "name": "get_component_api",
      "description": "Get props and API for a component",
      "input_schema": {
        "type": "object",
        "properties": {
          "component_name": { "type": "string" }
        },
        "required": ["component_name"]
      }
    }
  ]
}
```

**Update agent:**

```markdown
---
name: Web Agent Generator
description: Generates web-specific agents for React/Vue/Angular/FastAPI
target: vscode
tools: [search, read, edit, web, vscode/askQuestions, agent]
mcp-servers:
  - web-dev-tools
  - api-docs
---

[Agent uses these tools when generating web agents]
```

**Agent uses the tools:**

```markdown
## Phase 1: PERCEIVE

Call get_framework_conventions(framework="react")
→ Get React patterns: hooks, components, testing with Jest

Call list_components(component_type="form")
→ Find existing form components to match style

## Phase 3: ACT

Generate form component agent using:
- React conventions from tool
- Existing component style from tool
- Project patterns
```

### Example 2: Data Engineering MCP Server

```json
{
  "name": "data-tools",
  "description": "Data engineering and pipeline tools",
  "command": "python",
  "args": [".github/mcp-servers/data_tools.py"],
  "tools": [
    {
      "name": "get_table_schema",
      "description": "Get database table schema"
    },
    {
      "name": "list_pipelines",
      "description": "List existing data pipelines"
    },
    {
      "name": "get_data_lineage",
      "description": "Trace data flow from source to destination"
    },
    {
      "name": "get_dbt_models",
      "description": "List dbt models and their relationships"
    }
  ]
}
```

### Example 3: Security & Compliance MCP Server

```json
{
  "name": "security-tools",
  "description": "Security and compliance checking tools",
  "command": "python",
  "args": [".github/mcp-servers/security_tools.py"],
  "tools": [
    {
      "name": "get_threat_model",
      "description": "Get threat model for the application"
    },
    {
      "name": "get_compliance_requirements",
      "description": "Get compliance checklist (GDPR, SOC2, etc.)"
    },
    {
      "name": "check_dependencies",
      "description": "Check for known CVEs in dependencies"
    },
    {
      "name": "get_security_patterns",
      "description": "Get security best practices used in codebase"
    }
  ]
}
```

---

## Integration with the Artifact Generator

### Step 1: Register MCP Servers

Create `.github/mcp-servers/registry.json`:

```json
{
  "servers": [
    {
      "name": "codebase-analyzer",
      "path": ".github/mcp-servers/codebase_analyzer.py",
      "enabled": true,
      "timeout": 10000
    },
    {
      "name": "database-schema",
      "path": ".github/mcp-servers/database_schema.py",
      "enabled": true,
      "timeout": 5000
    },
    {
      "name": "api-docs",
      "path": ".github/mcp-servers/api_docs.py",
      "enabled": true,
      "timeout": 5000
    },
    {
      "name": "web-dev-tools",
      "path": ".github/mcp-servers/web_dev.py",
      "enabled": true,
      "timeout": 10000
    }
  ]
}
```

### Step 2: Update the Orchestrator

Modify `.github/agents/generate-ai-artifact.agent.md`:

```markdown
---
name: Generate AI Artifact
description: [...existing...]
target: vscode
tools: [search, read, edit, vscode/memory, vscode/askQuestions, agent]
mcp-servers:
  - codebase-analyzer
  - database-schema
  - api-docs
---

You are the **AI Artifact Generator** with access to specialized MCP servers.

<rules>
- Before generating an agent, use codebase-analyzer to find similar existing agents
- When domain is web/api, call web-dev-tools to get framework conventions
- When domain is data, call database-schema to understand your data model
- Always respect project conventions discovered via MCP tools
</rules>

<workflow>

## 1. PERCEIVE (with MCP)
Classify input using standard analysis
If applicable, query relevant MCP servers:
- Domain = security → call security-tools
- Domain = web → call web-dev-tools
- Domain = data → call database-schema

## 2. PLAN
Use MCP tool results to select generation strategy

## 3. ACT
Generate artifact using:
- Project patterns from codebase-analyzer
- Domain-specific knowledge from MCP servers
- Your existing code conventions

## 4. OBSERVE & DECIDE
Evaluate quality, iterate if needed

</workflow>
```

### Step 3: Usage

Now when you use the generator:

```
/Generate AI Artifact I need an agent for web form validation in React

[PERCEIVE phase]
→ Calls get_framework_conventions("react")
→ Gets: "Your project uses hooks, Jest testing, TypeScript"

[ACT phase]
→ Generates form validation agent using React patterns + your conventions

[Result]
→ High-quality agent matching your project style
```

---

## Team MCP Server Setup

### 1. Shared MCP Server Infrastructure

**For teams, run MCP servers on a shared machine:**

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mcp-codebase-analyzer:
    build:
      context: .github/mcp-servers
      dockerfile: Dockerfile.codebase-analyzer
    ports:
      - "8001:8000"
    volumes:
      - .:/app
    environment:
      - PROJECT_ROOT=/app

  mcp-database-schema:
    build:
      context: .github/mcp-servers
      dockerfile: Dockerfile.database-schema
    ports:
      - "8002:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb

  mcp-api-docs:
    build:
      context: .github/mcp-servers
      dockerfile: Dockerfile.api-docs
    ports:
      - "8003:8000"
    volumes:
      - ./docs/openapi:/app/openapi
```

Update `.github/mcp-servers/registry.json` for remote servers:

```json
{
  "servers": [
    {
      "name": "codebase-analyzer",
      "endpoint": "http://mcp-servers:8001",
      "enabled": true,
      "timeout": 10000
    },
    {
      "name": "database-schema",
      "endpoint": "http://mcp-servers:8002",
      "enabled": true,
      "timeout": 5000
    },
    {
      "name": "api-docs",
      "endpoint": "http://mcp-servers:8003",
      "enabled": true,
      "timeout": 5000
    }
  ]
}
```

### 2. Dockerfile for MCP Server

**File: `.github/mcp-servers/Dockerfile.codebase-analyzer`**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY codebase_analyzer.py .
COPY requirements.txt .

RUN pip install -r requirements.txt

EXPOSE 8000

CMD ["python", "-m", "uvicorn", "codebase_analyzer:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 3. Team Onboarding

Document in `CONTRIBUTING.md`:

```markdown
## Using MCP Servers

Our artifact generator uses specialized MCP servers:

1. **codebase-analyzer**: Finds existing agents, extracts patterns
2. **database-schema**: Queries database structure for data-aware agents
3. **api-docs**: Provides API specifications for API agent generation
4. **web-dev-tools**: Guides web component generation with framework conventions

### For Local Development

```bash
# Start MCP servers locally
docker-compose up mcp-codebase-analyzer mcp-api-docs

# Or run individually
python .github/mcp-servers/codebase_analyzer.py
```

### For Team (Shared Server)

MCP servers run on `mcp-servers.internal`:
- Each server listens on a dedicated port
- All developers use the same servers
- Ensures consistency across generated artifacts
```

---

## Troubleshooting

### "MCP Server not found"

```bash
# Check registry
cat .github/mcp-servers/registry.json

# Verify server is running
curl http://localhost:8001/health  # or relevant port

# Check logs
docker logs mcp-codebase-analyzer
```

### "Tool call timed out"

```json
{
  "servers": [
    {
      "name": "codebase-analyzer",
      "path": ".github/mcp-servers/codebase_analyzer.py",
      "timeout": 30000  // Increased from 10000
    }
  ]
}
```

### "Tool returned unexpected error"

Add error handling in agent:

```markdown
## Tool Error Handling

If tool call fails:
  1. Log the error
  2. Fallback to NLP-based approach
  3. Continue artifact generation with generic templates
  4. Note in artifact: "Generated without project-specific context"
```

### "MCP Server consuming too much memory"

```yaml
services:
  mcp-codebase-analyzer:
    # ... 
    resources:
      limits:
        memory: 2G
      reservations:
        memory: 1G
```

### "Database schema queries are slow"

Implement caching:

```python
from functools import lru_cache
from datetime import datetime, timedelta

class DatabaseSchemaServer:
    def __init__(self):
        self.cache = {}
        self.cache_ttl = 3600  # 1 hour
    
    @lru_cache(maxsize=128)
    def get_table_schema(self, table_name: str) -> dict:
        """Cached schema query."""
        # Query database
        # Cache result
        return schema
```

---

## Real-World Example: End-to-End Artifact Generation with MCP

**Scenario:** Generate a user management agent for a web app.

```
User: /Generate AI Artifact I need an agent that manages user CRUD operations

┌────────────────────────────────────────────────┐
│ PERCEIVE Phase                                 │
├────────────────────────────────────────────────┤
│ Domain: "api/crud"                             │
│ Call MCP tools:                                │
│  - get_framework_conventions("fastapi")        │
│  - get_table_schema("users")                   │
│  - get_endpoint_info("/users")                 │
│  - find_similar_agents("CRUD")                 │
└────────────────────────────────────────────────┘
                      ↓
┌────────────────────────────────────────────────┐
│ MCP Results                                    │
├────────────────────────────────────────────────┤
│ • Framework: FastAPI with dependency injection │
│ • Table schema: id, email, name, created_at   │
│ • Endpoints: GET/POST /users, GET /users/{id} │
│ • Similar agents: "crud-generator" (reuse!)   │
└────────────────────────────────────────────────┘
                      ↓
┌────────────────────────────────────────────────┐
│ PLAN Phase                                     │
├────────────────────────────────────────────────┤
│ Strategy: B (Hybrid, refine existing template) │
│ Model: mistral (7B)                            │
└────────────────────────────────────────────────┘
                      ↓
┌────────────────────────────────────────────────┐
│ ACT Phase                                      │
├────────────────────────────────────────────────┤
│ Generate user-crud.agent.md using:             │
│  • FastAPI conventions from MCP                │
│  • Schema: users(id, email, name, created_at) │
│  • Existing endpoints                          │
│  • Similar "crud-generator" agent pattern      │
│  • Project code style from codebase-analyzer   │
└────────────────────────────────────────────────┘
                      ↓
┌────────────────────────────────────────────────┐
│ Result                                         │
├────────────────────────────────────────────────┤
│ ✅ user-crud.agent.md created                 │
│  - Uses FastAPI dependency injection           │
│  - References exact schema columns             │
│  - Includes validation for user fields         │
│  - Matches project code style                  │
│  - Quality score: 9.4/10                       │
└────────────────────────────────────────────────┘
```

---

## Next Steps

1. **Create first MCP server**: Start with `codebase-analyzer`
2. **Test locally**: Run and call tools manually
3. **Register in agent**: Add to `.github/mcp-servers/registry.json`
4. **Update orchestrator**: Enable in `generate-ai-artifact.agent.md`
5. **Generate with MCP**: `/Generate AI Artifact [problem]`
6. **Monitor results**: Track quality improvements from MCP integration
7. **Expand**: Add domain-specific servers (web-dev, data, security)
8. **Team rollout**: Deploy shared MCP servers via Docker Compose

---

## Related Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP Server Examples](https://github.com/modelcontextprotocol/servers)
- [Tool Calling Guide](https://platform.openai.com/docs/guides/function-calling)
- [AI Artifact Generator — Integration Guide](AI_artifact_generator_boilerplate_guide.md)
