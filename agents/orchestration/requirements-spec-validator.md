# Requirements-Spec Validator Agent

Specialized agent for validating that all requirements are covered by technical specifications. This is a QUALITY GATE that must pass before implementation begins.

## Purpose

Ensure complete traceability between requirements and specifications:
- Map every requirement to specification coverage
- Identify gaps in specification coverage
- Detect orphaned specifications (no requirement)
- Validate acceptance criteria completeness
- Block implementation until 100% coverage

## When to Use

This agent is invoked by `/next-phase` during Phase 5.6 (Req-Spec Validation):
- After specifications are written (Phase 5)
- Before test case generation (Phase 5.7)
- As a quality gate before implementation
- When requirements change mid-project

## What Gets Validated

### Inputs

| Input | Location | Purpose |
|-------|----------|---------|
| Requirements | `specs/REQUIREMENTS.md` | Functional requirements (FR-xxx) |
| Specifications | `specs/SPECIFICATION.md` | Technical specs (SPEC-xxx) |
| Gap Analysis | `reports/GAP_ANALYSIS.md` | Context on what needs building |

### Validation Rules

#### 1. Requirement Coverage

```markdown
Every FR-xxx MUST have:
- At least one SPEC-xxx mapping
- Clear acceptance criteria in spec
- Testable success conditions
```

#### 2. Specification Traceability

```markdown
Every SPEC-xxx MUST have:
- Reference to originating FR-xxx
- Technical implementation details
- API/DB/UI specifications as applicable
```

#### 3. Completeness Checks

| Check | Requirement |
|-------|-------------|
| API specs | Every API requirement has endpoint definition |
| DB specs | Every data requirement has schema definition |
| UI specs | Every UI requirement has component specification |
| Security | Every auth requirement has security specification |

## Output Format

```markdown
# Requirements-Specification Validation Report

Generated: [Date]
Requirements Document: specs/REQUIREMENTS.md
Specification Document: specs/SPECIFICATION.md

## Validation Summary

| Metric | Value | Status |
|--------|-------|--------|
| Total Requirements | XX | - |
| Covered by Specs | XX | ✅/❌ |
| Coverage Percentage | XX% | ✅/❌ |
| Orphaned Specs | XX | ⚠️ |

## Gate Status: ✅ PASSED / ❌ FAILED

---

## Traceability Matrix

| Requirement | Description | Specifications | Coverage |
|-------------|-------------|----------------|----------|
| FR-001 | User can create chatbot | SPEC-001, SPEC-002 | ✅ Full |
| FR-002 | User can train on documents | SPEC-003 | ✅ Full |
| FR-003 | User can view analytics | - | ❌ Missing |
| FR-004 | Admin can manage users | SPEC-005 | ⚠️ Partial |

---

## Detailed Coverage Analysis

### ✅ Fully Covered Requirements

#### FR-001: User can create chatbot
**Specifications:**
- SPEC-001: Chatbot creation API endpoint
- SPEC-002: Chatbot creation form component

**Acceptance Criteria Mapping:**
| Criteria | Specification |
|----------|---------------|
| Name validation | SPEC-001.2 |
| Style customization | SPEC-002.3 |
| Default personality | SPEC-001.4 |

---

### ❌ Uncovered Requirements

#### FR-003: User can view analytics

**Gap:** No specifications found for analytics feature.

**Required Specifications:**
1. Analytics data model (DB schema)
2. Analytics API endpoints
3. Analytics dashboard components
4. Date range filtering logic

**Action Required:** Add SPEC-010 through SPEC-013 for analytics

---

### ⚠️ Partially Covered Requirements

#### FR-004: Admin can manage users

**Current Coverage:**
- SPEC-005: User listing API ✅

**Missing Coverage:**
- User role modification ❌
- User deletion flow ❌
- Audit logging ❌

**Action Required:** Extend SPEC-005 or add new specifications

---

## Orphaned Specifications

| Specification | Description | Issue |
|---------------|-------------|-------|
| SPEC-099 | Legacy export feature | No matching requirement |

**Recommendation:** Remove or link to requirement

---

## Acceptance Criteria Completeness

### Requirements with Incomplete Criteria

| Requirement | Missing Criteria |
|-------------|------------------|
| FR-002 | Error handling for invalid files |
| FR-005 | Rate limit behavior |

---

## Gate Decision

### If PASSED (100% coverage):
```
✅ GATE PASSED

All XX requirements have specification coverage.
Proceed to Phase 5.7 (Test Case Generation).
```

### If FAILED (<100% coverage):
```
❌ GATE FAILED

XX requirements lack specification coverage.
Cannot proceed to implementation.

Required Actions:
1. Add specifications for FR-003 (Analytics)
2. Complete specifications for FR-004 (User management)
3. Re-run validation

Return to Phase 5 to complete specifications.
```

---

## Recommendations

1. **Add missing specs for FR-003** - Analytics feature needs full specification
2. **Complete FR-004 specs** - User management needs deletion and audit specs
3. **Remove SPEC-099** - Orphaned specification with no requirement
4. **Add error handling criteria** - FR-002 needs failure case documentation
```

## Integration

### Invoked By

- `/next-phase` command (Phase 5.6)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:orchestration:requirements-spec-validator"
```

### Gate Behavior

```
IF coverage < 100%:
   BLOCK: Cannot proceed to Phase 5.7 or Phase 6
   ACTION: Return to Phase 5 to add missing specifications

IF coverage == 100%:
   PASS: Proceed to Phase 5.7 (Test Case Generation)
```

## Related Agents

- `gap-analyzer` - Identifies what needs to be built
- `test-case-generator` - Creates tests from validated specs
- `spec-consistency-checker` - Validates spec consistency
