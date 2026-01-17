# Manual Test Generator Agent

Generate comprehensive manual test scenarios for AI BotKit that can be executed by QA testers or used as acceptance criteria.

## Purpose

This agent generates:
- User journey test scenarios
- Admin workflow tests
- Integration test scenarios
- Accessibility test cases (WCAG 2.1 AA)
- Cross-browser compatibility tests
- Mobile responsiveness tests

## When to Use

- Define acceptance criteria before development
- Create QA test documentation
- Prepare for UAT (User Acceptance Testing)
- `/fit-quality` test case generation phase

## What Gets Analyzed

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MANUAL TEST SOURCES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. SPECIFICATIONS                                                   │
│     specs/RECOVERED_SPECIFICATION.md                                 │
│     → Test scenarios for each requirement                            │
│                                                                      │
│  2. USER GUIDE                                                       │
│     docs/USER_GUIDE.md                                               │
│     → User journey test cases                                        │
│                                                                      │
│  3. API CONTRACTS                                                    │
│     specs/contracts/*.md                                             │
│     → API integration test scenarios                                 │
│                                                                      │
│  4. UI COMPONENTS                                                    │
│     Source files for UI features                                     │
│     → Accessibility and usability tests                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Test Categories

### 1. User Journey Tests

End-to-end scenarios that test complete user workflows.

### 2. Feature Tests

Individual feature functionality tests.

### 3. Integration Tests

Cross-system integration scenarios (SaaS ↔ WordPress).

### 4. Accessibility Tests

WCAG 2.1 AA compliance verification.

### 5. Security Tests

Manual security verification scenarios.

### 6. Performance Tests

User-perceived performance scenarios.

## Output Format

```markdown
# AI BotKit Manual Test Cases

## Test Plan Overview

| Category | Test Cases | Priority |
|----------|------------|----------|
| User Journeys | 12 | P0-P1 |
| Feature Tests | 45 | P0-P2 |
| Integration Tests | 8 | P0-P1 |
| Accessibility Tests | 15 | P1 |
| Security Tests | 10 | P0-P1 |
| Performance Tests | 5 | P1-P2 |

## Test Environment Requirements

| Environment | URL | Credentials |
|-------------|-----|-------------|
| Staging | staging.aibotkit.io | test@example.com / TestPass123 |
| WordPress Test | wp-test.aibotkit.io | admin / AdminPass123 |

---

## User Journey Tests

### UJ-001: New User Signup to First Chat

**Priority:** P0 (Critical)
**Estimated Time:** 10 minutes
**Prerequisites:** None (new user)

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Navigate to https://app.aibotkit.io | Landing page displays |
| 2 | Click "Sign Up" button | Signup form appears |
| 3 | Enter email: test+{timestamp}@example.com | Email field accepts input |
| 4 | Enter password: TestPass123! | Password field shows dots |
| 5 | Click "Create Account" | Account created, redirect to dashboard |
| 6 | Verify welcome modal appears | "Welcome to AI BotKit" modal shown |
| 7 | Click "Create Your First Chatbot" | Chatbot creation form opens |
| 8 | Enter name: "Test Bot" | Name field accepts input |
| 9 | Click "Create Chatbot" | Chatbot created, redirect to chatbot page |
| 10 | Click "Preview" button | Chat preview opens |
| 11 | Type "Hello" in chat input | Message appears in chat |
| 12 | Press Enter or click Send | AI response streams in |

**Pass Criteria:**
- [ ] User can complete signup without errors
- [ ] Dashboard loads within 3 seconds
- [ ] Chatbot is created successfully
- [ ] Chat preview shows AI response

**Notes:**
- Check for proper error messages if email already exists
- Verify password requirements are clearly shown

---

### UJ-002: Document Upload and RAG Testing

**Priority:** P0 (Critical)
**Estimated Time:** 15 minutes
**Prerequisites:** Existing chatbot

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Navigate to chatbot's Documents tab | Documents page loads |
| 2 | Click "Upload Document" | File picker opens |
| 3 | Select a PDF file (< 5MB) | File selected |
| 4 | Click "Upload" | Upload progress shown |
| 5 | Wait for processing | Status changes to "Processing" |
| 6 | Wait for completion | Status changes to "Completed" |
| 7 | Go to Preview tab | Chat preview opens |
| 8 | Ask question about document content | AI response includes document info |
| 9 | Verify context is from uploaded document | Response is accurate |

**Pass Criteria:**
- [ ] PDF uploads successfully
- [ ] Processing completes within 2 minutes
- [ ] Chat responses reference document content
- [ ] Character count updates correctly

---

### UJ-003: WordPress Plugin Connection

**Priority:** P0 (Critical)
**Estimated Time:** 15 minutes
**Prerequisites:** WordPress site with plugin installed

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Log into SaaS dashboard | Dashboard loads |
| 2 | Go to Settings > WordPress | WordPress settings page |
| 3 | Click "Connect WordPress Site" | Connection modal opens |
| 4 | Enter WordPress site URL | URL field accepts input |
| 5 | Click "Connect" | Connection initiated |
| 6 | In WordPress admin, go to AI BotKit | Plugin dashboard opens |
| 7 | Verify connection status shows "Connected" | Green connected badge |
| 8 | Select a chatbot to embed | Chatbot dropdown works |
| 9 | Save settings | Settings saved confirmation |
| 10 | Visit frontend of WordPress site | Chatbot widget appears |
| 11 | Test chat functionality | Chat works on WordPress |

**Pass Criteria:**
- [ ] Connection established without errors
- [ ] Token stored securely
- [ ] Chatbot appears on WordPress frontend
- [ ] Chat functionality works end-to-end

---

## Feature Tests

### FT-001: Chatbot Creation

**Priority:** P0
**FR Reference:** FR-010

**Preconditions:**
- User is logged in
- User has not reached chatbot limit

**Test Cases:**

| ID | Scenario | Steps | Expected | Priority |
|----|----------|-------|----------|----------|
| FT-001-01 | Create with valid name | Enter "My Bot", click Create | Chatbot created | P0 |
| FT-001-02 | Create with empty name | Leave name empty, click Create | Validation error shown | P0 |
| FT-001-03 | Create with long name | Enter 200 char name | Error: max 100 chars | P1 |
| FT-001-04 | Create with special chars | Name: "Bot <script>" | XSS prevented, name sanitized | P0 |
| FT-001-05 | Create at limit | Try to create when at plan limit | Error: upgrade plan | P1 |

---

### FT-002: Chat Streaming

**Priority:** P0
**FR Reference:** FR-001

**Preconditions:**
- Chatbot exists with documents

**Test Cases:**

| ID | Scenario | Steps | Expected | Priority |
|----|----------|-------|----------|----------|
| FT-002-01 | Normal message | Send "Hello" | Response streams word by word | P0 |
| FT-002-02 | Long message | Send 2000 char message | Message accepted, response returns | P1 |
| FT-002-03 | Empty message | Try to send empty | Send button disabled | P0 |
| FT-002-04 | Rapid messages | Send 5 messages quickly | Rate limit triggers appropriately | P1 |
| FT-002-05 | Network interruption | Disconnect during stream | Graceful error handling | P1 |

---

### FT-003: Stripe Subscription

**Priority:** P0
**FR Reference:** FR-030

**Preconditions:**
- User on Free plan
- Stripe test mode

**Test Cases:**

| ID | Scenario | Steps | Expected | Priority |
|----|----------|-------|----------|----------|
| FT-003-01 | Upgrade to Basic | Click Upgrade, complete Stripe checkout | Plan upgraded, limits increased | P0 |
| FT-003-02 | Cancel subscription | Go to Billing, click Cancel | Cancellation scheduled for period end | P0 |
| FT-003-03 | Failed payment | Use 4000000000000341 card | Payment fails, user notified | P0 |
| FT-003-04 | Downgrade plan | Switch from Essential to Basic | Downgrade scheduled for period end | P1 |

**Test Cards:**
- Success: 4242424242424242
- Decline: 4000000000000002
- Requires Auth: 4000002500003155

---

## Integration Tests

### IT-001: WordPress Post Sync

**Priority:** P0
**Prerequisites:** Connected WordPress site

**Test Steps:**

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Create new post in WordPress | Post saves |
| 2 | Publish the post | Post is published |
| 3 | Check SaaS documents list | Post appears as document |
| 4 | Verify content is indexed | Character count shows |
| 5 | Update post in WordPress | Post saves |
| 6 | Verify SaaS document updates | Updated timestamp changes |
| 7 | Delete post in WordPress | Post goes to trash |
| 8 | Verify document status | Document marked for removal |

---

## Accessibility Tests

### A11Y-001: Keyboard Navigation

**Priority:** P1
**WCAG:** 2.1.1 Keyboard

| ID | Element | Test | Expected | Pass |
|----|---------|------|----------|------|
| A11Y-001-01 | Chat input | Tab to input | Focus visible | [ ] |
| A11Y-001-02 | Send button | Tab to button, Enter | Message sends | [ ] |
| A11Y-001-03 | Navigation | Tab through nav | All links focusable | [ ] |
| A11Y-001-04 | Modals | Escape key | Modal closes | [ ] |
| A11Y-001-05 | Dropdowns | Arrow keys | Options navigate | [ ] |

---

### A11Y-002: Screen Reader

**Priority:** P1
**WCAG:** 4.1.2 Name, Role, Value

| ID | Element | Test | Expected | Pass |
|----|---------|------|----------|------|
| A11Y-002-01 | Chat messages | Read with NVDA | Role and content announced | [ ] |
| A11Y-002-02 | Buttons | Read with VoiceOver | Button purpose clear | [ ] |
| A11Y-002-03 | Forms | Read labels | Labels associated with inputs | [ ] |
| A11Y-002-04 | Alerts | New message | Live region announces | [ ] |

---

### A11Y-003: Color Contrast

**Priority:** P1
**WCAG:** 1.4.3 Contrast

| ID | Element | Foreground | Background | Ratio | Pass |
|----|---------|------------|------------|-------|------|
| A11Y-003-01 | Body text | #1F2937 | #FFFFFF | 12.6:1 | [ ] |
| A11Y-003-02 | Primary button | #FFFFFF | #10B981 | 4.5:1 | [ ] |
| A11Y-003-03 | Error text | #DC2626 | #FFFFFF | 4.5:1 | [ ] |
| A11Y-003-04 | Placeholder | #9CA3AF | #FFFFFF | 3:1* | [ ] |

*Placeholder text requires 4.5:1 as of WCAG 2.1

---

## Security Tests

### SEC-001: Authentication

**Priority:** P0

| ID | Scenario | Steps | Expected | Pass |
|----|----------|-------|----------|------|
| SEC-001-01 | Session expiry | Wait 24+ hours, refresh | Redirected to login | [ ] |
| SEC-001-02 | Invalid token | Manually change cookie | Access denied | [ ] |
| SEC-001-03 | CSRF protection | Submit form from external site | Request rejected | [ ] |
| SEC-001-04 | Password brute force | 10 failed attempts | Account locked/captcha | [ ] |

---

### SEC-002: Input Validation

**Priority:** P0

| ID | Scenario | Input | Expected | Pass |
|----|----------|-------|----------|------|
| SEC-002-01 | XSS in chat | `<script>alert(1)</script>` | Escaped, no execution | [ ] |
| SEC-002-02 | SQL injection | `'; DROP TABLE users;--` | Safely handled | [ ] |
| SEC-002-03 | Path traversal | `../../etc/passwd` | Rejected | [ ] |

---

## Performance Tests

### PERF-001: Page Load Times

**Priority:** P1

| Page | Target | Acceptable | Method |
|------|--------|------------|--------|
| Dashboard | < 2s | < 4s | First Contentful Paint |
| Chat Preview | < 1s | < 2s | Time to Interactive |
| Documents List | < 2s | < 4s | Largest Contentful Paint |

---

### PERF-002: Chat Response Times

**Priority:** P1

| Scenario | Target | Acceptable |
|----------|--------|------------|
| First token | < 1s | < 2s |
| Complete response (50 words) | < 5s | < 10s |
| With 10 documents | < 2s first token | < 4s |

---

## Cross-Browser Testing

| Browser | Version | Desktop | Mobile | Priority |
|---------|---------|---------|--------|----------|
| Chrome | Latest | Required | Required | P0 |
| Firefox | Latest | Required | - | P1 |
| Safari | Latest | Required | Required | P1 |
| Edge | Latest | Required | - | P2 |

---

## Test Execution Checklist

### Pre-Test
- [ ] Test environment accessible
- [ ] Test accounts created
- [ ] Test data prepared
- [ ] Browser/device ready

### Post-Test
- [ ] All P0 tests passed
- [ ] Bugs logged with screenshots
- [ ] Test results documented
- [ ] Stakeholders notified

---

## Bug Report Template

```
**Bug ID:** BUG-XXX
**Test Case:** [Reference]
**Severity:** Critical/High/Medium/Low
**Environment:** [Browser, OS, URL]

**Steps to Reproduce:**
1.
2.
3.

**Expected Result:**


**Actual Result:**


**Screenshots/Video:**


**Notes:**

```
```

## Integration

This agent is invoked by:
- `/fit-quality` command (Phase 6)
- `/next-phase` command (Phase 5.7)

## Related Agents

- `test-case-generator` - Generates automated test cases
- `e2e-test-generator` - Implements Playwright tests
- `accessibility-guardian` - Reviews accessibility issues
