# /next-phase

Continue development on AI BotKit following a **21-Phase Development Lifecycle** with full **Spec-Driven Development (SDD)** methodology. This command discovers existing code, analyzes gaps, and guides implementation of new features while maintaining backward compatibility.

## When to Use This Command

| Scenario | Use This? | Why |
|----------|-----------|-----|
| **Adding new features to existing codebase** | **YES** | Discovers existing capabilities, plans integration |
| **Implementing Phase 2, 3, N features** | **YES** | Gap analysis calculates reuse |
| **Extending SaaS or Plugin** | **YES** | Maintains backward compatibility |
| **Starting from scratch (no code)** | NO | Create basic structure first |

**Quick Decision:**
- **Has source code?** → Use `/next-phase`
- **Empty project?** → Create initial structure first

## Description

The `/next-phase` command orchestrates multiple agents through 21 phases - 3 discovery phases followed by 18 SDD phases. It's designed for projects where initial development is complete, allowing you to continue development while maintaining backward compatibility and building on existing functionality.

**Key Features:**
- Discovers and indexes existing codebase before any new development
- Recovers missing documentation from code analysis
- Performs gap analysis between current state and new requirements
- Extends existing architecture rather than creating new
- Generates regression tests to protect existing functionality
- Maintains backward compatibility throughout

**All Quality Gates Preserved:**
- Phase 5.6: Requirement-Spec Validation (must pass before coding)
- Phase 5.7: Manual Test Cases (test-first approach)
- Phase 5.8: Dependency Collection (before coding)
- Phase 6: Full Implementation (not scaffolding)
- Phase 7: Complete Test Writing
- Phase 8: Test & Fix Loop (until 100% pass)
- Phase 9-10: Full Review and Fix

## The 21-Phase Development Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│              /next-phase DEVELOPMENT LIFECYCLE                       │
│                    21 Phases (3 Discovery + 18 SDD)                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ══════════════════════════════════════════════════════════════════ │
│  DISCOVERY PHASES (Understanding Existing Code)                      │
│  ══════════════════════════════════════════════════════════════════ │
│                                                                      │
│  0.1. CODEBASE DISCOVERY   "What exists currently?"                 │
│          ↓                  → _project_specs/code-index.md          │
│                             → _project_specs/DISCOVERY_REPORT.md    │
│                                                                      │
│  0.2. DOCUMENTATION RECOVERY "Fill in missing docs"                 │
│          ↓                  → specs/RECOVERED_ARCHITECTURE.md       │
│                             → specs/RECOVERED_SPECIFICATION.md      │
│                             → specs/RECOVERED_DATA_MODEL.md         │
│                                                                      │
│  0.3. GAP ANALYSIS         "What does next phase need?"             │
│          ↓                  → reports/GAP_ANALYSIS.md               │
│                             → reports/COMPATIBILITY_ASSESSMENT.md   │
│                                                                      │
│  ══════════════════════════════════════════════════════════════════ │
│  STANDARD SDD PHASES (Modified for Extension Mode)                   │
│  ══════════════════════════════════════════════════════════════════ │
│                                                                      │
│  0. PROJECT INIT           "Ensure governance exists"               │
│          ↓                  → CLAUDE.md (if missing)                │
│                             → CONSTITUTION.md (if missing)          │
│                                                                      │
│  0.5. CLARIFICATION        "What exactly needs to be built?"        │
│          ↓                  → Refined requirements                  │
│                                                                      │
│  1. ESTIMATION             "How long will it take?"                 │
│          ↓                  → specs/ESTIMATION.md                   │
│                             → Uses gap analysis reuse credits       │
│                                                                      │
│  2. ANALYSIS               "What needs to be built?"                │
│          ↓                  → Requirements analysis                 │
│                             → specs/research.md (updated)           │
│                                                                      │
│  3. DESIGN                 "How should the UI look?"                │
│          ↓                  → docs/UI_DESIGN.md                     │
│                                                                      │
│  4. ARCHITECTURE           "How does it extend current system?"     │
│          ↓                  → docs/ARCHITECTURE.md (extended)       │
│                             → specs/data-model.md (extended)        │
│                             → specs/contracts/ (API definitions)    │
│                                                                      │
│  5. SPECIFICATION          "Detailed technical specs"               │
│          ↓                  → specs/SPECIFICATION.md                │
│                                                                      │
│  5.5. CROSS-ANALYSIS       "Are all artifacts consistent?"          │
│          ↓                  → reports/CROSS_ARTIFACT_ANALYSIS.md    │
│                                                                      │
│  5.6. REQ-SPEC VALIDATION  "Do specs cover all requirements?"       │
│          ↓                  → reports/REQ_SPEC_VALIDATION.md        │
│                             → GATE: Must pass before coding         │
│                                                                      │
│  5.7. MANUAL TEST CASES    "What does success look like?"           │
│          ↓                  → tests/TEST_CASES.md                   │
│                             → tests/REGRESSION_TEST_CASES.md        │
│                                                                      │
│  5.8. DEPENDENCY COLLECTION "What dependencies are needed?"         │
│          ↓                  → depend/ (dependencies)                │
│                             → specs/DEPENDENCY_MANIFEST.md          │
│                             → GATE: Dependencies must be available  │
│                                                                      │
│  6. CODING                 "Build the features"                     │
│          ↓                  → src/ (extended, not replaced)         │
│                             → Maintain backward compatibility       │
│                                                                      │
│  6.5. SPEC VALIDATION      "Are specs implemented correctly?"       │
│          ↓                  → reports/SPEC_VALIDATION.md            │
│                                                                      │
│  7. TESTING                "Write complete tests"                   │
│          ↓                  → tests/e2e/ (new + regression)         │
│                             → tests/unit/ (new classes)             │
│                             → tests/integration/ (new APIs)         │
│                                                                      │
│  8. TEST & FIX LOOP        "Run tests, fix bugs until ALL pass"     │
│          ↓                  → reports/TEST_EXECUTION.md             │
│                             → reports/BUG_FIX_LOG.md                │
│                             → LOOP: Until 100% tests pass           │
│                                                                      │
│  9. CODE REVIEW            "Quality and standards check"            │
│          ↓                  → reports/REVIEW.md                     │
│                             → Use /full-review command              │
│                                                                      │
│  10. FIX & VALIDATE        "Fix code issues"                        │
│          ↓                  → reports/FIX_ITERATIONS.md             │
│                             → LOOP: Until quality threshold met     │
│                                                                      │
│  11. DOCUMENTATION         "Update all docs"                        │
│          ↓                  → README.md, CHANGELOG.md               │
│                             → API docs, user guides                 │
│                             → Use /update-docs command              │
│                                                                      │
│  12. DEPLOYMENT            "Prepare for release"                    │
│          ↓                  → .github/workflows/ (if needed)        │
│                             → reports/DEPLOYMENT_CHECKLIST.md       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
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
| `--no-questions` | Skip clarification phase |
| `--answers <file>` | Pre-provided answers JSON file |
| `--validate` | Run full validation after generation |
| `--fix` | Auto-fix issues found during validation |
| `--fail-on-test-error` | Exit with error if tests fail |

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

## Agent Orchestration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    /next-phase ORCHESTRATION                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DISCOVERY BLOCK [Sequential - Must complete before continuing]      │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 0.1: code-capability-indexer                                  │
│       ↓                                                              │
│  Phase 0.2: spec-recovery-agent                                      │
│       ↓                                                              │
│  Phase 0.3: gap-analyzer                                             │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  GOVERNANCE BLOCK [Conditional]                                      │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 0: project-initializer (if CLAUDE.md missing)                 │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  PLANNING BLOCK [Sequential]                                         │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 0.5: requirements-clarifier (uses gap analysis context)       │
│       ↓                                                              │
│  Phase 1: project-estimator (uses reuse credits from gap analysis)   │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  ANALYSIS BLOCK [Parallel where possible]                            │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 2: ┌─ requirements-analyzer                                   │
│           └─ dependency-analyzer                                     │
│       ↓                                                              │
│  Phase 3: ux-analyzer + accessibility-guardian                       │
│       ↓                                                              │
│  Phase 4: ┌─ architecture-reviewer (EXTEND mode)                     │
│           ├─ drizzle-schema-reviewer (for SaaS)                      │
│           └─ wordpress-standards-reviewer (for Plugin)               │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  SPECIFICATION BLOCK [Sequential with validation gates]              │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 5: specification-writer                                       │
│       ↓                                                              │
│  Phase 5.5: spec-consistency-checker                                 │
│       ↓                                                              │
│  Phase 5.6: requirements-spec-validator                              │
│       ↓ (GATE: must pass before continuing)                          │
│  Phase 5.7: test-case-generator (new + regression)                   │
│       ↓                                                              │
│  Phase 5.8: dependency-collector                                     │
│       ↓ (GATE: deps must be available)                               │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  IMPLEMENTATION BLOCK [Iterative]                                    │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 6: code-implementer (EXTEND mode) ←────────┐                  │
│       ↓                                           │                  │
│  Phase 6.5: spec-implementation-validator         │                  │
│       ↓                                           │                  │
│  Phase 7: ┌─ e2e-test-writer (new + regression)   │                  │
│           ├─ unit-test-writer                     │                  │
│           └─ integration-test-specialist          │                  │
│       ↓                                           │                  │
│  Phase 8: test-runner + bug-fixer ────────────────┘                  │
│       ↓ (LOOP until 100% pass)                                       │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  QUALITY BLOCK [Sequential + Iterative]                              │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 9: ┌─ nextjs-standards-reviewer (SaaS)                        │
│           ├─ wordpress-standards-reviewer (Plugin)                   │
│           └─ security-auditor                                        │
│       ↓                                                              │
│  Phase 10: code-fixer (LOOP until quality threshold)                 │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  FINALIZATION BLOCK [Sequential]                                     │
│  ───────────────────────────────────────────────────────────────    │
│  Phase 11: documentation-updater (/update-docs)                      │
│       ↓                                                              │
│  Phase 12: deployment-validator (/deploy-saas --dry-run)             │
│       ↓                                                              │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│                         NEXT-PHASE REPORT                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Explicit Execution Instructions

**CRITICAL: Claude MUST follow these step-by-step execution instructions during command execution.**

---

### Phase 0.1 Execution: Codebase Discovery

**Purpose:** Index and understand existing codebase before any new development.

```
EXECUTE Phase 0.1:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Identify Project Structure                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Glob tool to find all source files:                                     │
│   - saas/src/**/*.ts, saas/src/**/*.tsx                                     │
│   - wordpress-plugin/**/*.php                                               │
│   - Look for package.json, composer.json                                    │
│                                                                              │
│ IDENTIFY:                                                                    │
│   □ Main entry points                                                        │
│   □ Directory structure pattern                                              │
│   □ Key configuration files                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Index SaaS Component                                                 │
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
│ STEP 3: Index WordPress Plugin                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:wordpress:wordpress-hooks-analyzer"   │
│   prompt: "Index the WordPress plugin at wordpress-plugin/.                  │
│            Discover:                                                         │
│            - Main plugin file and bootstrap                                  │
│            - All hooks registered (actions, filters)                         │
│            - Admin pages and menus                                           │
│            - REST API endpoints                                              │
│            - Shortcodes                                                      │
│            Output: _project_specs/wordpress-index.md"                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 4: Generate Discovery Report                                            │
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

### Database Tables (12 total)
- users, teams, team_members
- aibotkit_chatbots, aibotkit_conversations, aibotkit_messages
- aibotkit_documents, aibotkit_sites

## WordPress Plugin Component

### Core Features
| Feature | Location | Status |
|---------|----------|--------|
| Plugin Bootstrap | ai-botkit-for-lead-generation.php | Complete |
| Admin Interface | admin/ | Complete |
| REST API Handler | includes/integration/ | Complete |
| Shortcode Handler | includes/public/ | Complete |
```

---

### Phase 0.3 Execution: Gap Analysis

**Purpose:** Compare current codebase state against new requirements.

```
EXECUTE Phase 0.3:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Read Inputs                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ READ _project_specs/DISCOVERY_REPORT.md (what exists)                        │
│ READ requirements brief file (--brief) or user requirements                  │
│ READ existing specs if available                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Invoke Gap Analyzer                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:gap-analyzer"           │
│   prompt: "Analyze gap between current capabilities and new requirements.    │
│            Current: _project_specs/DISCOVERY_REPORT.md                       │
│            New requirements: [from brief or user input]                      │
│                                                                              │
│            Classify each requirement as:                                     │
│            - REUSABLE (>90% match): Use existing code as-is                  │
│            - EXTEND (60-90%): Modify existing code                           │
│            - PARTIAL (30-60%): Some code reusable                            │
│            - NEW (<30%): Build from scratch                                  │
│                                                                              │
│            Identify:                                                         │
│            - Breaking changes risk                                           │
│            - Refactoring needs                                               │
│            - Effort adjustment factor                                        │
│                                                                              │
│            Output: reports/GAP_ANALYSIS.md"                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Verify Output                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ CHECK that reports/GAP_ANALYSIS.md contains:                                 │
│   □ Summary table of all requirements with classification                    │
│   □ Effort adjustment calculation                                            │
│   □ Breaking changes assessment                                              │
│   □ Recommendations for implementation order                                 │
│                                                                              │
│ IF incomplete: LOOP back to STEP 2 with specific corrections                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Gap Classification:**

| Classification | Match Score | Effort Factor | Meaning |
|---------------|-------------|---------------|---------|
| **REUSABLE** | >90% | 0.1 (90% savings) | Use existing code as-is |
| **EXTEND** | 60-90% | 0.5 (50% savings) | Modify existing code |
| **PARTIAL** | 30-60% | 0.8 (20% savings) | Some reuse, significant new work |
| **NEW** | <30% | 1.0 (no savings) | Build from scratch |

---

### Phase 5.6 Execution: Requirement-Spec Validation (QUALITY GATE)

**Purpose:** Ensure specifications cover all requirements before coding begins.

```
EXECUTE Phase 5.6:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Read Inputs                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ READ specs/SPECIFICATION.md (technical specifications)                       │
│ READ specs/REQUIREMENTS.md (functional requirements)                         │
│ READ reports/GAP_ANALYSIS.md (what needs to be built)                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Invoke Requirements-Spec Validator                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:requirements-spec-validator"
│   prompt: "Validate that all requirements are covered by specifications.     │
│            Requirements: specs/REQUIREMENTS.md                               │
│            Specifications: specs/SPECIFICATION.md                            │
│                                                                              │
│            Create traceability matrix:                                       │
│            - REQ-xxx → SPEC-xxx mapping                                      │
│            - Identify any REQ without SPEC coverage                          │
│            - Identify any SPEC without REQ traceability                      │
│                                                                              │
│            Output: reports/REQ_SPEC_VALIDATION.md"                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Check Gate                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ PARSE reports/REQ_SPEC_VALIDATION.md                                         │
│                                                                              │
│ IF coverage < 100%:                                                          │
│   ❌ GATE FAILED                                                             │
│   LIST uncovered requirements                                                │
│   RETURN to Phase 5 to add missing specifications                            │
│   DO NOT proceed to Phase 5.7                                                │
│                                                                              │
│ IF coverage == 100%:                                                         │
│   ✅ GATE PASSED                                                             │
│   PROCEED to Phase 5.7                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

**CRITICAL:** This is a QUALITY GATE. Do NOT proceed to coding (Phase 6) until this gate passes.

---

### Phase 5.7 Execution: Manual Test Cases (Test-First)

**Purpose:** Define what success looks like before implementation.

```
EXECUTE Phase 5.7:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Read Inputs                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ READ specs/SPECIFICATION.md (what to test)                                   │
│ READ reports/REQ_SPEC_VALIDATION.md (coverage matrix)                        │
│ READ reports/GAP_ANALYSIS.md (what's reusable vs new)                        │
│ READ docs/ARCHITECTURE.md (system structure)                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Generate New Feature Test Cases                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:test-case-generator"    │
│   prompt: "Generate comprehensive manual test cases for new features.        │
│            Input: specs/SPECIFICATION.md                                     │
│            Output: tests/TEST_CASES.md                                       │
│                                                                              │
│            Requirements:                                                     │
│            - Create TC-xxx for EVERY FR-xxx in specification                 │
│            - Include functional, security, performance, edge case tests      │
│            - Assign priorities: P0 (critical), P1 (high), P2 (medium)        │
│            - Create traceability matrix: FR-xxx → TC-xxx                     │
│            - Define clear expected results for each test case"               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Generate Regression Test Cases                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ USE Task tool with:                                                          │
│   subagent_type: "aibotkit-engineering:orchestration:test-case-generator"    │
│   prompt: "Generate regression test cases for existing functionality.        │
│            Input: _project_specs/DISCOVERY_REPORT.md                         │
│            Output: tests/REGRESSION_TEST_CASES.md                            │
│                                                                              │
│            Requirements:                                                     │
│            - Create regression TC for each existing feature                  │
│            - Focus on features that new code might affect                    │
│            - Include backward compatibility tests                            │
│            - Verify existing functionality still works                       │
│            - Mark all as P0 (critical) - regression must pass"               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 4: Verify Output                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ CHECK that tests/TEST_CASES.md exists and contains:                          │
│   □ At least one TC-xxx for each FR-xxx                                      │
│   □ Traceability matrix section                                              │
│   □ Test case categories (functional, security, performance, edge)           │
│   □ Priority assignments (P0, P1, P2)                                        │
│   □ Clear steps and expected results                                         │
│                                                                              │
│ CHECK that tests/REGRESSION_TEST_CASES.md exists and contains:               │
│   □ Tests for existing SaaS features                                         │
│   □ Tests for existing WordPress plugin features                             │
│   □ All marked as P0 priority                                                │
│                                                                              │
│ IF missing any: LOOP back to STEP 2/3 with specific corrections              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 7 Execution: Complete Test Writing

**Purpose:** Write executable tests for all test cases.

```
EXECUTE Phase 7:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Read Inputs                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ READ tests/TEST_CASES.md (new feature TC-xxx definitions)                    │
│ READ tests/REGRESSION_TEST_CASES.md (regression TCs)                         │
│ READ specs/SPECIFICATION.md (acceptance criteria)                            │
│ READ src/ directory structure (existing + new code)                          │
│ READ docs/ARCHITECTURE.md (class structure)                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Invoke Test Writers IN PARALLEL                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ AGENT 1: Unit Test Writer (SaaS)                                        │ │
│ │ USE Task tool with:                                                     │ │
│ │   subagent_type: "aibotkit-engineering:testing:unit-test-writer"        │ │
│ │   prompt: "Write COMPLETE Jest/Vitest unit tests for SaaS.              │ │
│ │            Input: saas/src/, tests/TEST_CASES.md                        │ │
│ │            Output: saas/tests/unit/                                     │ │
│ │            Requirements:                                                │ │
│ │            - Create test file for EVERY new class/function              │ │
│ │            - Map each test to TC-xxx from TEST_CASES.md                 │ │
│ │            - Include proper mocks for Drizzle, Stripe, Pinecone         │ │
│ │            - Tests MUST be executable (not stubs)"                      │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ AGENT 2: E2E Test Writer                                                │ │
│ │ USE Task tool with:                                                     │ │
│ │   subagent_type: "aibotkit-engineering:testing:e2e-test-generator"      │ │
│ │   prompt: "Write COMPLETE Playwright E2E tests.                         │ │
│ │            Input: tests/TEST_CASES.md, tests/REGRESSION_TEST_CASES.md   │ │
│ │            Output: tests/e2e/                                           │ │
│ │            Requirements:                                                │ │
│ │            - Create spec file for each feature area                     │ │
│ │            - Include page objects, helpers, fixtures                    │ │
│ │            - Test both SaaS app and WordPress plugin                    │ │
│ │            - Tests MUST be executable (not stubs)"                      │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ AGENT 3: Integration Test Writer                                        │ │
│ │ USE Task tool with:                                                     │ │
│ │   subagent_type: "aibotkit-engineering:testing:integration-test-specialist"│
│ │   prompt: "Write COMPLETE integration tests.                            │ │
│ │            Input: src/, specs/data-model.md, specs/contracts/           │ │
│ │            Output: tests/integration/                                   │ │
│ │            Requirements:                                                │ │
│ │            - Test all new database operations                           │ │
│ │            - Test all new API endpoints                                 │ │
│ │            - Test SaaS ↔ WordPress integration                          │ │
│ │            - Tests MUST be executable (not stubs)"                      │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Verify Output                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ CHECK test files contain:                                                    │
│   □ Actual test implementations (NOT empty/TODO)                             │
│   □ Proper imports and setup                                                 │
│   □ Assertions that verify behavior                                          │
│   □ Coverage for P0 and P1 test cases                                        │
│                                                                              │
│ IF any tests are stubs/empty: REJECT and request complete implementation     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 8 Execution: Test & Fix Loop (CRITICAL)

**Purpose:** Run all tests and fix bugs until 100% pass. This is an iterative loop.

```
EXECUTE Phase 8:
┌─────────────────────────────────────────────────────────────────────────────┐
│ INITIALIZATION                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ SET iteration = 1                                                            │
│ SET max_iterations = 10                                                      │
│ SET all_tests_passed = false                                                 │
│ CREATE reports/TEST_EXECUTION.md                                             │
│ CREATE reports/BUG_FIX_LOG.md                                                │
│                                                                              │
│ IMPORTANT: Run ALL tests (regression + new features)                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ LOOP START: While NOT all_tests_passed AND iteration <= max_iterations       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ STEP A: Run ALL Tests                                               │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ USE Task tool with:                                                 │   │
│   │   subagent_type: "aibotkit-engineering:testing:e2e-test-runner"     │   │
│   │   prompt: "Execute ALL test suites and report results.              │   │
│   │            Order:                                                   │   │
│   │            1. Unit tests (pnpm test:unit)                           │   │
│   │            2. Integration tests (pnpm test:integration)             │   │
│   │            3. E2E tests (pnpm test:e2e)                             │   │
│   │                                                                     │   │
│   │            Collect for each failure:                                │   │
│   │            - Test name                                              │   │
│   │            - Error message                                          │   │
│   │            - Stack trace with file:line                             │   │
│   │            - Expected vs actual                                     │   │
│   │                                                                     │   │
│   │            CATEGORIZE:                                              │   │
│   │            - Regression failures (existing features broken)         │   │
│   │            - New feature failures (new code not working)"           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                              ↓                                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ STEP B: Analyze Results                                             │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ PARSE test results                                                  │   │
│   │ COUNT: total_tests, passed, failed                                  │   │
│   │                                                                     │   │
│   │ IF failed == 0:                                                     │   │
│   │   SET all_tests_passed = true                                       │   │
│   │   BREAK LOOP → Go to SUCCESS                                        │   │
│   │                                                                     │   │
│   │ IF regression failures > 0:                                         │   │
│   │   PRIORITY: Fix regression failures FIRST (backward compat)         │   │
│   │                                                                     │   │
│   │ LIST all failing tests with details                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                              ↓                                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ STEP C: Fix Bugs                                                    │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ FOR EACH failing test (prioritize regression):                      │   │
│   │                                                                     │   │
│   │   USE Task tool with:                                               │   │
│   │     subagent_type: "aibotkit-engineering:testing:bug-fixer"         │   │
│   │     prompt: "Fix the bug causing test failure.                      │   │
│   │              Test: [test name]                                      │   │
│   │              Category: [Regression / New Feature]                   │   │
│   │              Error: [error message]                                 │   │
│   │              Location: [file:line from stack trace]                 │   │
│   │                                                                     │   │
│   │              RULES:                                                 │   │
│   │              1. Fix SOURCE CODE in src/, NOT the test               │   │
│   │              2. For regression: ensure existing behavior preserved  │   │
│   │              3. Analyze root cause before fixing                    │   │
│   │              4. Document fix in BUG_FIX_LOG.md"                     │   │
│   │                                                                     │   │
│   │ LOG iteration results to reports/BUG_FIX_LOG.md                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                              ↓                                               │
│   INCREMENT iteration                                                        │
│   LOOP BACK to STEP A (re-run ALL tests)                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ SUCCESS: All Tests Passed                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ UPDATE reports/TEST_EXECUTION.md with:                                       │
│   - Final test counts (all passing)                                          │
│   - Regression tests: X/X passing                                            │
│   - New feature tests: Y/Y passing                                           │
│   - Total iterations required                                                │
│   - Total bugs fixed                                                         │
│   - Coverage percentage                                                      │
│                                                                              │
│ OUTPUT: "✅ Phase 8 Complete:                                                │
│          - Regression: X tests passing                                       │
│          - New Features: Y tests passing                                     │
│          - Iterations: Z                                                     │
│          - Backward Compatibility: Preserved ✓"                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ FAILURE: Max Iterations Reached                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ IF iteration > max_iterations AND failed > 0:                                │
│                                                                              │
│   UPDATE reports/TEST_EXECUTION.md with:                                     │
│   - Tests still failing                                                      │
│   - Bugs that couldn't be fixed                                              │
│   - Recommendations                                                          │
│                                                                              │
│   IF regression failures > 0:                                                │
│     CRITICAL: "⚠️ BACKWARD COMPATIBILITY BROKEN"                             │
│                                                                              │
│   IF --fail-on-test-error:                                                   │
│     EXIT with error                                                          │
│   ELSE:                                                                      │
│     WARN: "⚠️ X tests still failing after Y iterations"                      │
│     CONTINUE to Phase 9 (review may identify remaining issues)               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 9-10 Execution: Code Review & Fix Loop

**Purpose:** Review code quality and fix issues until quality threshold met.

```
EXECUTE Phase 9-10:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Run Code Review (Use /full-review)                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ EXECUTE /full-review command                                                 │
│                                                                              │
│ This will invoke:                                                            │
│   - nextjs-standards-reviewer (for SaaS)                                     │
│   - wordpress-standards-reviewer (for Plugin)                                │
│   - security-auditor                                                         │
│   - accessibility-reviewer                                                   │
│   - drizzle-schema-reviewer                                                  │
│   - rag-engine-reviewer                                                      │
│                                                                              │
│ OUTPUT: reports/REVIEW.md                                                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Analyze Review Results                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ PARSE reports/REVIEW.md                                                      │
│                                                                              │
│ COUNT issues by severity:                                                    │
│   - CRITICAL: Must fix before proceeding                                     │
│   - HIGH: Should fix before deployment                                       │
│   - MEDIUM: Fix when possible                                                │
│   - LOW: Document for future                                                 │
│                                                                              │
│ IF CRITICAL or HIGH issues exist:                                            │
│   PROCEED to Phase 10 (Fix Loop)                                             │
│ ELSE:                                                                        │
│   SKIP to Phase 11                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Fix Loop (Phase 10)                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ SET fix_iteration = 1                                                        │
│ SET max_fix_iterations = 5                                                   │
│                                                                              │
│ WHILE CRITICAL or HIGH issues remain AND fix_iteration <= max_fix_iterations:│
│                                                                              │
│   USE Task tool with:                                                        │
│     subagent_type: "aibotkit-engineering:review:code-fixer"                  │
│     prompt: "Fix the following code issues:                                  │
│              [List of CRITICAL and HIGH issues from REVIEW.md]               │
│                                                                              │
│              For each issue:                                                 │
│              1. Read the affected file                                       │
│              2. Understand the issue                                         │
│              3. Apply the fix                                                │
│              4. Document the change                                          │
│                                                                              │
│              Output: reports/FIX_ITERATIONS.md (append iteration)"           │
│                                                                              │
│   RE-RUN /full-review to verify fixes                                        │
│   INCREMENT fix_iteration                                                    │
│                                                                              │
│ END WHILE                                                                    │
│                                                                              │
│ IF issues remain after max_fix_iterations:                                   │
│   WARN: "⚠️ Some issues could not be auto-fixed. Manual review required."    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Phase 11 Execution: Documentation Update

**Purpose:** Update all documentation to reflect new features.

```
EXECUTE Phase 11:
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: Run /update-docs                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ EXECUTE /update-docs --apply                                                 │
│                                                                              │
│ This will:                                                                   │
│   - Sync plan limits from code to documentation                              │
│   - Update feature lists                                                     │
│   - Validate all links                                                       │
│   - Verify image references                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Update README and CHANGELOG                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ UPDATE README.md:                                                            │
│   - Add new features section                                                 │
│   - Update usage examples                                                    │
│   - Update configuration options                                             │
│                                                                              │
│ UPDATE CHANGELOG.md:                                                         │
│   - Add new version entry                                                    │
│   - List all new features                                                    │
│   - List all bug fixes                                                       │
│   - List breaking changes (if any)                                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Update API Documentation                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ IF new API endpoints added:                                                  │
│   UPDATE docs/API.md with new endpoints                                      │
│   UPDATE specs/contracts/ with new API contracts                             │
│                                                                              │
│ IF database schema changed:                                                  │
│   UPDATE specs/data-model.md                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Output Structure

After running `/next-phase`, the project structure is updated:

```
ai-botkit-mono-repo/
├── _project_specs/                    # Discovery artifacts
│   ├── code-index.md                  # Phase 0.1: Capability index
│   ├── saas-index.md                  # Phase 0.1: SaaS component index
│   ├── wordpress-index.md             # Phase 0.1: WordPress component index
│   └── DISCOVERY_REPORT.md            # Phase 0.1: Discovery summary
├── specs/
│   ├── RECOVERED_ARCHITECTURE.md      # Phase 0.2: If recovered
│   ├── RECOVERED_SPECIFICATION.md     # Phase 0.2: If recovered
│   ├── REQUIREMENTS.md                # Phase 1: Requirements
│   ├── SPECIFICATION.md               # Phase 5: Specifications
│   ├── ESTIMATION.md                  # Phase 1: Effort estimates
│   ├── data-model.md                  # Phase 4: Extended data model
│   └── contracts/                     # Phase 4: API contracts
├── docs/
│   ├── ARCHITECTURE.md                # Phase 4: Extended architecture
│   ├── UI_DESIGN.md                   # Phase 3: UI design
│   └── API.md                         # Phase 11: API documentation
├── tests/
│   ├── TEST_CASES.md                  # Phase 5.7: New feature test cases
│   ├── REGRESSION_TEST_CASES.md       # Phase 5.7: Regression test cases
│   ├── e2e/                           # Phase 7: E2E tests
│   ├── unit/                          # Phase 7: Unit tests
│   └── integration/                   # Phase 7: Integration tests
├── reports/
│   ├── GAP_ANALYSIS.md                # Phase 0.3: Gap analysis
│   ├── COMPATIBILITY_ASSESSMENT.md    # Phase 0.3: Compatibility
│   ├── CROSS_ARTIFACT_ANALYSIS.md     # Phase 5.5
│   ├── REQ_SPEC_VALIDATION.md         # Phase 5.6
│   ├── SPEC_VALIDATION.md             # Phase 6.5
│   ├── TEST_EXECUTION.md              # Phase 8
│   ├── BUG_FIX_LOG.md                 # Phase 8
│   ├── REVIEW.md                      # Phase 9
│   ├── FIX_ITERATIONS.md              # Phase 10
│   └── DEPLOYMENT_CHECKLIST.md        # Phase 12
├── saas/src/                          # Extended with new features
├── wordpress-plugin/                  # Extended with new features
├── documentation/                     # Updated by /update-docs
├── README.md                          # Updated with new features
└── CHANGELOG.md                       # New version entry
```

## Final Report Structure

```markdown
# Next-Phase Completion Report

## Phase Summary
| Phase | Status | Duration |
|-------|--------|----------|
| 0.1 Codebase Discovery | ✅ Complete | 2 min |
| 0.2 Documentation Recovery | ✅ Complete | 1 min |
| 0.3 Gap Analysis | ✅ Complete | 1 min |
| 5.6 Req-Spec Validation | ✅ GATE PASSED | - |
| 5.8 Dependency Collection | ✅ GATE PASSED | - |
| 8 Test & Fix Loop | ✅ Complete (3 iterations) | 15 min |
| 9-10 Review & Fix | ✅ Complete (2 iterations) | 10 min |

## Discovery Summary
- **Files Indexed:** 127
- **SaaS Components:** 45
- **WordPress Components:** 32
- **API Endpoints:** 28
- **Database Tables:** 12

## Gap Analysis Summary
- **Reusable:** 3 features (25%)
- **Extend:** 4 features (33%)
- **New:** 5 features (42%)
- **Effort Adjustment:** -17% (from reuse)

## Implementation Summary
- **Specs Written:** 12 functional requirements
- **Test Cases Created:** 28 (18 new + 10 regression)
- **Code Implemented:** 15 new files, 8 extended files
- **Tests Written:** 45 E2E + 62 unit + 12 integration

## Test Results
| Suite | Total | Passed | Failed |
|-------|-------|--------|--------|
| Regression | 45 | 45 | 0 |
| New Features | 74 | 74 | 0 |
| **Total** | **119** | **119** | **0** |

## Quality Score
| Category | Score |
|----------|-------|
| Standards Compliance | 94/100 |
| Security | 98/100 |
| Performance | 91/100 |
| Accessibility | 89/100 |

## Backward Compatibility
- ✅ All existing functionality preserved
- ✅ All regression tests passing
- ✅ No breaking API changes

## Documentation Updated
- ✅ README.md - Added new features
- ✅ CHANGELOG.md - Added version entry
- ✅ GitBook documentation - Synced via /update-docs
```

## Agents Used

| Agent | Phase | Purpose |
|-------|-------|---------|
| `code-capability-indexer` | 0.1 | Index existing codebase |
| `wordpress-hooks-analyzer` | 0.1 | Index WordPress hooks |
| `spec-recovery-agent` | 0.2 | Recover missing documentation |
| `gap-analyzer` | 0.3 | Analyze gaps |
| `requirements-clarifier` | 0.5 | Clarify requirements |
| `project-estimator` | 1 | Estimate effort |
| `requirements-analyzer` | 2 | Analyze requirements |
| `ux-analyzer` | 3 | UI design |
| `architecture-reviewer` | 4 | Extend architecture |
| `drizzle-schema-reviewer` | 4 | Database schema |
| `specification-writer` | 5 | Write specifications |
| `spec-consistency-checker` | 5.5 | Cross-artifact validation |
| `requirements-spec-validator` | 5.6 | Requirement coverage |
| `test-case-generator` | 5.7 | Generate test cases |
| `dependency-collector` | 5.8 | Collect dependencies |
| `code-implementer` | 6 | Implement features |
| `spec-implementation-validator` | 6.5 | Verify implementation |
| `unit-test-writer` | 7 | Write unit tests |
| `e2e-test-generator` | 7 | Write E2E tests |
| `integration-test-specialist` | 7 | Write integration tests |
| `e2e-test-runner` | 8 | Run tests |
| `bug-fixer` | 8 | Fix bugs |
| `nextjs-standards-reviewer` | 9 | SaaS code review |
| `wordpress-standards-reviewer` | 9 | WordPress code review |
| `security-auditor` | 9 | Security review |
| `code-fixer` | 10 | Auto-fix issues |
| `documentation-updater` | 11 | Update documentation |
| `deployment-validator` | 12 | Deployment checks |

## Related Commands

- `/full-review` - Comprehensive code review (used in Phase 9)
- `/deploy-saas` - Deployment workflow (used in Phase 12)
- `/test-rag` - RAG engine testing
- `/sync-db` - Database migrations
- `/update-docs` - Documentation sync (used in Phase 11)

## Success Criteria

A successful `/next-phase` execution:
- ✅ Discovery phases completed with accurate analysis
- ✅ Gap analysis provides actionable insights
- ✅ Phase 5.6 gate passed (100% requirement coverage)
- ✅ Phase 5.8 gate passed (all dependencies available)
- ✅ Phase 8 loop completed (100% tests passing)
- ✅ Phase 9-10 loop completed (quality threshold met)
- ✅ All regression tests passing (backward compatibility)
- ✅ Documentation updated
- ✅ Final report generated
