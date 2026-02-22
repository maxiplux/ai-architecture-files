# Generate Remediation Plan

## Objective

Read all test execution reports, identify every error, security issue, cross-layer discrepancy, issue, incident, and gap found during test case execution, perform root cause analysis by reading the actual source code, and generate a structured remediation plan under `/docs/remediation/` organized by role with a general folder for cross-cutting concerns. The plan must guide an AI agent through a prioritized, dependency-aware sequence of fixes with inline verification commands and full test re-run references.

## Parameters

- **`$ROLE`** (optional): Generate the plan for a specific role only. Values: `super-admin`, `admin`, `approver`, `purchaser`. If omitted, generates the full plan for all roles.
- **`$LAYER`** (optional): Filter by layer. Values: `ui`, `api`, `both` (default: `both`).
- **`$SEVERITY`** (optional): Filter by severity. Values: `errors-only`, `errors-and-security`, `all` (default: `all`).

**Examples:**
- `/testing/generate-remediation-plan` → full plan, all roles, all layers, all severities
- `/testing/generate-remediation-plan $ROLE=admin` → plan for Admin role only
- `/testing/generate-remediation-plan $ROLE=purchaser $LAYER=api` → API fixes for Purchaser only
- `/testing/generate-remediation-plan $SEVERITY=errors-only` → only errors, skip incidents/gaps

## Instructions

### Phase 0: Project Intelligence Gathering

**This phase is mandatory and must be completed before any analysis.**

1. **Read agent/project documentation:**
   - Search for and read the following files (check root, `.claude/`, `/docs/`, and subdirectories):
     - `AGENTS.md` — architecture patterns, coding conventions, folder structure, dependency injection patterns, error handling patterns.
     - `claude.md` or `.claude/claude.md` or `CLAUDE.md` — project-specific instructions, preferences, constraints.
     - `README.md` — tech stack, project structure, setup, start commands.
   - **Extract and internalize:**
     - Architecture pattern (Vertical Slices, Clean Architecture, etc.) — this dictates WHERE fixes should be applied.
     - Coding conventions and naming patterns — fixes must follow existing patterns.
     - Error handling approach (ProblemDetail, RFC 7807, custom exceptions) — error fixes must use the established pattern.
     - Security configuration approach — security fixes must integrate with existing RBAC setup.
     - Testing conventions — any existing test patterns to follow.
     - Frontend patterns (component structure, state management, API client patterns).

2. **Read the test execution reports:**
   - **Global report:** Read `/docs/test-cases/execution-report.md` — understand the overall pass/fail picture, cross-layer discrepancies, and recommendations.
   - **Role reports:** For each role in scope, read `/docs/test-cases/<role>/report.md` — understand per-role errors, issues, incidents, gaps, and cross-role workflow failures.
   - **Role READMEs:** Read `/docs/test-cases/<role>/README.md` — understand what each role can/cannot do, the test case inventory.
   - **Main README:** Read `/docs/test-cases/README.md` — understand the unified feature map, project type, tech stack summary.

3. **Read failing test cases:**
   - For every test case that has a status of `FAIL`, `BLOCKED`, or contains entries in Errors, Issues, Incidents, or Gaps sections:
     - Read the **test case file** (`TC-*-UI-*.md` or `TC-*-API-*.md`) to understand what was expected.
     - Read the **individual report file** (`TC-*-report.md`) to understand what actually happened, the exact error, screenshots referenced, curl outputs, and discrepancies.
   - Build a **Failure Inventory**: a complete list of every failure with its test case ID, layer, role, severity category, error description, and related test cases.

4. **Read the actual source code:**
   - For each failure in the Failure Inventory, trace to the relevant source code:
     - **API failures:** Read the controller, service, repository, security configuration, exception handler, and DTO files related to the failing endpoint.
     - **UI failures:** Read the page component, form component, API client/hook, middleware, auth provider, and layout files related to the failing page/interaction.
     - **Cross-layer failures:** Read both the backend endpoint code AND the frontend component that calls it.
     - **Auth failures:** Read the security configuration, auth controller, JWT filter, login page component, auth provider/context.
   - This step is critical — **do not generate fix suggestions without reading the actual code**. The fix must be accurate and specific to the codebase.

### Phase 1: Root Cause Analysis & Grouping

1. **Classify every failure** into severity categories (in this order):
   - **S1 — Errors:** Test cases that failed with unexpected behavior, crashes, wrong responses, broken functionality.
   - **S2 — Security / Defense-in-Depth:** Role authorization failures, endpoints accessible by wrong roles, UI hides feature but API allows it.
   - **S3 — Cross-Layer Discrepancies:** UI passed but API failed, or API passed but UI failed for the same feature.
   - **S4 — Issues:** Confirmed defects (wrong data, incorrect status codes, broken validations, UI glitches).
   - **S5 — Incidents:** Unexpected behavior that didn't cause failure (slow responses, deprecation warnings, UI flickers).
   - **S6 — Gaps:** Missing functionality, untestable scenarios, features in docs but not in code.

2. **Group by root cause:**
   - Analyze the Failure Inventory and source code to identify **shared root causes**.
   - Example: If `TC-AD-API-003`, `TC-AP-API-005`, and `TC-PU-API-007` all fail with 403 on endpoints that should be accessible, the root cause might be a single misconfigured `@PreAuthorize` expression or a role hierarchy issue.
   - Example: If `TC-AD-UI-002` and `TC-AD-UI-005` both fail because a form doesn't submit, the root cause might be a broken form handler component.
   - **Group these into a single remediation task** that fixes the root cause and resolves all related test case failures.
   - For isolated failures with no shared root cause, create an individual remediation task.

3. **Categorize tasks by scope:**
   - **General (cross-cutting):** Issues affecting multiple roles or the entire system (auth bugs, shared components, database schema, CORS, API client config, global middleware).
   - **Role-specific:** Issues affecting only a single role's functionality.
   - If a root cause is shared across 2+ roles, it goes in General. If it only affects one role, it goes in that role's folder.

4. **Build dependency graph:**
   - Identify which fixes must happen before others:
     - Auth/security fixes MUST come first (if login is broken, nothing else works).
     - Shared component fixes before role-specific fixes that use those components.
     - Backend fixes before frontend fixes (if the API is wrong, fixing the UI won't help).
     - Database/schema fixes before service-layer fixes.
   - Record dependencies: "REM-GEN-001 must be completed before REM-AD-003".

### Phase 2: Remediation Plan Structure

Create the following folder structure under `/docs/remediation/`:

```
docs/
└── remediation/
    ├── README.md                                    # Master plan: summary, execution order, dependency map
    ├── general/
    │   ├── README.md                                # Cross-cutting issues summary
    │   ├── REM-GEN-001-<fix-slug>.md                # Cross-cutting remediation task
    │   ├── REM-GEN-002-<fix-slug>.md
    │   └── ...
    ├── super-admin/
    │   ├── README.md                                # Role-specific issues summary
    │   ├── REM-SA-001-<fix-slug>.md
    │   └── ...
    ├── admin/
    │   ├── README.md
    │   ├── REM-AD-001-<fix-slug>.md
    │   └── ...
    ├── approver/
    │   ├── README.md
    │   ├── REM-AP-001-<fix-slug>.md
    │   └── ...
    └── purchaser/
        ├── README.md
        ├── REM-PU-001-<fix-slug>.md
        └── ...
```

**Naming conventions:**
- Prefix codes: `GEN` = General, `SA` = Super Admin, `AD` = Admin, `AP` = Approver, `PU` = Purchaser
- Fix slug: short kebab-case description of the fix (e.g., `auth-token-refresh`, `order-form-validation`, `role-hierarchy-config`)
- Sequential numbering per scope: `001`, `002`, etc.

### Phase 3: Remediation Task Generation

Each remediation task file MUST follow this template:

```markdown
# REM-<SCOPE>-<NNN>: <Fix Title>

## Severity
- **Category:** S1 Error | S2 Security | S3 Cross-Layer | S4 Issue | S5 Incident | S6 Gap
- **Priority:** Critical | High | Medium | Low
- **Layer:** Backend | Frontend | Both | Infrastructure

## Problem Description

### What failed
<Clear description of the failure as observed during test execution. What was the user/agent trying to do and what happened instead.>

### Affected Test Cases
| Test Case ID | Role | Layer | Status | Report Link |
|-------------|------|-------|--------|-------------|
| TC-<ID> | <Role> | UI/API | FAIL | [report](../../test-cases/<role>/TC-<ID>-report.md) |
| TC-<ID> | <Role> | UI/API | FAIL | [report](../../test-cases/<role>/TC-<ID>-report.md) |

### Error Evidence
<Copy the exact error messages, wrong status codes, incorrect responses, or screenshot references from the reports.>

**From TC-<ID> report:**
> <Exact error text or observation from the report>

**From TC-<ID> report:**
> <Exact error text or observation from the report>

## Root Cause Analysis

### Identified Root Cause
<Specific explanation of WHY this is failing, based on reading the actual source code.>

### Affected Source Files
| File | Path | What's Wrong |
|------|------|-------------|
| <FileName> | `<full/path/to/file>` | <Specific issue in this file> |
| <FileName> | `<full/path/to/file>` | <Specific issue in this file> |

### Code Reference
<Show the specific lines of code that are causing the issue. Keep it focused — only the relevant section.>

```java
// Example: Current code in OrderController.java (line ~45)
@PreAuthorize("hasRole('ADMIN')")  // ← Bug: should also allow PURCHASER
@PostMapping("/api/v1/orders")
public ResponseEntity<OrderResponse> createOrder(...) {
```

## Remediation Steps

### Step-by-step fix
<Numbered steps that an AI agent can follow to implement the fix. Be specific about file paths, method names, and code changes.>

1. **Open** `<full/path/to/file>`
2. **Locate** `<method/component/section>`
3. **Change** `<specific code to change>`
   - **From:** `<current code>`
   - **To:** `<fixed code>`
4. **Also update** `<any related file that needs to change>`
5. <Additional steps as needed>

### Coding conventions to follow
<Reference any patterns from AGENTS.md or claude.md that apply to this fix.>
- <Convention 1>
- <Convention 2>

## Dependencies

### Must be completed AFTER
| Task ID | Title | Reason |
|---------|-------|--------|
| REM-<ID> | <title> | <Why this dependency exists> |

### Unblocks
| Task ID | Title | Reason |
|---------|-------|--------|
| REM-<ID> | <title> | <Why completing this fix unblocks the other> |

## Verification

### Quick inline verification (run immediately after fix)

**For API fixes:**
```bash
# Verify the fix by reproducing the original failing request
curl -s -w "\n%{http_code}" -X <METHOD> http://localhost:8080<ENDPOINT> \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '<BODY>'

# Expected: <status code>
# Verify specific field:
# jq -r '.<path>' to check the response
```

**For UI fixes:**
<Step-by-step browser actions to verify the fix:>
1. Navigate to `<route>`
2. <Perform the action that was failing>
3. **Expected:** <What should now happen correctly>

### Full test re-execution
Run the following to fully validate the fix and check for regressions:

```bash
# Re-run the specific failing test cases:
/testing/execute-test-cases $TEST_CASE_ID=TC-<ID>
/testing/execute-test-cases $TEST_CASE_ID=TC-<ID>

# After all related fixes in this root cause group are done, run the role suite:
/testing/execute-test-cases $ROLE=<role>
```

## Regression Risk

### Passing test cases that could be affected by this fix
| Test Case ID | Role | Layer | Why it might be affected |
|-------------|------|-------|--------------------------|
| TC-<ID> | <Role> | UI/API | <Reason: shares the same component/endpoint/service> |
| TC-<ID> | <Role> | UI/API | <Reason> |

### Mitigation
<How to minimize regression risk — e.g., "Run the full admin test suite after this fix", "This change is isolated to one component", etc.>

## Notes
- <Any additional context, workarounds, edge cases>
- <Related documentation or external references>
- <If this is a Gap (S6), note what needs to be built/implemented, not just fixed>
```

### Phase 4: Master Plan & README Generation

#### 4.1 — Master README (`/docs/remediation/README.md`)

```markdown
# Remediation Plan

## Context
- **Generated At:** <ISO 8601 timestamp>
- **Based On:** Test Execution Report from <execution timestamp>
- **Parameters:** ROLE=<value or "all"> | LAYER=<value> | SEVERITY=<value>
- **Docs Read:** AGENTS.md, claude.md, credentials.md, execution-report.md, <N> role reports, <N> individual test case reports, <N> source files

## How to Use This Plan

### For AI Agents
This plan is designed to be read and executed sequentially by an AI agent. **Start with the Execution Order below and follow it step by step.**

1. **Read the general folder first** — cross-cutting fixes must be applied before role-specific fixes.
2. **Within each task**, follow the Remediation Steps exactly. The steps reference specific file paths, code locations, and coding conventions from this project.
3. **After each fix**, run the Quick Inline Verification to confirm the fix works.
4. **After completing a group of related fixes**, run the Full Test Re-execution commands.
5. **After completing ALL fixes**, run the Regression Sweep (see below).

### Critical: Holistic Understanding First
Before starting ANY remediation, the agent MUST read the following in this order:
1. `AGENTS.md` and/or `claude.md` — understand the project architecture and conventions
2. This `README.md` — understand the full scope of issues and their dependencies
3. `/docs/test-cases/README.md` — understand the feature map and role structure
4. `/docs/test-cases/execution-report.md` — understand the overall test results

This holistic understanding prevents fixing symptoms instead of root causes, and ensures fixes follow project conventions.

## Executive Summary

| Severity | Count | Description |
|----------|-------|-------------|
| S1 — Errors | <N> | Test failures with broken functionality |
| S2 — Security | <N> | Authorization and defense-in-depth issues |
| S3 — Cross-Layer | <N> | UI/API result discrepancies |
| S4 — Issues | <N> | Confirmed defects |
| S5 — Incidents | <N> | Unexpected non-failure behavior |
| S6 — Gaps | <N> | Missing functionality |
| **Total** | **<N>** | |

## Scope Distribution

| Scope | Tasks | Affected Test Cases | Severity Breakdown |
|-------|-------|--------------------|--------------------|
| General | <N> | <N> | S1: <N>, S2: <N>, S3: <N>, S4: <N>, S5: <N>, S6: <N> |
| Super Admin | <N> | <N> | ... |
| Admin | <N> | <N> | ... |
| Approver | <N> | <N> | ... |
| Purchaser | <N> | <N> | ... |
| **Total** | **<N>** | **<N>** | |

## Dependency Map

<Visual representation of task dependencies. Use a text-based diagram or mermaid-style notation.>

```
REM-GEN-001 (Auth fix)
├── REM-GEN-002 (Role hierarchy)
│   ├── REM-AD-001 (Admin user management)
│   ├── REM-AP-001 (Approver workflow)
│   └── REM-PU-001 (Purchaser ordering)
├── REM-GEN-003 (API client config)
│   ├── REM-AD-UI-002 (Admin dashboard)
│   └── REM-PU-UI-003 (Purchaser order form)
└── REM-SA-001 (Super Admin panel)

REM-GEN-004 (Shared form component)
├── REM-AD-003 (Admin branch form)
└── REM-PU-002 (Purchaser order form)
```

## Execution Order (Linear Plan)

**Follow this exact sequence. Each step lists the task and why it's positioned here.**

| Order | Task ID | Title | Severity | Layer | Why Here |
|-------|---------|-------|----------|-------|----------|
| 1 | REM-GEN-001 | <title> | S1 | Backend | Auth is the foundation — nothing works without it |
| 2 | REM-GEN-002 | <title> | S2 | Backend | Role hierarchy must be correct before role-specific fixes |
| 3 | REM-GEN-003 | <title> | S3 | Frontend | API client config affects all frontend test cases |
| 4 | REM-SA-001 | <title> | S1 | Both | Super Admin has platform-wide scope |
| 5 | REM-AD-001 | <title> | S1 | API | Admin feature fix (backend first) |
| 6 | REM-AD-002 | <title> | S1 | UI | Admin feature fix (frontend, depends on #5) |
| ... | ... | ... | ... | ... | ... |
| N | **REGRESSION SWEEP** | Run full test suite | — | Both | Final validation of all fixes |

### Regression Sweep (Final Step)
After ALL remediation tasks are completed, run the full test suite:
```bash
/testing/execute-test-cases
```
Compare the new `execution-report.md` with the previous one. All previously failing test cases should now pass. Any new failures indicate regressions that need immediate attention.

## Files Modified Tracker
<This section should be updated by the agent as fixes are applied>

| File Path | Modified By Tasks | Status |
|-----------|------------------|--------|
| `<path>` | REM-GEN-001, REM-AD-002 | Pending |
```

#### 4.2 — Scope READMEs

**`/docs/remediation/general/README.md`:**

```markdown
# General (Cross-Cutting) Remediation Tasks

## Overview
These tasks address issues that affect **multiple roles** or the **entire system**. They include authentication bugs, shared component defects, database schema issues, CORS configuration, API client configuration, global middleware, cross-layer discrepancies, and cross-role workflow failures.

**These tasks MUST be completed BEFORE role-specific tasks** because role-specific fixes often depend on the cross-cutting infrastructure being correct first.

## Tasks

| Order | ID | Title | Severity | Layer | Affected Roles | Affected Test Cases |
|-------|-----|-------|----------|-------|----------------|---------------------|
| 1 | REM-GEN-001 | <title> | S1 | Backend | All | TC-xx, TC-yy, TC-zz |
| 2 | REM-GEN-002 | <title> | S2 | Both | Admin, Approver | TC-xx, TC-yy |
```

**`/docs/remediation/<role>/README.md`:**

```markdown
# <Role Name> Remediation Tasks

## Overview
- **Role:** <Role Name>
- **Scope:** <scope from credentials>
- **Total tasks:** <N>
- **Depends on General tasks:** <list of REM-GEN-* IDs that must be done first>

## Prerequisites
Before starting these tasks, ensure the following General tasks are completed:
| Task ID | Title | Status |
|---------|-------|--------|
| REM-GEN-001 | <title> | Pending |

## Tasks

| Order | ID | Title | Severity | Layer | Affected Test Cases | Depends On |
|-------|-----|-------|----------|-------|---------------------|------------|
| 1 | REM-<PREFIX>-001 | <title> | S1 | API | TC-xx | REM-GEN-002 |
| 2 | REM-<PREFIX>-002 | <title> | S4 | UI | TC-yy | REM-<PREFIX>-001 |
```

### Phase 5: Validation

Before finishing:

1. **Coverage check:** Every test case with status FAIL, or with entries in Errors/Issues/Incidents/Gaps sections, must be referenced in at least one remediation task.
2. **Dependency integrity:** Every dependency reference (`Must be completed AFTER`, `Unblocks`) must point to a valid task ID.
3. **Execution order consistency:** The linear execution order must respect all dependencies (no task listed before its dependency).
4. **Verification completeness:** Every remediation task must have both inline verification commands AND full test re-execution references.
5. **Regression coverage:** Every task must list at least the directly related passing test cases in the Regression Risk section.
6. **Source file accuracy:** Every file path in `Affected Source Files` must exist in the codebase. Verify with a file existence check.
7. **No orphan tasks:** Every role-specific task must have a corresponding entry in the role's README. Every general task must be in the general README.

Report a summary:
```
============================================
REMEDIATION PLAN GENERATED
============================================
Source: execution-report.md from <timestamp>

Tasks Generated:
  General:     <N> (S1: <N>, S2: <N>, S3: <N>, S4: <N>, S5: <N>, S6: <N>)
  Super Admin: <N>
  Admin:       <N>
  Approver:    <N>
  Purchaser:   <N>
  Total:       <N>

Affected Test Cases: <N> out of <total executed>
Affected Source Files: <N>
Dependency Chains: <N>

Execution Order: <N> steps + 1 regression sweep

Docs Read:
  - AGENTS.md / claude.md
  - execution-report.md
  - <N> role reports
  - <N> individual test case reports
  - <N> source code files
============================================
```

## Important Notes

- **ALWAYS read AGENTS.md and/or claude.md FIRST.** The remediation must follow existing project patterns. Fixes that break conventions create new problems.
- **ALWAYS read the actual source code** referenced by failing test cases. Do not guess at fixes based solely on error descriptions — trace to the code.
- **Root cause grouping is critical.** If 5 test cases fail because of one misconfigured annotation, there should be ONE remediation task, not five. The task lists all 5 test cases as affected.
- **General fixes come first.** The execution order MUST place cross-cutting fixes (auth, shared components, global config) before role-specific fixes. This prevents wasted effort fixing symptoms.
- **Holistic understanding before remediation.** The README explicitly instructs the agent to read project docs, the full plan, and the test execution reports before starting any fix. This prevents tunnel vision.
- **Defense-in-depth discrepancies are S2 Security.** If the UI hides a button but the API allows the call (or vice versa), this is a security issue, not just a mismatch.
- **Gaps (S6) require implementation, not just fixes.** Gap tasks should describe what needs to be built, referencing the feature from `AGENTS.md` or requirements docs, not just what was missing during testing.
- **Do not modify test cases.** The remediation fixes the application, not the tests. If a test case is wrong, note it in the task's Notes section but fix the app to match the expected behavior.
- **After completing ALL tasks, the agent MUST run the regression sweep.** This is the final step in the execution order and is non-optional.
