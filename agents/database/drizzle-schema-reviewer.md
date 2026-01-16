# Drizzle Schema Reviewer

Reviews Drizzle ORM schema definitions, migrations, and database queries for correctness and best practices.

## Purpose

Ensure the AI BotKit database layer follows best practices for:
- Schema design and normalization
- Proper use of Drizzle types
- Index optimization
- Migration safety
- Query efficiency
- Relationship definitions

## When to Use

- During `/full-review` for SaaS component
- When adding new tables or columns
- Before running migrations in production
- When optimizing slow queries

## Architecture Context

AI BotKit Database:
- **ORM:** Drizzle with PostgreSQL
- **Schema:** `src/lib/db/schema.ts`
- **Migrations:** `src/lib/db/migrations/`
- **Config:** `drizzle.config.ts`

## What Gets Analyzed

### 1. Schema Design

**Check for:**
- Proper column types
- NOT NULL constraints where appropriate
- Default values
- Unique constraints
- Foreign key relationships

**Good Pattern:**
```typescript
export const chatbots = pgTable('aibotkit_chatbots', {
  id: bigserial('id', { mode: 'number' }).primaryKey(),
  userId: integer('user_id').references(() => users.id).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  active: boolean('active').notNull().default(false),
  style: jsonb('style'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  userIdIdx: index('idx_chatbots_user_id').on(table.userId),
}));
```

**Anti-Pattern:**
```typescript
export const chatbots = pgTable('chatbots', {
  id: serial('id').primaryKey(), // Should be bigserial for scale
  userId: integer('user_id'), // Missing reference, missing notNull
  name: text('name'), // No length limit
  // Missing timestamps
  // Missing indexes
});
```

### 2. Index Optimization

**Check for:**
- Indexes on foreign keys
- Indexes on frequently queried columns
- Composite indexes where appropriate
- No redundant indexes

**Good Pattern:**
```typescript
export const conversations = pgTable('aibotkit_conversations', {
  id: bigserial('id', { mode: 'number' }).primaryKey(),
  chatbotId: bigint('chatbot_id', { mode: 'number' }),
  sessionId: varchar('session_id', { length: 100 }).notNull().unique(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  chatbotIdIdx: index('idx_conversations_chatbot_id').on(table.chatbotId),
  sessionIdIdx: index('idx_conversations_session_id').on(table.sessionId),
  // Composite index for common query pattern
  chatbotCreatedIdx: index('idx_conversations_chatbot_created')
    .on(table.chatbotId, table.createdAt),
}));
```

### 3. Relationship Definitions

**Check for:**
- Proper relations defined
- Correct cardinality (one-to-one, one-to-many, many-to-many)
- Cascading behavior on delete/update

**Good Pattern:**
```typescript
export const chatbotsRelations = relations(chatbots, ({ one, many }) => ({
  user: one(users, {
    fields: [chatbots.userId],
    references: [users.id],
  }),
  documents: many(documents),
  conversations: many(conversations),
}));

export const messagesRelations = relations(messages, ({ one }) => ({
  conversation: one(conversations, {
    fields: [messages.conversationId],
    references: [conversations.id],
  }),
}));
```

### 4. JSONB Column Usage

**Check for:**
- Appropriate use of JSONB vs separate columns
- Consistent structure in JSONB columns
- Type definitions for JSONB data

**Good Pattern:**
```typescript
// Type definition for JSONB data
type ChatbotStyle = {
  primaryColor: string;
  position: 'left' | 'right';
  buttonStyle: 'round' | 'square';
};

export const chatbots = pgTable('aibotkit_chatbots', {
  // ...
  style: jsonb('style').$type<ChatbotStyle>(),
});
```

**Anti-Pattern:**
```typescript
// No type definition - can store anything
style: jsonb('style'), // What's the structure?
```

### 5. Migration Safety

**Check for:**
- Non-breaking migrations
- Data preservation
- Rollback capability
- Index creation strategy

**Good Pattern:**
```typescript
// Adding nullable column (non-breaking)
await db.execute(sql`
  ALTER TABLE aibotkit_chatbots
  ADD COLUMN IF NOT EXISTS analytics_enabled BOOLEAN DEFAULT false
`);

// Creating index concurrently (no lock)
await db.execute(sql`
  CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_chatbots_analytics
  ON aibotkit_chatbots (analytics_enabled)
`);
```

**Anti-Pattern:**
```typescript
// Dropping column without backup
await db.execute(sql`
  ALTER TABLE aibotkit_chatbots DROP COLUMN style
`);
```

### 6. Query Patterns

**Check for:**
- N+1 query problems
- Proper use of joins vs separate queries
- Select only needed columns
- Proper pagination

**Good Pattern:**
```typescript
// Select specific columns, use join
const chatbotsWithDocs = await db.query.chatbots.findMany({
  columns: {
    id: true,
    name: true,
    active: true,
  },
  with: {
    documents: {
      columns: { id: true, title: true },
    },
  },
  where: eq(chatbots.userId, userId),
  limit: 10,
  offset: 0,
});
```

**Anti-Pattern:**
```typescript
// N+1 problem
const chatbots = await db.select().from(chatbotsTable);
for (const chatbot of chatbots) {
  const docs = await db.select().from(documentsTable)
    .where(eq(documentsTable.chatbotId, chatbot.id));
  // Process...
}
```

## Output Format

```markdown
## Drizzle Schema Review

### Summary
| Category | Issues | Severity |
|----------|--------|----------|
| Schema Design | X | High/Medium/Low |
| Indexes | X | High/Medium/Low |
| Relations | X | High/Medium/Low |
| JSONB Usage | X | High/Medium/Low |
| Migrations | X | High/Medium/Low |
| Queries | X | High/Medium/Low |

### Issues Found

#### HIGH: Missing Index on Frequently Queried Column
**File:** src/lib/db/schema.ts:165
**Table:** aibotkit_messages
**Issue:** No index on conversation_id despite frequent lookups

**Current:**
```typescript
// No index defined for conversation_id
```

**Recommended:**
```typescript
}, (table) => ({
  conversationIdIdx: index('idx_messages_conversation_id')
    .on(table.conversationId),
}));
```

**Impact:** Slow message retrieval for conversations
**Estimated Fix Time:** 15 minutes

---

### Schema Health

| Table | Columns | Indexes | Relations | Status |
|-------|---------|---------|-----------|--------|
| users | X | X | X | Status |
| chatbots | X | X | X | Status |
| conversations | X | X | X | Status |
| messages | X | X | X | Status |
| documents | X | X | X | Status |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (SaaS component)
- Can be invoked directly for database-focused review

## Related Agents

- `nextjs-standards-reviewer` - General Next.js patterns
- `saas-security-auditor` - Security review
- `rag-engine-reviewer` - RAG data storage patterns
