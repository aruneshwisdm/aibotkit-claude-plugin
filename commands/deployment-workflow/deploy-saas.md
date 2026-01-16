# /deploy-saas

Comprehensive deployment workflow for the AI BotKit SaaS application with pre-deployment validation, deployment execution, and post-deployment verification.

## Context

Deploying the AI BotKit SaaS involves:
- Database migrations (Drizzle)
- Environment variable validation
- Stripe webhook configuration
- Vercel deployment
- Post-deployment health checks

This command ensures safe, reliable deployments with rollback capability.

## Prerequisites

Before running this command:
- All tests passing (`pnpm build` succeeds)
- `/full-review` completed with no critical issues
- Database backup completed (for production)
- Stripe webhooks configured for target environment

## Deployment Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    /deploy-saas WORKFLOW                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ PHASE 1: PRE-DEPLOYMENT VALIDATION                          │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ 1.1 Environment Check                                        │    │
│  │     → Verify all required env vars set                       │    │
│  │     → Validate database connection                           │    │
│  │     → Check Stripe API keys                                  │    │
│  │     → Verify Pinecone configuration                          │    │
│  │                                                              │    │
│  │ 1.2 Build Validation                                         │    │
│  │     → Run pnpm build                                         │    │
│  │     → Check for TypeScript errors                            │    │
│  │     → Verify bundle size                                     │    │
│  │                                                              │    │
│  │ 1.3 Database Migration Check                                 │    │
│  │     → List pending migrations                                │    │
│  │     → Verify migration safety                                │    │
│  │     → Check for breaking changes                             │    │
│  │                                                              │    │
│  │ 1.4 Security Scan                                            │    │
│  │     → Run quick security check                               │    │
│  │     → Verify no secrets in code                              │    │
│  │     → Check dependency vulnerabilities                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ↓                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ PHASE 2: DEPLOYMENT EXECUTION                               │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ 2.1 Database Migrations                                      │    │
│  │     → Create backup point                                    │    │
│  │     → Run pnpm db:migrate                                    │    │
│  │     → Verify migration success                               │    │
│  │                                                              │    │
│  │ 2.2 Application Deployment                                   │    │
│  │     → Deploy to Vercel (or target platform)                  │    │
│  │     → Monitor deployment logs                                │    │
│  │     → Wait for deployment ready                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ↓                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ PHASE 3: POST-DEPLOYMENT VERIFICATION                       │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ 3.1 Health Checks                                            │    │
│  │     → Verify application responding                          │    │
│  │     → Check API endpoints                                    │    │
│  │     → Verify database connectivity                           │    │
│  │                                                              │    │
│  │ 3.2 Functional Tests                                         │    │
│  │     → Test authentication flow                               │    │
│  │     → Test chatbot creation                                  │    │
│  │     → Test chat streaming                                    │    │
│  │     → Test Stripe checkout (test mode)                       │    │
│  │                                                              │    │
│  │ 3.3 Integration Tests                                        │    │
│  │     → Test WordPress plugin connection                       │    │
│  │     → Verify webhook endpoints                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ↓                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ PHASE 4: DEPLOYMENT REPORT                                  │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │     → Generate deployment summary                            │    │
│  │     → Document any issues encountered                        │    │
│  │     → Provide rollback instructions if needed                │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

## Usage

```bash
/deploy-saas [environment] [options]
```

### Environments

| Environment | Description |
|-------------|-------------|
| `staging` | Staging/preview deployment (default) |
| `production` | Production deployment |

### Options

| Option | Description |
|--------|-------------|
| `--skip-migrations` | Skip database migrations |
| `--skip-tests` | Skip post-deployment tests |
| `--dry-run` | Validate without deploying |

### Examples

```bash
/deploy-saas staging              # Deploy to staging
/deploy-saas production           # Deploy to production
/deploy-saas --dry-run            # Validate only
/deploy-saas staging --skip-tests # Deploy without post-tests
```

## Phase 1: Pre-Deployment Validation

### 1.1 Environment Check

**Required Environment Variables:**

| Variable | Purpose | Validation |
|----------|---------|------------|
| `POSTGRES_URL` | Database connection | Connection test |
| `STRIPE_SECRET_KEY` | Stripe API | Key format check |
| `STRIPE_WEBHOOK_SECRET` | Webhook verification | Key format check |
| `AUTH_SECRET` | JWT signing | Minimum length |
| `PINECONE_API_KEY` | Vector database | Connection test |
| `PINECONE_INDEX` | Vector index name | Index exists |
| `PINECONE_HOST` | Pinecone host | Reachable |
| `TOGETHER_API_KEY` | LLM API | Key format |
| `BASE_URL` | Application URL | Valid URL |

**Validation Script:**
```bash
# Check environment variables
node -e "
const required = [
  'POSTGRES_URL',
  'STRIPE_SECRET_KEY',
  'STRIPE_WEBHOOK_SECRET',
  'AUTH_SECRET',
  'PINECONE_API_KEY',
  'PINECONE_INDEX',
  'BASE_URL'
];

const missing = required.filter(key => !process.env[key]);
if (missing.length > 0) {
  console.error('Missing required environment variables:', missing);
  process.exit(1);
}
console.log('All required environment variables present');
"
```

### 1.2 Build Validation

```bash
# Clean build
rm -rf .next
pnpm build

# Check for build errors
if [ $? -ne 0 ]; then
  echo "Build failed - aborting deployment"
  exit 1
fi

# Check bundle size (warn if > 500KB first load JS)
```

### 1.3 Database Migration Check

```bash
# List pending migrations
pnpm db:generate --dry-run

# Review migration SQL
# Ensure no DROP TABLE or DROP COLUMN without backup plan
```

### 1.4 Security Scan

**Checks performed:**
- No hardcoded secrets in codebase
- No `.env` files committed
- Dependencies without known vulnerabilities
- No debug code in production build

## Phase 2: Deployment Execution

### 2.1 Database Migrations

```bash
# Create backup (production only)
pg_dump $POSTGRES_URL > backup_$(date +%Y%m%d_%H%M%S).sql

# Run migrations
pnpm db:migrate

# Verify migration
pnpm db:studio  # Quick visual check
```

### 2.2 Application Deployment

**Vercel Deployment:**
```bash
# Deploy to Vercel
vercel --prod  # or vercel for preview

# Or using git push (if Vercel GitHub integration)
git push origin main
```

**Manual Deployment:**
```bash
# Build
pnpm build

# Start production server
pnpm start
```

## Phase 3: Post-Deployment Verification

### 3.1 Health Checks

```bash
# Application health
curl -f https://app.aibotkit.io/api/health || exit 1

# Database connectivity
curl -f https://app.aibotkit.io/api/health/db || exit 1
```

### 3.2 Functional Tests

**Endpoints to test:**

| Endpoint | Method | Expected |
|----------|--------|----------|
| `/` | GET | 200 OK |
| `/api/health` | GET | 200 OK |
| `/api/chatbots` | GET (auth) | 200 OK or 401 |
| `/api/chat` | POST | 200 OK (streaming) |
| `/api/stripe/webhook` | POST | 400 (no signature) |

### 3.3 Integration Tests

```bash
# Test WordPress plugin connectivity
curl -X GET "https://app.aibotkit.io/api/wordpress/chatbots" \
  -H "Authorization: Bearer test-token" \
  -H "Content-Type: application/json"
```

## Phase 4: Deployment Report

### Output Format

```markdown
# Deployment Report

## Summary

| Field | Value |
|-------|-------|
| Environment | production |
| Deployment Time | 2024-12-15 14:30:00 UTC |
| Duration | 5m 32s |
| Status | ✅ SUCCESS |

## Pre-Deployment Checks

| Check | Status | Notes |
|-------|--------|-------|
| Environment Variables | ✅ Pass | All 12 variables set |
| Build | ✅ Pass | Built in 45s |
| Migrations | ✅ Pass | 2 migrations applied |
| Security Scan | ✅ Pass | No issues found |

## Migrations Applied

| Migration | Status |
|-----------|--------|
| 0005_add_analytics_table | ✅ Applied |
| 0006_add_user_preferences | ✅ Applied |

## Post-Deployment Tests

| Test | Status | Response Time |
|------|--------|---------------|
| Health Check | ✅ Pass | 120ms |
| Auth Flow | ✅ Pass | 350ms |
| Chatbot API | ✅ Pass | 280ms |
| Chat Streaming | ✅ Pass | 450ms |
| Stripe Webhook | ✅ Pass | 180ms |
| WordPress API | ✅ Pass | 220ms |

## Rollback Instructions

If issues are discovered:

1. **Revert Application:**
   ```bash
   vercel rollback
   ```

2. **Revert Database (if needed):**
   ```bash
   psql $POSTGRES_URL < backup_20241215_143000.sql
   ```

## Next Steps

- [ ] Monitor error tracking (Sentry)
- [ ] Check analytics for anomalies
- [ ] Verify Stripe webhooks receiving events
- [ ] Test from WordPress plugin
```

## Rollback Procedures

### Application Rollback

**Vercel:**
```bash
# List deployments
vercel ls

# Rollback to previous
vercel rollback [deployment-url]
```

### Database Rollback

```bash
# Restore from backup
psql $POSTGRES_URL < backup_YYYYMMDD_HHMMSS.sql

# Or run down migrations (if implemented)
pnpm db:migrate:down
```

## Related Commands

- `/full-review` - Pre-deployment code review
- `/next-phase` - Development workflow
- `/test-rag` - RAG engine testing
- `/sync-db` - Database synchronization

## Success Criteria

A successful deployment:
- ✅ All pre-deployment checks pass
- ✅ Build completes without errors
- ✅ Migrations apply successfully
- ✅ Application responds to health checks
- ✅ All post-deployment tests pass
- ✅ No errors in monitoring
- ✅ Rollback plan documented
