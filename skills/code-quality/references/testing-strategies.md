# Testing Strategies

Practical testing guidance organized by what to test and how.

## Testing Pyramid

```
         ┌─────────┐
         │   E2E   │  5-10% — Critical user journeys only
         ├─────────┤
         │  Integ  │  20-30% — API contracts, DB queries, component integration
         ├─────────┤
         │  Unit   │  60-70% — Business logic, utilities, pure functions
         └─────────┘
```

## Unit Testing with Vitest

### Setup
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
import path from "path";

export default defineConfig({
  test: {
    globals: true,
    environment: "node", // or "jsdom" for React
    setupFiles: ["./tests/setup.ts"],
  },
  resolve: {
    alias: { "@": path.resolve(__dirname, "./src") },
  },
});
```

### Testing Pure Functions
```typescript
import { describe, it, expect } from "vitest";
import { formatCurrency, slugify, calculateDiscount } from "@/lib/utils";

describe("formatCurrency", () => {
  it("formats positive amounts", () => {
    expect(formatCurrency(1234.56)).toBe("$1,234.56");
  });

  it("handles zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });

  it("handles negative amounts", () => {
    expect(formatCurrency(-50)).toBe("-$50.00");
  });
});
```

### Testing API Routes (Hono)
```typescript
import { describe, it, expect, beforeEach } from "vitest";
import { app } from "@/index";

describe("POST /api/users", () => {
  it("creates a user with valid data", async () => {
    const res = await app.request("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: "Alice", email: "alice@test.com" }),
    });

    expect(res.status).toBe(201);
    const body = await res.json();
    expect(body).toHaveProperty("id");
    expect(body.name).toBe("Alice");
  });

  it("rejects invalid email", async () => {
    const res = await app.request("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: "Alice", email: "not-an-email" }),
    });

    expect(res.status).toBe(400);
  });
});
```

### Testing React Hooks
```typescript
import { renderHook, act } from "@testing-library/react";
import { useCounter } from "@/hooks/useCounter";

describe("useCounter", () => {
  it("increments and decrements", () => {
    const { result } = renderHook(() => useCounter(0));

    act(() => result.current.increment());
    expect(result.current.count).toBe(1);

    act(() => result.current.decrement());
    expect(result.current.count).toBe(0);
  });
});
```

## Integration Testing

### Database Integration
```typescript
import { describe, it, expect, beforeEach } from "vitest";
import { db } from "@/lib/db";
import { users } from "@/lib/db/schema";
import { createUser, getUser } from "@/services/users";

beforeEach(async () => {
  await db.delete(users); // Clean slate
});

describe("User Service", () => {
  it("creates and retrieves a user", async () => {
    const created = await createUser({ name: "Alice", email: "alice@test.com" });
    const retrieved = await getUser(created.id);

    expect(retrieved).toBeDefined();
    expect(retrieved!.email).toBe("alice@test.com");
  });
});
```

## E2E Testing with Playwright

### Setup
```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  webServer: {
    command: "npm run dev",
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: "http://localhost:3000",
  },
});
```

### Critical Path Test
```typescript
import { test, expect } from "@playwright/test";

test("user can sign up and reach dashboard", async ({ page }) => {
  await page.goto("/signup");
  await page.fill('[name="email"]', "test@example.com");
  await page.fill('[name="password"]', "SecureP@ss123");
  await page.click('button[type="submit"]');

  // Should redirect to dashboard
  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toContainText("Dashboard");
});
```

## What to Test at Each Level

| Layer | Test Type | What to Assert |
|-------|-----------|---------------|
| Utility functions | Unit | Input/output correctness, edge cases |
| API routes | Integration | Status codes, response shapes, auth |
| Database queries | Integration | CRUD operations, constraints, migrations |
| React components | Unit | Renders correctly, handles interactions |
| Full user flows | E2E | Critical paths work end-to-end |
| Auth flows | E2E | Login, signup, protected routes |

## Coverage Targets

- **Critical business logic**: 90%+
- **API routes**: 80%+
- **UI components**: 60-70% (focus on logic, not layout)
- **Utility functions**: 95%+
- **Overall project**: 70-80% is healthy

## Anti-Patterns to Avoid

- Testing implementation details instead of behavior
- Mocking everything (you're testing mocks, not code)
- Testing trivial getters/setters
- Brittle snapshot tests that break on every change
- E2E tests for things unit tests can cover
- 100% coverage as a goal (diminishing returns past 80%)
