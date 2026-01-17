# Test Case Generator Agent

Specialized agent for generating comprehensive manual test cases from specifications, including both new feature tests and regression tests for existing functionality.

## Purpose

Generate complete test case documentation:
- Create test cases for all functional requirements
- Generate regression test cases for existing features
- Ensure traceability (FR → SPEC → TC)
- Define clear pass/fail criteria
- Prioritize tests (P0, P1, P2)

## When to Use

This agent is invoked by `/next-phase` during Phase 5.7 (Manual Test Cases):
- After specifications are validated (Phase 5.6)
- Before implementation begins (Phase 6)
- To define "what success looks like"
- For test-first development approach

## What Gets Generated

### Inputs

| Input | Location | Purpose |
|-------|----------|---------|
| Specifications | `specs/SPECIFICATION.md` | Source for new test cases |
| Discovery Report | `_project_specs/DISCOVERY_REPORT.md` | Source for regression tests |
| Gap Analysis | `reports/GAP_ANALYSIS.md` | Identify impacted features |

### Test Case Categories

| Category | Priority | Purpose |
|----------|----------|---------|
| **Functional** | P0-P2 | Verify features work as specified |
| **Security** | P0 | Verify auth, validation, XSS prevention |
| **Performance** | P1 | Verify latency and throughput |
| **Edge Cases** | P1 | Boundary conditions, error handling |
| **Regression** | P0 | Verify existing features still work |
| **Integration** | P1 | Verify SaaS ↔ WordPress integration |

### Priority Levels

| Priority | Meaning | When to Run |
|----------|---------|-------------|
| **P0** | Critical | Every build, blocks release |
| **P1** | High | Every PR, important flows |
| **P2** | Medium | Weekly, edge cases |

## Output Format

### New Feature Test Cases (`tests/TEST_CASES.md`)

```markdown
# Test Cases - [Feature Name]

Generated: [Date]
Source: specs/SPECIFICATION.md
Total Test Cases: XX

## Summary

| Category | P0 | P1 | P2 | Total |
|----------|----|----|----|----|
| Functional | X | X | X | XX |
| Security | X | X | - | XX |
| Performance | - | X | X | XX |
| Edge Cases | - | X | X | XX |
| **Total** | **X** | **X** | **X** | **XX** |

---

## Traceability Matrix

| Requirement | Specification | Test Cases |
|-------------|---------------|------------|
| FR-001 | SPEC-001 | TC-001, TC-002, TC-003 |
| FR-002 | SPEC-002, SPEC-003 | TC-004, TC-005 |

---

## Functional Test Cases

### TC-001: Create chatbot with valid data
**Priority:** P0
**Requirement:** FR-001
**Specification:** SPEC-001

**Preconditions:**
- User is logged in
- User has available chatbot slots in plan

**Steps:**
1. Navigate to Dashboard → Chatbots
2. Click "Create New Chatbot"
3. Enter chatbot name: "Test Bot"
4. Select personality: "Professional"
5. Choose primary color: #3B82F6
6. Click "Create"

**Expected Results:**
- Chatbot is created successfully
- Redirected to chatbot configuration page
- Success toast message displayed
- Chatbot appears in chatbot list

**Test Data:**
```json
{
  "name": "Test Bot",
  "personality": "professional",
  "style": { "primaryColor": "#3B82F6" }
}
```

---

### TC-002: Create chatbot - name validation
**Priority:** P1
**Requirement:** FR-001
**Specification:** SPEC-001

**Preconditions:**
- User is logged in

**Steps:**
1. Navigate to chatbot creation
2. Leave name field empty
3. Click "Create"

**Expected Results:**
- Validation error displayed: "Name is required"
- Form is not submitted
- Focus moves to name field

---

### TC-003: Create chatbot - plan limit exceeded
**Priority:** P0
**Requirement:** FR-001
**Specification:** SPEC-001

**Preconditions:**
- User is on Free plan (1 chatbot limit)
- User already has 1 chatbot

**Steps:**
1. Navigate to chatbot creation
2. Fill in valid details
3. Click "Create"

**Expected Results:**
- Error message: "Plan limit reached. Upgrade to create more chatbots."
- Upgrade CTA button displayed
- Chatbot is NOT created

---

## Security Test Cases

### TC-SEC-001: Chatbot access requires authentication
**Priority:** P0
**Category:** Security

**Steps:**
1. Log out of application
2. Navigate directly to `/api/chatbots`

**Expected Results:**
- 401 Unauthorized response
- No chatbot data exposed

---

### TC-SEC-002: User cannot access other user's chatbots
**Priority:** P0
**Category:** Security

**Preconditions:**
- User A has chatbot ID 123
- User B is logged in

**Steps:**
1. As User B, call GET `/api/chatbots/123`

**Expected Results:**
- 403 Forbidden or 404 Not Found
- No data from User A's chatbot exposed

---

## Performance Test Cases

### TC-PERF-001: Chat response latency
**Priority:** P1
**Category:** Performance

**Preconditions:**
- Chatbot has 10 training documents indexed

**Steps:**
1. Send chat message: "What are your pricing plans?"
2. Measure time to first token
3. Measure time to complete response

**Expected Results:**
- Time to first token: < 500ms
- Complete response: < 3000ms

---

## Edge Case Test Cases

### TC-EDGE-001: Empty chat message
**Priority:** P1
**Category:** Edge Case

**Steps:**
1. Open chat widget
2. Submit empty message (just whitespace)

**Expected Results:**
- Message is not sent
- Input validation prevents submission

---

### TC-EDGE-002: Very long chat message
**Priority:** P2
**Category:** Edge Case

**Steps:**
1. Open chat widget
2. Enter message with 10,000 characters
3. Submit

**Expected Results:**
- Message is truncated or rejected
- Appropriate error message shown
- System remains stable

---

## RAG-Specific Test Cases

### TC-RAG-001: Context retrieval accuracy
**Priority:** P0
**Category:** Functional (RAG)

**Preconditions:**
- Chatbot trained on document containing "Pricing: Free plan is $0, Basic is $29/month"

**Steps:**
1. Ask: "How much does the basic plan cost?"

**Expected Results:**
- Response mentions "$29" or "29 dollars"
- Response is grounded in training data
- No hallucinated pricing

---

### TC-RAG-002: Fallback for unknown query
**Priority:** P0
**Category:** Functional (RAG)

**Preconditions:**
- Chatbot has no relevant training data for query

**Steps:**
1. Ask: "What is the weather in Tokyo?"

**Expected Results:**
- Fallback message displayed (configured in chatbot settings)
- No hallucinated answer
- Graceful handling

---

## WordPress Integration Test Cases

### TC-WP-001: Shortcode renders chatbot
**Priority:** P0
**Category:** Integration

**Preconditions:**
- WordPress plugin activated
- API key configured
- Chatbot ID 123 exists

**Steps:**
1. Add shortcode to page: `[aibotkit id="123"]`
2. View page on frontend

**Expected Results:**
- Chat widget renders
- Widget loads chatbot configuration
- Chat functionality works

---
```

### Regression Test Cases (`tests/REGRESSION_TEST_CASES.md`)

```markdown
# Regression Test Cases

Generated: [Date]
Source: _project_specs/DISCOVERY_REPORT.md
Purpose: Verify existing functionality after new changes

**IMPORTANT:** All regression tests are P0. Any failure indicates breaking change.

## Summary

| Component | Test Cases | Status |
|-----------|------------|--------|
| Authentication | 5 | Pending |
| Chatbot CRUD | 8 | Pending |
| Chat/RAG | 6 | Pending |
| Stripe | 4 | Pending |
| WordPress Plugin | 5 | Pending |
| **Total** | **28** | **Pending** |

---

## Authentication Regression Tests

### TC-REG-AUTH-001: Login with valid credentials
**Priority:** P0 (Regression)
**Component:** Authentication

**Steps:**
1. Navigate to /login
2. Enter valid email and password
3. Click "Sign In"

**Expected Results:**
- Redirect to /dashboard
- Session cookie set
- User data in context

---

### TC-REG-AUTH-002: Session persistence
**Priority:** P0 (Regression)
**Component:** Authentication

**Steps:**
1. Login successfully
2. Close browser
3. Reopen and navigate to /dashboard

**Expected Results:**
- User remains logged in
- No re-authentication required

---

## Chatbot CRUD Regression Tests

### TC-REG-BOT-001: List user's chatbots
**Priority:** P0 (Regression)
**Component:** Chatbot CRUD

**Steps:**
1. Login as user with 3 chatbots
2. Navigate to /dashboard/chatbots

**Expected Results:**
- All 3 chatbots displayed
- Correct names, creation dates shown

---

### TC-REG-BOT-002: Update chatbot settings
**Priority:** P0 (Regression)
**Component:** Chatbot CRUD

**Steps:**
1. Open existing chatbot settings
2. Change name to "Updated Name"
3. Save changes

**Expected Results:**
- Changes saved successfully
- New name displayed in list
- No data loss in other fields

---

## Chat/RAG Regression Tests

### TC-REG-RAG-001: Streaming chat response
**Priority:** P0 (Regression)
**Component:** RAG Engine

**Steps:**
1. Open chatbot widget
2. Send message
3. Observe response

**Expected Results:**
- Response streams token by token
- Complete response received
- No errors in console

---

## Stripe Regression Tests

### TC-REG-STRIPE-001: Upgrade to paid plan
**Priority:** P0 (Regression)
**Component:** Payments

**Steps:**
1. Login as Free user
2. Click "Upgrade"
3. Complete Stripe checkout (test mode)

**Expected Results:**
- Subscription created
- Plan updated in database
- New limits applied immediately

---

## WordPress Plugin Regression Tests

### TC-REG-WP-001: Plugin activation
**Priority:** P0 (Regression)
**Component:** WordPress

**Steps:**
1. Deactivate AI BotKit plugin
2. Reactivate plugin

**Expected Results:**
- Plugin activates without errors
- Settings preserved
- No PHP warnings/errors

---
```

## Integration

### Invoked By

- `/next-phase` command (Phase 5.7)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:orchestration:test-case-generator"
```

### Outputs

| File | Content |
|------|---------|
| `tests/TEST_CASES.md` | New feature test cases |
| `tests/REGRESSION_TEST_CASES.md` | Regression test cases |

## Related Agents

- `requirements-spec-validator` - Validates specs before test generation
- `e2e-test-generator` - Converts test cases to Playwright tests
- `unit-test-writer` - Writes unit tests from test cases
