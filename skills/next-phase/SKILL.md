---
name: next-phase
description: 21-phase development lifecycle orchestrator with discovery, SDD methodology, and quality gates
user_invocable: true
---

# /next-phase - Development Lifecycle Orchestrator

This skill orchestrates the 21-phase development lifecycle for AI BotKit, automatically chaining agents, managing state, and enforcing quality gates.

## Invocation

```bash
/next-phase                    # Start from Phase 0.1 or resume
/next-phase 5.6                # Jump to specific phase
/next-phase --brief "Add PDF export for conversations"
/next-phase --status           # Show current progress
/next-phase --reset            # Reset and start fresh
```

## Orchestrator Execution Instructions

When `/next-phase` is invoked, follow this orchestration protocol:

### Step 1: Initialize State

Check for existing state file at `_project_specs/.next-phase-state.json`:

```json
{
  "currentPhase": "0.1",
  "completedPhases": [],
  "brief": "",
  "startedAt": "ISO-DATE",
  "lastUpdated": "ISO-DATE",
  "gateResults": {},
  "artifacts": []
}
```

If no state exists, create it. If state exists, resume from `currentPhase`.

### Step 2: Phase Router

Execute the appropriate phase based on current state:

```
PHASE ROUTING TABLE
═══════════════════════════════════════════════════════════════════════════════

DISCOVERY PHASES
───────────────────────────────────────────────────────────────────────────────
Phase 0.1 │ CODEBASE_DISCOVERY
          │ Agent: code-capability-indexer
          │ Output: _project_specs/DISCOVERY_REPORT.md
          │ Action: Index all existing code, APIs, database, services
          │ Next: 0.2
───────────────────────────────────────────────────────────────────────────────
Phase 0.2 │ DOCUMENTATION_RECOVERY
          │ Agent: (inline analysis)
          │ Output: specs/RECOVERED_ARCHITECTURE.md, specs/RECOVERED_DATA_MODEL.md
          │ Action: Generate missing documentation from code analysis
          │ Next: 0.3
───────────────────────────────────────────────────────────────────────────────
Phase 0.3 │ GAP_ANALYSIS
          │ Agent: gap-analyzer
          │ Input: Discovery report + brief requirements
          │ Output: reports/GAP_ANALYSIS.md
          │ Action: Compare current vs requirements, classify reuse potential
          │ Next: 0
═══════════════════════════════════════════════════════════════════════════════

SDD PHASES
───────────────────────────────────────────────────────────────────────────────
Phase 0   │ PROJECT_INIT
          │ Action: Ensure CLAUDE.md and governance files exist
          │ Output: CLAUDE.md (if missing)
          │ Next: 0.5
───────────────────────────────────────────────────────────────────────────────
Phase 0.5 │ CLARIFICATION
          │ Action: Use AskUserQuestion to clarify ambiguous requirements
          │ Output: specs/REQUIREMENTS_CLARIFIED.md
          │ Next: 1
───────────────────────────────────────────────────────────────────────────────
Phase 1   │ ESTIMATION
          │ Agent: (inline with gap analysis data)
          │ Output: specs/ESTIMATION.md
          │ Action: Calculate effort with reuse credits from gap analysis
          │ Next: 2
───────────────────────────────────────────────────────────────────────────────
Phase 2   │ ANALYSIS
          │ Action: Deep requirements analysis
          │ Output: specs/REQUIREMENTS.md
          │ Next: 3
───────────────────────────────────────────────────────────────────────────────
Phase 3   │ DESIGN
          │ Action: UI/UX design specifications
          │ Output: docs/UI_DESIGN.md
          │ Next: 4
───────────────────────────────────────────────────────────────────────────────
Phase 4   │ ARCHITECTURE
          │ Agent: architecture-reviewer (extend mode)
          │ Output: docs/ARCHITECTURE.md (extended)
          │ Action: Design system extensions, not replacements
          │ Next: 5
───────────────────────────────────────────────────────────────────────────────
Phase 5   │ SPECIFICATION
          │ Output: specs/SPECIFICATION.md
          │ Action: Detailed technical specifications with SPEC-xxx IDs
          │ Next: 5.5
───────────────────────────────────────────────────────────────────────────────
Phase 5.5 │ CROSS_ANALYSIS
          │ Action: Verify consistency across all artifacts
          │ Output: reports/CROSS_ARTIFACT_ANALYSIS.md
          │ Next: 5.6
───────────────────────────────────────────────────────────────────────────────
Phase 5.6 │ REQ_SPEC_VALIDATION ⛔ QUALITY GATE
          │ Agent: requirements-spec-validator
          │ Output: reports/REQ_SPEC_VALIDATION.md
          │ Gate: Must achieve 100% coverage to proceed
          │ On Fail: Return to Phase 5 to add missing specs
          │ Next: 5.7 (if passed)
───────────────────────────────────────────────────────────────────────────────
Phase 5.7 │ TEST_CASE_GENERATION
          │ Agent: test-case-generator
          │ Output: tests/TEST_CASES.md, tests/REGRESSION_TEST_CASES.md
          │ Action: Generate manual test cases from specs
          │ Next: 5.8
───────────────────────────────────────────────────────────────────────────────
Phase 5.8 │ DEPENDENCY_COLLECTION ⛔ QUALITY GATE
          │ Action: Collect and verify all dependencies
          │ Output: specs/DEPENDENCY_MANIFEST.md
          │ Gate: All dependencies must be available/installable
          │ On Fail: Resolve dependency issues before proceeding
          │ Next: 6 (if passed)
───────────────────────────────────────────────────────────────────────────────
Phase 6   │ CODING
          │ Action: Implement features (EXTEND mode, not replace)
          │ Output: src/ (extended code)
          │ Principle: Maintain backward compatibility
          │ Next: 6.5
───────────────────────────────────────────────────────────────────────────────
Phase 6.5 │ SPEC_VALIDATION
          │ Action: Verify implementation matches specifications
          │ Output: reports/SPEC_VALIDATION.md
          │ Next: 7
───────────────────────────────────────────────────────────────────────────────
Phase 7   │ TESTING
          │ Agents: unit-test-writer, e2e-test-generator, integration-test-specialist
          │ Output: tests/unit/, tests/e2e/, tests/integration/
          │ Action: Write comprehensive tests (new + regression)
          │ Next: 8
───────────────────────────────────────────────────────────────────────────────
Phase 8   │ TEST_FIX_LOOP ⛔ QUALITY GATE (LOOP)
          │ Agents: e2e-test-runner, bug-fixer
          │ Output: reports/TEST_EXECUTION.md, reports/BUG_FIX_LOG.md
          │ Gate: 100% tests must pass
          │ Loop: Run tests → Fix failures → Repeat until all pass
          │ Next: 9 (when 100% pass)
───────────────────────────────────────────────────────────────────────────────
Phase 9   │ CODE_REVIEW
          │ Command: /full-review
          │ Output: reports/REVIEW.md
          │ Action: Run comprehensive code review
          │ Next: 10
───────────────────────────────────────────────────────────────────────────────
Phase 10  │ FIX_VALIDATE_LOOP ⛔ QUALITY GATE (LOOP)
          │ Agent: code-fixer
          │ Output: reports/FIX_ITERATIONS.md
          │ Gate: Quality threshold must be met (no Critical/High issues)
          │ Loop: Fix issues → Re-review → Repeat until threshold met
          │ Next: 11 (when threshold met)
───────────────────────────────────────────────────────────────────────────────
Phase 11  │ DOCUMENTATION
          │ Command: /update-docs
          │ Output: Updated README.md, CHANGELOG.md, API docs
          │ Next: 12
───────────────────────────────────────────────────────────────────────────────
Phase 12  │ DEPLOYMENT
          │ Output: reports/DEPLOYMENT_CHECKLIST.md
          │ Action: Generate deployment checklist and validation
          │ Next: COMPLETE
═══════════════════════════════════════════════════════════════════════════════
```

### Step 3: Execute Current Phase

For each phase, follow this execution pattern:

```markdown
## Executing Phase {PHASE_NUMBER}: {PHASE_NAME}

### Pre-Execution
1. Log phase start to state file
2. Create output directory if needed
3. Load required inputs from previous phases

### Execution
1. Invoke appropriate agent(s) via Task tool
2. Generate required artifacts
3. Write output files to specified locations

### Post-Execution
1. Update state file with completion
2. Add artifacts to state
3. Determine next phase

### Quality Gate Handling (Phases 5.6, 5.8, 8, 10)
IF gate fails:
  - Log failure reason
  - Set state to remediation phase
  - Inform user of required actions
  - DO NOT proceed to next phase

IF gate passes:
  - Log success
  - Proceed to next phase
```

### Step 4: State Management

After each phase, update the state file:

```json
{
  "currentPhase": "5.7",
  "completedPhases": ["0.1", "0.2", "0.3", "0", "0.5", "1", "2", "3", "4", "5", "5.5", "5.6"],
  "brief": "Add PDF export for conversations",
  "startedAt": "2024-12-15T10:00:00Z",
  "lastUpdated": "2024-12-15T14:30:00Z",
  "gateResults": {
    "5.6": { "passed": true, "coverage": "100%", "timestamp": "..." }
  },
  "artifacts": [
    "_project_specs/DISCOVERY_REPORT.md",
    "reports/GAP_ANALYSIS.md",
    "specs/REQUIREMENTS.md",
    "specs/SPECIFICATION.md"
  ]
}
```

### Step 5: User Communication

At each phase transition, output a status update:

```markdown
═══════════════════════════════════════════════════════════════════════════════
✅ Phase 5.6 COMPLETE: Req-Spec Validation

Gate Result: PASSED (100% coverage)
Artifacts: reports/REQ_SPEC_VALIDATION.md

═══════════════════════════════════════════════════════════════════════════════
▶ Starting Phase 5.7: Test Case Generation

Agent: test-case-generator
Output: tests/TEST_CASES.md, tests/REGRESSION_TEST_CASES.md
═══════════════════════════════════════════════════════════════════════════════
```

## Directory Structure

The orchestrator creates and manages these directories:

```
project-root/
├── _project_specs/              # Discovery and state
│   ├── .next-phase-state.json   # Orchestrator state
│   ├── DISCOVERY_REPORT.md      # Phase 0.1 output
│   └── code-index.md            # Code capability index
├── specs/                       # Specifications
│   ├── REQUIREMENTS.md          # Phase 2 output
│   ├── SPECIFICATION.md         # Phase 5 output
│   ├── ESTIMATION.md            # Phase 1 output
│   └── DEPENDENCY_MANIFEST.md   # Phase 5.8 output
├── docs/                        # Documentation
│   ├── ARCHITECTURE.md          # Phase 4 output
│   └── UI_DESIGN.md             # Phase 3 output
├── reports/                     # Reports and validations
│   ├── GAP_ANALYSIS.md          # Phase 0.3 output
│   ├── REQ_SPEC_VALIDATION.md   # Phase 5.6 output
│   ├── TEST_EXECUTION.md        # Phase 8 output
│   ├── REVIEW.md                # Phase 9 output
│   └── DEPLOYMENT_CHECKLIST.md  # Phase 12 output
└── tests/                       # Test artifacts
    ├── TEST_CASES.md            # Phase 5.7 output
    ├── REGRESSION_TEST_CASES.md # Phase 5.7 output
    ├── unit/                    # Phase 7 output
    ├── e2e/                     # Phase 7 output
    └── integration/             # Phase 7 output
```

## Agent Mapping

| Phase | Agent | Invocation |
|-------|-------|------------|
| 0.1 | code-capability-indexer | `subagent_type: "aibotkit-engineering:architecture:code-capability-indexer"` |
| 0.3 | gap-analyzer | `subagent_type: "aibotkit-engineering:orchestration:gap-analyzer"` |
| 4 | architecture-reviewer | `subagent_type: "aibotkit-engineering:review:architecture-reviewer"` |
| 5.6 | requirements-spec-validator | `subagent_type: "aibotkit-engineering:orchestration:requirements-spec-validator"` |
| 5.7 | test-case-generator | `subagent_type: "aibotkit-engineering:orchestration:test-case-generator"` |
| 7 | unit-test-writer | `subagent_type: "aibotkit-engineering:testing:unit-test-writer"` |
| 7 | e2e-test-generator | `subagent_type: "aibotkit-engineering:testing:e2e-test-generator"` |
| 7 | integration-test-specialist | `subagent_type: "aibotkit-engineering:testing:integration-test-specialist"` |
| 8 | e2e-test-runner | `subagent_type: "aibotkit-engineering:testing:e2e-test-runner"` |
| 8 | bug-fixer | `subagent_type: "aibotkit-engineering:testing:bug-fixer"` |
| 9 | (invoke /full-review) | Command invocation |
| 10 | code-fixer | `subagent_type: "aibotkit-engineering:review:code-fixer"` |
| 11 | (invoke /update-docs) | Command invocation |

## Quality Gates

### Gate 5.6: Requirement-Specification Validation

```
PASS Criteria: 100% of requirements (FR-xxx) have specification coverage (SPEC-xxx)
FAIL Action: Return to Phase 5, add missing specifications
```

### Gate 5.8: Dependency Collection

```
PASS Criteria: All dependencies available and installable
FAIL Action: Resolve dependency issues (find alternatives, request access)
```

### Gate 8: Test Execution Loop

```
PASS Criteria: 100% of tests pass
FAIL Action:
  1. Invoke bug-fixer for each failure
  2. Prioritize regression failures
  3. Re-run tests
  4. Repeat until 100% pass or max iterations (5)
```

### Gate 10: Code Review Loop

```
PASS Criteria: No Critical or High severity issues
FAIL Action:
  1. Invoke code-fixer for each issue
  2. Re-run /full-review
  3. Repeat until threshold met or max iterations (3)
```

## Example Execution

### Starting Fresh

```
User: /next-phase --brief "Add PDF export for conversations"

Orchestrator:
═══════════════════════════════════════════════════════════════════════════════
🚀 Starting /next-phase Development Lifecycle

Brief: Add PDF export for conversations
State: NEW (no previous state found)

═══════════════════════════════════════════════════════════════════════════════
▶ Phase 0.1: Codebase Discovery

Invoking: code-capability-indexer agent
...
[Agent discovers 64 endpoints, 15 tables, etc.]
...

✅ Phase 0.1 COMPLETE
Artifact: _project_specs/DISCOVERY_REPORT.md

═══════════════════════════════════════════════════════════════════════════════
▶ Phase 0.2: Documentation Recovery
...
```

### Resuming

```
User: /next-phase

Orchestrator:
═══════════════════════════════════════════════════════════════════════════════
📂 Resuming /next-phase Development Lifecycle

Brief: Add PDF export for conversations
Last Phase: 5.5 (Cross Analysis) - COMPLETED
Next Phase: 5.6 (Req-Spec Validation)

═══════════════════════════════════════════════════════════════════════════════
▶ Phase 5.6: Requirement-Specification Validation ⛔ GATE

Invoking: requirements-spec-validator agent
...
```

### Gate Failure

```
═══════════════════════════════════════════════════════════════════════════════
⛔ Phase 5.6 FAILED: Req-Spec Validation

Gate Result: FAILED (85% coverage)
Missing Specifications:
- FR-003: Analytics export (no SPEC-xxx found)
- FR-007: Bulk download (partial coverage)

Action Required:
Return to Phase 5 and add specifications for:
1. SPEC-010: Analytics export API endpoint
2. SPEC-011: Bulk download implementation

Run: /next-phase 5 to continue from specifications
═══════════════════════════════════════════════════════════════════════════════
```

## Integration with Other Commands

| Command | Integration Point |
|---------|-------------------|
| `/full-review` | Invoked at Phase 9 |
| `/update-docs` | Invoked at Phase 11 |
| `/sync-db` | Can be invoked during Phase 6 for schema changes |
| `/test-rag` | Can be invoked during Phase 8 for RAG testing |

## Customization

### Skip Discovery (for known codebases)

```bash
/next-phase --skip-discovery --brief "Add feature X"
```

Starts from Phase 0 instead of Phase 0.1.

### Component-Specific

```bash
/next-phase --component saas --brief "Add feature X"
```

Focuses discovery and analysis on SaaS component only.

### Validate Only

```bash
/next-phase --validate
```

Runs validation phases (5.6, 6.5) without making changes.
