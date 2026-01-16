# Next.js Standards Reviewer

Reviews Next.js 16 App Router code for best practices, patterns, and common issues.

## Purpose

Ensure the AI BotKit SaaS application follows Next.js best practices including:
- Proper use of Server vs Client Components
- Correct data fetching patterns
- Appropriate use of caching and revalidation
- Route handler best practices
- Middleware usage
- Error handling patterns

## When to Use

- During `/full-review` for SaaS component
- When adding new pages or API routes
- After major refactoring
- Before deployment

## What Gets Analyzed

### 1. Server/Client Component Usage

**Check for:**
- 'use client' directive placement
- Server components using client-only APIs
- Unnecessary 'use client' directives
- Props serialization issues

**Good Pattern:**
```tsx
// Server Component (default) - fetches data
async function ChatbotList() {
  const chatbots = await getChatbots();
  return <ChatbotListClient chatbots={chatbots} />;
}

// Client Component - handles interactivity
'use client';
function ChatbotListClient({ chatbots }) {
  const [selected, setSelected] = useState(null);
  // ...
}
```

**Anti-Pattern:**
```tsx
'use client'; // Unnecessary - no client features used
function StaticContent() {
  return <div>Static content</div>;
}
```

### 2. Data Fetching Patterns

**Check for:**
- fetch() in Server Components vs Client Components
- Proper use of React cache()
- Parallel data fetching where possible
- Waterfall request patterns

**Good Pattern:**
```tsx
// Parallel fetching
async function Dashboard() {
  const [user, chatbots, analytics] = await Promise.all([
    getUser(),
    getChatbots(),
    getAnalytics()
  ]);
  // ...
}
```

**Anti-Pattern:**
```tsx
// Waterfall - each waits for previous
async function Dashboard() {
  const user = await getUser();
  const chatbots = await getChatbots(user.id); // Waits for user
  const analytics = await getAnalytics(chatbots); // Waits for chatbots
}
```

### 3. Route Handlers (API Routes)

**Check for:**
- Proper HTTP method handling
- Request validation
- Error response format
- Authentication checks
- Response caching headers

**Good Pattern:**
```tsx
// src/app/api/chatbots/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getUser } from '@/lib/auth/session';
import { z } from 'zod';

const createChatbotSchema = z.object({
  name: z.string().min(1).max(255),
  // ...
});

export async function POST(request: NextRequest) {
  try {
    const user = await getUser();
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();
    const validated = createChatbotSchema.parse(body);

    const chatbot = await createChatbot(validated, user.id);
    return NextResponse.json(chatbot, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: error.errors }, { status: 400 });
    }
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 });
  }
}
```

### 4. Layout and Page Structure

**Check for:**
- Proper layout hierarchy
- Loading and error boundaries
- Metadata configuration
- Route groups usage

**Good Pattern:**
```
app/
├── (dashboard)/
│   ├── layout.tsx      # Dashboard layout with sidebar
│   ├── dashboard/
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   └── error.tsx
│   └── chatbots/
│       └── [id]/
│           └── page.tsx
├── (auth)/
│   ├── layout.tsx      # Auth layout (minimal)
│   └── login/
│       └── page.tsx
└── api/
    └── ...
```

### 5. Performance Patterns

**Check for:**
- Dynamic imports for heavy components
- Image optimization with next/image
- Font optimization with next/font
- Proper use of Suspense boundaries

**Good Pattern:**
```tsx
import dynamic from 'next/dynamic';

// Lazy load heavy chart component
const AnalyticsChart = dynamic(() => import('./AnalyticsChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false // Client-only component
});
```

## Output Format

```markdown
## Next.js Standards Review

### Summary
| Category | Issues | Severity |
|----------|--------|----------|
| Server/Client Components | X | High/Medium/Low |
| Data Fetching | X | High/Medium/Low |
| Route Handlers | X | High/Medium/Low |
| Performance | X | High/Medium/Low |

### Issues Found

#### HIGH: Waterfall Data Fetching in Dashboard
**File:** src/app/(dashboard)/dashboard/page.tsx:15
**Issue:** Sequential data fetching causing slow page load

**Current:**
```tsx
const user = await getUser();
const chatbots = await getChatbots(user.id);
```

**Recommended:**
```tsx
const [user, chatbots] = await Promise.all([
  getUser(),
  getChatbots()
]);
```

**Impact:** Page load time increased by ~500ms
**Estimated Fix Time:** 15 minutes

---

[Continue for each issue...]

### Recommendations
1. [Recommendation 1]
2. [Recommendation 2]
```

## Integration

This agent is used by:
- `/full-review` command (SaaS component)
- Can be invoked directly for focused review

## Related Agents

- `saas-security-auditor` - Security-focused review
- `rag-engine-reviewer` - RAG-specific patterns
- `drizzle-schema-reviewer` - Database patterns
