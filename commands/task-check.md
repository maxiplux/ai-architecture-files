---
description: AI Agent Guide: Loop Validation - Plan to Tasks 
---

## Document Purpose
This guide instructs AI coding agents on how to validate that `tasks.md` fully addresses all items in `plan.md`. The validation ensures 100% traceability from implementation plan to executable tasks with no gaps and no orphans.

**Critical Rule:** 100% coverage is required. Every component, API, architectural decision, and NFR in `plan.md` must trace to tasks in `tasks.md`. Every task must trace back to `plan.md`.

**Applicability:** This guide is technology-agnostic and applies to any project regardless of language or framework.

don't read docs/done
---

## 1. Validation Scope

### 1.1 What This Validation Covers

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOOP-TASK VALIDATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   plan.md                              tasks.md                 │
│   ┌─────────────────┐                 ┌─────────────────┐      │
│   │ Component A     │ ───────────────▶│ Task 1.1.1      │      │
│   │ Component B     │ ───────────────▶│ Task 1.1.2      │      │
│   │ API Endpoint 1  │ ───────────────▶│ Task 2.1.1      │      │
│   │ API Endpoint 2  │ ───────────────▶│ Task 2.1.2      │      │
│   │ AD-001          │ ───────────────▶│ Task 1.2.1      │      │
│   │ NFR-001 (via AD)│ ───────────────▶│ Task 3.1.1      │      │
│   │ Phase 1 (plan)  │ ───────────────▶│ Phase 1 (tasks) │      │
│   └─────────────────┘                 └─────────────────┘      │
│                                                                 │
│   ◀──────────────── Orphan Check ────────────────▶             │
│                                                                 │
│   Verification Commands: Every task must have one              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Validation Directions

| Direction | Description | Failure Type |
|-----------|-------------|--------------|
| Forward | plan.md → tasks.md | GAP (missing task coverage) |
| Backward | tasks.md → plan.md | ORPHAN (untraceable task) |

### 1.3 Coverage Requirements

| Item Type | Required Coverage | Validation Target |
|-----------|-------------------|-------------------|
| Components (Section 1.3) | 100% | At least one task per component |
| API Endpoints (Section 4) | 100% | At least one task per endpoint |
| Architectural Decisions (Section 1.2) | 100% | Implementation task exists |
| NFRs (via ADs) | 100% | Task implements NFR support |
| Phases (Section 6) | 100% | Phase alignment between documents |
| Schema/DTOs (Section 4.2) | 100% | Task for each data structure |
| Verification Commands | 100% | Every task has verification |

---

## 2. Pre-Validation Checklist

Before starting validation:

```
[ ] plan.md is accessible and complete
[ ] tasks.md is accessible and complete
[ ] Both documents are version-controlled
[ ] Document versions are compatible (check metadata)
[ ] Phase definitions align conceptually
```

---

## 3. Validation Execution Protocol

### 3.1 Reading Order

```
1. Read plan.md completely
   └── Extract: Components, APIs, ADs, NFR mappings, Phases, Schemas
   
2. Read tasks.md completely
   └── Extract: All tasks with IDs, phases, epics, verification commands

3. Build traceability index (Section 4)

4. Execute forward validation (Section 5)

5. Execute backward validation (Section 6)

6. Validate verification commands (Section 7)

7. Generate coverage report with skeleton suggestions (Section 10)
```

### 3.2 Data Extraction Templates

**From plan.md, extract:**

```
Components (Section 1.3):
- Name: [component name]
- Type: [Controller/Service/Repository/Component/Module]
- Location: [expected file path from Section 5]
- Purpose: [description]

API Endpoints (Section 4.1):
- Method: [HTTP method]
- Path: [endpoint path]
- Description: [what it does]
- Request/Response: [schemas referenced]

Architectural Decisions (Section 1.2):
- ID: AD-xxx
- Decision: [what was decided]
- Supports: [NFR-xxx if applicable]

Phases (Section 6):
- Phase Number: [N]
- Phase Name: [name]
- Goal: [objective]
- Deliverables: [list]
- Verification Command: [command]

Schemas/DTOs (Section 4.2):
- Name: [schema name]
- Fields: [field list]
- Validations: [rules]

Directory Structure (Section 5):
- Path: [file path]
- Purpose: [what goes here]
```

**From tasks.md, extract:**

```
Phases:
- Phase Number: [N]
- Phase Name: [name]
- Goal: [from tasks.md]

Epics:
- ID: [N.M]
- Name: [epic name]
- Goal: [epic goal]

Tasks:
- ID: [N.M.P]
- Name: [task name]
- Description: [what to do]
- Artifacts: [files to create/modify]
- Verification: [command]

Subtasks:
- ID: [N.M.P.Q]
- Description: [atomic action]
- Status: [ ] / [x] / [~] / [!] / [?]

Checkpoints:
- ID: [N.X]
- Description: [what to verify]
```

---

## 4. Build Traceability Index

### 4.1 Forward Index (Plan → Tasks)

Create a mapping table:

```
┌────────────────┬─────────────┬─────────────┬──────────┐
│ Plan Item      │ Plan Section│ Task ID     │ Status   │
├────────────────┼─────────────┼─────────────┼──────────┤
│ Component A    │ Section 1.3 │ Task 1.1.1  │ FOUND    │
│ Component B    │ Section 1.3 │ Task 1.1.2  │ FOUND    │
│ POST /api/users│ Section 4.1 │ Task 2.1.1  │ FOUND    │
│ GET /api/users │ Section 4.1 │ -           │ GAP      │
│ AD-001         │ Section 1.2 │ Task 1.2.1  │ FOUND    │
│ UserDTO        │ Section 4.2 │ Task 1.1.1  │ FOUND    │
└────────────────┴─────────────┴─────────────┴──────────┘
```

### 4.2 Backward Index (Tasks → Plan)

Create a reverse mapping table:

```
┌─────────────┬─────────────────┬────────────────┬──────────┐
│ Task ID     │ Task Name       │ Plan Item      │ Status   │
├─────────────┼─────────────────┼────────────────┼──────────┤
│ Task 1.1.1  │ Create Entity   │ Component A    │ TRACED   │
│ Task 1.1.2  │ Create Service  │ Component B    │ TRACED   │
│ Task 2.1.1  │ Create Endpoint │ POST /api/users│ TRACED   │
│ Task 3.5.1  │ Add Logging     │ -              │ ORPHAN   │
└─────────────┴─────────────────┴────────────────┴──────────┘
```

### 4.3 Phase Alignment Index

```
┌─────────────┬─────────────────┬─────────────────┬──────────┐
│ Phase (plan)│ Goal (plan)     │ Phase (tasks)   │ Status   │
├─────────────┼─────────────────┼─────────────────┼──────────┤
│ Phase 1     │ Foundation      │ Phase 1         │ ALIGNED  │
│ Phase 2     │ Core Logic      │ Phase 2         │ ALIGNED  │
│ Phase 3     │ API Layer       │ -               │ MISSING  │
└─────────────┴─────────────────┴─────────────────┴──────────┘
```

---

## 5. Forward Validation: Plan → Tasks

### 5.1 Component Task Coverage

For EACH component in `plan.md` Section 1.3:

```
Component Validation:
├── Task Exists for Creation?
│   └── Search tasks.md for task creating this component
├── Task Artifacts Include Component File?
│   └── Check task "Artifacts" field lists expected file
├── Component in Correct Phase?
│   └── Verify task phase matches plan.md phase for this component
└── Status: COVERED / GAP
```

**Validation Rules:**
- Each component must have at least one task that creates/implements it
- Task artifacts must reference the component's expected file path
- Task must be in the appropriate phase per plan.md Section 6

### 5.2 API Endpoint Task Coverage

For EACH endpoint in `plan.md` Section 4.1:

```
Endpoint Validation:
├── Task Exists for Implementation?
│   └── Search tasks.md for task implementing this endpoint
├── Request/Response Handling Tasks?
│   └── Tasks exist for DTOs used by this endpoint
├── Error Handling Tasks?
│   └── Task covers error responses for this endpoint
├── Integration Test Task?
│   └── Task exists for testing this endpoint
└── Status: COVERED / GAP
```

**Validation Rules:**
- Each endpoint must have implementation task
- Associated DTOs must have tasks
- Test task should exist for the endpoint

### 5.3 Architectural Decision Task Coverage

For EACH AD-xxx in `plan.md` Section 1.2:

```
AD Validation:
├── Implementation Task Exists?
│   └── Task that implements this architectural decision
├── If AD Supports NFR, NFR Task Exists?
│   └── Task specifically addresses the NFR requirement
├── Decision Reflected in Task Description?
│   └── Task description mentions AD or its purpose
└── Status: COVERED / GAP
```

**Validation Rules:**
- Each AD must have at least one task implementing it
- If AD supports an NFR, that NFR must be traceable to a task

### 5.4 Schema/DTO Task Coverage

For EACH schema in `plan.md` Section 4.2:

```
Schema Validation:
├── Task Exists for Creation?
│   └── Task creates this DTO/schema file
├── Validation Rules Task?
│   └── Task implements validation annotations/rules
├── Mapper Task (if needed)?
│   └── Task for entity-DTO mapping
└── Status: COVERED / GAP
```

### 5.5 Phase Alignment Validation

For EACH phase in `plan.md` Section 6:

```
Phase Validation:
├── Phase Exists in tasks.md?
│   └── Matching phase number and similar name
├── Phase Goals Align?
│   └── tasks.md phase goal matches plan.md phase goal
├── Phase Deliverables Have Tasks?
│   └── Each deliverable in plan.md has corresponding tasks
├── Phase Verification Command Matches?
│   └── tasks.md phase verification matches plan.md
├── Entry/Exit Criteria Defined?
│   └── tasks.md has entry/exit criteria for phase
└── Status: ALIGNED / MISALIGNED / MISSING
```

### 5.6 NFR Task Traceability

For EACH NFR referenced in `plan.md` (via ADs or directly):

```
NFR Task Validation:
├── AD Supporting NFR Has Task?
│   └── AD-xxx task exists (from 5.3)
├── Task Description References NFR?
│   └── Task mentions NFR-xxx or its category
├── Implementation Addresses NFR?
│   └── Task deliverable satisfies NFR metric
└── Status: COVERED / GAP
```

---

## 6. Backward Validation: Tasks → Plan

### 6.1 Task Origin Check

For EACH task in `tasks.md`:

```
Task Validation:
├── Traces to Component?
│   └── Task creates/modifies component from plan.md Section 1.3
├── Traces to API Endpoint?
│   └── Task implements endpoint from plan.md Section 4.1
├── Traces to AD?
│   └── Task implements decision from plan.md Section 1.2
├── Traces to Schema?
│   └── Task creates schema from plan.md Section 4.2
├── Traces to Directory Structure?
│   └── Task artifacts match paths in plan.md Section 5
└── Status: TRACED / ORPHAN
```

### 6.2 Epic Origin Check

For EACH epic in `tasks.md`:

```
Epic Validation:
├── Epic Goal Maps to Plan?
│   └── Epic objective aligns with plan.md component/feature
├── All Epic Tasks Traced?
│   └── Every task in epic has plan.md origin
└── Status: TRACED / ORPHAN
```

### 6.3 Phase Origin Check

For EACH phase in `tasks.md`:

```
Phase Validation:
├── Phase Exists in plan.md?
│   └── Matching phase in plan.md Section 6
├── Phase Goal Matches Plan?
│   └── Goals are aligned
└── Status: TRACED / ORPHAN
```

---

## 7. Verification Command Validation

### 7.1 Task Verification Check

For EACH task in `tasks.md`:

```
Verification Validation:
├── Verification Command Exists?
│   └── Task has non-empty "Verification" field
├── Command is Executable?
│   └── Command syntax is valid for project type
├── Command Tests Task Outcome?
│   └── Command would verify task completion
└── Status: VALID / MISSING / INVALID
```

### 7.2 Phase Verification Check

For EACH phase in `tasks.md`:

```
Phase Verification Validation:
├── Phase Verification Command Exists?
│   └── Phase has verification command defined
├── Command Matches plan.md?
│   └── Command aligns with plan.md Section 6 verification
├── Checkpoints Have Verification?
│   └── Each checkpoint can be verified
└── Status: VALID / MISSING / MISMATCHED
```

---

## 8. Gap Classification

### 8.1 Gap Types

| Gap Type | Severity | Description |
|----------|----------|-------------|
| MISSING_COMPONENT_TASK | 🔴 CRITICAL | Component has no imple