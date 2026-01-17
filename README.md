# AI BotKit Engineering Plugin

A Claude Code plugin providing specialized development workflows for the AI BotKit monorepo - a SaaS chatbot platform with WordPress plugin integration.

## Overview

- **22 Agents** for code review, testing, security auditing, accessibility, architecture analysis, orchestration, and documentation sync
- **6 Commands** for development workflows
- **3 Skills** for domain knowledge

## Quick Start

See the [Getting Started Guide](GETTING_STARTED.md) for detailed installation and usage instructions.

```bash
# Clone and install
git clone https://github.com/WisdmLabs/aibotkit-claude-plugin.git
claude plugins add /path/to/aibotkit-claude-plugin

# Run your first review
cd /path/to/ai-botkit-mono-repo
claude
> /full-review
```

## Documentation

| Document | Description |
|----------|-------------|
| [Getting Started](GETTING_STARTED.md) | Installation, setup, and first steps |
| [Commands Reference](docs/COMMANDS.md) | Detailed command documentation |
| [Agents Reference](docs/AGENTS.md) | Agent descriptions and usage |
| [Skills Reference](docs/SKILLS.md) | Skill documentation |
| [Workflow Examples](docs/WORKFLOWS.md) | Real-world workflow examples |

## Installation

```bash
# Add the plugin to Claude Code
claude plugins add /path/to/aibotkit-claude-plugin
```

## Commands

| Command | Description |
|---------|-------------|
| `/full-review` | Comprehensive multi-agent code review for SaaS and WordPress plugin |
| `/next-phase` | 21-phase development lifecycle with discovery, SDD methodology, and quality gates |
| `/deploy-saas` | SaaS deployment checklist and validation |
| `/test-rag` | RAG engine testing workflow |
| `/sync-db` | Database schema synchronization and migration workflow |
| `/update-docs` | Automatic GitBook documentation sync from SaaS/WordPress code |

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

Guides development through 21 phases (3 Discovery + 18 SDD):

**Discovery Phases (0.1-0.3):**
- Codebase Discovery - Index existing code
- Documentation Recovery - Fill missing docs
- Gap Analysis - Compare current vs requirements

**SDD Phases (0-12):**
- 0: Project Init - Governance setup
- 0.5: Clarification - Requirements refinement
- 1-5: Planning & Specs - Estimation, analysis, design, architecture, specification
- 5.5-5.8: Quality Gates - Cross-analysis, req-spec validation, test cases, dependencies
- 6-6.5: Implementation - Coding with spec validation
- 7-8: Testing - Complete test writing, test & fix loop (until 100% pass)
- 9-10: Review & Fix - Code review loop (until quality threshold)
- 11-12: Finalization - Documentation and deployment

```bash
/next-phase              # Start from discovery
/next-phase 4            # Continue from phase 4
/next-phase --brief requirements.md  # Provide requirements file
```

### /sync-db

Database schema synchronization workflow:

```bash
/sync-db                    # Full schema sync (generate + migrate)
/sync-db --generate-only    # Only generate migration files
/sync-db --push             # Push schema directly (dev only)
/sync-db --studio           # Open Drizzle Studio
```

### /update-docs

Automatically sync GitBook documentation with SaaS and WordPress codebases:

```bash
/update-docs                    # Full sync with diff preview
/update-docs --apply            # Apply changes without confirmation
/update-docs --section plans    # Update only plans & billing docs
/update-docs --validate         # Validate only, no changes
```

Syncs:
- Plan limits from `saas/src/lib/checkRateLimit.ts`
- Features from code and `wordpress-plugin/readme.txt`
- Validates all internal links and image references

## Agents

### Review Agents (5)

| Agent | Purpose |
|-------|---------|
| `nextjs-standards-reviewer` | Next.js 16 App Router patterns, Server/Client components |
| `api-integration-reviewer` | SaaS ↔ WordPress plugin API consistency |
| `accessibility-guardian` | WCAG 2.1 AA compliance, keyboard navigation, screen reader support |
| `architecture-reviewer` | SOLID principles, clean architecture, code coupling analysis |
| `code-fixer` | Fix code quality issues from reviews, security fixes, standards compliance |

### AI Agents (1)

| Agent | Purpose |
|-------|---------|
| `rag-engine-reviewer` | RAG implementation, Pinecone integration, prompt construction |

### Database Agents (1)

| Agent | Purpose |
|-------|---------|
| `drizzle-schema-reviewer` | Schema design, indexes, migrations, query patterns |

### Security Agents (2)

| Agent | Purpose |
|-------|---------|
| `saas-security-auditor` | Auth, authorization, XSS, CSRF, API security |
| `stripe-integration-reviewer` | Payment security, webhooks, PCI compliance |

### WordPress Agents (3)

| Agent | Purpose |
|-------|---------|
| `wordpress-standards-reviewer` | WPCS compliance, i18n, hooks |
| `wordpress-security-auditor` | SQL injection, XSS, CSRF, capabilities |
| `wordpress-hooks-analyzer` | Discover and document all WordPress hooks in plugin |

### Architecture Agents (1)

| Agent | Purpose |
|-------|---------|
| `code-capability-indexer` | Index codebase capabilities for gap analysis |

### Orchestration Agents (3)

| Agent | Purpose |
|-------|---------|
| `gap-analyzer` | Compare current capabilities vs requirements |
| `requirements-spec-validator` | Validate spec coverage of requirements (quality gate) |
| `test-case-generator` | Generate manual test cases from specifications |

### Testing Agents (5)

| Agent | Purpose |
|-------|---------|
| `unit-test-writer` | Write Vitest unit tests with mocking for Drizzle, Stripe, Pinecone |
| `e2e-test-generator` | Generate Playwright E2E tests for user flows |
| `integration-test-specialist` | Write integration tests for API, DB, external services |
| `e2e-test-runner` | Execute tests, parse failures, coordinate fixes |
| `bug-fixer` | Analyze and fix test failures |

### Documentation Agents (1)

| Agent | Purpose |
|-------|---------|
| `documentation-updater` | Sync GitBook docs with code, validate links, update plan limits |

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

## Support

- **Issues:** [GitHub Issues](https://github.com/WisdmLabs/aibotkit-claude-plugin/issues)
- **Documentation:** See [docs/](docs/) folder
- **Claude Code Help:** Run `/help` in Claude Code

## License

MIT

## Author

WisdmLabs - https://wisdmlabs.com
