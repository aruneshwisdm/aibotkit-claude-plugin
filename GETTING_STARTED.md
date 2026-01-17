# Getting Started with AI BotKit Engineering Plugin

This guide will help you set up and start using the AI BotKit Engineering Plugin with Claude Code.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/claude-code) installed and configured
- Access to the AI BotKit monorepo (or similar Next.js + WordPress project)
- Git installed on your system

## Installation

### Option 1: Install from Local Path

```bash
# Clone the plugin repository
git clone https://github.com/aruneshwisdm/aibotkit-claude-plugin.git

# Add the plugin to Claude Code
claude plugins add /path/to/aibotkit-claude-plugin
```

### Option 2: Install from GitHub (when supported)

```bash
claude plugins add github:aruneshwisdm/aibotkit-claude-plugin
```

### Verify Installation

```bash
# List installed plugins
claude plugins list

# You should see:
# - aibotkit-engineering (v1.6.0)
```

## Quick Start

### 1. Navigate to Your Project

```bash
cd /path/to/ai-botkit-mono-repo
```

### 2. Run Your First Command

Start with a full code review:

```bash
claude
> /full-review
```

This will orchestrate 10 specialized agents to review your entire codebase.

### 3. Start a Development Workflow

Begin a new feature using the phased development lifecycle:

```bash
claude
> /next-phase
```

This guides you through 21 phases (3 discovery + 18 SDD) with quality gates and test loops.

## Core Commands

| Command | What It Does | When to Use |
|---------|--------------|-------------|
| `/full-review` | Multi-agent code review | Before PRs, after major changes |
| `/next-phase` | Development lifecycle | Starting new features |
| `/fit-quality` | Generate specs, docs, tests | Documenting existing codebases |
| `/deploy-saas` | Deployment checklist | Before production deployments |
| `/test-rag` | RAG engine testing | After RAG changes |
| `/sync-db` | Database migrations | After schema changes |
| `/update-docs` | Sync GitBook documentation | After code changes affect docs |

## Your First Review

### Step 1: Run Full Review

```bash
claude
> /full-review
```

### Step 2: Review the Output

The plugin will analyze:
- Next.js patterns and Server/Client component usage
- RAG engine implementation
- Database schema design
- Security vulnerabilities
- WordPress coding standards
- Accessibility compliance
- Architecture principles

### Step 3: Address Issues

Each issue includes:
- **Severity**: CRITICAL, HIGH, MEDIUM, LOW
- **Location**: File path and line number
- **Problem**: What's wrong
- **Recommendation**: How to fix it

Example output:
```
### HIGH: Missing Error Boundary
**File:** src/app/(dashboard)/chatbots/page.tsx
**Issue:** Page lacks error boundary for graceful error handling

**Recommended Fix:**
Create error.tsx in the same directory...
```

## Your First Feature Development

### Step 1: Start Discovery

```bash
claude
> /next-phase
```

### Step 2: Follow the Phases

**Phase 1-3: Discovery**
- Codebase indexing
- Documentation review
- Gap analysis

**Phase 4-6: Planning**
- Requirements documentation
- Architecture design
- Technical specifications

**Phase 7-12: Implementation**
- Build features
- Write tests
- Code review
- Documentation
- Deployment

### Step 3: Continue from Any Phase

```bash
# Resume from phase 4 (Requirements)
> /next-phase 4

# Provide requirements file
> /next-phase --brief requirements.md
```

## Project Structure

The plugin expects this project structure:

```
your-project/
├── saas/                    # Next.js SaaS application
│   ├── src/
│   │   ├── app/             # App Router pages
│   │   ├── components/      # React components
│   │   ├── lib/
│   │   │   ├── ai/          # RAG engine
│   │   │   ├── db/          # Drizzle schema
│   │   │   └── payments/    # Stripe
│   │   └── hooks/           # Custom hooks
│   └── package.json
│
└── wordpress-plugin/        # WordPress plugin
    ├── admin/               # Admin interface
    ├── includes/            # Core PHP
    └── public/              # Frontend
```

## Configuration

### Environment Variables

The plugin works best when these are set in your project:

```bash
# .env.local (SaaS)
POSTGRES_URL=postgresql://...
PINECONE_API_KEY=...
PINECONE_INDEX=...
STRIPE_SECRET_KEY=...
TOGETHER_API_KEY=...
```

### Customizing Reviews

Target specific components:

```bash
# Review only SaaS
> /full-review --component saas

# Review only WordPress plugin
> /full-review --component wordpress

# Review specific files
> /full-review --files src/lib/ai/rag-engine.ts
```

## Common Workflows

### Starting a New Feature

```bash
# 1. Start discovery phase
> /next-phase

# 2. After discovery, continue to requirements
> /next-phase 4

# 3. Implement the feature
# ... write code ...

# 4. Run review before PR
> /full-review

# 5. Deploy
> /deploy-saas
```

### Fixing RAG Issues

```bash
# 1. Test RAG engine
> /test-rag

# 2. Review RAG implementation
> /full-review --component saas --focus rag

# 3. After fixes, test again
> /test-rag
```

### Database Schema Changes

```bash
# 1. Make schema changes in src/lib/db/schema.ts

# 2. Generate and apply migration
> /sync-db

# 3. Verify schema
> /sync-db --studio
```

### Documenting Existing Codebase

```bash
# 1. Run fit-quality to generate all artifacts
> /fit-quality

# 2. Review generated specifications
cat specs/RECOVERED_SPECIFICATION.md

# 3. Review generated documentation
cat docs/USER_GUIDE.md
cat docs/DEVELOPER.md

# 4. Review generated test cases
cat tests/manual/MANUAL_TEST_CASES.md
```

### Updating Documentation

```bash
# 1. After changing plan limits, features, or pricing
> /update-docs

# 2. Review proposed changes
# 3. Apply with confirmation
> /update-docs --apply

# 4. Validate documentation links
> /update-docs --validate
```

## Tips for Best Results

### 1. Keep Context Focused

```bash
# Good: Work in the relevant directory
cd saas && claude

# Better: Specify what you're working on
> I'm working on the chatbot creation flow
```

### 2. Use Phase Checkpoints

Save progress between phases:

```bash
> /next-phase 4
# Complete phase 4 work
> /next-phase 5
# Continue to next phase
```

### 3. Review Before Commits

```bash
# Always run review before creating PRs
> /full-review

# Fix issues, then commit
> git add . && git commit -m "feat: add chatbot templates"
```

### 4. Use Skills for Learning

Access domain knowledge:

```bash
> How should I implement vector search?
# Plugin will use rag-development skill

> What's the best pagination pattern for Drizzle?
# Plugin will use drizzle-patterns skill
```

## Troubleshooting

### Plugin Not Found

```bash
# Verify plugin is installed
claude plugins list

# Reinstall if needed
claude plugins remove aibotkit-engineering
claude plugins add /path/to/aibotkit-claude-plugin
```

### Commands Not Working

```bash
# Check you're in the right directory
pwd

# Verify project structure exists
ls -la saas/ wordpress-plugin/
```

### Review Taking Too Long

```bash
# Target specific components
> /full-review --component saas

# Or specific files
> /full-review --files src/lib/ai/
```

## Next Steps

1. **Read Command Documentation**: See [docs/COMMANDS.md](docs/COMMANDS.md)
2. **Explore Agents**: See [docs/AGENTS.md](docs/AGENTS.md)
3. **Learn Patterns**: See [docs/SKILLS.md](docs/SKILLS.md)
4. **Example Workflows**: See [docs/WORKFLOWS.md](docs/WORKFLOWS.md)

## Getting Help

- **Plugin Issues**: [GitHub Issues](https://github.com/aruneshwisdm/aibotkit-claude-plugin/issues)
- **Claude Code Help**: Run `/help` in Claude Code
- **AI BotKit Documentation**: [https://docs.aibotkit.io](https://docs.aibotkit.io)

---

Happy coding with AI BotKit!
