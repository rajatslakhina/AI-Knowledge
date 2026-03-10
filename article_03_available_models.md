# Article 3: Which AI Models Are Available? A Developer's Map of the LLM Landscape

> *GPT, Claude, Gemini, Llama, Mistral — cutting through the noise to understand what each model family offers and when to choose each.*

---

## Introduction

The LLM landscape in 2024–2025 is rich, noisy, and moves fast. New model releases and benchmarks appear weekly, each claiming to be "state of the art." For developers building real products, the signal-to-noise ratio can be brutal.

This article maps the major model families, their capabilities, and their practical trade-offs — so you can make informed decisions rather than chasing benchmarks.

---

## 1. The Major Players

### 1.1 Anthropic — Claude Family

Anthropic was founded in 2021 by former OpenAI researchers, including Dario and Daniela Amodei. Their mission is "AI safety" — building powerful AI while ensuring it remains aligned, honest, and harmless. This philosophy is baked into Claude at the model level (Constitutional AI).

**Current Claude Model Family:**

| Model | Context Window | Best For |
|---|---|---|
| **Claude Opus 4** | 200K tokens | Complex reasoning, research, architecture |
| **Claude Sonnet 4.5** | 200K tokens | Balanced: speed + intelligence, coding |
| **Claude Haiku 4.5** | 200K tokens | Fast tasks, high-volume, summarisation |

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

---

### 1.2 OpenAI — GPT Family

OpenAI is the company that popularised LLMs with ChatGPT in late 2022. Their GPT and o-series models remain dominant in market share.

**Current OpenAI Models:**

| Model | Context Window | Best For |
|---|---|---|
| **GPT-4o** | 128K tokens | General purpose, multimodal |
| **GPT-4o mini** | 128K tokens | Fast, cost-effective tasks |
| **o1** | 200K tokens | Complex mathematical reasoning |
| **o3 / o3-mini** | 200K tokens | Advanced reasoning, research |
| **o4-mini** | 200K tokens | Fast reasoning tasks |

**Key strengths:**
- Largest ecosystem: plugins, tools, integrations
- GPT-4o is multimodal (text, image, audio, video)
- o-series models excel at step-by-step mathematical and logical reasoning
- Strong function calling and tool use implementation
- Mature API with extensive community documentation

**Unique differentiators:**
- OpenAI leads in multimodal capabilities
- The o-series "thinking" models spend time on internal chain-of-thought before responding — best for puzzles, proofs, and complex reasoning

---

### 1.3 Google DeepMind — Gemini Family

Google's Gemini models are deeply integrated into the Google ecosystem and offer some of the largest context windows available.

**Current Gemini Models:**

| Model | Context Window | Best For |
|---|---|---|
| **Gemini 2.5 Pro** | 1M tokens | Ultra long-context, research |
| **Gemini 2.5 Flash** | 1M tokens | Fast, efficient, long-document |
| **Gemini 2.0 Flash** | 1M tokens | Real-time apps, voice |

**Key strengths:**
- 1 million token context window (industry-leading)
- Native multimodal from the ground up (trained on text, code, image, audio, video simultaneously)
- Deep Google integrations (Workspace, Search, Maps)
- Strong performance on structured data tasks

**Unique differentiators:**
- The 1M token context window enables ingesting entire codebases or document libraries
- Google's infrastructure advantage means extremely fast inference at scale

---

### 1.4 Meta — Llama Family

Meta's Llama models are open-weights — you can download the weights and run them yourself. This is a fundamental differentiator for use cases where data privacy, custom training, or on-premises deployment are required.

**Current Llama Models:**

| Model | Parameters | Best For |
|---|---|---|
| **Llama 3.3 70B** | 70B | High-quality open-source, coding |
| **Llama 3.1 8B** | 8B | Edge, embedded, fast inference |
| **Llama 3.2 Vision** | 11B/90B | Multimodal, vision tasks |
| **Llama 3.1 405B** | 405B | Near-frontier, self-hosted |

**Key strengths:**
- Free to use, modify, and deploy
- Can be run locally or on your own infrastructure
- Community-driven fine-tunes for every use case
- No API costs for high-volume applications

**Deployment options:** Ollama, vLLM, Hugging Face TGI, LM Studio, Together.ai, Groq, AWS Bedrock.

---

### 1.5 Mistral AI

French AI lab producing highly efficient open and commercial models, known for punching above their weight in quality-per-parameter.

**Current Mistral Models:**

| Model | Type | Best For |
|---|---|---|
| **Mistral Large 2** | Proprietary | Complex reasoning, multilingual |
| **Mixtral 8x22B** | Open (MoE) | General purpose, cost-efficient |
| **Codestral** | Open | Code generation |
| **Mistral Small** | Open | Fast, lightweight tasks |

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

**Models:** Grok 3, Grok 3 mini

**Unique strength:** Real-time internet access and X data integration. Strong reasoning in Grok 3.

---

## 2. Open-Source vs. Closed-Source: The Core Trade-off

| Dimension | Closed (GPT-4, Claude, Gemini) | Open (Llama, Mistral) |
|---|---|---|
| **Quality** | Generally frontier quality | Catching up fast |
| **Data privacy** | Your data leaves your infra | Full data control |
| **Cost** | Per-token API fees | Infrastructure cost only |
| **Customisation** | Limited fine-tuning | Full fine-tuning possible |
| **Latency** | Dependent on API | Can be optimised locally |
| **Compliance** | Data processing agreements needed | Full on-prem possible |
| **Setup** | Simple API key | Infrastructure investment |

---

## 3. Specialised Models Worth Knowing

### Code-Specialised
- **GitHub Copilot** (GPT-4o based) — IDE autocomplete leader
- **Codestral** (Mistral) — Code generation, strong on fill-in-the-middle
- **DeepSeek Coder V2** — Strong open-source code model
- **Code Llama** — Meta's code-focused Llama fine-tune

### Embedding Models
- **OpenAI text-embedding-3-large** — Strong general-purpose embeddings
- **Cohere Embed v3** — Best-in-class for retrieval/RAG
- **all-MiniLM-L6-v2** (Sentence Transformers) — Lightweight, local-first

### Vision Models
- **GPT-4o** — Versatile vision + text
- **Claude 3.5 Sonnet** — Vision + document analysis
- **Llama 3.2 Vision** — Open, self-hostable vision
- **PaliGemma** (Google) — Efficient vision-language model

### Audio / Speech
- **Whisper** (OpenAI, open-source) — Speech-to-text gold standard
- **ElevenLabs** — Best-in-class text-to-speech
- **Deepgram** — High-speed STT for real-time applications

---

## 4. Model Comparison by Task

| Task | Best Choice | Runner-Up | Budget Option |
|---|---|---|---|
| Complex coding | Claude Sonnet 4.5 | GPT-4o | Codestral |
| Long document analysis | Gemini 2.5 Pro | Claude Opus 4 | Claude Sonnet 4.5 |
| Mathematical reasoning | o3 / o1 | Gemini 2.5 Pro | Llama 3.3 70B |
| Fast API calls | Claude Haiku 4.5 | GPT-4o mini | Mistral Small |
| RAG / retrieval | Claude Sonnet 4.5 | Command R+ | Llama 3.3 70B |
| Multimodal (vision) | GPT-4o | Gemini 2.5 Pro | Llama 3.2 Vision |
| Privacy-sensitive tasks | Llama 3.3 70B (local) | Mistral Large 2 | Llama 3.1 8B |
| Multilingual | Mistral Large 2 | GPT-4o | Llama 3.3 70B |
| Enterprise search | Cohere Command R+ | Claude Sonnet 4.5 | — |
| Real-time/speed | Groq + Llama | Gemini Flash | Claude Haiku 4.5 |

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
- **Computer use** — Claude and GPT-4o can interact with computer interfaces
- **Code execution** — Models can run code in sandboxes and iterate based on output
- **Multi-agent systems** — Multiple LLMs collaborating on complex tasks

This is where the next wave of developer productivity gains will come from — not just answering questions, but completing entire workflows.

---

## Summary

The LLM landscape offers a rich spectrum of choices:
- **Anthropic Claude** — Best for coding, long-context, reliable instruction-following
- **OpenAI GPT/o-series** — Best ecosystem, strongest reasoning (o-series), multimodal leader
- **Google Gemini** — Longest context, deeply integrated in Google stack
- **Meta Llama** — Best open-source option, essential for data-sensitive deployments
- **Mistral** — Efficient, multilingual, strong open-source code model

In the next article, we'll go deep on specific models — which tasks each excels at, head-to-head comparisons, and a decision framework for building multi-model workflows.

---

*Next: Article 4 — Model Purpose and Task Matching: Which Model Wins at What?*
