# API Docs Generator Agent

Generate comprehensive API reference documentation for AI BotKit's REST APIs, including examples, error handling, and SDKs.

## Purpose

This agent generates:
- Complete API reference (docs/API.md)
- RAG engine internals (docs/RAG_ENGINE.md)
- Database documentation (docs/DATABASE.md)
- Authentication guide (docs/AUTHENTICATION.md)
- WordPress hooks reference (docs/WORDPRESS_HOOKS.md)

## When to Use

- Creating API documentation for developers
- Generating SDK usage examples
- Documenting internal systems
- `/fit-quality` developer documentation phase

## What Gets Analyzed

```
┌─────────────────────────────────────────────────────────────────────┐
│                    API DOCUMENTATION SOURCES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. API CONTRACTS                                                    │
│     specs/contracts/*.md                                             │
│     → Detailed endpoint documentation                                │
│                                                                      │
│  2. SOURCE CODE                                                      │
│     saas/src/app/api/**/*.ts                                         │
│     → Request/response examples                                      │
│                                                                      │
│  3. ZOD SCHEMAS                                                      │
│     Validation schemas in route files                                │
│     → Request body documentation                                     │
│                                                                      │
│  4. DRIZZLE SCHEMA                                                   │
│     saas/src/lib/db/schema.ts                                        │
│     → Database documentation                                         │
│                                                                      │
│  5. WORDPRESS PLUGIN                                                 │
│     wordpress-plugin/includes/                                       │
│     → Hooks and filters reference                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Output Templates

### docs/API.md

```markdown
# AI BotKit API Reference

## Overview

The AI BotKit API provides programmatic access to chatbot management, conversations, and document processing.

**Base URL:** `https://app.aibotkit.io/api`

## Authentication

All authenticated endpoints require a valid session cookie or API token.

### Session Authentication (Web)

```typescript
// Automatically handled by browser cookies after login
```

### API Token Authentication (Server-to-Server)

```bash
curl -X GET https://app.aibotkit.io/api/chatbots \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

## Rate Limits

| Endpoint | Guest | Free | Basic | Essential | Business |
|----------|-------|------|-------|-----------|----------|
| POST /api/chat | 20/min | 100/day | 500/day | 2000/day | 10000/day |
| GET /api/chatbots | - | 60/min | 60/min | 120/min | 300/min |
| POST /api/documents | - | 10/hour | 20/hour | 50/hour | 100/hour |

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

## Error Handling

All errors follow a consistent format:

```json
{
  "success": false,
  "error": "Human-readable error message",
  "code": "ERROR_CODE",
  "details": {}
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| VALIDATION_ERROR | 400 | Invalid request body |
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Insufficient permissions |
| NOT_FOUND | 404 | Resource not found |
| RATE_LIMIT_EXCEEDED | 429 | Too many requests |
| INTERNAL_ERROR | 500 | Server error |

---

## Chat API

### Send Message

Send a message to a chatbot and receive a streaming response.

```
POST /api/chat
```

#### Request

```typescript
interface ChatRequest {
  chatbotId: string;       // Required: Chatbot namespace
  message: string;         // Required: User message (max 4000 chars)
  conversationId?: string; // Optional: Continue existing conversation
  sessionId?: string;      // Optional: Guest session identifier
}
```

#### Example

```bash
curl -X POST https://app.aibotkit.io/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "chatbotId": "abc123-def456",
    "message": "What are your business hours?"
  }'
```

#### Response (Streaming)

The response uses Server-Sent Events (SSE) format:

```
Content-Type: text/event-stream

data: {"choices":[{"delta":{"content":"Our"}}]}

data: {"choices":[{"delta":{"content":" business"}}]}

data: {"choices":[{"delta":{"content":" hours"}}]}

data: {"choices":[{"delta":{"content":" are..."}}]}

data: [DONE]
```

#### JavaScript Example

```javascript
const response = await fetch('/api/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    chatbotId: 'abc123',
    message: 'Hello!'
  })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value);
  const lines = chunk.split('\n');

  for (const line of lines) {
    if (line.startsWith('data: ')) {
      const data = line.slice(6);
      if (data === '[DONE]') break;

      const parsed = JSON.parse(data);
      const content = parsed.choices[0]?.delta?.content;
      if (content) {
        console.log(content);
      }
    }
  }
}
```

---

## Chatbots API

### List Chatbots

```
GET /api/chatbots
```

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | number | 1 | Page number |
| limit | number | 10 | Items per page (max 100) |

#### Response

```json
{
  "success": true,
  "chatbots": [
    {
      "id": 1,
      "name": "Customer Support Bot",
      "style": {
        "primaryColor": "#10B981",
        "position": "bottom-right"
      },
      "active": true,
      "totalConversations": 150,
      "totalMessages": 1234,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 3,
    "totalPages": 1
  }
}
```

---

### Create Chatbot

```
POST /api/chatbots
```

#### Request

```json
{
  "name": "My Support Bot",
  "style": {
    "primaryColor": "#10B981",
    "secondaryColor": "#F3F4F6",
    "position": "bottom-right",
    "avatarUrl": "https://example.com/avatar.png"
  },
  "messagesTemplate": {
    "systemPrompt": "You are a helpful customer support assistant.",
    "welcomeMessage": "Hello! How can I help you today?",
    "fallbackMessage": "I don't have information about that."
  }
}
```

#### Response

```json
{
  "success": true,
  "chatbot": {
    "id": 5,
    "name": "My Support Bot",
    "namespace": "uuid-v4-here",
    "style": {...},
    "messagesTemplate": {...},
    "active": true,
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

---

### Get Chatbot

```
GET /api/chatbots/{id}
```

#### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | number | Chatbot ID |

#### Response

```json
{
  "success": true,
  "chatbot": {
    "id": 1,
    "name": "Customer Support Bot",
    "namespace": "abc123-def456",
    "style": {...},
    "messagesTemplate": {...},
    "active": true,
    "totalConversations": 150,
    "documents": [
      {
        "id": 10,
        "title": "FAQ Document",
        "status": "completed",
        "characterCount": 5000
      }
    ]
  }
}
```

---

### Update Chatbot

```
PATCH /api/chatbots/{id}
```

#### Request

```json
{
  "name": "Updated Bot Name",
  "style": {
    "primaryColor": "#3B82F6"
  }
}
```

---

### Delete Chatbot

```
DELETE /api/chatbots/{id}
```

#### Response

```json
{
  "success": true,
  "message": "Chatbot deleted successfully"
}
```

---

## Documents API

### Upload Document

```
POST /api/documents
```

#### Request (File Upload)

```bash
curl -X POST https://app.aibotkit.io/api/documents \
  -H "Authorization: Bearer TOKEN" \
  -F "file=@document.pdf" \
  -F "chatbotId=1"
```

#### Request (URL Import)

```json
{
  "url": "https://example.com/page",
  "chatbotId": 1,
  "title": "Example Page"
}
```

#### Response

```json
{
  "success": true,
  "document": {
    "id": 15,
    "title": "document.pdf",
    "status": "processing",
    "characterCount": 0,
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

### Document Status

| Status | Description |
|--------|-------------|
| pending | Waiting to be processed |
| processing | Currently being indexed |
| completed | Successfully indexed |
| failed | Processing failed |

---

## Webhooks

### Stripe Webhook

```
POST /api/stripe/webhook
```

Handles Stripe subscription events. Requires valid Stripe signature.

#### Supported Events

| Event | Action |
|-------|--------|
| checkout.session.completed | Create subscription |
| customer.subscription.updated | Update plan |
| customer.subscription.deleted | Cancel subscription |
| invoice.payment_failed | Mark as unpaid |

---

## WordPress Integration API

### Connect WordPress

```
POST /api/connect-wordpress
```

Initiates connection with a WordPress site.

#### Request

```json
{
  "siteUrl": "https://example.com",
  "nonce": "wp-generated-nonce"
}
```

### Sync Content

```
POST /api/wordpress/sync-content
```

Receives content from WordPress for RAG indexing.

---

## SDK Examples

### JavaScript/TypeScript

```typescript
import { AIBotKit } from '@aibotkit/sdk';

const client = new AIBotKit({
  apiKey: 'your-api-key'
});

// Create chatbot
const chatbot = await client.chatbots.create({
  name: 'My Bot'
});

// Send message
const stream = await client.chat.send({
  chatbotId: chatbot.namespace,
  message: 'Hello!'
});

for await (const chunk of stream) {
  process.stdout.write(chunk.content);
}
```

### Python

```python
from aibotkit import AIBotKit

client = AIBotKit(api_key="your-api-key")

# Create chatbot
chatbot = client.chatbots.create(name="My Bot")

# Send message
for chunk in client.chat.send(
    chatbot_id=chatbot.namespace,
    message="Hello!"
):
    print(chunk.content, end="")
```

### cURL

```bash
# Create chatbot
curl -X POST https://app.aibotkit.io/api/chatbots \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "My Bot"}'

# Send message
curl -X POST https://app.aibotkit.io/api/chat \
  -H "Content-Type: application/json" \
  -d '{"chatbotId": "namespace", "message": "Hello!"}'
```
```

### docs/RAG_ENGINE.md

```markdown
# RAG Engine Documentation

## Overview

The RAG (Retrieval-Augmented Generation) engine is the core AI system that powers AI BotKit chatbots.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RAG Engine Flow                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  User Message                                            │
│       │                                                  │
│       ▼                                                  │
│  ┌─────────────┐    ┌─────────────┐                     │
│  │   Context   │───►│   Pinecone  │                     │
│  │  Retrieval  │    │   Search    │                     │
│  └──────┬──────┘    └─────────────┘                     │
│         │                                                │
│         ▼                                                │
│  ┌─────────────┐                                        │
│  │   Prompt    │  System Prompt + Context + History     │
│  │ Construction│                                        │
│  └──────┬──────┘                                        │
│         │                                                │
│         ▼                                                │
│  ┌─────────────┐    ┌─────────────┐                     │
│  │     LLM     │───►│  Together   │                     │
│  │   Request   │    │     AI      │                     │
│  └──────┬──────┘    └─────────────┘                     │
│         │                                                │
│         ▼                                                │
│  ┌─────────────┐                                        │
│  │  Response   │  Server-Sent Events                    │
│  │  Streaming  │                                        │
│  └─────────────┘                                        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Components

### Context Retrieval

Uses Pinecone hybrid search (dense + sparse vectors) to find relevant documents.

```typescript
const searchResults = await pineconeClient.hybridSearch({
  query: userMessage,
  namespace: chatbot.namespace,
  topK: 5
});
```

### Prompt Construction

Builds the final prompt with:
1. System instructions
2. Business context
3. Retrieved documents
4. Conversation history
5. User message

### LLM Providers

| Provider | Model | Use Case |
|----------|-------|----------|
| Together AI | Llama-3-70b | Primary |
| OpenAI | gpt-4-turbo | Fallback |
| Google | gemini-pro | Alternative |

### Tool Execution

Supported tools:
- `collectData`: Form collection
- `createTicket`: Support ticket creation
- `suggestHandover`: Human handover

## Configuration

### Environment Variables

```bash
PINECONE_API_KEY=your-key
PINECONE_INDEX=your-index
TOGETHER_API_KEY=your-key
OPENAI_API_KEY=your-key (optional)
```

### Chatbot Settings

```typescript
messagesTemplate: {
  namespace: "uuid",           // Pinecone namespace
  systemPrompt: "...",         // AI instructions
  modelConfig: {
    model: "meta-llama/Llama-3-70b-chat-hf",
    temperature: 0.7,
    maxTokens: 2048
  }
}
```
```

### docs/WORDPRESS_HOOKS.md

```markdown
# WordPress Hooks Reference

## Actions

### Plugin Lifecycle

| Hook | Description |
|------|-------------|
| `ai_botkit_activated` | Fired when plugin is activated |
| `ai_botkit_deactivated` | Fired when plugin is deactivated |

### Content Sync

| Hook | Parameters | Description |
|------|------------|-------------|
| `ai_botkit_post_synced` | `$post_id`, `$response` | After post synced to SaaS |
| `ai_botkit_post_deleted` | `$post_id` | After post removed from SaaS |

### Chat Events

| Hook | Parameters | Description |
|------|------------|-------------|
| `ai_botkit_before_render` | `$chatbot_id` | Before chatbot widget renders |
| `ai_botkit_after_render` | `$chatbot_id` | After chatbot widget renders |

## Filters

### Configuration

| Filter | Parameters | Description |
|--------|------------|-------------|
| `ai_botkit_embed_url` | `$url` | Modify embed script URL |
| `ai_botkit_widget_attributes` | `$attrs`, `$chatbot_id` | Modify widget HTML attributes |

### Content

| Filter | Parameters | Description |
|--------|------------|-------------|
| `ai_botkit_sync_post_types` | `$post_types` | Filter which post types sync |
| `ai_botkit_sync_content` | `$content`, `$post_id` | Filter content before sync |

## Usage Examples

```php
// Modify synced content
add_filter('ai_botkit_sync_content', function($content, $post_id) {
    // Remove shortcodes before syncing
    return strip_shortcodes($content);
}, 10, 2);

// Add custom post types to sync
add_filter('ai_botkit_sync_post_types', function($types) {
    $types[] = 'product';
    $types[] = 'faq';
    return $types;
});

// Track sync events
add_action('ai_botkit_post_synced', function($post_id, $response) {
    error_log("Post {$post_id} synced to AI BotKit");
}, 10, 2);
```
```

## Integration

This agent is invoked by:
- `/fit-quality` command (Phase 5)
- `/update-docs` command

## Related Agents

- `api-contract-generator` - Provides API contracts
- `documentation-generator` - Generates user documentation
- `data-modeler` - Provides database documentation
