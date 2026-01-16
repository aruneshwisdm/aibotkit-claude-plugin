# Commands Reference

Complete reference for all AI BotKit Engineering Plugin commands.

## Overview

| Command | Description | Use Case |
|---------|-------------|----------|
| `/full-review` | Multi-agent code review | Pre-PR validation |
| `/next-phase` | Development lifecycle | Feature development |
| `/deploy-saas` | Deployment workflow | Production releases |
| `/test-rag` | RAG engine testing | AI feature validation |
| `/sync-db` | Database synchronization | Schema migrations |

---

## /full-review

Orchestrates 10 specialized agents to perform comprehensive code review across the entire AI BotKit codebase.

### Usage

```bash
# Full review of all components
/full-review

# Review specific component
/full-review --component saas
/full-review --component wordpress

# Review specific files or directories
/full-review --files src/lib/ai/rag-engine.ts
/full-review --files src/lib/ai/

# Focus on specific areas
/full-review --focus security
/full-review --focus performance
/full-review --focus accessibility
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--component` | Target SaaS or WordPress | `--component saas` |
| `--files` | Specific files/directories | `--files src/lib/ai/` |
| `--focus` | Focus area | `--focus security` |
| `--severity` | Minimum severity to report | `--severity high` |
| `--output` | Output format | `--output markdown` |

### Agents Invoked

1. **nextjs-standards-reviewer** - Next.js 16 patterns
2. **rag-engine-reviewer** - RAG implementation
3. **drizzle-schema-reviewer** - Database schema
4. **saas-security-auditor** - SaaS security
5. **stripe-integration-reviewer** - Payment security
6. **wordpress-standards-reviewer** - WPCS compliance
7. **wordpress-security-auditor** - WordPress security
8. **api-integration-reviewer** - API consistency
9. **accessibility-guardian** - WCAG compliance
10. **architecture-reviewer** - Architecture principles

### Output Format

```markdown
## Code Review Summary

### Overall Score: 85/100

| Category | Score | Issues |
|----------|-------|--------|
| Next.js Standards | 90/100 | 2 |
| RAG Engine | 85/100 | 3 |
| Database | 88/100 | 1 |
| Security | 82/100 | 4 |
| Accessibility | 80/100 | 5 |

### Critical Issues (0)
None found

### High Priority Issues (3)
1. [Issue details...]
2. [Issue details...]
3. [Issue details...]

### Recommendations
1. [Recommendation...]
```

### Examples

**Pre-PR Review:**
```bash
# Run full review before creating PR
/full-review

# After fixing issues
git add . && git commit -m "fix: address review findings"
```

**Security Audit:**
```bash
# Focus on security issues only
/full-review --focus security --severity high
```

**Component-Specific:**
```bash
# Review only WordPress plugin before release
/full-review --component wordpress
```

---

## /next-phase

Guides development through a 12-phase lifecycle from discovery to deployment.

### Usage

```bash
# Start from beginning (Phase 1: Discovery)
/next-phase

# Continue from specific phase
/next-phase 4
/next-phase 7

# Provide requirements document
/next-phase --brief requirements.md
/next-phase --brief "Add user authentication with OAuth"

# Skip to implementation (after planning)
/next-phase --skip-to implementation
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `[phase]` | Phase number (1-12) | `/next-phase 4` |
| `--brief` | Requirements file or text | `--brief requirements.md` |
| `--skip-to` | Jump to phase group | `--skip-to implementation` |
| `--status` | Show current progress | `--status` |

### Phases

#### Discovery Phases (1-3)

| Phase | Name | Purpose |
|-------|------|---------|
| 1 | Codebase Discovery | Index and understand existing code |
| 2 | Documentation Review | Identify documentation gaps |
| 3 | Gap Analysis | Compare current state vs requirements |

#### Planning Phases (4-6)

| Phase | Name | Purpose |
|-------|------|---------|
| 4 | Requirements | Document detailed specifications |
| 5 | Architecture | Design system extensions |
| 6 | Specification | Write technical specs |

#### Implementation Phases (7-12)

| Phase | Name | Purpose |
|-------|------|---------|
| 7 | Implementation | Build the feature |
| 8 | Testing | Write and run tests |
| 9 | Code Review | Run `/full-review` |
| 10 | Documentation | Update documentation |
| 11 | Pre-Deployment | Final validation checks |
| 12 | Deployment | Deploy to production |

### Phase Outputs

Each phase produces specific artifacts:

```
Phase 1 → codebase-index.md
Phase 2 → documentation-gaps.md
Phase 3 → gap-analysis.md
Phase 4 → requirements-spec.md
Phase 5 → architecture-design.md
Phase 6 → technical-spec.md
Phase 7 → implementation (code)
Phase 8 → test-results.md
Phase 9 → review-report.md
Phase 10 → updated documentation
Phase 11 → pre-deploy-checklist.md
Phase 12 → deployment-report.md
```

### Examples

**New Feature Development:**
```bash
# Start discovery
/next-phase
# Output: Codebase analysis complete. Proceed to Phase 2?

/next-phase 2
# Output: Documentation gaps identified. Proceed to Phase 3?

# Continue through all phases...
```

**Quick Implementation:**
```bash
# Skip discovery if you know the codebase
/next-phase --brief "Add chatbot templates feature" --skip-to implementation

# This starts at Phase 7 with your requirements
```

---

## /deploy-saas

Validates and executes deployment workflow for the SaaS application.

### Usage

```bash
# Full deployment workflow
/deploy-saas

# Dry run (validation only)
/deploy-saas --dry-run

# Deploy to specific environment
/deploy-saas --env staging
/deploy-saas --env production

# Skip specific checks
/deploy-saas --skip-tests
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--env` | Target environment | `--env production` |
| `--dry-run` | Validate without deploying | `--dry-run` |
| `--skip-tests` | Skip test execution | `--skip-tests` |
| `--skip-build` | Skip build step | `--skip-build` |
| `--force` | Force deployment | `--force` |

### Deployment Stages

#### Stage 1: Pre-flight Checks

```markdown
## Pre-flight Checklist

- [ ] Environment variables configured
- [ ] Database migrations pending: 0
- [ ] All tests passing
- [ ] Build succeeds
- [ ] No critical security issues
- [ ] Dependencies up to date
```

#### Stage 2: Build & Test

```bash
# Automated steps:
pnpm install
pnpm build
pnpm test
pnpm lint
```

#### Stage 3: Database Migration

```bash
# If pending migrations exist:
pnpm db:migrate
```

#### Stage 4: Deployment

```bash
# Vercel deployment:
vercel --prod
```

#### Stage 5: Post-deployment Validation

```markdown
## Health Checks

- [ ] Homepage loads (200 OK)
- [ ] API endpoints responding
- [ ] Authentication working
- [ ] Database connected
- [ ] Stripe webhooks active
- [ ] Pinecone connected
```

### Examples

**Production Deployment:**
```bash
# Validate first
/deploy-saas --dry-run

# If all checks pass
/deploy-saas --env production
```

**Staging Deployment:**
```bash
/deploy-saas --env staging
```

---

## /test-rag

Comprehensive testing workflow for the RAG (Retrieval-Augmented Generation) engine.

### Usage

```bash
# Full RAG test suite
/test-rag

# Test specific component
/test-rag --component embeddings
/test-rag --component search
/test-rag --component generation

# Test with specific chatbot
/test-rag --chatbot-id 123

# Benchmark mode
/test-rag --benchmark
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--component` | Test specific component | `--component search` |
| `--chatbot-id` | Test specific chatbot | `--chatbot-id 123` |
| `--benchmark` | Run performance benchmarks | `--benchmark` |
| `--verbose` | Detailed output | `--verbose` |

### Test Categories

#### 1. Embedding Tests

```markdown
## Embedding Tests

- [ ] Text embedding generation
- [ ] Embedding dimensions correct (1536)
- [ ] Batch embedding performance
- [ ] Error handling for invalid input
```

#### 2. Vector Search Tests

```markdown
## Vector Search Tests

- [ ] Namespace isolation
- [ ] Similarity search accuracy
- [ ] Score thresholding
- [ ] Top-K retrieval
- [ ] Metadata filtering
```

#### 3. Prompt Construction Tests

```markdown
## Prompt Construction Tests

- [ ] System prompt generation
- [ ] Context injection
- [ ] Token budget management
- [ ] History truncation
- [ ] Template variable replacement
```

#### 4. Generation Tests

```markdown
## Generation Tests

- [ ] Streaming response
- [ ] Non-streaming response
- [ ] Error handling
- [ ] Rate limiting
- [ ] Fallback responses
```

### Output

```markdown
## RAG Test Results

### Summary
| Component | Tests | Passed | Failed |
|-----------|-------|--------|--------|
| Embeddings | 5 | 5 | 0 |
| Search | 8 | 7 | 1 |
| Prompts | 6 | 6 | 0 |
| Generation | 7 | 7 | 0 |

### Failed Tests
1. **Search: Score threshold filtering**
   - Expected: 3 results above 0.7
   - Actual: 5 results (threshold not applied)
   - File: src/lib/ai/rag-engine.ts:145

### Recommendations
1. Fix score threshold in searchContext function
```

---

## /sync-db

Database schema synchronization and migration workflow using Drizzle ORM.

### Usage

```bash
# Full sync (generate + migrate)
/sync-db

# Generate migration only
/sync-db --generate-only

# Push schema directly (dev only)
/sync-db --push

# Open Drizzle Studio
/sync-db --studio

# Show migration status
/sync-db --status
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--generate-only` | Generate without applying | `--generate-only` |
| `--push` | Push schema (dev only) | `--push` |
| `--studio` | Open Drizzle Studio | `--studio` |
| `--status` | Show migration status | `--status` |
| `--rollback` | Rollback last migration | `--rollback` |

### Workflow Steps

#### 1. Schema Validation

```markdown
## Schema Validation

- [ ] Schema file syntax valid
- [ ] All relations defined correctly
- [ ] Indexes defined for foreign keys
- [ ] No circular dependencies
```

#### 2. Migration Generation

```bash
# Generates migration in src/lib/db/migrations/
pnpm db:generate
```

#### 3. Migration Review

```markdown
## Migration Review

### New Migration: 0008_add_analytics.sql

**Changes:**
- ADD TABLE: aibotkit_analytics
- ADD COLUMN: chatbots.analytics_enabled
- ADD INDEX: idx_analytics_date

**Risk Assessment:** LOW
- No data loss
- No breaking changes
- Backward compatible
```

#### 4. Migration Application

```bash
pnpm db:migrate
```

#### 5. Verification

```markdown
## Post-Migration Verification

- [ ] All tables exist
- [ ] Indexes created
- [ ] Foreign keys valid
- [ ] Sample queries work
```

### Examples

**Development Workflow:**
```bash
# Edit schema
vim src/lib/db/schema.ts

# Generate and apply
/sync-db

# Verify in Studio
/sync-db --studio
```

**Production Migration:**
```bash
# Generate migration
/sync-db --generate-only

# Review generated SQL
cat src/lib/db/migrations/0008_*.sql

# Apply in production
/sync-db --env production
```

---

## Command Chaining

Commands can be used together in workflows:

```bash
# Development workflow
/next-phase 7          # Implement feature
/test-rag              # Test RAG changes
/full-review           # Review code
/sync-db               # Sync database
/deploy-saas --dry-run # Validate deployment

# All checks passed? Deploy!
/deploy-saas --env production
```

## See Also

- [Agents Reference](AGENTS.md)
- [Skills Reference](SKILLS.md)
- [Workflow Examples](WORKFLOWS.md)
- [Getting Started](../GETTING_STARTED.md)
