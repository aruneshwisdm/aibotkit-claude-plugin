# Skills Reference

Complete reference for all AI BotKit Engineering Plugin skills.

## Overview

Skills provide domain knowledge and best practices that Claude Code uses when helping you with related tasks.

| Skill | Purpose | Use Case |
|-------|---------|----------|
| `next-phase` | 21-phase development lifecycle orchestrator | Feature development |
| `rag-development` | RAG patterns and best practices | Building AI features |
| `drizzle-patterns` | Drizzle ORM patterns | Database operations |
| `nextjs-saas` | Next.js SaaS patterns | SaaS development |

---

## next-phase

**User Invocable:** `/next-phase`

Development lifecycle orchestrator that guides you through 21 phases of spec-driven development with automatic state management and quality gates.

### Invocation

```bash
/next-phase                    # Start from Phase 0.1 or resume
/next-phase 5.6                # Jump to specific phase
/next-phase --brief "Add PDF export"  # Start with requirements brief
/next-phase --status           # Show current progress
/next-phase --reset            # Reset and start fresh
```

### Phase Overview

| Block | Phases | Purpose |
|-------|--------|---------|
| **Discovery** | 0.1-0.3 | Understand existing codebase |
| **Planning** | 0-2 | Requirements and estimation |
| **Design** | 3-4 | UI and architecture |
| **Specification** | 5-5.8 | Technical specs with quality gates |
| **Implementation** | 6-6.5 | Coding with validation |
| **Testing** | 7-8 | Write and run tests (100% pass loop) |
| **Quality** | 9-10 | Review and fix (threshold loop) |
| **Finalization** | 11-12 | Documentation and deployment |

### Quality Gates

| Gate | Phase | Pass Criteria |
|------|-------|---------------|
| Req-Spec Validation | 5.6 | 100% requirements have specifications |
| Dependency Collection | 5.8 | All dependencies available/installable |
| Test Execution | 8 | 100% tests pass (loop until achieved) |
| Code Review | 10 | No Critical/High severity issues |

### State Management

State is persisted in `_project_specs/.next-phase-state.json`:

```json
{
  "currentPhase": "5.7",
  "completedPhases": ["0.1", "0.2", "0.3", "0", "0.5", "1", "2", "3", "4", "5", "5.5", "5.6"],
  "brief": "Add PDF export for conversations",
  "gateResults": {
    "5.6": { "passed": true, "coverage": "100%" }
  },
  "artifacts": ["_project_specs/DISCOVERY_REPORT.md", "specs/SPECIFICATION.md"]
}
```

### When This Skill Applies

- Starting new feature development
- Continuing existing development
- Following spec-driven development methodology
- Ensuring quality gates are met

### Example Usage

```bash
> /next-phase --brief "Add analytics dashboard"

# Claude orchestrates through:
# 1. Codebase discovery (existing analytics code?)
# 2. Gap analysis (what exists vs what's needed)
# 3. Planning phases with reuse credits
# 4. Implementation with quality gates
# 5. Test & fix loops until 100% pass
```

---

## rag-development

Comprehensive patterns for building production-ready RAG (Retrieval-Augmented Generation) systems.

### Topics Covered

#### 1. Document Processing

**Chunking Strategies:**

| Content Type | Chunk Size | Overlap | Strategy |
|--------------|------------|---------|----------|
| Documentation | 800 tokens | 100 | Paragraph-based |
| FAQ | 200-400 tokens | 50 | Question-answer pairs |
| Long articles | 1000 tokens | 150 | Section-based |
| Code | 500 tokens | 100 | Function-based |

**Metadata Enrichment:**
```typescript
interface ChunkMetadata {
  source: string;       // Original document
  title: string;        // Document title
  chunk_index: number;  // Position in document
  total_chunks: number; // Total chunks from doc
  content_type: string; // 'text' | 'code' | 'faq'
  created_at: string;   // Indexing timestamp
}
```

#### 2. Vector Search

**Namespace Isolation:**
```typescript
// Pattern: {chatbot-slug}-user-{userId}
const namespace = `${slugify(chatbot.name)}-user-${userId}`;
```

**Score Thresholding:**
```typescript
const SCORE_THRESHOLD = 0.7;
const relevantResults = searchResults.matches
  .filter(match => match.score >= SCORE_THRESHOLD)
  .slice(0, 5);
```

#### 3. Prompt Engineering

**System Prompt Template:**
```typescript
const systemPrompt = `You are ${chatbot.personality}.

CONTEXT:
---
${context}
---

INSTRUCTIONS:
1. Answer based ONLY on the provided context
2. If context doesn't contain the answer, say: "${chatbot.fallbackMessage}"
3. Be concise and helpful`;
```

**Token Budget Allocation:**
```
Total: 8192 tokens
├── System Prompt:     500 tokens
├── Retrieved Context: 3000 tokens
├── History:           2000 tokens
├── User Query:        200 tokens
└── Response Buffer:   2492 tokens
```

#### 4. Streaming Responses

**Server-Side (SSE):**
```typescript
const stream = new ReadableStream({
  async start(controller) {
    for await (const chunk of llmClient.stream(messages)) {
      controller.enqueue(encoder.encode(`data: ${JSON.stringify({content: chunk})}\n\n`));
    }
    controller.enqueue(encoder.encode('data: [DONE]\n\n'));
    controller.close();
  },
});
```

#### 5. Error Handling

**Graceful Degradation:**
```typescript
try {
  const context = await retrieveContext(chatbotId, query);
  return await generateWithContext(query, context);
} catch (error) {
  if (error.code === 'PINECONE_UNAVAILABLE') {
    return await generateWithoutContext(query);
  }
  return chatbot.messagesTemplate.fallback;
}
```

### When This Skill Applies

- Building chat features
- Implementing document indexing
- Optimizing search relevance
- Improving response quality
- Debugging RAG issues

### Example Usage

```bash
> How should I implement chunking for PDF documents?

# Claude uses rag-development skill to provide:
# - Recommended chunk size (800-1000 tokens)
# - Overlap strategy (100-150 tokens)
# - Metadata to include
# - Code examples
```

---

## drizzle-patterns

Best practices for using Drizzle ORM with PostgreSQL.

### Topics Covered

#### 1. Schema Design

**Table Definition:**
```typescript
export const chatbots = pgTable('aibotkit_chatbots', {
  id: bigserial('id', { mode: 'number' }).primaryKey(),
  userId: integer('user_id').references(() => users.id).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  style: jsonb('style').$type<ChatbotStyle>(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow(),
}, (table) => ({
  userIdIdx: index('idx_chatbots_user_id').on(table.userId),
}));
```

**Type-Safe JSONB:**
```typescript
type ChatbotStyle = {
  primaryColor: string;
  position: 'left' | 'right';
};

style: jsonb('style').$type<ChatbotStyle>(),
```

#### 2. Relationships

**One-to-Many:**
```typescript
export const chatbotsRelations = relations(chatbots, ({ one, many }) => ({
  user: one(users, {
    fields: [chatbots.userId],
    references: [users.id],
  }),
  documents: many(documents),
}));
```

#### 3. Query Patterns

**Relational Queries:**
```typescript
const chatbots = await db.query.chatbots.findMany({
  where: eq(chatbots.userId, userId),
  with: {
    documents: true,
    conversations: {
      limit: 10,
      orderBy: [desc(conversations.createdAt)],
    },
  },
});
```

**Pagination:**
```typescript
// Cursor-based (recommended for large datasets)
const results = await db
  .select()
  .from(chatbots)
  .where(cursor ? sql`${chatbots.id} < ${cursor}` : undefined)
  .orderBy(desc(chatbots.id))
  .limit(limit + 1);

const hasMore = results.length > limit;
const nextCursor = hasMore ? results[results.length - 2].id : null;
```

#### 4. Mutations

**Insert with Returning:**
```typescript
const [chatbot] = await db
  .insert(chatbots)
  .values({ userId, name })
  .returning();
```

**Upsert:**
```typescript
await db
  .insert(chatbots)
  .values({ id: existingId, name: 'Updated' })
  .onConflictDoUpdate({
    target: chatbots.id,
    set: { name: 'Updated', updatedAt: new Date() },
  });
```

#### 5. Transactions

```typescript
const result = await db.transaction(async (tx) => {
  const [chatbot] = await tx.insert(chatbots).values({...}).returning();
  await tx.insert(conversations).values({ chatbotId: chatbot.id });
  return chatbot;
});
```

#### 6. Performance

**Avoiding N+1:**
```typescript
// Bad: N+1 queries
for (const chatbot of chatbots) {
  const docs = await db.select().from(documents).where(eq(documents.chatbotId, chatbot.id));
}

// Good: Single query with relation
const chatbotsWithDocs = await db.query.chatbots.findMany({
  with: { documents: true },
});
```

### When This Skill Applies

- Creating database schemas
- Writing queries
- Optimizing performance
- Running migrations
- Debugging database issues

### Example Usage

```bash
> How do I implement pagination for chatbots?

# Claude uses drizzle-patterns skill to provide:
# - Cursor-based pagination pattern
# - Offset-based alternative
# - Performance considerations
# - Type-safe implementation
```

---

## nextjs-saas

Best practices for building SaaS applications with Next.js 16 App Router.

### Topics Covered

#### 1. Project Structure

```
src/
├── app/                      # App Router
│   ├── (auth)/               # Auth route group
│   │   ├── login/
│   │   └── layout.tsx        # Minimal auth layout
│   ├── (dashboard)/          # Dashboard route group
│   │   ├── dashboard/
│   │   └── layout.tsx        # Dashboard layout
│   ├── api/                  # API routes
│   └── layout.tsx            # Root layout
├── components/
│   ├── ui/                   # Base UI (shadcn)
│   └── features/             # Feature components
├── lib/
│   ├── auth/                 # Authentication
│   ├── db/                   # Database
│   └── payments/             # Stripe
└── hooks/                    # Custom hooks
```

#### 2. Server vs Client Components

**Default to Server:**
```tsx
// Server Component - can fetch data, access DB
async function ChatbotList() {
  const chatbots = await db.query.chatbots.findMany();
  return <ul>{chatbots.map(c => <ChatbotCard key={c.id} chatbot={c} />)}</ul>;
}
```

**Client for Interactivity:**
```tsx
'use client';

function ChatbotCard({ chatbot }: { chatbot: Chatbot }) {
  const [isEditing, setIsEditing] = useState(false);
  return <div onClick={() => setIsEditing(true)}>...</div>;
}
```

#### 3. Data Fetching

**Parallel Fetching:**
```tsx
async function Dashboard() {
  const [user, chatbots, analytics] = await Promise.all([
    getUser(),
    getChatbots(),
    getAnalytics(),
  ]);
  return <div>...</div>;
}
```

**Request Deduplication:**
```tsx
import { cache } from 'react';

export const getUser = cache(async () => {
  const session = await getSession();
  return db.query.users.findFirst({ where: eq(users.id, session.userId) });
});
```

#### 4. Authentication

**Session Management:**
```tsx
export async function createSession(userId: number) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('7d')
    .sign(secretKey);

  (await cookies()).set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
  });
}
```

**Middleware Protection:**
```tsx
export async function middleware(request: NextRequest) {
  const session = request.cookies.get('session')?.value;

  if (protectedRoutes.some(r => pathname.startsWith(r)) && !session) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}
```

#### 5. API Routes

**Route Handler Pattern:**
```tsx
export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  const body = await request.json();
  const result = schema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ error: result.error }, { status: 400 });
  }

  const chatbot = await createChatbot(result.data, user.id);
  return NextResponse.json(chatbot, { status: 201 });
}
```

#### 6. Stripe Integration

**Checkout Session:**
```tsx
const session = await stripe.checkout.sessions.create({
  mode: 'subscription',
  line_items: [{ price: priceId, quantity: 1 }],
  success_url: `${BASE_URL}/dashboard?success=true`,
  cancel_url: `${BASE_URL}/pricing`,
});
```

**Webhook Handler:**
```tsx
export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  const event = stripe.webhooks.constructEvent(body, signature, WEBHOOK_SECRET);

  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutCompleted(event.data.object);
      break;
  }
}
```

#### 7. Performance

**Dynamic Imports:**
```tsx
const Chart = dynamic(() => import('@/components/Chart'), {
  loading: () => <Skeleton />,
  ssr: false,
});
```

**Image Optimization:**
```tsx
<Image
  src={user.avatarUrl}
  alt={user.name}
  width={40}
  height={40}
  priority={false}
/>
```

### When This Skill Applies

- Building new pages
- Implementing authentication
- Creating API endpoints
- Adding Stripe integration
- Optimizing performance

### Example Usage

```bash
> How should I structure the settings page?

# Claude uses nextjs-saas skill to provide:
# - Server component for data fetching
# - Client component for forms
# - Proper layout structure
# - Loading/error states
```

---

## Skill Activation

Skills are automatically activated when you work on related code:

| Task | Skills Used |
|------|-------------|
| Feature development | `next-phase` |
| Building chat features | `rag-development`, `nextjs-saas` |
| Database changes | `drizzle-patterns` |
| API development | `nextjs-saas`, `drizzle-patterns` |
| Payment integration | `nextjs-saas` |

## See Also

- [Commands Reference](COMMANDS.md)
- [Agents Reference](AGENTS.md)
- [Workflow Examples](WORKFLOWS.md)
