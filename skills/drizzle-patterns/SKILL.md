---
name: drizzle-patterns
description: Best practices for Drizzle ORM with PostgreSQL
---

# Drizzle ORM Patterns

Best practices for using Drizzle ORM with PostgreSQL in the AI BotKit SaaS application.

## Schema Design

### Table Definition

```typescript
import {
  pgTable,
  serial,
  bigserial,
  varchar,
  text,
  timestamp,
  integer,
  boolean,
  jsonb,
  index,
  uniqueIndex,
} from 'drizzle-orm/pg-core';

// Use bigserial for tables that may grow large
export const chatbots = pgTable('aibotkit_chatbots', {
  // Primary key - use bigserial for scale
  id: bigserial('id', { mode: 'number' }).primaryKey(),

  // Foreign keys - reference parent tables
  userId: integer('user_id')
    .references(() => users.id)
    .notNull(),

  // Required fields with constraints
  name: varchar('name', { length: 255 }).notNull(),

  // Optional fields (nullable by default)
  description: text('description'),

  // Boolean with default
  active: boolean('active').notNull().default(false),

  // JSON columns for flexible data
  style: jsonb('style').$type<ChatbotStyle>(),
  messagesTemplate: jsonb('messages_template').$type<MessagesTemplate>(),

  // Timestamps
  createdAt: timestamp('created_at', { withTimezone: true })
    .notNull()
    .defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true })
    .notNull()
    .defaultNow(),
}, (table) => ({
  // Indexes for common queries
  userIdIdx: index('idx_chatbots_user_id').on(table.userId),
  activeIdx: index('idx_chatbots_active').on(table.active),
}));
```

### Type Safety with JSONB

```typescript
// Define types for JSONB columns
type ChatbotStyle = {
  primaryColor: string;
  secondaryColor: string;
  position: 'left' | 'right';
  buttonStyle: 'round' | 'square';
  fontFamily?: string;
};

type MessagesTemplate = {
  greeting: string;
  fallback: string;
  tone: 'professional' | 'friendly' | 'casual';
  personality: string;
  namespace?: string;
};

// Use $type<T>() for type inference
style: jsonb('style').$type<ChatbotStyle>(),
```

### Relationships

```typescript
import { relations } from 'drizzle-orm';

// One-to-Many: User has many Chatbots
export const usersRelations = relations(users, ({ many }) => ({
  chatbots: many(chatbots),
  teamMembers: many(teamMembers),
}));

export const chatbotsRelations = relations(chatbots, ({ one, many }) => ({
  // Many-to-One: Chatbot belongs to User
  user: one(users, {
    fields: [chatbots.userId],
    references: [users.id],
  }),
  // One-to-Many: Chatbot has many Documents
  documents: many(documents),
  conversations: many(conversations),
}));

// Many-to-Many through join table
export const teamMembersRelations = relations(teamMembers, ({ one }) => ({
  user: one(users, {
    fields: [teamMembers.userId],
    references: [users.id],
  }),
  team: one(teams, {
    fields: [teamMembers.teamId],
    references: [teams.id],
  }),
}));
```

## Query Patterns

### Basic Queries

```typescript
import { db } from '@/lib/db/drizzle';
import { chatbots, documents } from '@/lib/db/schema';
import { eq, and, or, desc, asc, like, isNull, sql } from 'drizzle-orm';

// Select all
const allChatbots = await db.select().from(chatbots);

// Select with condition
const userChatbots = await db
  .select()
  .from(chatbots)
  .where(eq(chatbots.userId, userId));

// Select specific columns
const chatbotNames = await db
  .select({
    id: chatbots.id,
    name: chatbots.name,
  })
  .from(chatbots)
  .where(eq(chatbots.userId, userId));

// Multiple conditions
const activeChatbots = await db
  .select()
  .from(chatbots)
  .where(
    and(
      eq(chatbots.userId, userId),
      eq(chatbots.active, true),
      eq(chatbots.deleteFlag, false)
    )
  );
```

### Relational Queries

```typescript
// Query with relations (recommended)
const chatbotsWithDocs = await db.query.chatbots.findMany({
  where: eq(chatbots.userId, userId),
  with: {
    documents: true,
    conversations: {
      limit: 10,
      orderBy: [desc(conversations.createdAt)],
    },
  },
});

// Find one with relations
const chatbot = await db.query.chatbots.findFirst({
  where: eq(chatbots.id, chatbotId),
  with: {
    user: {
      columns: {
        id: true,
        name: true,
        email: true,
      },
    },
    documents: {
      where: eq(documents.status, 'completed'),
    },
  },
});
```

### Joins

```typescript
// Inner join
const chatbotsWithUsers = await db
  .select({
    chatbot: chatbots,
    user: users,
  })
  .from(chatbots)
  .innerJoin(users, eq(chatbots.userId, users.id));

// Left join
const usersWithChatbots = await db
  .select({
    user: users,
    chatbotCount: sql<number>`count(${chatbots.id})`,
  })
  .from(users)
  .leftJoin(chatbots, eq(users.id, chatbots.userId))
  .groupBy(users.id);
```

### Pagination

```typescript
// Offset-based pagination
async function getChatbotsPaginated(
  userId: number,
  page: number = 1,
  pageSize: number = 10
) {
  const offset = (page - 1) * pageSize;

  const [results, countResult] = await Promise.all([
    db
      .select()
      .from(chatbots)
      .where(eq(chatbots.userId, userId))
      .orderBy(desc(chatbots.createdAt))
      .limit(pageSize)
      .offset(offset),
    db
      .select({ count: sql<number>`count(*)` })
      .from(chatbots)
      .where(eq(chatbots.userId, userId)),
  ]);

  return {
    data: results,
    pagination: {
      page,
      pageSize,
      total: Number(countResult[0].count),
      totalPages: Math.ceil(Number(countResult[0].count) / pageSize),
    },
  };
}

// Cursor-based pagination (better for large datasets)
async function getChatbotsCursor(
  userId: number,
  cursor?: number,
  limit: number = 10
) {
  const results = await db
    .select()
    .from(chatbots)
    .where(
      cursor
        ? and(eq(chatbots.userId, userId), sql`${chatbots.id} < ${cursor}`)
        : eq(chatbots.userId, userId)
    )
    .orderBy(desc(chatbots.id))
    .limit(limit + 1); // Fetch one extra to check hasMore

  const hasMore = results.length > limit;
  const data = hasMore ? results.slice(0, -1) : results;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  return { data, nextCursor, hasMore };
}
```

## Mutations

### Insert

```typescript
// Single insert
const newChatbot = await db
  .insert(chatbots)
  .values({
    userId,
    name: 'My Chatbot',
    active: true,
    style: { primaryColor: '#008858' },
  })
  .returning();

// Insert with returning specific columns
const [{ id }] = await db
  .insert(chatbots)
  .values({ userId, name })
  .returning({ id: chatbots.id });

// Bulk insert
await db.insert(documents).values([
  { chatbotId, title: 'Doc 1', sourceType: 'text' },
  { chatbotId, title: 'Doc 2', sourceType: 'url' },
]);

// Upsert (insert or update)
await db
  .insert(chatbots)
  .values({ id: existingId, userId, name: 'Updated Name' })
  .onConflictDoUpdate({
    target: chatbots.id,
    set: { name: 'Updated Name', updatedAt: new Date() },
  });
```

### Update

```typescript
// Update with condition
await db
  .update(chatbots)
  .set({
    active: false,
    updatedAt: new Date(),
  })
  .where(eq(chatbots.id, chatbotId));

// Update with returning
const updated = await db
  .update(chatbots)
  .set({ name: 'New Name' })
  .where(eq(chatbots.id, chatbotId))
  .returning();

// Conditional update
await db
  .update(chatbots)
  .set({ active: true })
  .where(
    and(
      eq(chatbots.userId, userId),
      eq(chatbots.active, false)
    )
  );
```

### Delete

```typescript
// Hard delete
await db.delete(chatbots).where(eq(chatbots.id, chatbotId));

// Soft delete (recommended for important data)
await db
  .update(chatbots)
  .set({ deleteFlag: true, updatedAt: new Date() })
  .where(eq(chatbots.id, chatbotId));

// Cascade delete (use foreign key ON DELETE CASCADE)
// Or manual cascade:
await db.transaction(async (tx) => {
  await tx.delete(messages).where(
    sql`${messages.conversationId} IN (
      SELECT id FROM ${conversations} WHERE chatbot_id = ${chatbotId}
    )`
  );
  await tx.delete(conversations).where(eq(conversations.chatbotId, chatbotId));
  await tx.delete(documents).where(eq(documents.chatbotId, chatbotId));
  await tx.delete(chatbots).where(eq(chatbots.id, chatbotId));
});
```

## Transactions

```typescript
// Transaction with rollback on error
const result = await db.transaction(async (tx) => {
  // Create chatbot
  const [chatbot] = await tx
    .insert(chatbots)
    .values({ userId, name })
    .returning();

  // Create default conversation
  const [conversation] = await tx
    .insert(conversations)
    .values({
      chatbotId: chatbot.id,
      sessionId: generateSessionId(),
    })
    .returning();

  // Create welcome message
  await tx.insert(messages).values({
    conversationId: conversation.id,
    role: 'assistant',
    content: 'Hello! How can I help you?',
  });

  return chatbot;
});
```

## Migrations

### Generate and Apply

```bash
# Generate migration from schema changes
pnpm db:generate

# Apply migrations
pnpm db:migrate

# Push schema directly (development only)
pnpm db:push

# View database in GUI
pnpm db:studio
```

### Safe Migration Patterns

```typescript
// migrations/0007_add_analytics.ts
import { sql } from 'drizzle-orm';

export async function up(db: Database) {
  // Create new table (safe)
  await db.execute(sql`
    CREATE TABLE IF NOT EXISTS aibotkit_analytics (
      id BIGSERIAL PRIMARY KEY,
      chatbot_id BIGINT REFERENCES aibotkit_chatbots(id),
      date DATE NOT NULL,
      message_count INTEGER NOT NULL DEFAULT 0,
      created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
    )
  `);

  // Add nullable column (safe)
  await db.execute(sql`
    ALTER TABLE aibotkit_chatbots
    ADD COLUMN IF NOT EXISTS analytics_enabled BOOLEAN
  `);

  // Create index concurrently (no lock)
  await db.execute(sql`
    CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_analytics_date
    ON aibotkit_analytics (date)
  `);
}

export async function down(db: Database) {
  await db.execute(sql`DROP TABLE IF EXISTS aibotkit_analytics`);
  await db.execute(sql`
    ALTER TABLE aibotkit_chatbots
    DROP COLUMN IF EXISTS analytics_enabled
  `);
}
```

## Performance Patterns

### Avoiding N+1 Queries

```typescript
// Bad: N+1 queries
const chatbots = await db.select().from(chatbotsTable);
for (const chatbot of chatbots) {
  const docs = await db
    .select()
    .from(documents)
    .where(eq(documents.chatbotId, chatbot.id));
  // This runs N additional queries!
}

// Good: Single query with relation
const chatbotsWithDocs = await db.query.chatbots.findMany({
  with: { documents: true },
});

// Good: Manual join
const chatbotsWithDocs = await db
  .select()
  .from(chatbots)
  .leftJoin(documents, eq(chatbots.id, documents.chatbotId));
```

### Index Strategy

```typescript
// Index on foreign keys
userIdIdx: index('idx_chatbots_user_id').on(table.userId),

// Index on frequently filtered columns
activeIdx: index('idx_chatbots_active').on(table.active),
statusIdx: index('idx_documents_status').on(table.status),

// Composite index for common query patterns
chatbotCreatedIdx: index('idx_conversations_chatbot_created')
  .on(table.chatbotId, table.createdAt),

// Unique index for constraints
sessionIdIdx: uniqueIndex('idx_conversations_session_id').on(table.sessionId),

// GIN index for JSONB queries
metaIdx: index('idx_activity_logs_meta').using('gin', table.meta),
```

### Query Optimization

```typescript
// Select only needed columns
const chatbotIds = await db
  .select({ id: chatbots.id })
  .from(chatbots)
  .where(eq(chatbots.userId, userId));

// Use COUNT for existence check
const exists = await db
  .select({ count: sql<number>`count(*)` })
  .from(chatbots)
  .where(eq(chatbots.id, chatbotId));

// Use EXISTS for better performance
const hasDocuments = await db.execute(sql`
  SELECT EXISTS(
    SELECT 1 FROM ${documents}
    WHERE chatbot_id = ${chatbotId}
  )
`);
```

## Type Exports

```typescript
// Infer types from schema
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;

export type Chatbot = typeof chatbots.$inferSelect;
export type NewChatbot = typeof chatbots.$inferInsert;

// Use in application
async function createChatbot(data: NewChatbot): Promise<Chatbot> {
  const [chatbot] = await db.insert(chatbots).values(data).returning();
  return chatbot;
}
```

## Drizzle Kit Configuration

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/lib/db/schema.ts',
  out: './src/lib/db/migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.POSTGRES_URL!,
  },
} satisfies Config;
```
