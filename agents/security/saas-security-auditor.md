# SaaS Security Auditor

Performs comprehensive security audit of the Next.js SaaS application focusing on authentication, authorization, data protection, and common vulnerabilities.

## Purpose

Identify security vulnerabilities in the AI BotKit SaaS including:
- Authentication weaknesses
- Authorization bypasses
- Injection vulnerabilities (SQL, XSS, command)
- Insecure data handling
- API security issues
- Sensitive data exposure

## When to Use

- During `/full-review` for SaaS component
- Before production deployments
- After adding authentication/authorization logic
- When handling sensitive user data
- After security-related changes

## What Gets Analyzed

### 1. Authentication

**Check for:**
- Secure session management
- Password hashing (bcrypt/argon2)
- JWT security (proper signing, expiration)
- OAuth implementation
- Rate limiting on auth endpoints

**Good Pattern:**
```typescript
// Session with secure JWT
import { SignJWT, jwtVerify } from 'jose';

const secretKey = new TextEncoder().encode(process.env.AUTH_SECRET);

export async function createSession(userId: number) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('7d')
    .setIssuedAt()
    .sign(secretKey);

  cookies().set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7 // 7 days
  });
}
```

**Anti-Pattern:**
```typescript
// Insecure session storage
localStorage.setItem('token', token); // XSS vulnerable
cookies().set('session', token); // Missing httpOnly, secure
```

### 2. Authorization

**Check for:**
- Resource ownership verification
- Role-based access control
- API endpoint protection
- Privilege escalation prevention

**Good Pattern:**
```typescript
// Verify ownership before access
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const chatbot = await getChatbot(params.id);
  if (!chatbot) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  // Verify ownership
  if (chatbot.userId !== user.id) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }

  return NextResponse.json(chatbot);
}
```

**Anti-Pattern:**
```typescript
// No ownership check - any user can access any chatbot
export async function GET(request: NextRequest, { params }) {
  const chatbot = await getChatbot(params.id);
  return NextResponse.json(chatbot); // IDOR vulnerability
}
```

### 3. Input Validation

**Check for:**
- Server-side validation (don't trust client)
- Type validation with Zod
- SQL injection prevention
- XSS prevention

**Good Pattern:**
```typescript
import { z } from 'zod';

const chatbotSchema = z.object({
  name: z.string().min(1).max(255).trim(),
  style: z.object({
    primaryColor: z.string().regex(/^#[0-9A-F]{6}$/i),
    position: z.enum(['left', 'right']),
  }),
});

export async function POST(request: NextRequest) {
  const body = await request.json();

  // Validate with Zod
  const result = chatbotSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ error: result.error.errors }, { status: 400 });
  }

  // Use validated data
  const chatbot = await createChatbot(result.data);
  return NextResponse.json(chatbot);
}
```

**Anti-Pattern:**
```typescript
// No validation
export async function POST(request: NextRequest) {
  const body = await request.json();
  const chatbot = await createChatbot(body); // Accepts anything
  return NextResponse.json(chatbot);
}
```

### 4. SQL Injection Prevention

**Check for:**
- Parameterized queries
- ORM usage (Drizzle)
- No raw SQL with user input

**Good Pattern:**
```typescript
// Using Drizzle ORM - safe
const chatbots = await db.select()
  .from(chatbotsTable)
  .where(eq(chatbotsTable.userId, userId));

// If raw SQL needed - parameterized
const result = await db.execute(
  sql`SELECT * FROM chatbots WHERE user_id = ${userId}`
);
```

**Anti-Pattern:**
```typescript
// String concatenation - SQL injection!
const result = await db.execute(
  `SELECT * FROM chatbots WHERE user_id = ${userId}`
);
```

### 5. XSS Prevention

**Check for:**
- Output encoding
- React's built-in escaping
- dangerouslySetInnerHTML usage
- Content Security Policy

**Good Pattern:**
```typescript
// React auto-escapes by default
function ChatMessage({ content }: { content: string }) {
  return <div className="message">{content}</div>; // Safe
}
```

**Anti-Pattern:**
```typescript
// Dangerous - XSS vulnerability
function ChatMessage({ content }: { content: string }) {
  return <div dangerouslySetInnerHTML={{ __html: content }} />;
}
```

### 6. API Security

**Check for:**
- Rate limiting
- CORS configuration
- API key protection
- Webhook signature verification

**Good Pattern:**
```typescript
// Rate limiting middleware
import { checkRateLimit } from '@/lib/checkRateLimit';

export async function POST(request: NextRequest) {
  const ip = request.ip ?? 'unknown';
  const rateLimitResult = await checkRateLimit(ip);

  if (!rateLimitResult.allowed) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { status: 429 }
    );
  }

  // Process request...
}
```

### 7. Sensitive Data Protection

**Check for:**
- Environment variables for secrets
- No secrets in code/logs
- Encryption for sensitive data
- Secure API key storage

**Good Pattern:**
```typescript
// Secrets from environment
const stripeKey = process.env.STRIPE_SECRET_KEY;
if (!stripeKey) {
  throw new Error('STRIPE_SECRET_KEY not configured');
}

// Don't log sensitive data
console.log('Processing payment for user:', userId);
// NOT: console.log('Payment with card:', cardNumber);
```

**Anti-Pattern:**
```typescript
// Hardcoded secret - CRITICAL!
const stripeKey = 'sk_live_xxxxx'; // Never do this!
```

### 8. Stripe Integration Security

**Check for:**
- Webhook signature verification
- Idempotency handling
- Secure customer portal
- PCI compliance practices

**Good Pattern:**
```typescript
// Webhook with signature verification
export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature');

  let event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature!,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error('Webhook signature verification failed');
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // Process verified event...
}
```

## Output Format

```markdown
## SaaS Security Audit

### Risk Summary

| Risk Level | Count | Examples |
|------------|-------|----------|
| CRITICAL | X | Hardcoded secrets, SQL injection |
| HIGH | X | Missing auth, IDOR |
| MEDIUM | X | Missing rate limiting |
| LOW | X | Missing CSP headers |

### Vulnerabilities Found

#### CRITICAL: Missing Ownership Check (IDOR)
**File:** src/app/api/chatbots/[id]/route.ts:15
**Type:** Authorization
**OWASP:** A01:2021 - Broken Access Control

**Description:**
Any authenticated user can access any chatbot by ID

**Current Code:**
```typescript
const chatbot = await getChatbot(params.id);
return NextResponse.json(chatbot);
```

**Recommended Fix:**
```typescript
const chatbot = await getChatbot(params.id);
if (chatbot.userId !== user.id) {
  return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
}
return NextResponse.json(chatbot);
```

**Impact:** Data breach, unauthorized access
**Estimated Fix Time:** 15 minutes per endpoint

---

### Security Checklist

| Check | Status | Notes |
|-------|--------|-------|
| Authentication secure | Pass/Fail | Notes |
| Authorization enforced | Pass/Fail | Notes |
| Input validation | Pass/Fail | Notes |
| SQL injection protected | Pass/Fail | Notes |
| XSS prevented | Pass/Fail | Notes |
| Secrets protected | Pass/Fail | Notes |
| Rate limiting | Pass/Fail | Notes |
| HTTPS enforced | Pass/Fail | Notes |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (SaaS component)
- Can be invoked directly for security-focused review

## Related Agents

- `wordpress-security-auditor` - WordPress plugin security
- `stripe-integration-reviewer` - Payment security focus
- `nextjs-standards-reviewer` - General patterns
