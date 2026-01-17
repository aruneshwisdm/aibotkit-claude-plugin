# E2E Test Runner Agent

Specialized agent for executing E2E tests and managing the test-fix loop until all tests pass. This agent orchestrates test execution and coordinates with bug-fixer for failures.

## Purpose

Execute and manage E2E test cycles:
- Run Playwright tests across browsers
- Parse and categorize failures
- Prioritize regression failures over new feature failures
- Coordinate with bug-fixer agent for fixes
- Loop until 100% pass rate or max iterations

## When to Use

This agent is invoked by `/next-phase` during Phase 8 (Test & Fix Loop):
- After tests are written (Phase 7)
- As part of continuous integration
- For regression testing before release
- When validating bug fixes

## Test Execution Strategy

### Execution Order

| Priority | Test Category | Rationale |
|----------|---------------|-----------|
| 1 | Regression (P0) | Must not break existing features |
| 2 | New Feature (P0) | Critical new functionality |
| 3 | Regression (P1) | Important existing flows |
| 4 | New Feature (P1) | Important new functionality |
| 5 | All P2 | Edge cases and nice-to-haves |

### Browser Matrix

| Browser | When to Run |
|---------|-------------|
| Chromium | Always (primary) |
| Firefox | CI only, release |
| WebKit | CI only, release |

## Execution Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                  E2E TEST RUNNER WORKFLOW                    │
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │   RUN ALL   │───►│   PARSE     │───►│  CATEGORIZE │      │
│  │   TESTS     │    │  RESULTS    │    │  FAILURES   │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│                                              │               │
│                                              ▼               │
│                            ┌─────────────────────────────┐  │
│                            │  ANY FAILURES?              │  │
│                            └─────────────────────────────┘  │
│                               │               │              │
│                           NO  │               │ YES          │
│                               ▼               ▼              │
│                    ┌──────────────┐   ┌──────────────┐      │
│                    │   SUCCESS    │   │   INVOKE     │      │
│                    │   REPORT     │   │   BUG-FIXER  │      │
│                    └──────────────┘   └──────────────┘      │
│                                              │               │
│                                              ▼               │
│                                    ┌──────────────┐         │
│                                    │   RE-RUN     │         │
│                                    │   FAILED     │◄────┐   │
│                                    └──────────────┘     │   │
│                                           │             │   │
│                                           ▼             │   │
│                                   ┌──────────────┐      │   │
│                                   │   STILL      │──YES─┘   │
│                                   │  FAILING?    │          │
│                                   └──────────────┘          │
│                                           │ NO              │
│                                           ▼                 │
│                                   ┌──────────────┐          │
│                                   │   CONTINUE   │          │
│                                   │   TO NEXT    │          │
│                                   └──────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

## Execution Commands

### Run All Tests

```bash
# Run all E2E tests
pnpm test:e2e

# Run with specific config
pnpm playwright test --config=playwright.config.ts

# Run in CI mode (all browsers)
pnpm playwright test --project=chromium --project=firefox --project=webkit
```

### Run Specific Categories

```bash
# Run only regression tests
pnpm playwright test --grep "@regression"

# Run only P0 tests
pnpm playwright test --grep "@P0"

# Run specific file
pnpm playwright test tests/e2e/auth.spec.ts

# Run failed tests only
pnpm playwright test --last-failed
```

### Debug Mode

```bash
# Run with UI
pnpm playwright test --ui

# Run headed
pnpm playwright test --headed

# Debug specific test
pnpm playwright test --debug tests/e2e/auth.spec.ts
```

## Failure Analysis

### Parse Playwright Output

```typescript
interface TestResult {
  name: string;
  file: string;
  status: 'passed' | 'failed' | 'skipped';
  duration: number;
  error?: {
    message: string;
    stack: string;
    screenshot?: string;
  };
  tags: string[];  // @regression, @P0, etc.
}

function parsePlaywrightResults(jsonReport: string): TestResult[] {
  const report = JSON.parse(jsonReport);
  return report.suites.flatMap(suite =>
    suite.specs.map(spec => ({
      name: spec.title,
      file: spec.file,
      status: spec.tests[0].status,
      duration: spec.tests[0].duration,
      error: spec.tests[0].error,
      tags: extractTags(spec.title),
    }))
  );
}
```

### Categorize Failures

| Category | Identification | Priority |
|----------|----------------|----------|
| **Regression** | Tag `@regression` or file in `regression/` | Highest |
| **Auth** | File `auth.spec.ts` or tag `@auth` | Critical |
| **Payment** | File `payment.spec.ts` or tag `@payment` | Critical |
| **RAG** | File contains `rag` or tag `@rag` | High |
| **WordPress** | File `wordpress.spec.ts` | High |
| **Other** | All else | Normal |

## Output Format

### Test Execution Report

```markdown
# E2E Test Execution Report

Generated: [Date]
Run ID: run-20241215-143052
Duration: 5m 32s

## Summary

| Metric | Value |
|--------|-------|
| Total Tests | 45 |
| Passed | 42 |
| Failed | 3 |
| Skipped | 0 |
| Pass Rate | 93.3% |

## Browser Results

| Browser | Passed | Failed | Duration |
|---------|--------|--------|----------|
| Chromium | 42 | 3 | 2m 15s |
| Firefox | 45 | 0 | 2m 45s |
| WebKit | 45 | 0 | 3m 02s |

---

## Failed Tests

### 1. TC-REG-AUTH-002: Session persistence
**File:** tests/e2e/auth.spec.ts:45
**Category:** REGRESSION (P0)
**Browser:** Chromium

**Error:**
```
Expected: /dashboard
Received: /login

Timeout waiting for URL to match
```

**Screenshot:** [Link to screenshot]

**Analysis:**
- Session cookie not persisting across browser restart
- Likely regression from recent auth changes

**Suggested Fix:**
- Check session cookie settings (httpOnly, secure, sameSite)
- Verify session persistence in middleware

---

### 2. TC-001: Create chatbot with valid data
**File:** tests/e2e/chatbots.spec.ts:23
**Category:** NEW FEATURE (P0)
**Browser:** Chromium

**Error:**
```
Element [data-testid="submit-chatbot"] not found
```

**Screenshot:** [Link to screenshot]

**Analysis:**
- Test selector may be incorrect
- Or submit button not rendered

**Suggested Fix:**
- Verify data-testid attribute on submit button
- Check for conditional rendering logic

---

### 3. TC-PERF-001: Chat response latency
**File:** tests/e2e/performance.spec.ts:12
**Category:** PERFORMANCE (P1)
**Browser:** Chromium

**Error:**
```
Expected time to first token: < 500ms
Actual: 1250ms
```

**Analysis:**
- Performance degradation detected
- May be cold start or network issue

**Suggested Fix:**
- Check Pinecone connection pooling
- Review RAG engine initialization

---

## Fix Priority Queue

| Order | Test | Category | Action |
|-------|------|----------|--------|
| 1 | TC-REG-AUTH-002 | Regression | IMMEDIATE FIX |
| 2 | TC-001 | New Feature | Fix required |
| 3 | TC-PERF-001 | Performance | Investigate |

---

## Next Steps

1. **Invoke bug-fixer agent** for TC-REG-AUTH-002 (regression)
2. **Invoke bug-fixer agent** for TC-001 (new feature)
3. **Investigate** TC-PERF-001 performance issue
4. **Re-run failed tests** after fixes
5. **Full test run** when all individual fixes pass
```

### Loop Status Report

```markdown
# Test-Fix Loop Status

## Iteration Summary

| Iteration | Passed | Failed | Fixed This Round |
|-----------|--------|--------|------------------|
| 1 | 42/45 | 3 | - |
| 2 | 44/45 | 1 | 2 |
| 3 | 45/45 | 0 | 1 |

## Status: ✅ COMPLETE

All 45 tests passing after 3 iterations.

### Fixes Applied

| Test | Issue | Fix | Iteration |
|------|-------|-----|-----------|
| TC-REG-AUTH-002 | Cookie settings | Set sameSite: 'lax' | 2 |
| TC-001 | Missing testid | Added data-testid | 2 |
| TC-PERF-001 | Cold start | Added connection warmup | 3 |

### Time Spent

| Phase | Duration |
|-------|----------|
| Initial Run | 5m 32s |
| Fix Iteration 1 | 12m |
| Re-run 1 | 5m 45s |
| Fix Iteration 2 | 8m |
| Re-run 2 | 5m 30s |
| **Total** | **36m 47s** |
```

## Integration

### Invoked By

- `/next-phase` command (Phase 8)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:testing:e2e-test-runner"
```

### Loop Behavior

```
WHILE failed_tests > 0 AND iteration < MAX_ITERATIONS:
  1. Run all tests (or failed only after first run)
  2. Parse results
  3. For each failure:
     a. Categorize (regression/new/performance)
     b. Prioritize (regression first)
     c. Invoke bug-fixer agent
     d. Verify fix
  4. Increment iteration

IF failed_tests == 0:
  PASS: Proceed to Phase 9
ELSE:
  FAIL: Manual intervention required
```

## Related Agents

- `e2e-test-generator` - Creates the tests
- `bug-fixer` - Fixes failing tests
- `code-fixer` - Fixes code quality issues
