---
description: QA Agent
---

This guide instructs AI coding agents on how to execute comprehensive QA validation against the SDD (Spec-Driven Development) artifacts: `docs/requirements.md`, `docs/plan.md`, and `docs/tasks.md`. Following these instructions ensures systematic, evidence-based validation that confirms implementation completeness and correctness.

**Critical Rule:** This validation is performed at final delivery. Execute ALL validation steps and produce a complete QA Validation Report (Section 15) with evidence for every checkpoint.

**Applicability:** This guide is technology-agnostic and applies to any backend, frontend, or full-stack project regardless of language or framework.

---

## 1. Validation Scope & Philosophy

### 1.1 What This Validation Covers

This QA validation confirms that the implemented system satisfies:

| Document | Validation Focus |
|----------|------------------|
| `requirements.md` | Functional requirements, acceptance criteria, Gherkin scenarios, NFRs |
| `plan.md` | Architecture compliance, API/interface contract, directory structure, RTM status |
| `tasks.md` | Task completion, checkpoint verification, phase exit criteria |

### 1.2 Validation Philosophy

As an AI agent performing QA validation, you must:

1. **Be Evidence-Based:** Every validation claim requires proof (test output, screenshot, code reference, or command result)
2. **Be Comprehensive:** Do not skip any requirement, scenario, or checkpoint
3. **Be Objective:** Report findings accurately regardless of perceived impact
4. **Be Severity-Aware:** Classify failures correctly to enable appropriate response

### 1.3 Severity Classification

| Severity | Definition | Action |
|----------|------------|--------|
| **CRITICAL** | Requirement not implemented, security vulnerability, data integrity risk | Blocks release |
| **MAJOR** | Partial implementation, significant deviation from spec, missing tests | Blocks release |
| **MINOR** | Cosmetic issues, documentation gaps, non-blocking deviations | Warn and document |
| **INFO** | Observations, suggestions, minor improvements | Document only |

---

## 2. Pre-Validation Checklist

Before starting validation, confirm the following prerequisites:

### 2.1 Document Availability

```
[ ] requirements.md is accessible and version-controlled
[ ] plan.md is accessible and version-controlled
[ ] tasks.md is accessible and version-controlled
[ ] Source code repository is accessible
[ ] Test reports are available or tests can be executed
```

### 2.2 Environment Readiness

```
[ ] Application builds without errors
[ ] Application runs successfully
[ ] Required services (database, APIs, etc.) are accessible
[ ] All external dependencies are available or mocked
```

### 2.3 Artifact Collection Preparation

Prepare to collect the following evidence types:

| Evidence Type | Collection Method | Storage Format |
|---------------|-------------------|----------------|
| Test Results | Project test command | Console output / Report |
| Code Coverage | Coverage tool output | HTML/Text report |
| API/UI Responses | Manual or automated tests | JSON/Screenshots |
| Screenshots | Manual capture or automated | PNG/JPEG |
| Log Excerpts | Application logs | Text snippets |
| Static Analysis | Linter/analyzer output | Report summary |

### 2.4 Identify Project Commands

Before proceeding, identify the project-specific commands from `plan.md` or project documentation:

| Action | Command (from plan.md) |
|--------|------------------------|
| Build/Compile | `[PROJECT_BUILD_COMMAND]` |
| Run Tests | `[PROJECT_TEST_COMMAND]` |
| Run Linter/Static Analysis | `[PROJECT_LINT_COMMAND]` |
| Generate Coverage Report | `[PROJECT_COVERAGE_COMMAND]` |
| Start Application | `[PROJECT_START_COMMAND]` |

---

## 3. Validation Execution Protocol

### 3.1 Reading Order (Mandatory)

Execute validation in this precise order:

```
1. Read requirements.md completely
   └── Extract: FR list, NFR list, Gherkin scenarios, acceptance criteria
   
2. Read plan.md completely
   └── Extract: RTM entries, API/interface contracts, directory structure, architectural decisions, project commands
   
3. Read tasks.md completely
   └── Extract: Phase checkpoints, task completion status, blocking issues

4. Execute validation categories in order (Sections 4-9 of this guide)

5. Compile QA Validation Report (Section 15) with all findings
```

### 3.2 Evidence Collection Standards

For each validation item, collect evidence as follows:

**For Test-Based Validation:**
```markdown
**Evidence:** Test `[TestFileName.testName]` 
**Result:** PASSED / FAILED
**Output:** 
\`\`\`
[Relevant test output or assertion]
\`\`\`
```

**For Code-Based Validation:**
```markdown
**Evidence:** File `[path/to/file]`, lines [X-Y]
**Observation:** [What the code shows]
**Reference:** [Link to code or snippet]
```

**For Runtime Validation:**
```markdown
**Evidence:** Command `[command executed]`
**Result:** 
\`\`\`
[Command output]
\`\`\`
**Timestamp:** [YYYY-MM-DD HH:MM:SS]
```

---

## 4. Category 1: Functional Requirements Validation

### 4.1 Requirements Coverage Check

For EACH functional requirement in `requirements.md` Section 3:

```
┌─────────────────────────────────────────────────────────────┐
│ FR-XXX: [Requirement Title]                                 │
├─────────────────────────────────────────────────────────────┤
│ 1. Locate implementation component(s) from RTM in plan.md   │
│ 2. Verify code exists at expected location                  │
│ 3. Verify code comment references FR-XXX                    │
│ 4. Identify test case(s) covering this requirement          │
│ 5. Execute test(s) and capture result                       │
│ 6. Record evidence in validation report                     │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Acceptance Criteria Verification

For EACH acceptance criterion under each FR:

| Check | Method | Evidence Required |
|-------|--------|-------------------|
| Criterion is testable | Review criterion text | Assessment note |
| Test exists for criterion | Search test files | Test file/function name |
| Test passes | Execute test | Pass/Fail + output |
| Implementation matches criterion | Code review | Code reference |

### 4.3 Gherkin Scenario Validation

For EACH scenario in `requirements.md` Section 2.3:

```
Scenario: [Scenario Name]
├── Given: [Precondition]
│   └── Verify: Test setup establishes this state
├── When: [Action]
│   └── Verify: Test executes this action
├── Then: [Outcome]
│   └── Verify: Test asserts this result
└── Test Reference: [TestFile.testFunction]
    └── Result: PASSED / FAILED
```

### 4.4 Functional Validation Checklist

```
[ ] All FR-XXX entries have corresponding implementation
[ ] All FR-XXX entries have at least one test
[ ] All acceptance criteria are covered by tests
[ ] All Gherkin scenarios have corresponding test methods
[ ] All tests pass
[ ] No orphan tests (tests without requirement mapping)
```

---

## 5. Category 2: Non-Functional Requirements Validation

### 5.1 NFR Checklist Approach

For EACH NFR in `requirements.md` Section 4, confirm implementation:

| NFR ID | Category | Requirement | Addressed? | Evidence |
|--------|----------|-------------|------------|----------|
| NFR-001 | [Category] | [Requirement text] | YES / NO / PARTIAL | [How it's addressed] |

### 5.2 NFR Categories and Validation Methods

**Performance (NFR-PERF-XXX):**
```
[ ] Performance requirement documented
[ ] Implementation approach identified
[ ] Performance test exists (if applicable)
[ ] Evidence: [Description of how addressed]
```

**Security (NFR-SEC-XXX):**
```
[ ] Security requirement documented
[ ] Implementation approach identified
[ ] Security control implemented
[ ] Evidence: [Description of how addressed]
```

**Scalability (NFR-SCALE-XXX):**
```
[ ] Scalability requirement documented
[ ] Architecture supports requirement
[ ] Evidence: [Description of how addressed]
```

**Accessibility (NFR-A11Y-XXX):** *(Frontend)*
```
[ ] Accessibility requirement documented
[ ] WCAG compliance level identified
[ ] Accessibility testing performed
[ ] Evidence: [Description of how addressed]
```

**Compatibility (NFR-COMPAT-XXX):** *(Frontend)*
```
[ ] Browser/device compatibility documented
[ ] Cross-browser testing performed
[ ] Evidence: [Description of how addressed]
```

**Compliance (NFR-COMP-XXX):**
```
[ ] Compliance requirement documented
[ ] Implementation approach identified
[ ] Evidence: [Description of how addressed]
```

### 5.3 NFR Validation Output Format

```markdown
### NFR-XXX: [Title]

**Requirement:** [Exact text from requirements.md]

**Category:** [Performance / Security / Scalability / Accessibility / Compliance / Other]

**Addressed:** ✅ YES / ❌ NO / ⚠️ PARTIAL

**Evidence:** 
[Description of how the requirement is addressed in the implementation]

**Notes:** 
[Any observations or concerns]
```

---

## 6. Category 3: Architecture Compliance Validation

### 6.1 Directory Structure Compliance

Compare actual project structure against `plan.md` Section 5:

```
Expected (from plan.md)          Actual (from codebase)           Match?
─────────────────────────────    ─────────────────────────────    ──────
[expected/path/structure]        [actual path]                    ✅ / ❌
[expected/path/structure]        [actual path]                    ✅ / ❌
[expected/path/structure]        [actual path]                    ✅ / ❌
```

### 6.2 Component Diagram Compliance

For EACH component in `plan.md` Section 1.3 diagram:

```
Component: [Component Name]
├── Expected Location: [from plan.md]
├── Actual Location: [from codebase]
├── Expected Pattern/Type: [e.g., Controller, Service, Component, Module]
├── Actual Pattern/Type: [from code]
├── Dependencies Match Diagram: YES / NO
└── Status: COMPLIANT / NON-COMPLIANT
```

### 6.3 Layer/Module Dependency Validation

Verify dependencies flow correctly per the architecture defined in `plan.md`:

```
[ ] Dependency rules from plan.md are followed
[ ] No circular dependencies between modules/layers
[ ] Proper separation of concerns maintained
[ ] External integrations isolated as specified
```

### 6.4 Architectural Decision Compliance

For EACH decision in `plan.md` Section 1.2:

| AD-ID | Decision | Implemented? | Evidence |
|-------|----------|--------------|----------|
| AD-01 | [Decision text] | YES / NO | [Code reference or explanation] |

---

## 7. Category 4: Interface Contract Validation

*Note: This section covers API contracts (backend) and Component/UI contracts (frontend).*

### 7.1 Backend: API Endpoint Compliance

For EACH endpoint in `plan.md` Section 4.1:

```
Endpoint: [METHOD] [Path]
├── Handler/Controller Exists: YES / NO
├── Method/Route Mapping Correct: YES / NO
├── Request Schema Matches Spec: YES / NO
├── Response Schema Matches Spec: YES / NO
├── Status Codes Implemented: [List]
├── Integration Test Exists: YES / NO
└── Integration Test Passes: YES / NO
```

### 7.2 Frontend: Component Contract Compliance

For EACH component in `plan.md`:

```
Component: [Component Name]
├── Component Exists: YES / NO
├── Props/Inputs Match Spec: YES / NO
├── Events/Outputs Match Spec: YES / NO
├── State Management Correct: YES / NO
├── Unit Test Exists: YES / NO
└── Unit Test Passes: YES / NO
```

### 7.3 Request/Response or Props Schema Validation

For EACH data structure defined in `plan.md`:

```
Schema: [Schema Name]
├── Definition Exists: [path/to/definition]
├── Fields Match Spec:
│   ├── field_one: [Type] - MATCH / MISMATCH
│   ├── field_two: [Type] - MATCH / MISMATCH
│   └── ...
├── Validation Rules Present: YES / NO / PARTIAL
└── Status: COMPLIANT / NON-COMPLIANT
```
### 7.4 Error Handling Validation

For EACH error case in `plan.md`:

| Error Case | Error Code/Type | Implemented? | Test Exists? | Test Passes? |
|------------|-----------------|--------------|--------------|--------------|
| Validation Error | [code/type] | YES / NO