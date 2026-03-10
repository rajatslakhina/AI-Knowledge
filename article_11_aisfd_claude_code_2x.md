# Article 11: Maximising Developer Productivity with Claude Code 2.x — The Complete AISFD Playbook

> *Claude Code 2.1.0 shipped 1,096 commits and fundamentally repositioned itself from coding assistant to development infrastructure. Here's the complete playbook for AI-Assisted Software Feature Development (AISFD) — model selection, feature mastery, and daily workflows that compound over time.*

---

## Introduction

Andrej Karpathy put it bluntly in early 2026: he went from **80% manual coding + 20% agents** to **80% agent coding + 20% edits and touchups** in a matter of weeks. He wasn't alone. The inflection point was real, and for most developers it arrived with two things happening simultaneously: Claude Code 2.x's architectural maturation, and Claude Sonnet 4.6 matching near-Opus quality at Sonnet pricing.

This article is the practical guide that sits at the intersection of those two things. It covers:

1. What AISFD (AI-Assisted Software Feature Development) actually means as a workflow philosophy
2. Every major Claude Code 2.x feature — what it is, when to use it, with examples
3. A precise model selection strategy for each task in a developer's day
4. Real workflows from feature request to merged PR, powered entirely by Claude Code

This is the article you read once, then keep on your second screen.

---

## 1. What Is AISFD?

**AI-Assisted Software Feature Development (AISFD)** is a development methodology where AI agents handle the majority of implementation work while the developer operates at the level of architecture, intent, and judgment.

The mental model shift is fundamental:

```
TRADITIONAL DEVELOPMENT
Developer → writes code → reviews → ships

AISFD
Developer → defines intent + context → AI implements → Developer reviews + corrects → ships
```

The developer's job doesn't disappear — it upgrades. You stop being the one who types; you become the one who thinks clearly, provides the right context, and exercises judgment on outputs. The value moves from execution to architecture, from typing to reviewing.

**The three AISFD principles:**

1. **Context is the multiplier.** The quality of Claude's output is a direct function of the quality of context you provide (CLAUDE.md, domain model, reference files). Invest here first.

2. **Decompose, don't dump.** Complex features work better as a sequence of focused agent tasks than as a single giant prompt. Each task builds context the next one uses.

3. **Review is non-negotiable.** AISFD doesn't mean unreviewed code in production. It means faster cycles to a reviewed, tested, shipped feature.

---

## 2. Claude Code 2.x — Feature Reference

Claude Code has evolved from a terminal chatbot into a layered platform. Understanding its full capability set is the difference between using 20% of it and using 80%.

Here is the complete feature stack, in order of how they were added:

```
Claude Code Platform (2026)
├── Core Agent Loop        → file ops, bash, web, git (always available)
├── MCP Integration        → Nov 2024 (external tools + services)
├── Subagents             → Jul 2025 (parallel isolated agents)
├── Hooks                 → Sep 2025 (lifecycle automation)
├── Skills / Plugins      → Oct 2025 (reusable prompt workflows)
├── Tasks                 → Jan 2026 (persistent DAG task management)
├── Agent Teams           → Feb 2026 (collaborative multi-agent squads)
└── Context Compaction    → Feb 2026 (infinite conversations)
```

Let's go deep on each.

---

### 2.1 Core Agent Loop — The Foundation

At its core, Claude Code runs a simple but powerful loop: Claude produces a message; if it contains a tool call, the tool executes and feeds results back; no tool call ends the turn.

**The 14 built-in tools:**

| Category | Tools |
|---|---|
| File operations | Read, Write, Edit, MultiEdit, Glob |
| Search | Grep, WebSearch, WebFetch |
| Shell | Bash |
| Agent control | Task (spawn subagent), TodoRead, TodoWrite |
| Git integration | Native git awareness |
| Notebook | NotebookRead, NotebookEdit |

**Key keyboard shortcuts:**

```
Ctrl+C / ESC     → Interrupt current action
Shift+Tab        → Toggle Plan Mode (Claude plans before acting)
Ctrl+S           → Stash current prompt (save draft, come back)
Shift+Enter      → New line without submitting (v2.1.0+)
/resume          → Resume a previous session (up to 50 shown)
/fork            → Fork current conversation into a new branch
/plan [text]     → Enter plan mode with optional description
/rename          → Rename current session (updates terminal tab)
/tag             → Tag a session for later retrieval
Ctrl+F           → Kill all background agents (two-press confirm)
```

**Single-command mode (for scripting and CI):**

```bash
# Run a task without entering the REPL
claude -p "Generate a changelog from git log since the last tag"

# Pipe input
git diff main...HEAD | claude -p "Review this diff for security issues"

# Headless mode (CI/CD, skips startup API calls)
claude --headless -p "Run the test suite and summarise failures"
```

---

### 2.2 Plan Mode — Think Before Acting

One of the most important but underused features. Activate with `Shift+Tab`.

In Plan Mode, Claude reads the full context, explores the codebase, and produces a structured plan **before writing a single line of code**. You review the plan, add comments, and then approve.

```
Without Plan Mode:
You: "Add OAuth login"
Claude: [immediately starts editing 8 files] → 40% chance of wrong direction

With Plan Mode (Shift+Tab):
You: "Add OAuth login"
Claude: [reads auth.service.ts, package.json, existing routes]
        → "Here's my plan:
           1. Install passport.js + passport-google-oauth20
           2. Add GoogleStrategy to auth.service.ts
           3. Add /auth/google and /auth/google/callback routes
           4. Update User model to add googleId field
           5. Update environment variables
           Shall I proceed?"
You: [review → add note: "Use our existing AppError class, not passport errors"]
Claude: [proceeds with corrected approach]
```

**Use Plan Mode for:**
- Any feature that touches 3+ files
- Tasks involving database migrations
- Any refactoring
- Anything you're not sure about the approach for

**Tip:** `/plan fix the broken payment webhook` enters plan mode with a description and immediately starts.

---

### 2.3 Subagents — Parallelism and Isolation

Subagents are separate Claude instances with their own context window, tools, model, and permissions. Your main session spawns them, they do work, and they report back a summary.

**Why they matter:** Your main session doesn't get polluted with the noise of exploration. And they can run concurrently — while one subagent explores the auth module, another reads the database schema, and a third searches for similar patterns in the codebase.

**Built-in subagents (auto-invoked by Claude Code):**

- **Explore** — Read-only codebase navigation, uses Glob/Grep/Read. Spawned when Claude needs to understand a part of the codebase it hasn't seen yet.
- **Research** — Web search + fetching. Finds documentation, error solutions, API specs.
- **Bash** — Isolated shell execution for risky commands.

**Creating custom subagents:**

```markdown
<!-- ~/.claude/agents/security-reviewer.md -->
---
name: security-reviewer
description: Reviews code changes for security vulnerabilities
model: claude-opus-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a security engineer performing a thorough security review.
Focus on: injection vulnerabilities, authentication flaws, insecure 
direct object references, sensitive data exposure, CSRF, XSS.

For each issue:
- File and line number
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- Description of the vulnerability
- Concrete fix

Be thorough and adversarial. Assume the attacker has read the source code.
```

**Invoking subagents:**

```
# Automatic (Claude decides when to spawn)
"Analyse the entire payments module and identify all error handling gaps"

# Explicit
"Spawn a security-reviewer subagent on src/controllers/payment.controller.ts"

# Background (non-blocking)
"Run a security review of the auth module in the background. 
 Continue with the feature; notify me when it's done."
```

**Per-task model overrides (powerful for cost control):**

```markdown
<!-- ~/.claude/agents/quick-summariser.md -->
---
name: quick-summariser
description: Summarises files and meetings quickly
model: claude-haiku-4-5-20251001
tools: Read
---
You are a fast summariser. Be concise. Max 5 bullet points.
```

Now your main session uses Sonnet 4.6, but summarisation tasks automatically use Haiku — no manual switching, no wasted cost.

---

### 2.4 Hooks — Automating the Development Lifecycle

Hooks are scripts that fire automatically at specific points in Claude Code's execution. They're the difference between Claude being a tool you operate and Claude being infrastructure that runs your development process.

**Hook lifecycle events:**

| Hook | When It Fires | Use Cases |
|---|---|---|
| `PreToolUse` | Before any tool execution | Validate permissions, log actions, block risky ops |
| `PostToolUse` | After tool execution | Run linters, auto-format, notify |
| `Stop` | When Claude finishes a turn | Run tests, trigger CI, send Slack notification |
| `SubagentStart` | When a subagent is spawned | Log agent activity, inject context |
| `SubagentStop` | When a subagent finishes | Collect results, update tracking |
| `PermissionRequest` | When Claude requests new permissions | Custom approval logic |

**Example hooks in `.claude/settings.json`:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "npm run lint -- --fix ${CLAUDE_TOOL_INPUT_PATH} 2>/dev/null || true",
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
            "command": "node scripts/run-affected-tests.js",
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
            "command": "scripts/validate-bash-safety.sh"
          }
        ]
      }
    ]
  }
}
```

**HTTP hooks (new in 2.x):**

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "https://your-team-webhook.company.com/claude-events",
            "headers": {
              "Authorization": "Bearer ${TEAM_WEBHOOK_TOKEN}"
            }
          }
        ]
      }
    ]
  }
}
```

**Real-world hook patterns:**

```bash
# Hook: Auto-run affected tests after file edits
# scripts/run-affected-tests.js
const changedFiles = JSON.parse(process.env.CLAUDE_CHANGED_FILES || '[]');
const testFiles = changedFiles.map(f => f.replace('src/', 'src/__tests__/').replace('.ts', '.test.ts'));
const existing = testFiles.filter(f => fs.existsSync(f));
if (existing.length > 0) {
  execSync(`npx jest ${existing.join(' ')} --passWithNoTests`, { stdio: 'inherit' });
}
```

```bash
# Hook: Slack notification when Claude finishes a long task
# scripts/notify-slack.sh
SUMMARY="${CLAUDE_LAST_MESSAGE}"
DURATION="${CLAUDE_TURN_DURATION_SECONDS}"
if [ "$DURATION" -gt 120 ]; then
  curl -X POST "$SLACK_WEBHOOK" \
    -d "{\"text\": \"✅ Claude finished a ${DURATION}s task: ${SUMMARY}\"}"
fi
```

---

### 2.5 Skills — Reusable Prompt Workflows

Skills are Markdown files that encode repeatable workflows. They're invoked as slash commands, auto-activated by Claude when relevant, and since v2.1.0, they can spawn isolated subagents, restrict tools, hot-reload without restart, and be shared via plugins.

**Skills vs. CLAUDE.md:**
- CLAUDE.md = persistent context (always loaded, background knowledge)
- Skills = on-demand workflows (loaded when invoked or detected)

**Creating a skill:**

```markdown
<!-- .claude/skills/code-review.md -->
---
name: code-review
description: Comprehensive code review for quality, security, and performance
tools: Read, Glob, Grep
model: claude-sonnet-4-6
---

Perform a thorough code review of the specified file or diff.

Review in this exact order:
1. **Security** — injection, auth, data exposure, CSRF/XSS
2. **Correctness** — logic errors, edge cases, null handling
3. **Performance** — N+1 queries, unnecessary computations, memory leaks
4. **Code Quality** — naming, complexity, DRY violations, SOLID principles
5. **Test Coverage** — missing test cases for your findings

For each finding:
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- File:Line reference
- What the issue is
- Exactly how to fix it

At the end: overall quality score (1-10) with one-line summary.
```

```markdown
<!-- .claude/skills/feature-scaffold.md -->
---
name: feature-scaffold
description: Scaffolds a new feature following project architecture
---

Scaffold a new feature module. Steps:
1. Read CLAUDE.md to understand architecture patterns
2. Read the reference implementation in src/services/auth.service.ts
3. Create in order:
   - Zod validation schema in src/validators/[feature].validator.ts
   - Repository in src/repositories/[feature].repository.ts
   - Service in src/services/[feature].service.ts
   - Controller in src/controllers/[feature].controller.ts
   - Routes in src/routes/[feature].routes.ts
   - Register route in src/app.ts
4. After each file: run TypeScript compiler check
5. Generate test file at src/__tests__/[feature].test.ts

Feature to scaffold: $0
```

**Invoking skills:**

```
# Slash command
/code-review src/controllers/payment.controller.ts

# With argument shorthand ($0, $1 etc — new in v2.1.0)
/feature-scaffold subscription

# Auto-activation (Claude detects the task matches the skill)
"Review the auth module for security issues"  
→ Claude auto-activates code-review skill
```

**Hot reload (v2.1.0+):** Edit a skill file → changes take effect immediately. No restart.

**Skill in a `.claude/skills/` within `--add-dir` directories** are loaded automatically, enabling per-project skill libraries.

---

### 2.6 Tasks — Persistent Work Across Sessions

Tasks (introduced Jan 2026) solve the problem of work that spans multiple sessions or is too complex for a single conversation. They use a DAG (Directed Acyclic Graph) structure — tasks can have dependencies, so Claude won't start "Run Tests" until both "Build API" and "Configure Auth" are complete.

**What makes Tasks different from a todo list:**
- Stored on local filesystem (`~/.claude/tasks`) — survives context compaction
- Dependency enforcement — prevents the "hallucinated completion" error where Claude tests code it hasn't written yet
- Persist across sessions — close Claude Code, come back tomorrow, pick up where you left off
- Shared task lists in Agent Teams — multiple agents update the same list

**Activating Tasks:**

```bash
# Disable old system if needed during transition
export CLAUDE_CODE_ENABLE_TASKS=true

# In Claude Code REPL:
Ctrl+T           # View shared task list (in Agent Teams)
/tasks           # View current session tasks
```

**Real-world Tasks example:**

```
"Break this epic into tasks and implement them in order:

Epic: Add multi-currency support to the checkout flow

I want you to:
1. Create tasks for each step
2. Mark dependencies (schema before service, service before controller)
3. Implement each task in order, running tests after each
4. Don't start a task until its dependencies are complete
5. Report summary when done"
```

Claude creates:
```
Tasks:
[1] Update Prisma schema (currency, exchangeRate fields)    → no deps
[2] Create CurrencyService with exchange rate fetching      → depends on [1]
[3] Update OrderRepository for multi-currency amounts       → depends on [1]
[4] Update checkout Controller to handle currency param     → depends on [2,3]
[5] Update frontend checkout form                           → depends on [4]
[6] Write integration tests                                 → depends on [4]
[7] Update API documentation                                → depends on [4]
```

This graph is persisted. If context compaction occurs mid-task, Claude picks up from the last completed task automatically.

---

### 2.7 Agent Teams — Collaborative Multi-Agent Development

Agent Teams (released Feb 5, 2026 alongside Opus 4.6) are the most powerful and most expensive Claude Code feature. Where subagents are isolated workers that report back, Agent Teams are a collaborative squad: one Claude leads, others are teammates — they share a task list, can challenge each other's findings, and work in parallel.

**Architecture:**
```
                    Team Lead (Opus 4.6)
                    ↙       ↓        ↘
         Agent A        Agent B         Agent C
      (Backend Dev)   (Security)      (Test Writer)
      Sonnet 4.6      Opus 4.6       Sonnet 4.6
           ↓               ↓               ↓
      [shared task list] ← → [shared task list]
```

**Enabling Agent Teams:**

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude
```

**Agent Teams keyboard shortcuts:**
```
Shift+T          → Toggle team view
Shift+Down       → Cycle through teammates
Ctrl+T           → Open shared task list
```

**Creating a team in a prompt:**

```
"Create an agent team with 3 members to review and refactor the payment module:

- Team Lead: coordinate the work and produce the final report
- Security Analyst: focus on authentication, authorization, and data exposure
- Performance Engineer: identify N+1 queries, slow paths, caching opportunities

Shared goal: produce a prioritised action plan with specific code fixes.
Working files: src/services/payment.service.ts, src/repositories/payment.repository.ts"
```

**When Agent Teams are worth the cost:**
- Security + performance + correctness review simultaneously (3 specialists)
- Large feature implementations where frontend and backend can develop in parallel
- Code migration where different modules can be converted concurrently
- The famous Anthropic example: 16 agents building a C compiler in Rust — 100,000 lines over two weeks

**When NOT to use Agent Teams:**
- Anything a single well-prompted Sonnet 4.6 can handle — the token cost is real
- Sequential tasks with no parallelism opportunity
- Tasks where the overhead of coordination exceeds the benefit

---

### 2.8 Context Compaction — Infinite Conversations

Context compaction (v2.x, using the Claude 4.6 API feature) automatically summarises earlier conversation history when the context window approaches its limit. The result: effectively infinite conversations within a single session.

**Why this matters for AISFD:**
Long feature development sessions used to hit context limits and lose important history. With compaction, a session that started implementing a feature at 9am can still have relevant context at 6pm, even if hundreds of tool calls have occurred.

```bash
# Compaction is automatic — no configuration needed
# But you can trigger it manually:
/compact

# Or summarise from a specific message:
# Click the message, then select "Summarise from here"
```

**Tip:** Compaction preserves images in the summariser request (recent fix), enabling prompt cache reuse for cheaper and faster subsequent compaction.

---

### 2.9 Git Worktrees — Parallel Feature Development

Git worktrees let Claude work on multiple branches simultaneously without interfering. Combine with subagents for parallel feature development.

```bash
# Claude Code handles worktree creation and navigation
"Create a worktree for feature/payment-refactor and implement the 
 PaymentService refactor there, without touching the main working directory"
```

**Subagents with worktree isolation (new in 2.x):**

```markdown
<!-- Agent definition with worktree isolation -->
---
name: feature-implementer
description: Implements features in isolated git worktrees
subagentIsolation: worktree
---
```

This spawns the agent in a fresh worktree — any file changes are completely isolated from your main session until you review and merge.

---

### 2.10 The /teleport Feature (v2.1.0)

`/teleport` moves your active CLI session to the claude.ai web interface — or `claude --teleport` starts a web session directly. This enables:
- Handing off a terminal session to a colleague via the web UI
- Continuing a complex session on a different machine
- Accessing your session from mobile

```bash
# Teleport current session to the web
/teleport

# Or start a web session from terminal
claude --teleport
```

---

## 3. The Complete AISFD Model Selection Guide

This is the decision framework for choosing the right model for every task in your development day. Updated for Claude Code 2.x and the Sonnet 4.6 reality where near-Opus quality is available at Sonnet pricing.

### The Hierarchy

```
Task complexity
     ↑
     │  Opus 4.6 ─────────────── Multi-agent orchestration
     │                           10h+ autonomous tasks
     │                           Novel architectural problems
     │                           Highest-stakes irreversible actions
     │
     │  Sonnet 4.6 ─────────────  DEFAULT for 90%+ of development tasks
     │  (NEW DEFAULT)             Coding, debugging, review, testing, docs
     │                           Computer use (72.5% OSWorld)
     │                           Long-context analysis (1M beta)
     │                           Agent subagent work
     │
     │  Haiku 4.5 ───────────────  Speed + volume tier
     │                           Summarisation, classification
     │                           Real-time applications
     │                           Routing/triage
     ↓
Task simplicity
```

### Task-by-Task Model Reference

#### Feature Development

| Task | Model | Why | Example Command |
|---|---|---|---|
| New feature scaffold | **Sonnet 4.6** | Near-Opus code quality, 5× cheaper | `/feature-scaffold user-invitations` |
| Complex algorithm design | **Opus 4.6** | Novel reasoning needed | `"Design a conflict-free CRDT for our collaborative editing"` |
| API endpoint implementation | **Sonnet 4.6** | Pattern-following, well-defined | `"Implement POST /subscriptions following auth.controller.ts"` |
| Database migration | **Sonnet 4.6** | Well-defined, needs care | `"Write Prisma migration for adding soft-delete to orders"` |
| Frontend component | **Sonnet 4.6** | Design + code quality | `"Build a paginated data table component with sort and filter"` |
| Large codebase refactor | **Sonnet 4.6** + Tasks | Long-horizon, sequential | Use Tasks DAG for dependency management |
| Parallel module migration | **Opus 4.6** + Agent Teams | Genuinely parallel workstreams | 3 agents on 3 modules simultaneously |

#### Code Review and Quality

| Task | Model | Why | Example Command |
|---|---|---|---|
| Standard PR review | **Sonnet 4.6** | Excellent review quality | `git diff main | claude -p "Review for bugs and security"` |
| Security-critical review | **Opus 4.6** security-reviewer subagent | Adversarial depth needed | Custom subagent with Opus 4.6 |
| Team-wide review (3+ dimensions) | **Opus 4.6** Agent Teams | Parallel specialist review | Security + Perf + Correctness agents |
| Quick style/lint check | **Haiku 4.5** | Speed, simple task | Auto-hook on PostToolUse |
| Architecture review | **Opus 4.6** | Novel judgment needed | `"Review system design for [feature]"` |

#### Testing

| Task | Model | Why | Example Command |
|---|---|---|---|
| Unit test generation | **Sonnet 4.6** | Strong pattern recognition | `"Write Jest tests for PaymentService"` |
| E2E Playwright tests | **Sonnet 4.6** | Multi-step generation | `"Write Playwright tests for checkout flow"` |
| Test data factories | **Haiku 4.5** | Simple, high-volume | `"Generate 50 test users with edge-case data"` |
| Test strategy planning | **Sonnet 4.6** | Reasoning + patterns | `"Plan the test strategy for the billing module"` |
| Mutation testing analysis | **Opus 4.6** | Complex reasoning on failures | `"Analyse why these mutation tests survived"` |

#### Debugging

| Task | Model | Why | Example Command |
|---|---|---|---|
| Stack trace analysis | **Sonnet 4.6** | Strong debugging, near-Opus | `cat error.log | claude -p "Debug this"` |
| Intermittent/concurrency bug | **Opus 4.6** | Subtle reasoning required | `"This race condition only appears under load..."` |
| Performance profiling | **Sonnet 4.6** | Pattern recognition | `"Explain this Datadog flame graph"` |
| Quick syntax error | **Haiku 4.5** | Trivial, fast | Inline in editor |

#### Documentation

| Task | Model | Why | Example Command |
|---|---|---|---|
| JSDoc / docstrings | **Haiku 4.5** | Simple, high-volume | Hook on PostToolUse |
| README generation | **Sonnet 4.6** | Quality writing | `"Write README for this module"` |
| Architecture decision record | **Sonnet 4.6** | Structured reasoning | `"Write ADR for choosing Event Sourcing"` |
| API documentation | **Sonnet 4.6** | Completeness needed | `"Document all endpoints in payment.routes.ts"` |
| Long-form technical design doc | **Opus 4.6** | Deep synthesis | `"Write technical design for our new auth system"` |

#### Git and Release

| Task | Model | Why | Example Command |
|---|---|---|---|
| Commit message | **Haiku 4.5** | Fast, simple | `git diff --staged | claude-fast "Commit message"` |
| PR description | **Sonnet 4.6** | Clear, complete writing | `claude -p "Write PR description for this branch"` |
| Changelog generation | **Sonnet 4.6** | Synthesis from git log | `git log --oneline v1.2..HEAD | claude "Changelog"` |
| Release notes | **Sonnet 4.6** | Quality, stakeholder-facing | `"Draft release notes for v2.3.0"` |
| Hotfix analysis | **Sonnet 4.6** → Opus if unclear | Debugging clarity | `"Is this safe to hotfix? [diff]"` |

---

## 4. The AISFD Daily Development Workflow

This is a concrete, hour-by-hour workflow for a developer using AISFD with Claude Code 2.x.

### Morning: Sprint Planning (15 minutes)

```bash
cd my-project
claude

# 1. Understand today's tasks in context
> What is the state of feature/subscription-billing? 
  Read the PR, the related JIRA ticket in context, and the current implementation.

# 2. Break down the day's work into a Task graph
> I need to implement: coupon code validation, proration for mid-cycle upgrades,
  and dunning emails for failed payments.
  Create a Tasks DAG with correct dependencies and suggest the implementation order.

# Claude creates the task graph. You review and approve.
```

### Feature Implementation: The AISFD Loop

```bash
# Step 1: Plan before touching any files
[Shift+Tab] → Plan Mode

> Implement coupon code validation for the checkout flow.
  Reference: src/services/auth.service.ts for coding patterns.
  Constraints: Coupons can be single-use or multi-use, percentage or fixed amount,
  can expire, and can be restricted to specific plan tiers.

# Claude reads the codebase, proposes a plan.
# You review: "Good. Add validation that a coupon can't be applied to 
#              a plan it's not eligible for."
# Claude proceeds with your note incorporated.
```

```bash
# Step 2: Scaffold (let Claude do the boilerplate)
/feature-scaffold coupon

# Claude creates: validator, repository, service, controller, routes.
# TypeScript check after each file. Takes ~3 minutes.
```

```bash
# Step 3: Implement the business logic
> Implement the business logic in CouponService.
  Rules are in CLAUDE.md under "Discount Rules".
  Edge cases to handle:
  - Coupon already used by this user
  - Coupon expired (check both date and usage limit)
  - Coupon not valid for the selected plan tier
  - Concurrent redemption (race condition — use database transaction)

# Sonnet 4.6 handles this well. Near-Opus reasoning on well-specified problems.
```

```bash
# Step 4: Tests run automatically (hook)
# PostToolUse hook fires: lint runs, affected tests run.
# If tests fail, Claude sees the output and fixes immediately.
```

```bash
# Step 5: Spawn a background security review while you continue
> Run a security-reviewer subagent on the new CouponService in the background.
  Continue implementing the proration logic. Notify me when the review is done.

# You keep working. Security review happens in parallel.
# When complete, Claude reports the findings.
```

### Mid-Feature: Hard Problem Escalation

```bash
# Proration maths are complex. Escalate to Opus.
claude --model claude-opus-4-6

> The proration calculation needs to handle:
  - Mid-cycle upgrades with different billing cycles (monthly/annual)
  - Credits from partial periods at different rates
  - Tax recalculation after proration
  - Currency conversion for multi-currency accounts
  
  Design the proration engine. Start with the data model, then the algorithm.
  Show your reasoning step by step before writing code.
```

### End-of-Day: Review and Commit

```bash
# Automated pre-commit review
git diff --staged | claude -p \
  "You are a senior engineer. Review this diff.
   Flag: security issues, logic bugs, missing error handling, test gaps.
   If all clear, output only: LGTM. Otherwise list issues."
```

```bash
# If LGTM, generate commit message
git diff --staged | claude-fast \
  "Write a conventional commit message. 
   Output only the commit message, nothing else."
```

```bash
# Generate PR description
claude -p "Write a PR description for the current branch vs main.
  Format: What, Why, How to test, Screenshots placeholder."
```

---

## 5. Power Workflows — Putting It All Together

### Workflow: Full Feature from Ticket to PR (Solo)

```bash
# 1. Read the JIRA ticket (via JIRA MCP if configured)
"Read JIRA ticket PROJ-234 and explain what needs to be built"

# 2. Plan with Task graph
[Shift+Tab]
"Plan the full implementation of PROJ-234 with a Tasks DAG"

# 3. Scaffold
/feature-scaffold [feature-name]

# 4. Implement (Sonnet 4.6 default)
"Implement each task in the DAG. Run tests after each. Report blockers."

# 5. Background security review runs via hook/subagent

# 6. Auto-PR description
claude -p "Write the PR description for this branch"

# Time: ~2-4 hours vs ~2-3 days manually
```

### Workflow: Team Code Review (Agent Teams)

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

"Create a 3-agent review team for PR #247:
 - Lead: synthesise findings and produce the final review comment
 - Security Analyst (Opus 4.6): authentication, data handling, injection risks
 - Architecture Reviewer (Sonnet 4.6): SOLID, patterns, maintainability

 Post the consolidated review to GitHub via the GitHub MCP."
```

### Workflow: Incident Debug (Background + Logs MCP)

```bash
# With Datadog MCP and GitHub MCP configured:
"We have a production incident — payment service 500s.
 Tasks:
 1. Query Datadog logs for payment-service errors in the last 30 minutes
 2. Identify the error pattern and frequency
 3. Read the last 5 commits to the payment service in GitHub
 4. Correlate: which commit likely introduced this?
 5. Show me the specific code change that's the likely culprit
 6. Propose a minimal hotfix"

# What would take 30 minutes manually: ~4 minutes with Claude Code + MCPs
```

### Workflow: Continuous CI/CD Integration

```bash
# In your CI pipeline (headless mode)
- name: AI Code Review
  run: |
    git diff ${{ github.base_ref }}...${{ github.sha }} | \
    claude --headless -p \
      "Review this PR diff for: security vulnerabilities, logic errors, 
       missing tests. If critical issues found, exit 1. Else exit 0."
```

---

## 6. AISFD Anti-Patterns

Avoid these common mistakes:

**Anti-pattern 1: The Mega-Prompt**
```
Bad:  "Build the entire subscription billing system"
Good: Use Plan Mode + Tasks DAG — decompose into implementable units
```

**Anti-pattern 2: Skipping Plan Mode**
For any task > 3 files, skipping `Shift+Tab` means Claude starts coding before understanding the full picture. Costs more tokens to fix than Plan Mode costs to run.

**Anti-pattern 3: Using Opus for Everything**
Post-Sonnet 4.6, defaulting to Opus is a 5× cost overpay for equivalent output on ~90% of tasks. Audit your usage.

**Anti-pattern 4: No Hooks**
Without PostToolUse hooks running your linter and tests, Claude can go multiple steps before discovering a downstream test failure. Set up hooks early — they change the feedback loop fundamentally.

**Anti-pattern 5: No CLAUDE.md**
Claude without CLAUDE.md generates technically correct code that violates your project's patterns, architecture, and conventions. The first 2 hours you spend on CLAUDE.md saves 2 minutes per prompt for the life of the project.

**Anti-pattern 6: Accepting Without Review**
AISFD doesn't mean unreviewed code. The review is faster, not absent. Ship nothing you haven't read.

---

## 7. Claude Code 2.x Setup Checklist

```bash
# 1. Install / update to latest
npm install -g @anthropic-ai/claude-code
claude update

# 2. Verify version (should be 2.x)
claude --version

# 3. Set default model (Sonnet 4.6 is now the right default)
echo 'export ANTHROPIC_MODEL="claude-sonnet-4-6"' >> ~/.zshrc

# 4. Set up global CLAUDE.md
mkdir -p ~/.claude
touch ~/.claude/CLAUDE.md   # See Article 8 for content

# 5. Create project CLAUDE.md (see Article 8)
touch CLAUDE.md

# 6. Create skills directory
mkdir -p .claude/skills
# Add code-review.md, feature-scaffold.md, etc.

# 7. Create agents directory
mkdir -p ~/.claude/agents
# Add security-reviewer.md with Opus model

# 8. Configure hooks in .claude/settings.json

# 9. Configure MCPs in .claude/settings.json (see Article 10)

# 10. Enable Agent Teams (experimental)
echo 'export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1' >> ~/.zshrc
```

---

## Summary

Claude Code 2.x paired with Claude Sonnet 4.6 has fundamentally changed what a single developer can accomplish. The key principles:

**Use Sonnet 4.6 as your default.** It delivers near-Opus quality on 90%+ of development tasks at one-fifth the cost. The economics make sense at every scale.

**Reserve Opus 4.6 for specific scenarios:** multi-agent team orchestration, 10h+ autonomous tasks, novel reasoning problems, highest-stakes reviews. The capability difference is real but the use cases are narrower than before.

**Use Haiku 4.5 aggressively for volume.** Commit messages, summarisation, routing, quick lookups — Haiku's speed and cost profile is a perfect fit.

**Invest in infrastructure once.** CLAUDE.md, Skills, Hooks, and custom subagents are investments with compounding returns. Every hour you spend on them saves minutes per prompt indefinitely.

**Plan Mode is not optional.** For any non-trivial task, `Shift+Tab` before starting is the highest-ROI habit you can build.

The developers extracting the most value from AISFD in 2026 are the ones who treat Claude Code as infrastructure, not a chatbot. They've wired it into their CI, their hooks, their review process, and their daily workflow. They're not faster because they type less — they're faster because they've automated the mechanical parts of software development and freed their thinking for the parts that actually require it.

---

*This is Article 11 in the AI-Assisted Development series. Previous articles cover model fundamentals, setup, CLAUDE.md configuration, effective prompting, and MCP setup.*
