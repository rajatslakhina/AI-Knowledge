# Article 4: Model Purpose and Task Matching — Which AI Model Wins at What? (Updated March 2026)

> *The Sonnet 4.6 vs Opus 4.6 calculus, GPT-5.2 variants explained, and where Gemini 3.1 Pro changes the game — a complete decision framework for March 2026.*

---

## Introduction

The model selection question has changed in early 2026. Six months ago, the decision was relatively straightforward: Opus for hard problems, Sonnet for everyday work, Haiku for speed. Today, Sonnet 4.6 has closed the gap with Opus so dramatically that the calculus is fundamentally different.

This article maps every major task to the right model with current, accurate benchmarks and honest practical guidance.

---

## 1. The Updated Model Selection Framework

The four questions remain the same, but the answers have shifted:

**1. What is the cognitive demand?**
- Low-medium: Haiku 4.5 handles more than it used to
- Medium-high: **Sonnet 4.6 now covers what used to require Opus**
- Genuinely hard (multi-agent orchestration, extreme long-horizon): Opus 4.6

**2. How large is the context?**
- < 200K: Any model
- 200K–1M: Sonnet 4.6 (1M beta), Opus 4.6 (1M beta), Gemini 3.1 Pro (1M stable)
- 1M+ stable: Only Gemini 3.1 Pro currently offers this in production

**3. What are the latency requirements?**
- Real-time (< 1s): Haiku 4.5, GPT-5.2 Instant
- Interactive (1–10s): Sonnet 4.6, GPT-5.2 Thinking
- Opus Fast mode: ~2.5× faster at premium pricing ($30/$150 per MTok)
- Batch/async: Opus 4.6, GPT-5.2 Pro, Gemini 3 Deep Think

**4. What are the cost constraints?**
- High volume: Haiku 4.5, GPT-5.2 Instant
- Standard enterprise: **Sonnet 4.6 ($3/$15) — the new default**
- Quality-critical only: Opus 4.6 ($15/$75), GPT-5.2 Pro

---

## 2. The Claude 4.6 Family — The New Calculus

### The Headline Change: Sonnet 4.6 ≈ Opus 4.6 on Most Tasks

The most important shift in March 2026: Claude Sonnet 4.6 has essentially matched Opus 4.6 on the tasks that matter most to engineering teams.

| Benchmark | Sonnet 4.6 | Opus 4.6 | Previous Opus (4.5) |
|---|---|---|---|
| SWE-bench Verified (coding) | **79.6%** | 80.8% | ~75% |
| OSWorld computer use | **72.5%** | 72.7% | — |
| OfficeQA (document tasks) | **matches** | baseline | — |
| METR task horizon (50%) | — | **14h 30min** | — |

Sonnet 4.6 at $3/$15 per MTok vs Opus 4.6 at $15/$75 per MTok — 5× cheaper, ~1% worse on coding. For any team making millions of API calls, this arithmetic is decisive.

---

### Claude Opus 4.6 — When to Still Use It

Opus 4.6 remains the right choice for a specific set of tasks:

**1. Multi-Agent Orchestration (Agent Teams)**
Opus 4.6 introduced Agent Teams — the ability to coordinate multiple Claude sub-agents working in parallel. This is genuinely new capability: an orchestrator Opus delegates to specialist sub-agents, all running simultaneously. Use Opus when you need this coordination layer.

**2. Extreme Long-Horizon Autonomous Tasks**
50%-time horizon of 14h 30min. For tasks that need to run for hours without human intervention — massive codebase refactors, end-to-end feature development, complex research synthesis — Opus 4.6 maintains an edge.

**3. Deep Reasoning on Novel Problems**
For problems outside standard patterns — unusual algorithm design, complex architectural trade-offs with many constraints, synthesis across contradictory sources — Opus's additional reasoning depth pays off.

**4. Maximum Reliability for Irreversible Actions**
Opus 4.6 still has edge on the hardest bug detection and the deepest planning. When a wrong answer triggers an irreversible production action, use Opus.

**Fast Mode for Opus:** If you need Opus quality but are latency-constrained, Fast mode (beta) delivers ~2.5× faster output at $30/$150 per MTok. Same intelligence, faster inference.

```
The updated Opus vs Sonnet decision rule:

Use Sonnet 4.6 for:       → Coding, testing, documentation, review, 
                             office tasks, computer use, most agents
                             
Use Opus 4.6 for:         → Multi-agent orchestration, 10+ hour autonomous 
                             tasks, novel problems, irreversible high-stakes actions
```

---

### Claude Haiku 4.5 — Still the Speed Tier

Haiku 4.5 (October 2025) remains unchanged. Still the right choice for:
- Real-time applications needing sub-2-second responses
- High-volume batch processing (millions of requests)
- Simple classification and extraction
- Routing/triage before escalating to Sonnet

No 4.6 update yet — Anthropic typically updates Haiku at a slower cadence.

---

## 3. The OpenAI GPT-5 Family — Variants Explained

OpenAI shipped three model families in quick succession in late 2025. Understanding the variants is essential.

### GPT-5 (August 2025) → GPT-5.1 (November 2025) → GPT-5.2 (December 2025)

**GPT-5.2 Instant** (`gpt-5.2-chat-latest`)
The everyday workhorse. Warmer, more conversational tone. Designed for speed and routine tasks — info-seeking, how-tos, technical writing, translation. If your use case is interactive and doesn't require heavy reasoning, this is the cost-efficient choice in the GPT-5 family.

**GPT-5.2 Thinking** (`gpt-5.2`)
The standard API model. Adaptive reasoning — spends more compute on hard problems, faster on simple ones. First OpenAI model claimed to achieve human-expert performance on real-world knowledge work. Excels at:
- Multi-step professional tasks
- Complex data analysis and financial modelling
- Spreadsheet and presentation creation
- Agentic workflows — "collapsed a fragile, multi-agent system into a single mega-agent with 20+ tools" (real user quote from OpenAI release)

**GPT-5.2 Pro**
Maximum reasoning, more compute, longer wait times. Fewer major errors. Best for high-stakes domains where quality is worth waiting for. Similar positioning to Claude Opus 4.6.

**GPT-5.2-Codex**
Specialised for agentic coding in Codex CLI:
- Context compaction for maintaining context over long sessions
- Strong on large refactors, migrations, and feature builds in big repositories
- Improved Windows environment support
- 50%-time horizon: 6h 34min (vs Claude Opus 4.6's 14h 30min)

```
GPT-5 family decision tree:
├─ Everyday tasks, speed → GPT-5.2 Instant
├─ Professional work, agents → GPT-5.2 Thinking
├─ Maximum quality, no time pressure → GPT-5.2 Pro
└─ Agentic coding in Codex → GPT-5.2-Codex
```

---

## 4. Gemini 3 Family — The Long-Context Leader

### Gemini 3 Pro → Gemini 3.1 Pro (Upgrade Path)

> **Breaking:** `gemini-3-pro-preview` is deprecated and removed **March 26, 2026**. If you're using it, migrate to `gemini-3.1-pro-preview` immediately.

**Gemini 3.1 Pro** is now the production model. Smarter and more capable than Gemini 3 Pro, designed for tasks where a simple answer isn't enough.

**Where Gemini 3.1 Pro wins:**

1. **Massive context tasks (> 200K tokens)**
Gemini's 1M token context is stable in production. Claude's 1M window is currently beta. When you need to process an entire codebase, a year of meeting transcripts, or a large document library in a single request, Gemini 3.1 Pro is the production-ready choice.

2. **Multimodal superiority**
81% on MMMU-Pro, 87.6% on Video-MMMU. When your task combines text, images, audio, and video in complex ways, Gemini 3 leads.

3. **Google Workspace integration**
If your team lives in Google Docs, Sheets, Drive, and Gmail, Gemini 3 in Workspace is deeply integrated in a way no other provider matches.

**Gemini 3 Deep Think**
Google's highest-intensity reasoning mode. Rolling out to Ultra/premium subscribers. Gold-medal performance at IMO and ICPC. For genuinely hard mathematical and logical problems.

**New API features in Gemini 3:**
- `thinking_level` parameter — control reasoning depth (replaces thinking_budget)
- `media_resolution` — control vision processing (low/medium/high)
- Multimodal function responses — functions can return images and PDFs, not just text
- Streaming function calling

---

## 5. Task-to-Model Mapping — Updated for 2026

### Development Tasks

| Task | Primary | When to Upgrade | Avoid |
|---|---|---|---|
| Feature implementation | **Sonnet 4.6** | Opus 4.6 for agent coordination | Haiku |
| Code review | **Sonnet 4.6** | Opus 4.6 for security-critical | — |
| Simple bug fix | Haiku 4.5 | Sonnet 4.6 if complex | — |
| Complex debugging | **Sonnet 4.6** | Opus 4.6 if multi-file systemic | — |
| Architecture design | **Opus 4.6** | Sonnet 4.6 for most decisions | Haiku |
| Large codebase refactor | **Sonnet 4.6** (1M beta) | Gemini 3.1 Pro (1M stable) | — |
| SQL query generation | Haiku / Sonnet 4.6 | — | — |
| Algorithm design | **Opus 4.6** | GPT-5.2 Pro | — |
| Agentic coding pipeline | **GPT-5.2-Codex** | Claude Opus 4.6 | — |
| Computer use tasks | **Sonnet 4.6** | Opus 4.6 | — |
| Multi-agent orchestration | **Opus 4.6** (Agent Teams) | GPT-5.2 Thinking | — |

### QA Tasks

| Task | Primary | When to Upgrade | Notes |
|---|---|---|---|
| Unit test generation | **Sonnet 4.6** | — | Default choice |
| E2E test scenarios | **Sonnet 4.6** | Opus 4.6 for complex flows | — |
| Test data generation | Haiku 4.5 | Sonnet 4.6 | Fast, cost-efficient |
| Bug report writing | Haiku / Sonnet 4.6 | — | — |
| Accessibility review | **Sonnet 4.6** | GPT-4o (vision) | — |
| Load test scripting | **Sonnet 4.6** | — | — |
| Test plan | **Sonnet 4.6** | Opus 4.6 for complex systems | — |

### Business Analyst Tasks

| Task | Primary | When to Upgrade | Notes |
|---|---|---|---|
| User story writing | Haiku 4.5 / **Sonnet 4.6** | — | Haiku often sufficient |
| Requirements extraction | **Sonnet 4.6** | Opus 4.6 for complex specs | — |
| Gap analysis | **Sonnet 4.6** | Opus 4.6 for contradictory docs | — |
| Stakeholder communication | **Sonnet 4.6** | — | Writing quality |
| Large document analysis | **Gemini 3.1 Pro** | Sonnet 4.6 (1M beta) | Stable 1M context |
| Data analysis | **Sonnet 4.6** | GPT-5.2 Thinking | — |
| Meeting summarisation | Haiku 4.5 | — | — |
| Process diagrams | Haiku / Sonnet 4.6 | — | — |

---

## 6. The Cost-Quality Frontier in 2026

The most significant change in early 2026 is the dramatic shift in the cost-quality Pareto frontier.

```
Quality
  │                                    Opus 4.6 ●
  │                          Sonnet 4.6 ●
  │              GPT-5.2 ●
  │    Haiku 4.5 ●
  └─────────────────────────────────────── Cost (per MTok)
     Low                              High

The gap between Sonnet and Opus quality has nearly closed,
while the price gap remains 5×. Sonnet 4.6 has moved the 
quality frontier at the $3/$15 price point.
```

**Updated cost optimisation strategy:**

1. **Default to Sonnet 4.6** for everything (was: default to Sonnet, upgrade to Opus liberally)
2. **Use Haiku** for high-volume simple tasks and real-time
3. **Use Opus** only for: agent team coordination, 10h+ autonomous tasks, novel deep reasoning
4. **Use Gemini 3.1 Pro** when > 200K tokens in production (1M context is stable)
5. **Use GPT-5.2-Codex** if your team is already in the Codex ecosystem

In practice for most teams in 2026:
- 65–70% of tasks: Haiku 4.5 sufficient
- 25–30% of tasks: Sonnet 4.6 required
- **< 5% of tasks**: Opus 4.6 genuinely needed (down from ~10% in 2025)

---

## 7. Multi-Model Workflows — Updated Patterns

### Pattern 1: Updated Quality Tier Routing
```
Incoming task → Classify (Haiku)
├─ Simple/fast → Haiku
├─ Standard dev/QA/BA work → Sonnet 4.6 (covers what Opus did before)
└─ Agent orchestration / novel problems → Opus 4.6
```

### Pattern 2: Long-Context Pipeline
```
Document > 200K tokens?
├─ Yes, production-critical → Gemini 3.1 Pro (stable 1M)
├─ Yes, can accept beta → Sonnet 4.6 1M beta OR Opus 4.6 1M beta
└─ No → Sonnet 4.6 standard
```

### Pattern 3: Agentic Coding
```
Coding agent task?
├─ In Codex ecosystem → GPT-5.2-Codex
├─ Long-horizon, multi-agent → Opus 4.6 (Agent Teams)
├─ Standard feature/bug work → Sonnet 4.6
└─ Simple one-shot generation → Haiku or Sonnet 4.6
```

### Pattern 4: Infinite Conversations (New in 4.6)
```
Long-running session approaching context limit?
└─ Enable context compaction → Effectively infinite conversation
   (Claude automatically summarises earlier turns server-side)
```

---

## Summary

The model landscape in March 2026 has a clear story:

1. **Claude Sonnet 4.6** is the new default for virtually all development work — Opus-level quality, Sonnet price
2. **Claude Opus 4.6** is for agent orchestration, 10h+ horizons, and the genuinely hardest problems
3. **Claude Haiku 4.5** for speed/volume — still excellent, no 4.6 update yet
4. **GPT-5.2 Thinking/Codex** for enterprise agentic workflows, especially in Microsoft/Azure ecosystems
5. **Gemini 3.1 Pro** when you need stable 1M+ token context or Google Workspace integration
6. **Llama/Mistral** for data privacy, on-premises, and high-volume self-hosted deployments

The key insight: **the value of using Opus has fallen dramatically** now that Sonnet 4.6 has caught up on most tasks. Revisit your model routing logic — you're likely overpaying if you haven't updated it since late 2025.

---

*Next: Article 5 — Setting Up Claude in VS Code and Terminal: A Complete Configuration Guide*
