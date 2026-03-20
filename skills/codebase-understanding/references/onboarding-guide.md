# Onboarding Guide: Understanding Codebases

## Phase 1: Initial Reconnaissance (15 minutes)

### Read Configuration Files
- `package.json`: Dependencies, scripts, version info
- `tsconfig.json` / `jsconfig.json`: TypeScript setup, path aliases
- Framework config: `next.config.mjs`, `vite.config.ts`, etc.
- Deployment config: `vercel.json`, `wrangler.toml`, `.github/workflows/`
- `.env.example`: Required environment variables, external services

### List Directory Structure
```bash
tree -L 2 -I node_modules
# Or: find . -maxdepth 2 -type d | sort
```

Look for patterns: Are components by type or by feature? Is there a clear src/ vs build/ separation?

### Read README.md
- Project purpose
- Tech stack summary
- Local setup instructions
- Key files to understand
- Common development commands
- Known gotchas or workarounds

## Phase 2: Architecture Understanding (20 minutes)

### Identify the Pattern
```
Next.js App Router?          → app/ directory with [route-segment] folders
Next.js Pages Router?        → pages/ and pages/api/ directories
React SPA (Vite)?           → src/components/, src/pages/, src/App.tsx
Monorepo?                   → apps/, packages/, turbo.json or pnpm-workspace.yaml
Cloudflare Workers?         → src/workers/, wrangler.toml
Express/Hono API?           → src/routes/, src/middleware/, src/controllers/
```

### Find Key Entities
Database tables/models:
```bash
find . -name "*schema*" -o -name "*models*" | head -20
grep -r "CREATE TABLE\|@Table\|defineTable\|Table(" --include="*.ts" -l
```

API routes:
```bash
find . -path "*/api/*" -type f | head -20
find . -path "*/routes/*" -type f | head -20
```

Auth setup:
```bash
grep -r "auth\|Auth\|session\|Session" --include="*.ts" -l | grep -v node_modules | head -10
```

### Understand the Entry Point
- **Next.js**: Read `app/layout.tsx` or `pages/_app.tsx` (providers, global state)
- **React SPA**: Read `src/main.tsx` or `src/index.tsx` (entry point, provider setup)
- **Express/Hono**: Read `src/index.ts` or `src/server.ts` (middleware, routes)
- **Cloudflare Workers**: Read `src/index.ts` (handler, middleware)

Look for: Global providers (theme, auth, state management), middleware, error handlers.

## Phase 3: Data Flow Understanding (30 minutes)

### Pick a Core Feature and Trace It End-to-End

Example: "How does user signup work?"

1. **Find the UI**
   - Search for signup form: `grep -r "signup\|register\|create account" --include="*.tsx" -l`
   - Open the file, understand the form fields and submission handler

2. **Trace the API Call**
   - Find the onSubmit handler
   - What endpoint does it call? `POST /api/auth/signup`
   - Any request validation or error handling?

3. **Find the API Handler**
   - Open `app/api/auth/signup/route.ts` or equivalent
   - What does it do with the request body?
   - Does it validate? Does it sanitize? Does it check for duplicates?

4. **Trace to Database**
   - What database operation happens?
   - Is there a User model/schema? Open it: `grep -r "user\|User" --include="*.ts" --include="*.sql" | grep -i schema`
   - What fields get created/updated?

5. **Find Side Effects**
   - Does it send an email? Search: `grep -r "send\|email\|mail" --include="*.ts" -l | grep -v node_modules`
   - Does it call external services (Stripe, etc.)? Search the API handler for imports
   - Does it update analytics? Search for tracking calls

6. **Check Error Handling**
   - What if email already exists?
   - What if database insert fails?
   - What does the user see?

### Repeat for 2-3 Features
Do this for 2-3 more features (login, viewing data, creating a resource). You'll start to see patterns.

## Phase 4: Pattern Recognition (20 minutes)

### Document What You See

**Component Structure**
- How are components organized? By type, by feature, flat?
- Are there any custom component patterns? (compound components, render props, hooks?)
- What's the naming convention?

**State Management**
- Is there a central store? Redux, Zustand, Jotai, Context API?
- How is async data managed? TanStack Query, SWR, manual useState + useEffect?
- Are there any custom hooks that wrap common patterns?

**Error Handling**
- How do API errors get handled?
- Are there error boundaries? Global error handlers?
- What does an error look like to the user?

**Testing**
- Is there a test suite? Find it: `ls src/**/__tests__` or `ls **/*.test.ts`
- What testing framework? Jest, Vitest, Playwright?
- Are tests organized with code or in a separate directory?

**Deployment & Environment**
- How does this get deployed? Vercel? Docker? Manual?
- What environment variables are required?
- Are there environment-specific configs? (dev, staging, prod)

## Phase 5: Documentation & Memory

### Generate Architecture Doc
Create a markdown file with:
```markdown
# [Project Name] Architecture Overview

## Stack
- Framework: Next.js 15 (App Router)
- Database: PostgreSQL via Drizzle ORM
- Auth: NextAuth v5
- Deployment: Vercel
- Testing: Vitest + Playwright

## Directory Structure
```
src/
├── app/                    # Next.js app router
│   ├── api/               # API routes
│   ├── (auth)/            # Route group for auth pages
│   └── dashboard/         # Main app routes
├── components/            # Reusable React components
├── lib/                   # Utilities, helpers
├── db/                    # Database schema, migrations
├── hooks/                 # Custom React hooks
└── types/                 # TypeScript types and schemas
```

## Key Flows

### User Authentication
1. User visits /signup → SignupForm component
2. Form submits to POST /api/auth/signup
3. Server creates user in database
4. Sends verification email via Resend
5. Redirects to /verify-email
6. After verification, user can login

### Data Display (e.g., Dashboard)
1. User visits /dashboard
2. Server component fetches data via Drizzle ORM
3. Renders dashboard layout with data
4. Client components handle interactivity

## Common Patterns
- **Components**: All use React hooks (no class components)
- **Database**: All queries via Drizzle ORM with proper types
- **API validation**: All endpoints validate with Zod schemas
- **Error handling**: Try-catch with proper logging to Sentry

## Tech Debt
- [ ] Auth flow could be simplified (consider Lucia instead of NextAuth)
- [ ] Missing API rate limiting
- [ ] Test coverage is ~40%, should be 70%+

## External Services
- Email: Resend
- Analytics: PostHog
- Error tracking: Sentry
- Storage: S3 (via AWS SDK)
```

### Update CLAUDE.md
If this is a personal project, add to your global CLAUDE.md:
```markdown
## Projects
- **[project-name]**: [1-line description]. Tech: Next.js 15, PostgreSQL, Drizzle. Auth: NextAuth. Key files: app/api/, app/dashboard/.
```

### Create Project Memory
Save the architecture doc to: `memory/projects/{project-name}.md`

Save key decisions to: `memory/projects/{project-name}-decisions.md`

## Quick Reference: File Locations

Ask yourself these questions and search:

| Question | Search Command |
|----------|---|
| Where is the auth flow? | `grep -r "auth\|Auth" --include="*.ts" -l` |
| Where are database migrations? | `find . -name "*migration*" -o -name "*schema*"` |
| Where are API routes? | `find . -path "*/api/*" -type f` |
| Where are hooks? | `grep -r "^export function use" --include="*.ts" -l` |
| Where is error handling? | `grep -r "try\|catch\|error" --include="*.ts" -l \| head` |
| Where is state management? | `grep -r "useState\|useReducer\|Context\|Zustand\|Redux" --include="*.ts" -l` |
| Where are tests? | `find . -name "*.test.ts" -o -name "*.spec.ts"` |
| Where is logging? | `grep -r "console\|logger\|log" --include="*.ts" -l` |

## Avoiding Dead Ends

Don't spend time on:
- Third-party library internals (read the docs instead)
- Node modules directory
- Build output directories (dist/, .next/, build/)
- Historical branches or archived code
- Single-use scripts or experiments

Do spend time on:
- The app entry point (root layout, main.tsx)
- Core feature flows (pick 3, trace them)
- Auth and permissions (security-critical)
- Database schema (domain understanding)
- Error handling patterns
