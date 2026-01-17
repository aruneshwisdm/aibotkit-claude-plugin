# Code Fixer Agent

Specialized agent for fixing code quality issues identified during code review. Works with the `/full-review` results to address standards violations, security issues, and architectural concerns.

## Purpose

Fix code quality issues from review:
- Address code review findings
- Fix security vulnerabilities
- Resolve standards violations
- Improve architecture issues
- Loop until quality threshold met

## When to Use

This agent is invoked by `/next-phase` during Phase 10 (Fix & Validate Loop):
- After code review (Phase 9)
- When `/full-review` reports issues
- For addressing PR review comments
- For continuous code quality improvement

## Fix Categories

### Priority Matrix

| Category | Severity | Must Fix Before Deploy |
|----------|----------|------------------------|
| Security | Critical/High | Yes |
| Performance | High | Yes |
| Standards | Medium | Recommended |
| Architecture | Medium | Recommended |
| Style | Low | Optional |

### Issue Types

| Type | Examples | Fix Approach |
|------|----------|--------------|
| **Security** | XSS, SQL injection, auth bypass | Immediate, minimal change |
| **Performance** | N+1 queries, missing indexes | Optimize query/component |
| **Standards** | WPCS, Next.js patterns | Refactor to follow patterns |
| **Architecture** | SRP violation, tight coupling | Gradual refactoring |
| **Accessibility** | Missing ARIA, contrast | Add required attributes |

## Fix Patterns

### 1. Security Fixes

#### XSS Prevention (React/Next.js)

```typescript
// ISSUE: Dangerously setting HTML
// Before
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// AFTER: Sanitize or use safe rendering
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />

// OR: Use text content
<div>{userInput}</div>
```

#### XSS Prevention (WordPress)

```php
// ISSUE: Unescaped output
// Before
echo $user_input;

// AFTER: Escape appropriately
echo esc_html($user_input);       // For HTML content
echo esc_attr($user_input);       // For attributes
echo esc_url($user_input);        // For URLs
echo wp_kses_post($user_input);   // For allowed HTML
```

#### SQL Injection (WordPress)

```php
// ISSUE: Direct variable in query
// Before
$wpdb->query("SELECT * FROM {$table} WHERE id = {$_GET['id']}");

// AFTER: Use prepared statements
$wpdb->query($wpdb->prepare(
    "SELECT * FROM {$table} WHERE id = %d",
    absint($_GET['id'])
));
```

#### CSRF Protection (WordPress)

```php
// ISSUE: Missing nonce verification
// Before
function handle_form_submission() {
    update_option('my_option', $_POST['value']);
}

// AFTER: Add nonce verification
function handle_form_submission() {
    if (!wp_verify_nonce($_POST['_wpnonce'], 'my_action')) {
        wp_die('Security check failed');
    }
    update_option('my_option', sanitize_text_field($_POST['value']));
}
```

### 2. Performance Fixes

#### N+1 Query Fix (Drizzle)

```typescript
// ISSUE: N+1 query pattern
// Before
const chatbots = await db.select().from(chatbotsTable);
for (const chatbot of chatbots) {
    chatbot.documents = await db.select().from(documents)
        .where(eq(documents.chatbotId, chatbot.id));
}

// AFTER: Use relations
const chatbots = await db.query.chatbots.findMany({
    with: { documents: true },
});
```

#### Missing Index Fix

```typescript
// ISSUE: No index on frequently queried column
// Before
export const documents = pgTable('aibotkit_documents', {
    chatbotId: integer('chatbot_id').references(() => chatbots.id),
});

// AFTER: Add index
export const documents = pgTable('aibotkit_documents', {
    chatbotId: integer('chatbot_id').references(() => chatbots.id),
}, (table) => ({
    chatbotIdIdx: index('idx_documents_chatbot_id').on(table.chatbotId),
}));
```

#### React Re-render Fix

```typescript
// ISSUE: Unnecessary re-renders
// Before
function ChatList({ chatbots }) {
    const filtered = chatbots.filter(c => c.active); // Runs every render
    return filtered.map(c => <ChatItem chatbot={c} />);
}

// AFTER: Memoize
function ChatList({ chatbots }) {
    const filtered = useMemo(
        () => chatbots.filter(c => c.active),
        [chatbots]
    );
    return filtered.map(c => <ChatItem chatbot={c} />);
}
```

### 3. Standards Fixes

#### Next.js App Router Pattern

```typescript
// ISSUE: Using client component for server-only operation
// Before
'use client';
export default function ChatbotsPage() {
    const [chatbots, setChatbots] = useState([]);
    useEffect(() => {
        fetch('/api/chatbots').then(r => r.json()).then(setChatbots);
    }, []);
    return <ChatbotList chatbots={chatbots} />;
}

// AFTER: Server component with direct fetch
export default async function ChatbotsPage() {
    const chatbots = await db.query.chatbots.findMany({
        where: eq(chatbots.userId, userId),
    });
    return <ChatbotList chatbots={chatbots} />;
}
```

#### WordPress Hook Pattern

```php
// ISSUE: Direct function call in global scope
// Before
my_setup_function();

// AFTER: Use proper hook
add_action('plugins_loaded', 'my_setup_function');
```

### 4. Architecture Fixes

#### Single Responsibility Principle

```typescript
// ISSUE: Class doing too much
// Before
class RagEngine {
    chunk(document) { /* ... */ }
    embed(text) { /* ... */ }
    search(query) { /* ... */ }
    buildPrompt(context) { /* ... */ }
    generate(prompt) { /* ... */ }
    stream(response) { /* ... */ }
}

// AFTER: Separate concerns
class DocumentChunker { chunk(document) { /* ... */ } }
class EmbeddingService { embed(text) { /* ... */ } }
class VectorSearchService { search(query) { /* ... */ } }
class PromptBuilder { buildPrompt(context) { /* ... */ } }

class RagEngine {
    constructor(
        private chunker: DocumentChunker,
        private embedder: EmbeddingService,
        private search: VectorSearchService,
        private promptBuilder: PromptBuilder,
    ) {}

    async query(question: string) {
        // Orchestrate services
    }
}
```

### 5. Accessibility Fixes

#### Missing ARIA

```tsx
// ISSUE: Chat messages not announced
// Before
<div className="messages">
    {messages.map(m => <Message key={m.id} {...m} />)}
</div>

// AFTER: Add ARIA live region
<div
    className="messages"
    role="log"
    aria-live="polite"
    aria-label="Chat conversation"
>
    {messages.map(m => <Message key={m.id} {...m} />)}
</div>
```

#### Focus Management

```tsx
// ISSUE: Modal doesn't trap focus
// Before
function Modal({ isOpen }) {
    if (!isOpen) return null;
    return <div className="modal">...</div>;
}

// AFTER: Proper focus trap
import { FocusTrap } from '@headlessui/react';

function Modal({ isOpen }) {
    if (!isOpen) return null;
    return (
        <FocusTrap>
            <div className="modal" role="dialog" aria-modal="true">
                ...
            </div>
        </FocusTrap>
    );
}
```

## Output Format

```markdown
# Code Fix Report

## Summary

| Severity | Issues | Fixed | Remaining |
|----------|--------|-------|-----------|
| Critical | 0 | 0 | 0 |
| High | 3 | 3 | 0 |
| Medium | 5 | 4 | 1 |
| Low | 8 | 2 | 6 |

## Quality Gate Status: ✅ PASSED

All Critical and High issues resolved.

---

## Fixes Applied

### HIGH-001: SQL Injection in WordPress
**Agent:** wordpress-security-auditor
**File:** wordpress-plugin/includes/class-chatbot.php:78

**Before:**
```php
$wpdb->query("DELETE FROM {$table} WHERE id = {$_GET['id']}");
```

**After:**
```php
$wpdb->query($wpdb->prepare(
    "DELETE FROM {$table} WHERE id = %d",
    absint($_GET['id'])
));
```

---

### HIGH-002: N+1 Query in Chatbot List
**Agent:** drizzle-schema-reviewer
**File:** src/app/(dashboard)/chatbots/page.tsx:23

**Before:**
```typescript
for (const chatbot of chatbots) {
    chatbot.documents = await getDocuments(chatbot.id);
}
```

**After:**
```typescript
const chatbots = await db.query.chatbots.findMany({
    where: eq(chatbots.userId, user.id),
    with: { documents: true },
});
```

---

### HIGH-003: Missing ARIA on Chat
**Agent:** accessibility-guardian
**File:** src/components/ChatWidget/MessageList.tsx:25

**Before:**
```tsx
<div className="messages">
```

**After:**
```tsx
<div className="messages" role="log" aria-live="polite">
```

---

## Remaining Issues (Deferred)

| ID | Severity | Issue | Reason |
|----|----------|-------|--------|
| LOW-001 | Low | Missing JSDoc | Style preference |
| LOW-002 | Low | Long function | Working code |
| MEDIUM-005 | Medium | Component size | Requires refactor sprint |

---

## Re-Review Required

Run `/full-review` to verify all fixes.
```

## Integration

### Invoked By

- `/next-phase` command (Phase 10)
- After `/full-review` with issues

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:review:code-fixer"
```

### Loop Behavior

```
WHILE high_severity_issues > 0 AND iteration < MAX_ITERATIONS:
  1. Get review issues
  2. Prioritize by severity
  3. Fix highest priority first
  4. Run targeted tests
  5. Re-run review for fixed areas

IF high_severity_issues == 0:
  PASS: Proceed to Phase 11
ELSE:
  FAIL: Manual intervention required
```

## Related Agents

- `nextjs-standards-reviewer` - Reports Next.js issues
- `saas-security-auditor` - Reports security issues
- `accessibility-guardian` - Reports accessibility issues
- `architecture-reviewer` - Reports architecture issues
