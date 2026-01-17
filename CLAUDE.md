# AI BotKit Engineering Plugin - Development Guidelines

This document provides guidelines for developing and maintaining the AI BotKit Engineering Plugin for Claude Code.

## Philosophy

This plugin provides specialized development workflows for the AI BotKit monorepo:
- **SaaS Application** (Next.js 16, Drizzle, Stripe, RAG/Pinecone)
- **WordPress Plugin** (PHP, WordPress APIs)

## Repository Structure

```
aibotkit-claude-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── agents/
│   ├── ai/                      # AI/RAG-specific agents
│   ├── database/                # Database review agents
│   ├── documentation/           # Documentation sync agents
│   ├── review/                  # General code review agents
│   ├── security/                # Security audit agents
│   └── wordpress/               # WordPress-specific agents
├── commands/
│   ├── deployment-workflow/     # Deployment commands
│   ├── development-workflow/    # Development lifecycle commands
│   ├── documentation-workflow/  # Documentation sync commands
│   ├── review-workflow/         # Code review commands
│   └── testing-workflow/        # Testing commands
├── skills/
│   ├── drizzle-patterns/        # Drizzle ORM patterns
│   ├── nextjs-saas/             # Next.js SaaS patterns
│   └── rag-development/         # RAG development patterns
├── specs/                       # Component specifications
├── templates/                   # Code templates
├── CLAUDE.md                    # This file
└── README.md
```

## Versioning Requirements

Every change MUST include updates to:

1. **`.claude-plugin/plugin.json`** - Bump version using semver
2. **`README.md`** - Update component counts and tables

### Version Bumping Rules

- **MAJOR** (1.0.0 → 2.0.0): Breaking changes, major reorganization
- **MINOR** (1.0.0 → 1.1.0): New agents, commands, or skills
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, doc updates, minor improvements

## Working with Components

### Adding a New Agent

1. Create agent file in appropriate category:
   ```
   agents/{category}/{agent-name}.md
   ```

2. Agent file structure:
   ```markdown
   # {Agent Name}

   {Brief description}

   ## Purpose
   {What this agent does}

   ## When to Use
   {Trigger conditions}

   ## What Gets Analyzed
   {Detailed analysis steps with good/anti-patterns}

   ## Output Format
   {Expected output structure}

   ## Integration
   {How it's used by commands}

   ## Related Agents
   {Links to related agents}
   ```

3. Update README.md with agent in appropriate category table
4. Update plugin.json version and description counts

### Adding a New Command

1. Create command file:
   ```
   commands/{workflow}/{command-name}.md
   ```

2. Command file structure:
   ```markdown
   # /command-name

   {Brief description}

   ## Context
   {When/why to use}

   ## Your Task
   {What to accomplish}

   ## Orchestration Strategy
   {Phases and agent coordination}

   ## Output Format
   {Expected report structure}

   ## Related Commands
   {Links to related commands}
   ```

3. Update README.md and plugin.json

### Adding a New Skill

1. Create skill file:
   ```
   skills/{category}/SKILL.md
   ```

2. Skill file structure:
   ```markdown
   ---
   name: skill-name
   description: Brief description
   ---

   # {Skill Title}

   {Detailed documentation with examples}
   ```

## AI BotKit Specific Knowledge

### SaaS Technology Stack

| Technology | Purpose | Location |
|------------|---------|----------|
| Next.js 16 | Framework | App Router in `src/app/` |
| Drizzle ORM | Database | Schema in `src/lib/db/schema.ts` |
| PostgreSQL | Database | Via Drizzle |
| Stripe | Payments | `src/lib/payments/` |
| Pinecone | Vector DB | `src/lib/ai/pinecone-client.ts` |
| Together AI | LLM | `src/lib/ai/together-client.ts` |
| shadcn/ui | UI Components | `src/components/` |

### WordPress Plugin Structure

| Component | Purpose | Location |
|-----------|---------|----------|
| Main File | Bootstrap | `ai-botkit-for-lead-generation.php` |
| Admin | Dashboard | `admin/` |
| Integration | SaaS API | `includes/integration/` |
| Public | Frontend | `includes/public/` |

### Key Patterns to Enforce

**SaaS:**
- Server Components by default, 'use client' only when needed
- Zod validation on all API inputs
- Drizzle for all database operations
- JWT auth via `src/lib/auth/session.ts`

**WordPress:**
- WPCS compliance
- Nonce verification on all forms/AJAX
- `current_user_can()` before sensitive operations
- `esc_*()` for all output

## Agent Invocation

Agents are invoked via the Task tool:
```
Task tool with subagent_type: "aibotkit-engineering:{category}:{agent-name}"
```

Example:
```
subagent_type: "aibotkit-engineering:review:nextjs-standards-reviewer"
```

## Command Invocation

Commands are invoked via slash commands:
```
/full-review
/next-phase
/update-docs
```

## Documentation Sync

The `/update-docs` command syncs GitBook documentation with code:

| Source | Target |
|--------|--------|
| `saas/src/lib/checkRateLimit.ts` | Plan limits in `documentation/plans-and-billing/` |
| `wordpress-plugin/readme.txt` | Features in `documentation/introduction/` |

The documentation is located at:
- **Monorepo:** `ai-botkit-mono-repo/documentation/`
- **GitBook URL:** `https://aibotkit.gitbook.io/documentation`
