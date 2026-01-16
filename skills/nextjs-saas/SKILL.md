---
name: nextjs-saas
description: Next.js 16 App Router patterns for SaaS applications
---

# Next.js SaaS Patterns

Best practices for building SaaS applications with Next.js 16 App Router.

## Project Structure

```
src/
├── app/                      # App Router
│   ├── (auth)/               # Auth route group (no layout prefix)
│   │   ├── login/
│   │   ├── signup/
│   │   └── layout.tsx        # Minimal auth layout
│   ├── (dashboard)/          # Dashboard route group
│   │   ├── dashboard/
│   │   ├── chatbots/
│   │   ├── settings/
│   │   └── layout.tsx        # Dashboard layout with sidebar
│   ├── api/                  # API routes
│   │   ├── auth/
│   │   ├── chatbots/
│   │   └── stripe/
│   ├── layout.tsx            # Root layout
│   └── page.tsx              # Landing page
├── components/               # React components
│   ├── ui/                   # Base UI components (shadcn)
│   └── features/             # Feature-specific components
├── lib/                      # Utilities and services
│   ├── auth/                 # Authentication
│   ├── db/                   # Database (Drizzle)
│   └── payments/             # Stripe integration
└── hooks/                    # Custom React hooks
```

## Server vs Client Components

### Default to Server Components

```tsx
// Server Component (default) - runs on server
// Can fetch data, access database, keep secrets
async function ChatbotList() {
  const user = await getUser();
  const chatbots = await db.query.chatbots.findMany({
    where: eq(chatbotsTable.userId, user.id),
  });

  return (
    <ul>
      {chatbots.map((chatbot) => (
        <ChatbotCard key={chatbot.id} chatbot={chatbot} />
      ))}
    </ul>
  );
}
```

### Client Components for Interactivity

```tsx
'use client';

import { useState } from 'react';

// Client Component - runs in browser
// Use for: state, effects, event handlers, browser APIs
function ChatbotCard({ chatbot }: { chatbot: Chatbot }) {
  const [isEditing, setIsEditing] = useState(false);

  return (
    <div onClick={() => setIsEditing(true)}>
      {isEditing ? (
        <EditForm chatbot={chatbot} onClose={() => setIsEditing(false)} />
      ) : (
        <ChatbotDisplay chatbot={chatbot} />
      )}
    </div>
  );
}
```

### Composition Pattern

```tsx
// Server Component fetches data
async function ChatbotPage({ params }: { params: { id: string } }) {
  const chatbot = await getChatbot(params.id);

  return (
    <div>
      <h1>{chatbot.name}</h1>
      {/* Pass data to client component */}
      <ChatbotEditor initialData={chatbot} />
    </div>
  );
}

// Client Component handles interactivity
'use client';
function ChatbotEditor({ initialData }: { initialData: Chatbot }) {
  const [data, setData] = useState(initialData);
  // ...
}
```

## Data Fetching

### Server Component Fetching

```tsx
// Parallel data fetching
async function Dashboard() {
  // Start all fetches simultaneously
  const [user, chatbots, analytics, subscription] = await Promise.all([
    getUser(),
    getChatbots(),
    getAnalytics(),
    getSubscription(),
  ]);

  return (
    <div>
      <Header user={user} subscription={subscription} />
      <ChatbotList chatbots={chatbots} />
      <Analytics data={analytics} />
    </div>
  );
}
```

### React Cache for Deduplication

```tsx
import { cache } from 'react';

// Cached function - called once per request even if used multiple times
export const getUser = cache(async () => {
  const session = await getSession();
  if (!session) return null;

  return db.query.users.findFirst({
    where: eq(users.id, session.userId),
  });
});

// Both components call getUser, but it only executes once
async function Header() {
  const user = await getUser();
  return <nav>{user?.name}</nav>;
}

async function Sidebar() {
  const user = await getUser();
  return <aside>{user?.email}</aside>;
}
```

### Client-Side Data Fetching with SWR

```tsx
'use client';

import useSWR from 'swr';

function ChatbotStats({ chatbotId }: { chatbotId: number }) {
  const { data, error, isLoading } = useSWR(
    `/api/chatbots/${chatbotId}/stats`,
    fetcher,
    {
      refreshInterval: 30000, // Refresh every 30 seconds
    }
  );

  if (isLoading) return <Skeleton />;
  if (error) return <Error message={error.message} />;

  return <StatsDisplay data={data} />;
}
```

## API Routes

### Route Handler Pattern

```tsx
// src/app/api/chatbots/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const createChatbotSchema = z.object({
  name: z.string().min(1).max(255),
  style: z.object({
    primaryColor: z.string().regex(/^#[0-9A-F]{6}$/i),
  }).optional(),
});

// GET /api/chatbots
export async function GET(request: NextRequest) {
  try {
    const user = await getUser();
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const chatbots = await db.query.chatbots.findMany({
      where: eq(chatbotsTable.userId, user.id),
    });

    return NextResponse.json(chatbots);
  } catch (error) {
    console.error('Error fetching chatbots:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// POST /api/chatbots
export async function POST(request: NextRequest) {
  try {
    const user = await getUser();
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const result = createChatbotSchema.safeParse(body);

    if (!result.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: result.error.errors },
        { status: 400 }
      );
    }

    const chatbot = await createChatbot(result.data, user.id);
    return NextResponse.json(chatbot, { status: 201 });
  } catch (error) {
    console.error('Error creating chatbot:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Dynamic Route Handler

```tsx
// src/app/api/chatbots/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

type Params = { params: { id: string } };

// GET /api/chatbots/:id
export async function GET(request: NextRequest, { params }: Params) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const chatbot = await getChatbot(parseInt(params.id));
  if (!chatbot) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  // Authorization check
  if (chatbot.userId !== user.id) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
  }

  return NextResponse.json(chatbot);
}
```

## Authentication Pattern

### Session Management

```tsx
// src/lib/auth/session.ts
import { cookies } from 'next/headers';
import { SignJWT, jwtVerify } from 'jose';

const secretKey = new TextEncoder().encode(process.env.AUTH_SECRET);

export async function createSession(userId: number) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('7d')
    .setIssuedAt()
    .sign(secretKey);

  (await cookies()).set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7,
    path: '/',
  });
}

export async function getSession() {
  const token = (await cookies()).get('session')?.value;
  if (!token) return null;

  try {
    const { payload } = await jwtVerify(token, secretKey);
    return payload as { userId: number };
  } catch {
    return null;
  }
}

export async function getUser() {
  const session = await getSession();
  if (!session) return null;

  return db.query.users.findFirst({
    where: eq(users.id, session.userId),
  });
}
```

### Protected Routes with Middleware

```tsx
// src/middleware.ts
import { NextRequest, NextResponse } from 'next/server';

const protectedRoutes = ['/dashboard', '/chatbots', '/settings'];
const authRoutes = ['/login', '/signup'];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const session = request.cookies.get('session')?.value;

  // Redirect logged-in users away from auth pages
  if (authRoutes.some(route => pathname.startsWith(route))) {
    if (session) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
    return NextResponse.next();
  }

  // Protect dashboard routes
  if (protectedRoutes.some(route => pathname.startsWith(route))) {
    if (!session) {
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('redirect', pathname);
      return NextResponse.redirect(loginUrl);
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

## Layout Patterns

### Dashboard Layout

```tsx
// src/app/(dashboard)/layout.tsx
import { redirect } from 'next/navigation';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const user = await getUser();
  if (!user) redirect('/login');

  return (
    <div className="flex h-screen">
      <Sidebar user={user} />
      <main className="flex-1 overflow-y-auto">
        <TopBar user={user} />
        <div className="p-6">{children}</div>
      </main>
    </div>
  );
}
```

### Loading States

```tsx
// src/app/(dashboard)/chatbots/loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-8 w-48" />
      <div className="grid grid-cols-3 gap-4">
        {[...Array(6)].map((_, i) => (
          <Skeleton key={i} className="h-32" />
        ))}
      </div>
    </div>
  );
}
```

### Error Handling

```tsx
// src/app/(dashboard)/chatbots/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div className="text-center py-12">
      <h2 className="text-xl font-semibold">Something went wrong!</h2>
      <p className="text-gray-600 mt-2">{error.message}</p>
      <button
        onClick={reset}
        className="mt-4 px-4 py-2 bg-primary text-white rounded"
      >
        Try again
      </button>
    </div>
  );
}
```

## Stripe Integration

### Checkout Session

```tsx
// src/app/api/stripe/checkout/route.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { priceId } = await request.json();
  const team = await getTeamForUser(user.id);

  const session = await stripe.checkout.sessions.create({
    customer: team?.stripeCustomerId || undefined,
    customer_email: !team?.stripeCustomerId ? user.email : undefined,
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.BASE_URL}/dashboard?success=true`,
    cancel_url: `${process.env.BASE_URL}/pricing`,
    metadata: {
      userId: user.id.toString(),
      teamId: team?.id.toString(),
    },
  });

  return NextResponse.json({ url: session.url });
}
```

### Webhook Handler

```tsx
// src/app/api/stripe/webhook/route.ts
export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('stripe-signature')!;

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutCompleted(event.data.object);
      break;
    case 'customer.subscription.updated':
      await handleSubscriptionUpdated(event.data.object);
      break;
    case 'customer.subscription.deleted':
      await handleSubscriptionDeleted(event.data.object);
      break;
  }

  return NextResponse.json({ received: true });
}
```

## Environment Variables

```bash
# .env.local
# Database
POSTGRES_URL=postgresql://user:pass@localhost:5432/dbname

# Authentication
AUTH_SECRET=your-secret-key-min-32-chars

# Stripe
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx

# AI Services
TOGETHER_API_KEY=xxx
PINECONE_API_KEY=xxx
PINECONE_INDEX=your-index
PINECONE_HOST=https://your-index.svc.pinecone.io

# App
BASE_URL=http://localhost:3000
```

## Performance Optimization

### Dynamic Imports

```tsx
import dynamic from 'next/dynamic';

// Lazy load heavy components
const Chart = dynamic(() => import('@/components/Chart'), {
  loading: () => <Skeleton className="h-64" />,
  ssr: false, // Disable SSR for client-only components
});

// Lazy load modals
const EditModal = dynamic(() => import('@/components/EditModal'));
```

### Image Optimization

```tsx
import Image from 'next/image';

function Avatar({ user }: { user: User }) {
  return (
    <Image
      src={user.avatarUrl || '/default-avatar.png'}
      alt={user.name}
      width={40}
      height={40}
      className="rounded-full"
      priority={false} // Only use priority for above-the-fold images
    />
  );
}
```

### Metadata

```tsx
// src/app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'AI BotKit',
    template: '%s | AI BotKit',
  },
  description: 'AI-powered chatbots for your website',
  openGraph: {
    title: 'AI BotKit',
    description: 'AI-powered chatbots for your website',
    url: 'https://aibotkit.io',
    siteName: 'AI BotKit',
    type: 'website',
  },
};

// Dynamic metadata per page
// src/app/(dashboard)/chatbots/[id]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const chatbot = await getChatbot(params.id);
  return {
    title: chatbot.name,
    description: `Manage ${chatbot.name} chatbot`,
  };
}
```
