# Documentation Generator Agent

Generate comprehensive user and developer documentation for AI BotKit projects.

## Purpose

This agent generates:
- README.md (project overview)
- USER_GUIDE.md (end-user documentation)
- DEVELOPER.md (developer guide)
- INSTALLATION.md (setup instructions)
- CONFIGURATION.md (environment variables)
- CONTRIBUTING.md (contribution guidelines)
- CHANGELOG.md (version history scaffold)

## When to Use

- Project lacks proper documentation
- Onboarding new team members
- Preparing for open-source release
- `/fit-quality` documentation phase

## What Gets Analyzed

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DOCUMENTATION SOURCES                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. DISCOVERY REPORT                                                 │
│     _project_specs/DISCOVERY_REPORT.md                               │
│     → Project overview, capabilities, structure                      │
│                                                                      │
│  2. SPECIFICATIONS                                                   │
│     specs/RECOVERED_SPECIFICATION.md                                 │
│     → Feature descriptions, requirements                             │
│                                                                      │
│  3. API CONTRACTS                                                    │
│     specs/contracts/*.md                                             │
│     → API usage examples                                             │
│                                                                      │
│  4. SOURCE CODE                                                      │
│     Package.json, .env.example, config files                         │
│     → Dependencies, configuration options                            │
│                                                                      │
│  5. EXISTING DOCS                                                    │
│     Any existing README, docs/                                       │
│     → Preserve existing content where appropriate                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Output Templates

### README.md

```markdown
# AI BotKit

> AI-powered chatbot platform with RAG capabilities for WordPress and SaaS.

## Features

- **RAG-Powered Responses**: Retrieval-Augmented Generation using Pinecone vector database
- **Multi-LLM Support**: Together AI, OpenAI, and Google Gemini integration
- **WordPress Integration**: Seamless embedding via plugin and shortcodes
- **Real-time Streaming**: Server-Sent Events for instant responses
- **Document Processing**: PDF, DOCX, TXT, and URL content indexing
- **Team Collaboration**: Multi-user teams with role-based access
- **Stripe Billing**: Subscription management with multiple tiers

## Quick Start

### Prerequisites

- Node.js 18+
- PostgreSQL 14+
- Pinecone account
- Together AI API key (or OpenAI/Gemini)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/ai-botkit-mono-repo.git
cd ai-botkit-mono-repo

# Install SaaS dependencies
cd saas
pnpm install

# Set up environment
cp .env.example .env
# Edit .env with your credentials

# Run database migrations
pnpm db:push

# Start development server
pnpm dev
```

### WordPress Plugin

1. Download the plugin from releases
2. Upload to `/wp-content/plugins/`
3. Activate in WordPress admin
4. Connect to your SaaS instance

## Documentation

- [User Guide](docs/USER_GUIDE.md)
- [Developer Guide](docs/DEVELOPER.md)
- [API Reference](docs/API.md)
- [Configuration](docs/CONFIGURATION.md)

## Tech Stack

### SaaS Application
- **Framework**: Next.js 16 (App Router)
- **Database**: PostgreSQL with Drizzle ORM
- **Vector DB**: Pinecone
- **AI**: Together AI, OpenAI, Google Gemini
- **Payments**: Stripe
- **UI**: Tailwind CSS, shadcn/ui

### WordPress Plugin
- **Minimum PHP**: 7.4
- **Minimum WordPress**: 5.8
- **Integration**: REST API, AJAX

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

[MIT License](LICENSE)
```

### USER_GUIDE.md

```markdown
# AI BotKit User Guide

## Getting Started

### Creating Your First Chatbot

1. **Sign Up**: Create an account at [app.aibotkit.io](https://app.aibotkit.io)
2. **Create Chatbot**: Click "New Chatbot" in the dashboard
3. **Configure Appearance**: Set colors, position, and avatar
4. **Add Knowledge**: Upload documents or import URLs
5. **Test**: Use the preview to test responses
6. **Deploy**: Get embed code for your website

### Adding Knowledge to Your Chatbot

Your chatbot's responses are powered by the documents you provide.

#### Supported File Types
- PDF documents
- Word documents (.docx)
- Plain text files (.txt)
- Markdown files (.md)
- Web URLs

#### Uploading Documents

1. Go to your chatbot's "Documents" tab
2. Click "Upload Document" or "Import URL"
3. Wait for processing (may take a few minutes)
4. Document status will change to "Completed"

#### Best Practices
- Use clear, well-structured content
- Break large documents into smaller files
- Include FAQs and common questions
- Update documents when information changes

### Customizing Your Chatbot

#### Appearance Settings

| Setting | Description |
|---------|-------------|
| Primary Color | Main button and accent color |
| Secondary Color | Background color |
| Position | Bottom-right or bottom-left |
| Avatar | Custom image URL |

#### Behavior Settings

| Setting | Description |
|---------|-------------|
| System Prompt | Instructions for the AI |
| Welcome Message | First message shown to users |
| Fallback Message | Response when no context found |

### Embedding on Your Website

#### JavaScript Embed

```html
<script src="https://app.aibotkit.io/embed.js"
        data-chatbot-id="your-chatbot-id"></script>
```

#### WordPress Plugin

1. Install and activate the AI BotKit plugin
2. Enter your API token in Settings
3. Select chatbots to display
4. Use shortcode: `[ai_botkit_widget id="123"]`

### Managing Conversations

- View all conversations in the dashboard
- Export conversation history
- See user engagement metrics
- Identify common questions

### Plans and Billing

| Feature | Free | Basic | Essential | Business |
|---------|------|-------|-----------|----------|
| Chatbots | 3 | 5 | 10 | 25 |
| Messages/day | 100 | 500 | 2000 | 10000 |
| Documents/bot | 50 | 100 | 500 | 2000 |
| Team Members | 1 | 3 | 10 | Unlimited |

## Troubleshooting

### Chatbot Not Responding

1. Check document processing status
2. Verify chatbot is active
3. Check rate limits
4. Review system prompt for issues

### Poor Response Quality

1. Add more relevant documents
2. Improve document formatting
3. Adjust system prompt
4. Test with different questions
```

### DEVELOPER.md

```markdown
# AI BotKit Developer Guide

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    AI BotKit Architecture                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │   Next.js   │    │  PostgreSQL │    │   Pinecone  │  │
│  │    SaaS     │◄──►│  (Drizzle)  │    │  (Vectors)  │  │
│  └──────┬──────┘    └─────────────┘    └──────▲──────┘  │
│         │                                      │         │
│         │ API                                  │ RAG     │
│         ▼                                      │         │
│  ┌─────────────┐    ┌─────────────┐    ┌──────┴──────┐  │
│  │  WordPress  │    │  Together   │    │    RAG      │  │
│  │   Plugin    │    │     AI      │◄───│   Engine    │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Project Structure

```
ai-botkit-mono-repo/
├── saas/                      # Next.js SaaS application
│   ├── src/
│   │   ├── app/              # App Router pages & API routes
│   │   │   ├── api/          # API endpoints
│   │   │   ├── (dashboard)/  # Dashboard pages
│   │   │   └── (login)/      # Auth pages
│   │   ├── lib/              # Core libraries
│   │   │   ├── ai/           # RAG engine, LLM clients
│   │   │   ├── db/           # Drizzle schema, queries
│   │   │   ├── auth/         # Authentication
│   │   │   └── payments/     # Stripe integration
│   │   └── components/       # React components
│   └── tests/                # Test files
│
├── wordpress-plugin/          # WordPress plugin
│   ├── includes/
│   │   ├── admin/            # Admin functionality
│   │   ├── integration/      # SaaS API integration
│   │   └── public/           # Frontend (shortcodes)
│   └── admin/views/          # Admin page templates
│
└── documentation/             # GitBook documentation
```

## Development Setup

### Prerequisites

```bash
# Required
node >= 18.0.0
pnpm >= 8.0.0
postgresql >= 14

# Accounts needed
- Pinecone (vector database)
- Together AI or OpenAI (LLM)
- Stripe (payments)
```

### Environment Setup

```bash
# Clone and install
git clone <repo-url>
cd ai-botkit-mono-repo/saas
pnpm install

# Create environment file
cp .env.example .env

# Required environment variables
DATABASE_URL=postgresql://user:pass@localhost:5432/aibotkit
AUTH_SECRET=your-32-char-secret
ENCRYPTION_KEY=your-64-char-hex-key
PINECONE_API_KEY=your-pinecone-key
TOGETHER_API_KEY=your-together-key
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Database Setup

```bash
# Push schema to database
pnpm db:push

# Generate migrations (for production)
pnpm db:generate

# Run migrations
pnpm db:migrate
```

### Running Locally

```bash
# Development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start
```

## Key Components

### RAG Engine (src/lib/ai/rag-engine.ts)

The RAG engine handles:
1. Context retrieval from Pinecone
2. Prompt construction with system instructions
3. LLM response generation
4. Response streaming
5. Tool execution (forms, tickets, handover)

```typescript
// Basic usage
const ragEngine = new RAGEngine();
const response = await ragEngine.generateResponse({
  chatbot,
  message: userMessage,
  conversationId,
  // ...
});
```

### Drizzle Schema (src/lib/db/schema.ts)

Key tables:
- `users` - User accounts
- `teams` - Team/organization accounts
- `aibotkit_chatbots` - Chatbot configurations
- `aibotkit_conversations` - Chat sessions
- `aibotkit_messages` - Encrypted messages
- `aibotkit_documents` - RAG documents

### Authentication (src/lib/auth/)

- JWT-based sessions
- Google OAuth support
- WordPress token authentication

## API Development

### Creating a New API Route

```typescript
// src/app/api/example/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getUser } from '@/lib/db/queries';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1).max(100),
});

export async function POST(request: NextRequest) {
  const user = await getUser();
  if (!user) {
    return NextResponse.json(
      { success: false, error: 'Unauthorized' },
      { status: 401 }
    );
  }

  const body = await request.json();
  const validated = schema.parse(body);

  // ... business logic

  return NextResponse.json({ success: true, data: result });
}
```

## Testing

### Running Tests

```bash
# Unit tests
pnpm test

# Integration tests
pnpm test:integration

# E2E tests
pnpm test:e2e

# Coverage report
pnpm test:coverage
```

### Writing Tests

```typescript
// tests/unit/lib/ai/rag-engine.test.ts
import { describe, it, expect, vi } from 'vitest';
import { RAGEngine } from '@/lib/ai/rag-engine';

describe('RAGEngine', () => {
  it('should retrieve context from Pinecone', async () => {
    // ...
  });
});
```

## Debugging

### Common Issues

1. **Database connection failed**
   - Check DATABASE_URL format
   - Verify PostgreSQL is running

2. **Pinecone errors**
   - Verify API key
   - Check index exists
   - Validate namespace

3. **Streaming not working**
   - Check Content-Type header
   - Verify SSE format
   - Check CORS settings

### Logging

```typescript
// Use structured logging
console.log('Processing chat request', {
  chatbotId,
  conversationId,
  messageLength: message.length,
});
```

## Deployment

### Vercel (Recommended)

```bash
# Deploy to Vercel
vercel deploy

# Production deployment
vercel --prod
```

### Docker

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN pnpm install --frozen-lockfile
RUN pnpm build
CMD ["pnpm", "start"]
```

## Code Style

- TypeScript strict mode
- ESLint + Prettier
- Zod for runtime validation
- Server Components by default
- 'use client' only when needed
```

### CONTRIBUTING.md

```markdown
# Contributing to AI BotKit

Thank you for your interest in contributing!

## Development Process

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Write/update tests
5. Submit a pull request

## Code Standards

- Follow existing code style
- Add TypeScript types
- Write tests for new features
- Update documentation

## Commit Messages

Use conventional commits:

```
feat: add new chatbot feature
fix: resolve streaming issue
docs: update API documentation
test: add unit tests for RAG engine
```

## Pull Request Process

1. Update README if needed
2. Add tests for changes
3. Ensure all tests pass
4. Request review from maintainers

## Questions?

Open an issue or discussion.
```

## Integration

This agent is invoked by:
- `/fit-quality` command (Phases 4-5)
- `/update-docs` command

## Related Agents

- `spec-recovery-agent` - Provides feature descriptions
- `api-contract-generator` - Provides API documentation
- `api-docs-generator` - Generates detailed API reference
