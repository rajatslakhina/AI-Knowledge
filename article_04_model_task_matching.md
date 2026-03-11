# Article 4: Model Purpose and Task Matching — Which AI Model Wins at What?

> *A practical decision framework for choosing the right model for every task — from code generation to data analysis, document review to real-time APIs.*

---

## Introduction

Having many models is only useful if you know which one to reach for. Using a Ferrari to deliver pizza works, but it's wasteful. Using a moped to win a race works, but you'll lose.

Model selection is an engineering decision with measurable consequences: quality, latency, and cost all vary by 2–10× depending on which model you use for which task. This article gives you a concrete framework for making those decisions.

---

## 1. The Model Selection Framework

Before picking a model, answer four questions:

**1. What is the cognitive demand?**
- Low: Summarise, classify, extract, format
- Medium: Generate code, write docs, analyse requirements
- High: Multi-step reasoning, architectural decisions, complex debugging

**2. How large is the context?**

| Context Size | Suitable Models |
| :--- | :--- |
| < 8K tokens | Any model — even Haiku 4.5 |
| 8K – 100K | Any modern model |
| 100K – 200K | Claude Sonnet 4.6 / Opus 4.6, GPT-5.2 |
| 200K – 1M | Gemini 3.1 Pro (native 1M) or Claude 200K + chunking strategy |

**3. What are the latency requirements?**

| Tier | Target | Best Models |
| :--- | :---: | :--- |
| Real-time | < 500ms | Haiku 4.5, GPT-5.2 mini, Gemini 3.1 Flash, Groq + Llama 4 Scout |
| Interactive | < 5s | Sonnet 4.6, GPT-5.2, Gemini 3.1 Flash |
| Batch | Minutes OK | Opus 4.6, o4, Gemini 3.1 Pro |

**4. What are the cost constraints?**

| Volume | Price Sensitivity | Recommended Models |
| :--- | :---: | :--- |
| High volume | 🔴 Cost-critical | Haiku 4.5 ($0.80/$4 MTok), GPT-5.2 mini, Mistral Small 3 |
| Moderate volume | 🟡 Balanced | Sonnet 4.6 ($3/$15 MTok), GPT-5.2 |
| Low volume | 🟢 Quality-first | Opus 4.6 ($15/$75 MTok), o4, Gemini 3.1 Pro |

---

## 2. The Claude Model Family Deep-Dive

Since we're building toward a Claude-focused setup in this series, let's thoroughly understand the Claude model tiers.

### Claude Opus 4.6
**The Flagship — Highest Intelligence**

Opus is Anthropic's most capable model. Use it when the task demands maximum reasoning depth, nuanced judgment, or working through genuinely difficult problems.

**Strengths:**
- Complex, multi-step reasoning across domains
- Deep analysis of large documents (200K context)
- Sophisticated code architecture and design
- Research synthesis from multiple sources
- Tasks where a wrong answer has serious consequences

**Typical latency:** 15–60 seconds for complex requests
**Relative cost:** Highest in the Claude family

**Best for:**
```
✓ Designing system architecture from complex requirements
✓ Reviewing and synthesising 50+ page technical specs
✓ Complex debugging across multiple interdependent modules
✓ High-stakes code reviews (security, data integrity)
✓ Research reports combining 10+ sources
✓ Generating comprehensive test strategies
```

**Example use case:**
> "Here is our 200-page technical specification document. Identify all the architectural decision points, flag any contradictions, and produce a risk-ranked list of implementation challenges."

---

### Claude Sonnet 4.6
**The Workhorse — Best Performance-per-Dollar**

Sonnet is the model most developers will use most of the time. It's fast enough for interactive use, capable enough for almost all development tasks, and costs significantly less than Opus.

**Strengths:**
- Excellent code generation and review
- Strong reasoning on moderate-complexity problems
- Great at following complex, multi-part instructions
- Handles 200K context with strong recall
- Reliable formatting (JSON, XML, markdown)

**Typical latency:** 3–15 seconds
**Relative cost:** ~5x cheaper than Opus

**Best for:**
```
✓ Daily coding assistance (new features, refactoring, bug fixing)
✓ Test case generation
✓ Documentation writing
✓ Code review
✓ User story writing
✓ Data transformation and analysis
✓ API integration code
✓ Most production AI features in applications
```

**Rule of thumb:** Default to Sonnet. Upgrade to Opus only when Sonnet's output quality is genuinely insufficient.

---

### Claude Haiku 4.5
**The Speed Tier — Fast and Cheap**

Haiku is Anthropic's fastest and most cost-efficient model. It sacrifices some reasoning depth for dramatically improved speed and lower cost.

**Strengths:**
- Sub-second to 2-second response times
- Very low cost per token
- Surprisingly capable for well-defined, bounded tasks
- Good at classification, extraction, and formatting

**Typical latency:** 0.5–3 seconds
**Relative cost:** ~25x cheaper than Opus

**Best for:**
```
✓ Real-time applications (chatbots, autocomplete)
✓ High-volume batch processing (processing thousands of documents)
✓ Simple classification and extraction tasks
✓ Summarising short texts
✓ Generating short-form content
✓ Routing/triage tasks (classifying before sending to a better model)
✓ Simple Q&A on well-defined topics
```

**Pattern: Haiku as Router**
```
User query → Haiku classifies intent → 
  Simple task → Haiku answers 
  Complex task → Route to Sonnet
  Critical task → Route to Opus
```

This tiered routing pattern can reduce costs by 60–80% while maintaining quality where it matters.

---

## 3. OpenAI Model Deep-Dive

### GPT-5.2: The Multimodal Powerhouse
Best when your task involves images, diagrams, screenshots, or audio alongside text. GPT-5.2 is fully multimodal — text, image, audio, and video in a single API call.

**Unique strengths:**
- Process images directly (UI screenshots, diagrams, charts, handwritten notes)
- Real-time audio input/output (voice apps)
- GPT-5.2-Codex variant purpose-built for agentic software development
- Massive ecosystem (thousands of examples, integrations, plugins)

### o4: The Reasoning Model
OpenAI's o4 "thinks before it answers" — they spend tokens on internal chain-of-thought reasoning before producing output. This dramatically improves performance on:
- Mathematical proofs
- Complex algorithm design
- Multi-step logical puzzles
- Competitive programming problems

**Trade-off:** Much slower (30–120 seconds for hard problems). Not for interactive use.

**When to use o4:**
```
✓ The problem involves multiple logical steps that must all be correct
✓ Competitive programming or algorithmic challenges
✓ Complex mathematical or statistical analysis
✓ When standard LLMs give plausible-but-wrong answers
```

---

## 4. Gemini 3.1 Pro: The Long-Context King

When you need to process genuinely massive amounts of context — an entire codebase, a year of meeting transcripts, a library of documentation — Gemini 3.1 Pro's 1 million token window is unmatched.

**Practical threshold:** When your context exceeds 200K tokens and you can't (or don't want to) implement chunking, Gemini 3.1 Pro is the right choice.

**Best for:**
```
✓ Analysing an entire codebase at once
✓ Processing large research corpora
✓ Full-book analysis and synthesis
✓ End-to-end tracing across a long conversation history
✓ Processing large log files
```

---

## 5. When to Use Local/Open-Source Models

Use Llama, Mistral, or other open models when:

**Data privacy is paramount**
Healthcare records, financial data, legal documents, proprietary IP — anything you cannot send to third-party APIs.

**Volume makes API cost prohibitive**
At very high volumes (millions of requests/day), running your own inference cluster can be cheaper than API costs.

**Custom fine-tuning is required**
Proprietary terminology, domain-specific tasks, or unique style requirements that can't be addressed via prompting.

**Latency must be sub-100ms**
Running inference on-premises with dedicated hardware can achieve latencies impossible with API calls.

```
Decision tree for open vs. closed models:
├─ Data sensitivity HIGH → Local Llama / Mistral
├─ Volume > 10M requests/month → Evaluate self-hosting
├─ Need fine-tuning → Local open-source
├─ Need max quality → Claude Opus 4.6 / GPT-5.2
└─ Default → Claude Sonnet 4.6 (best quality/cost at $3/$15 MTok)
```

---

## 6. Task-to-Model Mapping Reference

### Development Tasks

| Task | ✅ Primary | 🔄 Fallback | ❌ Avoid |
| :--- | :--- | :--- | :--- |
| Feature implementation | Sonnet 4.6 | GPT-5.2-Codex | Haiku (complex tasks) |
| Code review | Sonnet 4.6 | Opus 4.6 | — |
| Bug fix — simple | Haiku 4.5 | Sonnet 4.6 | — |
| Bug fix — complex | Sonnet 4.6 | Opus 4.6 | Haiku |
| Architecture design | Opus 4.6 | Sonnet 4.6 | Haiku |
| Refactoring large codebase | Sonnet 4.6 (200K ctx) | Gemini 3.1 Pro (1M ctx) | — |
| SQL query generation | Sonnet 4.6 | Haiku 4.5 | — |
| Regex generation | Haiku 4.5 | Sonnet 4.6 | — |
| Algorithm / complexity design | o4 | Opus 4.6 | Haiku |
| API documentation | Sonnet 4.6 | Haiku 4.5 | — |

### QA Tasks

| Task | ✅ Primary | 🔄 Fallback | ❌ Avoid |
| :--- | :--- | :--- | :--- |
| Unit test generation | Sonnet 4.6 | Haiku 4.5 | — |
| E2E test scenarios | Sonnet 4.6 | Opus 4.6 | Haiku |
| Test data generation | Haiku 4.5 | Sonnet 4.6 | — |
| Bug report writing | Sonnet 4.6 | Haiku 4.5 | — |
| Accessibility review | Sonnet 4.6 | GPT-5.2 (vision) | — |
| Load test scripting | Sonnet 4.6 | Haiku 4.5 | — |
| Test plan writing | Sonnet 4.6 | Opus 4.6 | — |
| Exploratory test charters | Sonnet 4.6 | Opus 4.6 | Haiku |

### BA Tasks

| Task | ✅ Primary | 🔄 Fallback | ❌ Avoid |
| :--- | :--- | :--- | :--- |
| User story writing | Sonnet 4.6 | Haiku 4.5 | — |
| Requirements extraction | Sonnet 4.6 | Opus 4.6 | — |
| Stakeholder communication | Sonnet 4.6 | — | — |
| Gap analysis | Opus 4.6 | Sonnet 4.6 | Haiku |
| Data analysis | Sonnet 4.6 | Gemini 3.1 Pro (large data) | — |
| Meeting summarisation | Haiku 4.5 | Sonnet 4.6 | — |
| Process diagram (Mermaid) | Sonnet 4.6 | Haiku 4.5 | — |
| Competitive analysis | Opus 4.6 | Sonnet 4.6 | Haiku |

---

## 7. Multi-Model Workflows

The most sophisticated AI-powered engineering workflows use multiple models in sequence:

### Pattern 1: Quality Tiers
```
Incoming task → Classify complexity (Haiku)
├─ Simple → Haiku answers
├─ Medium → Sonnet answers
└─ Complex → Opus answers
```

### Pattern 2: Draft + Review
```
Initial code draft (Sonnet) → 
Security/logic review (Opus) → 
Final output
```

### Pattern 3: Extract then Reason
```
Large document (Gemini 3.1 Pro extracts key sections) → 
Reasoning task on extracted content (Claude Sonnet)
```

### Pattern 4: RAG Pipeline
```
User question → 
Embed question (text-embedding-3-large) → 
Retrieve relevant chunks (vector DB) → 
Synthesise answer (Claude Sonnet 4.6)
```

---

## 8. Cost Optimisation Strategy

A practical approach to minimising cost without sacrificing quality:

1. **Start with Haiku** for all tasks during development
2. **Identify failure points** — where does Haiku's output fall short?
3. **Upgrade those specific tasks** to Sonnet
4. **Repeat** — find the remaining failure points
5. **Use Opus only** for the tasks that genuinely require it

In practice, most engineering teams find:
- 60–70% of tasks: Haiku is sufficient
- 25–35% of tasks: Sonnet required
- 5–10% of tasks: Opus required

This tiered approach reduces API costs by 5–10× compared to using Opus for everything.

---

## Summary

Model selection is a cost/quality/latency optimisation problem. The key principles:

1. **Default to Sonnet 4.6** for most development work
2. **Use Haiku** for high-volume, simple, or real-time tasks
3. **Reserve Opus 4.6** for genuinely complex reasoning and high-stakes reviews
4. **Use o4** for algorithmic and mathematical problems
5. **Use Gemini 3.1 Pro** when context exceeds ~150K tokens
6. **Use local Llama 4 Scout/Maverick** when data privacy or cost at scale demands it

Now that we've mapped the landscape, in the next article we'll get hands-on: setting up Claude in VS Code and the terminal, configuring Claude Code, and establishing a productive daily workflow.

---

*Next: Article 5 — Setting Up Claude in VS Code and Terminal: A Complete Setup Guide*
