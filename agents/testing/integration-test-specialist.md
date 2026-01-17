# Integration Test Specialist Agent

Specialized agent for writing integration tests that verify component interactions in the AI BotKit SaaS application, focusing on database, API, and external service integrations.

## Purpose

Generate comprehensive integration tests:
- Test real database interactions (test database)
- Verify API contract compliance
- Test external service integrations (Stripe, Pinecone)
- Validate SaaS ↔ WordPress plugin communication
- Ensure proper error handling across boundaries

## When to Use

This agent is invoked by `/next-phase` during Phase 7 (Testing):
- After unit tests are written
- For testing component boundaries
- For API contract testing
- For database integration testing

## What Gets Tested

### Integration Boundaries

| Boundary | Components | Test Focus |
|----------|------------|------------|
| **API → Database** | Route handlers + Drizzle | CRUD operations, transactions |
| **API → Pinecone** | RAG engine + Vector DB | Indexing, search |
| **API → Stripe** | Payments + Webhooks | Checkout, subscriptions |
| **SaaS → WordPress** | REST API + Plugin | Authentication, data sync |

### Test Database Setup

```typescript
// tests/integration/setup.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from '@/lib/db/schema';

const connectionString = process.env.TEST_DATABASE_URL!;
const client = postgres(connectionString);
export const testDb = drizzle(client, { schema });

export async function resetDatabase() {
  // Clean all tables
  await testDb.delete(schema.messages);
  await testDb.delete(schema.conversations);
  await testDb.delete(schema.documents);
  await testDb.delete(schema.chatbots);
  await testDb.delete(schema.teamMembers);
  await testDb.delete(schema.teams);
  await testDb.delete(schema.users);
}

export async function seedTestData() {
  // Create test user
  const [user] = await testDb.insert(schema.users).values({
    email: 'test@example.com',
    name: 'Test User',
    passwordHash: 'hashed_password',
  }).returning();

  // Create test team
  const [team] = await testDb.insert(schema.teams).values({
    name: 'Test Team',
    stripeCustomerId: 'cus_test123',
    planName: 'free',
  }).returning();

  return { user, team };
}
```

## Test Patterns

### 1. Database Integration Tests

```typescript
// tests/integration/chatbot-crud.test.ts
import { describe, it, expect, beforeEach, afterAll } from 'vitest';
import { testDb, resetDatabase, seedTestData } from './setup';
import { ChatbotService } from '@/lib/services/chatbot-service';
import * as schema from '@/lib/db/schema';

describe('Chatbot CRUD Integration', () => {
  let testUser: typeof schema.users.$inferSelect;

  beforeEach(async () => {
    await resetDatabase();
    const data = await seedTestData();
    testUser = data.user;
  });

  afterAll(async () => {
    await resetDatabase();
  });

  it('creates chatbot in database', async () => {
    const chatbot = await ChatbotService.create({
      name: 'Integration Test Bot',
      userId: testUser.id,
      personality: 'professional',
    });

    expect(chatbot.id).toBeDefined();
    expect(chatbot.name).toBe('Integration Test Bot');

    // Verify in database
    const dbChatbot = await testDb.query.chatbots.findFirst({
      where: (chatbots, { eq }) => eq(chatbots.id, chatbot.id),
    });
    expect(dbChatbot).toBeDefined();
    expect(dbChatbot!.name).toBe('Integration Test Bot');
  });

  it('updates chatbot with relations intact', async () => {
    // Create chatbot with documents
    const chatbot = await ChatbotService.create({
      name: 'Bot with Docs',
      userId: testUser.id,
    });

    await testDb.insert(schema.documents).values({
      chatbotId: chatbot.id,
      title: 'Test Document',
      sourceType: 'text',
      content: 'Test content',
    });

    // Update chatbot
    await ChatbotService.update(chatbot.id, { name: 'Updated Bot' });

    // Verify documents still exist
    const documents = await testDb.query.documents.findMany({
      where: (docs, { eq }) => eq(docs.chatbotId, chatbot.id),
    });
    expect(documents).toHaveLength(1);
    expect(documents[0].title).toBe('Test Document');
  });

  it('deletes chatbot with cascade', async () => {
    const chatbot = await ChatbotService.create({
      name: 'Bot to Delete',
      userId: testUser.id,
    });

    // Add related data
    await testDb.insert(schema.documents).values({
      chatbotId: chatbot.id,
      title: 'Doc',
      sourceType: 'text',
    });

    await testDb.insert(schema.conversations).values({
      chatbotId: chatbot.id,
      sessionId: 'session123',
    });

    // Delete chatbot
    await ChatbotService.delete(chatbot.id);

    // Verify cascade deletion
    const docs = await testDb.query.documents.findMany({
      where: (d, { eq }) => eq(d.chatbotId, chatbot.id),
    });
    expect(docs).toHaveLength(0);
  });
});
```

### 2. API Integration Tests

```typescript
// tests/integration/api-chatbots.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { createServer } from '@/test-utils/server';
import { resetDatabase, seedTestData, createTestSession } from './setup';

describe('Chatbots API Integration', () => {
  let app: ReturnType<typeof createServer>;
  let authToken: string;
  let testUserId: number;

  beforeEach(async () => {
    await resetDatabase();
    const { user } = await seedTestData();
    testUserId = user.id;
    authToken = await createTestSession(user.id);
    app = createServer();
  });

  it('GET /api/chatbots returns user chatbots', async () => {
    // Create chatbots directly in DB
    await testDb.insert(schema.chatbots).values([
      { name: 'Bot 1', userId: testUserId },
      { name: 'Bot 2', userId: testUserId },
    ]);

    const response = await app.request('/api/chatbots', {
      headers: { Cookie: `session=${authToken}` },
    });

    expect(response.status).toBe(200);
    const data = await response.json();
    expect(data).toHaveLength(2);
  });

  it('POST /api/chatbots creates chatbot', async () => {
    const response = await app.request('/api/chatbots', {
      method: 'POST',
      headers: {
        Cookie: `session=${authToken}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        name: 'New Bot',
        personality: 'friendly',
      }),
    });

    expect(response.status).toBe(201);
    const data = await response.json();
    expect(data.name).toBe('New Bot');

    // Verify in database
    const dbBot = await testDb.query.chatbots.findFirst({
      where: (c, { eq }) => eq(c.id, data.id),
    });
    expect(dbBot).toBeDefined();
  });

  it('prevents access to other user chatbots', async () => {
    // Create chatbot for different user
    const [otherUser] = await testDb.insert(schema.users).values({
      email: 'other@example.com',
      name: 'Other',
      passwordHash: 'hash',
    }).returning();

    const [otherBot] = await testDb.insert(schema.chatbots).values({
      name: 'Other Bot',
      userId: otherUser.id,
    }).returning();

    // Try to access as test user
    const response = await app.request(`/api/chatbots/${otherBot.id}`, {
      headers: { Cookie: `session=${authToken}` },
    });

    expect(response.status).toBe(404); // Or 403
  });
});
```

### 3. Pinecone Integration Tests

```typescript
// tests/integration/rag-integration.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { RagEngine } from '@/lib/ai/rag-engine';
import { pinecone } from '@/lib/ai/pinecone-client';

const TEST_NAMESPACE = 'integration-test-namespace';

describe('RAG Engine Integration', () => {
  let ragEngine: RagEngine;

  beforeEach(async () => {
    ragEngine = new RagEngine();
    // Clean test namespace
    await pinecone.deleteAll({ namespace: TEST_NAMESPACE });
  });

  afterEach(async () => {
    await pinecone.deleteAll({ namespace: TEST_NAMESPACE });
  });

  it('indexes document and retrieves context', async () => {
    // Index test document
    await ragEngine.indexDocument({
      id: 'test-doc-1',
      content: 'Our Basic plan costs $29 per month and includes 5 chatbots.',
      metadata: { source: 'pricing.md' },
      namespace: TEST_NAMESPACE,
    });

    // Wait for indexing
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Search
    const results = await ragEngine.searchContext(
      'How much does the basic plan cost?',
      TEST_NAMESPACE
    );

    expect(results.length).toBeGreaterThan(0);
    expect(results[0].metadata.source).toBe('pricing.md');
  });

  it('returns empty for unrelated queries', async () => {
    await ragEngine.indexDocument({
      id: 'test-doc-2',
      content: 'Our pricing information is available on the website.',
      namespace: TEST_NAMESPACE,
    });

    await new Promise(resolve => setTimeout(resolve, 1000));

    const results = await ragEngine.searchContext(
      'What is quantum computing?',
      TEST_NAMESPACE
    );

    // Should have no high-score results
    const highScoreResults = results.filter(r => r.score > 0.7);
    expect(highScoreResults).toHaveLength(0);
  });
});
```

### 4. Stripe Integration Tests

```typescript
// tests/integration/stripe-integration.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import Stripe from 'stripe';
import { handleStripeWebhook } from '@/lib/payments/webhooks';
import { resetDatabase, seedTestData } from './setup';
import { testDb } from './setup';
import * as schema from '@/lib/db/schema';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

describe('Stripe Integration', () => {
  let testTeam: typeof schema.teams.$inferSelect;

  beforeEach(async () => {
    await resetDatabase();
    const data = await seedTestData();
    testTeam = data.team;
  });

  it('updates team plan on checkout.session.completed', async () => {
    // Simulate webhook event
    const event: Stripe.Event = {
      id: 'evt_test123',
      type: 'checkout.session.completed',
      data: {
        object: {
          id: 'cs_test123',
          customer: testTeam.stripeCustomerId,
          subscription: 'sub_test123',
          metadata: {
            teamId: String(testTeam.id),
            planName: 'basic',
          },
        } as Stripe.Checkout.Session,
      },
      // ... other event properties
    } as Stripe.Event;

    await handleStripeWebhook(event);

    // Verify team plan updated
    const updatedTeam = await testDb.query.teams.findFirst({
      where: (t, { eq }) => eq(t.id, testTeam.id),
    });

    expect(updatedTeam!.planName).toBe('basic');
    expect(updatedTeam!.stripeSubscriptionId).toBe('sub_test123');
  });

  it('reverts to free plan on subscription.deleted', async () => {
    // First set to paid plan
    await testDb.update(schema.teams)
      .set({ planName: 'basic', stripeSubscriptionId: 'sub_test123' })
      .where(eq(schema.teams.id, testTeam.id));

    // Simulate cancellation event
    const event: Stripe.Event = {
      id: 'evt_test456',
      type: 'customer.subscription.deleted',
      data: {
        object: {
          id: 'sub_test123',
          customer: testTeam.stripeCustomerId,
        } as Stripe.Subscription,
      },
    } as Stripe.Event;

    await handleStripeWebhook(event);

    // Verify reverted to free
    const updatedTeam = await testDb.query.teams.findFirst({
      where: (t, { eq }) => eq(t.id, testTeam.id),
    });

    expect(updatedTeam!.planName).toBe('free');
    expect(updatedTeam!.stripeSubscriptionId).toBeNull();
  });
});
```

### 5. WordPress Plugin API Tests

```typescript
// tests/integration/wordpress-api.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { createServer } from '@/test-utils/server';
import { resetDatabase, seedTestData } from './setup';

describe('WordPress Plugin API Integration', () => {
  let app: ReturnType<typeof createServer>;
  let apiKey: string;
  let chatbotId: number;

  beforeEach(async () => {
    await resetDatabase();
    const { user } = await seedTestData();

    // Create API key for WordPress
    const [key] = await testDb.insert(schema.apiKeys).values({
      userId: user.id,
      key: 'wp_test_api_key_123',
      name: 'WordPress Test',
    }).returning();
    apiKey = key.key;

    // Create chatbot
    const [chatbot] = await testDb.insert(schema.chatbots).values({
      name: 'WP Test Bot',
      userId: user.id,
    }).returning();
    chatbotId = chatbot.id;

    app = createServer();
  });

  it('authenticates WordPress requests with API key', async () => {
    const response = await app.request('/api/wordpress/chatbots', {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
      },
    });

    expect(response.status).toBe(200);
  });

  it('rejects invalid API key', async () => {
    const response = await app.request('/api/wordpress/chatbots', {
      headers: {
        'Authorization': 'Bearer invalid_key',
      },
    });

    expect(response.status).toBe(401);
  });

  it('returns chatbot configuration for WordPress', async () => {
    const response = await app.request(`/api/wordpress/chatbots/${chatbotId}`, {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
      },
    });

    expect(response.status).toBe(200);
    const data = await response.json();
    expect(data.name).toBe('WP Test Bot');
    expect(data.widgetConfig).toBeDefined();
  });
});
```

## Output Format

```markdown
# Integration Test Generation Report

Generated: [Date]
Test Framework: Vitest
Test Database: PostgreSQL (test instance)

## Files Created

| File | Tests | Focus Area |
|------|-------|------------|
| `tests/integration/setup.ts` | - | Test database setup |
| `tests/integration/chatbot-crud.test.ts` | 6 | Database operations |
| `tests/integration/api-chatbots.test.ts` | 8 | API + DB integration |
| `tests/integration/rag-integration.test.ts` | 4 | Pinecone integration |
| `tests/integration/stripe-integration.test.ts` | 4 | Payment webhooks |
| `tests/integration/wordpress-api.test.ts` | 5 | WP plugin API |

## Run Tests

```bash
# Run integration tests
pnpm test:integration

# With test database
DATABASE_URL=$TEST_DATABASE_URL pnpm test:integration
```
```

## Integration

### Invoked By

- `/next-phase` command (Phase 7)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:testing:integration-test-specialist"
```

## Related Agents

- `unit-test-writer` - Writes isolated unit tests
- `e2e-test-generator` - Writes browser-based tests
- `drizzle-schema-reviewer` - Reviews database design
