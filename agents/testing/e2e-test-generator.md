# E2E Test Generator Agent

Specialized agent for generating end-to-end tests using Playwright for the AI BotKit SaaS application and WordPress plugin integration.

## Purpose

Generate complete Playwright E2E tests:
- Test complete user flows in browser
- Validate SaaS ↔ WordPress integration
- Test authentication flows
- Verify chat widget functionality
- Test payment flows (Stripe test mode)

## When to Use

This agent is invoked by `/next-phase` during Phase 7 (Testing):
- After implementation is complete (Phase 6)
- For critical user flow testing
- For integration testing between SaaS and WordPress
- For regression testing

## What Gets Tested

### Test Categories

| Category | Focus | Examples |
|----------|-------|----------|
| **Auth Flows** | Login, logout, registration | Login page, session persistence |
| **Chatbot CRUD** | Create, edit, delete chatbots | Dashboard flows |
| **Chat Widget** | Real-time chat, streaming | Widget embed, messaging |
| **Payments** | Checkout, subscription management | Stripe integration |
| **WordPress** | Plugin configuration, shortcode | Admin settings, frontend embed |

## Test Patterns

### 1. Authentication Tests

```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('TC-REG-AUTH-001: Login with valid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'testpassword123');
    await page.click('[data-testid="login-button"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL('/dashboard');

    // Should show user info
    await expect(page.locator('[data-testid="user-email"]')).toContainText('test@example.com');
  });

  test('TC-REG-AUTH-002: Session persistence', async ({ page, context }) => {
    // Login first
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'testpassword123');
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/dashboard');

    // Close and reopen (simulate browser restart)
    const cookies = await context.cookies();
    const newContext = await page.context().browser()!.newContext();
    await newContext.addCookies(cookies);
    const newPage = await newContext.newPage();

    await newPage.goto('/dashboard');

    // Should still be logged in
    await expect(newPage).toHaveURL('/dashboard');
    await expect(newPage.locator('[data-testid="user-email"]')).toBeVisible();
  });

  test('Redirect to login when not authenticated', async ({ page }) => {
    await page.goto('/dashboard');

    // Should redirect to login
    await expect(page).toHaveURL(/\/login/);
  });
});
```

### 2. Chatbot CRUD Tests

```typescript
// tests/e2e/chatbots.spec.ts
import { test, expect } from '@playwright/test';
import { loginAsUser } from './helpers/auth';

test.describe('Chatbot Management', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsUser(page, 'test@example.com', 'testpassword123');
  });

  test('TC-001: Create chatbot with valid data', async ({ page }) => {
    await page.goto('/dashboard/chatbots');

    // Click create button
    await page.click('[data-testid="create-chatbot-button"]');

    // Fill form
    await page.fill('[data-testid="chatbot-name"]', 'Test Bot');
    await page.selectOption('[data-testid="chatbot-personality"]', 'professional');
    await page.fill('[data-testid="chatbot-welcome"]', 'Hello! How can I help?');

    // Submit
    await page.click('[data-testid="submit-chatbot"]');

    // Should show success
    await expect(page.locator('[data-testid="toast-success"]')).toBeVisible();

    // Should appear in list
    await expect(page.locator('text=Test Bot')).toBeVisible();
  });

  test('TC-002: Create chatbot - name validation', async ({ page }) => {
    await page.goto('/dashboard/chatbots/new');

    // Leave name empty, try to submit
    await page.click('[data-testid="submit-chatbot"]');

    // Should show validation error
    await expect(page.locator('[data-testid="name-error"]')).toContainText('Name is required');
  });

  test('TC-REG-BOT-002: Update chatbot settings', async ({ page }) => {
    await page.goto('/dashboard/chatbots');

    // Click on existing chatbot
    await page.click('[data-testid="chatbot-item"]:first-child');

    // Update name
    await page.fill('[data-testid="chatbot-name"]', 'Updated Bot Name');
    await page.click('[data-testid="save-settings"]');

    // Should show success
    await expect(page.locator('[data-testid="toast-success"]')).toBeVisible();

    // Should reflect new name
    await page.goto('/dashboard/chatbots');
    await expect(page.locator('text=Updated Bot Name')).toBeVisible();
  });
});
```

### 3. Chat Widget Tests

```typescript
// tests/e2e/chat-widget.spec.ts
import { test, expect } from '@playwright/test';
import { loginAsUser, createChatbot, trainChatbot } from './helpers';

test.describe('Chat Widget', () => {
  let chatbotId: string;

  test.beforeAll(async ({ browser }) => {
    const page = await browser.newPage();
    await loginAsUser(page, 'test@example.com', 'testpassword123');
    chatbotId = await createChatbot(page, 'E2E Test Bot');
    await trainChatbot(page, chatbotId, 'Our pricing is $29/month for Basic plan.');
    await page.close();
  });

  test('TC-RAG-001: Context retrieval accuracy', async ({ page }) => {
    // Go to chatbot preview/embed page
    await loginAsUser(page, 'test@example.com', 'testpassword123');
    await page.goto(`/dashboard/chatbots/${chatbotId}/preview`);

    // Open chat widget
    await page.click('[data-testid="chat-toggle"]');

    // Send message about pricing
    await page.fill('[data-testid="chat-input"]', 'What is the price of Basic plan?');
    await page.click('[data-testid="send-button"]');

    // Wait for response (streaming)
    await page.waitForSelector('[data-testid="bot-message"]');

    // Response should mention the trained price
    const response = await page.locator('[data-testid="bot-message"]:last-child').textContent();
    expect(response).toMatch(/\$29|29 dollars|twenty-nine/i);
  });

  test('TC-RAG-002: Fallback for unknown query', async ({ page }) => {
    await loginAsUser(page, 'test@example.com', 'testpassword123');
    await page.goto(`/dashboard/chatbots/${chatbotId}/preview`);

    await page.click('[data-testid="chat-toggle"]');
    await page.fill('[data-testid="chat-input"]', 'What is the weather in Tokyo?');
    await page.click('[data-testid="send-button"]');

    await page.waitForSelector('[data-testid="bot-message"]');

    // Should show fallback message (not hallucinated answer)
    const response = await page.locator('[data-testid="bot-message"]:last-child').textContent();
    expect(response).not.toMatch(/tokyo|weather|temperature/i);
  });

  test('TC-REG-RAG-001: Streaming chat response', async ({ page }) => {
    await loginAsUser(page, 'test@example.com', 'testpassword123');
    await page.goto(`/dashboard/chatbots/${chatbotId}/preview`);

    await page.click('[data-testid="chat-toggle"]');
    await page.fill('[data-testid="chat-input"]', 'Tell me about pricing');

    // Listen for streaming
    let tokensReceived = 0;
    page.on('response', async (response) => {
      if (response.url().includes('/api/chat')) {
        tokensReceived++;
      }
    });

    await page.click('[data-testid="send-button"]');

    // Wait for complete response
    await page.waitForSelector('[data-testid="bot-message"]');

    // Should have received streaming response
    await expect(page.locator('[data-testid="bot-message"]')).toBeVisible();
  });
});
```

### 4. Payment Flow Tests

```typescript
// tests/e2e/payments.spec.ts
import { test, expect } from '@playwright/test';
import { loginAsUser } from './helpers/auth';

test.describe('Payment Flows', () => {
  test('TC-REG-STRIPE-001: Upgrade to paid plan', async ({ page }) => {
    await loginAsUser(page, 'free-user@example.com', 'testpassword123');
    await page.goto('/dashboard/settings/billing');

    // Click upgrade
    await page.click('[data-testid="upgrade-button"]');

    // Should redirect to Stripe Checkout
    await expect(page).toHaveURL(/checkout\.stripe\.com/);

    // Fill Stripe test card
    await page.fill('[data-testid="cardNumber"]', '4242424242424242');
    await page.fill('[data-testid="cardExpiry"]', '12/30');
    await page.fill('[data-testid="cardCvc"]', '123');
    await page.fill('[data-testid="billingName"]', 'Test User');

    // Submit
    await page.click('[data-testid="submit-button"]');

    // Should redirect back to success page
    await page.waitForURL(/\/dashboard\?success=true/);

    // Should show upgraded plan
    await expect(page.locator('[data-testid="current-plan"]')).toContainText('Basic');
  });
});
```

### 5. WordPress Integration Tests

```typescript
// tests/e2e/wordpress.spec.ts
import { test, expect } from '@playwright/test';

const WP_URL = process.env.WP_TEST_URL || 'http://localhost:8080';

test.describe('WordPress Plugin Integration', () => {
  test('TC-REG-WP-001: Plugin activation', async ({ page }) => {
    // Login to WP Admin
    await page.goto(`${WP_URL}/wp-admin`);
    await page.fill('#user_login', 'admin');
    await page.fill('#user_pass', 'password');
    await page.click('#wp-submit');

    // Go to plugins
    await page.goto(`${WP_URL}/wp-admin/plugins.php`);

    // Find AI BotKit plugin
    const pluginRow = page.locator('tr[data-slug="ai-botkit-for-lead-generation"]');
    await expect(pluginRow).toBeVisible();

    // Should be activated (check for Deactivate link)
    await expect(pluginRow.locator('a.deactivate')).toBeVisible();
  });

  test('TC-WP-001: Shortcode renders chatbot', async ({ page }) => {
    // View page with shortcode
    await page.goto(`${WP_URL}/test-chatbot-page`);

    // Chat widget should be rendered
    await expect(page.locator('[data-aibotkit-widget]')).toBeVisible();

    // Should be able to open chat
    await page.click('[data-testid="chat-toggle"]');
    await expect(page.locator('[data-testid="chat-container"]')).toBeVisible();
  });

  test('WordPress to SaaS API connection', async ({ page }) => {
    await page.goto(`${WP_URL}/test-chatbot-page`);

    // Open chat and send message
    await page.click('[data-testid="chat-toggle"]');
    await page.fill('[data-testid="chat-input"]', 'Hello');
    await page.click('[data-testid="send-button"]');

    // Should receive response from SaaS
    await page.waitForSelector('[data-testid="bot-message"]');
    await expect(page.locator('[data-testid="bot-message"]')).toBeVisible();
  });
});
```

## Output Format

```markdown
# E2E Test Generation Report

Generated: [Date]
Test Framework: Playwright
Browser Targets: Chromium, Firefox, WebKit

## Files Created

| File | Tests | Test Cases Covered |
|------|-------|-------------------|
| `tests/e2e/auth.spec.ts` | 5 | TC-REG-AUTH-001, TC-REG-AUTH-002 |
| `tests/e2e/chatbots.spec.ts` | 8 | TC-001, TC-002, TC-REG-BOT-002 |
| `tests/e2e/chat-widget.spec.ts` | 6 | TC-RAG-001, TC-RAG-002, TC-REG-RAG-001 |
| `tests/e2e/payments.spec.ts` | 3 | TC-REG-STRIPE-001 |
| `tests/e2e/wordpress.spec.ts` | 4 | TC-REG-WP-001, TC-WP-001 |

## Run Tests

```bash
# Run all E2E tests
pnpm test:e2e

# Run with UI
pnpm test:e2e --ui

# Run specific file
pnpm test:e2e tests/e2e/auth.spec.ts

# Run in specific browser
pnpm test:e2e --project=chromium
```

## CI Configuration

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: pnpm exec playwright install
      - run: pnpm test:e2e
```
```

## Integration

### Invoked By

- `/next-phase` command (Phase 7)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:testing:e2e-test-generator"
```

## Related Agents

- `test-case-generator` - Provides test case definitions
- `unit-test-writer` - Writes unit tests
- `e2e-test-runner` - Runs E2E tests
- `integration-test-specialist` - Writes integration tests
