# Gap Analyzer Agent

Specialized agent for comparing current codebase capabilities against new requirements to identify gaps, reuse opportunities, and effort adjustments.

## Purpose

Analyze the gap between existing functionality and new requirements to:
- Classify requirements by reuse potential
- Calculate effort adjustment factors
- Identify breaking changes risks
- Recommend implementation order
- Preserve backward compatibility

## When to Use

This agent is invoked by `/next-phase` during Phase 0.3 (Gap Analysis):
- After codebase discovery (Phase 0.1)
- When planning new features
- Before estimation (Phase 1)
- To determine build vs extend decisions

## What Gets Analyzed

### Inputs

| Input | Source | Purpose |
|-------|--------|---------|
| Discovery Report | `_project_specs/DISCOVERY_REPORT.md` | Current capabilities |
| Requirements Brief | `--brief` flag or user input | New requirements |
| Existing Specs | `specs/` directory | Previous specifications |

### Gap Classification

| Classification | Match Score | Effort Factor | Meaning |
|----------------|-------------|---------------|---------|
| **REUSABLE** | >90% | 0.1 (90% savings) | Use existing code as-is |
| **EXTEND** | 60-90% | 0.5 (50% savings) | Modify existing code |
| **PARTIAL** | 30-60% | 0.8 (20% savings) | Some reuse, significant new work |
| **NEW** | <30% | 1.0 (no savings) | Build from scratch |

### Analysis Dimensions

#### 1. Functional Gap Analysis

```markdown
For each requirement:
1. Search existing capabilities for similar functionality
2. Compare API signatures, data models, UI patterns
3. Assess modification complexity
4. Classify reuse potential
```

#### 2. Data Model Gap Analysis

```markdown
For each data requirement:
1. Check existing Drizzle schema tables
2. Identify missing columns/tables
3. Assess migration complexity
4. Flag breaking changes
```

#### 3. API Gap Analysis

```markdown
For each API requirement:
1. Check existing Next.js API routes
2. Identify missing endpoints
3. Assess modification to existing endpoints
4. Check WordPress plugin API compatibility
```

#### 4. UI/UX Gap Analysis

```markdown
For each UI requirement:
1. Check existing React components
2. Identify reusable patterns (shadcn/ui)
3. Assess new component needs
4. Check accessibility compliance
```

## Output Format

```markdown
# Gap Analysis Report

Generated: [Date]
Requirements Source: [brief file or user input]
Discovery Report: _project_specs/DISCOVERY_REPORT.md

## Executive Summary

| Metric | Value |
|--------|-------|
| Total Requirements | XX |
| Reusable (>90%) | XX (XX%) |
| Extend (60-90%) | XX (XX%) |
| Partial (30-60%) | XX (XX%) |
| New (<30%) | XX (XX%) |
| **Effort Adjustment** | **-XX%** |

---

## Requirement Classification

### REUSABLE (Use As-Is)

| Requirement | Existing Capability | Match | Notes |
|-------------|---------------------|-------|-------|
| User authentication | `src/lib/auth/session.ts` | 95% | JWT auth already implemented |
| Chatbot CRUD | `src/app/api/chatbots/` | 92% | Full CRUD exists |

### EXTEND (Modify Existing)

| Requirement | Existing Capability | Match | Gap | Effort |
|-------------|---------------------|-------|-----|--------|
| Analytics dashboard | `src/app/(dashboard)/` | 75% | Need charts component | 0.5x |
| Multi-language support | `src/lib/ai/rag-engine.ts` | 65% | Add language detection | 0.5x |

### PARTIAL (Some Reuse)

| Requirement | Existing Capability | Match | Gap | Effort |
|-------------|---------------------|-------|-----|--------|
| Team collaboration | `teams` table exists | 45% | Need real-time sync | 0.8x |
| Custom training | Document upload exists | 40% | Need fine-tuning pipeline | 0.8x |

### NEW (Build from Scratch)

| Requirement | Similar Existing | Match | Notes | Effort |
|-------------|------------------|-------|-------|--------|
| Voice chat | None | 0% | New WebRTC integration | 1.0x |
| A/B testing | None | 10% | New feature entirely | 1.0x |

---

## Data Model Gaps

### New Tables Required

| Table | Purpose | Columns | Relations |
|-------|---------|---------|-----------|
| `aibotkit_analytics` | Usage tracking | 6 | chatbots |
| `aibotkit_ab_tests` | A/B testing | 8 | chatbots |

### Column Additions

| Table | Column | Type | Migration Risk |
|-------|--------|------|----------------|
| `aibotkit_chatbots` | `language` | varchar(10) | LOW - nullable |
| `aibotkit_chatbots` | `analytics_enabled` | boolean | LOW - default false |

### Breaking Changes

| Change | Risk | Mitigation |
|--------|------|------------|
| None identified | - | - |

---

## API Gaps

### New Endpoints Required

| Endpoint | Method | Purpose | Complexity |
|----------|--------|---------|------------|
| `/api/analytics` | GET | Fetch analytics | Medium |
| `/api/analytics/export` | GET | Export CSV | Low |

### Endpoint Modifications

| Endpoint | Current | Required Change | Breaking? |
|----------|---------|-----------------|-----------|
| `/api/chatbots` | Basic CRUD | Add analytics field | No |

---

## WordPress Plugin Gaps

### New Features Required

| Feature | SaaS Dependency | Complexity |
|---------|-----------------|------------|
| Analytics widget | `/api/analytics` | Medium |
| Language selector | Chatbot config | Low |

### API Compatibility

| Requirement | Compatible? | Notes |
|-------------|-------------|-------|
| Analytics API | Yes | New endpoint needed |
| Multi-language | Yes | Config change only |

---

## Backward Compatibility Assessment

### Safe Changes
- All new features are additive
- No existing API changes required
- Database migrations are non-breaking

### Risk Areas
- None identified

### Regression Test Focus
- Existing chatbot CRUD operations
- Chat streaming functionality
- Stripe webhook processing

---

## Effort Calculation

### Base Estimate Adjustments

| Category | Requirements | Avg Effort Factor | Adjustment |
|----------|--------------|-------------------|------------|
| REUSABLE | 5 | 0.1 | -4.5 units |
| EXTEND | 3 | 0.5 | -1.5 units |
| PARTIAL | 2 | 0.8 | -0.4 units |
| NEW | 2 | 1.0 | 0 units |

**Total Effort Adjustment: -35%** (from reuse)

---

## Implementation Order Recommendation

### Phase 1: Foundation (Low Risk)
1. Database schema additions (migrations)
2. New API endpoints
3. REUSABLE features verification

### Phase 2: Extensions (Medium Risk)
1. EXTEND features implementation
2. WordPress plugin updates
3. PARTIAL features

### Phase 3: New Features (Higher Risk)
1. NEW features implementation
2. Integration testing
3. Performance optimization

---

## Recommendations

1. **Start with schema migrations** - Non-breaking, enables other work
2. **Extend existing RAG engine** - Multi-language can reuse 65% of code
3. **Build analytics incrementally** - Start with basic metrics
4. **Delay voice chat** - Highest complexity, lowest reuse
```

## Integration

### Invoked By

- `/next-phase` command (Phase 0.3)

### Agent Invocation

```
Task tool with subagent_type: "aibotkit-engineering:orchestration:gap-analyzer"
```

## Related Agents

- `code-capability-indexer` - Provides discovery report input
- `requirements-spec-validator` - Validates gaps are addressed in specs
- `architecture-reviewer` - Reviews extension approach
