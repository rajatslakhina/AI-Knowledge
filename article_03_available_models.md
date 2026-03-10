# Article 3: Which AI Models Are Available? A Developer's Map of the LLM Landscape (Updated March 2026)

> *GPT-5.2, Claude Opus 4.6, Gemini 3.1 Pro — the model landscape has transformed dramatically since late 2025. This is the current state of play.*

---

## Introduction

The LLM landscape in early 2026 looks nothing like it did twelve months ago. In the span of three months — November 2025 through February 2026 — Google shipped Gemini 3, OpenAI shipped GPT-5.1 and GPT-5.2, and Anthropic shipped Claude Opus 4.5, Opus 4.6, and Sonnet 4.6 in rapid succession. The pace is relentless.

This article maps the current major model families, their capabilities, and their practical trade-offs — as of March 2026.

---

## 1. The Major Players

### 1.1 Anthropic — Claude 4.6 Family

Anthropic was founded in 2021 by former OpenAI researchers including Dario and Daniela Amodei, with AI safety as a core mission. Their Constitutional AI training approach is reflected in Claude's reliability and honest uncertainty.

**Current Claude Model Family (as of March 2026):**

| Model | API String | Context Window | Output | Pricing (per MTok) |
|---|---|---|---|---|
| **Claude Opus 4.6** | `claude-opus-4-6` | 200K (1M beta) | 128K | $15 / $75 |
| **Claude Sonnet 4.6** | `claude-sonnet-4-6` | 200K (1M beta) | 64K | $3 / $15 |
| **Claude Haiku 4.5** | `claude-haiku-4-5-20251001` | 200K | — | Lowest tier |

> **Note:** The 1M token context window is currently in beta for both Opus 4.6 and Sonnet 4.6.

**Claude Opus 4.6** — Released February 5, 2026
The current flagship. Key new capabilities over Opus 4.5:
- **Agent Teams** (research preview): Coordinates multiple Claude sub-agents in parallel — like having a team of AI collaborators working simultaneously
- **Claude in PowerPoint**: Side-panel integration directly in Microsoft PowerPoint
- Longest verified task-completion horizon of any model: 50%-time horizon of 14h 30min (METR benchmark)
- Fast mode (beta): up to 2.5× faster output generation at premium pricing ($30/$150 per MTok)

**Claude Sonnet 4.6** — Released February 17, 2026 (default model on claude.ai)
The defining story of this release cycle: Sonnet 4.6 matches Opus-level performance on most tasks at one-fifth the cost.
- **SWE-bench Verified: 79.6%** — essentially tied with Opus 4.6's 80.8%
- **OSWorld computer use: 72.5%** — tied with Opus 4.6's 72.7%
- **OfficeQA: matches Opus 4.6** for document comprehension
- Adaptive thinking: Claude dynamically decides when and how much to think
- Developers in Claude Code preferred Sonnet 4.6 over Opus 4.5 59% of the time
- Same pricing as Sonnet 4.5 ($3/$15 per MTok) — a major value leap

**Claude Haiku 4.5** — Released October 2025
Still the speed/cost tier. No 4.6 update yet — Anthropic typically updates Haiku at a slower pace.

**New API Features in Claude 4.6:**
- **Adaptive thinking** (`thinking: {type: "adaptive"}`) — recommended for both Opus 4.6 and Sonnet 4.6; Claude dynamically reasons when needed
- **Context compaction** — automatic server-side summarisation when context approaches window limit, enabling effectively infinite conversations
- **Dynamic web filtering** — Claude can filter search/fetch results before they consume context
- **Fast mode** — 2.5× faster output for Opus 4.6 at premium pricing

**Key strengths (family-wide):**
- Best-in-class instruction-following, now further improved in 4.6
- Exceptional coding ability at both Opus and Sonnet tiers
- Constitutional AI → reliable, less prone to harmful outputs
- 1M token context (beta) enabling entire codebase analysis

---

### 1.2 OpenAI — GPT-5 Family

OpenAI accelerated through three major model releases between August and December 2025, driven partly by competitive pressure from Google's Gemini 3.

**Current OpenAI Model Family (as of March 2026):**

| Model | Context | API String | Notes |
|---|---|---|---|
| **GPT-5.2 Thinking** | 200K+ | `gpt-5.2` | Flagship reasoning model |
| **GPT-5.2 Instant** | 200K+ | `gpt-5.2-chat-latest` | Fast, everyday tasks |
| **GPT-5.2 Pro** | 200K+ | — | Maximum reasoning, wait time |
| **GPT-5.2-Codex** | 200K+ | — | Agentic coding specialist |
| **GPT-5.1** | 200K | `gpt-5.1` | Available, 3-month legacy window |
| **GPT-5** | 128K | — | Legacy, available for 3 months |
| **GPT-4o** | 128K | `gpt-4o` | Still available, multimodal |

**GPT-5.2** — Released December 11, 2025
The GPT-5.2 family comes in three modes:

- **GPT-5.2 Instant** — Speed and efficiency for everyday tasks. Warmer conversational tone introduced in GPT-5.1, improved for clearer how-tos and technical writing.
- **GPT-5.2 Thinking** — Advanced reasoning for professional, real-world tasks. Adaptive thinking time — spends more time on hard problems, faster on simple ones.
- **GPT-5.2 Pro** — OpenAI's "smartest and most trustworthy model yet." More compute, fewer major errors, strongest performance on complex domains including programming.

Key GPT-5.2 improvements:
- First OpenAI model claimed to achieve human-expert performance on knowledge-work tasks
- Significantly stronger multi-agent and agentic execution (collapsed multi-agent systems into single mega-agent workflows)
- Superior spreadsheet creation, financial modelling, presentations
- Knowledge cutoff: August 2025
- Stronger safety behaviours — particularly on self-harm, mental health, and emotional reliance

**GPT-5.2-Codex** — Released December 2025
Optimised version of GPT-5.2 for agentic coding in Codex:
- Context compaction for long-horizon tasks across large repositories
- Stronger performance on large refactors and code migrations
- Improved Windows environment support
- 50%-time horizon: 6h 34min (METR benchmark)

**GPT-5.1** — Released November 2025
- Instant and Thinking variants with adaptive reasoning
- Warmer, more natural conversational tone
- Better instruction-following and everyday usefulness
- Significant improvements on AIME 2025 (maths) and Codeforces (coding)

**Key OpenAI strengths (unchanged):**
- Largest ecosystem — integrations, plugins, third-party tooling
- GPT-4o still the best multimodal (vision + audio + text) model for many workflows
- GPT-5.2-Codex for specialised agentic coding pipelines
- Microsoft Azure deep integration for enterprise deployments

---

### 1.3 Google DeepMind — Gemini 3 Family

Google's Gemini 3 launched November 18, 2025, triggered in part by competitive pressure from Claude and GPT-5.x. The launch was notable for immediate deployment across Google's entire ecosystem at scale — 2 billion Search users gained access on day one.

**Current Gemini Model Family (as of March 2026):**

| Model | Context | API String | Notes |
|---|---|---|---|
| **Gemini 3.1 Pro** | 1M tokens | `gemini-3.1-pro-preview` | Latest, smarter than 3 Pro |
| **Gemini 3 Pro** | 1M tokens | deprecated Mar 26 → migrate | Original Gemini 3 launch |
| **Gemini 3 Deep Think** | 1M tokens | — | Ultra/premium tier reasoning |
| **Gemini 2.5 Pro** | 1M tokens | `gemini-2.5-pro` | Stable, previous gen |
| **Gemini 2.5 Flash** | 1M tokens | `gemini-2.5-flash` | Fast, efficient |

> **Important:** `gemini-3-pro-preview` is deprecated and removed on March 26, 2026. Migrate workflows to `gemini-3.1-pro-preview`.

**Gemini 3 Pro** — Released November 18, 2025
- State-of-the-art multimodal: 81% on MMMU-Pro, 87.6% on Video-MMMU
- 1M token context — entire codebases, multi-document corpora
- **thinking_level parameter** (replaces thinking_budget) — control reasoning depth
- New features: media_resolution control, multimodal function responses, streaming function calling
- Record 1501 Elo score on LMArena at launch

**Gemini 3.1 Pro** — More recent release
Built on Gemini 3, "smarter and more capable for complex problem-solving." Advanced reasoning for tasks where a simple answer isn't enough.

**Gemini 3 Deep Think** — Rolling out to Ultra/premium subscribers
Google's highest-intensity reasoning mode. Powers the AI that won gold-medal performance at IMO and ICPC. Designed for hardest reasoning workloads and long-horizon planning.

**Key Google strengths:**
- 1M token context still a key differentiator for massive document/codebase tasks
- Native multimodal from ground up (text + image + audio + video simultaneously)
- Deep Google ecosystem integration (Workspace, Search, Cloud)
- Antigravity IDE integration from launch day

---

### 1.4 Meta — Llama Family

Meta's open-weights models remain the essential choice for privacy-sensitive deployments and self-hosted inference.

**Current Llama Models:**

| Model | Parameters | Best For |
|---|---|---|
| **Llama 3.3 70B** | 70B | High-quality open-source, coding |
| **Llama 3.1 405B** | 405B | Near-frontier, self-hosted |
| **Llama 3.2 Vision** | 11B/90B | Open multimodal |
| **Llama 3.1 8B** | 8B | Edge, fast inference |

Open-weights: download, self-host, fine-tune. No API costs. Full data privacy.

---

### 1.5 Mistral AI

Still highly relevant for multilingual tasks and efficient open models.

| Model | Type | Best For |
|---|---|---|
| **Mistral Large 2** | Proprietary | Multilingual, reasoning |
| **Codestral** | Open | Code generation |
| **Mixtral 8x22B** | Open (MoE) | Efficient general purpose |

---

## 2. The Competitive Landscape as of March 2026

The model race has produced a remarkable convergence: the gap between Sonnet-tier and Opus-tier has nearly closed, and the gap between providers on coding tasks is measured in single percentage points.

**SWE-bench Verified (real-world coding benchmark):**
- Claude Opus 4.6: 80.8%
- Claude Sonnet 4.6: 79.6%
- GPT-5.2: strong performance (comparable tier)
- Gemini 3 Pro: competitive

**The defining story of early 2026:** Sonnet 4.6 beats Opus 4.5 and is essentially tied with Opus 4.6 on coding and computer use — at one-fifth the cost. For enterprises running millions of API calls, this is transformational economics.

---

## 3. Open-Source vs. Closed-Source: The Core Trade-off

| Dimension | Closed (Claude, GPT-5.2, Gemini 3) | Open (Llama, Mistral) |
|---|---|---|
| **Quality** | Frontier quality | Strong, catching up |
| **Data privacy** | Leaves your infrastructure | Full data control |
| **Cost** | Per-token API fees | Infrastructure cost only |
| **Customisation** | Prompt-level only | Full fine-tuning |
| **Compliance** | Data agreements required | Full on-prem possible |
| **Agent ecosystem** | Rich (MCPs, tools) | Growing |

---

## 4. Specialised Models Worth Knowing

**Code-Specialised:**
- **Claude Sonnet 4.6** — Now the default for coding at all scales
- **GPT-5.2-Codex** — Agentic coding in Codex, long-horizon refactors
- **Codestral** (Mistral) — Open-source, strong fill-in-the-middle

**Embedding Models:**
- **OpenAI text-embedding-3-large** — Strong general-purpose
- **Cohere Embed v3** — Best for RAG/retrieval
- **all-MiniLM-L6-v2** — Lightweight local option

**Vision/Multimodal:**
- **Gemini 3 Pro / 3.1 Pro** — Strongest multimodal overall
- **GPT-4o** — Mature, versatile, strong ecosystem
- **Claude Sonnet 4.6** — Vision + 1M context document analysis
- **Llama 3.2 Vision** — Open, self-hostable

---

## 5. Choosing the Right Model in 2026

| Task | Best Choice | Runner-Up | Budget/Open |
|---|---|---|---|
| Complex coding | Claude Sonnet 4.6 | Opus 4.6 | Llama 3.3 70B |
| Agentic coding pipelines | GPT-5.2-Codex | Claude Opus 4.6 | — |
| Long document analysis | Gemini 3.1 Pro (1M) | Claude Sonnet 4.6 (1M beta) | Llama 3.1 405B |
| Complex reasoning | Claude Opus 4.6 | GPT-5.2 Pro | Llama 3.1 405B |
| Fast API / high volume | Claude Haiku 4.5 | GPT-5.2 Instant | Mistral Small |
| Computer use | Claude Sonnet 4.6 | Claude Opus 4.6 | — |
| Multimodal (vision) | Gemini 3 Pro | GPT-4o | Llama 3.2 Vision |
| Privacy-sensitive | Llama 3.3 70B (local) | Mistral Large 2 | — |
| Multilingual | Mistral Large 2 | GPT-5.2 Instant | — |
| Office/document tasks | Claude Sonnet 4.6 | Claude Opus 4.6 | — |
| Agent teams / orchestration | Claude Opus 4.6 | GPT-5.2 Thinking | — |

---

## 6. The Agentic Frontier

The defining characteristic of the 2026 model landscape is the shift from "chat" to "agents." Every major provider is now competing primarily on:

- **Task horizon** — how long an agent can work autonomously without human intervention
- **Tool use reliability** — how consistently models call tools correctly in multi-step workflows
- **Computer use** — ability to operate software like a human (clicking, typing, navigating)
- **Multi-agent coordination** — orchestrating teams of specialised sub-agents

Claude Opus 4.6's Agent Teams, GPT-5.2-Codex's long-horizon coding, and Gemini 3's agentic execution are all competing in this space. This is where the next wave of developer productivity gains will come from.

---

## Summary

The early 2026 model landscape:
- **Claude Sonnet 4.6** — The new default for most development tasks: Opus-level quality, Sonnet price
- **Claude Opus 4.6** — Flagship for deep reasoning, agent orchestration, highest-stakes tasks
- **GPT-5.2** — Strong all-rounder, best for multi-agent mega-workflows, Codex for agentic coding
- **Gemini 3.1 Pro** — Strongest multimodal, unmatched for 1M+ token tasks, Google ecosystem
- **Llama 3.x** — Essential for privacy-sensitive and self-hosted deployments
- **Mistral** — Best multilingual open-source option

In the next article, we'll deep-dive into task-to-model matching — which specific model to reach for for each development, QA, and BA task.

---

*Next: Article 4 — Model Purpose and Task Matching: Which Model Wins at What? (Updated 2026)*
