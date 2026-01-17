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
| `/update-docs` | Documentation sync | Keep GitBook docs current |

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

Guides development through a 21-phase lifecycle (3 Discovery + 18 SDD phases) with quality gates, test loops, and explicit execution instructions.

### Usage

```bash
# Start from beginning (Phase 0.1: Discovery)
/next-phase

# Continue from specific phase
/next-phase 4
/next-phase 5.6

# Provide requirements document
/next-phase --brief requirements.md
/next-phase --brief "Add user authentication with OAuth"

# Component-specific
/next-phase --component saas
/next-phase --component wordpress
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `[phase]` | Phase number | `/next-phase 5.6` |
| `--brief` | Requirements file or text | `--brief requirements.md` |
| `--component` | Target component | `--component saas` |
| `--skip-discovery` | Skip discovery phases | `--skip-discovery` |
| `--validate` | Run validation after generation | `--validate` |
| `--fix` | Auto-fix issues | `--fix` |

### Phases

#### Discovery Phases (0.1-0.3)

| Phase | Name | Purpose |
|-------|------|---------|
| 0.1 | Codebase Discovery | Index existing code, generate `_project_specs/DISCOVERY_REPORT.md` |
| 0.2 | Documentation Recovery | Fill missing docs, generate `specs/RECOVERED_*.md` |
| 0.3 | Gap Analysis | Compare current vs requirements, generate `reports/GAP_ANALYSIS.md` |

#### SDD Phases (0-12)

| Phase | Name | Purpose |
|-------|------|---------|
| 0 | Project Init | Ensure CLAUDE.md and CONSTITUTION.md exist |
| 0.5 | Clarification | Refine requirements with user |
| 1 | Estimation | Effort estimation with gap analysis credits |
| 2 | Analysis | Requirements analysis |
| 3 | Design | UI/UX design |
| 4 | Architecture | System architecture (EXTEND mode) |
| 5 | Specification | Technical specifications |
| 5.5 | Cross-Analysis | Cross-artifact consistency check |
| 5.6 | Req-Spec Validation | **GATE:** Must pass before coding |
| 5.7 | Manual Test Cases | Generate new + regression test cases |
| 5.8 | Dependency Collection | **GATE:** All deps must be available |
| 6 | Coding | Implementation (EXTEND mode) |
| 6.5 | Spec Validation | Verify implementation matches specs |
| 7 | Testing | Write unit, integration, E2E tests |
| 8 | Test & Fix Loop | **LOOP:** Run tests, fix bugs until 100% pass |
| 9 | Code Review | Run `/full-review` |
| 10 | Fix & Validate | **LOOP:** Fix issues until quality threshold |
| 11 | Documentation | Run `/update-docs` |
| 12 | Deployment | Deployment validation |

### Quality Gates

| Gate | Phase | Requirement |
|------|-------|-------------|
| Req-Spec Validation | 5.6 | 100% requirement coverage by specs |
| Dependency Collection | 5.8 | All dependencies available |
| Test & Fix Loop | 8 | 100% tests passing |
| Review & Fix Loop | 10 | Quality threshold met |

### Phase Outputs

```
Phase 0.1 → _project_specs/DISCOVERY_REPORT.md
Phase 0.3 → reports/GAP_ANALYSIS.md
Phase 5 → specs/SPECIFICATION.md
Phase 5.6 → reports/REQ_SPEC_VALIDATION.md
Phase 5.7 → tests/TEST_CASES.md, tests/REGRESSION_TEST_CASES.md
Phase 6 → src/ (extended code)
Phase 7 → tests/e2e/, tests/unit/, tests/integration/
Phase 8 → reports/TEST_EXECUTION.md, reports/BUG_FIX_LOG.md
Phase 9 → reports/REVIEW.md
Phase 11 → Updated README.md, CHANGELOG.md
Phase 12 → reports/DEPLOYMENT_CHECKLIST.md
```

### Examples

**New Feature Development:**
```bash
# Start with discovery
/next-phase
# Output: Discovery complete. 127 files indexed.

# Continue through phases
/next-phase 0.5
# Output: Requirements clarified.

# Continue through all phases...
```

**Continue from Specific Phase:**
```bash
# Jump to implementation after planning
/next-phase 6

# Run quality gate validation
/next-phase 5.6
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

## /update-docs

Automatically synchronizes GitBook documentation with SaaS and WordPress codebases.

### Usage

```bash
# Full sync with diff preview
/update-docs

# Apply changes without confirmation
/update-docs --apply

# Update specific section only
/update-docs --section plans
/update-docs --section features

# Validate only (no changes)
/update-docs --validate

# Generate API documentation
/update-docs --generate-api

# Dry run (show what would change)
/update-docs --dry-run
```

### Options

| Option | Description | Example |
|--------|-------------|---------|
| `--apply` | Apply without confirmation | `--apply` |
| `--section` | Target specific section | `--section plans` |
| `--validate` | Validate only | `--validate` |
| `--generate-api` | Generate API docs | `--generate-api` |
| `--dry-run` | Preview changes | `--dry-run` |

### What Gets Synced

| Data Source | Target Documentation |
|-------------|---------------------|
| `saas/src/lib/checkRateLimit.ts` | Plan limits in `plans-and-billing/*.md` |
| `saas/docs/plan-limits.md` | Plan descriptions |
| `wordpress-plugin/readme.txt` | Feature lists |
| `saas/src/app/api/` | API reference docs |

### Workflow Phases

#### Phase 1: Discovery
- Extract plan limits from `checkRateLimit.ts`
- Parse features from code and readme.txt
- Catalog API endpoints

#### Phase 2: Analysis
- Compare extracted data with documentation
- Identify discrepancies
- Calculate drift score

#### Phase 3: Generation
- Generate updated documentation sections
- Preserve existing formatting
- Update tables and lists

#### Phase 4: Validation
- Check all internal links
- Verify image references
- Validate SUMMARY.md structure

#### Phase 5: Application
- Show diff preview
- Apply changes (with `--apply`)
- Generate update report

### Output Format

```markdown
## Documentation Sync Report

### Discrepancies Found

| File | Issue | Current | Expected |
|------|-------|---------|----------|
| plans-and-billing/upgrading-to-paid-plans.md | Basic messages | 300 | 500 |

### Changes Applied
- Updated Basic plan messages in 2 files
- Added missing feature to key-features.md

### Validation Results
- ✅ 45 internal links valid
- ✅ 102 images valid
- ✅ SUMMARY.md complete
```

### Examples

**Pre-Release Doc Sync:**
```bash
# Sync all docs before release
/update-docs --apply

# Validate everything is correct
/update-docs --validate
```

**After Pricing Changes:**
```bash
# Update plan limits after changing PLAN_LIMITS constant
/update-docs --section plans --apply
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
