# Workflow Examples

Real-world workflow examples using the AI BotKit Engineering Plugin.

## Overview

| Workflow | Commands Used | Duration |
|----------|---------------|----------|
| New Feature Development | `/next-phase`, `/full-review`, `/deploy-saas` | Multi-session |
| Bug Fix | `/full-review` | Single session |
| RAG Optimization | `/test-rag`, `/full-review` | Single session |
| Database Migration | `/sync-db`, `/deploy-saas` | Single session |
| Security Audit | `/full-review --focus security` | Single session |
| Documentation Sync | `/update-docs` | Single session |
| Pre-Release Review | `/full-review`, `/test-rag`, `/update-docs`, `/deploy-saas --dry-run` | Single session |

---

## Workflow 1: New Feature Development

**Scenario:** Adding a chatbot templates feature that lets users create chatbots from pre-built templates.

### Phase 0.1-0.3: Discovery

```bash
# Start discovery
claude
> /next-phase

# Phase 0.1: Claude indexes codebase and generates:
# - _project_specs/DISCOVERY_REPORT.md
# - _project_specs/saas-index.md
# - _project_specs/wordpress-index.md

# Phase 0.2: Documentation Recovery (if needed)
# Phase 0.3: Gap Analysis
```

**Output:** Codebase indexed, gaps identified.

### Phase 0.5-1: Clarification & Estimation

```bash
> /next-phase 0.5

# Provide requirements
> /next-phase --brief "Users should be able to:
> 1. Browse pre-built chatbot templates
> 2. Preview a template before using it
> 3. Create a chatbot from a template
> 4. Templates should include: name, description, personality, style"

> /next-phase 1  # Estimation with gap analysis credits
```

**Output:** Refined requirements, effort estimates.

### Phase 2-4: Analysis, Design & Architecture

```bash
> /next-phase 4
```

**Output:**
```markdown
## Architecture Design (EXTEND Mode)

### Database Changes
- New table: `aibotkit_templates`
- Fields: id, name, description, personality, style, category, isPublic

### API Endpoints
- GET /api/templates - List templates
- GET /api/templates/:id - Get template
- POST /api/chatbots/from-template - Create from template

### UI Components
- TemplateGallery - Grid of template cards
- TemplatePreview - Modal with preview
- CreateFromTemplate - Creation flow
```

### Phase 5-5.8: Specification & Quality Gates

```bash
> /next-phase 5    # Technical specifications
> /next-phase 5.5  # Cross-artifact consistency
> /next-phase 5.6  # GATE: Req-Spec Validation (must pass!)
> /next-phase 5.7  # Generate test cases (new + regression)
> /next-phase 5.8  # GATE: Dependency collection
```

**Output:** specs/SPECIFICATION.md, tests/TEST_CASES.md

### Phase 6-6.5: Implementation

```bash
> /next-phase 6
```

**Implementation Steps:**

1. **Database Schema:**
```bash
> Help me create the templates table schema

# Claude uses drizzle-patterns skill
# Creates src/lib/db/schema/templates.ts
```

2. **API Endpoints:**
```bash
> Create the API routes for templates

# Claude uses nextjs-saas skill
# Creates src/app/api/templates/route.ts
# Creates src/app/api/templates/[id]/route.ts
```

3. **UI Components:**
```bash
> Build the template gallery component

# Claude creates components with proper patterns

> /next-phase 6.5  # Verify implementation matches specs
```

### Phase 7-8: Testing & Fix Loop

```bash
> /next-phase 7  # Write all tests (unit, integration, E2E)

> /next-phase 8  # LOOP: Run tests, fix bugs until 100% pass
# This loops automatically:
# - Run all tests
# - Fix failures (prioritize regression)
# - Re-run until 100% pass
```

### Phase 9-10: Code Review & Fix Loop

```bash
> /next-phase 9
# This automatically runs /full-review

> /next-phase 10  # LOOP: Fix issues until quality threshold met
```

### Phase 11-12: Documentation & Deployment

```bash
> /next-phase 11  # Runs /update-docs
> /next-phase 12  # Deployment validation
# Or directly: /deploy-saas
```

---

## Workflow 2: Bug Fix

**Scenario:** Chat messages not appearing in real-time for some users.

### Step 1: Investigate

```bash
claude
> Users report chat messages not appearing in real-time. Help me investigate.

# Claude analyzes:
# - Streaming implementation
# - WebSocket/SSE setup
# - Client-side handling
```

### Step 2: Identify Issue

```bash
> Check the streaming response code in rag-engine

# Claude reads and analyzes:
# src/lib/ai/rag-engine.ts
# src/app/api/chat/route.ts
# src/components/ChatWidget/
```

### Step 3: Fix

```bash
> The issue is the ReadableStream not flushing properly. Fix it.

# Claude applies fix with proper patterns
```

### Step 4: Review Fix

```bash
> /full-review --files src/lib/ai/rag-engine.ts src/app/api/chat/route.ts

# Ensures fix follows best practices
```

### Step 5: Test

```bash
> /test-rag --component generation

# Validates streaming works correctly
```

---

## Workflow 3: RAG Optimization

**Scenario:** Chatbot responses are slow and sometimes irrelevant.

### Step 1: Benchmark Current Performance

```bash
claude
> /test-rag --benchmark

# Output:
# Search Latency: 450ms (target: <200ms) ❌
# Generation Latency: 3.2s (target: <2s) ❌
# Relevance Score: 0.65 (target: >0.75) ❌
```

### Step 2: Analyze RAG Implementation

```bash
> /full-review --focus rag

# Reviews:
# - Chunking strategy
# - Embedding approach
# - Search configuration
# - Prompt construction
```

### Step 3: Implement Optimizations

```bash
> Help me optimize the vector search based on the review findings

# Claude applies optimizations:
# - Add score thresholding
# - Implement embedding caching
# - Optimize chunk size
# - Add reranking
```

### Step 4: Validate Improvements

```bash
> /test-rag --benchmark

# Output:
# Search Latency: 180ms ✅
# Generation Latency: 1.8s ✅
# Relevance Score: 0.78 ✅
```

---

## Workflow 4: Database Migration

**Scenario:** Adding analytics tracking to chatbots.

### Step 1: Design Schema

```bash
claude
> I need to add analytics tracking. Help me design the schema.

# Claude uses drizzle-patterns skill to design:
# - aibotkit_analytics table
# - Proper indexes
# - Foreign key relationships
```

### Step 2: Create Schema

```bash
> Create the analytics schema in src/lib/db/schema.ts
```

### Step 3: Generate Migration

```bash
> /sync-db --generate-only

# Reviews generated migration:
# migrations/0008_add_analytics.sql
```

### Step 4: Review Migration

```bash
> /full-review --files src/lib/db/schema.ts

# Checks:
# - Proper indexes
# - Type safety
# - Naming conventions
```

### Step 5: Apply Migration

```bash
> /sync-db

# Applies migration and verifies
```

### Step 6: Verify

```bash
> /sync-db --studio

# Opens Drizzle Studio to inspect tables
```

---

## Workflow 5: Security Audit

**Scenario:** Pre-release security review.

### Step 1: Run Security-Focused Review

```bash
claude
> /full-review --focus security

# Runs:
# - saas-security-auditor
# - stripe-integration-reviewer
# - wordpress-security-auditor
```

### Step 2: Review Findings

```markdown
## Security Audit Results

### Critical (0)
None found ✅

### High (2)
1. Missing rate limiting on /api/chat
2. CSRF token not validated on settings update

### Medium (5)
1. Session cookie missing Secure flag in dev
2. API key logged in error messages
...
```

### Step 3: Fix Critical/High Issues

```bash
> Help me implement rate limiting for the chat API

# Claude implements:
# - Rate limiter middleware
# - Per-user/IP limits
# - Proper error responses
```

### Step 4: Re-run Audit

```bash
> /full-review --focus security --severity high

# Verifies all high+ issues resolved
```

---

## Workflow 6: Pre-Release Checklist

**Scenario:** Preparing for production release.

### Step 1: Full Code Review

```bash
claude
> /full-review

# Complete review of all components
```

### Step 2: RAG Testing

```bash
> /test-rag

# Ensures AI features work correctly
```

### Step 3: Database Check

```bash
> /sync-db --status

# Verifies no pending migrations
```

### Step 4: Deployment Dry Run

```bash
> /deploy-saas --dry-run

# Validates:
# - All tests pass
# - Build succeeds
# - Environment configured
# - Health checks defined
```

### Step 5: Deploy

```bash
> /deploy-saas --env production

# Executes deployment with validation
```

### Step 6: Post-Deployment Verification

```bash
> /deploy-saas --verify

# Runs health checks on production
```

---

## Workflow 7: WordPress Plugin Update

**Scenario:** Adding new feature to WordPress plugin.

### Step 1: Plan

```bash
claude
> /next-phase --brief "Add widget position customization to WordPress plugin"
```

### Step 2: Implement

```bash
> Help me add the position settings to the admin panel

# Claude uses:
# - wordpress-standards-reviewer patterns
# - Proper sanitization
# - i18n compliance
```

### Step 3: Review

```bash
> /full-review --component wordpress

# Reviews:
# - WPCS compliance
# - Security (SQL injection, XSS, CSRF)
# - i18n completeness
# - Hook usage
```

### Step 4: Test API Integration

```bash
> /full-review --focus api-integration

# Ensures WordPress plugin correctly
# communicates with SaaS API
```

---

## Workflow 8: Documentation Sync

**Scenario:** Plan limits changed in code, need to update GitBook documentation.

### Step 1: Make Code Changes

```bash
# Edit plan limits in saas/src/lib/checkRateLimit.ts
vim saas/src/lib/checkRateLimit.ts

# Change Basic plan from 500 to 750 messages
```

### Step 2: Run Documentation Sync

```bash
claude
> /update-docs

# Output:
## Documentation Analysis

### Discrepancies Found

| File | Issue | Current | Expected |
|------|-------|---------|----------|
| plans-and-billing/upgrading-to-paid-plans.md | Basic messages | 500 | 750 |
| introduction/free-plan-and-upgrades.md | Basic messages | 500 | 750 |
| introduction/free-plan-and-upgrades.md | Table row | 500 | 750 |

Apply changes? [y/N]
```

### Step 3: Apply Changes

```bash
> /update-docs --apply

# Output:
## Changes Applied

- plans-and-billing/upgrading-to-paid-plans.md: Updated Basic messages
- introduction/free-plan-and-upgrades.md: Updated Basic messages (2 locations)

All documentation now matches code.
```

### Step 4: Validate

```bash
> /update-docs --validate

# Output:
## Validation Report

✅ 29 documentation files checked
✅ 0 discrepancies found
✅ 45 internal links valid
✅ 102 images valid
✅ SUMMARY.md complete
```

### Step 5: Commit Documentation

```bash
git add documentation/
git commit -m "docs: update Basic plan limits to 750 messages"
```

---

## Workflow 9: Pre-Release Documentation Review

**Scenario:** Before a major release, ensure all documentation is accurate.

### Step 1: Sync Documentation

```bash
claude
> /update-docs --apply

# Syncs all plan limits, features from code
```

### Step 2: Validate Links

```bash
> /update-docs --validate

# Checks:
# - All internal links work
# - All images exist
# - SUMMARY.md is complete
```

### Step 3: Review Changes

```bash
# Check git diff for documentation changes
git diff documentation/

# Review and stage changes
git add documentation/
```

### Step 4: Full Pre-Release Check

```bash
> /full-review
> /test-rag
> /deploy-saas --dry-run

# All checks should pass
```

---

## Quick Reference

### Common Command Combinations

```bash
# Feature development
/next-phase → /full-review → /deploy-saas

# Bug fix
/full-review --files <affected-files> → fix → /full-review

# Database changes
/sync-db --generate-only → review → /sync-db

# Security audit
/full-review --focus security → fix → /full-review --focus security

# RAG optimization
/test-rag --benchmark → optimize → /test-rag --benchmark

# Documentation sync
/update-docs → review changes → /update-docs --apply

# Pre-release
/full-review → /test-rag → /update-docs → /deploy-saas --dry-run → /deploy-saas
```

### Environment-Specific Workflows

**Development:**
```bash
/sync-db --push          # Quick schema updates
/full-review --files ... # Targeted reviews
```

**Staging:**
```bash
/sync-db                 # Proper migrations
/deploy-saas --env staging
```

**Production:**
```bash
/full-review             # Complete review
/test-rag                # Full RAG tests
/deploy-saas --dry-run   # Validate first
/deploy-saas --env production
```

## See Also

- [Commands Reference](COMMANDS.md)
- [Agents Reference](AGENTS.md)
- [Skills Reference](SKILLS.md)
- [Getting Started](../GETTING_STARTED.md)
