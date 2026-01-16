# /full-review

Orchestrate a comprehensive code review for the AI BotKit monorepo using multiple specialized agents working in parallel. This command supports both the **Next.js SaaS application** and the **WordPress plugin** components.

## Context

AI BotKit is a monorepo containing:
- **saas/**: Next.js 16 SaaS application with RAG-powered chatbots
- **wordpress-plugin/**: WordPress plugin for embedding chatbots

A comprehensive review ensures:
- Next.js/React best practices compliance
- RAG engine correctness and efficiency
- Database schema integrity (Drizzle/PostgreSQL)
- Stripe integration security
- WordPress coding standards (WPCS)
- Security best practices (OWASP Top 10)
- API consistency between SaaS and WordPress plugin

## Component Detection

The command auto-detects which components to review based on:

| Component | Detection Criteria |
|-----------|-------------------|
| **SaaS (Next.js)** | `saas/package.json`, `saas/src/app/` directory |
| **WordPress Plugin** | `wordpress-plugin/ai-botkit-for-lead-generation.php` |

You can also explicitly specify the component:
```bash
/full-review --component saas
/full-review --component wordpress
/full-review --component all
```

## Your Task

Orchestrate a comprehensive code review by launching multiple specialized agents in parallel, aggregating their results, and providing a unified, prioritized report.

## Orchestration Strategy

### Phase 0: Component Detection

Before launching agents, detect which components exist:

```
1. Check for SaaS component:
   - saas/package.json exists
   - saas/src/app/ directory exists
   - saas/src/lib/ai/rag-engine.ts exists

2. Check for WordPress Plugin component:
   - wordpress-plugin/ai-botkit-for-lead-generation.php exists
   - wordpress-plugin/includes/ directory exists

3. Determine review scope based on detection or user flag
```

### Phase 1: Launch Agents in Parallel

Launch component-specific agents simultaneously to analyze the codebase:

#### SaaS Component Agents

1. **`nextjs-standards-reviewer`**
   - Purpose: Check Next.js 16 App Router patterns
   - Focus: Server/Client components, data fetching, routing
   - Output: Next.js best practices violations

2. **`rag-engine-reviewer`**
   - Purpose: Review RAG implementation correctness
   - Focus: Pinecone integration, context retrieval, prompt construction
   - Output: RAG flow issues and optimization opportunities

3. **`drizzle-schema-reviewer`**
   - Purpose: Database schema and query review
   - Focus: Schema design, relations, migrations, query efficiency
   - Output: Database design issues and recommendations

4. **`stripe-integration-reviewer`**
   - Purpose: Payment integration security review
   - Focus: Webhook handling, subscription lifecycle, PCI compliance
   - Output: Payment security issues

5. **`saas-security-auditor`**
   - Purpose: Security audit for Next.js application
   - Focus: Authentication, authorization, API security, XSS, CSRF
   - Output: Security vulnerabilities by severity

#### WordPress Plugin Agents

6. **`wordpress-standards-reviewer`**
   - Purpose: Check WordPress coding standards
   - Focus: WPCS compliance, hooks usage, naming conventions
   - Output: WordPress standards violations

7. **`wordpress-security-auditor`**
   - Purpose: WordPress-specific security audit
   - Focus: Nonce verification, capability checks, SQL injection, XSS
   - Output: Security vulnerabilities by severity

8. **`api-integration-reviewer`**
   - Purpose: Review SaaS-Plugin API integration
   - Focus: REST API calls, token handling, error handling
   - Output: Integration issues and recommendations

#### Cross-Component Agents (Always Run)

9. **`accessibility-guardian`**
   - Purpose: Accessibility compliance check
   - Focus: WCAG 2.1 AA standards, keyboard navigation, screen readers
   - Output: Accessibility issues and fixes

10. **`architecture-reviewer`**
    - Purpose: Architecture and code quality
    - Focus: Design patterns, maintainability, coupling, complexity
    - Output: Architectural improvements

### Phase 2: Aggregate Results

1. **Collect all agent outputs**
   - Wait for all agents to complete
   - Handle any agent failures gracefully
   - Note which agents completed successfully

2. **Deduplicate findings**
   - Identify similar issues reported by multiple agents
   - Merge duplicate findings
   - Cross-reference related issues

3. **Categorize by severity**
   - **Critical**: Security vulnerabilities, data loss risks, payment issues
   - **High**: Major bugs, significant security/performance issues
   - **Medium**: Standards violations, minor bugs, performance issues
   - **Low**: Code style, minor improvements, suggestions

4. **Prioritize findings**
   - Sort by severity and impact
   - Group related issues
   - Identify quick wins

### Phase 3: Generate Comprehensive Report

Create a unified report with:

1. **Executive Summary**
   - Overall assessment
   - Total issues by severity
   - Top 3 priority areas
   - Deployment recommendation (ready/not ready)

2. **Critical & High Priority Issues**
   - Detailed description of each issue
   - File location and line numbers
   - Impact assessment
   - Recommended fix with code examples
   - Estimated time to fix

3. **Component-Specific Sections**
   - SaaS findings grouped by category
   - WordPress Plugin findings grouped by category

4. **Next Steps**
   - Prioritized action items
   - Testing recommendations

## Output Format

```markdown
# Comprehensive Code Review Report

## Executive Summary

**Project:** AI BotKit
**Components Reviewed:** [SaaS, WordPress Plugin]
**Review Date:** [Date]
**Reviewed By:** AI BotKit Engineering Plugin (Multi-Agent Review)
**Status:** [READY TO DEPLOY | MINOR ISSUES | CRITICAL ISSUES FOUND]

### Overview
[2-3 sentence summary of overall code quality]

### Component Health

| Component | Score | Status |
|-----------|-------|--------|
| SaaS (Next.js) | XX/100 | Status |
| WordPress Plugin | XX/100 | Status |
| RAG Engine | XX/100 | Status |
| Database Layer | XX/100 | Status |
| Security | XX/100 | Status |

### Issues Summary

| Severity | Count | Status |
|----------|-------|--------|
| Critical | X | Must fix before deployment |
| High | X | Should fix before deployment |
| Medium | X | Address soon |
| Low | X | Nice to have |

### Top Priority Areas
1. [Area 1] - [Issue count] issues
2. [Area 2] - [Issue count] issues
3. [Area 3] - [Issue count] issues

### Deployment Recommendation
[APPROVED / APPROVED WITH CONDITIONS / NOT APPROVED]

[Explanation of recommendation]

---

## Critical Issues (Must Fix Immediately)

### 1. [Issue Title]
**Agent:** [agent-name]
**Severity:** Critical
**Component:** [SaaS | WordPress Plugin]
**File:** path/to/file:line_number

**Description:**
[Detailed description of the issue]

**Impact:**
- [Impact point 1]
- [Impact point 2]

**Current Code:**
```[language]
// Problematic code
```

**Recommended Fix:**
```[language]
// Fixed code
```

**Estimated Time to Fix:** X minutes/hours

---

[Repeat for each critical issue]

---

## High Priority Issues

[Same format as critical issues]

---

## SaaS Component Findings

### Next.js Standards
**Agent:** nextjs-standards-reviewer
**Issues Found:** X

[List of issues with details]

### RAG Engine
**Agent:** rag-engine-reviewer
**Issues Found:** X

[List of issues with details]

### Database Layer
**Agent:** drizzle-schema-reviewer
**Issues Found:** X

[List of issues with details]

### Stripe Integration
**Agent:** stripe-integration-reviewer
**Issues Found:** X

[List of issues with details]

---

## WordPress Plugin Findings

### WordPress Standards
**Agent:** wordpress-standards-reviewer
**Issues Found:** X

[List of issues with details]

### Security
**Agent:** wordpress-security-auditor
**Issues Found:** X

[List of issues with details]

---

## Agent Reports Summary

| Agent | Status | Issues | Duration |
|-------|--------|--------|----------|
| nextjs-standards-reviewer | Status | X | Xs |
| rag-engine-reviewer | Status | X | Xs |
| drizzle-schema-reviewer | Status | X | Xs |
| stripe-integration-reviewer | Status | X | Xs |
| saas-security-auditor | Status | X | Xs |
| wordpress-standards-reviewer | Status | X | Xs |
| wordpress-security-auditor | Status | X | Xs |
| api-integration-reviewer | Status | X | Xs |
| accessibility-guardian | Status | X | Xs |
| architecture-reviewer | Status | X | Xs |

---

## Next Steps

### Immediate (Before Deployment)
- [ ] Fix X critical issues
- [ ] Fix X high priority issues
- [ ] Re-run security scan

### Short Term (Next Sprint)
- [ ] Address medium priority issues
- [ ] Improve test coverage
- [ ] Documentation updates

---

## Metrics

**Review Statistics:**
- Total files reviewed: XX
- Total lines of code: XXXX
- Total issues found: XX
- Review duration: X minutes
- Agents used: X

**Code Quality Score:** XX/100
```

## Important Notes

1. **Parallel Execution Benefits**
   - Up to 10 agents running in parallel = ~10x faster than sequential
   - Typical review time: 2-3 minutes vs 20-30 minutes

2. **Handling Agent Failures**
   - If an agent fails, note it in the report
   - Continue with other agents
   - Don't block entire review on one failure

3. **Priority Guidelines**
   - Critical: Data loss, security vulnerabilities, payment issues, site-breaking bugs
   - High: Significant security/performance issues, major standards violations
   - Medium: Minor bugs, standards violations, performance opportunities
   - Low: Code style, documentation, nice-to-have improvements

4. **Deployment Decisions**
   - Block deployment if critical issues exist
   - Allow deployment with high priority issues documented
   - Approve deployment with only medium/low issues

## Related Commands

- `/next-phase` - Phased development workflow
- `/deploy-saas` - Deployment checklist
- `/test-rag` - RAG engine testing

## Success Criteria

A successful /full-review execution:
- All applicable agents completed (or failures handled gracefully)
- Results aggregated and deduplicated
- Issues prioritized correctly
- Clear deployment recommendation provided
- Actionable next steps defined
- Report is comprehensive yet readable

Now, orchestrate the comprehensive code review based on the user's codebase.
