# Unit Test Writer Agent

Specialized agent for writing comprehensive unit tests for the AI BotKit SaaS application using Vitest/Jest with proper mocking for Drizzle, Stripe, and Pinecone.

## Purpose

Generate complete, executable unit tests:
- Test individual functions and classes in isolation
- Mock external dependencies (database, APIs)
- Achieve high code coverage
- Follow testing best practices
- Map tests to test cases (TC-xxx)

## When to Use

This agent is invoked by `/next-phase` during Phase 7 (Testing):
- After implementation is complete (Phase 6)
- For test-driven development
- When adding tests to existing code
- For coverage improvement

## What Gets Tested

### Test Scope

| Layer | What to Test | Mocking Required |
|-------|--------------|------------------|
| **API Routes** | Request/response, validation, auth | DB, external APIs |
| **Services** | Business logic, calculations | DB, external APIs |
| **Utilities** | Helper functions, formatters | Minimal |
| **Hooks** | Custom React hooks | React context |
| **Components** | Component logic (not rendering) | Props, context |

### AI BotKit Specific Testing

| Component | Test Focus | Mocks Needed |
|-----------|------------|--------------|
| RAG Engine | Chunking, prompt building | Pinecone, LLM |
| Auth | Session validation, permissions | DB |
| Payments | Plan limits, webhook handling | Stripe |
| Database | Query builders, transactions | Drizzle |

## Test Patterns

### 1. API Route Tests

```typescript
// src/app/api/chatbots/route.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GET, POST } from './route';
import { db } from '@/lib/db';
import { getUser } from '@/lib/auth/session';

// Mock dependencies
vi.mock('@/lib/db');
vi.mock('@/lib/auth/session');

describe('GET /api/chatbots', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('returns 401 when not authenticated', async () => {
    // TC-SEC-001: Chatbot access requires authentication
    vi.mocked(getUser).mockResolvedValue(null);

    const request = new Request('http://localhost/api/chatbots');
    const response = await GET(request);

    expect(response.status).toBe(401);
    expect(await response.json()).toEqual({ error: 'Unauthorized' });
  });

  it('returns user chatbots when authenticated', async () => {
    // TC-REG-BOT-001: List user's chatbots
    vi.mocked(getUser).mockResolvedValue({ id: 1, email: 'test@test.com' });
    vi.mocked(db.query.chatbots.findMany).mockResolvedValue([
      { id: 1, name: 'Bot 1', userId: 1 },
      { id: 2, name: 'Bot 2', userId: 1 },
    ]);

    const request = new Request('http://localhost/api/chatbots');
    const response = await GET(request);

    expect(response.status).toBe(200);
    const data = await response.json();
    expect(data).toHaveLength(2);
    expect(data[0].name).toBe('Bot 1');
  });
});

describe('POST /api/chatbots', () => {
  it('creates chatbot with valid data', async () => {
    // TC-001: Create chatbot with valid data
    vi.mocked(getUser).mockResolvedValue({ id: 1, email: 'test@test.com' });
    vi.mocked(db.insert).mockReturnValue({
      values: vi.fn().mockReturnValue({
        returning: vi.fn().mockResolvedValue([{ id: 1, name: 'Test Bot' }]),
      }),
    });

    const request = new Request('http://localhost/api/chatbots', {
      method: 'POST',
      body: JSON.stringify({ name: 'Test Bot', personality: 'professional' }),
    });
    const response = await POST(request);

    expect(response.status).toBe(201);
    const data = await response.json();
    expect(data.name).toBe('Test Bot');
  });

  it('returns 400 for invalid data', async () => {
    // TC-002: Create chatbot - name validation
    vi.mocked(getUser).mockResolvedValue({ id: 1, email: 'test@test.com' });

    const request = new Request('http://localhost/api/chatbots', {
      method: 'POST',
      body: JSON.stringify({ name: '' }), // Invalid: empty name
    });
    const response = await POST(request);

    expect(response.status).toBe(400);
  });
});
```

### 2. RAG Engine Tests

```typescript
// src/lib/ai/rag-engine.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { RagEngine } from './rag-engine';
import { pinecone } from './pinecone-client';
import { generateEmbedding } from './embeddings';

vi.mock('./pinecone-client');
vi.mock('./embeddings');

describe('RagEngine', () => {
  let ragEngine: RagEngine;

  beforeEach(() => {
    ragEngine = new RagEngine();
    vi.clearAllMocks();
  });

  describe('searchContext', () => {
    it('returns relevant documents above score threshold', async () => {
      // TC-RAG-001: Context retrieval accuracy
      vi.mocked(generateEmbedding).mockResolvedValue([0.1, 0.2, 0.3]);
      vi.mocked(pinecone.query).mockResolvedValue({
        matches: [
          { id: '1', score: 0.95, metadata: { text: 'Pricing info' } },
          { id: '2', score: 0.85, metadata: { text: 'More pricing' } },
          { id: '3', score: 0.45, metadata: { text: 'Unrelated' } }, // Below threshold
        ],
      });

      const results = await ragEngine.searchContext('pricing', 'namespace-1');

      expect(results).toHaveLength(2); // Only above 0.7 threshold
      expect(results[0].metadata.text).toBe('Pricing info');
    });

    it('returns empty array when no relevant results', async () => {
      // TC-RAG-002: Fallback for unknown query
      vi.mocked(generateEmbedding).mockResolvedValue([0.1, 0.2, 0.3]);
      vi.mocked(pinecone.query).mockResolvedValue({
        matches: [
          { id: '1', score: 0.3, metadata: { text: 'Unrelated' } },
        ],
      });

      const results = await ragEngine.searchContext('weather tokyo', 'namespace-1');

      expect(results).toHaveLength(0);
    });
  });

  describe('buildPrompt', () => {
    it('includes context in system prompt', () => {
      const context = [{ text: 'Our pricing is $29/month' }];
      const prompt = ragEngine.buildPrompt('What is the price?', context, {
        personality: 'professional',
        fallbackMessage: 'I cannot help with that.',
      });

      expect(prompt.system).toContain('$29/month');
      expect(prompt.system).toContain('professional');
    });

    it('includes fallback message when no context', () => {
      const prompt = ragEngine.buildPrompt('Random question', [], {
        personality: 'friendly',
        fallbackMessage: 'Sorry, I don\'t have that information.',
      });

      expect(prompt.system).toContain('Sorry, I don\'t have that information');
    });
  });
});
```

### 3. Service Tests with Drizzle Mocking

```typescript
// src/lib/services/chatbot-service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { ChatbotService } from './chatbot-service';
import { db } from '@/lib/db';
import { checkRateLimit } from '@/lib/checkRateLimit';

vi.mock('@/lib/db');
vi.mock('@/lib/checkRateLimit');

describe('ChatbotService', () => {
  describe('create', () => {
    it('creates chatbot when under plan limit', async () => {
      // TC-001: Create chatbot with valid data
      vi.mocked(checkRateLimit).mockResolvedValue({ allowed: true, remaining: 2 });
      vi.mocked(db.insert).mockReturnValue({
        values: vi.fn().mockReturnValue({
          returning: vi.fn().mockResolvedValue([{
            id: 1,
            name: 'Test Bot',
            userId: 1,
          }]),
        }),
      });

      const result = await ChatbotService.create({
        name: 'Test Bot',
        userId: 1,
      });

      expect(result.id).toBe(1);
      expect(result.name).toBe('Test Bot');
    });

    it('throws error when plan limit exceeded', async () => {
      // TC-003: Create chatbot - plan limit exceeded
      vi.mocked(checkRateLimit).mockResolvedValue({
        allowed: false,
        remaining: 0,
        limit: 1,
      });

      await expect(
        ChatbotService.create({ name: 'Test Bot', userId: 1 })
      ).rejects.toThrow('Plan limit reached');
    });
  });
});
```

### 4. Stripe Integration Tests

```typescript
// src/lib/payments/stripe.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { handleWebhook, createCheckoutSession } from './stripe';
import Stripe from 'stripe';

vi.mock('stripe');

describe('Stripe Integration', () => {
  describe('handleWebhook', () => {
    it('updates user plan on checkout.session.completed', async () => {
      // TC-REG-STRIPE-001: Upgrade to paid plan
      const event = {
        type: 'checkout.session.completed',
        data: {
          object: {
            customer: 'cus_123',
            subscription: 'sub_123',
            metadata: { userId: '1' },
          },
        },
      };

      const result = await handleWebhook(event as Stripe.Event);

      expect(result.success).toBe(true);
      // Verify DB was updated
    });

    it('handles subscription.deleted event', async () => {
      const event = {
        type: 'customer.subscription.deleted',
        data: {
          object: {
            id: 'sub_123',
            customer: 'cus_123',
          },
        },
      };

      const result = await handleWebhook(event as Stripe.Event);

      expect(result.success).toBe(true);
      // Verify plan reverted to free
    });
  });
});
```

## Output Format

```markdown
# Unit Test Generation Report

Generated: [Date]
Test Framework: Vitest
Coverage Target: 80%

## Files Created

| File | Tests | Test Cases Covered |
|------|-------|-------------------|
| `src/app/api/chatbots/route.test.ts` | 8 | TC-001, TC-002, TC-003, TC-SEC-001 |
| `src/lib/ai/rag-engine.test.ts` | 12 | TC-RAG-001, TC-RAG-002 |
| `src/lib/services/chatbot-service.test.ts` | 6 | TC-001, TC-003 |
| `src/lib/payments/stripe.test.ts` | 5 | TC-REG-STRIPE-001 |

## Mocking Setup

### Database (Drizzle)
```typescript
vi.mock('@/lib/db');
```

### External APIs
```typescript
vi.mock('./pinecone-client');
vi.mock('stripe');
```

### Auth
```typescript
vi.mock('@/lib/auth/session');
```

## Run Tests

```bash
pnpm test:unit
pnpm test:unit --coverage
```
```

## Integration

### Invoked By

- `/next-phase` command (Phase 7)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:testing:unit-test-writer"
```

## Related Agents

- `test-case-generator` - Provides test case definitions
- `e2e-test-generator` - Writes E2E tests
- `integration-test-specialist` - Writes integration tests
- `bug-fixer` - Fixes failing tests
