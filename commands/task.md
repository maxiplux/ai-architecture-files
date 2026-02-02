---
description: Development Tasks
---

Read the contents of docs/plan.md and generate a detailed, enumerated task list based on it.

Each task should be clear, actionable, and written in a way that allows it to be marked as completed once done.

Maintain a logical order of execution. Group the tasks logically into phases.

Save the result as docs/tasks.md, formatted as a numbered list with checkboxes (for example, - [ ] Implement data repository).

Do not start implementing any tasks yet—only create the list.

don't read docs/done

# Development Tasks: [Feature Name]
## Metadata

| Field            | Value                              |
|------------------|------------------------------------|
| Task List ID     | TASKS-[YYYYMMDD]-[NNN]             |
| Plan Reference   | [Link to plan.md]                  |
| Requirements Ref | [Link to requirements.md]          |
| Status           | [Not Started / In Progress / Blocked / Completed] |
| Current Phase    | [Phase Number]                     |
| Current Epic     | [Epic Number]                      |
| Current Task     | [Task Number]                      |
| Created          | [YYYY-MM-DD]                       |
| Last Updated     | [YYYY-MM-DD]                       |

---

## Progress Summary

| Phase   | Epic Count | Tasks Total | Completed | Remaining | Blocked |
|---------|------------|-------------|-----------|-----------|---------|
| Phase 1 | [N]        | [N]         | [N]       | [N]       | [N]     |
| Phase 2 | [N]        | [N]         | [N]       | [N]       | [N]     |
| Phase 3 | [N]        | [N]         | [N]       | [N]       | [N]     |
| Phase 4 | [N]        | [N]         | [N]       | [N]       | [N]     |
| **Total** | **[N]**  | **[N]**     | **[N]**   | **[N]**   | **[N]** |

---

## Legend

| Symbol | Meaning                                    |
|--------|---------------------------------------------|
| `[ ]`  | Not started                                |
| `[~]`  | In progress                                |
| `[x]`  | Completed                                  |
| `[!]`  | Blocked (see notes)                        |
| `[?]`  | Needs clarification                        |

---

## Phase 1: Foundation

**Phase Goal:** [High-level objective from plan.md Phase 1]

**Entry Criteria:** [What must be true before starting this phase]

**Exit Criteria:** [What must be true to consider this phase complete]

---

### Epic 1.1: [Epic Name - e.g., Domain Model Setup]

**Epic Goal:** [Specific goal for this epic]

**Estimated Effort:** [S/M/L or story points]

#### Task 1.1.1: [Task Name]

**Description:** [Clear description of what needs to be done]

**Acceptance Criteria:** [How we know this task is complete]

**Artifacts:** [Files to be created/modified]

- [ ] **1.1.1.1:** [Subtask description - atomic action]
- [ ] **1.1.1.2:** [Subtask description]
- [ ] **1.1.1.3:** [Subtask description]

**Verification:** `[Command to verify completion]`

---

#### Task 1.1.2: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **1.1.2.1:** [Subtask description]
- [ ] **1.1.2.2:** [Subtask description]
- [ ] **1.1.2.3:** [Subtask description]

**Verification:** `[Command]`

---

### Epic 1.2: [Epic Name - e.g., Database Schema]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 1.2.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **1.2.1.1:** [Subtask description]
- [ ] **1.2.1.2:** [Subtask description]

**Verification:** `[Command]`

---

#### Task 1.2.2: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **1.2.2.1:** [Subtask description]
- [ ] **1.2.2.2:** [Subtask description]

**Verification:** `[Command]`

---

### Phase 1 Checkpoint

- [ ] **Checkpoint 1.A:** [Verification step - e.g., All migrations run successfully]
- [ ] **Checkpoint 1.B:** [Verification step - e.g., Domain entities compile]
- [ ] **Checkpoint 1.C:** [Verification step - e.g., Repository tests pass]

**Phase Verification Command:** `[Command to verify entire phase]`

---

## Phase 2: Core Logic

**Phase Goal:** [High-level objective from plan.md Phase 2]

**Entry Criteria:** [Phase 1 checkpoints completed]

**Exit Criteria:** [What must be true to consider this phase complete]

---

### Epic 2.1: [Epic Name - e.g., Service Layer Implementation]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 2.1.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **2.1.1.1:** [Subtask description]
- [ ] **2.1.1.2:** [Subtask description]
- [ ] **2.1.1.3:** [Subtask description]

**Verification:** `[Command]`

---

#### Task 2.1.2: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **2.1.2.1:** [Subtask description]
- [ ] **2.1.2.2:** [Subtask description]

**Verification:** `[Command]`

---

### Epic 2.2: [Epic Name - e.g., Unit Test Coverage]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 2.2.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **2.2.1.1:** [Subtask description]
- [ ] **2.2.1.2:** [Subtask description]

**Verification:** `[Command]`

---

### Phase 2 Checkpoint

- [ ] **Checkpoint 2.A:** [Verification step]
- [ ] **Checkpoint 2.B:** [Verification step]
- [ ] **Checkpoint 2.C:** [Verification step]

**Phase Verification Command:** `[Command]`

---

## Phase 3: API Layer

**Phase Goal:** [High-level objective from plan.md Phase 3]

**Entry Criteria:** [Phase 2 checkpoints completed]

**Exit Criteria:** [What must be true to consider this phase complete]

---

### Epic 3.1: [Epic Name - e.g., Controller Implementation]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 3.1.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **3.1.1.1:** [Subtask description]
- [ ] **3.1.1.2:** [Subtask description]
- [ ] **3.1.1.3:** [Subtask description]

**Verification:** `[Command]`

---

#### Task 3.1.2: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **3.1.2.1:** [Subtask description]
- [ ] **3.1.2.2:** [Subtask description]

**Verification:** `[Command]`

---

### Epic 3.2: [Epic Name - e.g., Integration Testing]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 3.2.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **3.2.1.1:** [Subtask description]
- [ ] **3.2.1.2:** [Subtask description]

**Verification:** `[Command]`

---

### Phase 3 Checkpoint

- [ ] **Checkpoint 3.A:** [Verification step]
- [ ] **Checkpoint 3.B:** [Verification step]
- [ ] **Checkpoint 3.C:** [Verification step]

**Phase Verification Command:** `[Command]`

---

## Phase 4: Integration & Hardening

**Phase Goal:** [High-level objective from plan.md Phase 4]

**Entry Criteria:** [Phase 3 checkpoints completed]

**Exit Criteria:** [What must be true to consider this phase complete]

---

### Epic 4.1: [Epic Name - e.g., External Integrations]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 4.1.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **4.1.1.1:** [Subtask description]
- [ ] **4.1.1.2:** [Subtask description]

**Verification:** `[Command]`

---

### Epic 4.2: [Epic Name - e.g., Production Readiness]

**Epic Goal:** [Specific goal]

**Estimated Effort:** [S/M/L]

#### Task 4.2.1: [Task Name]

**Description:** [Clear description]

**Acceptance Criteria:** [Completion criteria]

**Artifacts:** [Files]

- [ ] **4.2.1.1:** [Subtask description]
- [ ] **4.2.1.2:** [Subtask description]

**Verification:** `[Command]`

---

### Phase 4 Checkpoint

- [ ] **Checkpoint 4.A:** [Verification step]
- [ ] **Checkpoint 4.B:** [Verification step]
- [ ] **Checkpoint 4.C:** [Verification step]

**Phase Verification Command:** `[Command]`

---

## Final Verification Checklist

### Functional Completeness

- [ ] All functional requirements implemented (cross-reference RTM in plan.md)
- [ ] All Gherkin scenarios from requirements.md have passing tests
- [ ] Error handling covers all documented error cases

### Non-Functional Compliance

- [ ] Performance requirements verified (NFR-xxx)
- [ ] Security requirements verified (NFR-xxx)
- [ ] Compliance requirements verified (NFR-xxx)

### Documentation

- [ ] API documentation generated and accurate
- [ ] README updated with setup instructions
- [ ] Architecture diagrams reflect final implementation

### Code Quality

- [ ] Code review completed
- [ ] Static analysis passing
- [ ] Test coverage meets threshold

---

## Blocking Issues Log

| Issue ID | Blocking Task(s) | Description                     | Owner   | Status      | Resolution Date |
|----------|------------------|---------------------------------|---------|-------------|-----------------|
| BLK-001  | [Task IDs]       | [What is blocking progress]     | [Name]  | [Open/Resolved] | [Date]       |

---

## Notes & Decisions

| Date       | Note / Decision                              | Impact                    |
|------------|----------------------------------------------|---------------------------|
| [Date]     | [Important decision or discovery]            | [What changed as result]  |

---

## Session Log

| Session    | Date       | Tasks Completed          | Tasks Started          | Notes                      |
|------------|------------|--------------------------|------------------------|----------------------------|
| Session 1  | [Date]     | [Task IDs]               | [Task IDs]             | [Any relevant context]     |
| Session 2  | [Date]     | [Task IDs]               | [Task IDs]             | [Notes]                    |

---