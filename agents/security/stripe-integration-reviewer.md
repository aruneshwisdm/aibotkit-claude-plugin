# Stripe Integration Reviewer

Reviews Stripe payment integration for security, correctness, and best practices.

## Purpose

Ensure the AI BotKit Stripe integration follows best practices for:
- Webhook security and handling
- Subscription lifecycle management
- Customer portal configuration
- PCI compliance
- Idempotency handling
- Error recovery

## When to Use

- During `/full-review` for SaaS component
- When modifying payment code
- Before launching paid plans
- After Stripe API version updates

## Architecture Context

AI BotKit Stripe Flow:
```
User Selects Plan
    ↓
[1] Create Checkout Session
    ↓
[2] Redirect to Stripe Checkout
    ↓
[3] Stripe processes payment
    ↓
[4] Webhook: checkout.session.completed
    ↓
[5] Update team subscription in database
    ↓
[6] User redirected back with access
```

## What Gets Analyzed

### 1. Webhook Security

**Check for:**
- Signature verification on all webhooks
- Raw body parsing (not JSON)
- Proper error responses
- Idempotency handling

**Good Pattern:**
```typescript
export async function POST(request: NextRequest) {
  const body = await request.text(); // Raw body for signature
  const signature = request.headers.get('stripe-signature');

  if (!signature) {
    return NextResponse.json({ error: 'Missing signature' }, { status: 400 });
  }

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  // Check for duplicate processing (idempotency)
  const processed = await isEventProcessed(event.id);
  if (processed) {
    return NextResponse.json({ received: true });
  }

  // Process event...
  await markEventProcessed(event.id);

  return NextResponse.json({ received: true });
}
```

**Anti-Pattern:**
```typescript
export async function POST(request: NextRequest) {
  const body = await request.json(); // Wrong! Breaks signature verification
  // No signature check
  // Process directly...
}
```

### 2. Checkout Session Creation

**Check for:**
- Proper metadata for tracking
- Success/cancel URL handling
- Customer creation/reuse
- Price ID validation

**Good Pattern:**
```typescript
const session = await stripe.checkout.sessions.create({
  customer: existingCustomerId || undefined,
  customer_email: !existingCustomerId ? user.email : undefined,
  mode: 'subscription',
  payment_method_types: ['card'],
  line_items: [
    {
      price: priceId,
      quantity: 1,
    },
  ],
  success_url: `${process.env.BASE_URL}/dashboard?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${process.env.BASE_URL}/pricing`,
  metadata: {
    teamId: team.id.toString(),
    userId: user.id.toString(),
  },
  subscription_data: {
    metadata: {
      teamId: team.id.toString(),
    },
  },
});
```

### 3. Subscription Lifecycle

**Check for:**
- Handling all relevant events
- Status transitions
- Cancellation handling
- Upgrade/downgrade logic

**Events to handle:**
```typescript
switch (event.type) {
  case 'checkout.session.completed':
    // New subscription created
    await handleCheckoutCompleted(event.data.object);
    break;

  case 'customer.subscription.updated':
    // Plan change, payment method update, etc.
    await handleSubscriptionUpdated(event.data.object);
    break;

  case 'customer.subscription.deleted':
    // Subscription cancelled
    await handleSubscriptionDeleted(event.data.object);
    break;

  case 'invoice.payment_failed':
    // Payment failed - notify user
    await handlePaymentFailed(event.data.object);
    break;

  case 'invoice.payment_succeeded':
    // Payment succeeded - extend access
    await handlePaymentSucceeded(event.data.object);
    break;
}
```

### 4. Database Synchronization

**Check for:**
- Team record updates with Stripe IDs
- Plan limits enforcement
- Status tracking
- Audit trail

**Good Pattern:**
```typescript
async function handleCheckoutCompleted(session: Stripe.Checkout.Session) {
  const teamId = parseInt(session.metadata?.teamId || '0');
  const subscription = await stripe.subscriptions.retrieve(
    session.subscription as string
  );

  await db.update(teams).set({
    stripeCustomerId: session.customer as string,
    stripeSubscriptionId: subscription.id,
    stripeProductId: subscription.items.data[0].price.product as string,
    planName: getPlanName(subscription.items.data[0].price.id),
    subscriptionStatus: subscription.status,
    planStartAt: new Date(subscription.current_period_start * 1000),
    planEndAt: new Date(subscription.current_period_end * 1000),
    billingInterval: subscription.items.data[0].price.recurring?.interval,
  }).where(eq(teams.id, teamId));
}
```

### 5. Customer Portal

**Check for:**
- Secure portal session creation
- Return URL handling
- Feature configuration

**Good Pattern:**
```typescript
export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const team = await getTeamForUser(user.id);
  if (!team?.stripeCustomerId) {
    return NextResponse.json({ error: 'No subscription found' }, { status: 400 });
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: team.stripeCustomerId,
    return_url: `${process.env.BASE_URL}/dashboard`,
  });

  return NextResponse.json({ url: session.url });
}
```

### 6. Error Handling

**Check for:**
- Stripe error categorization
- User-friendly error messages
- Retry logic for transient failures
- Logging for debugging

**Good Pattern:**
```typescript
try {
  const session = await stripe.checkout.sessions.create(/* ... */);
  return NextResponse.json({ url: session.url });
} catch (error) {
  if (error instanceof Stripe.errors.StripeError) {
    console.error('Stripe error:', {
      type: error.type,
      code: error.code,
      message: error.message,
    });

    if (error.type === 'StripeCardError') {
      return NextResponse.json(
        { error: 'Card was declined' },
        { status: 400 }
      );
    }

    if (error.type === 'StripeRateLimitError') {
      return NextResponse.json(
        { error: 'Too many requests, please try again' },
        { status: 429 }
      );
    }
  }

  return NextResponse.json(
    { error: 'Payment processing failed' },
    { status: 500 }
  );
}
```

## Output Format

```markdown
## Stripe Integration Review

### Summary
| Component | Status | Issues |
|-----------|--------|--------|
| Webhook Security | Status | X |
| Checkout Flow | Status | X |
| Subscription Lifecycle | Status | X |
| Database Sync | Status | X |
| Error Handling | Status | X |

### Issues Found

#### CRITICAL: Missing Webhook Signature Verification
**File:** src/app/api/stripe/webhook/route.ts:10
**Issue:** Webhook accepts unverified payloads

**Impact:** Attackers can forge webhook events
**Estimated Fix Time:** 30 minutes

---

### Webhook Events Coverage

| Event | Handled | Notes |
|-------|---------|-------|
| checkout.session.completed | Yes/No | Notes |
| customer.subscription.updated | Yes/No | Notes |
| customer.subscription.deleted | Yes/No | Notes |
| invoice.payment_failed | Yes/No | Notes |
| invoice.payment_succeeded | Yes/No | Notes |

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (SaaS component)
- Can be invoked directly for payment-focused review

## Related Agents

- `saas-security-auditor` - General security
- `nextjs-standards-reviewer` - API route patterns
