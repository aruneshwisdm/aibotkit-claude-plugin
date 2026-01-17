# /fit-quality

Generate specifications, documentation, and tests for an existing AI BotKit codebase. This command fits quality artifacts to projects that lack proper documentation and test coverage.

## When to Use This Command

| Scenario | Use This? | Why |
|----------|-----------|-----|
| **Existing project with no specs** | **YES** | Generates specifications from code |
| **Existing project with no tests** | **YES** | Creates unit, integration, and E2E tests |
| **Existing project with poor docs** | **YES** | Generates comprehensive documentation |
| **Legacy code needing quality artifacts** | **YES** | Fits all quality artifacts |
| **Starting a new project** | NO | Create initial structure first |
| **Adding new features to existing project** | NO | Use `/next-phase` instead |
| **Just need code review** | NO | Use `/full-review` instead |

**Quick Decision:**
- **Has code but needs specs/docs/tests?** → Use `/fit-quality`
- **Adding features?** → Use `/next-phase`
- **Just need review?** → Use `/full-review`

## Description

The `/fit-quality` command analyzes an existing AI BotKit codebase and generates comprehensive quality artifacts including specifications, documentation, and tests. It supports both the **Next.js SaaS application** and the **WordPress plugin** with platform-specific generators.

**Key Differentiator:**
- Does NOT modify existing source code
- ONLY generates quality artifacts (specs, docs, tests)
- Works with any existing codebase regardless of quality
- Platform-aware generation for accurate artifacts

## Platform Support

| Platform | Detection | Spec Generator | Test Framework |
|----------|-----------|----------------|----------------|
| **SaaS (Next.js)** | `saas/package.json`, `src/app/` | Next.js/React patterns | Jest/Vitest + Playwright |
| **WordPress Plugin** | Plugin header file, `includes/` | WordPress patterns | PHPUnit + Playwright |

## Usage

```bash
/fit-quality [path] [options]
```

### Arguments

| Argument | Description | Default |
|----------|-------------|---------|
| `path` | Path to existing project | Current directory |

### Options

#### Platform Options

| Option | Description |
|--------|-------------|
| `--component <type>` | Force component: `saas`, `wordpress`, `all` (auto-detect if omitted) |

#### Output Selection

| Option | Description |
|--------|-------------|
| `--specs` | Generate specifications only |
| `--docs` | Generate documentation only |
| `--tests` | Generate tests only (unit + integration) |
| `--e2e` | Generate E2E tests only |
| `--all` | Generate everything (default) |

#### Specification Options

| Option | Description |
|--------|-------------|
| `--include-data-model` | Generate data-model.md from Drizzle schema |
| `--include-api-contracts` | Generate API contracts from Next.js routes |
| `--include-rag-specs` | Generate RAG engine specifications |

#### Documentation Options

| Option | Description |
|--------|-------------|
| `--include-readme` | Generate/update README.md |
| `--include-developer` | Generate DEVELOPER.md |
| `--include-api-docs` | Generate API documentation |
| `--include-changelog` | Generate CHANGELOG.md scaffold |

#### Test Options

| Option | Description |
|--------|-------------|
| `--coverage-target <percent>` | Target test coverage (default: 80) |
| `--include-unit` | Generate unit tests |
| `--include-integration` | Generate integration tests |
| `--include-e2e` | Generate E2E tests |

#### Output Options

| Option | Description |
|--------|-------------|
| `--output <dir>` | Output directory (default: project root) |
| `--dry-run` | Preview what would be generated |
| `--overwrite` | Overwrite existing files |

## The 8-Phase Fit Quality Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│              /fit-quality LIFECYCLE                                  │
│                    8 Phases                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  0. COMPONENT DETECTION  "What kind of project is this?"            │
│          ↓                → SaaS (Next.js) / WordPress Plugin       │
│                                                                      │
│  1. CODEBASE DISCOVERY   "What exists in the code?"                 │
│          ↓                → _project_specs/code-index.md            │
│                           → _project_specs/DISCOVERY_REPORT.md      │
│                                                                      │
│  2. SPECIFICATION GEN    "Document what the code does"              │
│          ↓                → specs/RECOVERED_SPECIFICATION.md        │
│                           → specs/data-model.md                     │
│                           → specs/contracts/                        │
│                                                                      │
│  3. ARCHITECTURE DOC     "How is the code structured?"              │
│          ↓                → docs/ARCHITECTURE.md                    │
│                           → docs/CLASS_DIAGRAM.md                   │
│                                                                      │
│  4. USER DOCUMENTATION   "How do users use this?"                   │
│          ↓                → README.md                               │
│                           → docs/USER_GUIDE.md                      │
│                           → CHANGELOG.md                            │
│                                                                      │
│  5. DEVELOPER DOCS       "How do developers work with this?"        │
│          ↓                → docs/DEVELOPER.md                       │
│                           → docs/API.md                             │
│                           → CONTRIBUTING.md                         │
│                                                                      │
│  6. TEST CASE GENERATION "What should we test?"                     │
│          ↓                → tests/MANUAL_TEST_CASES.md              │
│                           → tests/TEST_PLAN.md                      │
│                                                                      │
│  7. TEST IMPLEMENTATION  "Write executable tests"                   │
│          ↓                → tests/unit/                             │
│                           → tests/integration/                      │
│                           → tests/e2e/                              │
│                                                                      │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│                      FIT QUALITY REPORT                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Phase Details

### Phase 0: Component Detection

**Purpose:** Identify which AI BotKit components exist for accurate artifact generation.

**Detection Criteria:**

| Component | Markers |
|-----------|---------|
| **SaaS (Next.js)** | `saas/package.json`, `saas/src/app/`, `saas/src/lib/ai/rag-engine.ts` |
| **WordPress Plugin** | `wordpress-plugin/ai-botkit-for-lead-generation.php`, `wordpress-plugin/includes/` |

### Phase 1: Codebase Discovery

**Agent:** `code-capability-indexer`

**Purpose:** Index all existing code to understand what needs documentation and testing.

**Discovers for SaaS:**
- React components (Server/Client)
- API routes (`src/app/api/`)
- Database schema (Drizzle tables, relations)
- RAG engine components
- Authentication system
- Stripe payment integration
- Third-party integrations (Pinecone, Together AI, OpenAI)

**Discovers for WordPress:**
- Plugin hooks (actions, filters)
- Admin pages and menus
- REST API endpoints
- AJAX handlers
- Shortcodes
- SaaS integration points

**Output:**
```
_project_specs/
├── code-index.md           # Searchable capability index
├── DISCOVERY_REPORT.md     # Human-readable summary
└── discovery-data.json     # Machine-readable data
```

### Phase 2: Specification Generation

**Agents:** `spec-recovery-agent`, `data-modeler`, `api-contract-generator`

**Purpose:** Generate specifications from code analysis.

**SaaS Output:**
```
specs/
├── RECOVERED_SPECIFICATION.md    # Functional requirements
├── data-model.md                 # Drizzle schema documentation
├── rag-engine-spec.md            # RAG engine specifications
└── contracts/
    ├── chat-api.contract.md      # Chat API spec
    ├── chatbots-api.contract.md  # Chatbot CRUD API spec
    ├── documents-api.contract.md # Document API spec
    ├── stripe-api.contract.md    # Payment API spec
    └── wordpress-api.contract.md # WordPress integration API spec
```

**WordPress Output:**
```
specs/
├── RECOVERED_SPECIFICATION.md    # Functional requirements
├── data-model.md                 # Options, settings, stored data
└── contracts/
    ├── rest-api.contract.md      # REST API specs
    ├── ajax.contract.md          # AJAX endpoint specs
    └── saas-integration.contract.md # SaaS API integration
```

### Phase 3: Architecture Documentation

**Agent:** `architecture-reviewer`

**Purpose:** Document code structure and architecture.

**Output:**
```
docs/
├── ARCHITECTURE.md          # System architecture
├── CLASS_DIAGRAM.md         # Class relationships (Mermaid)
├── SEQUENCE_DIAGRAMS.md     # Key workflows (Mermaid)
│   ├── Chat flow
│   ├── RAG retrieval flow
│   ├── Document processing flow
│   ├── Authentication flow
│   └── WordPress sync flow
└── DEPENDENCY_GRAPH.md      # Module dependencies
```

### Phase 4: User Documentation

**Agent:** `documentation-generator`

**Purpose:** Generate user-facing documentation.

**Output:**
```
README.md                    # Project overview, installation, usage
CHANGELOG.md                 # Version history scaffold
docs/
├── USER_GUIDE.md           # End-user documentation
├── INSTALLATION.md         # Installation instructions
├── CONFIGURATION.md        # Configuration options
├── WORDPRESS_SETUP.md      # WordPress plugin setup
└── FAQ.md                  # Common questions
```

### Phase 5: Developer Documentation

**Agent:** `api-docs-generator`, `documentation-generator`

**Purpose:** Generate developer-facing documentation.

**Output:**
```
CONTRIBUTING.md              # Contribution guidelines
docs/
├── DEVELOPER.md            # Developer guide
├── API.md                  # API reference (all endpoints)
├── RAG_ENGINE.md           # RAG engine internals
├── DATABASE.md             # Drizzle schema guide
├── AUTHENTICATION.md       # Auth system documentation
├── WORDPRESS_HOOKS.md      # WordPress hooks reference
└── TESTING.md              # Testing instructions
```

### Phase 6: Test Case Generation

**Agents:** `test-case-generator`, `manual-test-generator`

**Purpose:** Define what needs to be tested before writing tests.

**Output:**
```
tests/
├── MANUAL_TEST_CASES.md    # Manual test scenarios
├── TEST_PLAN.md            # Overall test strategy
└── COVERAGE_MATRIX.md      # Code → test mapping
```

**Test Case Categories:**
- Functional tests (chatbot CRUD, chat, documents)
- RAG engine tests (retrieval, context, streaming)
- Security tests (auth, validation, rate limiting)
- Payment tests (Stripe webhooks, subscriptions)
- Integration tests (WordPress sync, Pinecone)
- Accessibility tests (WCAG compliance)
- Performance tests (response times, load)

### Phase 7: Test Implementation

**Agents:** Component-specific test writers

**Purpose:** Generate executable test code.

#### SaaS Tests

**Agents:** `unit-test-writer`, `integration-test-specialist`, `e2e-test-generator`

```
saas/tests/
├── unit/
│   ├── setup.ts
│   ├── lib/
│   │   ├── ai/
│   │   │   ├── rag-engine.test.ts
│   │   │   ├── pinecone-client.test.ts
│   │   │   └── together-client.test.ts
│   │   ├── db/
│   │   │   └── queries.test.ts
│   │   ├── auth/
│   │   │   └── session.test.ts
│   │   └── payments/
│   │       └── stripe.test.ts
│   └── components/
│       └── ChatbotPreview.test.tsx
├── integration/
│   ├── api/
│   │   ├── chat.test.ts
│   │   ├── chatbots.test.ts
│   │   ├── documents.test.ts
│   │   └── stripe-webhook.test.ts
│   └── db/
│       └── schema.test.ts
├── e2e/
│   ├── playwright.config.ts
│   └── specs/
│       ├── auth.spec.ts
│       ├── dashboard.spec.ts
│       ├── chatbot-crud.spec.ts
│       ├── chat.spec.ts
│       └── billing.spec.ts
├── vitest.config.ts
└── package.json
```

#### WordPress Tests

**Agents:** `unit-test-writer`, `e2e-test-generator`

```
wordpress-plugin/tests/
├── unit/
│   ├── bootstrap.php
│   ├── TestCase.php
│   ├── Admin/
│   │   └── AdminTest.php
│   ├── Integration/
│   │   ├── ApiHandlerTest.php
│   │   └── RestApiTest.php
│   └── Public/
│       └── ShortcodeHandlerTest.php
├── integration/
│   ├── SaasConnectionTest.php
│   └── PostSyncTest.php
├── e2e/
│   ├── playwright.config.js
│   └── specs/
│       ├── activation.spec.js
│       ├── admin-dashboard.spec.js
│       ├── saas-connection.spec.js
│       └── chatbot-embed.spec.js
├── phpunit.xml
└── package.json
```

## Agent Orchestration

```
┌─────────────────────────────────────────────────────────────────────┐
│                /fit-quality ORCHESTRATION                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DISCOVERY BLOCK [Sequential]                                        │
│  ─────────────────────────────────────────────────────────────────  │
│  Phase 0: Component Detection (built-in)                             │
│       ↓                                                              │
│  Phase 1: code-capability-indexer                                    │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  SPECIFICATION BLOCK [Parallel]                                      │
│  ─────────────────────────────────────────────────────────────────  │
│  Phase 2: ┌─ spec-recovery-agent                                     │
│           ├─ data-modeler (Drizzle schema)                           │
│           └─ api-contract-generator (Next.js routes)                 │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  DOCUMENTATION BLOCK [Parallel]                                      │
│  ─────────────────────────────────────────────────────────────────  │
│  Phase 3: architecture-reviewer                                      │
│       ↓                                                              │
│  Phase 4-5: ┌─ documentation-generator (user docs)                   │
│             ├─ documentation-generator (developer docs)              │
│             └─ api-docs-generator                                    │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  TESTING BLOCK [Sequential then Parallel]                            │
│  ─────────────────────────────────────────────────────────────────  │
│  Phase 6: test-case-generator + manual-test-generator                │
│       ↓                                                              │
│  Phase 7: [Component-specific, Parallel]                             │
│           ┌─ SaaS: unit-test-writer + integration-test-specialist    │
│           │        + e2e-test-generator                              │
│           └─ WordPress: unit-test-writer + e2e-test-generator        │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│                      FIT QUALITY REPORT                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Explicit Execution Instructions

**CRITICAL: Claude MUST follow these step-by-step execution instructions during command execution.**

---

### Phase 0 Execution: Component Detection

```
EXECUTE Phase 0:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Detect SaaS Component                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ CHECK for:                                                                   │
│   - saas/package.json exists                                                │
│   - saas/src/app/ directory exists                                          │
│   - saas/src/lib/ai/rag-engine.ts exists                                    │
│                                                                              │
│ IF all exist: SaaS component detected                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Detect WordPress Component                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ CHECK for:                                                                   │
│   - wordpress-plugin/ai-botkit-for-lead-generation.php exists               │
│   - wordpress-plugin/includes/ directory exists                             │
│                                                                              │
│ IF all exist: WordPress component detected                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Set Component Scope                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ Based on detection and --component flag:                                     │
│   - If --component saas: Only SaaS                                           │
│   - If --component wordpress: Only WordPress                                 │
│   - If --component all or not specified: Both detected components            │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 1 Execution: Codebase Discovery

```
EXECUTE Phase 1:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Index SaaS Component (if detected)                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:architecture:code-capability-indexer" │
│   prompt: "Index the SaaS codebase at saas/.                                │
│            Discover:                                                         │
│            - All API routes in src/app/api/                                  │
│            - Database schema in src/lib/db/schema.ts                         │
│            - React components in src/components/                             │
│            - RAG engine in src/lib/ai/                                       │
│            - Auth system in src/lib/auth/                                    │
│            - Payment integration in src/lib/payments/                        │
│            Output: _project_specs/saas-index.md"                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Index WordPress Component (if detected)                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:wordpress:wordpress-hooks-analyzer"   │
│   prompt: "Index the WordPress plugin at wordpress-plugin/.                  │
│            Discover:                                                         │
│            - Main plugin file and bootstrap                                  │
│            - All hooks registered (actions, filters)                         │
│            - Admin pages and menus                                           │
│            - REST API endpoints                                              │
│            - AJAX handlers                                                   │
│            - Shortcodes                                                      │
│            Output: _project_specs/wordpress-index.md"                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Generate Discovery Report                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ CREATE _project_specs/DISCOVERY_REPORT.md with:                              │
│   - Project overview (files, LOC, components)                                │
│   - SaaS capabilities summary                                                │
│   - WordPress plugin capabilities summary                                    │
│   - Database tables list                                                     │
│   - API endpoints list                                                       │
│   - Extension points (hooks, filters)                                        │
│   - External dependencies                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 2 Execution: Specification Generation

```
EXECUTE Phase 2:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Invoke Spec Recovery Agent (PARALLEL with Steps 2-3)                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:spec-recovery-agent"    │
│   prompt: "Generate functional specifications from code analysis.            │
│            Input: _project_specs/DISCOVERY_REPORT.md                         │
│            Analyze: saas/src/, wordpress-plugin/                             │
│                                                                              │
│            Generate specs/RECOVERED_SPECIFICATION.md with:                   │
│            - FR-xxx for each discovered feature                              │
│            - NFR-xxx for non-functional requirements                         │
│            - Acceptance criteria for each requirement                        │
│            - Traceability to source files"                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓ (parallel)
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Invoke Data Modeler (if --include-data-model)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:data-modeler"           │
│   prompt: "Generate data model documentation from Drizzle schema.            │
│            Input: saas/src/lib/db/schema.ts                                  │
│                                                                              │
│            Generate specs/data-model.md with:                                │
│            - All tables with columns and types                               │
│            - Foreign key relationships                                       │
│            - Indexes                                                         │
│            - ER diagram in Mermaid format                                    │
│            - WordPress options schema if applicable"                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓ (parallel)
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Invoke API Contract Generator (if --include-api-contracts)           │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:api-contract-generator" │
│   prompt: "Generate API contracts from Next.js route handlers.               │
│            Input: saas/src/app/api/                                          │
│                                                                              │
│            Generate specs/contracts/:                                        │
│            - chat-api.contract.md (streaming chat endpoint)                  │
│            - chatbots-api.contract.md (CRUD operations)                      │
│            - documents-api.contract.md (document management)                 │
│            - stripe-api.contract.md (payment webhooks)                       │
│            - wordpress-api.contract.md (WordPress integration)               │
│                                                                              │
│            Include for each:                                                 │
│            - HTTP method and path                                            │
│            - Request schema (Zod validation)                                 │
│            - Response schema                                                 │
│            - Error responses                                                 │
│            - Authentication requirements"                                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 3-5 Execution: Documentation Generation

```
EXECUTE Phases 3-5:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Generate Architecture Documentation                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:review:architecture-reviewer"         │
│   prompt: "Generate architecture documentation for AI BotKit.                │
│            Input: _project_specs/DISCOVERY_REPORT.md                         │
│                                                                              │
│            Generate:                                                         │
│            - docs/ARCHITECTURE.md (system overview)                          │
│            - docs/CLASS_DIAGRAM.md (Mermaid diagrams)                        │
│            - docs/SEQUENCE_DIAGRAMS.md (key flows)                           │
│                                                                              │
│            Include diagrams for:                                             │
│            - Chat request flow (user → API → RAG → LLM → response)           │
│            - Document processing flow                                        │
│            - Authentication flow                                             │
│            - WordPress sync flow"                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Generate User & Developer Documentation (PARALLEL)                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:documentation-generator"│
│   prompt: "Generate comprehensive documentation for AI BotKit.               │
│            Input: _project_specs/, specs/                                    │
│                                                                              │
│            Generate:                                                         │
│            - README.md (project overview)                                    │
│            - CHANGELOG.md (scaffold)                                         │
│            - CONTRIBUTING.md (contribution guidelines)                       │
│            - docs/USER_GUIDE.md (end-user documentation)                     │
│            - docs/DEVELOPER.md (developer guide)                             │
│            - docs/INSTALLATION.md (setup instructions)                       │
│            - docs/CONFIGURATION.md (environment variables, options)"         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓ (parallel)
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Generate API Documentation                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:api-docs-generator"     │
│   prompt: "Generate API reference documentation.                             │
│            Input: specs/contracts/, saas/src/app/api/                        │
│                                                                              │
│            Generate:                                                         │
│            - docs/API.md (complete API reference)                            │
│            - docs/RAG_ENGINE.md (RAG internals)                              │
│            - docs/DATABASE.md (Drizzle schema guide)                         │
│            - docs/AUTHENTICATION.md (JWT, session management)                │
│            - docs/WORDPRESS_HOOKS.md (actions, filters)"                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 6-7 Execution: Test Generation

```
EXECUTE Phases 6-7:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Generate Test Cases                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:test-case-generator"    │
│   prompt: "Generate comprehensive test cases for AI BotKit.                  │
│            Input: specs/RECOVERED_SPECIFICATION.md                           │
│                                                                              │
│            Generate:                                                         │
│            - tests/TEST_PLAN.md (overall strategy)                           │
│            - tests/COVERAGE_MATRIX.md (code → test mapping)                  │
│                                                                              │
│            Create TC-xxx for each FR-xxx covering:                           │
│            - Functional tests (features work correctly)                      │
│            - Security tests (auth, validation, injection)                    │
│            - Performance tests (response times, load)                        │
│            - Edge cases (errors, boundaries)"                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓ (parallel)
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Generate Manual Test Cases                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:manual-test-generator"  │
│   prompt: "Generate manual test scenarios for AI BotKit.                     │
│            Input: specs/, docs/USER_GUIDE.md                                 │
│                                                                              │
│            Generate tests/MANUAL_TEST_CASES.md with:                         │
│            - User journey tests (signup → chatbot → chat)                    │
│            - Admin workflow tests                                            │
│            - WordPress integration tests                                     │
│            - Accessibility tests (WCAG 2.1 AA)                               │
│            - Cross-browser tests"                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Generate Executable Tests (PARALLEL by component)                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ FOR SaaS Component:                                                          │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:testing:unit-test-writer"           │
│     prompt: "Write Vitest unit tests for SaaS.                               │
│              Input: saas/src/, tests/TEST_CASES.md                           │
│              Output: saas/tests/unit/                                        │
│              Cover: RAG engine, auth, payments, queries"                     │
│                                                                              │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:testing:integration-test-specialist"│
│     prompt: "Write integration tests for SaaS API.                           │
│              Input: saas/src/app/api/, specs/contracts/                      │
│              Output: saas/tests/integration/                                 │
│              Cover: All API endpoints, DB operations"                        │
│                                                                              │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:testing:e2e-test-generator"         │
│     prompt: "Write Playwright E2E tests for SaaS dashboard.                  │
│              Input: saas/src/app/, tests/MANUAL_TEST_CASES.md                │
│              Output: saas/tests/e2e/                                         │
│              Cover: Auth, dashboard, chatbot CRUD, chat, billing"            │
│                                                                              │
│ FOR WordPress Component:                                                     │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:testing:unit-test-writer"           │
│     prompt: "Write PHPUnit tests for WordPress plugin.                       │
│              Input: wordpress-plugin/, tests/TEST_CASES.md                   │
│              Output: wordpress-plugin/tests/unit/                            │
│              Cover: Admin, REST API, AJAX, shortcodes"                       │
│                                                                              │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:testing:e2e-test-generator"         │
│     prompt: "Write Playwright E2E tests for WordPress admin.                 │
│              Input: wordpress-plugin/admin/, tests/MANUAL_TEST_CASES.md      │
│              Output: wordpress-plugin/tests/e2e/                             │
│              Cover: Activation, admin dashboard, SaaS connection, embed"     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Output Structure

### AI BotKit Monorepo After /fit-quality

```
ai-botkit-mono-repo/
├── _project_specs/
│   ├── code-index.md
│   ├── saas-index.md
│   ├── wordpress-index.md
│   ├── DISCOVERY_REPORT.md
│   └── discovery-data.json
├── specs/
│   ├── RECOVERED_SPECIFICATION.md
│   ├── data-model.md
│   ├── rag-engine-spec.md
│   └── contracts/
│       ├── chat-api.contract.md
│       ├── chatbots-api.contract.md
│       ├── documents-api.contract.md
│       ├── stripe-api.contract.md
│       └── wordpress-api.contract.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── CLASS_DIAGRAM.md
│   ├── SEQUENCE_DIAGRAMS.md
│   ├── USER_GUIDE.md
│   ├── DEVELOPER.md
│   ├── INSTALLATION.md
│   ├── CONFIGURATION.md
│   ├── API.md
│   ├── RAG_ENGINE.md
│   ├── DATABASE.md
│   ├── AUTHENTICATION.md
│   ├── WORDPRESS_HOOKS.md
│   └── TESTING.md
├── tests/
│   ├── TEST_PLAN.md
│   ├── MANUAL_TEST_CASES.md
│   └── COVERAGE_MATRIX.md
├── saas/
│   └── tests/
│       ├── unit/
│       ├── integration/
│       ├── e2e/
│       └── vitest.config.ts
├── wordpress-plugin/
│   └── tests/
│       ├── unit/
│       ├── e2e/
│       └── phpunit.xml
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
└── FIT_QUALITY_REPORT.md
```

## Fit Quality Report

After completion, a `FIT_QUALITY_REPORT.md` is generated:

```markdown
# Fit Quality Report

## Summary

| Metric | Value |
|--------|-------|
| Components | SaaS (Next.js), WordPress Plugin |
| Execution Time | 4m 30s |
| Files Generated | 65 |

## Discovery Summary

### SaaS Component
| Category | Count |
|----------|-------|
| API Routes | 28 |
| React Components | 45 |
| Database Tables | 12 |
| RAG Engine Files | 5 |

### WordPress Component
| Category | Count |
|----------|-------|
| PHP Classes | 8 |
| Hooks (Actions/Filters) | 35 |
| REST Endpoints | 4 |
| AJAX Handlers | 20 |

## Generated Artifacts

### Specifications
- [x] RECOVERED_SPECIFICATION.md (52 FRs)
- [x] data-model.md (12 tables)
- [x] contracts/ (5 API contracts)

### Documentation
- [x] ARCHITECTURE.md
- [x] CLASS_DIAGRAM.md
- [x] SEQUENCE_DIAGRAMS.md
- [x] USER_GUIDE.md
- [x] DEVELOPER.md
- [x] API.md

### Tests
- [x] TEST_PLAN.md
- [x] MANUAL_TEST_CASES.md (85 cases)
- [x] SaaS unit tests (45 files)
- [x] SaaS integration tests (12 files)
- [x] SaaS E2E tests (8 specs)
- [x] WordPress unit tests (8 files)
- [x] WordPress E2E tests (4 specs)

## Test Coverage Estimate

| Component | Type | Files | Coverage |
|-----------|------|-------|----------|
| SaaS | Unit | 45 | ~80% |
| SaaS | Integration | 12 | ~70% |
| SaaS | E2E | 8 | ~60% |
| WordPress | Unit | 8 | ~75% |
| WordPress | E2E | 4 | ~50% |

## Next Steps

1. Review generated specifications for accuracy
2. Run SaaS tests: `cd saas && pnpm test`
3. Run WordPress tests: `cd wordpress-plugin && ./vendor/bin/phpunit`
4. Run E2E tests: `pnpm test:e2e`
5. Update README.md with project-specific details
6. Review and customize test cases as needed
```

## Agents Used

| Agent | Phase | Purpose |
|-------|-------|---------|
| `code-capability-indexer` | 1 | Index codebase |
| `wordpress-hooks-analyzer` | 1 | Index WordPress hooks |
| `spec-recovery-agent` | 2 | Generate specifications |
| `data-modeler` | 2 | Generate data model |
| `api-contract-generator` | 2 | Generate API contracts |
| `architecture-reviewer` | 3 | Document architecture |
| `documentation-generator` | 4, 5 | Generate docs |
| `api-docs-generator` | 5 | Generate API docs |
| `test-case-generator` | 6 | Generate test cases |
| `manual-test-generator` | 6 | Generate manual tests |
| `unit-test-writer` | 7 | Write unit tests |
| `integration-test-specialist` | 7 | Write integration tests |
| `e2e-test-generator` | 7 | Write E2E tests |

## Related Commands

| Command | Use Case |
|---------|----------|
| `/next-phase` | Continue development with new features |
| `/full-review` | Review existing code quality |
| `/update-docs` | Sync documentation with GitBook |
| `/deploy-saas` | Deployment workflow |

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| "No source code found" | Empty directory | Verify path contains code |
| "Component detection failed" | Missing markers | Use `--component` flag |
| "Cannot write to directory" | Permission issue | Check write permissions |
| "Existing files found" | Files already exist | Use `--overwrite` flag |

## Performance

| Project Size | Estimated Time |
|--------------|----------------|
| Small (<50 files) | 1-2 minutes |
| Medium (50-200 files) | 2-4 minutes |
| Large (200-500 files) | 4-8 minutes |
| Very Large (>500 files) | 8-15 minutes |

## Best Practices

1. **Run discovery first** - Use `--dry-run` to preview what will be generated
2. **Review specifications** - Generated specs may need human refinement
3. **Customize tests** - Add edge cases specific to your RAG implementation
4. **Update README** - Add project-specific installation instructions
5. **Run tests** - Verify generated tests pass before committing
6. **Iterate** - Use `/full-review` after retrofit to check quality
