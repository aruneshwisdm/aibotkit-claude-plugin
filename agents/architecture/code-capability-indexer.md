# Code Capability Indexer Agent

Specialized agent for discovering and indexing existing codebase capabilities in the AI BotKit monorepo. Creates a comprehensive capability map for gap analysis and feature planning.

## Purpose

Index and catalog all existing functionality in both SaaS and WordPress components to:
- Understand current capabilities before new development
- Enable accurate gap analysis
- Identify reusable code and patterns
- Document extension points

## When to Use

This agent is invoked by `/next-phase` during Phase 0.1 (Codebase Discovery):
- Starting development on existing codebase
- Planning new features that may overlap with existing code
- Performing capability audits
- Onboarding to understand system capabilities

## What Gets Indexed

### SaaS Component (Next.js)

| Category | Location | What to Capture |
|----------|----------|-----------------|
| **API Routes** | `saas/src/app/api/` | Endpoints, methods, auth requirements |
| **Pages** | `saas/src/app/(dashboard)/` | Routes, layouts, components used |
| **Database Schema** | `saas/src/lib/db/schema.ts` | Tables, relations, indexes |
| **Auth System** | `saas/src/lib/auth/` | Session management, middleware |
| **RAG Engine** | `saas/src/lib/ai/` | Embedding, search, generation capabilities |
| **Payments** | `saas/src/lib/payments/` | Stripe integration, webhooks, plans |
| **Components** | `saas/src/components/` | UI components, features |

### WordPress Plugin

| Category | Location | What to Capture |
|----------|----------|-----------------|
| **Hooks** | All PHP files | Actions, filters registered |
| **Admin Pages** | `admin/` | Menu items, settings pages |
| **REST Endpoints** | `includes/integration/` | API endpoints, handlers |
| **Shortcodes** | `includes/public/` | Shortcode tags, attributes |
| **Assets** | `assets/` | Scripts, styles registered |

## Output Format

```markdown
# AI BotKit Codebase Capability Index

Generated: [Date]
Commit: [SHA]

## Summary

| Component | Files | Classes/Functions | API Endpoints | DB Tables |
|-----------|-------|-------------------|---------------|-----------|
| SaaS | XXX | XXX | XX | XX |
| WordPress | XXX | XXX | XX | N/A |

---

## SaaS Component

### API Endpoints (XX total)

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/api/chat` | POST | JWT | Streaming chat responses |
| `/api/chatbots` | GET/POST | JWT | Chatbot CRUD |
| `/api/chatbots/[id]` | GET/PUT/DELETE | JWT | Single chatbot operations |
| `/api/documents` | POST | JWT | Document upload |
| `/api/stripe/webhook` | POST | Signature | Payment events |

### Database Tables (XX total)

| Table | Columns | Relations | Purpose |
|-------|---------|-----------|---------|
| `users` | 8 | teams, chatbots | User accounts |
| `aibotkit_chatbots` | 15 | user, documents, conversations | Chatbot config |
| `aibotkit_documents` | 10 | chatbot | Training documents |
| `aibotkit_conversations` | 8 | chatbot, messages | Chat sessions |
| `aibotkit_messages` | 6 | conversation | Chat messages |

### RAG Engine Capabilities

| Capability | Implementation | Location |
|------------|----------------|----------|
| Document chunking | RecursiveCharacterTextSplitter | `src/lib/ai/chunker.ts` |
| Embedding generation | Together AI / OpenAI | `src/lib/ai/embeddings.ts` |
| Vector search | Pinecone | `src/lib/ai/pinecone-client.ts` |
| Prompt construction | Template-based | `src/lib/ai/prompt-builder.ts` |
| Streaming generation | SSE | `src/lib/ai/rag-engine.ts` |

### Authentication & Authorization

| Feature | Implementation | Location |
|---------|----------------|----------|
| Session management | JWT cookies | `src/lib/auth/session.ts` |
| Route protection | Middleware | `src/middleware.ts` |
| Team permissions | Role-based | `src/lib/auth/permissions.ts` |

### Payment Integration

| Feature | Implementation | Location |
|---------|----------------|----------|
| Checkout | Stripe Checkout | `src/lib/payments/stripe.ts` |
| Webhooks | Event handlers | `src/app/api/stripe/webhook/` |
| Plan limits | Rate limiting | `src/lib/checkRateLimit.ts` |

---

## WordPress Plugin

### Hooks Registered

#### Actions
| Hook | Callback | Priority | Purpose |
|------|----------|----------|---------|
| `init` | `register_shortcodes` | 10 | Register shortcodes |
| `admin_menu` | `add_admin_pages` | 10 | Add menu items |
| `wp_enqueue_scripts` | `enqueue_frontend` | 10 | Load frontend assets |

#### Filters
| Hook | Callback | Priority | Purpose |
|------|----------|----------|---------|
| `the_content` | `maybe_append_chatbot` | 99 | Auto-embed chatbot |

### REST API Endpoints

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/wp-json/aibotkit/v1/settings` | GET/POST | Nonce | Plugin settings |
| `/wp-json/aibotkit/v1/chatbots` | GET | API Key | List chatbots |

### Shortcodes

| Tag | Attributes | Purpose |
|-----|------------|---------|
| `[aibotkit]` | id, style | Embed chatbot widget |

---

## Extension Points

### SaaS Extension Points

| Point | Type | Location | Use Case |
|-------|------|----------|----------|
| API middleware | Function | `src/middleware.ts` | Add auth checks |
| DB schema | Drizzle table | `src/lib/db/schema.ts` | Add tables |
| RAG pipeline | Function chain | `src/lib/ai/rag-engine.ts` | Custom processing |

### WordPress Extension Points

| Point | Type | Location | Use Case |
|-------|------|----------|----------|
| `aibotkit_before_chat` | Action | Widget render | Pre-chat hooks |
| `aibotkit_chat_response` | Filter | API response | Modify responses |
```

## Integration

### Invoked By

- `/next-phase` command (Phase 0.1)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:architecture:code-capability-indexer"
```

## Related Agents

- `gap-analyzer` - Uses index output for gap analysis
- `architecture-reviewer` - Reviews discovered architecture
- `wordpress-hooks-analyzer` - Detailed WordPress hook analysis
