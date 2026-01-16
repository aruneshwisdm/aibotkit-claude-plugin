# AI BotKit Engineering Plugin

A Claude Code plugin providing specialized development workflows for the AI BotKit monorepo - a SaaS chatbot platform with WordPress plugin integration.

## Overview

- **8 Agents** for code review, security auditing, and RAG analysis
- **4 Commands** for development workflows
- **3 Skills** for domain knowledge

## Installation

```bash
# Add the plugin to Claude Code
claude /plugins add /path/to/aibotkit-claude-plugin
```

## Commands

| Command | Description |
|---------|-------------|
| `/full-review` | Comprehensive multi-agent code review for SaaS and WordPress plugin |
| `/next-phase` | 12-phase development lifecycle with discovery and gap analysis |
| `/deploy-saas` | SaaS deployment checklist and validation |
| `/test-rag` | RAG engine testing workflow |

### /full-review

Orchestrates 10 specialized agents in parallel to review:
- Next.js patterns and best practices
- RAG engine correctness
- Database schema design
- Stripe payment security
- WordPress coding standards
- Security vulnerabilities
- API integration consistency
- Accessibility compliance

```bash
/full-review                    # Review all components
/full-review --component saas   # Review SaaS only
/full-review --component wordpress  # Review WordPress plugin only
```

### /next-phase

Guides development through 12 phases:

**Discovery Phases:**
1. Codebase Discovery - Index existing code
2. Documentation Review - Identify doc gaps
3. Gap Analysis - Compare current vs requirements

**Implementation Phases:**
4. Requirements - Document detailed specs
5. Architecture - Design extensions
6. Specification - Technical specs
7. Implementation - Build features
8. Testing - Write and run tests
9. Code Review - Run /full-review
10. Documentation - Update docs
11. Pre-Deployment - Final checks
12. Deployment - Deploy to production

```bash
/next-phase              # Start from discovery
/next-phase 4            # Continue from phase 4
/next-phase --brief requirements.md  # Provide requirements file
```

## Agents

### Review Agents

| Agent | Purpose |
|-------|---------|
| `nextjs-standards-reviewer` | Next.js 16 App Router patterns, Server/Client components |
| `api-integration-reviewer` | SaaS ↔ WordPress plugin API consistency |

### AI Agents

| Agent | Purpose |
|-------|---------|
| `rag-engine-reviewer` | RAG implementation, Pinecone integration, prompt construction |

### Database Agents

| Agent | Purpose |
|-------|---------|
| `drizzle-schema-reviewer` | Schema design, indexes, migrations, query patterns |

### Security Agents

| Agent | Purpose |
|-------|---------|
| `saas-security-auditor` | Auth, authorization, XSS, CSRF, API security |
| `stripe-integration-reviewer` | Payment security, webhooks, PCI compliance |

### WordPress Agents

| Agent | Purpose |
|-------|---------|
| `wordpress-standards-reviewer` | WPCS compliance, i18n, hooks |
| `wordpress-security-auditor` | SQL injection, XSS, CSRF, capabilities |

## Skills

| Skill | Description |
|-------|-------------|
| `rag-development` | RAG patterns, vector search, prompt engineering |
| `nextjs-saas` | Next.js SaaS architecture patterns |
| `drizzle-patterns` | Drizzle ORM best practices |

## Project Context

This plugin is designed for the AI BotKit monorepo:

```
ai-botkit-mono-repo/
├── saas/                    # Next.js 16 SaaS application
│   ├── src/app/             # App Router pages and API routes
│   ├── src/lib/ai/          # RAG engine, AI clients
│   ├── src/lib/db/          # Drizzle schema and queries
│   └── src/lib/payments/    # Stripe integration
└── wordpress-plugin/        # WordPress plugin
    ├── admin/               # Admin interface
    ├── includes/            # Core functionality
    └── public/              # Frontend assets
```

### Technology Stack

**SaaS:**
- Next.js 16 (App Router)
- PostgreSQL + Drizzle ORM
- Stripe (subscriptions)
- Pinecone (vector database)
- Together AI / OpenAI / Gemini (LLM)
- Tailwind CSS + shadcn/ui

**WordPress Plugin:**
- PHP 7.4+
- WordPress 5.8+
- REST API integration

## Development

### Adding New Agents

1. Create agent file in `agents/{category}/{agent-name}.md`
2. Follow the agent template structure
3. Update this README
4. Bump version in `plugin.json`

### Adding New Commands

1. Create command file in `commands/{workflow}/{command-name}.md`
2. Follow the command template structure
3. Update this README
4. Bump version in `plugin.json`

## License

MIT

## Author

WisdmLabs - https://wisdmlabs.com
