# Article 3: Which AI Models Are Available? A Developer's Map of the LLM Landscape

> *GPT, Claude, Gemini, Llama, Mistral — cutting through the noise to understand what each model family offers and when to choose each.*

---

## Introduction

The LLM landscape in 2025–2026 is rich, noisy, and moves fast. New model releases and benchmarks appear weekly, each claiming to be "state of the art." For developers building real products, the signal-to-noise ratio can be brutal.

This article maps the major model families, their capabilities, and their practical trade-offs — so you can make informed decisions rather than chasing benchmarks.

---

## 1. The Major Players

### 1.1 Anthropic — Claude Family

Anthropic was founded in 2021 by former OpenAI researchers, including Dario and Daniela Amodei. Their mission is "AI safety" — building powerful AI while ensuring it remains aligned, honest, and harmless. This philosophy is baked into Claude at the model level (Constitutional AI).

**Current Claude Model Family:**

| Model | API String | Context | Best For | Price (input/output per MTok) |
| :--- | :--- | :---: | :--- | ---: |
| **Claude Opus 4.6** | `claude-opus-4-6` | 200K (1M β) | Complex reasoning, architecture, agent teams | $15 / $75 |
| **Claude Sonnet 4.6** | `claude-sonnet-4-6` | 200K | Coding, daily dev, balanced quality/speed | $3 / $15 |
| **Claude Haiku 4.5** | `claude-haiku-4-5-20251001` | 200K | High-volume, fast tasks, summarisation | $0.80 / $4 |

**Key strengths:**
- Best-in-class for long-context tasks (200K window)
- Exceptional instruction-following
- Strong coding ability, particularly for complex multi-file tasks
- Excellent at nuanced writing and communication
- Consistent, predictable output format
- Strong safety — less likely to produce harmful or misleading content

**Unique differentiators:**
- Constitutional AI training results in outputs that are more reliable and less prone to "going off the rails"
- Claude is known for being honest about its uncertainty
- Strongest performance on tasks requiring nuanced reasoning across large documents
- **Benchmarks (Sonnet 4.6):** SWE-bench 79.6% · OSWorld 72.5% | **Opus 4.6:** METR autonomous task horizon 14h 30min

---

### 1.2 OpenAI — GPT Family

OpenAI is the company that popularised LLMs with ChatGPT in late 2022. Their GPT-5 and o-series models remain the strongest competitors to Claude in the enterprise and developer market.

**Current OpenAI Models:**

| Model | Context | Best For | Notes |
| :--- | :---: | :--- | :--- |
| **GPT-5.2** | 200K | General purpose, multimodal, agentic | Full text + image + audio + video |
| **GPT-5.2 mini** | 200K | Fast, cost-effective tasks | Best OpenAI budget option |
| **GPT-5.2-Codex** | 200K | Agentic coding, autonomous dev tasks | Powers GitHub Copilot Workspace |
| **o4** | 200K | Advanced reasoning, maths, research | Extended thinking before answer |
| **o4-mini** | 200K | Fast reasoning, cost-efficient | Best OpenAI reasoning/cost ratio |

**Key strengths:**
- Largest ecosystem: plugins, tools, integrations
- GPT-5.2 is fully multimodal (text, image, audio, video, code)
- o-series models excel at step-by-step mathematical and logical reasoning
- Strong function calling and tool use implementation
- Mature API with extensive community documentation

**Unique differentiators:**
- GPT-5.2-Codex is purpose-built for agentic software development tasks
- The o4 "thinking" model spends tokens on internal chain-of-thought before responding — best for puzzles, proofs, and complex algorithmic reasoning

---

### 1.3 Google DeepMind — Gemini Family

Google's Gemini 3 models are deeply integrated into the Google ecosystem and hold the record for the largest production context windows available.

**Current Gemini Models:**

| Model | API String | Context | Best For |
| :--- | :--- | :---: | :--- |
| **Gemini 3.1 Pro** | `gemini-3.1-pro-preview` | 1M | Long-context reasoning, whole-codebase analysis |
| **Gemini 3.1 Flash** | `gemini-3.1-flash` | 1M | Fast long-document processing, scale |
| **Gemini 3.0 Flash** | `gemini-3.0-flash` | 1M | Real-time apps, voice, high-volume |

> ⚠️ **Deprecation notice:** `gemini-3-pro-preview` is deprecated as of March 26, 2026. Use `gemini-3.1-pro-preview`.

**Key strengths:**
- 1 million token context window — ingest entire codebases or document libraries in one call
- Native multimodal from the ground up (text, code, image, audio, video)
- Deep Google integrations (Workspace, Search, Maps, Vertex AI)
- Strong performance on structured data and long-document tasks

**Unique differentiators:**
- The 1M token context window has no practical peer for whole-codebase analysis
- Google's TPU infrastructure delivers fast inference at massive scale

---

### 1.4 Meta — Llama Family

Meta's Llama 4 models are open-weights — you can download the weights and run them yourself. This is a fundamental differentiator for data privacy, custom training, or on-premises deployment.

**Current Llama 4 Models:**

| Model | Architecture | Context | Best For | Self-hostable? |
| :--- | :--- | :---: | :--- | :---: |
| **Llama 4 Scout** | MoE (17B active / 109B total) | 10M | Edge, single-GPU local, fast inference | ✅ Consumer GPU |
| **Llama 4 Maverick** | MoE (17B active / 400B total) | 1M | Strong coding and reasoning, self-hosted | ✅ Multi-GPU |
| **Llama 4 Behemoth** | ~2T parameters | 1M | Frontier-class research, teacher model | ⚠️ Research only |

**Key strengths:**
- Free to use, modify, and deploy
- Llama 4 Scout runs on a single consumer GPU — genuinely local-first
- Llama 4 Maverick matches GPT-5.2 quality on many benchmarks when self-hosted
- Community-driven fine-tunes for every use case
- No API costs for high-volume applications

**Deployment options:** Ollama, vLLM, Hugging Face TGI, LM Studio, Together.ai, Groq, AWS Bedrock.

---

### 1.5 Mistral AI

French AI lab producing highly efficient open and commercial models, known for punching above their weight in quality-per-parameter.

**Current Mistral Models:**

| Model | Type | Self-host? | Best For |
| :--- | :---: | :---: | :--- |
| **Mistral Large 3** | Proprietary | ❌ | Complex reasoning, multilingual, 128K context |
| **Mixtral 8x22B** | Open MoE | ✅ | General purpose, strong cost efficiency |
| **Codestral 2025** | Open | ✅ | Code gen, fill-in-the-middle, IDE completion |
| **Mistral Small 3** | Open | ✅ | Fast, lightweight, embedded tasks |

**Key strengths:**
- Mixture of Experts (MoE) architecture is highly efficient
- Strong multilingual support (especially European languages)
- Codestral is purpose-built for code generation
- Open models allow self-hosting

---

### 1.6 Cohere

Enterprise-focused models, particularly strong for RAG and enterprise search use cases.

**Models:** Command R+, Command R, Embed v3

**Unique strength:** Embed v3 is among the best embedding models for semantic search. Command R+ is specifically optimised for RAG workflows with tool use.

---

### 1.7 xAI — Grok

Elon Musk's AI lab, integrated with X (formerly Twitter). Grok models have access to real-time X data.

**Models:** Grok 3.5, Grok 3.5 mini

**Unique strength:** Real-time internet access and X data integration. Strong reasoning in Grok 3.5.

---

## 2. Open-Source vs. Closed-Source: The Core Trade-off

| Dimension | 🔒 Closed (Claude, GPT-5.2, Gemini) | 🔓 Open (Llama 4, Mistral) |
| :--- | :--- | :--- |
| **Quality** | Frontier — Claude Sonnet 4.6 SWE-bench 79.6% | Llama 4 Maverick competitive with GPT-5.2 |
| **Data privacy** | Data leaves your infrastructure | Full on-premises, zero data sharing |
| **Cost model** | Per-token API fees (predictable) | Infrastructure cost only (variable) |
| **Customisation** | Prompt-only, limited system fine-tuning | Full LoRA / QLoRA fine-tuning |
| **Latency** | Network-dependent (50–500ms p50) | Sub-10ms possible on local hardware |
| **Compliance** | DPA required (GDPR, HIPAA via BAA) | Full on-prem — no agreements needed |
| **Setup effort** | API key + SDK — minutes | GPU infra + vLLM/Ollama — hours/days |
| **Ecosystem** | Managed, versioned, guaranteed uptime | Community-maintained, self-managed |

---

## 3. Specialised Models Worth Knowing

### Code-Specialised
- **GitHub Copilot** (GPT-5.2-Codex based) — IDE autocomplete and agentic PR generation
- **Codestral 2025** (Mistral) — Code generation, strong on fill-in-the-middle
- **DeepSeek V3** — Strong open-source code model, cost-efficient
- **Llama 4 Maverick** — Meta's general-purpose model with strong coding; Code Llama deprecated in favour of Llama 4

### Embedding Models
- **OpenAI text-embedding-3-large** — Strong general-purpose embeddings
- **Cohere Embed v3** — Best-in-class for retrieval/RAG
- **all-MiniLM-L6-v2** (Sentence Transformers) — Lightweight, local-first

### Vision Models
- **GPT-5.2** — Full multimodal: vision, audio, video + text
- **Claude Sonnet 4.6** — Vision + document analysis, highest accuracy on OCR and structured documents
- **Llama 4 Scout / Maverick** — Open, self-hostable vision (native multimodal in Llama 4)
- **Gemini 3.1 Flash** (Google) — Fast vision + text at scale

### Audio / Speech
- **Whisper** (OpenAI, open-source) — Speech-to-text gold standard
- **ElevenLabs** — Best-in-class text-to-speech
- **Deepgram** — High-speed STT for real-time applications

---

## 4. Model Comparison by Task

| Task | 🥇 Best Choice | 🥈 Runner-Up | 💰 Budget Option |
| :--- | :--- | :--- | :--- |
| Complex coding | Claude Sonnet 4.6 | GPT-5.2-Codex | Codestral 2025 |
| Long document analysis | Gemini 3.1 Pro (1M ctx) | Claude Opus 4.6 | Claude Sonnet 4.6 |
| Mathematical / algorithmic reasoning | o4 | Gemini 3.1 Pro | Llama 4 Maverick |
| Fast API calls | Claude Haiku 4.5 | GPT-5.2 mini | Mistral Small 3 |
| RAG / retrieval | Claude Sonnet 4.6 | Command R+ | Llama 4 Scout |
| Multimodal (vision) | GPT-5.2 | Gemini 3.1 Pro | Llama 4 Scout |
| Privacy-sensitive tasks | Llama 4 Scout (local) | Mistral Large 3 | Llama 4 Maverick |
| Multilingual | Mistral Large 3 | GPT-5.2 | Llama 4 Maverick |
| Enterprise search | Cohere Command R+ | Claude Sonnet 4.6 | — |
| Real-time/speed | Groq + Llama 4 Scout | Gemini 3.1 Flash | Claude Haiku 4.5 |

---

## 5. How to Evaluate Models for Your Use Case

Don't just trust benchmarks. Benchmarks measure performance on standardised tests — your use case is not a standardised test.

**A practical evaluation framework:**

1. **Define your 20 representative tasks** — actual prompts you'll use in production
2. **Run each model on all 20 tasks**
3. **Score outputs** on: correctness, format adherence, tone, completeness
4. **Measure latency** — p50 and p95 response times
5. **Calculate cost per task** — tokens in × input rate + tokens out × output rate
6. **Choose the model with the best quality/cost/latency trade-off**

Don't over-index on the best model. The best model for your use case is the cheapest model that produces acceptable quality.

---

## 6. The Emerging Landscape: Agents and Multimodality

Models are evolving from **chat interfaces** to **agentic systems** — models that can use tools, browse the web, execute code, and complete multi-step workflows autonomously.

Key developments:
- **Function calling / tool use** — Models can call external APIs and code functions
- **Computer use** — Claude Sonnet 4.6 (OSWorld 72.5%) and GPT-5.2 can interact with computer interfaces autonomously
- **Code execution** — Models can run code in sandboxes and iterate based on output
- **Multi-agent systems** — Multiple LLMs collaborating on complex tasks

This is where the next wave of developer productivity gains will come from — not just answering questions, but completing entire workflows.

---

## Summary

The LLM landscape offers a rich spectrum of choices:
- **Anthropic Claude** — Best for coding, long-context, reliable instruction-following
- **OpenAI GPT-5.2 / o4** — Strongest multimodal, best reasoning (o4), massive ecosystem
- **Google Gemini 3.1 Pro** — Longest context (1M tokens), deeply integrated in Google stack
- **Meta Llama 4** — Best open-source option, Llama 4 Scout runs locally on consumer hardware
- **Mistral** — Efficient, multilingual, Codestral 2025 is the strongest open-source code model

In the next article, we'll go deep on specific models — which tasks each excels at, head-to-head comparisons, and a decision framework for building multi-model workflows.

---

*Next: Article 4 — Model Purpose and Task Matching: Which Model Wins at What?*
