# Article 2: Where Can We Use AI? A Comprehensive Map of AI Applications in Software Engineering

> *Beyond the hype — a practical, role-by-role breakdown of where AI delivers real value and where it still falls short.*

---

## Introduction

Every week there's a new announcement: "AI can now do X." But knowing what AI *can* do and knowing where it *should* be used are two different skills. The developers who extract the most value from AI aren't the ones who throw everything at it — they're the ones who understand which problems AI solves well, which it solves partially, and which it shouldn't touch.

This article maps the AI application landscape for software engineering teams — developers, QA engineers, and business analysts — with honest assessments of where the ROI is real.

---

## 1. The Application Spectrum

Think of AI applications on two axes:

```
                    HIGH CREATIVITY
                          │
         Content Gen      │    Architectural Design
         (articles,       │    (system design,
          marketing)      │     trade-off analysis)
                          │
LOW ──────────────────────┼────────────────────── HIGH
AUTONOMY                  │                    AUTONOMY
                          │
         Data Extraction  │    Automated Testing
         (parsing,        │    (full test suites,
          classification) │     CI integration)
                          │
                    LOW CREATIVITY
```

Most enterprise AI use cases cluster in the **high autonomy, lower creativity** quadrant — automation, classification, code generation following patterns.

---

## 2. Software Development Use Cases

### 2.1 Code Generation
**Maturity: ★★★★☆ — Production-ready with review**

AI excels at generating boilerplate, implementing well-defined patterns, and translating specifications into code. The sweet spot:

- CRUD endpoints from a schema
- Unit test scaffolding from a function signature
- Data transformation functions with clear input/output
- Regular expressions from natural language descriptions
- SQL queries from business requirements
- Type definitions and interfaces from JSON samples

```
Example prompt that works well:
"Write a TypeScript function that takes an array of User objects 
 (id: string, email: string, createdAt: Date) and returns them 
 grouped by month of creation. Include JSDoc."
```

**Where it struggles:**
- Novel algorithms without clear precedent in training data
- Code requiring deep understanding of your specific domain model
- Performance-critical code (the model optimises for correctness, not speed)
- Code that requires knowing your exact production environment

### 2.2 Code Review
**Maturity: ★★★★☆ — Excellent as a first-pass reviewer**

AI can scan a diff or file for:
- Common bugs (null pointer risks, off-by-one errors, race conditions)
- Security vulnerabilities (SQL injection, XSS, insecure deserialization)
- Code smell (long methods, deep nesting, missing error handling)
- Documentation gaps
- Inconsistencies with common patterns

**Workflow:** Use AI review before human review. It catches the low-hanging fruit so human reviewers focus on architecture and business logic.

### 2.3 Debugging and Root Cause Analysis
**Maturity: ★★★★☆**

Paste an error, a stack trace, or a failing function. AI is remarkably good at:
- Identifying the likely cause from a stack trace
- Explaining what an error message means
- Suggesting fixes with explanation
- Identifying subtle issues like closure bugs, async/await misuse, type coercion

### 2.4 Refactoring
**Maturity: ★★★★☆**

AI handles many refactoring tasks well:
- Extract method/class
- Convert callbacks to async/await
- Migrate from one library to another (e.g., Axios to Fetch)
- Update deprecated API usage
- Translate code between languages (JavaScript → TypeScript, Python 2 → 3)

**Important:** Always test after AI-driven refactoring. It can introduce subtle regressions, especially in stateful code.

### 2.5 Documentation
**Maturity: ★★★★★ — One of the strongest use cases**

AI writes documentation faster than any human. Use it for:
- JSDoc / docstrings from existing code
- README files from project structure
- API documentation from route handlers
- Architecture decision records (ADRs)
- Runbooks and operational procedures
- Onboarding guides from codebase exploration

The cognitive asymmetry here is significant: humans find documentation tedious; AI does not.

### 2.6 Architecture and Design
**Maturity: ★★★☆☆ — Great for brainstorming, not for final decisions**

AI is an excellent rubber duck for architecture:
- "What are the trade-offs between Event Sourcing and a traditional CRUD model for this use case?"
- "Design a system that handles 100k concurrent websocket connections"
- "What are the failure modes of this microservice design?"

But AI doesn't know your team's skills, your existing infrastructure, or your business constraints. Use it for option generation and trade-off analysis, not as the final decision-maker.

---

## 3. QA and Testing Use Cases

### 3.1 Test Case Generation
**Maturity: ★★★★★**

Given a function, class, or user story, AI generates comprehensive test cases including:
- Happy path tests
- Edge cases (empty arrays, null values, max boundaries)
- Error cases and exception handling
- Security-focused tests (injection, boundary overflow)

This is arguably the single highest-ROI use case for QA teams. A task that takes 2–3 hours is done in 10 minutes.

### 3.2 Test Data Generation
**Maturity: ★★★★☆**

Generating realistic, structured test data:
- Fake user profiles with realistic but random PII
- JSON/XML payloads for API testing
- Database seed data matching a schema
- Edge-case data sets (unicode characters, very long strings, negative numbers)

### 3.3 Bug Report Triage
**Maturity: ★★★☆☆**

AI can help classify incoming bug reports:
- Duplicate detection
- Severity assessment
- Routing to the right team
- Extracting structured data from freeform text reports

### 3.4 Accessibility Testing Guidance
**Maturity: ★★★☆☆**

AI can review HTML/CSS for accessibility issues, suggest ARIA labels, and check colour contrast ratios against WCAG standards when given markup.

### 3.5 Performance Testing Scripts
**Maturity: ★★★☆☆**

Generating k6, JMeter, or Locust scripts from API specifications. AI can scaffold the structure; domain-specific load profiles still need human tuning.

---

## 4. Business Analysis Use Cases

### 4.1 Requirements Extraction and Structuring
**Maturity: ★★★★☆**

Transform vague stakeholder notes into structured requirements:
- Convert meeting transcripts into user stories
- Identify functional vs. non-functional requirements
- Flag ambiguities and missing acceptance criteria
- Generate BDD-style Given/When/Then scenarios

### 4.2 User Story Writing
**Maturity: ★★★★★**

One of the highest-ROI BA tasks. Given a feature description, AI produces properly formatted user stories with acceptance criteria in minutes.

```
Input: "Users should be able to reset their password"

Output:
As a registered user,
I want to reset my forgotten password via email,
So that I can regain access to my account without contacting support.

Acceptance Criteria:
- GIVEN I am on the login page
  WHEN I click "Forgot Password" and enter my email
  THEN I receive a reset link within 2 minutes

- GIVEN I click the reset link
  WHEN the link is less than 24 hours old
  THEN I can set a new password

- GIVEN I click the reset link  
  WHEN the link is more than 24 hours old
  THEN I see an "expired link" error and option to request a new one
```

### 4.3 Gap Analysis
**Maturity: ★★★★☆**

Compare a requirements document against a technical spec or existing system to identify gaps, contradictions, or unaddressed scenarios.

### 4.4 Data Analysis and Reporting
**Maturity: ★★★★☆**

AI + CSV/database data is a powerful combination:
- Summarise trends in business data
- Generate Python/SQL analysis scripts
- Produce executive summaries from raw metrics
- Identify anomalies in datasets

### 4.5 Diagram Generation
**Maturity: ★★★★☆**

AI can generate Mermaid, PlantUML, or draw.io XML for:
- Sequence diagrams from process descriptions
- ER diagrams from schema definitions
- Flow charts from business logic
- C4 context diagrams from architecture descriptions

---

## 5. DevOps and Infrastructure

### 5.1 CI/CD Pipeline Configuration
Generating GitHub Actions, GitLab CI, or Jenkins pipelines from project structure descriptions.

### 5.2 Infrastructure as Code
Terraform, CloudFormation, and Kubernetes manifests from high-level descriptions. Review carefully — IAM and security configurations require extra scrutiny.

### 5.3 Log Analysis
Parsing large log files and identifying patterns, error spikes, or anomalies. Feed AI a sample log and ask it to write a parsing script.

### 5.4 Incident Response
During an incident, AI can help rapidly search runbooks, draft status updates, and suggest remediation steps based on symptoms.

---

## 6. Communication and Collaboration

### 6.1 Technical Writing
- Release notes from git diffs or JIRA tickets
- Sprint retrospective summaries
- RFC (Request for Comments) drafts
- Post-mortems from incident timelines

### 6.2 Code Explanation
Explaining complex code to non-technical stakeholders, junior developers, or for onboarding purposes. Paste a function and ask AI to explain it at various levels of technical depth.

### 6.3 Meeting Summaries
Feed AI a meeting transcript (from Zoom, Teams, or a manual transcript) and receive structured summaries with action items, decisions made, and open questions.

---

## 7. Where AI Does NOT Belong (Yet)

Be deliberate about these boundaries:

| Use Case | Why AI Falls Short | Safe Mitigation |
| :--- | :--- | :--- |
| **Unreviewed production deploys** | Non-deterministic output + no domain context | Always require human sign-off before deploy |
| **Security-critical algorithm design** | Hallucination risk in crypto/auth logic | Use AI to explore options; expert signs off |
| **Database migrations on live data** | Irreversible operations — one mistake = data loss | AI drafts migration; DBA validates + dry-runs |
| **Legal/compliance decisions** | Hallucinated rules can become real liability | AI for first draft; legal counsel reviews |
| **Final architecture decisions** | AI lacks your team skills and business context | AI for trade-off analysis; team makes decision |
| **Real-time data retrieval (without RAG)** | Knowledge cutoff means stale answers | Add RAG pipeline; always cite sources |
| **Autonomous financial transactions** | No way to undo sent money or cancelled orders | Human approval gate required |

---

## 8. The 80/20 Rule for AI Adoption

The highest-ROI applications for most engineering teams:

1. **Code generation for boilerplate** (saves 30–40% of typing)
2. **Test case generation** (saves 60–70% of test writing time)
3. **Documentation writing** (saves 70–80% of doc writing time)
4. **User story and AC generation** (saves 50–60% of BA writing time)
5. **Bug triage and debugging assistance** (saves 20–40% of debugging time)

Start here. Get 10x value from these five use cases before expanding to more complex workflows.

---

## 9. AI in the Development Lifecycle

| 📋 Requirements | 🏗️ Design | 💻 Development | 🧪 QA | 🚀 Release |
| :--- | :--- | :--- | :--- | :--- |
| User story gen | Architecture brainstorm | Code generation | Test case gen | Release notes |
| Acceptance criteria | Diagram generation | Code review | Test data gen | Runbook gen |
| Gap analysis | Trade-off analysis | Debugging | Bug triage | Post-mortems |
| Stakeholder comms | ADR writing | Refactoring | Accessibility checks | Status updates |
| Requirements extraction | API contract design | Documentation | E2E test scripts | Changelog gen |

---

## Summary

AI is applicable across the entire software development lifecycle. The highest-value applications share three traits:
1. They involve **well-defined inputs and outputs**
2. They are **high-volume and repetitive** (high human cost)
3. They **benefit from human review** before the output is used

In the next article, we'll survey the landscape of available AI models — from OpenAI and Anthropic to open-source alternatives — so you can choose the right tool for each job.

---

*Next: Article 3 — Which AI Models Are Available? A Developer's Map of the LLM Landscape*
