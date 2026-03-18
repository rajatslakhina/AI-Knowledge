# Article 12: Configuring Claude Code with Git — The Complete Integration Guide

> *Git is not a tool Claude Code sits alongside. It's infrastructure Claude Code runs on top of. This article covers every layer of the integration — from the basic commit message workflow to parallel worktree development with isolated per-branch MCPs, automated pre-commit hooks, and CI/CD pipelines powered entirely by headless Claude.*

---

## Introduction

Most developers who start with Claude Code use it for isolated tasks: explain this function, refactor that class, write a test. That's fine. But Claude Code's most compelling value isn't in isolated tasks — it's in workflows where Claude has full context of what's happening in your repository, what branch you're on, what's changed, and what needs to happen next.

This article builds that full integration: Claude Code wired end-to-end into your git workflow, from the moment you start a new feature to the moment a PR is merged.

```mermaid
graph TD
    T["📋 Ticket<br/>Assigned"] --> B["🌿 Branch<br/>Created<br/><i>/git-start-feature</i>"]
    B --> I["💻 Implementation<br/><i>Claude Code</i>"]
    I --> C["✅ Commits<br/><i>/git-commit</i><br/>Conventional format"]
    C --> R["🔍 Self-Review<br/><i>/git-review</i>"]
    R --> PR["📝 PR Created<br/><i>/git-pr</i><br/>Auto-description"]
    PR --> CI["⚙️ CI/CD<br/><i>Headless Claude</i><br/>Lint + Test + Build"]
    CI --> HRE["👥 Human<br/>Review"]
    HRE --> M["🎉 Merged!"]

    style T fill:#f5a623,stroke:#d4891a,color:#fff
    style I fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style C fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style R fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style PR fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style CI fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style M fill:#50c878,stroke:#3da360,color:#fff
```


By the end you'll have:
- CLAUDE.md configured with your team's git conventions (enforced automatically)
- Skills for every git workflow (branch creation, commit messages, PR descriptions, changelogs)
- Hooks that run quality gates on every action
- Git worktree setup for parallel branch development
- Per-worktree MCP configuration
- CI/CD integration using headless Claude
- A complete feature-delivery workflow from ticket to merged PR

---

## 1. CLAUDE.md — Encoding Your Git Conventions

Before Claude touches a single file, it needs to know your team's git rules. Add a Git Conventions section to your project `CLAUDE.md`:

```markdown
## Git Conventions

### Branch Naming
- Features:    feature/[TICKET-ID]-short-description    (e.g., feature/PROJ-123-add-oauth-login)
- Bug fixes:   fix/[TICKET-ID]-short-description        (e.g., fix/PROJ-456-session-timeout-crash)
- Hotfixes:    hotfix/[TICKET-ID]-description           (e.g., hotfix/PROJ-789-payment-null-ptr)
- Refactors:   refactor/description                     (e.g., refactor/auth-service-cleanup)
- Experiments: experiment/description                   (e.g., experiment/redis-cache-strategy)

### Commit Message Format (Conventional Commits)

```mermaid
graph LR
    subgraph FORMAT["📝 Conventional Commit Format"]
        direction TB
        F1["<b>type(scope): description</b><br/><br/>feat(auth): add OAuth2 PKCE flow<br/>fix(cart): handle empty cart checkout<br/>refactor(api): extract validation middleware<br/>test(payment): add Stripe webhook tests<br/>docs(readme): update deployment guide"]
    end
    subgraph TYPES["🏷️ Common Types"]
        direction TB
        T1["<b>feat</b> — new feature"]
        T2["<b>fix</b> — bug fix"]
        T3["<b>refactor</b> — code change"]
        T4["<b>test</b> — tests only"]
        T5["<b>docs</b> — documentation"]
        T6["<b>chore</b> — maintenance"]
    end

    style FORMAT fill:#f0f4ff,stroke:#4a90d9
    style TYPES fill:#d4edda,stroke:#28a745
```

type(scope): short description [max 72 chars]

[optional body — wrap at 72 chars]

[optional footer: BREAKING CHANGE, ticket refs]

Types: feat | fix | docs | style | refactor | perf | test | chore | ci
Scopes: auth | api | db | ui | mobile | infra | config

Examples:
feat(auth): add Google OAuth2 login flow
fix(api): handle null customer ID in invoice creation
perf(db): add index on orders.customer_id for faster lookups
test(auth): add integration tests for token refresh

### PR Rules
- Title: same format as commit message (Conventional Commits)
- All PRs target `main` unless explicitly told otherwise
- PRs require passing CI + 1 approval minimum
- Draft PRs for work-in-progress (prefix title with "Draft: " or use --draft flag)
- Link to the Jira/Linear ticket in the PR description

### Branch Protection
- `main` and `develop` are protected — never push directly
- Squash merge for feature branches
- Merge commit for release branches

### What NOT to Do (Git)
- ❌ Never commit .env files or secrets
- ❌ Never force-push to shared branches
- ❌ Never merge without CI passing
- ❌ Never commit commented-out code
- ❌ Never commit with "WIP" or "fix" as the entire message
- ❌ Never include node_modules, dist/, or build/ in commits
```

With this in CLAUDE.md, every Claude action — branch creation, commit generation, PR descriptions — automatically follows your conventions without prompting.

---

## 2. Git Skills — Reusable Slash Commands for Every Git Task

```mermaid
graph TB
    subgraph SKILLS["📁 .claude/skills/ — Git Workflow Skills"]
        direction TB
        S1["🌿 git-start-feature.md<br/>Branch creation + naming"]
        S2["✅ git-commit.md<br/>Conventional commit format"]
        S3["📝 git-pr.md<br/>PR description + checklist"]
        S4["🔍 git-review.md<br/>Self-review before PR"]
        S5["📋 git-changelog.md<br/>Auto-generated changelog"]
    end

    DEV["👨‍💻 Developer<br/>types /git-commit"] --> SKILLS
    SKILLS --> CLAUDE["🤖 Claude follows<br/>team conventions"]

    style SKILLS fill:#f0f4ff,stroke:#4a90d9
    style CLAUDE fill:#7b68ee,stroke:#5a4dbd,color:#fff
```


### Create and configure your git skills directory

```
.claude/skills/
├── git-start-feature.md     # Start a new feature from a ticket
├── git-commit.md            # Staged diff → conventional commit message
├── git-pr.md                # Branch diff → full PR description
├── git-review.md            # Review an existing PR diff
├── git-changelog.md         # Tag range → categorised changelog
└── git-cleanup.md           # Identify and clean merged branches
```

---

### Skill: `git-start-feature.md`

```markdown
<!-- .claude/skills/git-start-feature.md -->
---
name: git-start-feature
description: Create a properly named branch for a feature or fix from a ticket ID
tools: Bash
---

Start a new feature branch for ticket: $0

Steps:
1. Run: git status (confirm clean working directory — warn if dirty)
2. Run: git checkout main && git pull origin main
3. Ask me to confirm the short description for the branch name
   (derive it from the ticket title if you have it, otherwise ask)
4. Create branch: git checkout -b feature/$0-[short-description]
5. Run: git branch --show-current (confirm correct branch)
6. Output: the exact branch name created, so I can reference it

Branch naming rules (from CLAUDE.md):
- feature/ prefix for new functionality
- fix/ prefix for bug fixes
- hotfix/ prefix for production emergencies
- Ticket ID first, then hyphenated short description
- All lowercase, hyphens not underscores
```

---

### Skill: `git-commit.md`

```markdown
<!-- .claude/skills/git-commit.md -->
---
name: git-commit
description: Generate a Conventional Commits message from staged diff
tools: Bash
---

Generate a commit message for the staged changes.

Steps:
1. Run: git diff --staged --stat (understand scope of changes)
2. Run: git diff --staged (read the actual changes)
3. Run: git branch --show-current (get branch name for ticket reference)
4. Generate a Conventional Commits message:
   - type(scope): short description (max 72 chars)
   - Body only if changes are complex or non-obvious
   - Footer: ticket reference extracted from branch name if available

Rules:
- Choose the most specific type (feat > chore, fix > refactor)
- Scope = the system area changed (auth, api, db, ui, etc.)
- Description: imperative present tense ("add" not "added" or "adding")
- If multiple unrelated changes are staged: warn me and suggest splitting

Output:
- The commit message ONLY (no preamble)
- Immediately after: run `git commit -m "[message]"` — do not wait for confirmation
  UNLESS the diff contains potential issues (then flag and wait)
```

---

### Skill: `git-pr.md`

```markdown
<!-- .claude/skills/git-pr.md -->
---
name: git-pr
description: Generate a complete PR description from current branch vs main
tools: Bash, Read
---

Generate a pull request description for this branch.

Steps:
1. Run: git log main...HEAD --oneline (understand all commits)
2. Run: git diff main...HEAD --stat (understand file scope)
3. Run: git diff main...HEAD (read the actual changes)
4. Read CLAUDE.md for project context and any relevant domain rules
5. Read any files that changed significantly to understand intent

Generate the PR description in this format:

---
## What
[2–3 sentences: what this PR does and what it enables]

## Why
[1–2 sentences: the business or technical reason this change is needed]

## Changes
- [bullet list of the meaningful changes — not every file, just the substance]

## How to Test
1. [Step-by-step testing instructions from a reviewer's perspective]
2. [Include setup steps, test data needed, expected outcomes]

## Acceptance Criteria
[Tick off each AC from the ticket if ticket details are available]
- [ ] AC1
- [ ] AC2

## Checklist
- [ ] Tests added / updated
- [ ] CLAUDE.md updated if architecture changed
- [ ] No hardcoded values or credentials
- [ ] Breaking changes noted in the footer
- [ ] Ticket linked
---

Rules:
- Be specific — reviewers should understand the change without reading the diff
- Flag any unusual decisions or tradeoffs in the "Why" section
- If the diff contains something that looks risky, flag it explicitly
```

---

### Skill: `git-review.md`

```markdown
<!-- .claude/skills/git-review.md -->
---
name: git-review
description: Review a PR diff as a senior engineer before merging
tools: Bash, Read, Grep
---

Review the current branch diff vs main as a senior engineer.

Steps:
1. Run: git diff main...HEAD (read the full diff)
2. Read the changed files in context (not just the diff)
3. Check git log for context on what was intended

Review in this order:
1. **Correctness** — logic errors, off-by-one, null handling, unhandled promises
2. **Security** — injection, auth gaps, data exposure, missing validation
3. **Performance** — N+1 queries, blocking operations, missing indexes
4. **Conventions** — does the code follow CLAUDE.md rules? naming, patterns, layer boundaries?
5. **Tests** — are changed paths covered? are edge cases tested?

Output format:
- For each issue: Severity (CRITICAL/HIGH/MEDIUM/LOW) | File:Line | Issue | Fix
- End with: overall assessment — APPROVE / REQUEST CHANGES / NEEDS DISCUSSION
- Include: one-line summary of the PR's purpose based on the diff

CRITICAL and HIGH issues: always block approval.
MEDIUM and below: flag but note they are non-blocking.
```

---

### Skill: `git-changelog.md`

```markdown
<!-- .claude/skills/git-changelog.md -->
---
name: git-changelog
description: Generate a categorised changelog between two git tags
tools: Bash
---

Generate a changelog for release: $0

Steps:
1. Run: git tag --sort=-version:refname | head -5 (find recent tags)
2. Run: git log [previous-tag]...HEAD --oneline --no-merges (get all commits)
3. Run: git log [previous-tag]...HEAD --format="%H %s %b" (get full messages)

Categorise commits by Conventional Commit type:

## ✨ New Features
[feat commits — user-facing features only]

## 🐛 Bug Fixes
[fix commits]

## ⚡ Performance Improvements
[perf commits]

## 🔧 Under the Hood
[refactor, chore, ci, build commits — summarise, don't list individually]

## 📚 Documentation
[docs commits — only if substantial]

## ⚠️ Breaking Changes
[BREAKING CHANGE footer commits — always list these]

Rules:
- Write for a technical audience but not an internal one
- Skip WIP, temp, and trivial commits
- Group related commits into a single entry where appropriate
- Version: $0
- Date: today
```

---

## 3. Git Hooks — Automated Quality Gates

```mermaid
graph LR
    subgraph HOOKS["⚡ Git Hook Architecture"]
        direction TB
        PRE["🔒 PreToolUse<br/><b>git-safety-check.sh</b><br/>Block force-push, rm -rf"]
        POST["✅ PostToolUse<br/><b>post-file-write.sh</b><br/>Auto lint + type check"]
        STOP["🏁 Stop<br/><b>on-turn-complete.sh</b><br/>Run tests + build"]
    end

    FILE["📝 File Edit"] --> POST
    GIT_CMD["🔀 Git Command"] --> PRE
    TURN_END["💬 Turn Complete"] --> STOP

    style HOOKS fill:#f8f9fa,stroke:#6c757d
    style PRE fill:#e74c3c,stroke:#c0392b,color:#fff
    style POST fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style STOP fill:#50c878,stroke:#3da360,color:#fff
```


These hooks wire Claude's quality checks into your git workflow automatically.

### `.claude/settings.json` — Git Hook Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/scripts/git-safety-check.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/scripts/post-file-write.sh",
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
            "command": ".claude/scripts/on-turn-complete.sh",
            "async": false
          }
        ]
      }
    ],
    "TaskCompleted": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/scripts/on-task-complete.sh",
            "async": false
          }
        ]
      }
    ]
  }
}
```

---

### `.claude/scripts/git-safety-check.sh` — Block Dangerous Git Commands

```bash
#!/bin/bash
# Intercepts PreToolUse on Bash — blocks dangerous git operations

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

# Block force-push to protected branches
if echo "$COMMAND" | grep -qE "git push.*(--force|-f)"; then
  BRANCH=$(git branch --show-current 2>/dev/null)
  if echo "$BRANCH" | grep -qE "^(main|master|develop|release/)"; then
    echo "BLOCKED: Force push to protected branch '$BRANCH' is not allowed." >&2
    exit 2  # Exit code 2 = block + show stderr to Claude
  fi
fi

# Block direct push to main/develop
if echo "$COMMAND" | grep -qE "git push origin (main|master|develop)$"; then
  echo "BLOCKED: Direct push to protected branch. Create a PR instead." >&2
  exit 2
fi

# Block commit of .env files
if echo "$COMMAND" | grep -qE "git (add|commit).*\.env"; then
  echo "BLOCKED: Attempted to commit .env file. Never commit secrets." >&2
  exit 2
fi

# Warn (don't block) on large staged file sets
STAGED_COUNT=$(git diff --staged --name-only 2>/dev/null | wc -l)
if [ "$STAGED_COUNT" -gt 20 ]; then
  echo "WARNING: $STAGED_COUNT files staged. Consider splitting into smaller commits." >&2
  # Don't exit — warn only
fi

exit 0
```

---

### `.claude/scripts/post-file-write.sh` — Auto Lint + Type Check

```bash
#!/bin/bash
# Runs after every file write/edit

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.path // ""')

if [[ -z "$FILE_PATH" ]]; then exit 0; fi

# TypeScript files: lint + type check
if [[ "$FILE_PATH" == *.ts || "$FILE_PATH" == *.tsx ]]; then
  npx eslint --fix "$FILE_PATH" --quiet 2>/dev/null || true
  # Don't run full tsc on every write — too slow. Run on Stop hook instead.
fi

# Check for secrets accidentally added
if grep -qiE "(password|secret|token|api_key)\s*=\s*['\"][^'\"]{8,}" "$FILE_PATH" 2>/dev/null; then
  echo "SECURITY WARNING: Possible hardcoded secret detected in $FILE_PATH" >&2
fi

exit 0
```

---

### `.claude/scripts/on-turn-complete.sh` — Type Check + Tests After Each Claude Turn

```bash
#!/bin/bash
# Runs when Claude finishes a turn (Stop event)

echo "Running post-turn quality checks..."

# TypeScript check
TS_RESULT=$(npx tsc --noEmit 2>&1)
if [ $? -ne 0 ]; then
  echo "TypeScript errors found:"
  echo "$TS_RESULT" | head -30
  # Output to stdout — Claude reads this and will fix the errors
fi

# Run affected tests (files touched this turn)
CHANGED_FILES=$(git diff --name-only HEAD 2>/dev/null | grep -E '\.(ts|tsx)$' | grep -v '\.test\.')
if [ -n "$CHANGED_FILES" ]; then
  TEST_FILES=$(echo "$CHANGED_FILES" | sed 's|src/|src/__tests__/|' | sed 's|\.ts$|.test.ts|')
  EXISTING_TESTS=$(echo "$TEST_FILES" | xargs -I{} sh -c 'test -f {} && echo {}' 2>/dev/null)
  
  if [ -n "$EXISTING_TESTS" ]; then
    echo "Running tests for changed files..."
    npx jest $EXISTING_TESTS --passWithNoTests 2>&1 | tail -20
  fi
fi
```

---

### `.gitignore` — Claude-specific entries

```
# Claude Code
.claude/settings.local.json   # Local MCP overrides (may have personal tokens)
.claude/agent-memory-local/   # Local agent memory (not shared)
.claude/worktrees/            # Worktree directories (git manages these)
```

Always commit:
- `CLAUDE.md`
- `.claude/settings.json` (no actual tokens — use `${ENV_VAR}` placeholders)
- `.claude/skills/`
- `.claude/agents/`
- `.claude/scripts/`
- `.claude/context/`

---

## 4. Git Worktrees — Parallel Branch Development

Git worktrees let you work on multiple branches simultaneously in separate directories, sharing the same `.git` folder. Claude Code 2.x has first-class worktree support via the `--worktree` flag and per-worktree configuration.

### The `--worktree` Flag (v2.1.49+)

```bash
# Start Claude in a new worktree for a feature
# Creates .claude/worktrees/feature-auth/ with a new branch feature-auth
claude --worktree feature-PROJ-123-auth

# Simultaneously, start another session on a separate task
claude --worktree fix-PROJ-456-session-timeout

# Each session:
# - Gets its own working directory
# - Gets its own branch
# - Has its own Claude Code session with independent context
# - Can have its own MCP configuration
# - Cannot affect the other session's files
```

### When to Use Worktrees

```mermaid
graph TB
    subgraph USE["✅ Use Worktrees When"]
        U1["Two features on<br/>different files"]
        U2["Reviewing a colleague's<br/>branch during your work"]
        U3["Running experiments<br/>without polluting main"]
        U4["Agent Teams working<br/>parallel sub-features"]
        U5["Background subagent<br/>doing exploratory work"]
    end
    subgraph AVOID["❌ Don't Use When"]
        A1["Tasks have<br/>sequential dependencies"]
        A2["Both tasks modify<br/>the same files"]
        A3["Coordination overhead<br/>exceeds benefit"]
    end

    style USE fill:#d4edda,stroke:#28a745
    style AVOID fill:#ffe5e5,stroke:#e74c3c
```

### Worktree Directory Structure

```mermaid
graph TB
    subgraph PROJECT["📁 my-project/"]
        GIT["🗄️ .git/<br/><i>Shared git history</i>"]
        MAIN["📂 main-branch/<br/>CLAUDE.md + .claude/settings.json<br/><i>Your main working directory</i>"]
        subgraph WORKTREES["📂 .claude/worktrees/"]
            WT1["🌿 feature-PROJ-123-auth/<br/>CLAUDE.md (inherited)<br/>.claude/settings.json<br/><i>Can override MCPs</i>"]
            WT2["🌿 fix-PROJ-456-timeout/<br/>CLAUDE.md (inherited)<br/>.claude/settings.json<br/><i>Independent MCP config</i>"]
        end
    end
    
    GIT -.->|"shared"| MAIN
    GIT -.->|"shared"| WT1
    GIT -.->|"shared"| WT2

    style PROJECT fill:#f8f9fa,stroke:#6c757d
    style MAIN fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style WORKTREES fill:#fff3cd,stroke:#ffc107
    style WT1 fill:#50c878,stroke:#3da360,color:#fff
    style WT2 fill:#50c878,stroke:#3da360,color:#fff
```

### Per-Worktree MCP Configuration

Each worktree can have its own `.claude/settings.json`, enabling different MCP servers per task:

```json
// .claude/worktrees/feature-PROJ-123-auth/.claude/settings.json
// This feature needs Playwright for browser auth testing
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    },
    "postgres-dev": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres",
               "postgresql://claude_readonly:${DB_PASSWORD}@localhost:5432/myapp_dev"]
    }
  }
}
```

```json
// .claude/worktrees/fix-PROJ-456-timeout/.claude/settings.json
// This fix only needs log access
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/var/log/app"]
    }
  }
}
```

### Subagent Worktree Isolation

For risky work, run subagents in temporary worktrees that auto-clean up:

```markdown
<!-- .claude/agents/experimental-refactor.md -->
---
name: experimental-refactor
description: Runs a refactor in an isolated worktree — auto-deleted if no changes committed
isolation: worktree
model: claude-sonnet-4-6
---

Perform the refactor in an isolated branch. If the approach doesn't work,
the worktree will be automatically cleaned up. Report back with:
1. Whether the refactor succeeded
2. Files changed
3. Test results
4. Whether it should be merged or abandoned
```

### Parallel Feature Development Workflow

```bash
# Terminal 1 — Feature A (auth)
claude --worktree feature-PROJ-123-auth
> /feature-scaffold oauth-login

# Terminal 2 — Feature B (payments, unrelated files)
claude --worktree feature-PROJ-456-payment-methods
> /feature-scaffold payment-methods

# Terminal 3 — Your main session (review, coordination)
claude
> What's the status of both worktrees? 
  Check git log in each and summarise what's been implemented.
```

---

## 5. Full git + Claude Code Settings

### `.claude/settings.json` — Complete Git-Optimised Configuration

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y", "@modelcontextprotocol/server-filesystem",
        "/Users/dev/projects/my-app",
        "/Users/dev/projects/my-app-docs"
      ]
    },
    "git": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/Users/dev/projects/my-app"]
    },
    "postgres-dev": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y", "@modelcontextprotocol/server-postgres",
        "postgresql://claude_readonly:${DB_PASSWORD}@localhost:5432/myapp_dev"
      ]
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    },
    "memory": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  },

  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": ".claude/scripts/git-safety-check.sh" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [{ "type": "command", "command": ".claude/scripts/post-file-write.sh", "async": true }]
      }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": ".claude/scripts/on-turn-complete.sh" }] }
    ]
  },

  "permissions": {
    "allow": [
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git branch:*)",
      "Bash(git checkout:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git worktree:*)",
      "Bash(npm run:*)",
      "Bash(npx tsc:*)",
      "Bash(npx jest:*)",
      "Bash(npx eslint:*)"
    ],
    "deny": [
      "Bash(git push --force:*)",
      "Bash(git push -f:*)",
      "Bash(rm -rf:*)"
    ]
  }
}
```

---

## 6. CI/CD Integration — Headless Claude in GitHub Actions

```mermaid
graph LR
    PR["📝 PR Opened"] --> GHA["⚙️ GitHub Action<br/>Triggered"]
    GHA --> DIFF["📄 Get PR Diff"]
    DIFF --> REVIEW["🤖 Claude Headless<br/><code>claude -p</code><br/><i>AI Code Review</i>"]
    REVIEW --> COMMENT["💬 Post Review<br/>Comment on PR"]
    REVIEW --> CHANGELOG["📋 Generate<br/>Changelog Entry"]
    
    subgraph GATES["Quality Gates"]
        LINT["✅ Lint"] 
        TEST["✅ Tests"]
        BUILD["✅ Build"]
    end
    
    COMMENT --> GATES
    GATES -->|"All pass"| READY["🟢 Ready for<br/>Human Review"]

    style PR fill:#f5a623,stroke:#d4891a,color:#fff
    style REVIEW fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style READY fill:#50c878,stroke:#3da360,color:#fff
```


Claude Code in headless mode (`-p` flag, no REPL) integrates cleanly into GitHub Actions, enabling AI-powered code review, changelog generation, and automated quality gates.

### Workflow: AI Code Review on Every PR

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for accurate diffs

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run AI Review
        id: review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # Get the diff
          git fetch origin ${{ github.base_ref }}
          DIFF=$(git diff origin/${{ github.base_ref }}...HEAD)
          
          # Run Claude review
          REVIEW=$(echo "$DIFF" | claude --headless -p \
            "You are a senior engineer reviewing a PR.
             Review for: security issues, logic bugs, missing error handling, test gaps.
             Be specific with file:line references.
             End with: APPROVE or REQUEST_CHANGES on its own line.")
          
          echo "review<<EOF" >> $GITHUB_OUTPUT
          echo "$REVIEW" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
          
          # Fail the step if Claude requests changes on critical issues
          if echo "$REVIEW" | grep -q "CRITICAL"; then
            echo "::error::Critical issues found in AI review"
            exit 1
          fi

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 🤖 AI Code Review\n\n${{ steps.review.outputs.review }}`
            })
```

### Workflow: Automated Changelog on Release

```yaml
# .github/workflows/changelog.yml
name: Generate Changelog

on:
  push:
    tags:
      - 'v*'

jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Generate Changelog
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          PREVIOUS_TAG=$(git tag --sort=-version:refname | sed -n '2p')
          COMMITS=$(git log ${PREVIOUS_TAG}..HEAD --format="%s %b" --no-merges)
          
          echo "$COMMITS" | claude --headless -p \
            "Generate a release changelog from these commits.
             Version: ${{ github.ref_name }}
             Categorise into: New Features, Bug Fixes, Performance, Breaking Changes.
             Write for a developer audience.
             Output markdown only." > CHANGELOG_NEW.md
          
          # Prepend to existing changelog
          cat CHANGELOG_NEW.md CHANGELOG.md > CHANGELOG_UPDATED.md
          mv CHANGELOG_UPDATED.md CHANGELOG.md
          
          git config user.name "Claude Code Bot"
          git config user.email "bot@yourcompany.com"
          git add CHANGELOG.md
          git commit -m "docs: update changelog for ${{ github.ref_name }}"
          git push origin HEAD:main
```

---

## 7. The Complete Feature Delivery Workflow

This is the AISFD git workflow from ticket to merged PR, using everything covered in this article:

```bash
# Step 1: Start from a Jira/Linear ticket
claude
> /git-start-feature PROJ-123
# Claude creates branch: feature/PROJ-123-add-oauth-login
# Pulls latest main first

# Step 2: Plan the implementation
[Shift+Tab] → Plan Mode
> Implement Google OAuth login. The ticket says:
  - Users click "Sign in with Google" on the login page
  - OAuth flow follows RFC 6749 Authorization Code flow
  - Store googleId on User model alongside email/password auth
  - Don't break existing email/password login
  
# Claude reads the codebase, produces a plan.
# Review and approve.

# Step 3: Implement (Tasks DAG for dependency management)
> Create a Tasks DAG for this feature and implement in order.
  Run TypeScript check after each file.

# Step 4: Hooks auto-run lint + tests after each file write

# Step 5: Security review in background while you test manually
> Spawn a security-reviewer subagent on the new auth files in the background.

# Step 6: Generate commit when ready
/git-commit
# Claude reads staged diff, generates conventional commit message, commits.

# Step 7: Generate PR description
/git-pr
# Claude reads full branch diff, writes complete PR description.

# Step 8: Create PR via GitHub MCP
> Create a GitHub PR from this branch to main.
  Use the PR description we just generated.
  Add label: "ready-for-review"

# Step 9: CI runs AI review (GitHub Actions)
# Claude Code in headless mode reviews the PR automatically.
# Results posted as a PR comment.

# Step 10: Address review comments
> Read the AI review comment on PR #234 via the GitHub MCP.
  Implement the fixes. Don't change anything that was marked APPROVE.

# Step 11: Merge
> The PR is approved. Squash and merge via GitHub.
  Delete the remote branch after merging.

# Total time for a standard feature: hours vs days.
```

---

## Summary

A fully wired Claude + git setup is the difference between Claude as a tool and Claude as a development platform. The key pieces:

- **CLAUDE.md git conventions** — branch naming, commit formats, protection rules all encoded once, applied everywhere
- **Git Skills** — `/git-start-feature`, `/git-commit`, `/git-pr`, `/git-review`, `/git-changelog` as reusable slash commands
- **Safety hooks** — block dangerous operations, auto-lint on write, type-check and test on turn completion
- **Worktrees** — parallel branch development with isolated sessions and per-branch MCP configurations
- **CI/CD integration** — headless Claude for automated PR review and changelog generation

The investment in setup is a few hours. The compound return is every commit, every PR, every release after that.

---

*This is Article 12 in the AI-Assisted Development series.*
