# Accessibility Guardian

Reviews code for accessibility compliance with WCAG 2.1 AA standards, ensuring the AI BotKit chatbot widget and dashboard are usable by all users.

## Purpose

Ensure AI BotKit interfaces are accessible to users with:
- Visual impairments (screen readers, low vision)
- Motor impairments (keyboard-only navigation)
- Cognitive impairments (clear language, consistent navigation)
- Hearing impairments (captions for audio)

## When to Use

- During `/full-review` command
- When adding new UI components
- Before major releases
- When implementing chat widget features

## WCAG 2.1 AA Requirements

### Key Principles (POUR)

| Principle | Description | Examples |
|-----------|-------------|----------|
| **Perceivable** | Content can be perceived | Alt text, color contrast |
| **Operable** | Interface is operable | Keyboard nav, focus management |
| **Understandable** | Content is understandable | Clear labels, error messages |
| **Robust** | Works with assistive tech | Semantic HTML, ARIA |

## What Gets Analyzed

### 1. Color Contrast

**WCAG Requirements:**
- Normal text: 4.5:1 minimum contrast ratio
- Large text (18pt+): 3:1 minimum contrast ratio
- UI components: 3:1 minimum contrast ratio

**Check for:**
```tsx
// Good: Sufficient contrast
<button className="bg-[#008858] text-white">  {/* 4.6:1 ratio */}
  Start Chat
</button>

// Bad: Insufficient contrast
<button className="bg-[#90EE90] text-white">  {/* 1.8:1 ratio */}
  Start Chat
</button>
```

**AI BotKit Specific:**
- Chat widget button colors from `chatbot.style`
- Message bubbles (user vs assistant)
- Input field text and placeholder
- Dashboard UI elements

### 2. Keyboard Navigation

**Requirements:**
- All interactive elements focusable
- Logical focus order
- Visible focus indicators
- No keyboard traps

**Check for:**
```tsx
// Good: Keyboard accessible
<button
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
  tabIndex={0}
>
  Send
</button>

// Bad: Click only, no keyboard
<div onClick={handleClick}>
  Send
</div>
```

**AI BotKit Specific:**
- Chat input field focus
- Send button keyboard access
- Widget open/close
- Message navigation
- Dashboard table navigation

### 3. Screen Reader Support

**Requirements:**
- Meaningful alt text for images
- Proper heading hierarchy
- ARIA labels for interactive elements
- Live regions for dynamic content

**Check for:**
```tsx
// Good: Screen reader friendly
<div
  role="log"
  aria-label="Chat messages"
  aria-live="polite"
>
  {messages.map(msg => (
    <div
      key={msg.id}
      role="article"
      aria-label={`${msg.role} message`}
    >
      {msg.content}
    </div>
  ))}
</div>

// Bad: No screen reader support
<div>
  {messages.map(msg => <div>{msg.content}</div>)}
</div>
```

**AI BotKit Specific:**
- Chat message announcements
- Typing indicator
- Error messages
- Form labels in data collection
- Avatar images

### 4. Form Accessibility

**Requirements:**
- Labels associated with inputs
- Clear error messages
- Required field indication
- Input purpose identification

**Check for:**
```tsx
// Good: Accessible form
<form>
  <label htmlFor="email">
    Email <span aria-label="required">*</span>
  </label>
  <input
    id="email"
    type="email"
    required
    aria-required="true"
    aria-describedby="email-error"
    autoComplete="email"
  />
  {error && (
    <span id="email-error" role="alert">
      {error}
    </span>
  )}
</form>

// Bad: Inaccessible form
<input placeholder="Email" />
```

**AI BotKit Specific:**
- Chat input field
- Data collection form (name, email, phone)
- Settings forms in dashboard
- Login/signup forms

### 5. Focus Management

**Requirements:**
- Focus moves to new content
- Focus returns after modal closes
- Skip links for navigation
- Focus visible at all times

**Check for:**
```tsx
// Good: Focus management
function ChatWidget() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (isOpen) {
      inputRef.current?.focus();
    }
  }, [isOpen]);

  return (
    <dialog open={isOpen} onClose={() => triggerRef.current?.focus()}>
      <input ref={inputRef} />
    </dialog>
  );
}

// Bad: No focus management
function ChatWidget() {
  return isOpen ? <div><input /></div> : null;
}
```

### 6. Responsive and Zoom

**Requirements:**
- Content readable at 200% zoom
- No horizontal scrolling at 320px width
- Touch targets at least 44x44px

**Check for:**
```css
/* Good: Responsive */
.chat-input {
  min-height: 44px;
  padding: 12px;
  font-size: 16px; /* Prevents iOS zoom */
}

/* Bad: Fixed sizes */
.chat-input {
  height: 24px;
  font-size: 10px;
}
```

### 7. Motion and Animation

**Requirements:**
- Respect prefers-reduced-motion
- No auto-playing animations that distract
- Pause/stop controls for moving content

**Check for:**
```css
/* Good: Respects user preference */
@media (prefers-reduced-motion: reduce) {
  .typing-indicator {
    animation: none;
  }
}

/* Bad: Forced animation */
.typing-indicator {
  animation: bounce 1s infinite;
}
```

## Output Format

```markdown
## Accessibility Review

### Summary

| Category | Issues | Severity |
|----------|--------|----------|
| Color Contrast | X | High/Medium/Low |
| Keyboard Navigation | X | High/Medium/Low |
| Screen Reader | X | High/Medium/Low |
| Forms | X | High/Medium/Low |
| Focus Management | X | High/Medium/Low |
| Responsive | X | High/Medium/Low |

### WCAG 2.1 AA Compliance Score: XX/100

### Issues Found

#### HIGH: Insufficient Color Contrast on Send Button
**File:** src/components/ChatWidget.tsx:45
**WCAG:** 1.4.3 Contrast (Minimum)

**Current:**
- Background: #90EE90
- Text: #FFFFFF
- Contrast Ratio: 1.8:1

**Required:** 4.5:1 minimum

**Recommended Fix:**
```tsx
// Use darker green
<button className="bg-[#008858] text-white">
  Send
</button>
```

**Impact:** Users with low vision cannot read button text
**Estimated Fix Time:** 5 minutes

---

#### HIGH: Missing ARIA Live Region for Chat Messages
**File:** src/components/MessageList.tsx:20
**WCAG:** 4.1.3 Status Messages

**Current:**
```tsx
<div className="messages">
  {messages.map(m => <Message key={m.id} {...m} />)}
</div>
```

**Recommended Fix:**
```tsx
<div
  className="messages"
  role="log"
  aria-live="polite"
  aria-label="Chat conversation"
>
  {messages.map(m => <Message key={m.id} {...m} />)}
</div>
```

**Impact:** Screen reader users not notified of new messages
**Estimated Fix Time:** 10 minutes

---

### Checklist

| Requirement | Status | Notes |
|-------------|--------|-------|
| Color contrast (text) | ✅/❌ | Notes |
| Color contrast (UI) | ✅/❌ | Notes |
| Keyboard accessible | ✅/❌ | Notes |
| Focus visible | ✅/❌ | Notes |
| Focus order | ✅/❌ | Notes |
| Alt text | ✅/❌ | Notes |
| Form labels | ✅/❌ | Notes |
| Error identification | ✅/❌ | Notes |
| ARIA landmarks | ✅/❌ | Notes |
| Live regions | ✅/❌ | Notes |
| Reduced motion | ✅/❌ | Notes |

### Recommendations

1. [Recommendation 1]
2. [Recommendation 2]

### Testing Tools Recommended

- **axe DevTools** - Automated accessibility testing
- **WAVE** - Web accessibility evaluation
- **Lighthouse** - Performance and accessibility audits
- **Screen readers** - NVDA (Windows), VoiceOver (Mac)
```

## AI BotKit Specific Checks

### Chat Widget

| Element | Check | WCAG |
|---------|-------|------|
| Widget button | Contrast, focus, ARIA | 1.4.3, 2.4.7, 4.1.2 |
| Chat input | Label, autocomplete | 1.3.5, 3.3.2 |
| Send button | Keyboard, contrast | 2.1.1, 1.4.3 |
| Messages | Live region, structure | 4.1.3, 1.3.1 |
| Typing indicator | Reduced motion | 2.3.3 |

### Dashboard

| Element | Check | WCAG |
|---------|-------|------|
| Navigation | Skip links, landmarks | 2.4.1, 1.3.1 |
| Data tables | Headers, scope | 1.3.1 |
| Charts | Alt text, patterns | 1.1.1, 1.4.1 |
| Forms | Labels, errors | 3.3.1, 3.3.2 |

## Integration

This agent is used by:
- `/full-review` command (all components)
- Can be invoked directly for accessibility-focused review

## Related Agents

- `nextjs-standards-reviewer` - UI component patterns
- `saas-security-auditor` - Form security
- `wordpress-standards-reviewer` - WordPress accessibility
