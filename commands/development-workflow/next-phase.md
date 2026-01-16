# /next-phase

Continue development on AI BotKit following a **12-Phase Development Lifecycle**. This command discovers existing code, analyzes gaps, and guides implementation of new features while maintaining backward compatibility.

## When to Use This Command

| Scenario | Use This? | Why |
|----------|-----------|-----|
| **Adding new features to existing codebase** | **YES** | Discovers existing capabilities, plans integration |
| **Implementing Phase 2, 3, N features** | **YES** | Gap analysis calculates reuse |
| **Extending SaaS or Plugin** | **YES** | Maintains backward compatibility |
| **Starting from scratch (no code)** | NO | Create basic structure first |

## Description

The `/next-phase` command orchestrates development through 12 phases - 3 discovery phases followed by 9 implementation phases. It's designed for projects where initial development is complete, allowing you to continue development while building on existing functionality.

## The 12-Phase Development Lifecycle

```
+---------------------------------------------------------------------+
|              /next-phase DEVELOPMENT LIFECYCLE                       |
|                    12 Phases (3 Discovery + 9 Implementation)        |
+---------------------------------------------------------------------+
|                                                                      |
|  ================================================================== |
|  DISCOVERY PHASES (Understanding Existing Code)                      |
|  ================================================================== |
|                                                                      |
|  0.1. CODEBASE DISCOVERY   "What exists currently?"                 |
|          |                  -> _project_specs/code-index.md         |
|          v                  -> _project_specs/DISCOVERY_REPORT.md   |
|                                                                      |
|  0.2. DOCUMENTATION REVIEW "What docs exist/are missing?"           |
|          |                  -> _project_specs/DOC_GAPS.md           |
|          v                                                          |
|                                                                      |
|  0.3. GAP ANALYSIS         "What does next phase need?"             |
|          |                  -> reports/GAP_ANALYSIS.md              |
|          v                  -> reports/EFFORT_ESTIMATE.md           |
|                                                                      |
|  ================================================================== |
|  IMPLEMENTATION PHASES                                               |
|  ================================================================== |
|                                                                      |
|  1. REQUIREMENTS           "What exactly needs to be built?"        |
|          |                  -> specs/REQUIREMENTS.md                |
|          v                                                          |
|                                                                      |
|  2. ARCHITECTURE           "How does it extend current system?"     |
|          |                  -> docs/ARCHITECTURE.md (extended)      |
|          v                  -> specs/data-model.md (extended)       |
|                                                                      |
|  3. SPECIFICATION          "Detailed technical specs"               |
|          |                  -> specs/SPECIFICATION.md               |
|          v                  -> specs/contracts/ (API definitions)   |
|                                                                      |
|  4. IMPLEMENTATION         "Build the features"                     |
|          |                  -> src/ (extended, not replaced)        |
|          v                  -> Maintain backward compatibility      |
|                                                                      |
|  5. TESTING                "Write and run tests"                    |
|          |                  -> tests/ (unit, integration, e2e)      |
|          v                  -> reports/TEST_RESULTS.md              |
|                                                                      |
|  6. CODE REVIEW            "Quality and standards check"            |
|          |                  -> reports/REVIEW.md                    |
|          v                  -> Use /full-review command             |
|                                                                      |
|  7. DOCUMENTATION          "Update all docs"                        |
|          |                  -> README.md, CHANGELOG.md              |
|          v                  -> API docs, user guides                |
|                                                                      |
|  8. PRE-DEPLOYMENT         "Final checks before deploy"             |
|          |                  -> reports/DEPLOYMENT_CHECKLIST.md      |
|          v                                                          |
|                                                                      |
|  9. DEPLOYMENT             "Deploy to production"                   |
|                             -> Use /deploy-saas command             |
|                                                                      |
+---------------------------------------------------------------------+
```

## Usage

```bash
/next-phase [phase-number] [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--component saas` | Focus on SaaS component only |
| `--component wordpress` | Focus on WordPress plugin only |
| `--component all` | Both components (default) |
| `--skip-discovery` | Skip discovery phases (use cached) |
| `--brief <file>` | Path to requirements brief file |

### Examples

```bash
# Start from discovery
/next-phase

# Continue from specific phase
/next-phase 4

# Focus on SaaS only
/next-phase --component saas

# Provide requirements file
/next-phase --brief requirements/phase2-analytics.md
```

## Phase Details

### Phase 0.1: Codebase Discovery

**Purpose:** Index and understand existing codebase

**Actions:**
1. Scan SaaS directory structure
2. Identify key components and their purposes
3. Map dependencies and relationships
4. Catalog existing APIs and endpoints
5. List database tables and schemas

**Output Files:**
- `_project_specs/code-index.md` - Capability index
- `_project_specs/DISCOVERY_REPORT.md` - Detailed analysis

**Example Discovery Output:**
```markdown
# AI BotKit Codebase Discovery

## SaaS Component (saas/)

### Core Features
| Feature | Location | Status |
|---------|----------|--------|
| User Authentication | src/lib/auth/ | Complete |
| RAG Engine | src/lib/ai/rag-engine.ts | Complete |
| Chatbot CRUD | src/app/api/chatbots/ | Complete |
| Stripe Integration | src/lib/payments/ | Complete |
| Document Processing | src/lib/document-processing.ts | Complete |

### API Endpoints (28 total)
- /api/chat - Streaming chat
- /api/chatbots - Chatbot management
- /api/documents - Document upload
- [... more endpoints]

### Database Tables (12 total)
- users, teams, team_members
- aibotkit_chatbots, aibotkit_conversations, aibotkit_messages
- aibotkit_documents, aibotkit_sites
- [... more tables]

## WordPress Plugin Component

### Core Features
| Feature | Location | Status |
|---------|----------|--------|
| Plugin Bootstrap | ai-botkit-for-lead-generation.php | Complete |
| Admin Interface | admin/ | Complete |
| REST API Handler | includes/integration/ | Complete |
| Shortcode Handler | includes/public/ | Complete |
```

### Phase 0.2: Documentation Review

**Purpose:** Assess existing documentation and identify gaps

**Actions:**
1. Check for README, CHANGELOG
2. Review code comments and docstrings
3. Identify missing API documentation
4. Note undocumented features

**Output:**
- `_project_specs/DOC_GAPS.md` - Documentation gaps report

### Phase 0.3: Gap Analysis

**Purpose:** Compare current state with new requirements

**Actions:**
1. Map new requirements to existing capabilities
2. Classify each requirement (reusable, extend, new)
3. Identify breaking changes
4. Calculate effort adjustment

**Gap Classifications:**

| Classification | Match | Effort Factor | Meaning |
|---------------|-------|---------------|---------|
| **REUSABLE** | >90% | 0.1 | Use existing code as-is |
| **EXTEND** | 60-90% | 0.5 | Modify existing code |
| **PARTIAL** | 30-60% | 0.8 | Some code reusable |
| **NEW** | <30% | 1.0 | Build from scratch |

**Output:**
- `reports/GAP_ANALYSIS.md` - Detailed gap analysis
- `reports/EFFORT_ESTIMATE.md` - Time estimates with adjustments

### Phase 1: Requirements

**Purpose:** Document detailed requirements for new features

**Actions:**
1. Clarify ambiguous requirements
2. Break down into user stories
3. Define acceptance criteria
4. Identify dependencies

**Output:**
- `specs/REQUIREMENTS.md` - Detailed requirements document

### Phase 2: Architecture

**Purpose:** Design how new features integrate with existing system

**Actions:**
1. Extend existing architecture diagrams
2. Design new database tables/columns
3. Define new API endpoints
4. Plan component interactions

**Output:**
- `docs/ARCHITECTURE.md` - Updated architecture
- `specs/data-model.md` - Updated data model

### Phase 3: Specification

**Purpose:** Create detailed technical specifications

**Actions:**
1. Write API contracts
2. Define data structures
3. Specify business logic
4. Document error handling

**Output:**
- `specs/SPECIFICATION.md` - Technical specification
- `specs/contracts/` - API contract definitions

### Phase 4: Implementation

**Purpose:** Build the features

**Actions:**
1. Create/modify database schema
2. Implement API endpoints
3. Build UI components
4. Integrate with existing code

**Guidelines:**
- Follow existing code patterns
- Maintain backward compatibility
- Write code incrementally
- Commit frequently with clear messages

### Phase 5: Testing

**Purpose:** Ensure quality through comprehensive testing

**Actions:**
1. Write unit tests for new code
2. Write integration tests for APIs
3. Write E2E tests for user flows
4. Run existing tests (regression)

**Output:**
- `tests/` - Test files
- `reports/TEST_RESULTS.md` - Test execution results

### Phase 6: Code Review

**Purpose:** Quality and standards check

**Actions:**
1. Run `/full-review` command
2. Address critical and high issues
3. Document deferred issues

**Output:**
- `reports/REVIEW.md` - Review findings

### Phase 7: Documentation

**Purpose:** Update all documentation

**Actions:**
1. Update README with new features
2. Add CHANGELOG entry
3. Update API documentation
4. Create/update user guides

### Phase 8: Pre-Deployment

**Purpose:** Final checks before deployment

**Actions:**
1. Environment variable check
2. Database migration ready
3. Rollback plan documented
4. Monitoring configured

**Output:**
- `reports/DEPLOYMENT_CHECKLIST.md` - Pre-deployment checklist

### Phase 9: Deployment

**Purpose:** Deploy to production

**Actions:**
1. Run database migrations
2. Deploy application
3. Verify deployment
4. Monitor for issues

## AI BotKit Specific Considerations

### SaaS Component

When extending the SaaS:
- Maintain Drizzle schema conventions
- Follow existing API route patterns in `src/app/api/`
- Use existing auth middleware from `src/lib/auth/`
- Integrate with RAG engine patterns in `src/lib/ai/`
- Follow Stripe integration patterns for payments

### WordPress Plugin

When extending the plugin:
- Follow WordPress coding standards (WPCS)
- Use existing admin view patterns in `admin/views/`
- Integrate with REST API handler in `includes/integration/`
- Maintain PHP 7.4+ compatibility

### Database Changes

For database changes:
1. Update `src/lib/db/schema.ts` with new tables/columns
2. Run `pnpm db:generate` to create migration
3. Run `pnpm db:migrate` to apply
4. Update types as needed

### API Changes

For new API endpoints:
1. Create route in `src/app/api/{endpoint}/route.ts`
2. Add authentication if needed
3. Validate inputs with Zod
4. Document in API specs

## Example Workflow

```markdown
# Example: Adding Analytics Dashboard

## Phase 0.1: Discovery
Found existing:
- User authentication (reusable)
- Activity logs table (extend)
- Chart.js in dependencies (reusable)

## Phase 0.3: Gap Analysis
| Requirement | Classification | Effort Factor |
|-------------|---------------|---------------|
| Analytics API | EXTEND | 0.5 |
| Dashboard UI | NEW | 1.0 |
| Data aggregation | NEW | 1.0 |
| Export to PDF | NEW | 1.0 |

Effort adjustment: -15% (reuse of existing components)

## Phase 1: Requirements
- R1: Display message count by day/week/month
- R2: Show top chatbots by usage
- R3: Export analytics to PDF
- R4: Filter by date range

## Phase 2: Architecture
- New table: aibotkit_analytics_cache
- New API: /api/analytics
- New page: /dashboard/analytics

[Continue through phases...]
```

## Related Commands

- `/full-review` - Comprehensive code review
- `/deploy-saas` - Deployment workflow
- `/test-rag` - RAG engine testing

## Success Criteria

A successful /next-phase execution:
- Discovery phases completed with accurate analysis
- Gap analysis provides actionable insights
- Requirements clearly documented
- Architecture extends existing system cleanly
- Implementation maintains backward compatibility
- All tests pass including regression
- Documentation updated
- Deployment successful
