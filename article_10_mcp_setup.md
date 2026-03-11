# Article 10: How to Set Up MCPs — Local and Remote Configuration Guide

> *Step-by-step instructions for configuring every major MCP type — filesystem, Git, GitHub, databases, and custom servers — with troubleshooting and team deployment patterns.*

---

## Introduction

This is the hands-on article. We're going to configure MCPs from scratch — starting with the simplest local filesystem MCP and building up to a full multi-MCP development environment with GitHub, databases, and Slack integration.

By the end, you'll have a production-ready MCP setup that makes Claude Code genuinely agentic across your development tools.

---

## 1. The MCP Configuration File

All MCP configuration lives in Claude Code's configuration file. Depending on your setup:

```bash
# Global config (applies to all projects)
~/.claude/settings.json    # or
~/.config/claude/settings.json

# Project-level config (committed to git, shared with team)
.claude/settings.json
```

**Priority:** Project-level config overrides global config.

### Config File Structure

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allow"],
      "env": {
        "OPTIONAL_ENV_VAR": "value"
      }
    }
  }
}
```

For remote servers:
```json
{
  "mcpServers": {
    "server-name": {
      "type": "sse",
      "url": "https://mcp.example.com/sse",
      "headers": {
        "Authorization": "Bearer your-token"
      }
    }
  }
}
```

---

## 2. Setting Up Local MCP Servers

### 2.1 Filesystem MCP

The filesystem MCP gives Claude read and/or write access to specified directories.

**Installation and configuration:**

```json
// .claude/settings.json
{
  "mcpServers": {
    "filesystem": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects/my-project",
        "/home/user/documents/specs"
      ]
    }
  }
}
```

**Security tip:** List specific directories — do not give access to `/` or `~`. Limit to what Claude genuinely needs.

**Available tools after setup:**
- `read_file` — Read any file in the allowed directories
- `write_file` — Create or overwrite files
- `edit_file` — Make targeted edits to existing files
- `list_directory` — List directory contents
- `create_directory` — Create new directories
- `move_file` — Move/rename files
- `search_files` — Search for files by pattern
- `get_file_info` — Get metadata (size, modified date)

**Test it:**
```
claude> list the files in the project root
claude> read the package.json and tell me the dependencies
claude> create a new file called NOTES.md with a summary of the project
```

---

### 2.2 Git MCP

```json
{
  "mcpServers": {
    "git": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/path/to/your/repo"]
    }
  }
}
```

**Prerequisite:** Install `uv` (Python package manager):
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or via Homebrew
brew install uv
```

**Available tools:**
- `git_log` — Commit history with diffs
- `git_diff` — Diff between commits or working tree
- `git_status` — Working tree status
- `git_show` — Show specific commit details
- `git_branch` — List/manage branches
- `git_blame` — Line-by-line authorship

**Example workflows:**
```
> What changed in the last 5 commits?
> Show me the diff for the auth module between main and feature/oauth
> Who last modified src/services/payment.ts and when?
> List all branches containing changes to the database schema
```

---

### 2.3 SQLite MCP

```json
{
  "mcpServers": {
    "sqlite": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-server-sqlite", "--db-path", "./dev.db"]
    }
  }
}
```

**Available tools:**
- `read_query` — Execute SELECT queries
- `write_query` — Execute INSERT/UPDATE/DELETE (use with care)
- `list_tables` — Show all tables
- `describe_table` — Show table schema

**Example workflows:**
```
> Show me the schema for all tables
> How many users were created this week?
> Find all orders with status 'pending' older than 7 days
```

---

### 2.4 PostgreSQL MCP

```bash
# Install the server
npm install -g @modelcontextprotocol/server-postgres
```

```json
{
  "mcpServers": {
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://username:password@localhost:5432/dbname"
      ]
    }
  }
}
```

**Security critical:** For the PostgreSQL MCP:
- Use a **read-only database user** for most tasks
- Never connect with a superuser or production write credentials
- Use a separate dev/staging database, not production

```sql
-- Create a read-only user for Claude
CREATE USER claude_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE your_db TO claude_readonly;
GRANT USAGE ON SCHEMA public TO claude_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO claude_readonly;
```

---

### 2.5 Memory MCP

The memory MCP gives Claude persistent memory that survives across sessions.

```json
{
  "mcpServers": {
    "memory": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**Available tools:**
- `create_entities` — Store facts about people, projects, concepts
- `create_relations` — Create relationships between entities
- `add_observations` — Add notes to existing entities
- `search_nodes` — Search memory graph
- `open_nodes` — Retrieve specific entities

**Example:**
```
> Remember that we decided to use Event Sourcing for the audit log module
> Remember that Sarah is our DBA and should be tagged on any migration PRs
> What have we decided about the payment service architecture?
```

---

### 2.6 Puppeteer MCP (Browser Automation)

```json
{
  "mcpServers": {
    "puppeteer": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

**Available tools:**
- `puppeteer_navigate` — Navigate to a URL
- `puppeteer_screenshot` — Take a screenshot
- `puppeteer_click` — Click an element
- `puppeteer_fill` — Fill form fields
- `puppeteer_evaluate` — Execute JavaScript

**Example workflows:**
```
> Navigate to localhost:3000/login and take a screenshot
> Check if the payment form validation works by filling in invalid data
> Screenshot the dashboard and describe any UI issues
```

---

## 3. Setting Up Remote MCP Servers

### 3.1 GitHub MCP

```bash
# Set GitHub token as environment variable
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_your_token_here"
```

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

**Creating a GitHub token:**
1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (Fine-grained recommended)
3. Required scopes: `repo`, `read:org`, `read:user`
4. For issue creation: also `write:issues`

**Available tools:**
- `get_file_contents` — Read any file in any repo
- `create_or_update_file` — Create/edit files
- `list_issues` / `create_issue` / `update_issue`
- `list_pull_requests` / `create_pull_request`
- `get_commit` / `list_commits`
- `search_repositories` / `search_code` / `search_issues`
- `create_branch` / `list_branches`

**Example workflows:**
```
> Show me all open PRs in the backend repo assigned to me
> Create a GitHub issue for the null pointer bug we just found
> Search for all files in the frontend repo that import from '../utils/auth'
> What were the last 10 commits to main? Summarise the changes.
```

---

### 3.2 Slack MCP

```json
{
  "mcpServers": {
    "slack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-your-bot-token",
        "SLACK_TEAM_ID": "T0XXXXXXX"
      }
    }
  }
}
```

**Creating a Slack Bot:**
1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Create New App → From Scratch
3. Add OAuth Scopes: `channels:read`, `channels:history`, `chat:write`, `users:read`
4. Install to Workspace → Copy Bot Token

**Available tools:**
- `slack_list_channels` — List workspace channels
- `slack_post_message` — Post a message
- `slack_reply_to_thread` — Reply in a thread
- `slack_get_channel_history` — Read recent messages

**Example workflows:**
```
> Post a message to #deployments: "API v2.3.1 deployed to production at 14:32 UTC"
> What was discussed in #incidents in the last 24 hours?
> Send a thread reply to the last message in #backend about the auth bug
```

---

### 3.3 AWS MCP

```bash
# Ensure AWS CLI is configured
aws configure
# Or use environment variables
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_DEFAULT_REGION="eu-west-1"
```

```json
{
  "mcpServers": {
    "aws": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@aws-sdk/mcp-server-aws"],
      "env": {
        "AWS_PROFILE": "your-profile-name"
      }
    }
  }
}
```

**Example workflows:**
```
> List all ECS services in the production cluster
> What's the current CPU utilisation of the payment-service task?
> Show me the last 50 CloudWatch log events for the auth service
> Which Lambda functions haven't been invoked in the last 30 days?
```

---

## 4. Complete Multi-MCP Configuration

Here's a production-ready configuration for a typical development environment:

```json
// .claude/settings.json
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
    }
  }
}
```

**The matching `.env` file (never commit this):**
```bash
GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxx
SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxx
SLACK_TEAM_ID=T0XXXXXXX
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxx
```

---

## 5. Building a Custom MCP Server

When no existing MCP meets your needs, build your own.

### Full Custom Server Example

This example creates an MCP server for an internal company API:

```typescript
// internal-api-mcp/src/index.ts

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import axios from "axios";

const API_BASE = process.env.INTERNAL_API_URL!;
const API_KEY = process.env.INTERNAL_API_KEY!;

const api = axios.create({
  baseURL: API_BASE,
  headers: { Authorization: `Bearer ${API_KEY}` },
});

const server = new Server(
  { name: "internal-api", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// Define available tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_customer",
      description: "Retrieve customer details from internal CRM",
      inputSchema: {
        type: "object",
        properties: {
          customerId: {
            type: "string",
            description: "Customer UUID"
          }
        },
        required: ["customerId"]
      }
    },
    {
      name: "list_active_subscriptions",
      description: "List all active subscriptions for a customer",
      inputSchema: {
        type: "object",
        properties: {
          customerId: { type: "string" },
          includeTrials: { 
            type: "boolean",
            description: "Include trial subscriptions",
            default: false
          }
        },
        required: ["customerId"]
      }
    },
    {
      name: "create_support_ticket",
      description: "Create a support ticket in internal ticketing system",
      inputSchema: {
        type: "object",
        properties: {
          title: { type: "string" },
          description: { type: "string" },
          priority: { 
            type: "string",
            enum: ["low", "medium", "high", "critical"]
          },
          customerId: { type: "string", description: "Related customer (optional)" }
        },
        required: ["title", "description", "priority"]
      }
    }
  ]
}));

// Implement tool handlers
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  try {
    switch (name) {
      case "get_customer": {
        const response = await api.get(`/customers/${args.customerId}`);
        return {
          content: [{
            type: "text",
            text: JSON.stringify(response.data, null, 2)
          }]
        };
      }

      case "list_active_subscriptions": {
        const response = await api.get(`/customers/${args.customerId}/subscriptions`, {
          params: { 
            status: "active",
            includeTrials: args.includeTrials ?? false
          }
        });
        return {
          content: [{
            type: "text",
            text: JSON.stringify(response.data, null, 2)
          }]
        };
      }

      case "create_support_ticket": {
        const response = await api.post("/tickets", {
          title: args.title,
          description: args.description,
          priority: args.priority,
          customerId: args.customerId,
          source: "claude-mcp"
        });
        return {
          content: [{
            type: "text",
            text: `Ticket created: #${response.data.id} - ${response.data.url}`
          }]
        };
      }

      default:
        throw new Error(`Unknown tool: ${name}`);
    }
  } catch (error: any) {
    return {
      content: [{
        type: "text",
        text: `Error: ${error.message}`
      }],
      isError: true
    };
  }
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
console.error("Internal API MCP server running");
```

**Package setup (`package.json`):**
```json
{
  "name": "internal-api-mcp",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "internal-api-mcp": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**Configuration:**
```json
{
  "mcpServers": {
    "internal-api": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/internal-api-mcp/dist/index.js"],
      "env": {
        "INTERNAL_API_URL": "https://api.internal.company.com",
        "INTERNAL_API_KEY": "${INTERNAL_API_KEY}"
      }
    }
  }
}
```

---

## 6. Verifying MCP Setup

```bash
# Start Claude Code and list available tools
claude

# In the Claude REPL:
> /mcp    # List all connected MCP servers and their status

# Or in terminal:
claude --list-tools

# Test individual tools:
> Use the filesystem MCP to list the files in the current directory
> Use the github MCP to list my open PRs
> Use the postgres MCP to show me the tables in the database
```

---

## 7. Troubleshooting Common Issues

| Symptom | Likely Cause | Diagnosis | Fix |
| :--- | :--- | :--- | :--- |
| **MCP server not starting** | Missing dependency or wrong Node version | `node --version` (need 18+) | `nvm use 20` then reinstall |
| **"command not found" error** | npx cache issue or global install missing | `npx -y @modelcontextprotocol/server-filesystem /tmp` | Clear npx cache: `npm cache clean --force` |
| **Auth error on remote MCP** | Missing or expired env var | `echo $GITHUB_PERSONAL_ACCESS_TOKEN` | Verify token set; check token scopes |
| **Timeout on first use** | Server startup too slow | Check logs with `--debug` flag | Add `"timeout": 30000` to server config |
| **"Permission denied" on filesystem** | Path not granted in config | `ls -la /path/you/granted` | Add path to `args` array in settings.json |
| **Claude doesn't use the MCP** | MCP not loaded or name mismatch | Check Claude Code startup output | Restart Claude Code; verify server name |
| **Rate limit errors** | Too many parallel MCP calls | Check API usage dashboard | Reduce parallel MCP usage; add retry logic |

### Debugging Commands

```bash
# Test a server command directly before adding to config
npx -y @modelcontextprotocol/server-filesystem /tmp

# Verify environment variables are set
echo $GITHUB_PERSONAL_ACCESS_TOKEN
echo $SLACK_BOT_TOKEN

# Test a remote API token manually
curl -H "Authorization: Bearer $GITHUB_PERSONAL_ACCESS_TOKEN" \
  https://api.github.com/user

# Restart Claude Code with debug output
claude --debug
```

---

## 8. Team Deployment Pattern

For team-wide MCP setup:

**Step 1: Commit `.claude/settings.json` to the repo** (with env var placeholders)
```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

**Step 2: Create `.env.example`** with required variables (no values):
```bash
GITHUB_PERSONAL_ACCESS_TOKEN=
SLACK_BOT_TOKEN=
ANTHROPIC_API_KEY=
```

**Step 3: Document setup in README.md:**
```markdown
## Claude Code Setup

1. Install: `npm install -g @anthropic-ai/claude-code`
2. Copy `.env.example` to `.env` and fill in values
3. Run `source .env` or use direnv
4. Run `claude` in project root — MCPs will auto-configure
```

**Step 4: Add to onboarding docs** so new team members have everything they need on day one.

---

## Summary: Complete MCP Setup Checklist

| Category | MCP Server | Priority | Use Case |
| :--- | :--- | :---: | :--- |
| **Local — All Projects** | `@modelcontextprotocol/server-filesystem` | 🔴 High | File read/write within project |
| **Local — All Projects** | `@modelcontextprotocol/server-git` | 🔴 High | Git history, blame, diffs |
| **Local — Databases** | `@modelcontextprotocol/server-sqlite` | 🟡 Medium | Local dev database queries |
| **Local — Databases** | `@modelcontextprotocol/server-postgres` | 🟡 Medium | PostgreSQL queries |
| **Local — Memory** | `@modelcontextprotocol/server-memory` | 🟡 Medium | Persistent project context |
| **Local — Browser** | `@modelcontextprotocol/server-puppeteer` | 🟢 Optional | Browser automation, testing |
| **Remote — Source** | `@modelcontextprotocol/server-github` | 🔴 High | PR reviews, issue management |
| **Remote — Comms** | `@modelcontextprotocol/server-slack` | 🟡 Medium | Team communication context |
| **Remote — PM** | `@linear/mcp-server` | 🟡 Medium | Linear ticket management |
| **Remote — PM** | `mcp-jira-server` | 🟡 Medium | Jira issue management |
| **Remote — Infra** | `@modelcontextprotocol/server-aws-kb-retrieval` | 🟢 Optional | AWS infrastructure queries |
| **Custom** | Build your own | 🟢 Optional | Internal APIs, proprietary data |

---

## Series Summary

Over these first 10 articles in the series, we've covered the complete journey from understanding AI fundamentals to deploying a production-grade Claude-powered development environment:

1. **What AI is** — LLMs, transformers, tokens, limitations
2. **Where to use AI** — Role-by-role application map
3. **Available models** — OpenAI, Anthropic, Google, Meta, Mistral
4. **Model selection** — Task-to-model matching framework
5. **Setup** — Claude Code CLI, VS Code integration, shell workflows
6. **Role workflows** — DEV, QA, and BA-specific patterns
7. **Prompt engineering** — The principles of high-quality prompts
8. **CLAUDE.md system** — Project context files for 95% accuracy
9. **MCP theory** — Types, benefits, and use cases
10. **MCP setup** — Complete local and remote configuration guide

The developers who master these tools aren't replacing their skills — they're multiplying their output. The goal isn't less thinking; it's using your thinking for the problems that actually require it.

---

*This concludes the foundational 10 articles of the 14-part series on AI in Software Engineering. Each article can be used as a standalone reference or read in sequence. Articles 11–14 continue with advanced topics: Claude Code 2.x, Git integration, GitMCP, and Jira/Linear MCP workflows.*
