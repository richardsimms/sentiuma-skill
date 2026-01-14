# Discovery Framework Reference

Guidance for framing product context using established discovery methodologies.

## Extracting Diagrams from Code

### Entity Relationship Diagram

**Sources to scan:**
- `models/`, `entities/`, `schemas/` directories
- Prisma schema (`schema.prisma`)
- TypeORM/Sequelize models
- GraphQL type definitions
- TypeScript interfaces in `types/`

**Extraction patterns:**
```
// Foreign key → relationship
userId: string  →  User ||--o{ ThisEntity

// Array field → one-to-many  
posts: Post[]  →  User ||--o{ Post

// Optional field → zero-or-one
profile?: Profile  →  User |o--|| Profile

// Required field → exactly-one
company: Company  →  Job }|--|| Company
```

**Cardinality notation:**
| Symbol | Meaning |
|--------|---------|
| `\|\|` | Exactly one |
| `\|o` | Zero or one |
| `o{` | Zero or many |
| `\|{` | One or many |

### User Journey Flowchart

**Sources to scan:**
- Route definitions (`routes/`, `pages/`, `app/`)
- Navigation components
- Stepper/wizard components
- State machines (XState, Redux)

**Extraction patterns:**
```
// Route → Node
/jobs/search  →  Search[Search Jobs]

// Nested routes → Flow
/apply/:id/resume → /apply/:id/review → /apply/:id/confirm
  →  Resume --> Review --> Confirm

// Conditional render → Decision
{isLoggedIn ? <Apply/> : <Login/>}
  →  Decision{Logged in?} -->|Yes| Apply
                          -->|No| Login
```

**Node shapes:**
| Shape | Use for |
|-------|---------|
| `([text])` | Entry/exit points |
| `[text]` | Actions/pages |
| `{text}` | Decisions |
| `[(text)]` | Data stores |

### Integration Context Diagram

**Sources to scan:**
- Environment variables (`.env`, `.env.example`)
- API client files (`/lib/`, `/clients/`, `/services/`)
- Package.json dependencies (SDKs)
- Webhook handlers

**Extraction patterns:**
```
// SDK import → External service
import Stripe from 'stripe'  →  Product --> Stripe

// Env variable → Service dependency
ALGOLIA_API_KEY  →  Product --> Algolia

// Fetch to external URL → Integration
fetch('https://api.sendgrid.com')  →  Product --> SendGrid
```

**Subgraph grouping:**
- Group internal services together
- Group external services by type (payments, comms, analytics)
- Show data flow direction with arrows

### Component Inventory Extraction

**Sources to scan:**
- `src/components/` directory
- `src/ui/` or `components/ui/` for design system primitives
- Feature folders: `src/features/*/components/`

**What to capture:**
```
| Component | Location | Purpose | Extensible? |
|-----------|----------|---------|-------------|
```

**Extensibility signals:**
- Props interface is generic → extensible
- Hardcoded values inside → limited
- Accepts `children` or render props → extensible
- Tightly coupled to specific data → limited

**Pattern identification:**
- Look for shared patterns (all cards use same shadow, all modals use same primitive)
- Note form libraries (react-hook-form, formik)
- Note UI primitives (Radix, Headless UI, shadcn)

### State Management Pattern Extraction

**Sources to scan:**
- `src/providers/`, `src/context/` — Context providers
- `src/stores/`, `src/state/` — Zustand/Redux stores
- `src/hooks/` — Custom hooks often reveal state patterns
- `app/layout.tsx` — Provider hierarchy

**Global state signals:**
```typescript
// Context pattern
export const SomeContext = createContext()
export function SomeProvider({ children }) { ... }

// Zustand pattern
export const useStore = create((set) => ({ ... }))

// Redux pattern
export const store = configureStore({ ... })
```

**Server state signals:**
```typescript
// React Query
useQuery(['key'], fetchFn)
useMutation(mutationFn)

// SWR
useSWR('/api/...', fetcher)
```

**Persistence signals:**
```typescript
localStorage.getItem('key')
localStorage.setItem('key', value)
sessionStorage.getItem('key')
```

### Type Definition Extraction

**Sources to scan:**
- `src/types/` — Shared type definitions
- `src/models/` — Domain models (often with Drizzle/Prisma)
- `*.d.ts` files — Type declaration files
- `src/schemas/` — Zod/Yup validation schemas (derive types)

**Priority types to extract:**
1. Domain entities (User, Article, Order, etc.)
2. Status enums (constrain behavior)
3. API request/response types
4. Form data types
5. Config/preferences types

**Type extraction pattern:**
```typescript
// Include in context output:
type EntityName = {
  id: string
  // ... key fields only, not internal/audit fields
}

type StatusEnum = 'value1' | 'value2' | 'value3'
```

**What to omit:**
- Internal/technical types (RequestConfig, ApiError)
- Utility types (Partial<T>, Pick<T, K>)
- Component prop types (unless shared pattern)

---

## Continuous Discovery Habits (Teresa Torres)

### Core Concepts

**Outcome**: The measurable change in customer behavior that drives business results.
- Look for metrics tracked in analytics code
- Identify what success looks like based on existing dashboards

**Opportunity**: An unmet customer need, pain point, or desire.
- Surface from gaps in current capabilities
- Identify from TODO comments expressing user problems
- Extract from error handling that reveals edge cases

**Assumption**: A belief that must be true for a solution to succeed.
- Hardcoded business rules reveal assumptions
- Default values encode assumptions about user behavior
- Permission structures assume user roles

**Experiment**: A test designed to evaluate an assumption.
- Feature flags indicate areas of uncertainty
- A/B test infrastructure shows what's being validated

### Opportunity Solution Tree Structure

When identifying opportunities from code:

```
Outcome (business/user metric)
├── Opportunity (user need/pain)
│   ├── Solution (feature/capability)
│   │   ├── Assumption
│   │   └── Experiment
│   └── Solution
└── Opportunity
    └── Solution
```

## Jobs-to-be-Done

### Extracting Jobs from Code

Look for:
- **Functional jobs**: What task is the user completing? (Routes, workflows)
- **Emotional jobs**: How does the user want to feel? (Copy, messaging)
- **Social jobs**: How does the user want to be perceived? (Sharing, profiles)

### Job Statement Format

When [situation], I want to [motivation], so I can [outcome].

Map to code:
- **Situation**: Entry points, triggers, conditions
- **Motivation**: Core action/transaction
- **Outcome**: Success states, confirmations, next steps

## Testing Business Ideas

### Assumption Categories

**Desirability**: Do users want this?
- Evidence: Usage analytics, feature adoption
- Risk signals: Low engagement features, abandoned flows

**Feasibility**: Can we build/deliver this?
- Evidence: Technical constraints, integration limits
- Risk signals: TODO/FIXME density, workarounds

**Viability**: Should we build this?
- Evidence: Business rules, pricing logic
- Risk signals: Complex conditional logic, edge case handling

### Evidence Strength

From codebase analysis, categorize evidence:

| Type | Strength | Example |
|------|----------|---------|
| Production data | Strong | Analytics events, DB queries |
| Feature flags | Medium | Experiments in progress |
| Comments/TODOs | Weak | Intent without validation |
| Hardcoded values | Assumption | Needs testing |

## Mapping Code to Discovery Artifacts

### Routes → Capabilities → Jobs

```
/api/jobs/apply → "Apply to Job" capability → 
  Job: "When I find a relevant role, I want to express interest quickly, 
        so I can be considered without friction"
```

### Models → Entities → Value Propositions

```
User model with preferences → "Personalization" entity →
  Value prop: "We remember what matters to you"
```

### Error Handling → Pain Points → Opportunities

```
try/catch with user message → Failure mode users experience →
  Opportunity: "Reduce friction when X fails"
```

### Business Rules → Constraints → Assumptions to Test

```
if (user.plan === 'free') { limit(5) } →
  Constraint: Free users limited to 5 →
  Assumption: "5 is enough for free tier to demonstrate value"
```
