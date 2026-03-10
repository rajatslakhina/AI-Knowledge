# Article 7: How to Write Effective and Optimised Prompts to Get the Best Results

> *Prompt engineering is a skill. Master it and you'll get production-quality output from AI. Ignore it and you'll spend more time editing AI output than writing from scratch.*

---

## Introduction

The most common complaint about AI tools is: "The outputs aren't good enough to use." Nine times out of ten, the problem isn't the model — it's the prompt.

Prompt engineering is the discipline of crafting inputs that consistently produce high-quality, usable outputs. It's not magic; it's structured communication. This article covers the principles, patterns, and anti-patterns that separate prompts that produce copy-paste-ready output from prompts that produce frustrating first drafts.

---

## 1. The Anatomy of a High-Quality Prompt

A high-quality prompt has six components. Not all are required for every task, but knowing them lets you decide which to include.

```
┌─────────────────────────────────────────────────────────────┐
│  1. ROLE     — Who Claude should be                         │
│  2. CONTEXT  — Background information Claude needs          │
│  3. TASK     — What you want Claude to do                   │
│  4. FORMAT   — How you want the output structured           │
│  5. CONSTRAINTS — What to avoid, rules to follow            │
│  6. EXAMPLES — Demonstrations of expected output            │
└─────────────────────────────────────────────────────────────┘
```

### The Difference It Makes

**Weak prompt:**
> "Write me a function to process payments"

**Strong prompt:**
> You are a senior TypeScript developer working on a fintech application.
> 
> Context: We use Stripe as our payment processor. Our existing code uses async/await, TypeScript strict mode, and follows the repository pattern. See reference: `src/services/auth.service.ts`.
> 
> Task: Write a `PaymentService` class with a `processPayment(amount: number, currency: string, paymentMethodId: string, customerId: string): Promise<PaymentResult>` method.
> 
> Requirements:
> - Handle Stripe errors gracefully (card declined, insufficient funds, network errors)
> - Log all payment attempts (success and failure) using our Winston logger
> - Return a typed `PaymentResult` object
> - Include JSDoc documentation
> 
> Constraints:
> - No try/catch at the controller layer — errors should propagate as typed exceptions
> - Do not log card numbers or sensitive payment data
> 
> Format: TypeScript class with interfaces, no default exports.

The second prompt produces something you can use immediately. The first produces something you'll rewrite.

---

## 2. The Role Pattern

Assigning Claude a role ("You are a...") primes the model to respond with the vocabulary, depth, and perspective of that expert.

**Generic role:**
```
"You are a developer."
```

**Effective role:**
```
"You are a senior backend engineer with 10 years of experience in distributed 
systems, specifically Node.js microservices on AWS. You prioritise reliability 
and observability in everything you build."
```

The effective role communicates:
- Seniority level (depth of response)
- Domain specialisation (vocabulary, patterns used)
- Values (what trade-offs the expert makes)

**Role examples by task:**

| Task | Role |
|---|---|
| Code review | "You are a security-focused senior engineer who has seen production incidents caused by the exact bugs you're looking for." |
| User stories | "You are a product manager with 8 years experience writing stories for agile teams, with deep understanding of BDD." |
| Architecture | "You are a principal engineer who has designed systems at scale for millions of users and has strong opinions about reliability." |
| Test cases | "You are a QA lead who approaches testing adversarially — you're trying to break the system." |
| Documentation | "You are a technical writer who values clarity and consistency and despises jargon." |

---

## 3. The Context Principle

Claude can only use information you give it. The single most impactful thing you can do to improve output quality is provide more relevant context.

**Three types of context:**

**1. Codebase context** — What does the existing code look like?
```
Here is our existing auth service as a reference for coding patterns: [paste file]
```

**2. Constraints context** — What rules apply?
```
Constraints:
- We are on Node.js 18 (no Fetch API)
- No new npm dependencies without approval
- Must work in both EU and US regions (GDPR compliance required)
```

**3. Goal context** — Why are you doing this?
```
Context: This endpoint will be called by our mobile app 10,000 times per second 
at peak load. Performance is critical.
```

**The context checklist:**
- [ ] What tech stack / language / version?
- [ ] What existing patterns should be followed?
- [ ] What are the business constraints?
- [ ] Who will use this output?
- [ ] What has already been tried?

---

## 4. Task Decomposition

For complex tasks, don't ask Claude to do everything in one shot. Break it down.

**Anti-pattern (one giant prompt):**
```
"Write me a complete e-commerce platform with user auth, product catalogue, 
cart, checkout, payment processing, order management, and admin dashboard."
```

**Pattern (decomposed):**
```
Step 1: "Design the database schema for an e-commerce platform with these 
        requirements. Output: Prisma schema."

Step 2: "Based on this schema, write the Prisma seed file with realistic test data."

Step 3: "Implement the ProductRepository following the repository pattern in 
        auth.repository.ts."

Step 4: "Write unit tests for ProductRepository."
```

**Why this works:**
- Each step's output becomes the context for the next step
- Errors are caught early (step 3 won't go wrong if step 1 schema is validated)
- You maintain control of the architecture
- Claude's context window is used efficiently

---

## 5. Output Format Specification

If you don't specify the output format, Claude will choose one. It may be right, but it may not be what you need.

**Specify format explicitly:**

```
Output format:
- TypeScript interface definitions first
- Then the class implementation
- JSDoc on all public methods
- No test code in the implementation file
- Maximum 200 lines
```

**For structured data, use examples:**

```
Return JSON in this exact format:
{
  "issues": [
    {
      "severity": "critical|high|medium|low",
      "type": "security|performance|bug|style",
      "line": 42,
      "description": "Clear description of the issue",
      "fix": "Specific fix suggestion"
    }
  ],
  "summary": "Overall assessment in one sentence"
}
```

**For documents:**
```
Format the output as a Markdown document with:
- H2 sections for each requirement category
- Tables where comparative data exists
- Code blocks for any technical examples
- Bold for key terms
- No introductory preamble — start directly with the content
```

---

## 6. Constraint Specification

Constraints prevent Claude from doing things you don't want. They're as important as the task description.

**Common constraint categories:**

**Scope constraints:**
```
Only modify the auth module. Do not change any other files.
Do not add any new dependencies.
The public API must remain identical.
```

**Quality constraints:**
```
Code must pass TypeScript strict mode with no errors.
All functions must have explicit return types.
Cyclomatic complexity should not exceed 10 per function.
```

**Style constraints:**
```
Use functional programming where possible (avoid classes unless required).
Follow the ESLint rules in .eslintrc.js.
Match the style of the existing codebase, not your personal preference.
```

**Safety constraints:**
```
Do not log or output sensitive data (passwords, tokens, card numbers).
Validate all inputs before use.
Use parameterised queries, never string concatenation for SQL.
```

---

## 7. The Few-Shot Pattern

Showing examples of desired output is one of the most powerful techniques in prompt engineering.

**Without examples:**
```
"Write a commit message for this diff."
```

**With examples (few-shot):**
```
Write a conventional commit message for this diff.

Examples of good commit messages:
- feat(auth): add OAuth2 login with Google provider
- fix(cart): resolve race condition in concurrent add-to-cart operations  
- refactor(payment): extract Stripe service into dedicated module
- test(orders): add edge cases for zero-quantity order validation
- docs(api): update authentication endpoints with Bearer token examples

Now write a message for this diff:
[paste diff]
```

The examples teach Claude: format, vocabulary, scope specificity, and length.

**When to use few-shot:**
- When the output format is precise and unusual
- When tone and style must match existing content
- When the task involves judgment calls (severity rating, code quality scoring)
- When you've had Claude get it wrong before

---

## 8. Chain-of-Thought Prompting

For complex reasoning tasks, instruct Claude to show its work before giving an answer. This dramatically improves accuracy on multi-step problems.

**Basic chain-of-thought:**
```
Before answering, think through this step by step.
[complex question]
```

**Structured chain-of-thought:**
```
Analyse this database query for performance issues.

Use this reasoning process:
1. First, identify what data the query is accessing
2. Identify the WHERE clauses and check if indexes exist
3. Look for N+1 patterns or unnecessary joins
4. Estimate the data volume at each step
5. Then recommend optimisations in priority order

Query:
[paste query]
```

**When chain-of-thought matters most:**
- Complex debugging across multiple files
- Architectural decisions with trade-offs
- Security analysis
- Complex data transformations
- Any task where getting the reasoning wrong leads to a wrong answer

---

## 9. The Verification Request Pattern

Ask Claude to check its own output. This often catches errors before they reach you.

```
[After Claude generates code]

Before finalising, verify:
1. Does the code compile with TypeScript strict mode?
2. Are there any null pointer risks?
3. Does it handle all error cases mentioned in requirements?
4. Is it consistent with the existing code style?
5. Are there any security concerns?

If you find any issues, fix them and show me the corrected version.
```

---

## 10. Anti-Patterns to Avoid

### 10.1 The Vague Request
```
Bad:  "Fix my code"
Good: "Fix the null pointer error on line 47 of auth.service.ts. The error 
       occurs when user.profile is undefined. Do not change the function signature."
```

### 10.2 Asking for Too Much at Once
```
Bad:  "Build the entire backend"
Good: "Implement the POST /users endpoint following the existing GET /users 
       pattern in users.controller.ts"
```

### 10.3 Missing Context
```
Bad:  "Write tests for this function"
Good: "Write Jest unit tests for this function. Use our test helpers from 
       test/utils.ts. Mock external services using jest.mock(). 
       Achieve 100% branch coverage."
```

### 10.4 No Output Specification
```
Bad:  "Explain this code"
Good: "Explain this code. Target audience: junior developer joining the team.
       Format: max 200 words, plain English, no jargon. 
       Include what the code does, why it exists, and any gotchas."
```

### 10.5 Accepting First Draft
For any complex task, iterate. The first output is a draft:
```
First prompt:  "Implement the payment service"
Second prompt: "The processPayment function doesn't handle timeout errors. 
                Also, the logging is too verbose — only log failures."
Third prompt:  "Now write tests for this implementation."
```

---

## 11. Template Library

Save these templates. Adapt for your specific context.

### Template: Feature Implementation
```
You are a [role] working on [project description].
Tech stack: [list]
Code style: [key conventions]

Task: Implement [specific feature].

Requirements:
- [req 1]
- [req 2]

Constraints:
- [constraint 1]
- [constraint 2]

Reference: Follow patterns from [existing file].

Output: [specific files/format expected]
```

### Template: Code Review
```
You are a senior engineer performing a code review.
Focus: [security/performance/correctness/all]

Review criteria (in priority order):
1. Security vulnerabilities
2. Logic errors or edge cases
3. Performance concerns
4. Missing error handling
5. Code quality and maintainability

For each issue found:
- Location (file:line)
- Severity (critical/high/medium/low)
- Description
- Recommended fix

Code to review:
[paste code]
```

### Template: Test Generation
```
You are a QA engineer writing [Jest/Playwright/pytest] tests.
Testing framework: [framework and version]
Test utilities: [list any custom helpers]

Generate comprehensive tests for:
[paste code/spec]

Required coverage:
- Happy path
- Input validation (invalid types, boundary values, null/undefined)
- Error handling
- [domain-specific cases]

Format: [describe/it blocks, AAA pattern, etc.]
Mocking strategy: [how to mock dependencies]
```

### Template: Documentation
```
You are a technical writer creating developer documentation.
Audience: [junior dev/senior dev/external API consumer]

Write documentation for:
[paste code or describe component]

Include:
- Purpose (1-2 sentences)
- Parameters/props with types and descriptions
- Return value
- Example usage (working code)
- Error cases
- Any important caveats or gotchas

Format: [JSDoc/Markdown/OpenAPI]
```

---

## 12. Measuring Prompt Quality

How do you know if your prompt is good? Run the "5-minute test":

1. Give the output to a colleague without showing the prompt
2. Ask: "Would you use this as-is or would it need significant rework?"
3. If significant rework is needed — improve the prompt

Track your prompt performance:
- What percentage of first outputs are usable without major edits?
- How many iterations does a typical task take?
- Which task types consistently produce poor output?

The goal: 80%+ of your first outputs are usable with minor (< 5 minute) edits.

---

## Summary

High-quality prompting is built on six principles:

1. **Assign a role** — prime the model for expert-level response
2. **Provide context** — give Claude everything it needs, nothing it doesn't
3. **Decompose complex tasks** — break into steps, use output as input for the next step
4. **Specify format** — be explicit about how you want the answer structured
5. **Define constraints** — tell Claude what NOT to do, not just what to do
6. **Show examples** — demonstrate the desired output, especially for format and style

In the next article, we'll cover the files that make Claude consistently accurate in your specific project context — `CLAUDE.md`, the `.claude` folder, and how to build a prompt infrastructure that achieves 95%+ accuracy.

---

*Next: Article 8 — The CLAUDE.md System: What Files You Need for 95% Accuracy*
