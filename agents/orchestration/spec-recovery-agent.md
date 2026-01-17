# Spec Recovery Agent

Analyze existing AI BotKit codebase and generate functional specifications by reverse-engineering code behavior into documented requirements.

## Purpose

This agent examines source code to extract and document:
- Functional requirements (FR-xxx)
- Non-functional requirements (NFR-xxx)
- Acceptance criteria
- Feature traceability to source files

## When to Use

- Existing codebase lacks formal specifications
- Documentation is outdated or missing
- Need to understand what the code actually does
- Preparing for `/fit-quality` specification phase

## What Gets Analyzed

### SaaS Component (Next.js)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SAAS SPECIFICATION RECOVERY                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. API ROUTES (src/app/api/)                                        │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: HTTP methods, paths, request/response schemas           │
│     Output: FR for each endpoint functionality                       │
│     Example: FR-001: System shall provide streaming chat endpoint    │
│                                                                      │
│  2. RAG ENGINE (src/lib/ai/)                                         │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Context retrieval, prompt construction, streaming       │
│     Output: FR for RAG capabilities                                  │
│     Example: FR-010: System shall retrieve relevant context from     │
│              Pinecone using hybrid search                            │
│                                                                      │
│  3. DATABASE SCHEMA (src/lib/db/schema.ts)                           │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Tables, relations, constraints                          │
│     Output: FR for data management                                   │
│     Example: FR-020: System shall store chatbot configurations       │
│              including name, style, and message templates            │
│                                                                      │
│  4. AUTHENTICATION (src/lib/auth/)                                   │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Session management, JWT, OAuth flows                    │
│     Output: FR/NFR for auth requirements                             │
│     Example: NFR-001: System shall authenticate users via JWT        │
│              with 24-hour expiration                                 │
│                                                                      │
│  5. PAYMENTS (src/lib/payments/)                                     │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Stripe integration, webhook handlers, plans             │
│     Output: FR for billing features                                  │
│     Example: FR-030: System shall handle Stripe subscription         │
│              lifecycle events via webhooks                           │
│                                                                      │
│  6. REACT COMPONENTS (src/components/)                               │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: UI features, user interactions                          │
│     Output: FR for UI capabilities                                   │
│     Example: FR-040: System shall provide chatbot preview component  │
│              with real-time message rendering                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### WordPress Component

```
┌─────────────────────────────────────────────────────────────────────┐
│                 WORDPRESS SPECIFICATION RECOVERY                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. PLUGIN BOOTSTRAP (ai-botkit-for-lead-generation.php)             │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Plugin metadata, activation/deactivation hooks          │
│     Output: FR for plugin lifecycle                                  │
│                                                                      │
│  2. ADMIN FUNCTIONALITY (includes/admin/)                            │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Admin pages, settings, AJAX handlers                    │
│     Output: FR for admin features                                    │
│     Example: FR-100: Plugin shall provide admin dashboard for        │
│              managing SaaS connection                                │
│                                                                      │
│  3. REST API (includes/integration/class-rest-api.php)               │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: REST endpoints, authentication                          │
│     Output: FR for REST functionality                                │
│     Example: FR-110: Plugin shall expose REST endpoint for           │
│              receiving SaaS authentication token                     │
│                                                                      │
│  4. SAAS INTEGRATION (includes/integration/class-api-handler.php)    │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: API calls, token management, sync operations            │
│     Output: FR for SaaS integration                                  │
│     Example: FR-120: Plugin shall sync WordPress posts to            │
│              SaaS for RAG indexing                                   │
│                                                                      │
│  5. PUBLIC/FRONTEND (includes/public/)                               │
│     ─────────────────────────────────────────────────────────────   │
│     Extract: Shortcodes, frontend rendering                          │
│     Output: FR for public-facing features                            │
│     Example: FR-130: Plugin shall render chatbot widget via          │
│              [ai_botkit_widget] shortcode                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Analysis Patterns

### Good Patterns (Extract Requirements From)

```typescript
// Pattern: API endpoint with validation → FR for endpoint functionality
export async function POST(request: NextRequest) {
  const body = await request.json();
  const validated = chatSchema.parse(body);  // Extract: Input validation requirement
  const response = await ragEngine.generateResponse(validated);  // Extract: RAG functionality
  return new Response(response);  // Extract: Response format
}
// → FR-xxx: System shall accept chat requests with chatbotId and message,
//           validate input, and return streaming AI response
```

```typescript
// Pattern: Database schema → FR for data storage
export const chatbots = pgTable('aibotkit_chatbots', {
  id: bigserial('id', { mode: 'number' }).primaryKey(),
  name: text('name').notNull(),
  style: jsonb('style'),  // Extract: Configurable styling
  active: boolean('active').default(true),  // Extract: Enable/disable feature
});
// → FR-xxx: System shall store chatbot configurations including name, style,
//           and active status
```

```php
// Pattern: WordPress hook → FR for integration point
add_action('save_post', [$this, 'handle_post_sync']);
// → FR-xxx: Plugin shall synchronize post content to SaaS when posts are saved
```

### Anti-Patterns (Flag for Review)

```typescript
// Anti-pattern: Unclear business logic
if (someCondition && anotherCondition || thirdCondition) {
  // Complex conditional without documentation
}
// → FLAG: Complex logic requires clarification from developers
```

```typescript
// Anti-pattern: Hardcoded values without explanation
const LIMIT = 100;  // Why 100?
// → FLAG: Business rule needs documentation
```

## Output Format

```markdown
# Recovered Specifications

## Document Information

| Property | Value |
|----------|-------|
| Project | AI BotKit |
| Version | Recovered from code analysis |
| Date | [Generated date] |
| Source | [Analyzed directories] |

## Functional Requirements

### Chat & RAG Features

#### FR-001: Streaming Chat API
**Source:** `saas/src/app/api/chat/route.ts`
**Priority:** P0 (Critical)

**Description:**
System shall provide a streaming chat endpoint that accepts user messages
and returns AI-generated responses in real-time.

**Acceptance Criteria:**
- [ ] Accepts POST requests with chatbotId and message
- [ ] Validates input using Zod schema
- [ ] Retrieves context from Pinecone via RAG engine
- [ ] Streams response using Server-Sent Events
- [ ] Handles rate limiting per IP/user
- [ ] Stores conversation history

**Technical Notes:**
- Uses Together AI as primary LLM provider
- Falls back to OpenAI on failure
- Supports tool execution (forms, tickets, handover)

---

#### FR-002: Context Retrieval (RAG)
**Source:** `saas/src/lib/ai/rag-engine.ts`
**Priority:** P0 (Critical)

**Description:**
System shall retrieve relevant context from vector database to augment
LLM responses with domain-specific knowledge.

**Acceptance Criteria:**
- [ ] Performs hybrid search (dense + sparse) on Pinecone
- [ ] Retrieves top-K relevant documents
- [ ] Constructs prompt with system instructions and context
- [ ] Respects token limits for context window
- [ ] Falls back gracefully when no context found

---

[Continue for all discovered features...]

### Chatbot Management Features

#### FR-010: Chatbot CRUD Operations
**Source:** `saas/src/app/api/chatbots/`
**Priority:** P0 (Critical)

**Description:**
System shall provide complete CRUD operations for chatbot management.

**Acceptance Criteria:**
- [ ] Create chatbot with name, style, and configuration
- [ ] Read chatbot details and list user's chatbots
- [ ] Update chatbot settings and appearance
- [ ] Delete chatbot and associated data
- [ ] Validate ownership before operations

---

### WordPress Integration Features

#### FR-100: WordPress Plugin Dashboard
**Source:** `wordpress-plugin/includes/admin/class-admin.php`
**Priority:** P1 (High)

**Description:**
Plugin shall provide WordPress admin dashboard for managing SaaS connection
and chatbot settings.

**Acceptance Criteria:**
- [ ] Display connection status with SaaS
- [ ] Allow token configuration
- [ ] Show available chatbots from SaaS
- [ ] Configure sitewide/per-page chatbot embedding

---

## Non-Functional Requirements

### NFR-001: Authentication
**Source:** `saas/src/lib/auth/session.ts`
**Category:** Security

**Description:**
System shall authenticate users using JWT tokens with secure session management.

**Acceptance Criteria:**
- [ ] JWT tokens expire after 24 hours
- [ ] Tokens are signed with AUTH_SECRET
- [ ] Session cookies use httpOnly, secure, sameSite attributes
- [ ] Support for Google OAuth integration

---

### NFR-002: Rate Limiting
**Source:** `saas/src/lib/checkRateLimit.ts`
**Category:** Performance/Security

**Description:**
System shall enforce rate limits to prevent abuse.

**Acceptance Criteria:**
- [ ] 20 messages per minute per IP (guest)
- [ ] 100 messages per day per IP (guest)
- [ ] Plan-based limits for authenticated users

---

### NFR-003: Data Encryption
**Source:** `saas/src/lib/encryption.ts`
**Category:** Security

**Description:**
System shall encrypt sensitive data at rest.

**Acceptance Criteria:**
- [ ] Message content encrypted with AES-256-GCM
- [ ] Encryption key stored in environment variable
- [ ] IV and auth tag stored with encrypted data

---

## Traceability Matrix

| Requirement | Source File | Test Coverage |
|-------------|-------------|---------------|
| FR-001 | saas/src/app/api/chat/route.ts | Pending |
| FR-002 | saas/src/lib/ai/rag-engine.ts | Pending |
| FR-010 | saas/src/app/api/chatbots/ | Pending |
| FR-100 | wordpress-plugin/includes/admin/class-admin.php | Pending |
| NFR-001 | saas/src/lib/auth/session.ts | Pending |
| NFR-002 | saas/src/lib/checkRateLimit.ts | Pending |

## Flagged Items (Require Clarification)

| ID | Location | Issue | Priority |
|----|----------|-------|----------|
| FLAG-001 | rag-engine.ts:280 | Complex prompt construction logic | High |
| FLAG-002 | checkRateLimit.ts:101 | Hardcoded rate limits | Medium |
| FLAG-003 | class-admin.php:300 | Weak nonce generation | Critical |

## Summary

| Category | Count |
|----------|-------|
| Functional Requirements | XX |
| Non-Functional Requirements | XX |
| Flagged Items | XX |
| Source Files Analyzed | XX |
```

## Integration

This agent is invoked by:
- `/fit-quality` command (Phase 2)
- `/next-phase` command (Phase 0.2)

## Related Agents

- `code-capability-indexer` - Provides discovery data as input
- `data-modeler` - Generates data model from schema
- `api-contract-generator` - Generates API contracts
- `test-case-generator` - Uses specifications to generate tests
