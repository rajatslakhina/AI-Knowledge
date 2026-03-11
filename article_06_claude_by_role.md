# Article 6: How to Utilise Claude for Every Role — DEVs, QAs, and BAs

> *Role-specific workflows, prompt patterns, and automation strategies for developers, QA engineers, and business analysts.*

---

## Introduction

Claude is not a one-size-fits-all tool. A developer needs code generation and debugging. A QA engineer needs test coverage and scenario analysis. A business analyst needs requirements clarity and stakeholder communication. Each role has a distinct set of high-value use cases, and each benefits from tailored workflows.

This article gives you concrete, copy-paste-ready workflows for all three roles — not theory, but real tasks you can start doing today.

---

# PART 1: DEVELOPERS

## DEV Use Case 1: Feature Implementation

**Workflow: Spec to Code**

Before writing a single line of code, give Claude the full context:

```
You are an expert TypeScript developer working on a Node.js REST API.
Tech stack: Node.js 20, Express 4, TypeScript 5, PostgreSQL, Prisma ORM.
Code style: functional where possible, explicit typing, no any, JSDoc on public APIs.

Task: Implement a user invitation system.

Requirements:
- Admin can invite a user by email
- Generates a unique token (expires 48 hours)
- Sends invitation email (we use Nodemailer, SMTP config in process.env)
- Invited user completes registration with token
- Token is invalidated after use

Please implement:
1. Prisma schema additions
2. Service layer (invitation.service.ts)
3. Controller (invitation.controller.ts)
4. Routes (invitation.routes.ts)
5. Email template

Follow existing patterns in src/services/auth.service.ts
```

This prompt gives Claude: tech stack, code style, full requirements, file structure expectations, and a reference file to match patterns. The output will require far less editing than a vague prompt.

---

## DEV Use Case 2: Debugging

**Workflow: Error-Driven Debugging**

```
I'm getting this error in production. Help me identify the root cause and fix it.

Error:
TypeError: Cannot read properties of undefined (reading 'userId')
    at OrderService.createOrder (/app/src/services/order.service.ts:47:32)
    at async OrderController.create (/app/src/controllers/order.controller.ts:23:5)

Here is the relevant code:

[order.service.ts - paste the file]

[order.controller.ts - paste the file]

Context: This error happens only when a guest user (not logged in) tries to add an 
item to cart and we redirect them to checkout. The error started appearing after 
the auth middleware refactor last week.
```

**Key debug context to always include:**
1. Full stack trace
2. Relevant code files
3. When the error occurs (user action, conditions)
4. Recent changes that may have caused it

---

## DEV Use Case 3: Code Review Automation

**Workflow: Pre-PR Review**

Add this to your git workflow:

```bash
# .git/hooks/pre-push (or run manually)
#!/bin/bash
echo "Running AI code review..."
git diff main...HEAD | claude --model claude-sonnet-4-6 \
  "You are a senior engineer doing a code review. 
   Analyse this diff and provide:
   1. Security vulnerabilities (CRITICAL)
   2. Logic bugs or edge cases
   3. Performance concerns
   4. Missing tests
   5. Documentation gaps
   Format as a numbered list, sorted by severity."
```

---

## DEV Use Case 4: Test-Driven Development

**Workflow: Write Tests First, Then Implement**

```
I'm building a PricingCalculator class. Help me with TDD.

Requirements:
- Calculate price based on: base price, quantity, customer tier (standard/premium/enterprise)
- Premium gets 10% discount over 100 units, 15% over 500
- Enterprise gets 20% discount always, 25% over 200 units
- Apply VAT at rate specified in constructor (default 20%)
- Minimum order quantity: 10 units
- Maximum discount cannot exceed 30%

Step 1: Write failing Jest tests that fully specify this behaviour.
Do NOT write the implementation yet.
Use describe/it blocks, test edge cases, and use meaningful assertions.
```

Then, once you have tests:

```
Now implement the PricingCalculator class to make all these tests pass.
Follow SOLID principles and add JSDoc documentation.
```

---

## DEV Use Case 5: Refactoring Legacy Code

**Workflow: Incremental Refactoring**

```
Here is a legacy function that needs to be refactored. It works, but it's 
unmaintainable. Do NOT change its behaviour.

[paste legacy code]

Refactor this to:
1. Break into smaller single-responsibility functions
2. Add TypeScript types (currently vanilla JS)
3. Replace callbacks with async/await
4. Add proper error handling
5. Add JSDoc

Constraints:
- Public API (function signature) must stay identical
- All existing tests must still pass
- Do not use any external libraries not already in package.json
```

---

## DEV Use Case 6: Infrastructure and DevOps

**Workflow: IaC Generation**

```
Generate a GitHub Actions CI/CD pipeline for this Node.js project.

Requirements:
- Trigger on: PR to main, push to main
- Steps: 
  1. Install dependencies (npm ci)
  2. TypeScript check (tsc --noEmit)
  3. Lint (eslint)
  4. Unit tests (jest, with coverage)
  5. Build (npm run build)
  6. On push to main only: Deploy to AWS ECS
- Environment: Use GitHub Secrets for AWS credentials
- Fail fast on any step failure
- Cache node_modules between runs
- Coverage threshold: 80% or fail

Project uses: Node 20, npm, TypeScript, Jest, ESLint
```

---

# PART 2: QA ENGINEERS

## QA Use Case 1: Comprehensive Test Case Generation

**Workflow: Story to Test Suite**

```
You are a senior QA engineer. Generate a comprehensive test suite for this user story.

User Story:
As a user, I want to reset my password so that I can regain access if I forget it.

Acceptance Criteria:
- User can request reset via email
- Reset link expires after 24 hours
- Password must meet complexity rules (8+ chars, 1 uppercase, 1 number, 1 special)
- Old password is invalidated immediately after reset
- User receives confirmation email after successful reset

Generate:
1. Positive test cases (happy path)
2. Negative test cases (invalid inputs, expired tokens)
3. Boundary test cases (exactly 8 chars, 24h boundary, etc.)
4. Security test cases (token reuse, brute force, concurrent resets)
5. UX/accessibility test cases

Format each test case as:
- Test ID
- Test Title
- Preconditions
- Steps
- Expected Result
- Priority (P0/P1/P2)
```

---

## QA Use Case 2: Test Data Generation

**Workflow: Schema to Seed Data**

```
Generate test data for our user registration flow.

Schema:
- email: valid email format, unique
- username: 3-20 chars, alphanumeric + underscore, unique
- password: 8-72 chars, minimum complexity
- dateOfBirth: must be 18+ years old
- country: ISO 3166-1 alpha-2

Generate:
1. 5 valid standard users
2. 3 users at boundary ages (exactly 18, 17 years 364 days, very old)
3. 5 invalid email formats
4. 5 invalid usernames (too short, too long, special chars, SQL injection attempt, XSS attempt)
5. 5 weak passwords that fail complexity rules
6. 1 user for each of these countries: GB, US, DE, JP, IN

Format as JSON array.
```

---

## QA Use Case 3: Exploratory Testing Charters

**Workflow: Feature to Testing Charter**

```
Write exploratory testing charters for our new checkout flow.

Feature: Multi-item shopping cart with coupon codes and multiple payment methods.

Payment methods: Visa, Mastercard, PayPal, Apple Pay, bank transfer.
Coupon types: Percentage discount, fixed amount, free shipping, BOGO.

Write 5 exploratory testing charters covering:
1. Payment flow edge cases
2. Coupon stacking and conflict scenarios
3. Cart persistence and session management
4. Multi-tab and concurrent session behaviour
5. Error recovery and partial failures

Each charter should specify:
- Mission
- Target area
- Hints to explore
- Time box (minutes)
```

---

## QA Use Case 4: API Test Script Generation

**Workflow: OpenAPI Spec to Test Suite**

```
Generate a Playwright API test suite for this endpoint.

Endpoint: POST /api/v1/orders
Content-Type: application/json

Request body:
{
  "items": [{"productId": "string", "quantity": number}],
  "shippingAddressId": "string",
  "paymentMethodId": "string",
  "couponCode": "string (optional)"
}

Response 201:
{
  "orderId": "string",
  "totalAmount": number,
  "estimatedDelivery": "ISO8601 date",
  "status": "pending"
}

Error responses: 400 (validation), 401 (auth), 404 (product not found), 
409 (out of stock), 422 (coupon invalid), 500 (server error)

Auth: Bearer token required.

Generate tests covering all status codes, input validation, and auth scenarios.
Use Playwright's APIRequestContext. Include setup/teardown.
```

---

## QA Use Case 5: Bug Report Writing

**Workflow: Observation to Formal Bug Report**

```
Help me write a formal bug report from these notes.

My notes:
"checkout breaks on safari when you have more than 5 items and apply a discount 
code. the total shows wrong. doesnt happen on chrome. started after last release."

Write a complete bug report with:
- Title (clear, specific)
- Severity and Priority assessment with justification
- Environment (ask me what you need)
- Preconditions
- Steps to reproduce
- Expected result
- Actual result
- Impact assessment
- Suggested investigation areas

Then list what additional information you'd need from me to complete the report.
```

---

# PART 3: BUSINESS ANALYSTS

## BA Use Case 1: Requirements Extraction from Meetings

**Workflow: Transcript to Requirements**

```
You are a business analyst. Extract structured requirements from this meeting transcript.

[paste meeting transcript]

Extract and organise:

1. FUNCTIONAL REQUIREMENTS
   - User stories (As a... I want... So that...)
   - Business rules
   - Constraints

2. NON-FUNCTIONAL REQUIREMENTS
   - Performance requirements
   - Security requirements
   - Availability/reliability

3. ASSUMPTIONS (stated or implied)

4. OPEN QUESTIONS (things not resolved in the meeting)

5. DEPENDENCIES (on other teams, systems, or decisions)

6. RISKS (mentioned or implied)

Format as a structured document ready to share with the development team.
```

---

## BA Use Case 2: User Story Writing

**Workflow: Feature Idea to Sprint-Ready Stories**

```
Break this feature into sprint-ready user stories with full acceptance criteria.

Feature: Customer portal where users can manage their subscription.

Capabilities needed:
- View current plan and billing date
- Upgrade or downgrade plan
- Update payment method
- View and download invoices
- Cancel subscription
- Pause subscription (max 3 months)

For each user story:
1. Write the story in "As a / I want / So that" format
2. Write acceptance criteria in Given/When/Then (BDD) format
3. Add a Definition of Done checklist
4. Estimate story points (Fibonacci: 1,2,3,5,8,13)
5. Identify dependencies
6. Flag technical risks or questions for dev team

Target: Stories should be independently deliverable within a 2-week sprint.
Suggest how to sequence them (what's MVP, what's phase 2).
```

---

## BA Use Case 3: Gap Analysis

**Workflow: Requirements vs. Technical Spec**

```
Perform a gap analysis between these two documents.

BUSINESS REQUIREMENTS (from stakeholder sign-off):
[paste requirements document]

TECHNICAL SPECIFICATION (from dev team):
[paste tech spec]

Identify:
1. Requirements in business doc NOT addressed in tech spec
2. Technical features in spec with no corresponding requirement
3. Contradictions between the two documents
4. Ambiguous requirements that could be interpreted multiple ways
5. Requirements where technical approach may not meet business goal

Format as a table: Gap | Type | Business Requirement | Technical Spec | Risk Level | Recommended Action
```

---

## BA Use Case 4: Stakeholder Communication

**Workflow: Technical Update to Executive Summary**

```
Translate this technical update into an executive-level summary.

Technical update:
[paste sprint report, incident report, or technical document]

Audience: C-suite and non-technical stakeholders
Tone: Professional, clear, no jargon
Format:
- 3-sentence summary (what happened, impact, status)
- Business impact (revenue, customers, operations)
- What we did / are doing
- Timeline and next steps
- Risks still outstanding

Avoid: technical terms without explanation, passive voice, vague language.
```

---

## BA Use Case 5: Process Mapping

**Workflow: Description to Mermaid Diagram**

```
Convert this business process description into a Mermaid sequence diagram.

Process: Customer makes a purchase on our platform.

Steps:
1. Customer adds items to cart
2. Customer proceeds to checkout
3. System validates inventory
4. Customer enters shipping address
5. System calculates shipping cost and tax
6. Customer enters payment details
7. Payment processor authorises payment
8. On success: order created, inventory updated, confirmation email sent
9. On failure: customer notified, cart preserved

Include: decision points for out-of-stock and payment failure.
Actors: Customer, Frontend, Backend API, Inventory Service, Payment Processor, Email Service

Output: Valid Mermaid sequenceDiagram code.
```

---

## Role Workflow Summary

### Quick Reference: Use Cases by Role

| Use Case | DEV | QA | BA | Recommended Model |
| :--- | :---: | :---: | :---: | :--- |
| Feature implementation | ✅ | — | — | Sonnet 4.6 |
| Test case generation | — | ✅ | — | Sonnet 4.6 |
| User story writing | — | — | ✅ | Sonnet 4.6 |
| Code review / PR review | ✅ | ✅ | — | Sonnet 4.6 |
| Debugging & root cause | ✅ | ✅ | — | Sonnet 4.6 → Opus 4.6 |
| Requirements extraction | — | — | ✅ | Sonnet 4.6 |
| Test data generation | — | ✅ | — | Haiku 4.5 |
| Architecture & design | ✅ | — | ✅ | Opus 4.6 |
| Gap analysis | — | — | ✅ | Opus 4.6 |
| IaC / pipeline generation | ✅ | — | — | Sonnet 4.6 |
| Commit / PR descriptions | ✅ | — | — | Haiku 4.5 |
| Stakeholder communication | — | — | ✅ | Sonnet 4.6 |
| Exploratory test charters | — | ✅ | — | Sonnet 4.6 |
| Meeting summarisation | ✅ | ✅ | ✅ | Haiku 4.5 |

---

### Daily AI Workflow for Developers

| Time of Day | Task | Tool / Alias |
| :--- | :--- | :--- |
| **Morning** | Review overnight PRs | `ai-review` |
| **Morning** | Plan implementation tasks | `claude "Plan today's tasks for [feature]"` |
| **Development** | Implement a feature | `claude "Implement [task] following [ref file]"` |
| **Development** | Debug an error | `claude "Debug: [error + code]"` |
| **Development** | Generate tests | `claude "Generate tests for what I just wrote"` |
| **End of day** | Commit message | `ai-commit` |
| **End of day** | Self-review before push | `ai-review` |

### Daily AI Workflow for QA

| Phase | Task | Tool / Command |
| :--- | :--- | :--- |
| **Sprint planning** | Generate test cases from stories | `claude "Generate test suite for: [story]"` |
| **Development** | Generate test data for endpoint | `claude "Generate test data for: [spec]"` |
| **Development** | Write Playwright tests | `claude "Write Playwright tests for: [spec]"` |
| **Bug reporting** | Formalise bug notes | `claude "Write formal bug report from: [notes]"` |
| **Release** | Regression test planning | `claude "Generate regression test plan for: [release]"` |

### Daily AI Workflow for BAs

| Phase | Task | Tool / Command |
| :--- | :--- | :--- |
| **Requirements** | Extract from meeting transcript | `claude "Extract requirements from: [transcript]"` |
| **Story writing** | Break feature into sprint stories | `claude "Break into stories: [feature description]"` |
| **Communication** | Translate tech update → exec summary | `claude "Write stakeholder update for: [context]"` |
| **Gap analysis** | Requirements vs. tech spec | `claude "Gap analysis between: [doc A] and [doc B]"` |
| **Diagramming** | Convert process to Mermaid | `claude "Convert to Mermaid sequence: [process]"` |

---

## Summary

The key to effective AI-assisted work is specificity. Vague prompts produce vague outputs. Every workflow above shares the same pattern: **provide role, context, constraints, and expected format**. The more precise your input, the more usable the output.

In the next article, we'll go deep on the science of prompt engineering — how to consistently get high-quality, production-ready outputs from Claude.

---

*Next: Article 7 — How to Write Effective and Optimised Prompts for Best Results*
