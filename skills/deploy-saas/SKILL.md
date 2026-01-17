---
name: deploy-saas
description: SaaS deployment orchestrator with pre-checks, quality gates, rollback tracking, and resume capability
user_invocable: true
---

# /deploy-saas - Deployment Orchestrator

This skill orchestrates safe deployment of the AI BotKit SaaS application with quality gates, state persistence, and rollback capability.

## Invocation

```bash
/deploy-saas                      # Deploy to staging (default)
/deploy-saas production           # Deploy to production
/deploy-saas --dry-run            # Validate without deploying
/deploy-saas --status             # Show current deployment state
/deploy-saas --resume             # Resume from failed step
/deploy-saas --rollback           # Rollback to previous deployment
/deploy-saas --skip-migrations    # Skip database migrations
```

## Orchestrator Execution Instructions

When `/deploy-saas` is invoked, follow this orchestration protocol:

### Step 1: Initialize State

Check for existing state file at `_project_specs/.deploy-state.json`:

```json
{
  "environment": "production",
  "currentPhase": "1.1",
  "status": "in_progress",
  "startedAt": "ISO-DATE",
  "lastUpdated": "ISO-DATE",
  "preChecks": {
    "envVars": { "status": "pending" },
    "build": { "status": "pending" },
    "migrations": { "status": "pending", "pending": 0 },
    "security": { "status": "pending" }
  },
  "deployment": {
    "backupLocation": null,
    "previousDeploymentUrl": null,
    "newDeploymentUrl": null,
    "migrationsApplied": []
  },
  "postChecks": {
    "healthCheck": { "status": "pending" },
    "functionalTests": { "status": "pending" },
    "integrationTests": { "status": "pending" }
  }
}
```

If `--resume` flag provided and state exists with `status: "failed"`, resume from `currentPhase`.
If no state exists or `--reset`, create fresh state.

### Step 2: Phase Router

Execute the appropriate phase based on current state:

```
PHASE ROUTING TABLE
===============================================================================

PRE-DEPLOYMENT CHECKS (Quality Gates)
-------------------------------------------------------------------------------
Phase 1.1 | ENV_VARS_CHECK
          | Action: Verify all required environment variables
          | Gate: ALL required vars must be present
          | On Fail: STOP - List missing variables
          | Next: 1.2
-------------------------------------------------------------------------------
Phase 1.2 | BUILD_VALIDATION
          | Action: Run pnpm build, check for errors
          | Gate: Build must succeed with no TypeScript errors
          | On Fail: STOP - Show build errors
          | Next: 1.3
-------------------------------------------------------------------------------
Phase 1.3 | MIGRATION_CHECK
          | Action: List pending migrations, assess risk
          | Gate: No high-risk migrations without explicit approval
          | On Fail: STOP - Request migration approval
          | Next: 1.4
-------------------------------------------------------------------------------
Phase 1.4 | SECURITY_SCAN
          | Action: Check for secrets in code, dependency vulnerabilities
          | Gate: No critical security issues
          | On Fail: STOP - List security issues
          | Next: 2.1
===============================================================================

DEPLOYMENT EXECUTION
-------------------------------------------------------------------------------
Phase 2.1 | DATABASE_BACKUP
          | Action: Create database backup (production only)
          | Output: Store backup location in state
          | Next: 2.2
-------------------------------------------------------------------------------
Phase 2.2 | RUN_MIGRATIONS
          | Action: Apply pending database migrations
          | Output: Record applied migrations in state
          | On Fail: STOP - Provide rollback instructions
          | Next: 2.3
-------------------------------------------------------------------------------
Phase 2.3 | DEPLOY_APPLICATION
          | Action: Deploy to Vercel (or target platform)
          | Output: Store new deployment URL in state
          | On Fail: STOP - Provide rollback instructions
          | Next: 3.1
===============================================================================

POST-DEPLOYMENT VERIFICATION
-------------------------------------------------------------------------------
Phase 3.1 | HEALTH_CHECK
          | Action: Verify application responding
          | Gate: Health endpoint must return 200
          | On Fail: WARN - Consider rollback
          | Next: 3.2
-------------------------------------------------------------------------------
Phase 3.2 | FUNCTIONAL_TESTS
          | Action: Test auth, chatbot API, chat streaming
          | Gate: Critical paths must work
          | On Fail: WARN - Consider rollback
          | Next: 3.3
-------------------------------------------------------------------------------
Phase 3.3 | INTEGRATION_TESTS
          | Action: Test WordPress plugin connection, webhooks
          | Gate: Integrations must work
          | On Fail: WARN - Document issues
          | Next: 4
===============================================================================

FINALIZATION
-------------------------------------------------------------------------------
Phase 4   | GENERATE_REPORT
          | Action: Generate deployment report
          | Output: reports/DEPLOYMENT_REPORT.md
          | Next: COMPLETE
===============================================================================
```

### Step 3: Execute Current Phase

For each phase, follow this execution pattern:

```markdown
## Executing Phase {PHASE_NUMBER}: {PHASE_NAME}

### Pre-Execution
1. Log phase start to state file
2. Update status to "in_progress"

### Execution
1. Perform phase actions
2. Collect results

### Post-Execution
1. Update state with results
2. Determine pass/fail

### Quality Gate Handling (Phases 1.1-1.4)
IF gate fails:
  - Set status to "failed"
  - Set failureReason in state
  - Output clear error message
  - Provide fix instructions
  - DO NOT proceed to next phase

IF gate passes:
  - Update state
  - Proceed to next phase
```

### Step 4: State Management

After each phase, update the state file:

```json
{
  "environment": "production",
  "currentPhase": "2.3",
  "status": "in_progress",
  "startedAt": "2024-12-15T14:00:00Z",
  "lastUpdated": "2024-12-15T14:15:00Z",
  "preChecks": {
    "envVars": { "status": "passed", "count": 12 },
    "build": { "status": "passed", "duration": "45s" },
    "migrations": { "status": "passed", "pending": 2 },
    "security": { "status": "passed", "warnings": 0 }
  },
  "deployment": {
    "backupLocation": "backup_20241215_140500.sql",
    "previousDeploymentUrl": "https://aibotkit-xyz789.vercel.app",
    "newDeploymentUrl": "https://aibotkit-abc123.vercel.app",
    "migrationsApplied": ["0007_add_analytics", "0008_add_flag"]
  },
  "postChecks": {
    "healthCheck": { "status": "pending" },
    "functionalTests": { "status": "pending" },
    "integrationTests": { "status": "pending" }
  }
}
```

### Step 5: User Communication

At each phase transition, output a status update:

```markdown
===============================================================================
PHASE 1.2 COMPLETE: Build Validation

Result: PASSED
Duration: 45s
Bundle Size: 423KB (within limits)

===============================================================================
Starting Phase 1.3: Migration Check

Checking for pending migrations...
===============================================================================
```

## Quality Gates

### Gate 1.1: Environment Variables

```
REQUIRED VARIABLES:
- POSTGRES_URL
- STRIPE_SECRET_KEY
- STRIPE_WEBHOOK_SECRET
- AUTH_SECRET
- PINECONE_API_KEY
- PINECONE_INDEX
- PINECONE_HOST
- TOGETHER_API_KEY (or OPENAI_API_KEY)
- BASE_URL

PASS Criteria: All required variables present and valid format
FAIL Action: List missing variables, STOP deployment
```

### Gate 1.2: Build Validation

```
CHECKS:
- pnpm build completes successfully
- No TypeScript errors
- Bundle size within limits (<500KB first load)

PASS Criteria: Build succeeds, no errors
FAIL Action: Show build errors, STOP deployment
```

### Gate 1.3: Migration Check

```
RISK ASSESSMENT:
- LOW: Add table, add nullable column, add index
- MEDIUM: Add NOT NULL column with default
- HIGH: Drop table, drop column, change column type

PASS Criteria: No HIGH risk without approval
FAIL Action: List risky migrations, request approval
```

### Gate 1.4: Security Scan

```
CHECKS:
- No .env files in commit
- No hardcoded secrets
- No known vulnerable dependencies (critical)
- No debug code in production

PASS Criteria: No critical security issues
FAIL Action: List issues, STOP deployment
```

## Rollback Procedure

When `/deploy-saas --rollback` is invoked:

```markdown
## Rollback Execution

### Step 1: Load State
Read _project_specs/.deploy-state.json
Verify deployment exists to rollback

### Step 2: Application Rollback
IF Vercel:
  - Run: vercel rollback {previousDeploymentUrl}
  - Verify rollback successful

### Step 3: Database Rollback (if migrations were applied)
IF migrationsApplied.length > 0:
  - WARN: Database rollback may cause data loss
  - IF backupLocation exists:
    - Offer to restore from backup
    - Run: psql $POSTGRES_URL < {backupLocation}
  - ELSE:
    - Manual rollback required

### Step 4: Update State
Set status to "rolled_back"
Record rollback timestamp
```

## Environment-Specific Behavior

| Environment | Backup | Migrations | Post-Tests |
|-------------|--------|------------|------------|
| `staging` | Optional | Auto-apply | Basic |
| `production` | **Required** | Approval needed | Full suite |

## Dry Run Mode

When `--dry-run` is specified:

```markdown
## Dry Run Report

### Pre-Checks Would Run:
- Environment variables: 12 required
- Build validation: pnpm build
- Migration check: 2 pending
- Security scan: Full scan

### Deployment Would:
- Create backup: Yes (production)
- Apply migrations: 0007_add_analytics, 0008_add_flag
- Deploy to: Vercel production

### Post-Checks Would Run:
- Health check: /api/health
- Functional tests: Auth, Chatbot, Chat
- Integration tests: WordPress, Webhooks

No changes made.
```

## Output Format

### Deployment Report

```markdown
# Deployment Report

## Summary

| Field | Value |
|-------|-------|
| Environment | production |
| Status | SUCCESS |
| Started | 2024-12-15 14:00:00 UTC |
| Completed | 2024-12-15 14:18:00 UTC |
| Duration | 18m |

## Pre-Deployment Checks

| Check | Status | Details |
|-------|--------|---------|
| Environment Variables | PASSED | 12/12 present |
| Build | PASSED | 45s, 423KB bundle |
| Migrations | PASSED | 2 pending, low risk |
| Security | PASSED | No issues |

## Deployment

| Step | Status | Details |
|------|--------|---------|
| Database Backup | DONE | backup_20241215_140500.sql |
| Migrations | APPLIED | 2 migrations |
| Application | DEPLOYED | https://aibotkit-abc123.vercel.app |

## Post-Deployment Tests

| Test | Status | Response Time |
|------|--------|---------------|
| Health Check | PASSED | 120ms |
| Auth Flow | PASSED | 350ms |
| Chatbot API | PASSED | 280ms |
| Chat Streaming | PASSED | 450ms |
| WordPress API | PASSED | 220ms |
| Stripe Webhook | PASSED | 180ms |

## Rollback Information

IF issues discovered, run:
```bash
/deploy-saas --rollback
```

Backup location: `backup_20241215_140500.sql`
Previous deployment: `https://aibotkit-xyz789.vercel.app`

## Next Steps

- [ ] Monitor error tracking (Sentry)
- [ ] Verify Stripe webhooks receiving events
- [ ] Test from WordPress plugin
```

## Resume Capability

When deployment fails, state is preserved:

```json
{
  "status": "failed",
  "currentPhase": "2.2",
  "failureReason": "Migration 0007 failed: column already exists",
  "resumeInstructions": "Fix migration conflict, then run /deploy-saas --resume"
}
```

User can fix the issue and resume:

```bash
/deploy-saas --resume

# Output:
===============================================================================
Resuming Deployment

Previous Status: FAILED at Phase 2.2 (Run Migrations)
Failure Reason: Migration 0007 failed: column already exists

Pre-checks already passed, skipping to Phase 2.2...
===============================================================================
```

## Integration with Other Commands

| Command | Integration Point |
|---------|-------------------|
| `/full-review` | Run before deployment to ensure code quality |
| `/sync-db` | Use for migration management |
| `/next-phase` | Phase 12 invokes deployment |

## Example Execution

### Successful Production Deployment

```
User: /deploy-saas production

===============================================================================
STARTING DEPLOYMENT: AI BotKit SaaS
Environment: production
===============================================================================

Phase 1.1: Environment Variables Check
- Checking 12 required variables...
- All variables present
PASSED

Phase 1.2: Build Validation
- Running pnpm build...
- Build successful (45s)
- Bundle size: 423KB
PASSED

Phase 1.3: Migration Check
- Found 2 pending migrations
- Risk assessment: LOW
PASSED

Phase 1.4: Security Scan
- Checking for secrets...
- Checking dependencies...
- No issues found
PASSED

===============================================================================
PRE-DEPLOYMENT CHECKS: ALL PASSED
Proceeding to deployment...
===============================================================================

Phase 2.1: Database Backup
- Creating backup...
- Saved: backup_20241215_140500.sql
DONE

Phase 2.2: Run Migrations
- Applying 0007_add_analytics...
- Applying 0008_add_flag...
- 2 migrations applied
DONE

Phase 2.3: Deploy Application
- Deploying to Vercel...
- Deployment URL: https://aibotkit-abc123.vercel.app
DONE

===============================================================================
DEPLOYMENT COMPLETE
Running post-deployment verification...
===============================================================================

Phase 3.1: Health Check
- GET /api/health: 200 OK (120ms)
PASSED

Phase 3.2: Functional Tests
- Auth flow: PASSED
- Chatbot API: PASSED
- Chat streaming: PASSED
PASSED

Phase 3.3: Integration Tests
- WordPress API: PASSED
- Stripe webhooks: PASSED
PASSED

===============================================================================
DEPLOYMENT SUCCESSFUL

New URL: https://aibotkit-abc123.vercel.app
Duration: 18 minutes
Report: reports/DEPLOYMENT_REPORT.md

Rollback available: /deploy-saas --rollback
===============================================================================
```

### Failed Deployment with Resume

```
User: /deploy-saas production

...
Phase 2.2: Run Migrations
- Applying 0007_add_analytics...
- ERROR: column "analytics_enabled" already exists

===============================================================================
DEPLOYMENT FAILED

Phase: 2.2 (Run Migrations)
Error: Migration 0007 failed: column already exists

To fix:
1. Check if column was manually added
2. Update migration or mark as applied
3. Run: /deploy-saas --resume

State saved to: _project_specs/.deploy-state.json
===============================================================================

User: /deploy-saas --resume

===============================================================================
RESUMING DEPLOYMENT

Previous failure: Phase 2.2 (Migration error)
Skipping completed phases...

Phase 2.2: Run Migrations (retry)
- Checking migration state...
- 0007 already applied (skipping)
- Applying 0008_add_flag...
- 1 migration applied
DONE

Phase 2.3: Deploy Application
...
```
