# Article 5: Setting Up Claude in VS Code and Terminal — A Complete Configuration Guide

> *From zero to a fully configured Claude development environment — VS Code extensions, Claude Code CLI, shell integration, and model selection workflows.*

---

## Introduction

Knowing about Claude is one thing. Having it integrated into every step of your development workflow is another. This article covers the complete setup: Claude Code in the terminal, VS Code integration, API key configuration, and how to wire it all together into a productive daily workflow.

By the end of this article you'll have:
- Claude Code running in your terminal
- Claude integrated in VS Code
- A shell alias setup for daily use
- A model selection strategy for your daily workflow

---

## 1. Prerequisites

Before we begin:

```bash
# Verify Node.js version (18+ required for Claude Code)
node --version  # Should be v18.0.0 or higher

# If you need to update Node.js:
# Using nvm (recommended):
nvm install 20
nvm use 20

# Verify npm
npm --version
```

You'll also need an Anthropic account and API key. Get one at: [console.anthropic.com](https://console.anthropic.com)

---

## 2. Installing Claude Code

Claude Code is Anthropic's official CLI tool for agentic coding in the terminal. It's the most powerful way to interact with Claude for development tasks.

### Installation

```bash
# Install globally via npm
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

### Authentication

```bash
# Start Claude Code (will prompt for authentication)
claude

# OR set API key as environment variable
export ANTHROPIC_API_KEY="sk-ant-..."

# For persistent configuration, add to your shell profile:
# ~/.bashrc, ~/.zshrc, or ~/.profile
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

> **Security note:** Never commit your API key to version control. Use environment variables or a secrets manager. Add `.env` to `.gitignore`.

### First Run

```bash
# Navigate to your project
cd /path/to/your/project

# Start Claude Code
claude

# You'll see the interactive REPL:
# ╔═══════════════════════════════════════╗
# ║          Claude Code v2.x.x           ║
# ╚═══════════════════════════════════════╝
# >
```

---

## 3. Claude Code — Core Usage Patterns

### Interactive Mode (REPL)

```bash
# Start interactive session in current directory
claude

# Claude has full context of your project files
> Explain what this codebase does
> Add error handling to the auth service
> Write tests for the UserRepository class
```

### Single Command Mode

```bash
# Run a specific task without entering the REPL
claude "Write a README for this project"
claude "Find all TODO comments and create a list"
claude "Review the security of the authentication module"
```

### Piping Input

```bash
# Pipe file content to Claude
cat src/auth/login.ts | claude "Review this code for security issues"

# Pipe git diff for review
git diff | claude "Summarise what changed and identify any issues"

# Pipe test output
npm test 2>&1 | claude "Explain these test failures and suggest fixes"
```

### Common Terminal Workflows

```bash
# Analyse a stack trace
cat error.log | claude "What is causing this error and how do I fix it?"

# Generate tests for a specific file
claude "Generate comprehensive Jest tests for src/services/payment.ts"

# Review before committing
git diff --staged | claude "Code review this diff. Flag any issues before I commit."

# Explain an unfamiliar codebase
claude "I'm new to this codebase. Explain the overall architecture and the main data flow."
```

---

## 4. Selecting Models in Claude Code

```bash
# Specify model explicitly
claude --model claude-opus-4-6 "Design the database schema for this project"
claude --model claude-sonnet-4-6 "Implement the user registration endpoint"
claude --model claude-haiku-4-5-20251001 "Summarise this file in 3 bullet points"

# Set default model for the session
export ANTHROPIC_MODEL="claude-sonnet-4-6"

# Or set in CLAUDE.md (covered in Article 8)
```

### Recommended Model Aliases

Add these to your `~/.zshrc` or `~/.bashrc`:

```bash
# Model shortcuts
alias claude-fast="claude --model claude-haiku-4-5-20251001"
alias claude-smart="claude --model claude-sonnet-4-6"
alias claude-pro="claude --model claude-opus-4-6"

# Task-specific aliases
alias ai-review="git diff --staged | claude-smart 'Review this diff for bugs, security issues, and code quality'"
alias ai-test="claude-smart 'Generate comprehensive tests for'"
alias ai-docs="claude-smart 'Write documentation for'"
alias ai-explain="claude-smart 'Explain this code in plain English:'"

# Quick commit message
alias ai-commit="git diff --staged | claude-fast 'Write a conventional commit message for this diff. Output only the commit message, nothing else.'"
```

---

## 5. VS Code Setup

### Option A: Continue.dev (Recommended for Full IDE Integration)

Continue is an open-source VS Code extension that connects to any LLM including Claude. It's the most powerful VS Code AI integration available.

**Installation:**
1. Open VS Code
2. Go to Extensions (`Ctrl+Shift+X` / `Cmd+Shift+X`)
3. Search for "Continue"
4. Install "Continue - Codestral, Claude, and more"

**Configuration:**
Create or edit `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Claude Sonnet 4.6",
      "provider": "anthropic",
      "model": "claude-sonnet-4-6",
      "apiKey": "sk-ant-your-key-here"
    },
    {
      "title": "Claude Haiku 4.5 (Fast)",
      "provider": "anthropic",
      "model": "claude-haiku-4-5-20251001",
      "apiKey": "sk-ant-your-key-here"
    },
    {
      "title": "Claude Opus 4.6 (Complex Tasks)",
      "provider": "anthropic",
      "model": "claude-opus-4-6",
      "apiKey": "sk-ant-your-key-here"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Claude Haiku 4.5",
    "provider": "anthropic",
    "model": "claude-haiku-4-5-20251001",
    "apiKey": "sk-ant-your-key-here"
  },
  "customCommands": [
    {
      "name": "review",
      "description": "Review current file",
      "prompt": "Review this code for bugs, security issues, and improvements. Be specific."
    },
    {
      "name": "test",
      "description": "Generate tests",
      "prompt": "Write comprehensive tests for this code. Cover happy path, edge cases, and error cases."
    },
    {
      "name": "docs",
      "description": "Generate documentation",
      "prompt": "Write clear JSDoc/docstring documentation for this code."
    }
  ],
  "contextProviders": [
    {
      "name": "code",
      "params": {}
    },
    {
      "name": "docs",
      "params": {}
    },
    {
      "name": "git",
      "params": {
        "Diff with main branch": true
      }
    },
    {
      "name": "terminal",
      "params": {}
    }
  ]
}
```

**Key Continue Features:**
- `Ctrl+L` / `Cmd+L` — Open chat panel
- `Ctrl+Shift+L` / `Cmd+Shift+L` — Add current file to context
- `Ctrl+I` / `Cmd+I` — Inline edit (highlight code → Cmd+I → describe change)
- `/review`, `/test`, `/docs` — Custom slash commands

---

### Option B: Cline (Autonomous Agent in VS Code)

Cline is an agentic VS Code extension — it can create files, run terminal commands, and complete multi-step tasks autonomously.

**Installation:**
1. Extensions → Search "Cline"
2. Install "Cline (formerly Claude Dev)"

**Configuration:**
1. Open Cline panel (sidebar)
2. Click settings icon
3. Set API Provider: Anthropic
4. Enter API key
5. Select model: `claude-sonnet-4-6`

**Cline is best for:**
- Multi-file feature implementation
- Autonomous bug fixing
- Scaffolding new projects
- Complex refactoring tasks that touch many files

---

### Option C: GitHub Copilot + Claude (Hybrid)

If your team uses GitHub Copilot, you can use it for autocomplete and Continue/Cline for chat.

```
GitHub Copilot → Inline autocomplete suggestions
Continue/Cline → Chat, code review, test generation
Claude Code CLI → Terminal tasks, batch operations
```

---

## 6. VS Code Settings for AI Development

Add to your VS Code `settings.json` (`Ctrl+Shift+P` → "Open User Settings JSON"):

```json
{
  // Better Claude Code terminal experience
  "terminal.integrated.fontSize": 14,
  "terminal.integrated.scrollback": 5000,
  
  // Git integration (helps AI tools understand changes)
  "git.autofetch": true,
  "git.confirmSync": false,
  
  // Editor settings that improve AI context
  "editor.formatOnSave": true,
  "editor.suggestSelection": "first",
  
  // Files to exclude from AI context (sensitive/irrelevant)
  "files.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/.env": true,
    "**/dist": true
  },
  
  // Recommended extensions for AI-assisted development
  "extensions.recommendations": [
    "continue.continue",
    "saoudrizwan.claude-dev",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode"
  ]
}
```

---

## 7. Environment Configuration Best Practices

### Project-Level API Key Management

```bash
# Create .env file (never commit this)
touch .env
echo "ANTHROPIC_API_KEY=sk-ant-your-key-here" >> .env

# Add to .gitignore
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore

# Load in your shell session
source .env
```

### Using a Secrets Manager (Production Approach)

```bash
# Using direnv for per-project environment management
brew install direnv
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc

# Create .envrc in project root
echo 'export ANTHROPIC_API_KEY=$(cat ~/.secrets/anthropic_key)' > .envrc
direnv allow
```

---

## 8. When to Use Which Interface

| Interface | Best For | Speed | Context Access | Autonomous? |
| :--- | :--- | :---: | :--- | :---: |
| **Claude Code REPL** | Complex multi-file tasks, architecture work | 🐢 Slow | Full codebase | ✅ Yes |
| **Claude Code — single command** | Quick tasks, scripting, CI/CD pipelines | ⚡ Fast | Piped input only | ✅ Yes |
| **Continue.dev — chat** | Interactive dev, Q&A, code explanation | 🚀 Fast | Selected files + git | ❌ No |
| **Continue.dev — inline** | In-place edits, targeted refactoring | ⚡ Fast | Current file | ❌ No |
| **Cline** | Autonomous multi-file feature development | 🔄 Varies | Full project | ✅ Yes |
| **claude.ai (browser)** | Exploration, non-code tasks, large documents | 🚀 Fast | None (paste only) | ❌ No |

---

## 9. Model Selection in Daily Workflow

Here's a practical model selection guide for your day:

```
Morning standup → Planning
├── Reviewing yesterday's PRs? → Sonnet (code review)
├── Writing today's tasks? → Haiku (quick summarisation)
└── Architecture meeting prep? → Opus (complex analysis)

Feature development
├── Scaffolding new code? → Sonnet (generation)
├── Debugging a hard bug? → Sonnet → Opus if stuck
├── Writing tests? → Sonnet (comprehensive coverage)
└── Writing docs? → Haiku → Sonnet for complex APIs

End of day
├── Commit message? → Haiku (fast, simple)
├── PR description? → Sonnet (clear, complete)
└── Release notes? → Sonnet (polished writing)
```

---

## 10. Verifying Your Setup

Run this checklist to confirm everything works:

```bash
# 1. Verify Claude Code CLI
claude --version
echo "✓ Claude Code installed"

# 2. Verify API key
claude "Say 'API key works!' and nothing else"
echo "✓ Authentication working"

# 3. Test project context
cd your-project
claude "List the main files in this project"
echo "✓ Project context working"

# 4. Test piping
echo "function add(a, b) { return a + b }" | claude "Write a test for this"
echo "✓ Piping working"

# 5. Verify VS Code extension (Continue)
# Open VS Code → Cmd+L → Type "hello" → Confirm response
echo "✓ VS Code integration working"
```

---

## Summary

You now have a fully configured Claude development environment:
- **Claude Code CLI** for terminal-based agentic development
- **Continue.dev** for VS Code chat and inline editing
- **Cline** for autonomous multi-file tasks
- **Shell aliases** for common AI-assisted workflows
- **Model selection strategy** matched to task types

In the next article, we'll explore how each role — developer, QA engineer, and business analyst — can build powerful workflows on top of this setup.

---

*Next: Article 6 — How to Utilise Claude for Every Role: DEVs, QAs, and BAs*
