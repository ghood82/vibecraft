# Playwright E2E Testing Patterns

## Setup

```bash
npm init playwright@latest
# Creates: playwright.config.ts, tests/, .github/workflows/playwright.yml
```

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ["html"],
    ["list"],
    process.env.CI ? ["github"] : ["line"],
  ],
  use: {
    baseURL: process.env.BASE_URL || "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },
    { name: "mobile-chrome", use: { ...devices["Pixel 5"] } },
    { name: "mobile-safari", use: { ...devices["iPhone 12"] } },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

## Core Patterns

### Page Object Model
```typescript
// pages/login.page.ts
import { type Page, type Locator, expect } from "@playwright/test";

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel("Email");
    this.passwordInput = page.getByLabel("Password");
    this.submitButton = page.getByRole("button", { name: "Sign in" });
    this.errorMessage = page.getByRole("alert");
  }

  async goto() {
    await this.page.goto("/login");
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}

// pages/dashboard.page.ts
export class DashboardPage {
  readonly page: Page;
  readonly heading: Locator;
  readonly sidebar: Locator;

  constructor(page: Page) {
    this.page = page;
    this.heading = page.getByRole("heading", { name: "Dashboard" });
    this.sidebar = page.getByRole("navigation");
  }

  async expectLoaded() {
    await expect(this.heading).toBeVisible();
  }
}
```

### Authentication Fixture
```typescript
// fixtures/auth.fixture.ts
import { test as base, expect } from "@playwright/test";
import { LoginPage } from "../pages/login.page";
import { DashboardPage } from "../pages/dashboard.page";

type Fixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  authenticatedPage: Page;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: "tests/.auth/user.json",
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});

export { expect };

// Global setup: save auth state
// global-setup.ts
import { chromium, type FullConfig } from "@playwright/test";

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto("http://localhost:3000/login");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByLabel("Password").fill("password123");
  await page.getByRole("button", { name: "Sign in" }).click();
  await page.waitForURL("/dashboard");
  await page.context().storageState({ path: "tests/.auth/user.json" });
  await browser.close();
}

export default globalSetup;
```

### Test Examples
```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from "../fixtures/auth.fixture";

test.describe("Authentication", () => {
  test("should login with valid credentials", async ({ loginPage, dashboardPage }) => {
    await loginPage.goto();
    await loginPage.login("test@example.com", "password123");
    await dashboardPage.expectLoaded();
  });

  test("should show error for invalid credentials", async ({ loginPage }) => {
    await loginPage.goto();
    await loginPage.login("wrong@example.com", "wrong");
    await loginPage.expectError("Invalid email or password");
  });

  test("should redirect to login when unauthenticated", async ({ page }) => {
    await page.goto("/dashboard");
    await expect(page).toHaveURL(/.*login/);
  });
});

// tests/e2e/crud.spec.ts
test.describe("Products CRUD", () => {
  test.use({ storageState: "tests/.auth/admin.json" });

  test("should create a new product", async ({ page }) => {
    await page.goto("/products");
    await page.getByRole("button", { name: "Add Product" }).click();

    await page.getByLabel("Name").fill("Test Product");
    await page.getByLabel("Price").fill("29.99");
    await page.getByLabel("Description").fill("A test product");
    await page.getByRole("button", { name: "Save" }).click();

    await expect(page.getByText("Product created successfully")).toBeVisible();
    await expect(page.getByRole("cell", { name: "Test Product" })).toBeVisible();
  });

  test("should edit a product", async ({ page }) => {
    await page.goto("/products");
    await page.getByRole("row", { name: /Test Product/ })
      .getByRole("button", { name: "Edit" }).click();

    await page.getByLabel("Name").clear();
    await page.getByLabel("Name").fill("Updated Product");
    await page.getByRole("button", { name: "Save" }).click();

    await expect(page.getByText("Product updated")).toBeVisible();
  });

  test("should delete a product", async ({ page }) => {
    await page.goto("/products");
    await page.getByRole("row", { name: /Updated Product/ })
      .getByRole("button", { name: "Delete" }).click();

    // Confirm dialog
    await page.getByRole("button", { name: "Confirm" }).click();
    await expect(page.getByText("Product deleted")).toBeVisible();
    await expect(page.getByRole("cell", { name: "Updated Product" })).not.toBeVisible();
  });
});
```

## Best Practice Locators

```typescript
// Preferred (resilient to implementation changes):
page.getByRole("button", { name: "Submit" })    // Semantic role
page.getByLabel("Email address")                  // Form label
page.getByPlaceholder("Search...")                // Placeholder text
page.getByText("Welcome back")                    // Visible text
page.getByTestId("submit-button")                 // data-testid attribute

// Avoid (fragile):
page.locator(".btn-primary")                      // CSS class
page.locator("#submit")                           // ID
page.locator("div > form > button:nth-child(2)")  // Complex CSS
page.locator("xpath=//button")                    // XPath
```

## API Testing with Playwright

```typescript
import { test, expect } from "@playwright/test";

test.describe("API Tests", () => {
  test("should create and retrieve a user", async ({ request }) => {
    // Create
    const createResponse = await request.post("/api/v1/users", {
      data: { name: "Alice", email: "alice@test.com" },
    });
    expect(createResponse.ok()).toBeTruthy();
    const user = await createResponse.json();
    expect(user.name).toBe("Alice");

    // Retrieve
    const getResponse = await request.get(`/api/v1/users/${user.id}`);
    expect(getResponse.ok()).toBeTruthy();
    const fetched = await getResponse.json();
    expect(fetched.email).toBe("alice@test.com");

    // Cleanup
    await request.delete(`/api/v1/users/${user.id}`);
  });
});
```

## Visual Regression Testing

```typescript
test("homepage matches snapshot", async ({ page }) => {
  await page.goto("/");
  await expect(page).toHaveScreenshot("homepage.png", {
    maxDiffPixels: 100,
    fullPage: true,
  });
});

test("component visual test", async ({ page }) => {
  await page.goto("/storybook/button");
  const button = page.getByTestId("primary-button");
  await expect(button).toHaveScreenshot("primary-button.png");
});

// Update snapshots: npx playwright test --update-snapshots
```

## Network Mocking

```typescript
test("should handle API errors gracefully", async ({ page }) => {
  // Mock API to return error
  await page.route("**/api/v1/products", (route) => {
    route.fulfill({
      status: 500,
      contentType: "application/json",
      body: JSON.stringify({ error: { message: "Internal server error" } }),
    });
  });

  await page.goto("/products");
  await expect(page.getByText("Something went wrong")).toBeVisible();
  await expect(page.getByRole("button", { name: "Retry" })).toBeVisible();
});

test("should show loading state", async ({ page }) => {
  // Delay response to test loading state
  await page.route("**/api/v1/dashboard", async (route) => {
    await new Promise(resolve => setTimeout(resolve, 2000));
    await route.continue();
  });

  await page.goto("/dashboard");
  await expect(page.getByTestId("loading-skeleton")).toBeVisible();
});
```

## CI Integration

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [push, pull_request]

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

## Debugging Tips

```bash
# Run in headed mode (see the browser)
npx playwright test --headed

# Run in debug mode (step through)
npx playwright test --debug

# Run specific test file
npx playwright test tests/e2e/auth.spec.ts

# Run with trace viewer
npx playwright test --trace on
npx playwright show-trace test-results/trace.zip

# Generate tests by recording
npx playwright codegen http://localhost:3000
```
