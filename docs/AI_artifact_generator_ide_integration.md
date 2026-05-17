# AI Artifact Generator — IDE Integration Guide

Integrate the AI Artifact Generator into your development workflow using VS Code, JetBrains IDEs, Vim, Neovim, or Emacs. Streamline artifact generation with keyboard shortcuts, context-aware invocations, and seamless integration with your coding environment.

---

## Table of Contents

1. [VS Code Setup](#vs-code-setup)
2. [Keyboard Shortcuts & Workflows](#keyboard-shortcuts--workflows)
3. [Context-Aware Generation](#context-aware-generation)
4. [JetBrains IDEs (IntelliJ, PyCharm, etc.)](#jetbrains-ides)
5. [Vim & Neovim](#vim--neovim)
6. [Emacs](#emacs)
7. [IDE-Agnostic Workflows](#ide-agnostic-workflows)
8. [Troubleshooting](#troubleshooting)

---

## VS Code Setup

### 1. Install Required Extensions

```bash
# GitHub Copilot (required for agent execution)
code --install-extension GitHub.copilot

# GitHub Copilot Chat (required for chat interface)
code --install-extension GitHub.copilot-chat

# Optional: Better comments
code --install-extension aaron-bond.better-comments

# Optional: Markdown preview
code --install-extension yzhang.markdown-all-in-one
```

Verify in VS Code:
1. Open Extensions (Cmd+Shift+X / Ctrl+Shift+X)
2. Search "Copilot"
3. Click "Install" on both extensions

### 2. Configure VS Code Settings

Create or update `.vscode/settings.json` in your project:

```json
{
  // Enable Copilot chat for this workspace
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": true
  },

  // Show inline suggestions from Copilot
  "github.copilot.editor.enableAutoCompletions": true,

  // AI Artifact Generator specific settings
  "artifactGenerator.defaultModel": "mistral",
  "artifactGenerator.strategy": "hybrid",
  "artifactGenerator.qualityThreshold": 9.0,
  "artifactGenerator.maxIterations": 3,
  "artifactGenerator.ollama.endpoint": "http://localhost:11434",

  // Format generated files on save
  "editor.formatOnSave": true,
  "[markdown]": {
    "editor.defaultFormatter": "yzhang.markdown-all-in-one"
  }
}
```

### 3. Create Custom Keybindings

Create `.vscode/keybindings.json`:

```json
[
  {
    "key": "cmd+shift+a",
    "mac": "cmd+shift+a",
    "linux": "ctrl+shift+a",
    "windows": "ctrl+shift+a",
    "command": "workbench.action.chat.open",
    "when": "!chatFocus"
  },
  {
    "key": "cmd+shift+g",
    "mac": "cmd+shift+g",
    "linux": "ctrl+shift+g",
    "windows": "ctrl+shift+g",
    "command": "workbench.action.quickOpen",
    "args": "/Generate AI Artifact "
  }
]
```

### 4. Create a Command Palette Profile

VS Code Command Palette (`Cmd+Shift+P` / `Ctrl+Shift+P`) defaults:

```bash
# Open chat
> Chat: Open
Shortcut: Cmd+L (default)

# Generate artifact (type this in chat)
/Generate AI Artifact

# Ask a question
/Ask

# Create a plan
/Plan
```

---

## Keyboard Shortcuts & Workflows

### Quick Reference

| Action | Shortcut | Notes |
|--------|----------|-------|
| Open Chat | `Cmd+L` / `Ctrl+L` | In any editor |
| Generate Artifact | `Cmd+Shift+G` | Opens chat with `/Generate AI Artifact` pre-filled |
| Open Chat | `Cmd+Shift+A` | Alternative via custom keybinding |
| Ask Question | Type `/Ask` in chat | Read-only query |
| Create Plan | Type `/Plan` in chat | Planning without implementation |
| Close Chat | `Escape` | From chat focus |
| Submit Message | `Enter` | Send to agent |
| New Chat | `Cmd+Shift+N` | Create new conversation |

### Typical Workflow: Generate a Security Agent

```
1. Press Cmd+Shift+G
   → Chat opens with cursor after "/Generate AI Artifact "

2. Type the problem:
   /Generate AI Artifact I need an agent that reviews pull requests for security vulnerabilities

3. Press Enter
   → Agent runs through PERCEIVE → ACT → OBSERVE → DECIDE pipeline
   → Takes 10–30 seconds
   → Artifact saved to .github/agents/security-pr-reviewer.agent.md

4. Review generated file:
   Cmd+P → security-pr-reviewer.agent.md → opens in editor

5. Customize if needed:
   Edit frontmatter, rules, workflow, etc.

6. Commit to git:
   Cmd+Shift+G (Source Control) → Stage → Commit
```

### Typical Workflow: Generate a Testing Skill

```
1. Cmd+Shift+G → Chat opens

2. Type:
   /Generate AI Artifact create a skill that generates unit tests for Python

3. Wait for pipeline to complete

4. Skill saved to .github/skills/generate-unit-tests/SKILL.md

5. Test the skill:
   /generate-unit-tests [paste a Python function]
   → Generates pytest-compatible test cases

6. Iterate if needed:
   /generate-unit-tests [different function or more complex scenario]
```

### Pair Programming With Chat

```
1. Cmd+L → Chat opens

2. Ask questions without generating:
   /Ask what patterns does this codebase use for authentication?
   
3. Then generate an agent based on findings:
   /Generate AI Artifact create an auth guard agent using patterns from our codebase

4. Agent is generated with knowledge of your existing patterns
```

---

## Context-Aware Generation

### 1. Use File Context

When you have a file open, the agent has context:

```typescript
// src/auth/guards.ts — file is open in editor

// Press Cmd+L → Chat opens
// Type:
/Generate AI Artifact create a guard that follows patterns from this file
```

The agent searches your codebase, finds `guards.ts`, and generates artifacts that match your style.

### 2. Use Selection Context

Select code, then generate:

```python
# src/utils/validation.py
def validate_email(email: str) -> bool:
    return "@" in email and "." in email.split("@")[1]

def validate_phone(phone: str) -> bool:
    return len(phone.replace("-", "")) == 10

# Select these two functions, press Cmd+L:
/Generate AI Artifact create a skill that generates test cases for these validation functions
```

The agent generates test skills specific to your validation patterns.

### 3. Use Project Context (.github/AGENTS.md)

The agent automatically reads your project conventions:

```markdown
# .github/AGENTS.md

## Code Style
- Use type hints (Python 3.9+)
- Follow PEP 8
- 100-character line limit

## Testing
- pytest for unit tests
- 80%+ coverage required
- Mock external APIs

## Architecture
- Domain-driven design
- Repository pattern for data access
```

When you generate artifacts:

```
/Generate AI Artifact create a test generator skill
```

The generator reads AGENTS.md, understands your conventions, and creates a skill that generates tests matching your pytest setup, coverage requirements, etc.

---

## JetBrains IDEs

### 1. Install Plugins

Works with: IntelliJ IDEA, PyCharm, WebStorm, GoLand, RubyMine, CLion, etc.

```
1. Settings/Preferences → Plugins
2. Search: "GitHub Copilot" or "JetBrains AI Assistant"
3. Install: GitHub Copilot plugin (official)
4. Restart IDE
```

### 2. Configure in JetBrains

**Settings → Tools → GitHub Copilot:**
```
✓ Enable GitHub Copilot
✓ Show inline suggestions
✓ Enable code completions
✓ Show signature help
```

### 3. Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Open Chat | `Cmd+Shift+A` (macOS) / `Ctrl+Alt+A` (Windows/Linux) |
| Generate Artifact | `Alt+Enter` → "Generate AI Artifact" |
| Inline Suggestion | `Tab` to accept / `Escape` to dismiss |

### 4. JetBrains-Specific Workflow

**Generate a test from a class:**

```java
// src/main/java/com/example/UserService.java
public class UserService {
    public User createUser(String email, String name) {
        // Validate email
        // Save to database
        // Return user
    }
}

// Right-click → GitHub Copilot → Generate Tests
// Or: Cmd+Shift+A → /Generate AI Artifact create unit tests for this class
```

**Analyze code performance:**

```kotlin
// src/main/kotlin/DataProcessor.kt
// Open file, then:
Cmd+Shift+A → /Ask what are performance bottlenecks in this code?
```

### 5. Create Custom Run Configurations

Create `.idea/runConfigurations/GenerateArtifact.xml`:

```xml
<component name="ProjectRunConfigurationManager">
  <configuration default="false" name="Generate AI Artifact" type="JavaScriptDebugType">
    <method v="2">
      <option name="RunConfigurationTask" enabled="true" />
    </method>
  </configuration>
</component>
```

Then create a quick action:
- Tools → Create Tool
- Program: `code` or `idea64` (your IDE command)
- Arguments: `--command vscode.executeCommand("vscode.openFolder", "$ProjectFileDir$")`

---

## Vim & Neovim

### 1. Install GitHub Copilot for Vim/Neovim

**For Neovim (recommended):**

```bash
# Using packer.nvim
use 'github/copilot.vim'

# Or using vim-plug
Plug 'github/copilot.vim'
```

**For Vim:**

```bash
# Using vim-plug
Plug 'github/copilot.vim'

# Or download: https://github.com/github/copilot.vim/releases
```

### 2. Configure Neovim

Create `~/.config/nvim/init.lua`:

```lua
-- GitHub Copilot
vim.keymap.set('i', '<C-J>', 'copilot#Accept("<CR>")', {
  noremap = true,
  silent = true,
  expr = true,
  replace_keycodes = false
})
vim.g.copilot_no_tab_map = true

-- Trigger suggestions
vim.keymap.set('i', '<C-Space>', 'copilot#Next()', {
  noremap = true,
  silent = true,
  expr = true
})
```

### 3. Open VS Code Chat from Vim/Neovim

Since Vim/Neovim Copilot doesn't have a chat interface, use VS Code alongside:

**Option A: Split windows**

```bash
# Terminal 1: Neovim
nvim src/my-file.ts

# Terminal 2: VS Code (opened to same project)
code .

# Use VS Code chat (Cmd+L) while keeping Neovim for editing
```

**Option B: Use tmux**

```bash
# ~/.tmux.conf
bind -n C-l send-keys -t 1 "clear && code . --new-window" Enter

# In tmux:
# Ctrl+L → Opens VS Code in a new window while keeping Neovim active
```

**Option C: Custom mapping to open chat**

```lua
-- ~/.config/nvim/init.lua
vim.keymap.set('n', '<Leader>ag', function()
  vim.fn.system('code . --command "vscode.executeCommand(\'workbench.action.chat.open\')"')
end, { noremap = true })

-- Usage: <Leader>ag → Opens VS Code chat
```

### 4. Vim Workflow for Artifact Generation

```vim
" 1. Edit your Vim config to add mapping
" ~/.config/nvim/init.lua:
map <C-A> :!code . --command "vscode.executeCommand('workbench.action.chat.open')"<CR>

" 2. In Vim, press Ctrl+A to open VS Code chat

" 3. Generate artifact in VS Code chat

" 4. Return to Vim to edit the generated file
:e .github/agents/my-agent.agent.md
```

### 5. Neovim LSP Integration

For inline completions while editing (without chat):

```lua
-- ~/.config/nvim/init.lua
require('copilot').setup({
  suggestion = {
    enabled = true,
    auto_trigger = true,
    debounce = 75,
    keymap = {
      accept = "<C-J>",
      next = "<M-]>",
      prev = "<M-[>",
      dismiss = "<C-]>",
    }
  }
})
```

Then use inline completions:
```
:edit .github/agents/myagent.agent.md
# Start typing the agent structure, Copilot suggests completions
# Press Ctrl+J to accept
```

---

## Emacs

### 1. Install Copilot for Emacs

```elisp
;; Using straight.el
(straight-use-package
 '(copilot :host github :repo "copilot-emacs/copilot.el"
           :branch "main"))

;; Or using use-package + straight
(use-package copilot
  :straight (:host github :repo "copilot-emacs/copilot.el" :branch "main")
  :ensure t)
```

### 2. Configure Emacs

Create/update `~/.emacs.d/init.el`:

```elisp
(use-package copilot
  :ensure t
  :config
  (add-hook 'prog-mode-hook 'copilot-mode)
  
  ;; Keybindings
  (define-key copilot-completion-map (kbd "TAB") 'copilot-accept-completion)
  (define-key copilot-completion-map (kbd "M-[") 'copilot-previous-completion)
  (define-key copilot-completion-map (kbd "M-]") 'copilot-next-completion)
  
  ;; Open chat
  (define-key global-map (kbd "C-c c a") 'copilot-chat-open))
```

### 3. Generate Artifacts from Emacs

**Option A: Shell command**

```elisp
(defun generate-ai-artifact (description)
  "Generate an AI artifact using the artifact generator."
  (interactive "sDescribe what you want to generate: ")
  (shell-command
    (concat "code . --command \"vscode.executeCommand('workbench.action.chat.open')\""))
  (message "VS Code chat opened. Type: /Generate AI Artifact %s" description))

;; Bind to a key
(define-key global-map (kbd "C-c c g") 'generate-ai-artifact)
```

**Option B: Direct Copilot Chat (if available)**

```elisp
(defun copilot-generate-artifact ()
  "Generate artifact via Copilot chat."
  (interactive)
  (copilot-chat-open)
  (insert "/Generate AI Artifact ")
  (read-string "Problem description: "))

(define-key global-map (kbd "C-c c g") 'copilot-generate-artifact)
```

### 4. Emacs Workflow

```
1. C-c c g → Prompted for description

2. Type: "create a security code review agent"

3. VS Code opens with chat pre-filled

4. Agent is generated

5. C-c f f → Find file: .github/agents/security-pr-reviewer.agent.md

6. Edit in Emacs with inline Copilot suggestions
   - Tab to accept suggestions
   - M-[ / M-] to cycle through suggestions
```

---

## IDE-Agnostic Workflows

### 1. Terminal-Based Generation

Generate artifacts from the command line (works in any IDE):

```bash
# 1. Set up alias in ~/.bashrc or ~/.zshrc
alias gen-artifact="code . --command \"vscode.executeCommand('workbench.action.chat.open')\""

# 2. Use from terminal
gen-artifact
# VS Code opens with chat ready

# 3. Type the artifact description
```

### 2. Git Hook Integration

Automate artifact generation on specific triggers:

Create `.git/hooks/post-commit`:

```bash
#!/bin/bash
# Post-commit hook: Generate test suite if tests.txt added

if git diff HEAD~1 HEAD --name-only | grep -q "tests\.txt"; then
  echo "tests.txt detected. Generate test suite?"
  read -p "Generate tests (y/n)? " -n 1 -r
  echo
  if [[ $REPLY =~ ^[Yy]$ ]]; then
    code . --command "vscode.executeCommand('workbench.action.chat.open')"
  fi
fi
```

Make executable:
```bash
chmod +x .git/hooks/post-commit
```

### 3. Makefile Integration

Add artifact generation to your build:

```makefile
.PHONY: generate-agent
generate-agent:
	@read -p "Agent description: " desc; \
	code . --command "vscode.executeCommand('workbench.action.chat.open')" && \
	echo "Type: /Generate AI Artifact $$desc"

.PHONY: generate-skill
generate-skill:
	@read -p "Skill description: " desc; \
	code . --command "vscode.executeCommand('workbench.action.chat.open')" && \
	echo "Type: /Generate AI Artifact $$desc"

.PHONY: review-artifacts
review-artifacts:
	@cat .github/ARTIFACT_LIBRARY.md
```

Usage:
```bash
make generate-agent
# Prompts for description, opens chat
```

### 4. Task Runner Integration

In package.json (Node projects):

```json
{
  "scripts": {
    "generate:agent": "code . --command \"vscode.executeCommand('workbench.action.chat.open')\"",
    "generate:skill": "code . --command \"vscode.executeCommand('workbench.action.chat.open')\"",
    "review:artifacts": "cat .github/ARTIFACT_LIBRARY.md && code ."
  }
}
```

Usage:
```bash
npm run generate:agent
# Opens VS Code chat
```

---

## IDE-Specific Best Practices

### VS Code

✅ **Strengths:**
- Native Copilot Chat integration
- Full AI Artifact Generator support
- Context from entire workspace
- Inline completions while typing

**Recommended workflow:**
- `Cmd+L` for quick questions
- `Cmd+Shift+G` for artifact generation
- Edit generated files directly in editor
- Use Source Control for git integration

### JetBrains IDEs (IntelliJ, PyCharm, etc.)

✅ **Strengths:**
- Excellent code navigation (⌘B)
- Advanced refactoring tools
- Language-specific context

**Recommended workflow:**
- Use JetBrains for exploration and editing
- `Cmd+Shift+A` to open Chat
- Generate artifacts for each language idiom (Kotlin, Python, etc.)
- Use IDE's built-in test generation where available

### Vim/Neovim

✅ **Strengths:**
- Fast, keyboard-driven
- Lightweight
- Excellent for remote editing

**Recommended workflow:**
- Use Neovim for code editing
- Keep VS Code in split/tmux window for chat
- Generate artifacts in VS Code
- Edit generated artifacts in Neovim

### Emacs

✅ **Strengths:**
- Highly customizable
- Powerful for text manipulation
- Excellent org-mode integration

**Recommended workflow:**
- Use Emacs for editing and navigation
- Shell command to open VS Code chat
- Edit generated artifacts in Emacs
- Track artifacts in org-mode

---

## Troubleshooting

### "Chat won't open (Cmd+L not working)"

**VS Code:**
```bash
# Check keybinding
Code → Preferences → Keyboard Shortcuts → Search "chat.open"
# Should show: Cmd+L bound to "Chat: Open"

# If missing, add manually:
Code → Preferences → Keybindings (JSON) → Add:
{
  "key": "cmd+l",
  "command": "workbench.action.chat.open"
}
```

### "Copilot extension not recognized"

```bash
# Reinstall extension
code --uninstall-extension GitHub.copilot
code --uninstall-extension GitHub.copilot-chat
code --install-extension GitHub.copilot
code --install-extension GitHub.copilot-chat

# Restart VS Code
```

### "Generated file appears in wrong location"

Check `.github/agents/` and `.github/skills/` directories exist:

```bash
mkdir -p .github/agents
mkdir -p .github/skills
```

### "Vim/Neovim suggestions not appearing"

```lua
-- ~/.config/nvim/init.lua
-- Verify copilot is enabled
require('copilot').setup({
  suggestion = { enabled = true, auto_trigger = true }
})

-- Trigger manually if auto_trigger = false
vim.keymap.set('i', '<C-Space>', 'copilot#Next()', { expr = true })
```

### "JetBrains chat not opening"

```
1. Settings/Preferences → Plugins
2. Ensure "GitHub Copilot" is installed and enabled
3. Settings/Preferences → Tools → GitHub Copilot
4. Ensure "Enable GitHub Copilot" is checked
5. Restart IDE
```

### "Context not picked up from open files"

```bash
# Make sure VS Code recognizes your project:
1. File → Open Folder → Select your project root
2. Verify .github/AGENTS.md exists in project root
3. Close and reopen project

# For JetBrains:
1. Project → Open → Select project root
2. Wait for indexing to complete
```

### "Agent generation is slow (>60 seconds)"

```bash
# Check Ollama status
curl http://localhost:11434/api/tags

# If Ollama is running but slow:
1. Reduce model complexity: mistral instead of llama2:13b
2. Reduce max_tokens in ollama.config.json
3. Check available VRAM: nvidia-smi
4. Switch to NLP-only strategy if LLM not needed

# In VS Code settings:
"artifactGenerator.strategy": "npl"  // Skip LLM
```

---

## Multi-IDE Workflow (Recommended for Teams)

Use **each IDE for what it does best**:

```
┌─────────────────────────────────────────────────┐
│                  VS Code                        │
│  - AI Artifact Generation (Cmd+L)               │
│  - Code completion & suggestions                │
│  - Git integration                              │
└─────────────────────────────────────────────────┘
              ↓
        .github/agents/
        .github/skills/
              ↓
┌─────────────────────────────────────────────────┐
│               PyCharm / IntelliJ                │
│  - Editing generated Python/Kotlin/Java code    │
│  - Advanced refactoring                         │
│  - Language-specific inspections                │
└─────────────────────────────────────────────────┘
              ↓
        src/ directory
              ↓
┌─────────────────────────────────────────────────┐
│            Vim / Neovim (Terminal)              │
│  - Fast remote editing                          │
│  - Quick fixes and tweaks                       │
│  - SSH sessions                                 │
└─────────────────────────────────────────────────┘
```

**Example:** Generate security agent in VS Code, refactor in PyCharm, edit rules in Neovim over SSH.

---

## Next Steps

1. **Choose your primary IDE**: VS Code (recommended), JetBrains, or Vim/Neovim
2. **Install required extensions/plugins**
3. **Configure keybindings** for quick access
4. **Set up project context** (.github/AGENTS.md, .vscode/settings.json)
5. **Generate your first artifact** using the IDE's chat interface
6. **Create aliases/tasks** for frequent generation workflows
7. **Document your team's IDE preferences** in CONTRIBUTING.md

---

## Related Resources

- [VS Code Copilot Chat Docs](https://code.visualstudio.com/docs/copilot/copilot-chat)
- [GitHub Copilot Docs](https://docs.github.com/en/copilot)
- [Neovim Copilot Plugin](https://github.com/github/copilot.vim)
- [JetBrains AI Assistant](https://www.jetbrains.com/help/idea/ai-assistant.html)
- [AI Artifact Generator — Usage Guide](AI_artifact_generator_guide.md)
