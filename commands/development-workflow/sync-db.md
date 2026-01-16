# /sync-db

Database synchronization and migration workflow for the AI BotKit SaaS application using Drizzle ORM.

## Context

AI BotKit uses Drizzle ORM with PostgreSQL. Database changes require:
- Schema modifications in `src/lib/db/schema.ts`
- Migration generation
- Migration execution
- Type synchronization

This command streamlines the database workflow and ensures safe schema changes.

## Database Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AI BOTKIT DATABASE                                │
│                                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │    USERS    │    │    TEAMS    │    │  CHATBOTS   │             │
│  │  ─────────  │    │  ─────────  │    │  ─────────  │             │
│  │  id         │◄───┤  id         │    │  id         │             │
│  │  email      │    │  name       │    │  user_id ───┼──►          │
│  │  name       │    │  stripe_*   │    │  name       │             │
│  │  password   │    │  plan_*     │    │  style      │             │
│  └─────────────┘    └─────────────┘    │  messages   │             │
│         │                  │           └─────────────┘             │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │TEAM_MEMBERS │    │CONVERSATIONS│    │  DOCUMENTS  │             │
│  │  ─────────  │    │  ─────────  │    │  ─────────  │             │
│  │  user_id    │    │  chatbot_id │    │  chatbot_id │             │
│  │  team_id    │    │  session_id │    │  title      │             │
│  │  role       │    │  guest_ip   │    │  source     │             │
│  └─────────────┘    └─────────────┘    │  pinecone   │             │
│                            │           └─────────────┘             │
│                            ▼                                        │
│                     ┌─────────────┐                                 │
│                     │  MESSAGES   │                                 │
│                     │  ─────────  │                                 │
│                     │  conv_id    │                                 │
│                     │  role       │                                 │
│                     │  content    │                                 │
│                     └─────────────┘                                 │
└─────────────────────────────────────────────────────────────────────┘
```

## Usage

```bash
/sync-db [action] [options]
```

### Actions

| Action | Description |
|--------|-------------|
| `status` | Show current migration status (default) |
| `generate` | Generate migrations from schema changes |
| `migrate` | Apply pending migrations |
| `push` | Push schema directly (dev only) |
| `studio` | Open Drizzle Studio |
| `seed` | Seed database with test data |
| `backup` | Create database backup |
| `restore` | Restore from backup |

### Options

| Option | Description |
|--------|-------------|
| `--env <env>` | Target environment (dev/staging/prod) |
| `--dry-run` | Show what would happen |
| `--force` | Skip confirmation prompts |

### Examples

```bash
/sync-db status                    # Check migration status
/sync-db generate                  # Generate new migrations
/sync-db migrate                   # Apply pending migrations
/sync-db push                      # Push schema directly (dev)
/sync-db backup --env production   # Backup production DB
/sync-db studio                    # Open Drizzle Studio
```

## Workflow Phases

### Phase 1: Schema Modification

**Location:** `src/lib/db/schema.ts`

**Adding a new table:**
```typescript
// 1. Define the table
export const analytics = pgTable('aibotkit_analytics', {
  id: bigserial('id', { mode: 'number' }).primaryKey(),
  chatbotId: bigint('chatbot_id', { mode: 'number' })
    .references(() => chatbots.id),
  date: timestamp('date', { withTimezone: true }).notNull(),
  messageCount: integer('message_count').notNull().default(0),
  uniqueUsers: integer('unique_users').notNull().default(0),
  createdAt: timestamp('created_at', { withTimezone: true })
    .notNull().defaultNow(),
}, (table) => ({
  chatbotIdIdx: index('idx_analytics_chatbot_id').on(table.chatbotId),
  dateIdx: index('idx_analytics_date').on(table.date),
}));

// 2. Define relations
export const analyticsRelations = relations(analytics, ({ one }) => ({
  chatbot: one(chatbots, {
    fields: [analytics.chatbotId],
    references: [chatbots.id],
  }),
}));

// 3. Export types
export type Analytics = typeof analytics.$inferSelect;
export type NewAnalytics = typeof analytics.$inferInsert;
```

**Adding a column to existing table:**
```typescript
export const chatbots = pgTable('aibotkit_chatbots', {
  // ... existing columns ...
  analyticsEnabled: boolean('analytics_enabled').notNull().default(false), // NEW
});
```

### Phase 2: Generate Migrations

```bash
# Generate migration files
pnpm db:generate

# This creates:
# src/lib/db/migrations/XXXX_migration_name.sql
```

**Review generated SQL:**
```sql
-- Example generated migration
CREATE TABLE IF NOT EXISTS "aibotkit_analytics" (
  "id" bigserial PRIMARY KEY,
  "chatbot_id" bigint REFERENCES "aibotkit_chatbots"("id"),
  "date" timestamp with time zone NOT NULL,
  "message_count" integer NOT NULL DEFAULT 0,
  "unique_users" integer NOT NULL DEFAULT 0,
  "created_at" timestamp with time zone NOT NULL DEFAULT now()
);

CREATE INDEX "idx_analytics_chatbot_id" ON "aibotkit_analytics" ("chatbot_id");
CREATE INDEX "idx_analytics_date" ON "aibotkit_analytics" ("date");
```

### Phase 3: Apply Migrations

**Development:**
```bash
# Push directly (faster for dev)
pnpm db:push

# Or apply migrations
pnpm db:migrate
```

**Staging/Production:**
```bash
# Always use migrations for production
pnpm db:migrate

# With dry-run first
pnpm db:migrate --dry-run
```

### Phase 4: Verify Changes

```bash
# Open Drizzle Studio to verify
pnpm db:studio

# Or check via psql
psql $POSTGRES_URL -c "\d aibotkit_analytics"
```

## Migration Safety Checklist

### Safe Operations (No Downtime)

| Operation | Safe? | Notes |
|-----------|-------|-------|
| Add table | ✅ | Always safe |
| Add nullable column | ✅ | No data changes needed |
| Add column with default | ✅ | PostgreSQL handles efficiently |
| Add index | ⚠️ | Use CONCURRENTLY for large tables |
| Rename column | ⚠️ | Update application code first |

### Dangerous Operations (Require Planning)

| Operation | Risk | Mitigation |
|-----------|------|------------|
| Drop table | HIGH | Backup data first |
| Drop column | HIGH | Remove code references first |
| Change column type | HIGH | May fail if data incompatible |
| Add NOT NULL | MEDIUM | Add default or backfill first |
| Rename table | HIGH | Update all references |

### Safe Migration Pattern for NOT NULL Column

```sql
-- Step 1: Add column as nullable
ALTER TABLE chatbots ADD COLUMN analytics_enabled boolean;

-- Step 2: Backfill with default
UPDATE chatbots SET analytics_enabled = false WHERE analytics_enabled IS NULL;

-- Step 3: Add NOT NULL constraint
ALTER TABLE chatbots ALTER COLUMN analytics_enabled SET NOT NULL;

-- Step 4: Add default for new rows
ALTER TABLE chatbots ALTER COLUMN analytics_enabled SET DEFAULT false;
```

## Backup and Restore

### Create Backup

```bash
# Full backup
pg_dump $POSTGRES_URL > backup_$(date +%Y%m%d_%H%M%S).sql

# Schema only
pg_dump $POSTGRES_URL --schema-only > schema_backup.sql

# Data only
pg_dump $POSTGRES_URL --data-only > data_backup.sql

# Specific tables
pg_dump $POSTGRES_URL -t aibotkit_chatbots -t aibotkit_conversations > chatbot_backup.sql
```

### Restore from Backup

```bash
# Full restore (CAUTION: overwrites existing data)
psql $POSTGRES_URL < backup_20241215_143000.sql

# Restore to new database
createdb new_database
psql new_database < backup_20241215_143000.sql
```

## Drizzle Commands Reference

| Command | Purpose |
|---------|---------|
| `pnpm db:generate` | Generate migrations from schema |
| `pnpm db:migrate` | Apply pending migrations |
| `pnpm db:push` | Push schema directly (dev only) |
| `pnpm db:studio` | Open Drizzle Studio GUI |
| `pnpm db:seed` | Seed with test data |

## Output Format

```markdown
# Database Sync Report

## Current Status

| Property | Value |
|----------|-------|
| Environment | development |
| Database | saasdb |
| Schema Version | 0006 |
| Pending Migrations | 2 |

## Pending Migrations

| Migration | Description | Risk |
|-----------|-------------|------|
| 0007_add_analytics_table | New analytics table | LOW |
| 0008_add_chatbot_analytics_flag | New column with default | LOW |

## Schema Changes Detected

### New Tables
- `aibotkit_analytics` (5 columns, 2 indexes)

### Modified Tables
- `aibotkit_chatbots`: +1 column (analytics_enabled)

## Migration Preview

```sql
-- 0007_add_analytics_table.sql
CREATE TABLE IF NOT EXISTS "aibotkit_analytics" (
  ...
);

-- 0008_add_chatbot_analytics_flag.sql
ALTER TABLE "aibotkit_chatbots"
ADD COLUMN "analytics_enabled" boolean NOT NULL DEFAULT false;
```

## Recommendations

1. ✅ Both migrations are safe for production
2. ✅ No data loss expected
3. ⚠️ Consider adding index on analytics.date for time-range queries

## Next Steps

```bash
# Apply migrations
pnpm db:migrate

# Verify in studio
pnpm db:studio
```
```

## Troubleshooting

### Migration Conflicts

```bash
# If migrations are out of sync
pnpm db:generate --name=fix_sync

# Reset migration state (DANGEROUS - dev only)
rm -rf src/lib/db/migrations
pnpm db:generate
```

### Connection Issues

```bash
# Test connection
psql $POSTGRES_URL -c "SELECT 1"

# Check connection string format
# postgresql://user:pass@host:port/database
```

### Type Errors After Migration

```bash
# Regenerate types
pnpm db:generate

# Restart TypeScript server in IDE
```

## Related Commands

- `/deploy-saas` - Deployment with migration
- `/full-review` - Includes schema review
- `/next-phase` - Development workflow

## Success Criteria

A successful database sync:
- ✅ Schema changes reflected in schema.ts
- ✅ Migrations generated and reviewed
- ✅ Migrations applied without errors
- ✅ Types properly inferred
- ✅ Application code updated for new schema
- ✅ Existing data preserved
