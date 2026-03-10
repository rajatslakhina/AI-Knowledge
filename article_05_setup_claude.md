# Article 5: Setting Up Claude in VS Code and Terminal — The Complete 2026 Configuration Guide

> *Claude Code 2.x is not a chatbot you run in a terminal. It's a development platform. This guide covers the full setup — installation, VS Code integration, Skills, Hooks, Subagents, Agent Teams, model selection, and a production-ready project structure that makes Claude work like a developer who's been on your team for months.*

---

## Introduction

The version of Claude Code covered in most tutorials online is Claude Code 1.x — a capable coding assistant you chat with in a terminal. Claude Code 2.x is something different: a layered agent platform with persistent task management, custom sub-agents, lifecycle hooks, reusable skills, and multi-agent collaboration teams.

Getting the basics installed takes 5 minutes. Getting fully configured — with the right project structure, custom agents per role, hooks wiring into your CI, and Skills encoding your team's workflows — takes a few hours and pays back every single day.

This article covers both levels. By the end you'll have:
- Claude Code 2.x installed and updated
- Native VS Code extension configured with all sessions visible
- A `.claude/` project structure with Skills, Hooks, and custom Subagents
- Agent Teams enabled and ready for complex tasks
- Shell aliases for your daily workflow
- A clear model selection strategy for every type of task

---

## 1. Prerequisites

```bash
# Node.js 18+ is required
node --version   # Must be v18.0.0 or higher

# Recommended: Node 20 LTS via nvm
nvm install 20 && nvm use 20

# Verify npm
npm --version
```

You need an Anthropic account and API key from [console.anthropic.com](https://console.anthropic.com).

For the Claude Pro / Max subscription path (no API key needed), sign in with your Anthropic account when prompted on first run.

---

## 2. Installing Claude Code 2.x

### Install or Update

```bash
# Fresh install
npm install -g @anthropic-ai/claude-code

# Already installed? Update to latest 2.x
claude update

# Confirm you're on 2.x
claude --version
# Should output: 2.1.x or higher
```

> If `claude --version` shows `1.x`, run `claude update` — Claude Code ships frequent updates and 2.x includes fundamental architecture changes.

### Native Installer (Alternative — Faster Startup)

The native installer produces a self-contained binary with faster startup and better terminal rendering performance, especially on macOS:

```bash
# macOS
curl -fsSL https://code.claude.com/install.sh | sh

# Verify
claude --version
```

### Authentication

```bash
# Interactive login (recommended — handles both API key and Pro/Max subscriptions)
claude

# API key via environment variable
export ANTHROPIC_API_KEY="sk-ant-your-key-here"

# Persist in shell profile
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

> **Security:** Never commit your API key. Use environment variables or direnv (covered in Section 7). Add `.env` to `.gitignore`.

### First Run

```bash
cd /path/to/your/project
claude
# Claude Code reads your project directory, loads CLAUDE.md if present,
# activates any Skills in .claude/skills/, and starts the REPL.
```

---

## 3. The Claude Code 2.x Interface — Full Reference

### The REPL and Keyboard Shortcuts

Claude Code 2.x ships a significantly richer interface than 1.x. These shortcuts are worth memorising:

```
Navigation & Input
──────────────────────────────────────────────────
Shift+Enter        New line without submitting (v2.1.0+)
Ctrl+S             Stash current prompt (save draft, retrieve later)
↑ / ↓              Browse prompt history
Ctrl+C / ESC       Interrupt current operation

Session Management
──────────────────────────────────────────────────
/resume            Resume a previous session (shows up to 50)
/fork              Fork current conversation to a new branch
/rename            Rename current session (updates terminal tab title)
/tag               Tag a session for retrieval
/teleport          Move CLI session to claude.ai web UI

Planning & Tasks
──────────────────────────────────────────────────
Shift+Tab          Toggle Plan Mode (Claude plans before acting)
/plan [text]       Enter plan mode with optional description
/tasks             View current task list
Ctrl+T             View shared task list (Agent Teams)

Agents & Background Work
──────────────────────────────────────────────────
Ctrl+F             Kill all background agents (two-press confirm)
&                  Prefix a message to run as background task (web)

Agent Teams (requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)
──────────────────────────────────────────────────
Shift+T            Toggle team view
Shift+Down         Cycle through teammates

Skills & Commands
──────────────────────────────────────────────────
/[skill-name]      Invoke a skill
/agents            Open agent management UI
/mcp               Open MCP server management (VS Code)
/compact           Manually trigger context compaction
/copy              Copy output (w key writes directly to file)
```

### Running Single Commands (Non-Interactive)

```bash
# Run a task without entering the REPL (-p flag)
claude -p "Write a README for this project"
claude -p "Find all TODO comments and summarise them"

# Headless mode — optimised for CI/CD, skips startup API calls
claude --headless -p "Run the linter and report issues"

# Pipe input
git diff | claude -p "Summarise changes and flag any issues"
git diff --staged | claude -p "Code review this diff before I commit"
cat error.log | claude -p "Debug this stack trace"
npm test 2>&1 | claude -p "Explain these test failures and suggest fixes"
```

### Single-Command CI/CD Integration

```bash
# In GitHub Actions or any CI pipeline
- name: AI Code Review
  run: |
    git diff ${{ github.base_ref }}...${{ github.sha }} | \
    claude --headless -p \
      "Review this PR diff. Flag security issues or logic bugs.
       Exit 1 if critical issues found, else exit 0."
```

---

## 4. Plan Mode — The Most Important Habit in Claude Code 2.x

Plan Mode is activated with `Shift+Tab` before typing your prompt. It instructs Claude to **read and explore the codebase first, then produce a written plan for your review, before touching a single file**.

Without Plan Mode, Claude starts implementing immediately — useful for trivial tasks, wrong direction for anything complex.

```
Without Plan Mode:
You:    "Add Google OAuth login"
Claude: [immediately edits 8 files — 40% chance of wrong pattern or approach]

With Plan Mode (Shift+Tab first):
You:    "Add Google OAuth login"
Claude: [reads auth.service.ts, package.json, existing routes, User model]
        → Plan:
          1. Install passport + passport-google-oauth20
          2. Add GoogleStrategy in src/services/auth.service.ts
          3. Add /auth/google and /auth/google/callback to auth.routes.ts
          4. Add googleId field to User Prisma model + migration
          5. Update .env.example with GOOGLE_CLIENT_ID / SECRET

You:    [add comment: "Use our AppError class for errors, not passport defaults"]
Claude: [proceeds with your adjustment incorporated]
```

**Rule:** Use Plan Mode for any task touching 3+ files, any database change, or anything where the right approach isn't immediately obvious.

You can also enter plan mode with a description directly:

```bash
/plan refactor the payment service to support multi-currency
# Enters plan mode and immediately begins planning
```

---

## 5. The `.claude/` Project Structure

A properly configured project has this structure:

```
your-project/
├── CLAUDE.md                    ← Project context (always loaded)
├── .claude/
│   ├── settings.json            ← Hooks, MCP servers, permissions
│   ├── skills/                  ← Reusable slash-command workflows
│   │   ├── code-review.md
│   │   ├── feature-scaffold.md
│   │   ├── security-audit.md
│   │   └── pr-description.md
│   ├── agents/                  ← Custom subagent definitions
│   │   ├── security-reviewer.md
│   │   ├── test-writer.md
│   │   └── quick-summariser.md
│   └── context/                 ← Additional domain/tech context
│       ├── domain-model.md
│       └── api-contracts.md
```

And globally:

```
~/.claude/
├── CLAUDE.md                    ← Personal preferences (all projects)
├── agents/                      ← Personal subagents (all projects)
│   └── security-reviewer.md
└── backups/                     ← Auto-backups of config (v2.x)
```

---

## 6. Skills — Reusable Workflow Slash Commands

Skills are Markdown files in `.claude/skills/`. Each becomes a `/slash-command`. Claude auto-activates them when it detects a relevant task, or you invoke them explicitly. Since v2.1.0 they hot-reload — edit a skill file and the change takes effect immediately, no restart.

### Creating Your First Skill

```markdown
<!-- .claude/skills/code-review.md -->
---
name: code-review
description: Comprehensive code review — security, correctness, performance
tools: Read, Glob, Grep
model: claude-sonnet-4-6
---

Perform a thorough code review of the specified file or diff.

Review in this exact order:
1. **Security** — injection, auth bypass, data exposure, CSRF/XSS
2. **Correctness** — logic errors, off-by-one, null handling, race conditions
3. **Performance** — N+1 queries, unnecessary allocations, blocking operations
4. **Code Quality** — naming, complexity, DRY violations, SOLID principles
5. **Test Coverage** — missing tests for the issues you've found

For each finding output:
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- File:Line
- Issue description (1-2 sentences)
- Exact fix (show the corrected code)

End with: overall quality score 1–10 and one-line summary.
```

```markdown
<!-- .claude/skills/feature-scaffold.md -->
---
name: feature-scaffold
description: Scaffolds a new feature following project architecture patterns
---

Scaffold a new feature module. Steps:
1. Read CLAUDE.md for architecture conventions
2. Read src/services/auth.service.ts as the reference implementation
3. Create files in this order:
   - src/validators/$0.validator.ts     (Zod schema)
   - src/repositories/$0.repository.ts  (DB access)
   - src/services/$0.service.ts         (business logic)
   - src/controllers/$0.controller.ts   (route handler)
   - src/routes/$0.routes.ts            (Express routes)
4. Register route in src/app.ts
5. After each file: run npx tsc --noEmit to catch type errors
6. Create src/__tests__/$0.test.ts with scaffold test structure

Feature name: $0
```

```markdown
<!-- .claude/skills/pr-description.md -->
---
name: pr-description
description: Generates a PR description from current branch diff
tools: Bash, Read
---

Generate a pull request description for this branch.

Steps:
1. Run: git log main...HEAD --oneline
2. Run: git diff main...HEAD --stat
3. Read any changed files relevant to understanding the change

Output in this format:

## What
[2-3 sentences: what changed and what it enables]

## Why
[1-2 sentences: the business/technical reason]

## How to Test
1. [Step-by-step testing instructions]
2. [...]

## Checklist
- [ ] Tests added/updated
- [ ] CLAUDE.md updated if architecture changed
- [ ] No hardcoded values or credentials
- [ ] Breaking changes documented
```

### Invoking Skills

```bash
# Direct slash command
/code-review src/controllers/payment.controller.ts

# With argument shorthand ($0 = first argument, v2.1.0+)
/feature-scaffold subscription-billing

# Auto-activation — Claude detects the task matches and loads the skill
"Review the auth module for security vulnerabilities"
# Claude auto-activates code-review skill

# Generate PR description
/pr-description
```

---

## 7. Custom Subagents — Specialists With Their Own Context

Subagents are separate Claude instances with their own context window, tools, model, and permissions. They're defined in `.claude/agents/` (project-level) or `~/.claude/agents/` (global). Use them to assign different models to different types of work — Opus for security review, Haiku for summarisation — automatically, without manual switching.

### Creating Subagents

```markdown
<!-- ~/.claude/agents/security-reviewer.md -->
---
name: security-reviewer
description: Adversarial security review of code changes
model: claude-opus-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a senior security engineer performing an adversarial code review.
Assume the attacker has read the full source code.

Focus on:
- Authentication and authorisation flaws
- Injection vulnerabilities (SQL, command, LDAP, XPath)
- Insecure Direct Object References (IDOR)
- Sensitive data exposure (credentials, PII, tokens in logs)
- CSRF/XSS/SSRF vulnerabilities
- Race conditions in concurrent operations
- Insecure deserialisation

For every finding:
- Severity: CRITICAL / HIGH / MEDIUM / LOW (CVSS-informed)
- File:Line reference
- Attack scenario — how would this be exploited?
- Remediation — exact code fix

Be thorough. One missed critical is worse than ten false positives.
```

```markdown
<!-- ~/.claude/agents/quick-summariser.md -->
---
name: quick-summariser
description: Fast summary of files, PRs, or meeting notes
model: claude-haiku-4-5-20251001
tools: Read
---

Summarise the provided content concisely.
Output: maximum 5 bullet points.
Each bullet: one sentence, plain language, no jargon.
```

```markdown
<!-- .claude/agents/test-writer.md -->
---
name: test-writer
description: Generates comprehensive Jest/Playwright tests for new code
model: claude-sonnet-4-6
tools: Read, Glob, Write
---

You are a QA engineer writing tests for newly implemented code.

For each piece of code:
1. Read the implementation and understand all code paths
2. Write tests covering:
   - Happy path (valid input, expected output)
   - Input validation (null, undefined, wrong type, boundary values)
   - Error cases (thrown exceptions, rejected promises)
   - Edge cases specific to this logic

Use our test patterns:
- Jest describe/it blocks
- AAA pattern (Arrange, Act, Assert)
- Factories from src/__tests__/factories/
- jest.mock() for external dependencies

Filename: src/__tests__/[source-filename].test.ts
Coverage target: 100% of branches.
```

### Invoking Subagents

```bash
# Explicit invocation in REPL
> Spawn a security-reviewer subagent on src/controllers/payment.controller.ts

# Background (non-blocking — keep working while it runs)
> Run a security-reviewer subagent on the entire auth module in the background.
  Continue with the current task. Notify me when it's done.

# Automatic — Claude spawns subagents based on description matching
"Write comprehensive tests for the subscription service"
# → Claude auto-spawns test-writer subagent
```

---

## 8. Hooks — Automating the Development Lifecycle

Hooks are scripts (or HTTP calls) that fire automatically at points in Claude Code's execution. Once set up, they wire Claude into your existing toolchain: linting, test running, Slack notifications, audit logging.

### The `.claude/settings.json` Hooks Configuration

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix ${CLAUDE_TOOL_INPUT_PATH} --quiet 2>/dev/null || true",
            "async": true
          }
        ]
      }
    ],

    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/scripts/run-affected-tests.js",
            "async": false
          }
        ]
      }
    ],

    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/scripts/validate-bash.sh"
          }
        ]
      }
    ]
  }
}
```

### The Three Hooks Every Project Should Have

**Hook 1 — Auto-lint on every file write:**

```bash
# .claude/scripts/auto-lint.sh
#!/bin/bash
FILE_PATH="$1"
if [[ "$FILE_PATH" == *.ts || "$FILE_PATH" == *.tsx ]]; then
  npx eslint --fix "$FILE_PATH" --quiet 2>/dev/null || true
fi
```

**Hook 2 — Run affected tests after Claude finishes a turn:**

```javascript
// .claude/scripts/run-affected-tests.js
const { execSync } = require('child_process');
const { existsSync } = require('fs');

// Claude Code sets this env var with files changed in the turn
const changedFiles = (process.env.CLAUDE_CHANGED_FILES || '').split(',').filter(Boolean);

const testFiles = changedFiles
  .filter(f => f.startsWith('src/') && !f.includes('__tests__'))
  .map(f => f.replace('src/', 'src/__tests__/').replace('.ts', '.test.ts'))
  .filter(f => existsSync(f));

if (testFiles.length > 0) {
  console.log(`Running tests for: ${testFiles.join(', ')}`);
  try {
    execSync(`npx jest ${testFiles.join(' ')} --passWithNoTests`, { stdio: 'inherit' });
  } catch {
    process.exit(1); // Signal failure back to Claude
  }
}
```

**Hook 3 — Slack notification for long tasks (async, non-blocking):**

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
            "async": true
          }
        ]
      }
    ]
  }
}
```

---

## 9. Agent Teams Setup

Agent Teams (released Feb 2026) allow multiple Claude instances to collaborate on a shared task list, challenging each other's findings and working in parallel.

```bash
# Enable in your shell profile
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
source ~/.zshrc

# Start a team in the REPL
claude
```

### Using Agent Teams

```bash
# In the REPL, after enabling:
> Create a 3-agent review team for the payment module:
  - Team Lead: coordinate and produce final action plan
  - Security Analyst (use claude-opus-4-6): auth, data exposure, injection risks
  - Performance Engineer (use claude-sonnet-4-6): N+1 queries, caching, slow paths

# Navigation shortcuts:
Shift+T        → Toggle team panel view
Shift+Down     → Cycle between teammates
Ctrl+T         → View shared task list
```

**When Agent Teams are worth the cost:**
- Parallel specialist review (security + performance + correctness simultaneously)
- Large feature implementations with independent frontend/backend workstreams
- Large codebase migrations where modules can be converted concurrently

**When to stick with a single agent:**
- Any task a well-prompted Sonnet 4.6 can handle cleanly (the vast majority)
- Sequential tasks with no parallelism opportunity
- Cost-sensitive workflows — Agent Teams multiply token consumption

---

## 10. Tasks — Persistent Work Across Sessions

Tasks (introduced Jan 2026) manage work that spans multiple sessions or exceeds a single context window. Unlike a flat todo list, Tasks use a DAG (Directed Acyclic Graph) — dependencies are enforced, so Claude won't test code it hasn't built yet.

Tasks are stored on the local filesystem (`~/.claude/tasks`) and survive context compaction and session restarts.

```bash
# In the REPL
> Break this epic into a Tasks DAG and implement it in dependency order:

  Epic: Add multi-currency support to checkout.

  Create tasks for:
  1. Prisma schema update (currency fields)
  2. CurrencyService (exchange rate fetching) — depends on 1
  3. OrderRepository updates — depends on 1
  4. CheckoutController — depends on 2, 3
  5. Frontend checkout form — depends on 4
  6. Integration tests — depends on 4
  7. API documentation — depends on 4

  Don't start a task until its dependencies are marked complete.
  Run TypeScript check after each file is written.
```

View and manage tasks:
```bash
/tasks          # View current task list with dependency status
Ctrl+T          # View shared task list (in Agent Teams)
```

---

## 11. Context Compaction — Infinite Sessions

Context compaction automatically summarises earlier conversation history when the context window approaches its limit. Long development sessions — spanning hours of tool calls, file reads, and edits — no longer hit a wall and lose context.

```bash
# Compaction is automatic. Trigger manually if needed:
/compact

# Summarise from a specific message point:
# Click the message in the conversation → "Summarise from here"
```

Compaction is especially valuable for Tasks-driven feature implementations: even if context compacts mid-work, Claude picks up from the last completed task automatically.

---

## 12. Git Worktrees — Parallel Branch Work

Claude Code 2.x has native worktree support, letting you work on multiple branches simultaneously without context pollution between them.

```bash
# Ask Claude to work in a worktree
> Create a worktree for feature/payment-refactor and implement 
  the PaymentService refactor there without affecting the main directory

# Subagents with worktree isolation (each agent gets its own branch)
# In agent definition frontmatter:
# subagentIsolation: worktree
```

This is powerful for Agent Teams: one agent builds the backend API in `feature/backend`, another builds the frontend in `feature/frontend`, both in isolated worktrees. You review and merge both.

---

## 13. VS Code Integration

### Option A: Claude Code Native VS Code Extension (Recommended — 2026)

Claude Code now ships a first-party VS Code extension. Install it directly from the VS Code marketplace.

```
Extensions (Ctrl+Shift+X) → Search "Claude Code" → Install
```

**What it adds over the terminal:**

- **Spark icon in the VS Code activity bar** — lists all your active Claude Code sessions; sessions open as full editor tabs
- **Inline diffs** — file edits show as native VS Code diffs before acceptance
- **Plan view** — `/plan` produces a full Markdown document in VS Code with comment support
- **MCP management dialog** — `/mcp` in the chat panel opens a UI to enable/disable servers, reconnect, and manage OAuth without leaving the editor
- **Session forking** — `/fork` works natively in the VS Code panel
- **Works in Cursor, Windsurf, and Antigravity IDE** — the extension is cross-editor

**VS Code Claude Code settings to add (`settings.json`):**

```json
{
  "terminal.integrated.fontSize": 14,
  "terminal.integrated.scrollback": 10000,
  "git.autofetch": true,
  "editor.formatOnSave": true,
  "files.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/.env": true,
    "**/dist": true,
    "**/.claude/backups": true
  }
}
```

---

### Option B: Continue.dev (Best for Multi-LLM Workflows)

Continue is an open-source extension connecting VS Code to any LLM. Use it when you want to switch between Claude and other providers, or when you need its rich context provider system.

**Installation:** Extensions → Search "Continue" → Install

**`~/.continue/config.json`:**

```json
{
  "models": [
    {
      "title": "Claude Sonnet 4.6 (Default)",
      "provider": "anthropic",
      "model": "claude-sonnet-4-6",
      "apiKey": "sk-ant-your-key-here"
    },
    {
      "title": "Claude Opus 4.6 (Complex/Agents)",
      "provider": "anthropic",
      "model": "claude-opus-4-6",
      "apiKey": "sk-ant-your-key-here"
    },
    {
      "title": "Claude Haiku 4.5 (Fast)",
      "provider": "anthropic",
      "model": "claude-haiku-4-5-20251001",
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
      "description": "Security + correctness review of this file",
      "prompt": "Review this code for security vulnerabilities, logic errors, and missing error handling. Be specific with file:line references."
    },
    {
      "name": "test",
      "description": "Generate comprehensive tests",
      "prompt": "Write comprehensive Jest tests. Cover happy path, input validation, error cases, and edge cases. Use AAA pattern."
    },
    {
      "name": "explain",
      "description": "Explain for a new team member",
      "prompt": "Explain this code for a new developer joining the team. Cover: what it does, why it exists, dependencies, and any gotchas. Max 250 words."
    }
  ],
  "contextProviders": [
    { "name": "code", "params": {} },
    { "name": "git", "params": { "Diff with main branch": true } },
    { "name": "terminal", "params": {} },
    { "name": "docs", "params": {} }
  ]
}
```

**Key shortcuts:**
- `Cmd+L` — Open Continue chat panel
- `Cmd+Shift+L` — Add current file to context
- `Cmd+I` — Inline edit at cursor/selection
- `/review`, `/test`, `/explain` — Custom slash commands

---

### Option C: Cline (Autonomous Multi-File Agent)

Cline operates autonomously — it creates files, runs shell commands, and completes multi-file tasks without per-action approval.

**Install:** Extensions → "Cline (formerly Claude Dev)" → Install

**Configure:** Cline settings → API Provider: Anthropic → API key → Model: `claude-sonnet-4-6`

Best for scaffolding new projects, large autonomous refactors, and feature builds across many files.

---

### Which VS Code Option to Use

| Situation | Extension | Why |
|---|---|---|
| Daily Claude-primary workflow | **Claude Code native** | Sessions visible in editor, inline diffs, MCP management |
| Multi-LLM or multi-provider team | **Continue.dev** | Provider-agnostic, rich context system |
| Autonomous multi-file feature builds | **Cline** | Operates without per-step approval |
| Autocomplete alongside any of the above | **GitHub Copilot** | Claude Code + Copilot complement each other |

---

## 14. Shell Aliases — Daily Workflow Shortcuts

Add to `~/.zshrc` or `~/.bashrc`:

```bash
# ── Model shortcuts ──────────────────────────────────────────
alias claude-fast="claude --model claude-haiku-4-5-20251001"
alias claude-smart="claude --model claude-sonnet-4-6"   # 90%+ of tasks
alias claude-pro="claude --model claude-opus-4-6"       # Agents, hard problems

# ── Git workflow ─────────────────────────────────────────────
# AI commit message (Haiku — fast, cheap, accurate enough)
alias ai-commit='git diff --staged | claude-fast -p \
  "Write a conventional commit message for this diff. \
   Format: type(scope): description. Output ONLY the message, nothing else."'

# AI pre-commit review (Sonnet — thorough)
alias ai-review='git diff --staged | claude-smart -p \
  "Senior engineer code review. Flag: security issues, logic bugs, \
   missing error handling, test gaps. If all clear, output: LGTM"'

# AI PR description
alias ai-pr='claude-smart -p \
  "Write a PR description for the current branch vs main. \
   Sections: What, Why, How to Test, Checklist."'

# AI changelog from last tag
alias ai-changelog='git log $(git describe --tags --abbrev=0)..HEAD --oneline \
  | claude-smart -p "Write a changelog from these commits. Group by: Features, Fixes, Chores."'

# ── Development tasks ─────────────────────────────────────────
# Generate tests for a file
alias ai-test='claude-smart -p "Write comprehensive Jest tests for"'

# Explain code
alias ai-explain='claude-smart -p "Explain this code for a new team member:"'

# Debug a stack trace
alias ai-debug='claude-smart -p "Debug this error. Root cause + minimal fix:"'

# Generate docs
alias ai-docs='claude-smart -p "Write JSDoc/docstring documentation for:"'

# ── Security ──────────────────────────────────────────────────
# Spawn a background security review
alias ai-security='claude -p \
  "Spawn a security-reviewer subagent on the staged changes in the background. \
   Report findings when done."'
```

---

## 15. Environment and Secrets Configuration

```bash
# .env (never commit)
ANTHROPIC_API_KEY=sk-ant-your-key-here
SLACK_WEBHOOK_URL=https://hooks.slack.com/...
GITHUB_PERSONAL_ACCESS_TOKEN=ghp_...

# .gitignore
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore
echo ".claude/backups/" >> .gitignore

# direnv (recommended — per-project automatic env loading)
brew install direnv
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc

# .envrc in project root
export ANTHROPIC_API_KEY=$(cat ~/.secrets/anthropic_key)
direnv allow
```

---

## 16. When to Use Which Interface

| Interface | Best For | Context Scope | Notes |
|---|---|---|---|
| **Claude Code REPL (terminal)** | Complex tasks, Plan Mode, Skills, Agents | Full project | Core daily driver |
| **Claude Code in VS Code** | Inline diffs, visual plans, MCP management | Full project | Best for visual review |
| **Continue.dev chat** | Q&A, explanation, multi-provider | Selected files | Good for multi-LLM teams |
| **Continue.dev inline** | In-place edits, quick refactors | Current file | Fastest for single changes |
| **Cline** | Autonomous feature scaffolding | Full project | Best for multi-file autonomy |
| **`claude -p` headless** | CI/CD, scripting, automation | Piped input | Scriptable, non-interactive |
| **claude.ai browser** | Non-code tasks, sensitive discussions | None | No project context |
| **/teleport** | Hand off session to web or colleague | Full session history | Cross-device continuity |

---

## 17. Model Selection in Daily Workflow

```
Task → Model selection

Morning
├── PR review                     → Sonnet 4.6 (near-Opus review quality)
├── Standup summary               → Haiku 4.5  (fast, simple)
└── Architecture decision         → Opus 4.6   (deep reasoning)

Feature development
├── Feature scaffold (/feature-scaffold) → Sonnet 4.6 (79.6% SWE-bench)
├── Business logic implementation       → Sonnet 4.6
├── Complex algorithm                   → Opus 4.6 (novel reasoning)
├── Database migration                  → Sonnet 4.6 (well-defined, careful)
├── Debugging — standard                → Sonnet 4.6
├── Debugging — intermittent/concurrency → Opus 4.6
└── Computer use tasks                  → Sonnet 4.6 (72.5% OSWorld)

Tests and quality
├── Unit test generation          → Sonnet 4.6 or test-writer subagent
├── Test data generation          → Haiku 4.5 (fast, volume)
├── Security review               → security-reviewer subagent (Opus 4.6)
├── Team review (parallel)        → Agent Teams

Documentation and release
├── JSDoc / docstrings            → Haiku 4.5 (auto-hook)
├── README                        → Sonnet 4.6
├── Commit message                → Haiku 4.5 (ai-commit alias)
├── PR description                → Sonnet 4.6 (ai-pr alias)
└── Technical design doc          → Opus 4.6 (deep synthesis)

Orchestration
├── Agent Teams (parallel work)   → Opus 4.6 as lead
├── Background subagent tasks     → Match subagent's model to task type
└── CI/CD headless reviews        → Sonnet 4.6 (quality + cost)
```

---

## 18. Full Verification Checklist

Run through this after setup to confirm everything is wired correctly:

```bash
# 1. Version check
claude --version   # Must be 2.x

# 2. API authentication
claude -p "Respond with: setup verified"

# 3. Project context
cd your-project
claude -p "List the top-level files in this project"

# 4. Plan Mode (visual check)
# Press Shift+Tab in REPL → confirm "Plan Mode" indicator appears

# 5. Skills
claude
> /code-review src/index.ts   # Should run the skill, not a blank response

# 6. Subagents
> Spawn the quick-summariser subagent on README.md

# 7. Hooks (check lint runs after a file edit)
> Edit src/index.ts to add a blank line at the end
# → ESLint should run automatically (check terminal output)

# 8. Agent Teams
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude
# Press Shift+T → confirm team panel appears

# 9. VS Code extension
# Open VS Code → click Spark icon in activity bar → confirm sessions listed

# 10. Shell aliases
ai-commit   # Confirm commit message generated from staged diff
```

---

## Summary

You now have a production-grade Claude Code 2.x development environment:

- **Claude Code 2.x** — installed, updated, with full keyboard shortcut reference
- **Plan Mode** — `Shift+Tab` before any non-trivial task
- **`.claude/` project structure** — Skills, Agents, Hooks, context files
- **Custom Subagents** — Opus for security, Haiku for summarisation, Sonnet as default
- **Hooks** — auto-lint, auto-test, Slack notifications wired into the lifecycle
- **Agent Teams** — enabled for parallel specialist workflows
- **Tasks** — persistent DAG-based work management across sessions
- **Native VS Code extension** — sessions visible in editor with inline diffs
- **Continue.dev** — for multi-provider or multi-LLM team environments
- **Shell aliases** — daily workflow shortcuts for commit, review, PR, changelog
- **Model selection strategy** — Sonnet 4.6 default, Haiku for volume, Opus for agents

In the next article we go deep on role-specific workflows — concrete, copy-paste-ready prompt patterns for developers, QA engineers, and business analysts.

---

*Next: Article 6 — How to Utilise Claude for Every Role: DEVs, QAs, and BAs*
