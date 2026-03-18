# GitMCP.io as Your Knowledge Repo — Zero-Infrastructure Context That Actually Improves Claude's Accuracy

> *You asked: "If I wire my GitHub knowledge repo through gitmcp.io into the same `.claude/settings.json` as Jira/Linear, filesystem, git, postgres, and memory — will Claude get better context and produce more accurate output?" The short answer is yes, meaningfully so. This article explains exactly why, how the mechanism works under the hood, how to structure your knowledge repo to maximise the gains, and how the full combined stack behaves in practice.*

---

## What You're Actually Building

```mermaid
graph TB
    CLAUDE["🤖 Claude Code<br/>Full Context"] 
    
    CLAUDE --> FS["📁 Filesystem<br/><i>Current codebase</i>"]
    CLAUDE --> GIT["🔀 Git<br/><i>Change history</i>"]
    CLAUDE --> PG["🗄️ Postgres<br/><i>Data shape</i>"]
    CLAUDE --> GH["🐙 GitHub<br/><i>PRs & issues</i>"]
    CLAUDE --> JR["📋 Jira/Linear<br/><i>What to build</i>"]
    CLAUDE --> SL["💬 Slack<br/><i>Decisions</i>"]
    CLAUDE --> MEM["🧠 Memory<br/><i>Past sessions</i>"]
    CLAUDE --> KR["📚 Knowledge Repo<br/><i>Team patterns<br/>& rules</i>"]

    style CLAUDE fill:#7b68ee,stroke:#5a4dbd,color:#fff
    style KR fill:#f5a623,stroke:#d4891a,color:#fff
    style FS fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style GIT fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style PG fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style GH fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style JR fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style SL fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style MEM fill:#4a90d9,stroke:#2d6cb4,color:#fff
```

> 🎯 **The Knowledge Repo (via GitMCP) is the only MCP that tells Claude HOW your team builds software.** Without it, Claude falls back to generic "best practices" that may conflict with your actual architecture.


When you combine these servers in `.claude/settings.json`:

```
filesystem + git + postgres + github + jira/linear + slack + memory + gitmcp knowledge-repo
```

You are giving Claude **nine dimensions of context** simultaneously. Each MCP server answers a different question Claude would otherwise have to guess:

| MCP Server | Question it answers |
|---|---|
| `filesystem` | What does the actual codebase look like right now? |
| `git` | What changed, when, and by whom? |
| `postgres-dev` | What is the real data shape and schema? |
| `github` | What PRs, issues, and reviews exist? |
| `jira` / `linear` | What is the team trying to build and why? |
| `slack` | What decisions were made in conversation? |
| `memory` | What did Claude learn across previous sessions? |
| **gitmcp knowledge-repo** | **How does THIS team build software — patterns, rules, templates?** |

The knowledge repo via GitMCP is the only one that answers that last question. Without it, Claude falls back to generic "best practices" — which may conflict with your actual architecture, naming conventions, and security rules. **That gap is where most AI-generated code introduces technical debt.**

---

## How GitMCP.io Works Mechanically

```mermaid
graph LR
    REPO["📦 GitHub Repo<br/>your-org/knowledge-repo"] -->|"Replace github.com<br/>with gitmcp.io"| GITMCP["🔌 GitMCP.io<br/>Remote MCP Server"]
    GITMCP -->|"mcp-remote"| CC["🤖 Claude Code"]
    
    CC -->|"fetch_docs"| D["📄 llms.txt<br/>Overview"]
    CC -->|"search_docs"| S["🔍 Semantic<br/>Search"]
    CC -->|"search_code"| C["💻 Code<br/>Examples"]

    style REPO fill:#6c757d,stroke:#495057,color:#fff
    style GITMCP fill:#f5a623,stroke:#d4891a,color:#fff
    style CC fill:#7b68ee,stroke:#5a4dbd,color:#fff
```


GitMCP is a free, open-source remote MCP server. The core mechanic is extremely simple: replace `github.com` with `gitmcp.io` in any GitHub URL.

```
https://github.com/your-org/knowledge-repo
        ↓
https://gitmcp.io/your-org/knowledge-repo
```

That URL becomes a fully functioning MCP server endpoint. You connect via `mcp-remote`:

```json
{
  "mcpServers": {
    "knowledge-repo": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://gitmcp.io/your-org/knowledge-repo"
      ]
    }
  }
}
```

Zero infrastructure. Zero build step. No custom server to maintain. The repo IS the server.

### What Tools GitMCP Exposes to Claude

Once connected, Claude gets access to **five tools** it can call autonomously during any session:

| Tool | What it does | When Claude uses it |
|---|---|---|
| `fetch_{repo}_documentation` | Retrieves full documentation — prioritises `llms.txt`, falls back to `README.md` | Broad overview questions, project orientation |
| `search_{repo}_documentation` | **Semantic search** across all docs — returns only the relevant chunks | "How does this team handle auth?", "What's the error pattern?" |
| `search_{repo}_code` | GitHub code search API — finds actual code snippets | "Show me a service layer example" |
| `fetch_url_content` | Follows links inside docs to retrieve referenced material | When a pattern doc links to an external spec |
| `match_common_libs_owner_repo_mapping` | Maps library names to owner/repo pairs | Dynamic endpoint mode |

The **semantic search** tool is the critical one. Claude does not dump the entire repo into its context on every prompt. It queries intelligently and retrieves only the chunks relevant to the current task. This is why a large, well-structured knowledge repo does not bloat the context window — it is queried on demand.

### Document Priority: llms.txt → llms-full.txt → README.md

GitMCP uses a strict priority order when fetching documentation:

```mermaid
graph TD
    FETCH["🤖 Claude fetches docs"] --> Q{"llms.txt<br/>exists?"}
    Q -->|"Yes"| L1["📄 <b>llms.txt</b><br/><i>Optimised for AI<br/>Concise, structured</i>"]
    Q -->|"No"| Q2{"llms-full.txt<br/>exists?"}
    Q2 -->|"Yes"| L2["📄 <b>llms-full.txt</b><br/><i>Extended AI-readable</i>"]
    Q2 -->|"No"| L3["📄 <b>README.md</b><br/><i>Human-facing fallback</i>"]
    
    L1 --> SEARCH["🔍 Semantic search discovers<br/>other .md files on demand"]
    L2 --> SEARCH
    L3 --> SEARCH

    style L1 fill:#50c878,stroke:#3da360,color:#fff
    style L2 fill:#4a90d9,stroke:#2d6cb4,color:#fff
    style L3 fill:#6c757d,stroke:#495057,color:#fff
    style SEARCH fill:#f5a623,stroke:#d4891a,color:#fff
```

> 🎯 **Highest-leverage action:** Add an `llms.txt` file to your knowledge repo root. It's the first thing Claude reads.

**The single highest-leverage thing you can do** is add an `llms.txt` file to the root of your knowledge repo. This file is the first thing Claude reads when it connects to the repo MCP. It sets the frame for every subsequent query.

---

## Does It Actually Improve Accuracy? Breaking Down the Mechanisms

```mermaid
graph TD
    subgraph WITHOUT["❌ Without Knowledge Repo"]
        W1["Claude uses generic patterns"] --> W2["Code works but doesn't<br/>match team conventions"] --> W3["2+ rounds of<br/>review comments"] --> W4["⏱️ 3x longer delivery"]
    end
    
    subgraph WITH["✅ With Knowledge Repo"]
        K1["Claude reads team patterns"] --> K2["Code matches architecture<br/>+ naming + security rules"] --> K3["Minor review comments<br/>if any"] --> K4["⚡ Ship fast"]
    end

    style WITHOUT fill:#ffe5e5,stroke:#e74c3c
    style WITH fill:#d4edda,stroke:#28a745
    style W4 fill:#e74c3c,stroke:#c0392b,color:#fff
    style K4 fill:#50c878,stroke:#3da360,color:#fff
```


Yes. Here is exactly why — broken down by the type of improvement:

### 1. Eliminates Architectural Hallucination

Claude's training data contains patterns from thousands of codebases. Without team-specific context, it will generate code that is generically correct but architecturally wrong for your project. Examples:

- You use Repository pattern → Claude generates direct Prisma calls in service layers
- You use React Query for server state → Claude adds Redux for server data
- You have a custom `AppError` class → Claude throws generic `new Error()`
- Your auth uses `requestContext.user` → Claude imports from `express-jwt` directly

With the knowledge repo connected, Claude queries it before scaffolding and finds the actual patterns. It generates code that fits your architecture on the first attempt.

### 2. Enforces Security Rules at Generation Time

Security rules in a knowledge repo are enforced *before* code is written, not in a review after. Example content in `security/rules.md`:

```markdown
# Security Rules — Non-Negotiable

## SQL
- NEVER use string interpolation in queries
- ALL queries go through Prisma or parameterised pg
- Raw SQL only in /db/migrations, never in application code

## Auth  
- JWT validation only in middleware, never in route handlers
- requestContext.user is the ONLY authoritative user object
- NEVER trust client-supplied user IDs without re-validation

## Secrets
- NEVER log req.body on authenticated routes
- NEVER include credentials in error messages returned to client
```

When Claude queries the knowledge repo before writing a feature, it reads these rules and applies them. This is qualitatively different from hoping a security reviewer catches violations after the fact.

### 3. Templates Eliminate Boilerplate Drift

When your knowledge repo contains actual production-ready templates, Claude uses them as the scaffold instead of generating from scratch:

```
knowledge-repo/
  templates/
    service.template.ts       ← Standard service layer skeleton
    controller.template.ts    ← Route handler with validation shape
    repository.template.ts    ← Data access layer pattern
    react-component.template.tsx
    expo-screen.template.tsx
```

The generated code starts from your template, not from a generic interpretation of "TypeScript service". The diff between what Claude produces and what your senior engineer would produce shrinks dramatically.

### 4. Architecture Decision Records Prevent Revisiting Settled Decisions

A common AI failure mode: Claude suggests using a library or pattern your team explicitly evaluated and rejected. ADRs in the knowledge repo prevent this:

```
knowledge-repo/
  architecture/
    decisions/
      ADR-001-use-react-query-not-redux.md
      ADR-002-prisma-not-typeorm.md  
      ADR-003-zod-validation-at-controller-boundary.md
      ADR-004-no-class-based-services.md
```

When Claude queries "how should I manage server state", it finds ADR-001 and uses React Query. When it considers TypeORM, it finds ADR-002 and stays on Prisma. Your settled technical decisions stay settled.

### 5. The Memory MCP Creates Durable Pattern Reinforcement

The `memory` MCP in your stack creates an important compounding loop with GitMCP. When Claude discovers a pattern from the knowledge repo and successfully applies it, you can instruct it to store that association in memory:

```
"Remember: this team uses AppError with statusCode and code fields for all 
service-layer errors. Knowledge repo has the pattern at patterns/backend/error-handling.md"
```

In future sessions, Claude retrieves this from memory and immediately knows to query the knowledge repo for the specific error-handling pattern. The two MCPs reinforce each other.

---

## The Complete Combined settings.json

Here is the full `.claude/settings.json` that wires all nine MCPs together, with the knowledge repo added via GitMCP:

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
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
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://claude_readonly:password@localhost:5432/myapp_dev"
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
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-server-jira"],
      "env": {
        "JIRA_HOST": "${JIRA_HOST}",
        "JIRA_EMAIL": "${JIRA_EMAIL}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    },
    "slack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    },
    "memory": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "knowledge-repo": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://gitmcp.io/your-org/your-knowledge-repo"
      ]
    }
  }
}
```

> **Note:** Replace `your-org/your-knowledge-repo` with your actual GitHub org and repo name. The repo must be **public** — GitMCP only supports public repositories. For private repos, see the Self-Hosting section below.

---

## How to Structure Your Knowledge Repo for Maximum GitMCP Accuracy

```mermaid
mindmap
  root((📚 Knowledge<br/>Repo Structure))
    📄 llms.txt
      Stack summary
      Architecture rules
      Security rules
      Key patterns
    🏗️ architecture/
      ADRs
      System diagrams
      Module boundaries
    📝 patterns/
      Service patterns
      Error handling
      API contracts
    🔒 security/
      Auth patterns
      Input validation
      Secrets handling
    🧪 testing/
      Test conventions
      Coverage rules
      Fixture patterns
    📋 templates/
      PR templates
      Issue templates
      RFC templates
```


The quality of Claude's context is directly proportional to the quality of your knowledge repo structure. Here is the recommended layout:

```
your-org/knowledge-repo/
│
├── llms.txt                          ← CRITICAL: AI entry point, read first
├── llms-full.txt                     ← Extended AI-readable full content  
├── README.md                         ← Human-readable index
│
├── architecture/
│   ├── overview.md                   ← System architecture, domain model
│   ├── tech-stack.md                 ← Approved libraries with versions
│   └── decisions/
│       ├── ADR-001-*.md
│       └── ADR-00N-*.md
│
├── patterns/
│   ├── backend/
│   │   ├── service-layer.md          ← How services are structured
│   │   ├── repository-layer.md       ← Data access patterns
│   │   ├── error-handling.md         ← AppError, status codes, logging
│   │   ├── validation.md             ← Zod schemas, controller boundary
│   │   └── testing.md                ← Unit/integration test patterns
│   ├── frontend/
│   │   ├── component-structure.md    ← Component anatomy, naming
│   │   ├── state-management.md       ← React Query vs Zustand rules
│   │   ├── api-layer.md              ← How frontend calls backend
│   │   └── testing.md
│   └── mobile/
│       ├── screen-structure.md
│       ├── navigation.md
│       └── native-features.md
│
├── templates/
│   ├── service.template.ts
│   ├── controller.template.ts
│   ├── repository.template.ts
│   ├── react-component.template.tsx
│   └── expo-screen.template.tsx
│
├── security/
│   ├── rules.md                      ← Non-negotiable security constraints
│   ├── auth-patterns.md              ← How auth/authz works in this codebase
│   └── data-handling.md              ← PII, secrets, logging rules
│
├── api-contracts/
│   ├── rest-conventions.md           ← URL structure, response shapes, errors
│   └── openapi/                      ← OpenAPI specs if applicable
│
└── examples/
    └── feature-complete/
        └── user-notifications/       ← Full end-to-end working example
            ├── controller.ts
            ├── service.ts
            ├── repository.ts
            └── tests/
```

### The llms.txt File — The Most Important File in the Repo

```mermaid
graph TB
    subgraph LLMS["📄 llms.txt — The Most Important File"]
        direction TB
        S["🏗️ <b>Stack</b><br/>Languages, frameworks,<br/>databases, versions"]
        AR["🏛️ <b>Architecture Rules</b><br/>Non-negotiable patterns<br/>and anti-patterns"]
        SR["🔒 <b>Security Rules</b><br/>Auth, validation, secrets<br/>handling requirements"]
        KP["📋 <b>Key Patterns</b><br/>Links to pattern docs<br/>for each domain"]
    end

    CLAUDE["🤖 Claude reads this FIRST<br/>on every session"] --> LLMS

    style LLMS fill:#d4edda,stroke:#28a745
    style CLAUDE fill:#7b68ee,stroke:#5a4dbd,color:#fff
```


The `llms.txt` file is read first on every GitMCP connection. It is your opportunity to prime Claude with the most critical team context before any work begins. Keep it concise and scannable — Claude reads it to orient, not to learn everything.

```markdown
# YourApp Knowledge Base

## Stack
- Backend: Node.js 22, TypeScript 5.4, Express 5, Prisma 5, PostgreSQL 16
- Frontend: React 19, TypeScript 5.4, Vite 6, React Query v6, shadcn/ui
- Mobile: React Native 0.76, Expo SDK 52, Expo Router v4
- Testing: Vitest, React Testing Library, Playwright

## Architecture Rules (Non-Negotiable)
- Controller → Service → Repository (never skip layers)
- All validation at controller boundary using Zod
- All errors via AppError class (see patterns/backend/error-handling.md)
- No direct DB calls outside Repository layer
- React Query for ALL server state, Zustand for UI-only state

## Security Rules (Non-Negotiable)
- Never string-interpolate SQL — Prisma or parameterised queries only
- Never log request bodies on authenticated routes
- Never trust client-supplied user IDs without re-validation
- JWT validation in middleware only, never in route handlers

## Key Patterns
- Service layer: patterns/backend/service-layer.md
- Error handling: patterns/backend/error-handling.md
- Component structure: patterns/frontend/component-structure.md
- State management decision tree: patterns/frontend/state-management.md

## Templates
- New service: templates/service.template.ts
- New controller: templates/controller.template.ts
- New React component: templates/react-component.template.tsx

## ADRs
- ADR-001: React Query over Redux for server state
- ADR-002: Prisma over TypeORM
- ADR-003: Zod at controller boundary, not service layer
- ADR-004: Functional services, no classes
```

This file answers the most common questions Claude would need to ask before generating any code. Every section directly maps to a tool call Claude would otherwise have to make (or guess).

---

## How the Multi-MCP Stack Behaves Together: A Concrete Example

```mermaid
sequenceDiagram
    participant Dev as 👨‍💻 Developer
    participant CC as 🤖 Claude Code
    participant FS as 📁 Filesystem
    participant Jira as 📋 Jira
    participant KR as 📚 Knowledge Repo
    participant Git as 🔀 Git

    Dev->>CC: "Add rate limiting to the orders API"
    CC->>Jira: Search for rate-limiting tickets
    Jira-->>CC: PROJ-89: Rate limit all public APIs
    CC->>KR: search("rate limiting pattern")
    KR-->>CC: middleware/rate-limit.md — token bucket, Redis-backed
    CC->>FS: Read src/middleware/
    FS-->>CC: Current middleware files
    CC->>Git: Recent rate-limit commits?
    Git-->>CC: None found
    CC->>FS: Create rate-limit middleware following team pattern
    CC->>Dev: "Implemented per knowledge repo pattern. Shall I add tests?"
```


You open Claude Code and type:

```
Implement PROJ-247: Add email notification when an order is shipped
```

Here is what happens across the MCP stack:

**Step 1 — Jira MCP** reads PROJ-247. Retrieves the ticket description, acceptance criteria, and any linked design specs. Claude now knows the business requirement in detail.

**Step 2 — Knowledge Repo MCP (GitMCP)** is queried. Claude calls:
- `search_knowledge-repo_documentation("notification service pattern")` → finds `patterns/backend/service-layer.md`
- `search_knowledge-repo_documentation("email integration")` → finds any existing email patterns or approved library (e.g. Resend, SendGrid)
- `search_knowledge-repo_code("notification")` → finds any existing notification code in the repo

**Step 3 — Filesystem MCP** reads the existing codebase. Claude locates `src/services/`, `src/repositories/`, existing service files to understand naming conventions.

**Step 4 — Git MCP** checks recent commits. Claude sees if similar work was done recently and understands the current branch state.

**Step 5 — Postgres MCP** queries the schema. Claude checks the `orders` table structure, confirms column names, understands the data shape it will work with.

**Step 6 — Claude generates code** — not from generic training data, but grounded in:
- The exact business requirement from Jira
- Your team's service/repository layer pattern
- Your approved email library and its usage pattern
- The actual `orders` schema from your database
- The naming conventions from your existing codebase

The result is code that matches your architecture, uses your approved libraries, follows your security rules, and satisfies the ACs from the ticket — on the first pass.

---

## Limitations You Need to Know

### GitMCP is Public Repos Only

GitMCP works with any public GitHub repository. **Private repositories are not accessible.** For teams with private knowledge repos, there are two options:

**Option A — Self-host GitMCP**

GitMCP is fully open-source. You can run your own instance with access to private repos:

```bash
git clone https://github.com/idosal/git-mcp
cd git-mcp
npm install
npm run build
GITHUB_TOKEN=ghp_your_token npm start
```

Then point your config at your local instance:

```json
{
  "knowledge-repo": {
    "command": "npx",
    "args": ["mcp-remote", "http://localhost:3000/your-org/private-repo"]
  }
}
```

**Option B — Use the custom MCP server** from Article 13 of this series. The custom TypeScript MCP server serves content from a local directory, requires no GitHub at all, and works with any file system. Best choice if your knowledge content should never leave your network.

### Content Quality Determines Query Quality

GitMCP's semantic search is only as good as what it has to search. A knowledge repo with vague, undated, or contradictory content will produce ambiguous results. The improvement in Claude's accuracy scales directly with the clarity and specificity of your documentation.

Treat the knowledge repo as production code: PR reviews, clear ownership, regular updates. When your patterns change, update the repo. When Claude generates something wrong, that is often a signal the relevant pattern is missing or unclear.

### The Repo is Queried On Demand, Not Preloaded

GitMCP does not dump the entire repo into Claude's context at the start of every session. Claude calls the search and fetch tools when it judges them relevant. This is efficient for token usage but means Claude might not query the repo for a task where a pattern exists but wasn't obviously signalled.

You can make this more reliable by explicitly telling Claude to query first:

```
Before implementing, check the knowledge repo for any existing patterns 
related to authentication middleware
```

Or add this instruction to your `CLAUDE.md`:

```markdown
## Knowledge Repo
Always query the knowledge-repo MCP before:
- Scaffolding any new service, controller, or component
- Implementing any auth or security-sensitive code  
- Choosing between libraries or patterns
- Writing database queries
```

### Network Latency

Each GitMCP query is an HTTP round-trip to `gitmcp.io` (or your self-hosted instance). In practice this adds 200–600ms per query. For a typical feature implementation with 3–5 knowledge repo queries, this is a 1–3 second overhead over the full session — negligible for the accuracy gains.

---

## Comparing the Three Knowledge Delivery Approaches

```mermaid
graph TB
    subgraph APPROACHES["📊 Three Knowledge Delivery Approaches"]
        direction LR
        A["📄 <b>CLAUDE.md Only</b><br/>━━━━━━━━<br/>✅ Simple<br/>⚠️ Limited size<br/>❌ Not searchable<br/>❌ Static"]
        B["📂 <b>.claude/skills/</b><br/>━━━━━━━━<br/>✅ Structured<br/>✅ Repo-scoped<br/>⚠️ Manual loading<br/>⚠️ Local only"]
        C["🌐 <b>GitMCP Repo</b><br/>━━━━━━━━<br/>✅ Semantic search<br/>✅ Zero infra<br/>✅ Team-shared<br/>✅ Auto-queried"]
    end

    REC["💡 Best: Combine all three — CLAUDE.md for essentials,<br/>skills for workflows, GitMCP for deep patterns"]

    style A fill:#ffe5e5,stroke:#e74c3c
    style B fill:#fff3cd,stroke:#ffc107
    style C fill:#d4edda,stroke:#28a745
    style REC fill:#f0f4ff,stroke:#4a90d9
```


| Approach | Setup | Maintenance | Private? | Accuracy |
|---|---|---|---|---|
| **GitMCP (gitmcp.io)** | 2 lines of config | Git commit | ❌ Public only | ★★★★☆ |
| **GitMCP (self-hosted)** | Run local server | Git commit + server | ✅ | ★★★★☆ |
| **Custom MCP Server** (Article 13) | Build TypeScript server | Update files | ✅ | ★★★★★ |
| **CLAUDE.md only** | Edit a file | Edit a file | ✅ | ★★★☆☆ |
| **No knowledge layer** | None | None | N/A | ★★☆☆☆ |

The custom MCP server from Article 13 has a slight accuracy edge because it allows structured tools (`get_template`, `get_security_rules`, `search_knowledge` with typed returns) rather than document-level semantic search. But GitMCP gets you to ~90% of that benefit with zero build overhead.

**Recommended progression:**

1. **Week 1** — Start with GitMCP on a public repo. Establish the structure, add `llms.txt`, populate the key patterns. Validate that Claude is querying and using it correctly.
2. **Month 1** — If the accuracy gains are clear (they will be), either self-host GitMCP for private access or invest in building the custom MCP server for typed tool access.
3. **Ongoing** — Treat knowledge repo updates as part of every feature PR. When a new pattern is established, document it. When an old pattern is deprecated, update it.

---

## Quick Start: From Zero to Working in 15 Minutes

### Step 1 — Create the knowledge repo on GitHub

Create a new public repo: `your-org/ai-knowledge`

### Step 2 — Add the minimum viable structure

```
ai-knowledge/
├── llms.txt        ← Copy the template from this article, fill in your stack
├── README.md       ← Link to all sections
└── patterns/
    └── backend/
        └── service-layer.md   ← Your most common pattern
```

### Step 3 — Add to .claude/settings.json

```json
"knowledge-repo": {
  "command": "npx",
  "args": [
    "mcp-remote",
    "https://gitmcp.io/your-org/ai-knowledge"
  ]
}
```

### Step 4 — Add query instructions to CLAUDE.md

```markdown
## Knowledge Repo
MCP server: knowledge-repo (via gitmcp.io/your-org/ai-knowledge)

Query it before scaffolding any new file, choosing a library, or 
implementing auth or security-sensitive code. Use:
- search_{repo}_documentation("topic") for patterns and rules  
- fetch_{repo}_documentation() for full orientation
- search_{repo}_code("snippet") for code examples
```

### Step 5 — Test it

```
Open Claude Code, type:
"What patterns does this team use for the service layer? Check the knowledge repo."
```

Claude should call `fetch_knowledge-repo_documentation` or `search_knowledge-repo_documentation` and return your team's actual pattern.

### Step 6 — Expand iteratively

Add one document per sprint. Each document you add compounds — every future feature that touches that pattern benefits from it. After two months, the knowledge repo typically has 20–40 documents covering the most-touched areas of your codebase. That coverage produces a measurable and consistent uplift in first-pass code quality.

---

## Summary

Wiring a GitMCP knowledge repo into your existing multi-MCP stack is the single highest-leverage, lowest-effort improvement you can make to Claude's accuracy in your codebase. It requires:

- A public GitHub repo with structured markdown documentation
- Two lines of config in `.claude/settings.json`
- An `llms.txt` file that primes Claude on your stack and rules

What you get in return is Claude that generates code aligned with your architecture, enforces your security rules, uses your approved libraries, and applies your team's patterns — rather than generic "best practice" code that requires extensive revision to fit your actual system.

The other MCPs in your stack (filesystem, git, postgres, jira, slack, memory) tell Claude about the *current state* of your project. The knowledge repo tells Claude about the *intended character* of your project — how your team thinks, what you value, what you have decided. Combined, they close the gap between what Claude generates and what your team would write.

---

*Part of the AI & Claude for Software Engineering Teams series. See also:*
- *Article 13 — Jira/Linear MCP + Building a Custom Knowledge Repo MCP Server*
- *Article 12 — Git + Claude Code: Automating the Full Commit-to-PR Workflow*
- *Article 11 — AISFD: AI-Assisted Software Feature Development with Claude Code 2.x*
