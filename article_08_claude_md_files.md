# Article 8: The CLAUDE.md System — Complete Configuration for Backend, Frontend, and Mobile

> *CLAUDE.md is the difference between Claude generating generic code and Claude generating code that looks like a senior member of your team wrote it. This article delivers three complete, production-ready configurations — one for backend APIs, one for frontend React applications, and one for mobile development — plus the Skills, Subagents, and Hooks that make each one precision-calibrated for its context.*

---

## Introduction

Claude Code reads your project before it responds to anything. Every session starts with Claude loading your CLAUDE.md, scanning the repository structure, and building a working model of your codebase. The quality of that model determines the quality of every output that follows.

Teams that invest in this system get:
- Code that matches existing patterns without prompting
- Architecture decisions that align with established conventions
- Security practices and testing standards applied automatically
- Consistent outputs across every team member's machine

This article covers three distinct setups:
1. **Backend** — Node.js/TypeScript REST API with Prisma
2. **Frontend** — React/TypeScript SPA
3. **Mobile** — React Native (plus notes for Swift and Kotlin native)

Each setup includes the CLAUDE.md template, `.claude/` folder structure, role-specific Skills, and custom Subagents. Start with the section matching your stack. Section 5 covers the global `~/.claude/CLAUDE.md` and the maturity model that applies to all three.

---

## 1. How Claude Code Loads Context

```
Load order (applied at session start):
1. ~/.claude/CLAUDE.md            → Personal preferences (all projects)
2. /project-root/CLAUDE.md        → Project configuration  
3. /project-root/.claude/         → Skills, Agents, Hooks, context files
4. --add-dir directories          → Additional directories + their .claude/ folders
5. Files explicitly referenced    → Anything you @ mention in your prompt
```

> **Auto-memories (v2.x):** Claude Code now automatically records and recalls project-specific facts it discovers during sessions. These supplement but don't replace your CLAUDE.md — explicit configuration always wins.

---

## 2. Backend — Node.js / TypeScript REST API

### 2.1 Project Structure

```
api-project/
├── CLAUDE.md
├── .claude/
│   ├── settings.json          ← Hooks + MCP config
│   ├── skills/
│   │   ├── new-endpoint.md
│   │   ├── db-migration.md
│   │   ├── code-review.md
│   │   └── debug.md
│   ├── agents/
│   │   ├── security-reviewer.md
│   │   └── test-writer.md
│   └── context/
│       ├── domain-model.md
│       └── api-contracts.md
├── src/
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   ├── validators/
│   ├── utils/
│   ├── config/
│   └── __tests__/
├── prisma/
│   └── schema.prisma
└── package.json
```

### 2.2 Backend CLAUDE.md Template

```markdown
# CLAUDE.md — [Project Name] API

## Project Overview
[2–3 sentences: what it does, who uses it, scale]

Example:
B2B invoice management REST API. Serves 500+ enterprise customers.
Node.js/TypeScript backend. Handles ~2M requests/day at peak.
The API is consumed by a React SPA (../frontend) and mobile apps (../mobile).

## Tech Stack
- **Language:** TypeScript 5.x (strict mode — no any)
- **Runtime:** Node.js 20 LTS
- **Framework:** Express 4.x
- **Database:** PostgreSQL 15 via Prisma ORM 5.x
- **Cache:** Redis 7
- **Auth:** JWT (access 15min / refresh 7 days)
- **Validation:** Zod
- **Testing:** Jest 29 + Supertest
- **Linting:** ESLint + Prettier
- **CI/CD:** GitHub Actions → AWS ECS
- **Logging:** Winston (structured JSON to Datadog)

## Repository Structure
```
src/
├── controllers/     # Route handlers only — no business logic
├── services/        # All business logic
├── repositories/    # All database access
├── middleware/      # Express middleware (auth, error, rate-limit)
├── validators/      # Zod schemas (one per entity)
├── utils/           # Shared pure utilities
├── config/          # Config loaded from env vars
└── __tests__/       # Tests mirroring src/ structure

Key files:
- src/app.ts                     → Express setup + middleware registration
- src/config/index.ts            → All env var access (never access process.env directly)
- src/middleware/error.ts        → Global error handler (reads AppError)
- src/middleware/auth.ts         → JWT verification
- src/utils/errors.ts            → AppError class
- prisma/schema.prisma           → Database schema
```

## Architecture — Non-Negotiable Rules
- **Layer enforcement:** Controller → Service → Repository (strictly)
- **Controllers:** validate input, call one service method, return response. Nothing else.
- **Services:** contain all business logic. Never access the DB directly.
- **Repositories:** the only layer that touches Prisma/SQL. Never contain business logic.
- **Dependency injection:** services receive repositories via constructor parameters
- **Error handling:** always use AppError. Never throw raw Error. Never swallow errors.

```typescript
// CORRECT
throw new AppError('User not found', 404, 'USER_NOT_FOUND');

// WRONG — never do this
throw new Error('User not found');
res.status(404).json({ message: 'User not found' });
```

## TypeScript Conventions
- `strict: true` — no exceptions
- No `any`, no `@ts-ignore` (use `unknown` + type narrowing)
- Explicit return types on all exported functions
- `interface` for object shapes, `type` for unions and aliases
- `readonly` on properties that should not be mutated
- No `null` — use `undefined` (consistent null-safety checks)

## Naming
- Files: `kebab-case.ts`
- Classes / Interfaces: `PascalCase`
- Functions / variables: `camelCase`
- Constants / env vars: `SCREAMING_SNAKE_CASE`
- DB tables / columns: `snake_case` (Prisma maps these automatically)

## Functions
- Max 30 lines — extract if longer
- Max 3 parameters — use an options object for more
- Single responsibility — one function does one thing
- Pure functions preferred — side effects isolated to services/repositories

## Async Rules
- Always `async/await`, never raw `.then()/.catch()` chains
- Never `await` inside a loop — use `Promise.all()` or `Promise.allSettled()`
- Always handle rejections — either catch or let propagate as `AppError`

## Logging
```typescript
// CORRECT — structured logger with context
import { logger } from '../utils/logger';
logger.info('Payment processed', { orderId, amount, currency });
logger.error('Stripe webhook failed', { error: err.message, eventId });

// NEVER log: passwords, tokens, card numbers, full request bodies, PII
// NEVER use console.log in production code
```

## Validation
- ALL inputs validated with Zod at the controller layer before reaching the service
- Schemas live in src/validators/[entity].validator.ts
- Export both the schema and the inferred TypeScript type

```typescript
export const CreateOrderSchema = z.object({
  customerId: z.string().uuid(),
  lineItems: z.array(LineItemSchema).min(1),
  currency: z.enum(['GBP', 'USD', 'EUR']),
});
export type CreateOrderInput = z.infer<typeof CreateOrderSchema>;
```

## Database Conventions
- All migrations via `prisma migrate dev` — never edit existing migrations
- Every model: `id` (UUID), `createdAt`, `updatedAt`
- Soft deletes: `deletedAt` timestamp — never hard delete user/financial data
- Monetary values: **integer pence/cents** — never float
- Foreign keys: always add explicit Prisma `@relation` and an index
- UUIDs for all PKs — never auto-increment integers on user-facing models

## Security Rules (CRITICAL — Always Apply)
- Zod validation on ALL inputs — no exceptions
- Parameterised queries only — Prisma handles this; never build raw SQL strings
- Rate limiting on all auth endpoints (see `src/middleware/rate-limit.ts`)
- RBAC middleware before every protected endpoint
- Never expose internal PKs — use UUIDs on all API responses
- No sensitive data in logs, error messages, or API responses
- CORS: whitelist only — never `origin: '*'` in production config

## Testing Standards
- Unit tests: every service method and utility
- Integration tests: every controller endpoint (use Supertest + real test DB)
- File location: `src/__tests__/` mirroring `src/` structure
- Coverage: 85% line, 80% branch (enforced in CI — `jest --coverage`)
- Factories: `src/__tests__/factories/` — use these, never inline test data

```typescript
// Test pattern (AAA):
describe('OrderService', () => {
  describe('createOrder', () => {
    it('should throw USER_NOT_FOUND when customer does not exist', async () => {
      // Arrange
      mockCustomerRepo.findById.mockResolvedValue(null);
      // Act + Assert
      await expect(orderService.createOrder(input)).rejects.toThrow(AppError);
    });
  });
});
```

## Endpoint Creation Pattern (Follow This Exactly)
1. Add Zod schema: `src/validators/[entity].validator.ts`
2. Add repository method: `src/repositories/[entity].repository.ts`
3. Add service method: `src/services/[entity].service.ts`
4. Add controller method: `src/controllers/[entity].controller.ts`
5. Register route: `src/routes/[entity].routes.ts` → `src/app.ts`
6. Add integration test: `src/__tests__/controllers/[entity].test.ts`

Reference implementation: `src/controllers/user.controller.ts`

## What NOT to Do
- ❌ No `any` type — use `unknown` and narrow
- ❌ No business logic in controllers
- ❌ No database access in services
- ❌ No `console.log` — use logger
- ❌ No raw `Error` throws — use `AppError`
- ❌ No hardcoded values — use `src/config/index.ts`
- ❌ No direct `process.env` access outside `src/config/`
- ❌ No `await` inside loops
- ❌ No float for monetary values
- ❌ No skipping Zod validation
- ❌ No modifying existing migrations
```

### 2.3 Backend Skills

```markdown
<!-- .claude/skills/new-endpoint.md -->
---
name: new-endpoint
description: Scaffold a new REST endpoint following project architecture
---

Scaffold a new endpoint for: $0

Steps (in this exact order):
1. Read src/controllers/user.controller.ts as the reference pattern
2. Read prisma/schema.prisma to understand the data model
3. Create src/validators/$0.validator.ts (Zod schema + inferred type)
4. Create src/repositories/$0.repository.ts (Prisma queries only)
5. Create src/services/$0.service.ts (business logic, uses repository)
6. Create src/controllers/$0.controller.ts (validates, delegates, responds)
7. Create src/routes/$0.routes.ts (Express router)
8. Register in src/app.ts
9. Run: npx tsc --noEmit (fix any type errors before continuing)
10. Create src/__tests__/controllers/$0.test.ts with test scaffolding

After each file creation: verify TypeScript compiles.
```

```markdown
<!-- .claude/skills/db-migration.md -->
---
name: db-migration
description: Create a Prisma migration for a schema change
tools: Read, Bash
---

Create a database migration for: $0

Steps:
1. Read prisma/schema.prisma (understand current schema)
2. Determine the exact change needed
3. Update prisma/schema.prisma with the change
4. Run: npx prisma migrate dev --name $0 --create-only
5. Show the generated SQL for review
6. Check if any existing repositories need updating for this schema change
7. Flag any data migrations needed (never lose data on migration)

Rules:
- Never modify existing migrations
- Add indexes for all new foreign keys
- Soft-delete columns only (deletedAt) — never drop user data columns
- Monetary fields: Int (pence/cents), never Float
```

```markdown
<!-- .claude/skills/debug.md -->
---
name: debug
description: Systematic root cause analysis of errors and failing tests
tools: Read, Bash, Grep
---

Debug the following issue: $0

Process:
1. Identify what error is occurring (read error message + stack trace carefully)
2. Trace through the code path: controller → service → repository
3. Read the relevant source files
4. Check recent git changes: `git log --oneline -20`
5. Check if related tests exist and if they pass
6. Identify the root cause (be specific — file:line)
7. Provide the minimal fix (don't over-engineer)
8. Explain why the bug occurred so it can be avoided in future

Output format:
- Root cause: [specific explanation]
- Location: [file:line]
- Fix: [exact code change]
- Prevention: [one sentence]
```

### 2.4 Backend Subagents

```markdown
<!-- .claude/agents/security-reviewer.md -->
---
name: security-reviewer
description: Adversarial security review of backend code changes
model: claude-opus-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a senior security engineer performing an adversarial review.
Assume the attacker has read all source code.

Review for:
- SQL/NoSQL injection (check Prisma raw query usage)
- Authentication bypass (JWT validation, session handling)
- Authorisation flaws (IDOR, missing RBAC checks)
- Sensitive data exposure (logs, error messages, API responses)
- Rate limiting gaps on auth/sensitive endpoints
- CORS misconfiguration
- Mass assignment vulnerabilities
- Missing Zod validation on any input path

For every finding:
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- File:Line
- Attack scenario (concrete exploit description)
- Exact remediation code
```

```markdown
<!-- .claude/agents/test-writer.md -->
---
name: test-writer
description: Writes comprehensive Jest + Supertest tests for backend code
model: claude-sonnet-4-6
tools: Read, Glob, Write
---

Write comprehensive tests for the specified backend code.

For each service method:
- Happy path with valid inputs
- Validation failures (null, wrong type, boundary values)
- Repository error propagation
- Business rule violations

For each controller endpoint (Supertest):
- 200/201 success cases
- 400 validation errors
- 401/403 auth/permission errors
- 404 not found
- 500 unexpected errors

Use:
- AAA pattern (Arrange, Act, Assert)
- Factories from src/__tests__/factories/
- jest.mock() for external services
- Real test DB for integration tests (see jest.config.ts)

Coverage target: 100% branches on business logic.
```

---

## 3. Frontend — React / TypeScript SPA

### 3.1 Project Structure

```
frontend-project/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
│   ├── skills/
│   │   ├── new-component.md
│   │   ├── new-page.md
│   │   ├── new-hook.md
│   │   └── accessibility-audit.md
│   ├── agents/
│   │   ├── a11y-reviewer.md
│   │   └── perf-reviewer.md
│   └── context/
│       ├── design-system.md
│       └── state-management.md
├── src/
│   ├── components/
│   │   ├── ui/              # Primitives (Button, Input, Modal)
│   │   └── features/        # Feature-specific components
│   ├── pages/               # Route-level components
│   ├── hooks/               # Custom React hooks
│   ├── stores/              # Zustand stores
│   ├── api/                 # API client + query hooks
│   ├── utils/               # Pure utility functions
│   ├── types/               # Shared TypeScript types
│   └── __tests__/
└── package.json
```

### 3.2 Frontend CLAUDE.md Template

```markdown
# CLAUDE.md — [Project Name] Frontend

## Project Overview
[2–3 sentences: what the app does, who uses it, tech context]

Example:
React SPA for the invoice management platform. Used by 500+ enterprise finance teams.
Consumes the REST API at ../api. Deployed as a static site on Cloudfront.
Design system based on shadcn/ui with custom theming.

## Tech Stack
- **Language:** TypeScript 5.x (strict mode)
- **Framework:** React 18 (functional components only — no class components)
- **Build:** Vite 5
- **Routing:** React Router v6 (file-based route structure)
- **State:** Zustand for global state, React Query (TanStack) for server state
- **Styling:** Tailwind CSS + shadcn/ui component library
- **Forms:** React Hook Form + Zod validation
- **Testing:** Vitest + React Testing Library + Playwright (E2E)
- **Linting:** ESLint + Prettier
- **Icons:** Lucide React (never install other icon libraries)
- **Dates:** date-fns (never Moment.js)

## Repository Structure
```
src/
├── components/
│   ├── ui/          # shadcn/ui primitives (Button, Input, Dialog, etc.)
│   └── features/    # Feature-specific components (InvoiceList, PaymentForm)
├── pages/           # Route components (one per route)
├── hooks/           # Custom hooks (useAuth, useInvoices, useDebounce)
├── stores/          # Zustand stores (one per domain)
├── api/
│   ├── client.ts    # Axios instance with interceptors
│   └── hooks/       # React Query hooks (useGetInvoices, useCreateOrder)
├── utils/           # Pure utility functions (formatting, validation)
├── types/           # Shared TypeScript types (not component-specific)
└── __tests__/       # Vitest unit tests + Playwright E2E

Key files:
- src/main.tsx               → App entry point
- src/router.tsx             → Route definitions
- src/stores/auth.store.ts   → Auth state (JWT, user session)
- src/api/client.ts          → Axios with auth interceptors
- tailwind.config.ts         → Theme tokens (use these, not hardcoded values)
```

## Component Architecture Rules
- **One component per file** — no multiple exports of components from same file
- **Feature components** in `src/components/features/[feature-name]/`
- **No business logic in components** — extract to custom hooks
- **No direct API calls in components** — use React Query hooks from `src/api/hooks/`
- **No Zustand store access in leaf components** — only in page-level or provider components
- **Props interface named after component** — `ButtonProps`, `InvoiceCardProps`
- **Export default for the component, named export for the props type**

## Component Patterns

### Standard component structure:
```typescript
// src/components/features/invoices/InvoiceCard.tsx
import { type FC } from 'react';
import { formatCurrency } from '@/utils/format';
import { Badge } from '@/components/ui/badge';

export interface InvoiceCardProps {
  invoiceId: string;
  amount: number;
  status: InvoiceStatus;
  customerName: string;
  dueDate: Date;
  onSelect?: (id: string) => void;
}

const InvoiceCard: FC<InvoiceCardProps> = ({
  invoiceId, amount, status, customerName, dueDate, onSelect,
}) => {
  return (
    <div
      className="rounded-lg border border-border p-4 hover:bg-muted/50 cursor-pointer"
      onClick={() => onSelect?.(invoiceId)}
    >
      <div className="flex items-center justify-between">
        <span className="font-medium text-sm">{customerName}</span>
        <Badge variant={getStatusVariant(status)}>{status}</Badge>
      </div>
      <p className="text-2xl font-bold mt-1">{formatCurrency(amount)}</p>
    </div>
  );
};

export default InvoiceCard;
```

### Custom hook pattern:
```typescript
// src/hooks/useInvoiceFilter.ts
export function useInvoiceFilter(invoices: Invoice[]) {
  const [filter, setFilter] = useState<InvoiceFilter>({ status: 'all' });
  
  const filteredInvoices = useMemo(
    () => applyFilter(invoices, filter),
    [invoices, filter]
  );

  return { filteredInvoices, filter, setFilter };
}
// Logic lives here. Component just calls useInvoiceFilter(data).
```

### React Query hook pattern:
```typescript
// src/api/hooks/useInvoices.ts
export function useGetInvoices(params: GetInvoicesParams) {
  return useQuery({
    queryKey: ['invoices', params],
    queryFn: () => invoiceApi.getAll(params),
    staleTime: 30_000,
  });
}

export function useCreateInvoice() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: invoiceApi.create,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['invoices'] }),
  });
}
```

## TypeScript Conventions
- `strict: true` — no `any`, no `@ts-ignore`
- Explicit return types on hooks and utilities
- `interface` for props and object shapes, `type` for unions
- Never use React.FC — use `const Component: FC<Props> = ...` or inline type
- Path alias `@/` maps to `src/` — always use this, never relative `../../`

## Styling Rules
- **Tailwind only** — no inline `style={{}}` except for dynamic values impossible in Tailwind
- **Design tokens only** — use `text-foreground`, `bg-card`, `border-border` etc. — never `text-gray-700`
- **shadcn/ui for all UI primitives** — never write your own Button, Input, Dialog
- **No magic numbers** — use spacing scale (`p-4`, `gap-6`) not `p-[17px]`
- **Responsive first** — mobile breakpoints before desktop (`sm:`, `md:`, `lg:`)
- **Dark mode** — all components must work in both light and dark mode

## State Management Rules
- **Server state** (API data): React Query — always. Never Zustand for API data.
- **Global client state** (auth, UI preferences, cart): Zustand
- **Local UI state** (open/closed, form values): useState / useReducer
- **Never** use Context API for state that changes frequently (re-render cost)
- **One Zustand store per domain** — auth.store.ts, ui.store.ts, cart.store.ts

```typescript
// Zustand store pattern:
interface AuthStore {
  user: User | null;
  token: string | null;
  setUser: (user: User, token: string) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  token: null,
  setUser: (user, token) => set({ user, token }),
  clearAuth: () => set({ user: null, token: null }),
}));
```

## Form Handling
- React Hook Form + Zod for ALL forms — no exceptions
- Define the Zod schema first, infer the TypeScript type from it
- Use `zodResolver` from `@hookform/resolvers/zod`

```typescript
const schema = z.object({
  email: z.string().email('Invalid email'),
  amount: z.number().positive('Must be positive'),
});
type FormValues = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<FormValues>({
  resolver: zodResolver(schema),
});
```

## Error Handling in Components
- Wrap page-level components in React Error Boundaries
- API errors: display via `useToast()` for transient errors, inline for form errors
- Loading states: always show skeleton components, never empty content

## Accessibility (A11Y) — Non-Negotiable
- Semantic HTML — `<button>` not `<div onClick>`, `<nav>` not `<div className="nav">`
- All interactive elements: keyboard accessible + focus-visible styles
- Images: `alt` text always (empty string `alt=""` for decorative images)
- Forms: `<label>` associated with every input via `htmlFor` + `id`
- Colour contrast: minimum 4.5:1 for normal text (use design tokens — they're calibrated)
- ARIA: only when HTML semantics are insufficient

## Performance Rules
- Lazy load all route-level components (`React.lazy` + `Suspense`)
- `useMemo` only for expensive computations (not for simple object creation)
- `useCallback` only when passing callbacks to memoised child components
- Avoid anonymous functions in JSX (`onClick={() => fn(id)}` is fine for simple cases)
- Images: use `loading="lazy"` and explicit `width`/`height`
- No unused dependencies in useEffect dependency arrays

## Testing Standards
- **Unit tests (Vitest + RTL):** all hooks, utilities, and complex components
- **Integration tests:** page-level components with mocked API
- **E2E (Playwright):** critical user journeys (login → create invoice → pay)
- Test behaviour, not implementation — query by role/label, not test IDs
- Mock at the network layer (MSW), not at the component level

```typescript
// Testing Library pattern:
it('should show error message when email is invalid', async () => {
  render(<LoginForm />);
  await userEvent.type(screen.getByLabelText('Email'), 'notanemail');
  await userEvent.click(screen.getByRole('button', { name: /sign in/i }));
  expect(screen.getByText('Invalid email')).toBeInTheDocument();
});
```

## What NOT to Do
- ❌ No class components
- ❌ No business logic in components — extract to hooks
- ❌ No direct API calls in components — use React Query hooks
- ❌ No inline styles except dynamic values
- ❌ No hardcoded colour values — use Tailwind tokens
- ❌ No `any` type
- ❌ No `useEffect` for data fetching — use React Query
- ❌ No `document.getElementById` or direct DOM manipulation
- ❌ No installing icon libraries other than Lucide
- ❌ No Moment.js — use date-fns
- ❌ No non-accessible interactive elements (`<div onClick>`)
```

### 3.3 Frontend Skills

```markdown
<!-- .claude/skills/new-component.md -->
---
name: new-component
description: Scaffold a new React component following project conventions
---

Create a new component: $0

Steps:
1. Read src/components/features/invoices/InvoiceCard.tsx as the reference pattern
2. Determine: is this a ui/ primitive or a features/ component?
3. Create the component file with:
   - Named Props interface (exported)
   - FC typing
   - Tailwind classes using design tokens only
   - Accessibility attributes
   - Proper event handler typing
4. Create the test file in src/__tests__/components/
5. Export from the feature's index.ts barrel file

Do NOT create a new shadcn/ui component if one already exists.
Run: npx tsc --noEmit after creating both files.
```

```markdown
<!-- .claude/skills/new-page.md -->
---
name: new-page
description: Scaffold a new route page with layout, loading, and error states
---

Create a new page: $0

Steps:
1. Read src/pages/InvoicesPage.tsx as the reference pattern
2. Create src/pages/$0Page.tsx with:
   - React Query hook for data fetching
   - Skeleton loading state (use existing Skeleton component)
   - Error boundary or error state
   - Empty state component
   - Proper page title (update document.title)
3. Register route in src/router.tsx (lazy loaded with React.lazy)
4. Add to navigation if it should appear in the sidebar
5. Create Playwright E2E test stub in e2e/$0.spec.ts
```

```markdown
<!-- .claude/skills/accessibility-audit.md -->
---
name: accessibility-audit
description: Full accessibility audit of a component or page
tools: Read, Glob
---

Perform a WCAG 2.1 AA accessibility audit on: $0

Check every element for:
1. **Semantic HTML** — correct element for role (button, nav, main, header, etc.)
2. **Keyboard navigation** — can every interactive element be reached and operated by keyboard?
3. **Focus management** — visible focus ring? Focus trap in modals? Focus restoration on close?
4. **ARIA** — any redundant or incorrect ARIA? Missing roles on custom widgets?
5. **Labels** — every form input has an associated label?
6. **Colour contrast** — flag any hardcoded colours that might fail 4.5:1 ratio
7. **Images** — meaningful alt text? Decorative images have empty alt=""?
8. **Motion** — any animation that could cause vestibular issues without prefers-reduced-motion?

For each issue:
- WCAG criterion (e.g., 1.3.1 Info and Relationships)
- Severity: CRITICAL / HIGH / MEDIUM / LOW
- File:Line
- Fix (exact JSX change)
```

### 3.4 Frontend Subagents

```markdown
<!-- .claude/agents/a11y-reviewer.md -->
---
name: a11y-reviewer
description: WCAG 2.1 AA accessibility review of React components
model: claude-sonnet-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are an accessibility specialist reviewing React components for WCAG 2.1 AA compliance.

Be thorough. Many a11y issues are subtle:
- Missing focus management in modals and drawers
- Incorrectly used ARIA roles
- Dynamic content changes not announced to screen readers
- Insufficient colour contrast (flag hardcoded values)
- Missing skip navigation links on page-level components
- Form field error messages not associated with inputs

For every finding:
- WCAG criterion
- Severity
- Component + line number
- Concrete JSX fix
```

```markdown
<!-- .claude/agents/perf-reviewer.md -->
---
name: perf-reviewer
description: React performance review — re-renders, bundle size, and Core Web Vitals
model: claude-sonnet-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a React performance engineer reviewing for common performance issues.

Review for:
- Unnecessary re-renders (missing memoisation where it matters)
- Objects/arrays created on every render passed as props to memoised children
- Missing React.lazy on route-level components
- Missing Suspense boundaries
- Images without explicit width/height (CLS issues)
- Waterfall data fetching (sequential queries that could be parallel)
- Large bundle imports (importing full library for one function)
- useEffect dependencies that cause excessive fetching
- Missing staleTime on React Query queries

For every finding:
- Component + line
- Performance impact (re-renders / bundle size / CLS / LCP)
- Fix with example code
```

### 3.5 Frontend Context Files

```markdown
<!-- .claude/context/design-system.md -->
# Design System Reference

## Colour Tokens (always use these — never hardcode hex values)
- `bg-background` / `text-foreground` — page background / default text
- `bg-card` / `text-card-foreground` — card surfaces
- `bg-primary` / `text-primary-foreground` — primary action colour
- `bg-destructive` / `text-destructive-foreground` — errors, danger
- `bg-muted` / `text-muted-foreground` — secondary text, disabled states
- `border-border` — default border colour
- `ring-ring` — focus ring colour

## Typography Scale
- `text-xs` (12px), `text-sm` (14px), `text-base` (16px)
- `text-lg` (18px), `text-xl` (20px), `text-2xl` (24px)
- `font-normal` (400), `font-medium` (500), `font-semibold` (600), `font-bold` (700)
- Headings: always `font-semibold` or `font-bold`

## Spacing Scale
- 4px unit system: `p-1`=4px, `p-2`=8px, `p-4`=16px, `p-6`=24px, `p-8`=32px
- Consistent gaps: `gap-2` between tight items, `gap-4` standard, `gap-6` sections
- Page padding: `px-6 py-8` (mobile), `px-8 py-10` (md+)

## shadcn/ui Component Inventory
Available components (do NOT recreate these):
Button, Input, Label, Textarea, Select, Checkbox, RadioGroup, Switch,
Dialog, AlertDialog, Sheet (drawer), Popover, Tooltip, DropdownMenu,
Card, Badge, Avatar, Skeleton, Progress, Separator, Tabs, Accordion,
Table, DataTable, Form, Toast / Sonner, Calendar, DatePicker

## Status Badge Variants
- `variant="default"` — neutral/pending
- `variant="secondary"` — informational
- `variant="destructive"` — error/danger
- Custom: `variant="success"` and `variant="warning"` added in components/ui/badge.tsx
```

---

## 4. Mobile — React Native / Expo

### 4.1 Project Structure

```
mobile-project/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
│   ├── skills/
│   │   ├── new-screen.md
│   │   ├── new-component.md
│   │   └── native-module.md
│   ├── agents/
│   │   ├── performance-reviewer.md
│   │   └── platform-reviewer.md
│   └── context/
│       ├── navigation-structure.md
│       └── platform-differences.md
├── src/
│   ├── screens/             # Route-level screen components
│   ├── components/
│   │   ├── ui/              # Primitive components
│   │   └── features/        # Feature-specific components
│   ├── navigation/          # React Navigation config
│   ├── hooks/               # Custom hooks
│   ├── stores/              # Zustand stores
│   ├── api/                 # API client + React Query hooks
│   ├── utils/               # Pure utilities
│   └── __tests__/
├── app.json                 # Expo config
└── eas.json                 # EAS Build config
```

### 4.2 Mobile CLAUDE.md Template

```markdown
# CLAUDE.md — [Project Name] Mobile App

## Project Overview
[2–3 sentences: what the app does, platforms, tech context]

Example:
React Native / Expo app for the invoice management platform.
Available on iOS 15+ and Android 8+. Shares API with the web frontend.
Uses Expo Router for navigation and EAS Build for CI/CD.

## Tech Stack
- **Framework:** React Native 0.76+ with Expo SDK 52
- **Language:** TypeScript 5.x (strict mode)
- **Navigation:** Expo Router (file-based, similar to Next.js App Router)
- **State:** Zustand (global), React Query / TanStack Query (server state)
- **Styling:** StyleSheet API + NativeWind (Tailwind for React Native)
- **Forms:** React Hook Form + Zod
- **Storage:** Expo SecureStore (auth tokens), AsyncStorage (non-sensitive)
- **Notifications:** Expo Notifications
- **Testing:** Jest + React Native Testing Library + Detox (E2E)
- **CI/CD:** GitHub Actions + EAS Build + EAS Submit
- **Analytics:** Expo Updates + custom event tracking
- **Crash Reporting:** Sentry React Native

## Repository Structure
```
src/
├── screens/          # File-based routes (Expo Router)
│   ├── (auth)/       # Auth group (login, register, forgot-password)
│   ├── (tabs)/       # Bottom tab routes (home, invoices, settings)
│   └── _layout.tsx   # Root layout
├── components/
│   ├── ui/           # Primitives: Button, Input, Card, Modal, etc.
│   └── features/     # Feature components: InvoiceListItem, PaymentForm
├── navigation/       # Navigation types + deep link config
├── hooks/            # useAuth, useInvoices, useOffline, useHaptics
├── stores/           # auth.store.ts, ui.store.ts
├── api/              # API client + React Query hooks
├── utils/            # Pure utilities (format, validation)
└── __tests__/

Key files:
- app/_layout.tsx              → Root layout (providers, auth check)
- app/(tabs)/_layout.tsx       → Bottom tab navigator config
- src/api/client.ts            → Axios with auth + token refresh
- src/stores/auth.store.ts     → Auth state (persisted to SecureStore)
- src/hooks/useOffline.ts      → Network state + offline queue
- eas.json                     → Build profiles (development/preview/production)
```

## Platform Rules (Critical)
- **Always test both iOS and Android** — never assume cross-platform parity
- **Never use `Platform.OS === 'ios'`** for layout — use proper responsive design
- **Platform-specific files** for genuine differences: `Component.ios.tsx` / `Component.android.tsx`
- **Status bar:** explicitly configure `<StatusBar>` on every screen
- **Safe areas:** always wrap screens in `<SafeAreaView>` or use `useSafeAreaInsets()`
- **Keyboard:** use `KeyboardAvoidingView` on all screens with form inputs
- **Back button:** always implement Android back button handling for custom navigators

## Component Rules
- Functional components only — no class components
- Props typed with explicit interfaces
- No business logic in components — extract to hooks
- No API calls in components — use React Query hooks
- All custom components in src/components/ui/ or src/components/features/

## Styling Rules
- NativeWind (Tailwind) for all styling — no inline StyleSheet except for dynamic values
- Use design tokens from tailwind.config.js — never hardcode colours
- **Platform-specific sizing:** `Platform.select({ ios: 16, android: 14 })` only for font sizes that genuinely differ
- Support both light and dark mode using NativeWind's `dark:` variants
- Font: `font-sans` (maps to SF Pro on iOS, Roboto on Android via Expo Font)
- **Touch targets:** minimum 44×44pt on all interactive elements
- **No fixed pixel values for layout** — use flex, percentage, or dimension hooks

## Navigation Rules (Expo Router)
- File-based routing — screen file = route
- Groups: `(auth)` for unauthenticated, `(tabs)` for authenticated tab screens
- Protected routes via middleware in `src/navigation/auth-guard.ts`
- Deep links configured in `app.json` under `scheme` and `intentFilters`
- Never use `useNavigation()` directly — use typed navigation hooks

```typescript
// Typed navigation:
import { useRouter } from 'expo-router';
const router = useRouter();
router.push('/invoices/123');
router.replace('/(auth)/login');
```

## State Management
- **Server state:** React Query — same pattern as web
- **Auth state:** Zustand + persisted to Expo SecureStore (never AsyncStorage for tokens)
- **Local UI state:** useState / useReducer
- **Offline support:** React Query with persistence adapter + optimistic updates

## Auth Token Handling
```typescript
// CORRECT — use SecureStore for tokens
import * as SecureStore from 'expo-secure-store';
await SecureStore.setItemAsync('access_token', token);
const token = await SecureStore.getItemAsync('access_token');

// NEVER — do not store auth tokens in AsyncStorage (insecure)
await AsyncStorage.setItem('token', token); // ❌
```

## Performance Rules
- Lazy load heavy screens with `React.lazy` equivalent (Expo Router handles this)
- `FlatList` for all lists > 5 items — never `ScrollView + map()`
- `keyExtractor` on every FlatList
- `getItemLayout` on FlatLists with fixed-height items (avoids layout measurement)
- `removeClippedSubviews={true}` on large FlatLists
- `useCallback` for `renderItem` and `keyExtractor` on FlatLists
- Avoid re-renders on scroll: memoize list items with `React.memo`
- `InteractionManager.runAfterInteractions` for heavy operations after navigation

```typescript
// FlatList pattern:
const renderItem = useCallback<ListRenderItem<Invoice>>(
  ({ item }) => <InvoiceListItem invoice={item} onPress={handlePress} />,
  [handlePress]
);

<FlatList
  data={invoices}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  removeClippedSubviews
  maxToRenderPerBatch={10}
  windowSize={10}
/>
```

## Offline Support
- All READ operations: React Query with network-mode persistence
- Write operations: optimistic updates with rollback on error
- Queue offline mutations (see src/hooks/useOfflineMutation.ts)
- Show offline banner using useNetInfo() from @react-native-netinfo

## Haptics
- Every primary action: `Haptics.impactAsync(ImpactFeedbackStyle.Medium)`
- Success/completion: `Haptics.notificationAsync(NotificationFeedbackType.Success)`
- Error: `Haptics.notificationAsync(NotificationFeedbackType.Error)`
- Tab press: `Haptics.impactAsync(ImpactFeedbackStyle.Light)`

## Push Notifications
- Always request permissions on first meaningful moment (not on app launch)
- Token registration in src/hooks/usePushNotifications.ts
- Deep link from notification → screen must be handled in app/_layout.tsx

## Testing Standards
- Unit tests: all hooks, utilities, complex components (Jest + RNTL)
- Integration tests: screen-level with mocked API (MSW)
- E2E: Detox for critical flows (login, create invoice, payment)
- Test IDs: use `testID` prop for Detox selectors (naming: `[screen]-[element]`)

```typescript
// RNTL test pattern:
it('should show invoice total after items added', async () => {
  render(<InvoiceForm />);
  fireEvent.changeText(screen.getByTestId('invoice-form-amount'), '100');
  expect(screen.getByText('£100.00')).toBeTruthy();
});
```

## Build and Release
- Development build: `eas build --profile development`
- Preview (internal testing): `eas build --profile preview`
- Production: `eas build --profile production`
- OTA updates: `eas update --branch production` for JS-only changes
- Native changes require a full EAS Build — document them in the PR

## What NOT to Do
- ❌ No class components
- ❌ No business logic in components
- ❌ No direct `fetch()` — use the API client in src/api/client.ts
- ❌ No auth tokens in AsyncStorage — use SecureStore
- ❌ No `ScrollView` for lists > 5 items — use FlatList
- ❌ No `any` type
- ❌ No hardcoded colours — use NativeWind tokens
- ❌ No missing safe area handling
- ❌ No touch targets smaller than 44pt
- ❌ No `Platform.OS` branches in styling — use platform-specific files
- ❌ No heavy computation on the main thread — use `InteractionManager` or move to a worker
```

### 4.3 Mobile Skills

```markdown
<!-- .claude/skills/new-screen.md -->
---
name: new-screen
description: Scaffold a new Expo Router screen with full layout, loading, and error states
---

Create a new screen: $0

Steps:
1. Read src/screens/(tabs)/InvoicesScreen.tsx as the reference
2. Create app/(tabs)/$0.tsx with:
   - SafeAreaView wrapper
   - StatusBar configuration
   - React Query hook for data
   - FlatList (if showing a list) or ScrollView with KeyboardAvoidingView (if form)
   - Skeleton loading state (use existing ScreenSkeleton component)
   - Error state with retry button
   - Empty state illustration
3. Add screen to the tab navigator if it appears in bottom tabs
4. Create the screen's test file in src/__tests__/screens/
5. Add any required deep link configuration to app.json

Platform checklist for each screen:
- [ ] SafeAreaView or useSafeAreaInsets used
- [ ] StatusBar configured
- [ ] Back button handled (Android)
- [ ] Keyboard handling if form inputs present
- [ ] Both light and dark mode tested
```

```markdown
<!-- .claude/skills/new-component.md -->
---
name: new-component
description: Create a new React Native component following project conventions
---

Create a new component: $0

Steps:
1. Determine: ui/ primitive or features/ component?
2. Create with:
   - Typed Props interface
   - NativeWind styling (no StyleSheet except for dynamic values)
   - Minimum 44pt touch target on interactive elements
   - accessibilityLabel on all interactive elements
   - accessibilityRole where semantic meaning isn't implicit
   - Haptic feedback on press actions
3. Create test file in src/__tests__/components/
4. Export from the feature barrel file

Run npx tsc --noEmit after creation.
```

```markdown
<!-- .claude/skills/native-module.md -->
---
name: native-module
description: Scaffold or integrate an Expo native module
tools: Read, Bash
---

Set up native module integration for: $0

Steps:
1. Check if an Expo SDK module exists: search docs.expo.dev for "$0"
2. If Expo module exists: install via `npx expo install expo-$0`
3. If no Expo module: check if React Native Community version exists
4. Document whether this requires a custom development build (prebuild)
5. Update eas.json if new permissions needed (iOS: Info.plist, Android: manifest)
6. Create the hook in src/hooks/use$0.ts wrapping the native API
7. Add permission request logic with explanation of why permission is needed
8. Write tests for the hook with proper mocking

Note: any native module requiring prebuild = new EAS Build required. Flag this clearly.
```

### 4.4 Mobile Subagents

```markdown
<!-- .claude/agents/performance-reviewer.md -->
---
name: performance-reviewer
description: React Native performance review — re-renders, FlatList, JS thread
model: claude-sonnet-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a React Native performance engineer.

Review for:
- ScrollView used for lists that should be FlatList
- Missing keyExtractor, removeClippedSubviews, getItemLayout
- renderItem functions not wrapped in useCallback (causes re-renders on scroll)
- List items not memoised with React.memo
- Heavy computation in render paths (should use useMemo or move off main thread)
- Missing InteractionManager.runAfterInteractions for post-navigation tasks
- Unnecessary state updates causing cascade re-renders
- useEffect with broad dependency arrays causing excessive side effects
- Images without explicit dimensions (causes layout thrash)
- Async operations blocking the JS thread

For each issue: component + line, performance impact, exact fix.
```

```markdown
<!-- .claude/agents/platform-reviewer.md -->
---
name: platform-reviewer
description: iOS/Android platform compatibility review
model: claude-sonnet-4-6
tools: Read, Glob, Grep
permissionMode: read-only
---

You are a cross-platform mobile engineer reviewing for platform compatibility issues.

Review for:
- Missing SafeAreaView or useSafeAreaInsets usage
- Hard-coded heights that don't account for notches or navigation bars
- Missing Android back button handling in custom navigators
- Platform-specific behaviours handled with Platform.OS in JSX (use platform files)
- Font size differences not accounted for (iOS vs Android rendering)
- Missing accessibilityLabel on elements that need them on both platforms
- Keyboard behaviour differences not handled (KeyboardAvoidingView behaviour prop)
- SecureStore vs Keychain differences for sensitive data
- Push notification permission flows differing between platforms

For each issue: file + line, platform affected, exact fix with platform notes.
```

---

## 5. Global `~/.claude/CLAUDE.md` — Personal Preferences

This file applies to every project you work on. Set it once, update as your preferences evolve.

```markdown
# Global Claude Preferences

## My Role and Experience
Senior engineer with [X years] experience in [your main stack].
Skip basic explanations. I prefer direct, concrete responses.

## Communication Style
- Direct and opinionated — don't hedge unnecessarily
- Flag trade-offs and risks before implementing
- If my approach has a problem, say so before starting
- Ask clarifying questions BEFORE writing code, not after
- When you see a better approach, say so once, then do what I asked

## Code Preferences
- TypeScript always where supported
- Functional patterns over imperative where it reads clearly
- Explicit over clever — readability beats brevity
- Tests always — no code without tests
- Never explain common language features
- No lengthy summaries of changes — just show the output

## Output Format
- Code first, explanation after (or skip explanation if straightforward)
- No "Here's what I did:" preambles
- No "I hope this helps!" closings
- Flag important caveats in one sentence, not three paragraphs

## Security
Always apply OWASP Top 10 awareness without being asked.
Flag security implications as part of any review.
```

---

## 6. `.claude/context/` — Deeper Domain Knowledge

### `domain-model.md` (Backend + Mobile examples)

```markdown
# Domain Model — Invoice Management

## Core Entities

### Invoice
States: DRAFT → SENT → VIEWED → PAID | OVERDUE | DISPUTED | CANCELLED
- DRAFT: editable. SENT: immutable, triggers email.
- OVERDUE: set by daily cron (dueDate < today AND status = SENT)
- State transitions: no skipping states (DRAFT → PAID is invalid)

### Monetary Rules (Critical)
- ALL monetary values stored as integers (pence/cents) — NEVER float
- Currency: ISO 4217 codes (GBP, USD, EUR) — never symbols
- Display: divide by 100 and format with `formatCurrency(amount, currency)`
- Never perform arithmetic on formatted display strings

### RBAC Roles
- OWNER: full access including billing and user management
- ADMIN: all except billing
- ACCOUNTANT: read/write invoices and payments, no user management
- VIEWER: read-only

## Business Rules
1. An invoice total MUST equal sum of line items + tax - discounts
2. A vendor cannot be deleted if outstanding invoice balance > 0
3. Invoice numbers: [VENDOR_PREFIX]-[YEAR]-[SEQUENCE] e.g. ACME-2025-00142
4. Payment terms: NET_7 / NET_15 / NET_30 / NET_60

## Common Mistakes to Avoid
- Never use float for money calculations
- Never allow status transitions that skip states
- Never create a Customer entity accessible across different Vendors
```

### `state-management.md` (Frontend)

```markdown
# State Management Reference

## Principle: Right state in the right place
- API data → React Query (never Zustand)
- Auth session → Zustand (persisted)
- UI state (open/closed, filter values) → useState

## React Query Key Conventions
['invoices'] — all invoices list
['invoices', { status, page }] — filtered/paged
['invoices', id] — single invoice
['customers'] — all customers
['customers', id] — single customer

## Zustand Store Slices
auth.store.ts → user, token, permissions
ui.store.ts   → sidebar open, theme, notification prefs
filters.store.ts → active filters (persisted to localStorage)

## Optimistic Updates Pattern
On mutation: update query cache immediately, rollback on error.
Use React Query's `onMutate`/`onError`/`onSettled` — see api/hooks/useCreateInvoice.ts.
```

---

## 7. The `.claude/settings.json` — Hooks for All Three Stacks

### Backend hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command", "command": "npx eslint --fix ${CLAUDE_TOOL_INPUT_PATH} 2>/dev/null || true", "async": true },
          { "type": "command", "command": "npx tsc --noEmit 2>&1 | head -20", "async": false }
        ]
      }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": "node .claude/scripts/run-affected-tests.js" }] }
    ]
  }
}
```

### Frontend hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command", "command": "npx eslint --fix ${CLAUDE_TOOL_INPUT_PATH} 2>/dev/null || true", "async": true },
          { "type": "command", "command": "npx tsc --noEmit 2>&1 | head -20", "async": false }
        ]
      }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": "npx vitest run --reporter=verbose 2>&1 | tail -30", "async": false }] }
    ]
  }
}
```

### Mobile hooks

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          { "type": "command", "command": "npx eslint --fix ${CLAUDE_TOOL_INPUT_PATH} 2>/dev/null || true", "async": true },
          { "type": "command", "command": "npx tsc --noEmit 2>&1 | head -20", "async": false }
        ]
      }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": "npx jest --passWithNoTests --onlyChanged 2>&1 | tail -20" }] }
    ]
  }
}
```

---

## 8. Note for Native iOS (Swift) and Native Android (Kotlin) Teams

If you're building fully native rather than React Native, the CLAUDE.md structure is the same — adapt the tech stack, conventions, and reference implementations.

### Swift / iOS CLAUDE.md additions:

```markdown
## Swift Conventions
- Swift 5.9+, target iOS 15+
- SwiftUI for all new views (no UIKit except for features SwiftUI can't support)
- Combine for reactive streams, async/await for one-shot async operations
- MVVM architecture: View → ViewModel (ObservableObject) → Repository → API
- No force unwrapping — use guard let, if let, or ?? with defaults
- Keychain for all auth tokens (never UserDefaults)
- Core Data for local persistence (use NSPersistentCloudKitContainer for iCloud sync)
- XCTest + ViewInspector for UI tests, XCUITest for E2E

## iOS Platform Rules
- Support Dynamic Type (use .font(.body) not fixed sizes)
- Support both Light and Dark mode (use semantic colours)
- Support accessibility: VoiceOver labels on all interactive elements
- Universal app: iPhone + iPad layouts with size classes
```

### Kotlin / Android CLAUDE.md additions:

```markdown
## Kotlin Conventions
- Kotlin 2.x, target Android API 26+ (Android 8.0)
- Jetpack Compose for all new UI (no XML layouts except legacy screens)
- Coroutines + Flow for async/reactive
- MVVM + Repository pattern: Composable → ViewModel → Repository → API
- No nullable unless necessary — use sealed classes for state
- EncryptedSharedPreferences for auth tokens (never SharedPreferences)
- Room for local database
- Hilt for dependency injection
- JUnit5 + Compose UI testing + Espresso for E2E

## Android Platform Rules
- Support back gesture (predictive back on Android 13+)
- Handle configuration changes correctly (ViewModel survives rotation)
- Support system font size scaling (sp units, not dp for text)
- Handle all lifecycle states (onPause, onStop, onDestroy)
- Request permissions at point of use with rationale, not on app launch
```

---

## 9. The 95% Accuracy Maturity Model

| Level | What You Have | Typical Accuracy |
|---|---|---|
| **1 — Basic** | CLAUDE.md with tech stack and file structure | 60–70% |
| **2 — Intermediate** | + Architecture rules, naming conventions, security rules, what-NOT-to-do | 75–85% |
| **3 — Advanced** | + Domain model, reference implementations, skills, testing standards | 85–92% |
| **4 — Expert** | + Subagents per role, hooks wired in, memory.md maintained, context files per domain area | 92–97% |

**The calibration loop:**
```
Claude produces wrong output
       ↓
Ask: "What context was missing?"
       ↓
Add that context to CLAUDE.md or the relevant context file
       ↓
Claude gets it right automatically next time
```

Every mistake Claude makes is a CLAUDE.md improvement opportunity. The fastest path to 95%+ is running the loop consistently for the first two weeks.

---

## 10. Complete File Reference

| File | Purpose | Commit? | All Stacks? |
|---|---|---|---|
| `CLAUDE.md` | Main project config | ✅ Yes | ✅ All |
| `.claude/settings.json` | Hooks + MCP + permissions | ✅ Yes | ✅ All |
| `.claude/skills/*.md` | Reusable slash-command workflows | ✅ Yes | ✅ All |
| `.claude/agents/*.md` | Custom subagent definitions | ✅ Yes | ✅ All |
| `.claude/context/domain-model.md` | Business domain knowledge | ✅ Yes | ✅ All |
| `.claude/context/design-system.md` | UI tokens + component inventory | ✅ Yes | Frontend only |
| `.claude/context/state-management.md` | State patterns + query keys | ✅ Yes | Frontend + Mobile |
| `.claude/context/navigation-structure.md` | Screen hierarchy + deep links | ✅ Yes | Mobile only |
| `.claude/memory.md` | Evolving project decisions + known issues | ✅ Yes | ✅ All |
| `~/.claude/CLAUDE.md` | Personal preferences (all projects) | ❌ Personal | N/A |
| `.env` | Environment variables | ❌ Never | ✅ All |

---

## Summary

CLAUDE.md is not documentation — it's runtime configuration. Every section you write is context that Claude carries into every session, every tool call, every line of code it generates for your project.

The three setups in this article give you a production-ready starting point. Your job from here:

1. **Copy the template for your stack.** Fill in the specifics — your tech versions, your architecture decisions, your team's conventions.
2. **Add Skills for your most repeated tasks.** The ones you type the same long prompt for every time.
3. **Add Subagents with the right model per role.** Security review on Opus 4.6. Summarisation on Haiku. Everything else on Sonnet 4.6.
4. **Wire up Hooks.** Lint and type-check on every file write. Tests on every completed turn.
5. **Run the calibration loop.** Every time Claude misses, add the missing context.

Level 4 takes a few hours to set up. It pays back every day after.

---

*Next: Article 9 — What Are MCPs? Types, Benefits, and When to Use Each*
