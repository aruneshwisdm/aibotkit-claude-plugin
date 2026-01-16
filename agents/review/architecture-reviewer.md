# Architecture Reviewer

Reviews code architecture for maintainability, scalability, and adherence to software design principles.

## Purpose

Evaluate AI BotKit architecture for:
- Clean architecture principles
- SOLID principles adherence
- Separation of concerns
- Code coupling and cohesion
- Scalability considerations
- Technical debt identification

## When to Use

- During `/full-review` command
- Before major refactoring
- When planning new features
- During architecture reviews

## Architecture Context

AI BotKit follows a layered architecture:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AI BOTKIT ARCHITECTURE                            │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    PRESENTATION LAYER                        │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │    │
│  │  │    Pages     │  │  Components  │  │    Hooks     │       │    │
│  │  │  (app/*)     │  │              │  │              │       │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    APPLICATION LAYER                         │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │    │
│  │  │ API Routes   │  │   Actions    │  │   Domain     │       │    │
│  │  │ (app/api/*)  │  │              │  │  Services    │       │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    INFRASTRUCTURE LAYER                      │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │    │
│  │  │   Database   │  │  AI Clients  │  │   External   │       │    │
│  │  │   (Drizzle)  │  │  (Pinecone)  │  │   (Stripe)   │       │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘       │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

## What Gets Analyzed

### 1. Layer Separation

**Check for:**
- Presentation layer doesn't access database directly
- Business logic not in API routes
- Infrastructure concerns isolated

**Good Pattern:**
```typescript
// API Route (thin, delegates to service)
// src/app/api/chatbots/route.ts
export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) return unauthorized();

  const data = await request.json();
  const validated = chatbotSchema.parse(data);

  const chatbot = await chatbotService.create(validated, user.id);
  return NextResponse.json(chatbot);
}

// Service (business logic)
// src/domain/chatbot/chatbot.service.ts
export class ChatbotService {
  async create(data: CreateChatbotDto, userId: number) {
    // Business logic here
    const namespace = this.generateNamespace(data.name, userId);
    return this.repository.create({ ...data, userId, namespace });
  }
}
```

**Anti-Pattern:**
```typescript
// API Route with everything (bad)
export async function POST(request: NextRequest) {
  const user = await getUser();
  const data = await request.json();

  // Direct database access
  const chatbot = await db.insert(chatbots).values({
    ...data,
    userId: user.id,
    namespace: `${data.name}-user-${user.id}`, // Business logic in route
  });

  // Side effects in route
  await pinecone.createNamespace(chatbot.namespace);
  await sendEmail(user.email, 'Chatbot created');

  return NextResponse.json(chatbot);
}
```

### 2. SOLID Principles

#### Single Responsibility Principle (SRP)

**Check for:**
- Classes/functions do one thing
- Clear, focused responsibilities

**Good:**
```typescript
// Each class has one responsibility
class ChatbotRepository {
  async create(data: NewChatbot) { /* DB operations only */ }
  async findById(id: number) { /* ... */ }
}

class ChatbotValidator {
  validate(data: unknown): CreateChatbotDto { /* validation only */ }
}

class ChatbotNamespaceGenerator {
  generate(name: string, userId: number): string { /* namespace logic */ }
}
```

**Bad:**
```typescript
// God class doing everything
class ChatbotManager {
  async create(data: unknown) {
    // Validates
    // Creates in DB
    // Creates Pinecone namespace
    // Sends email
    // Logs analytics
  }
}
```

#### Open/Closed Principle (OCP)

**Check for:**
- Extension through interfaces
- New features without modifying existing code

**Good:**
```typescript
// Extensible through interface
interface LLMClient {
  complete(messages: Message[]): Promise<string>;
  stream(messages: Message[]): AsyncIterable<string>;
}

class TogetherClient implements LLMClient { /* ... */ }
class OpenAIClient implements LLMClient { /* ... */ }
class GeminiClient implements LLMClient { /* ... */ }

// RAG engine accepts any LLM client
class RagEngine {
  constructor(private llmClient: LLMClient) {}
}
```

#### Dependency Inversion Principle (DIP)

**Check for:**
- Depend on abstractions, not implementations
- Dependency injection

**Good:**
```typescript
// Depends on abstraction
class RagEngine {
  constructor(
    private vectorStore: VectorStore,  // Interface
    private llmClient: LLMClient,      // Interface
  ) {}
}

// Inject implementations
const ragEngine = new RagEngine(
  new PineconeVectorStore(),
  new TogetherClient(),
);
```

**Bad:**
```typescript
// Depends on concrete implementation
class RagEngine {
  private pinecone = new PineconeClient();  // Hardcoded
  private together = new TogetherClient();   // Hardcoded
}
```

### 3. Code Coupling

**Check for:**
- Loose coupling between modules
- No circular dependencies
- Clear module boundaries

**Coupling Metrics:**

| Level | Description | Status |
|-------|-------------|--------|
| Low | Module changes don't affect others | Good |
| Medium | Some shared interfaces | OK |
| High | Changes ripple through codebase | Bad |

**Check circular dependencies:**
```typescript
// Bad: Circular dependency
// file-a.ts
import { funcB } from './file-b';

// file-b.ts
import { funcA } from './file-a';  // Circular!
```

### 4. Code Cohesion

**Check for:**
- Related code grouped together
- High cohesion within modules
- Clear module purposes

**Good Structure:**
```
src/lib/ai/
├── index.ts           # Public API
├── rag-engine.ts      # Core RAG logic
├── pinecone-client.ts # Vector store
├── together-client.ts # LLM client
└── types.ts           # Shared types
```

**Bad Structure:**
```
src/lib/
├── utils.ts           # Everything dumped here
├── helpers.ts         # More random functions
└── misc.ts            # Even more random stuff
```

### 5. Scalability Considerations

**Check for:**
- Stateless components where possible
- Cacheable operations identified
- Database query efficiency
- External service rate limiting

**Good:**
```typescript
// Stateless, cacheable
export async function getChatbot(id: number) {
  // Can be cached
  return db.query.chatbots.findFirst({
    where: eq(chatbots.id, id),
    with: { documents: true },
  });
}

// Rate limiting considered
export async function chat(chatbotId: number, message: string) {
  await checkRateLimit(chatbotId);
  // ...
}
```

### 6. Technical Debt Indicators

**Check for:**
- TODO/FIXME comments
- Commented out code
- Magic numbers
- Duplicate code
- Long functions (>50 lines)
- Deep nesting (>3 levels)

**Report technical debt:**
```typescript
// TODO: Refactor this mess  <- Flag
function processMessage(msg) {
  if (msg) {
    if (msg.content) {
      if (msg.content.length > 0) {  // Deep nesting
        // 200 lines of code...  // Long function
        const x = 42;  // Magic number
      }
    }
  }
}
```

## Output Format

```markdown
## Architecture Review

### Summary

| Category | Score | Status |
|----------|-------|--------|
| Layer Separation | X/10 | Status |
| SOLID Compliance | X/10 | Status |
| Coupling | X/10 | Status |
| Cohesion | X/10 | Status |
| Scalability | X/10 | Status |
| Technical Debt | X/10 | Status |

### Overall Architecture Score: XX/100

---

### Layer Violations

#### MEDIUM: Business Logic in API Route
**File:** src/app/api/chatbots/[id]/route.ts:45
**Issue:** Namespace generation logic in route handler

**Current:**
```typescript
export async function PUT(request, { params }) {
  const namespace = `${data.name}-user-${user.id}`;  // Business logic
  // ...
}
```

**Recommended:**
```typescript
// Move to service
export async function PUT(request, { params }) {
  const chatbot = await chatbotService.update(params.id, data);
  // ...
}
```

---

### SOLID Violations

#### SRP Violation: ChatbotService
**File:** src/domain/chatbot/chatbot.service.ts
**Issue:** Service handles too many responsibilities

**Current responsibilities:**
1. CRUD operations
2. Namespace management
3. Document indexing
4. Analytics tracking
5. Email notifications

**Recommended split:**
- ChatbotService (CRUD only)
- NamespaceService
- IndexingService
- AnalyticsService
- NotificationService

---

### Technical Debt

| Type | Count | Severity |
|------|-------|----------|
| TODO comments | X | Low |
| Long functions | X | Medium |
| Duplicate code | X | High |
| Magic numbers | X | Low |

### Dependency Graph

```
┌─────────────┐     ┌─────────────┐
│   API       │────►│   Domain    │
└─────────────┘     └─────────────┘
      │                   │
      │                   ▼
      │            ┌─────────────┐
      └───────────►│Infrastructure│
                   └─────────────┘
```

**Circular Dependencies Found:** 0 ✅

### Recommendations

1. Extract business logic from API routes
2. Split ChatbotService into focused services
3. Add dependency injection for testability
4. Create shared constants file for magic numbers
5. Refactor long functions in rag-engine.ts
```

## Integration

This agent is used by:
- `/full-review` command (all components)
- Can be invoked directly for architecture-focused review

## Related Agents

- `nextjs-standards-reviewer` - Framework patterns
- `drizzle-schema-reviewer` - Database architecture
- `saas-security-auditor` - Security architecture
