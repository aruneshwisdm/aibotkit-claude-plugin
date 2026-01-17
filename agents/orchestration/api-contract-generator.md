# API Contract Generator Agent

Generate API contract documentation from AI BotKit's Next.js route handlers and WordPress REST endpoints.

## Purpose

This agent analyzes API routes and generates:
- HTTP method and path documentation
- Request/response schemas (from Zod validation)
- Error response documentation
- Authentication requirements
- Rate limiting information

## When to Use

- Document existing API endpoints
- Generate API contracts for `/fit-quality`
- Create API reference documentation
- Prepare for API versioning

## What Gets Analyzed

### SaaS Component (Next.js App Router)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    NEXT.JS API ROUTE ANALYSIS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUT: saas/src/app/api/**/*.ts                                     │
│                                                                      │
│  EXTRACT:                                                            │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  1. ROUTE HANDLERS                                                   │
│     - GET, POST, PUT, PATCH, DELETE functions                        │
│     - Dynamic route parameters ([id], [chatbotId])                   │
│     - Route groups and layouts                                       │
│                                                                      │
│  2. REQUEST VALIDATION (Zod schemas)                                 │
│     - Input schemas                                                  │
│     - Query parameter validation                                     │
│     - Body payload validation                                        │
│                                                                      │
│  3. RESPONSE FORMATS                                                 │
│     - Success response structure                                     │
│     - Error response structure                                       │
│     - Streaming responses (SSE)                                      │
│                                                                      │
│  4. AUTHENTICATION                                                   │
│     - getUser() / getTeamForUser() usage                             │
│     - Public vs protected endpoints                                  │
│     - Admin-only endpoints                                           │
│                                                                      │
│  5. RATE LIMITING                                                    │
│     - checkRateLimit() usage                                         │
│     - Per-endpoint limits                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### WordPress Component

```
┌─────────────────────────────────────────────────────────────────────┐
│                 WORDPRESS REST API ANALYSIS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUT: wordpress-plugin/includes/integration/class-rest-api.php     │
│                                                                      │
│  EXTRACT:                                                            │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  1. REST ROUTES                                                      │
│     - register_rest_route() calls                                    │
│     - Namespace and route paths                                      │
│     - Methods (GET, POST, etc.)                                      │
│                                                                      │
│  2. PERMISSION CALLBACKS                                             │
│     - Permission callback functions                                  │
│     - Capability requirements                                        │
│                                                                      │
│  3. REQUEST ARGUMENTS                                                │
│     - 'args' array with validation                                   │
│     - Required vs optional parameters                                │
│     - Sanitize callbacks                                             │
│                                                                      │
│  4. AJAX ENDPOINTS                                                   │
│     - wp_ajax_* actions                                              │
│     - Nonce verification                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## AI BotKit API Endpoints

### SaaS Endpoints

| Category | Endpoints |
|----------|-----------|
| Chat | `/api/chat` |
| Chatbots | `/api/chatbots`, `/api/chatbots/[id]/*` |
| Documents | `/api/documents`, `/api/documents/[id]/*` |
| Users | `/api/user`, `/api/user/*` |
| Teams | `/api/team/*` |
| Payments | `/api/stripe/*` |
| WordPress | `/api/wordpress/*`, `/api/connect-wordpress` |
| Admin | `/api/admin/*` |

## Output Format

```markdown
# API Contracts

## Overview

| Component | Endpoints | Auth Methods |
|-----------|-----------|--------------|
| SaaS | 28 | JWT, API Key |
| WordPress | 4 | Bearer Token, Nonce |

---

## SaaS API Contracts

### Chat API

#### POST /api/chat

**Purpose:** Send a message to a chatbot and receive a streaming AI response.

**Authentication:** Optional (supports guest users)

**Rate Limits:**
- Guest: 20 messages/minute, 100 messages/day per IP
- Authenticated: Plan-based limits

**Request:**

```typescript
// Content-Type: application/json
interface ChatRequest {
  chatbotId: string;      // Required: Chatbot namespace/ID
  message: string;        // Required: User message (max 4000 chars)
  conversationId?: string; // Optional: Existing conversation
  sessionId?: string;     // Optional: Guest session ID
}
```

**Response (Streaming):**

```typescript
// Content-Type: text/event-stream
// Server-Sent Events format

// Message chunks
data: {"choices":[{"delta":{"content":"Hello"}}]}

// Tool calls (optional)
data: {"choices":[{"delta":{"tool_calls":[{"function":{"name":"collectData"}}]}}]}

// Done signal
data: [DONE]
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid request body |
| 404 | CHATBOT_NOT_FOUND | Chatbot does not exist |
| 429 | RATE_LIMIT_EXCEEDED | Too many requests |
| 500 | INTERNAL_ERROR | Server error |

```json
{
  "success": false,
  "error": "Rate limit exceeded",
  "code": "RATE_LIMIT_EXCEEDED",
  "retryAfter": 60
}
```

---

### Chatbots API

#### GET /api/chatbots

**Purpose:** List all chatbots for the authenticated user.

**Authentication:** Required (JWT)

**Request:**

```typescript
// Query parameters
interface ListChatbotsQuery {
  page?: number;      // Default: 1
  limit?: number;     // Default: 10, Max: 100
  includeDeleted?: boolean; // Default: false
}
```

**Response:**

```typescript
interface ListChatbotsResponse {
  success: true;
  chatbots: {
    id: number;
    name: string;
    style: ChatbotStyle;
    active: boolean;
    totalConversations: number;
    totalMessages: number;
    createdAt: string;
    updatedAt: string;
  }[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

---

#### POST /api/chatbots

**Purpose:** Create a new chatbot.

**Authentication:** Required (JWT)

**Request:**

```typescript
interface CreateChatbotRequest {
  name: string;           // Required: 1-100 chars
  style?: {
    primaryColor?: string;   // Hex color
    secondaryColor?: string; // Hex color
    position?: 'bottom-right' | 'bottom-left';
    avatarUrl?: string;      // Valid URL
  };
  messagesTemplate?: {
    systemPrompt?: string;   // Max 2000 chars
    welcomeMessage?: string; // Max 500 chars
    fallbackMessage?: string; // Max 500 chars
  };
}
```

**Response:**

```typescript
interface CreateChatbotResponse {
  success: true;
  chatbot: {
    id: number;
    name: string;
    namespace: string;  // UUID for Pinecone
    // ... full chatbot object
  };
}
```

---

#### GET /api/chatbots/[id]

**Purpose:** Get a specific chatbot by ID.

**Authentication:** Required (JWT)

**Path Parameters:**
- `id` (number): Chatbot ID

**Response:**

```typescript
interface GetChatbotResponse {
  success: true;
  chatbot: Chatbot;
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 401 | UNAUTHORIZED | Not authenticated |
| 403 | FORBIDDEN | Not chatbot owner |
| 404 | NOT_FOUND | Chatbot does not exist |

---

### Documents API

#### POST /api/documents

**Purpose:** Upload a document for RAG indexing.

**Authentication:** Required (JWT)

**Request:**

```typescript
// Content-Type: multipart/form-data
interface UploadDocumentRequest {
  file: File;           // Required: PDF, TXT, DOCX, MD
  chatbotId: number;    // Required: Target chatbot
  title?: string;       // Optional: Override filename
}

// OR Content-Type: application/json for URL import
interface ImportUrlRequest {
  url: string;          // Required: Valid URL
  chatbotId: number;    // Required: Target chatbot
  title?: string;       // Optional: Page title
}
```

**Response:**

```typescript
interface UploadDocumentResponse {
  success: true;
  document: {
    id: number;
    title: string;
    status: 'pending' | 'processing' | 'completed' | 'failed';
    characterCount: number;
    createdAt: string;
  };
}
```

---

### Stripe Webhook API

#### POST /api/stripe/webhook

**Purpose:** Handle Stripe webhook events.

**Authentication:** Stripe signature verification

**Headers:**
- `stripe-signature`: Stripe webhook signature

**Events Handled:**

| Event | Action |
|-------|--------|
| `checkout.session.completed` | Create/upgrade subscription |
| `customer.subscription.updated` | Update subscription status |
| `customer.subscription.deleted` | Cancel subscription |
| `invoice.payment_failed` | Mark subscription as unpaid |

**Response:**

```typescript
// 200 OK on success
{ received: true }

// 400 Bad Request on signature failure
{ error: "Webhook signature verification failed" }
```

---

## WordPress API Contracts

### REST API Namespace: `/ai-botkit/v1`

#### POST /ai-botkit/v1/setToken

**Purpose:** Receive authentication token from SaaS.

**Authentication:** Nonce verification

**Request:**

```json
{
  "token": "eyJhbG...",  // JWT token from SaaS
  "nonce": "abc123"      // One-time nonce
}
```

**Response:**

```json
{
  "success": true,
  "message": "Token stored successfully"
}
```

---

#### GET /ai-botkit/v1/post-types

**Purpose:** List available post types for sync.

**Authentication:** Bearer token

**Response:**

```json
{
  "success": true,
  "post_types": [
    { "name": "post", "label": "Posts" },
    { "name": "page", "label": "Pages" }
  ]
}
```

---

#### GET /ai-botkit/v1/posts/{post_type}

**Purpose:** List posts of a specific type.

**Authentication:** Bearer token

**Path Parameters:**
- `post_type` (string): WordPress post type

**Query Parameters:**
- `page` (int): Page number
- `per_page` (int): Items per page

**Response:**

```json
{
  "success": true,
  "posts": [
    {
      "id": 123,
      "title": "Hello World",
      "excerpt": "...",
      "modified": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 50,
  "pages": 5
}
```

---

## AJAX Endpoints (WordPress)

| Action | Method | Nonce | Description |
|--------|--------|-------|-------------|
| `ai_botkit_test_api_connection` | POST | Required | Test SaaS connection |
| `ai_botkit_save_chatbot` | POST | Required | Save chatbot settings |
| `ai_botkit_get_chatbot` | GET | Required | Get chatbot details |
| `ai_botkit_upload_file` | POST | Required | Upload document |
| `ai_botkit_import_url` | POST | Required | Import URL content |

---

## Authentication Summary

### SaaS Authentication

| Method | Header | Usage |
|--------|--------|-------|
| JWT | `Cookie: session=...` | User sessions |
| API Key | `Authorization: Bearer ...` | WordPress integration |

### WordPress Authentication

| Method | Header/Parameter | Usage |
|--------|------------------|-------|
| Bearer Token | `Authorization: Bearer ...` | SaaS API calls |
| Nonce | `X-WP-Nonce` or `nonce` param | AJAX requests |

---

## Rate Limits

### SaaS

| Endpoint | Guest | Free | Basic | Essential | Business |
|----------|-------|------|-------|-----------|----------|
| /api/chat | 20/min | 100/day | 500/day | 2000/day | 10000/day |
| /api/chatbots | - | 3 total | 5 total | 10 total | 25 total |
| /api/documents | - | 50/bot | 100/bot | 500/bot | 2000/bot |

### WordPress

| Endpoint | Limit |
|----------|-------|
| /ai-botkit/v1/* | WordPress default (no custom limit) |
```

## Integration

This agent is invoked by:
- `/fit-quality` command (Phase 2)
- `/next-phase` command (Phase 4)

## Related Agents

- `spec-recovery-agent` - References API contracts
- `data-modeler` - Provides data model context
- `api-docs-generator` - Uses contracts for documentation
