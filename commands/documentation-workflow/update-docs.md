# /update-docs

Automatically update GitBook documentation for AI BotKit by syncing with SaaS and WordPress codebases. This command ensures documentation stays accurate and up-to-date with the actual product.

## When to Use This Command

| Scenario | Use This? | Why |
|----------|-----------|-----|
| **After changing plan limits/pricing** | **YES** | Syncs pricing tables from code to docs |
| **After adding new features** | **YES** | Generates feature documentation |
| **Before major releases** | **YES** | Ensures docs match release |
| **Routine documentation audit** | **YES** | Catches drift between code and docs |
| **After UI changes** | **YES** | Updates screenshots references |

## Description

The `/update-docs` command orchestrates documentation synchronization between:
- **SaaS codebase** (`saas/`) → GitBook documentation (`documentation/`)
- **WordPress plugin** (`wordpress-plugin/`) → GitBook documentation

It identifies outdated documentation, generates updates, and validates documentation integrity.

## Documentation Sources

The command syncs from these authoritative sources:

| Data | Source Location | Target Documentation |
|------|-----------------|---------------------|
| Plan Limits | `saas/src/lib/checkRateLimit.ts` | `plans-and-billing/*.md` |
| Plan Details | `saas/docs/plan-limits.md` | `plans-and-billing/*.md` |
| Feature List | `saas/src/app/` routes | `introduction/key-features.md` |
| API Endpoints | `saas/src/app/api/` | API reference docs |
| WordPress readme | `wordpress-plugin/readme.txt` | `embedding-your-chatbot/*.md` |

## Usage

```bash
/update-docs                      # Full sync with diff preview
/update-docs --apply              # Apply changes without confirmation
/update-docs --section plans      # Update only plans & billing docs
/update-docs --section features   # Update only feature docs
/update-docs --validate           # Validate only, no changes
/update-docs --generate-api       # Generate API documentation
```

### Options

| Option | Description |
|--------|-------------|
| `--apply` | Apply changes without interactive confirmation |
| `--section <name>` | Focus on specific section: `plans`, `features`, `embedding`, `troubleshooting` |
| `--validate` | Validate documentation only, report discrepancies |
| `--generate-api` | Generate API reference documentation |
| `--dry-run` | Show what would change without modifying files |

## Workflow Phases

```
+---------------------------------------------------------------------+
|              /update-docs WORKFLOW                                   |
+---------------------------------------------------------------------+
|                                                                      |
|  PHASE 1: DISCOVERY                                                  |
|          |                                                           |
|          +-> Scan SaaS codebase for authoritative data               |
|          +-> Parse plan limits from checkRateLimit.ts                |
|          +-> Extract feature list from routes and components         |
|          +-> Read WordPress plugin readme.txt                        |
|          v                                                           |
|                                                                      |
|  PHASE 2: ANALYSIS                                                   |
|          |                                                           |
|          +-> Compare code data with documentation                    |
|          +-> Identify discrepancies                                  |
|          +-> Calculate drift score                                   |
|          v                                                           |
|                                                                      |
|  PHASE 3: GENERATION                                                 |
|          |                                                           |
|          +-> Generate updated documentation sections                 |
|          +-> Preserve existing formatting and images                 |
|          +-> Update SUMMARY.md if structure changed                  |
|          v                                                           |
|                                                                      |
|  PHASE 4: VALIDATION                                                 |
|          |                                                           |
|          +-> Validate all internal links                             |
|          +-> Check image references                                  |
|          +-> Verify SUMMARY.md completeness                          |
|          v                                                           |
|                                                                      |
|  PHASE 5: APPLICATION                                                |
|          |                                                           |
|          +-> Show diff preview                                       |
|          +-> Apply changes (with --apply)                            |
|          +-> Generate update report                                  |
|                                                                      |
+---------------------------------------------------------------------+
```

## Phase Details

### Phase 1: Discovery

**Purpose:** Extract authoritative data from codebases

**Actions:**
1. Parse `saas/src/lib/checkRateLimit.ts` for PLAN_LIMITS constant
2. Read `saas/docs/plan-limits.md` for plan descriptions
3. Scan `saas/src/app/api/` for API endpoint documentation
4. Parse `wordpress-plugin/readme.txt` for plugin features
5. Extract UI feature list from React components

**Data Extracted:**
```typescript
interface DiscoveredData {
  plans: {
    name: string;
    chatbots: number;
    monthlyMessages: number;
    storage: number;
  }[];
  features: string[];
  apiEndpoints: {
    path: string;
    method: string;
    description: string;
  }[];
  wordpressFeatures: string[];
}
```

### Phase 2: Analysis

**Purpose:** Compare extracted data with documentation

**Actions:**
1. Parse each documentation file in `documentation/`
2. Extract plan limits from `plans-and-billing/*.md`
3. Extract feature claims from `introduction/key-features.md`
4. Compare with discovered data
5. Generate discrepancy report

**Discrepancy Types:**

| Type | Severity | Description |
|------|----------|-------------|
| **Value Mismatch** | HIGH | Numbers don't match (e.g., 300 vs 500 messages) |
| **Missing Feature** | MEDIUM | Feature in code not in docs |
| **Extra Feature** | LOW | Feature in docs not in code |
| **Outdated Link** | MEDIUM | Link to deprecated resource |
| **Missing Image** | LOW | Referenced image doesn't exist |

### Phase 3: Generation

**Purpose:** Generate updated documentation content

**Actions:**
1. Update plan limit tables with correct values
2. Add missing features to feature lists
3. Remove deprecated features
4. Preserve existing prose and formatting
5. Keep image references intact

**Update Strategy:**
- Tables: Replace entire table with regenerated version
- Lists: Add/remove items preserving order
- Prose: Flag for manual review
- Images: Never auto-remove, flag missing

### Phase 4: Validation

**Purpose:** Ensure documentation integrity

**Actions:**
1. Parse all markdown files for internal links
2. Verify each link target exists
3. Check image paths resolve
4. Validate SUMMARY.md matches file structure

**Validation Report:**
```markdown
## Documentation Validation

### Link Check
- ✅ 45 internal links valid
- ❌ 2 broken links found:
  - `introduction/pricing.md` → File not found
  - `#invalid-anchor` → Anchor not found

### Image Check
- ✅ 98 images valid
- ⚠️ 3 images potentially outdated (>90 days old)

### Structure Check
- ✅ SUMMARY.md matches directory structure
```

### Phase 5: Application

**Purpose:** Apply changes and generate report

**Output:**
```markdown
## Documentation Update Report

### Changes Applied

#### plans-and-billing/upgrading-to-paid-plans.md
- Updated Basic plan messages: 300 → 500
- Updated storage limits table

#### introduction/key-features.md
- Added: "Multi-language support (50+ languages)"
- Removed: (none)

### Manual Review Required
- `troubleshooting-and-faqs/security.md` - Prose may be outdated

### Summary
- 3 files updated
- 5 values corrected
- 0 new files created
- 1 file flagged for review
```

## Data Sync Specifications

### Plan Limits Sync

**Source:** `saas/src/lib/checkRateLimit.ts`
```typescript
export const PLAN_LIMITS = {
  Free: { chatbots: 1, monthlyMessages: 50, storage: 10000000 },
  Basic: { chatbots: 3, monthlyMessages: 500, storage: 15000000 },
  Essential: { chatbots: 6, monthlyMessages: 3000, storage: 25000000 },
  Business: { chatbots: 10, monthlyMessages: 5000, storage: 40000000 },
} as const;
```

**Target:** Documentation tables in:
- `plans-and-billing/upgrading-to-paid-plans.md`
- `plans-and-billing/where-to-find-your-usage.md`
- `introduction/free-plan-and-upgrades.md`

**Sync Rules:**
1. Format messages as "X messages/month"
2. Format storage as "XM characters"
3. Preserve surrounding context

### Feature Sync

**Source:** Multiple files
- `saas/src/app/api/` - API capabilities
- `saas/src/components/` - UI features
- `wordpress-plugin/readme.txt` - Plugin features

**Target:** `introduction/key-features.md`

**Sync Rules:**
1. Group features by category
2. Use consistent phrasing
3. Include WordPress plugin features

## Agent Integration

This command uses the `documentation-updater` agent:

```
Task tool with subagent_type: "aibotkit-engineering:documentation:documentation-updater"
```

The agent handles:
- Code parsing and data extraction
- Markdown manipulation
- Diff generation
- Report formatting

## Examples

### Example 1: Full Sync

```bash
/update-docs

# Output:
## Documentation Analysis

### Discrepancies Found

| File | Issue | Current | Expected |
|------|-------|---------|----------|
| plans-and-billing/upgrading-to-paid-plans.md | Basic messages | 300 | 500 |
| introduction/key-features.md | Missing feature | - | GDPR compliance |

### Proposed Changes

1. Update Basic plan messages in 2 files
2. Add GDPR compliance to features list

Apply changes? [y/N]
```

### Example 2: Validate Only

```bash
/update-docs --validate

# Output:
## Validation Report

✅ 28 documentation files checked
❌ 2 discrepancies found
✅ 45 internal links valid
✅ 98 images valid

See reports/DOC_VALIDATION.md for details
```

### Example 3: Section Update

```bash
/update-docs --section plans --apply

# Output:
## Plans & Billing Updated

Updated 3 files:
- upgrading-to-paid-plans.md
- where-to-find-your-usage.md
- free-plan-and-upgrades.md

All plan limits now match saas/src/lib/checkRateLimit.ts
```

## Output Files

| File | Description |
|------|-------------|
| `reports/DOC_SYNC_REPORT.md` | Full sync report |
| `reports/DOC_VALIDATION.md` | Validation results |
| `reports/DOC_DISCREPANCIES.md` | List of found issues |

## Related Commands

- `/full-review` - Code review (includes doc review)
- `/next-phase` - Phase 7 is documentation phase

## Success Criteria

A successful `/update-docs` execution:
- All plan limits in docs match code
- All features documented correctly
- No broken internal links
- All image references valid
- SUMMARY.md structure complete
- Generated report saved to reports/
