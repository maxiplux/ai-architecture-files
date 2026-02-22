# Execute Remediation

## Objective

Read the remediation plan from `/docs/remediation/`, understand the project architecture from `AGENTS.md`/`claude.md`, and **autonomously apply fixes** to both backend and frontend codebases following the prioritized execution order. After each fix, verify it works (retry with an alternative strategy if it doesn't), commit to git, and produce a comprehensive post-remediation report comparing before/after test results.

## Parameters

- **`$ROLE`** (optional): Apply fixes for a specific role only. Values: `super-admin`, `admin`, `approver`, `purchaser`, `general`. If omitted and no `$TASK_ID` is provided, executes the full plan in order.
- **`$LAYER`** (optional): Filter by layer. Values: `ui`, `api`, `both` (default: `both`).
- **`$SEVERITY`** (optional): Filter by severity. Values: `errors-only` (S1), `errors-and-security` (S1+S2), `all` (default: `all`).
- **`$TASK_ID`** (optional): Execute a single remediation task by ID (e.g., `REM-GEN-001`, `REM-AD-003`). Takes precedence over other filters.

**Examples:**
- `/testing/execute-remediation` → execute the full plan in order (general first, then roles)
- `/testing/execute-remediation $ROLE=general` → apply only cross-cutting fixes
- `/testing/execute-remediation $ROLE=admin $LAYER=api` → apply only Admin backend fixes
- `/testing/execute-remediation $SEVERITY=errors-only` → apply only S1 Error fixes across all scopes
- `/testing/execute-remediation $TASK_ID=REM-GEN-001` → apply a single specific fix

## Instructions

### Phase 0: Intelligence Gathering & Setup

**This phase is mandatory. Do NOT modify any code before completing it.**

1. **Read agent/project documentation:**
   - Search for and read:
     - `AGENTS.md` — architecture patterns, coding conventions, folder structure, build/start commands, dependency injection, error handling patterns.
     - `claude.md` or `.claude/claude.md` or `CLAUDE.md` — project-specific instructions, preferences, constraints.
     - `README.md` — tech stack, project structure, how to build and start services.
   - **Extract critical execution context:**
     - How to build the backend (e.g., `./mvnw compile`, `gradle build`)
     - How to start/restart the backend (e.g., `./mvnw spring-boot:run`, `gradle bootRun`, `docker compose restart backend`)
     - How to start the frontend (e.g., `npm run dev`, `pnpm dev`) — hot reload usually handles frontend changes automatically.
     - Coding conventions: naming, patterns, file organization — **all fixes MUST follow these**.
     - Test conventions: if there are unit/integration tests, any fix should not break them.

2. **Read the remediation plan:**
   - **Master README:** Read `/docs/remediation/README.md` — understand the executive summary, dependency map, and **linear execution order**.
   - **Scope READMEs:** Read each `/docs/remediation/<scope>/README.md` to understand per-scope task lists and prerequisites.
   - **Individual tasks:** Read every `REM-*.md` file in scope (filtered by parameters) to understand the full picture before starting.
   - **Execution report:** Read `/docs/test-cases/execution-report.md` to understand the baseline test results (this is the "before" snapshot for comparison).

3. **Verify environment:**
   - **Backend running:** Check with `curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health` (or detected health endpoint).
   - **Frontend running:** Check with `curl -s -o /dev/null -w "%{http_code}" http://localhost:3000`.
   - **Git status:** Verify the working tree is clean (`git status --porcelain`). If there are uncommitted changes, warn but proceed (the agent should not lose the user's work).
   - **Browser strategy:** Detect MCP Browser Agent first, fall back to Playwright (same as `execute-test-cases`).
   - **Linux tools:** Verify `curl`, `jq`, `grep`, `sed`, `awk`, `diff` are available.
   - Report environment status before proceeding.

4. **Build execution queue:**
   - Read the linear execution order from `/docs/remediation/README.md`.
   - Apply parameter filters (`$ROLE`, `$LAYER`, `$SEVERITY`, `$TASK_ID`).
   - **Respect dependencies:** If a filtered task depends on a task NOT in the current scope, warn: "REM-AD-001 depends on REM-GEN-002 which is not in scope. Results may be incomplete."
   - Store the ordered queue of tasks to execute.
   - Log the queue:
     ```
     ============================================
     REMEDIATION EXECUTION PLAN
     ============================================
     Tasks in queue: <N>
     Order:
       1. REM-GEN-001 — <title> (S1, Backend)
       2. REM-GEN-002 — <title> (S2, Both)
       ...
     ============================================
     ```

### Phase 1: Fix Execution Loop

For each task in the execution queue, follow this workflow:

#### Step 1.1 — Read & Understand the Task

1. Read the full `REM-<SCOPE>-<NNN>.md` file.
2. Identify:
   - **Severity and layer** (Backend, Frontend, or Both)
   - **Affected source files** and their paths
   - **Remediation steps** (the specific code changes)
   - **Dependencies** — verify all prerequisite tasks are already completed in this run. If not, skip with status `SKIPPED: dependency REM-<ID> not yet completed`.
   - **Coding conventions** referenced in the task
3. Read the **actual current source code** of every file listed in `Affected Source Files`. The code may have changed since the plan was generated (e.g., if another fix modified the same file). Adapt the fix if needed.

#### Step 1.2 — Apply the Fix

Execute the remediation steps. The approach depends on the layer:

**Backend fixes (Java/Spring Boot/Kotlin):**

1. Open and modify the source file(s) as specified in the remediation steps.
2. Apply the code change using precise edits (str_replace or targeted modifications). **Do not rewrite entire files** — make surgical changes.
3. Follow the coding conventions from `AGENTS.md`:
   - Use the project's naming patterns
   - Follow the architecture pattern (Vertical Slices, etc.)
   - Use the established error handling approach (ProblemDetail, custom exceptions)
   - Match existing code style (indentation, imports, annotations order)
4. **Check if the backend needs rebuilding/restarting:**
   - If using Spring Boot DevTools with auto-reload → wait 5-10 seconds for restart.
   - If NOT using DevTools → restart the backend using the command from `AGENTS.md`/`README.md`:
     ```bash
     # Example — adapt to the actual project command
     ./mvnw spring-boot:run &
     # Wait for startup
     sleep 15
     curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health
     ```
   - Wait until the backend health endpoint returns 200 before proceeding to verification.

**Frontend fixes (Next.js/React):**

1. Open and modify the source file(s) as specified in the remediation steps.
2. Apply the code change using precise edits. **Do not rewrite entire files.**
3. Follow the coding conventions from `AGENTS.md`:
   - Component naming and structure patterns
   - State management patterns
   - API client/hook patterns
   - Form validation approach (Zod, Yup, etc.)
   - Styling approach (Tailwind, CSS Modules, etc.)
4. **Frontend hot reload handles changes automatically** in most cases.
   - Wait 3-5 seconds for Next.js/Vite hot reload to pick up the change.
   - If the change is to `middleware.ts`, `next.config.*`, `.env*`, or other config files → the frontend may need a full restart:
     ```bash
     # Kill and restart frontend
     # Adapt to actual project command from AGENTS.md
     cd <frontend-path> && npm run dev &
     sleep 10
     curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
     ```

**Cross-layer fixes (Both backend and frontend):**

1. Apply the backend fix first.
2. Verify the backend is running and healthy.
3. Then apply the frontend fix.
4. Wait for hot reload or restart.
5. Verify both services are running before proceeding to verification.

#### Step 1.3 — Inline Verification (Attempt 1)

Run the inline verification commands from the remediation task.

**For API fixes — verify via curl + Linux tools:**
```bash
# Run the exact curl command from the task's Verification section
curl -s -w "\n%{http_code}" -X <METHOD> http://localhost:8080<ENDPOINT> \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '<BODY>'

# Validate with jq
jq -r '.<expected_field>' /tmp/rem-verify-response.json
```

**For UI fixes — verify via browser:**

*If using MCP Browser Agent:*
1. Navigate to the page/route specified in the verification steps.
2. Perform the actions that were previously failing.
3. Visually verify the expected outcome is now correct.
4. Capture a verification screenshot: `/docs/remediation/<scope>/screenshots/REM-<ID>-verified.png`

*If using Playwright:*
1. Launch browser, navigate to the target page.
2. Execute the verification steps programmatically.
3. Assert the expected outcomes.
4. Capture verification screenshot.

**For cross-layer fixes — verify both:**
1. Run the API curl verification first.
2. Then run the browser verification.
3. Both must pass for the fix to be considered successful.

**Evaluate the result:**
- If ALL verification checks pass → **proceed to Step 1.4 (Commit)**.
- If ANY verification check fails → **proceed to Step 1.3b (Retry)**.

#### Step 1.3b — Retry with Alternative Strategy (Attempt 2)

If the first fix attempt fails verification:

1. **Analyze the failure:** Read the verification output. Understand WHY the fix didn't work.
   - Did the code change not take effect? (build/restart issue)
   - Was the root cause analysis incorrect? (need to re-read source code)
   - Did the fix introduce a syntax error? (check build output/console)
   - Was the fix incomplete? (missed a related file)

2. **Re-read the relevant source code** to find what was missed.

3. **Apply an adjusted fix:**
   - If the root cause was partially correct: extend the fix to cover the missed part.
   - If the root cause was wrong: revert the first attempt and apply a different approach.
   - If the fix was correct but a build/restart issue: rebuild/restart and re-verify.

4. **Run inline verification again.**

5. **Evaluate:**
   - If verification passes → proceed to Step 1.4 (Commit).
   - If verification fails again → mark the task as `UNRESOLVED`, log the details, revert the code changes (`git checkout -- <files>`), and continue to the next task. The agent should NOT attempt a third fix — escalate to human review.

#### Step 1.4 — Git Commit

After successful verification:

1. Stage the modified files:
   ```bash
   git add <list of modified files>
   ```

2. Commit with a conventional commit message:
   ```bash
   git commit -m "<type>(<scope>): <description> [REM-<ID>]"
   ```
   
   **Commit message conventions:**
   - **type:** `fix` for S1/S4, `security` for S2, `refactor` for S3, `perf` for S5, `feat` for S6
   - **scope:** the module/feature name from the remediation task (e.g., `orders`, `auth`, `dashboard`, `user-management`)
   - **description:** concise description of what was fixed
   - **tag:** always include `[REM-<ID>]` at the end
   
   **Examples:**
   ```bash
   git commit -m "fix(orders): allow Purchaser role to create orders [REM-GEN-002]"
   git commit -m "security(auth): enforce role hierarchy in SecurityFilterChain [REM-GEN-001]"
   git commit -m "fix(dashboard): show correct stats for Admin scope [REM-AD-UI-001]"
   git commit -m "feat(reports): implement missing export functionality [REM-PU-003]"
   ```

3. Log the commit hash for the report.

#### Step 1.5 — Update Task Status

Update the remediation task file in-place by appending a `## Resolution` section at the end:

```markdown
## Resolution
- **Status:** FIXED | UNRESOLVED
- **Applied At:** <ISO 8601 timestamp>
- **Attempt:** 1 | 2
- **Commit:** `<commit hash>` — `<commit message>`
- **Files Modified:**
  | File | Change Summary |
  |------|---------------|
  | `<path>` | <what was changed> |
- **Verification Result:** PASS | FAIL
- **Verification Evidence:**
  - <For API: curl output summary>
  - <For UI: screenshot path>
- **Notes:** <Any observations, adjustments made, or issues encountered>
```

For `UNRESOLVED` tasks:
```markdown
## Resolution
- **Status:** UNRESOLVED
- **Applied At:** <ISO 8601 timestamp>
- **Attempts:** 2
- **Reason:** <Why the fix couldn't be applied — what went wrong on both attempts>
- **Suggested Next Steps:** <What a human developer should investigate>
- **Files Reverted:** <List of files that were reverted to their original state>
```

#### Step 1.6 — Scope-Group Test Execution

After completing **all tasks in a scope group** (e.g., all General tasks, or all Admin tasks):

1. Run the test suite for that scope:
   ```bash
   # After all General fixes:
   /testing/execute-test-cases
   
   # After all Admin fixes:
   /testing/execute-test-cases $ROLE=admin
   
   # After all Approver fixes:
   /testing/execute-test-cases $ROLE=approver
   ```

2. Compare results against the baseline (the execution report from before remediation):
   - Count how many previously failing test cases now pass.
   - Check if any previously passing test cases now fail (regression).

3. **If regressions are detected:**
   - Identify which commit(s) caused the regression.
   - Attempt to adjust the fix(es) that caused the regression:
     - Read the regressing test case and its report.
     - Read the source code modified by the recent commits.
     - Apply a targeted adjustment that fixes the regression without undoing the original fix.
     - Re-verify both the original fix AND the regression.
   - If the adjustment works → commit with message: `fix(<scope>): resolve regression in <test> caused by [REM-<ID>]`
   - If the adjustment fails → log the regression, do NOT revert (the original fix is more important), and flag for human review in the report.

4. Log the scope-group test results for the final report.

### Phase 2: Final Regression Sweep

After ALL tasks in the queue have been executed:

1. **Run the complete test suite:**
   ```bash
   /testing/execute-test-cases
   ```

2. **Save the new execution report** — this is the "after" snapshot.

3. **Compare before vs after:**
   - Read the original `/docs/test-cases/execution-report.md` (renamed or backed up before the sweep overwrites it).
   - Before running the sweep, copy the original report:
     ```bash
     cp /docs/test-cases/execution-report.md /docs/test-cases/execution-report-before-remediation.md
     ```
   - After the sweep, the new `/docs/test-cases/execution-report.md` is the "after" snapshot.

### Phase 3: Report Generation

Generate reports at three levels.

#### 3.1 — Update Individual Task Files

Every `REM-*.md` file in scope should already have a `## Resolution` section added during Step 1.5. Verify all are present.

#### 3.2 — Global Remediation Results Report

**File:** `/docs/remediation/remediation-results.md`

```markdown
# Remediation Execution Results

## Execution Metadata
- **Executed At:** <ISO 8601 timestamp>
- **Parameters:** ROLE=<value or "all"> | LAYER=<value> | SEVERITY=<value> | TASK_ID=<value or "N/A">
- **Duration:** <total time>
- **Browser Strategy:** <MCP Browser Agent | Playwright>
- **Docs Read:** AGENTS.md, claude.md, remediation/README.md, <N> task files, <N> source files

## Executive Summary

| Metric | Count |
|--------|-------|
| Tasks in Queue | <N> |
| Successfully Fixed | <N> |
| Unresolved | <N> |
| Skipped (dependency) | <N> |
| Total Commits | <N> |
| Regressions Detected | <N> |
| Regressions Resolved | <N> |
| Regressions Open | <N> |

**Fix Rate:** <pct>%

## Results by Scope

| Scope | Total | Fixed | Unresolved | Skipped | Fix Rate |
|-------|-------|-------|------------|---------|----------|
| General | <N> | <N> | <N> | <N> | <pct>% |
| Super Admin | <N> | <N> | <N> | <N> | <pct>% |
| Admin | <N> | <N> | <N> | <N> | <pct>% |
| Approver | <N> | <N> | <N> | <N> | <pct>% |
| Purchaser | <N> | <N> | <N> | <N> | <pct>% |

## Results by Severity

| Severity | Total | Fixed | Unresolved | Skipped |
|----------|-------|-------|------------|---------|
| S1 — Errors | <N> | <N> | <N> | <N> |
| S2 — Security | <N> | <N> | <N> | <N> |
| S3 — Cross-Layer | <N> | <N> | <N> | <N> |
| S4 — Issues | <N> | <N> | <N> | <N> |
| S5 — Incidents | <N> | <N> | <N> | <N> |
| S6 — Gaps | <N> | <N> | <N> | <N> |

## Task-by-Task Results

| Order | Task ID | Title | Severity | Layer | Status | Attempts | Commit |
|-------|---------|-------|----------|-------|--------|----------|--------|
| 1 | REM-GEN-001 | <title> | S1 | Backend | FIXED | 1 | `abc1234` |
| 2 | REM-GEN-002 | <title> | S2 | Both | FIXED | 2 | `def5678` |
| 3 | REM-AD-001 | <title> | S1 | API | UNRESOLVED | 2 | — |
| 4 | REM-AD-002 | <title> | S4 | UI | SKIPPED | — | — |

## Commit Log

| # | Hash | Message | Files Changed | Task |
|---|------|---------|---------------|------|
| 1 | `abc1234` | fix(auth): enforce role hierarchy [REM-GEN-001] | 2 | REM-GEN-001 |
| 2 | `def5678` | security(orders): restrict endpoint access [REM-GEN-002] | 3 | REM-GEN-002 |

## Unresolved Tasks

<For each UNRESOLVED task, provide a summary for human review:>

### REM-<ID>: <Title>
- **Severity:** <severity>
- **Attempts:** 2
- **Reason:** <Why it couldn't be fixed>
- **Suggested Investigation:** <What a human should look at>
- **Affected Test Cases:** <list>

## Regressions

### Detected During Execution
| Regression | Caused By | Test Case | Status | Resolution Commit |
|-----------|-----------|-----------|--------|-------------------|
| <desc> | REM-<ID> | TC-<ID> | Resolved / Open | `hash` / — |

### Open Regressions (Require Human Review)
<Details on any regressions that could not be resolved>
```

#### 3.3 — Before/After Comparison Report

**File:** `/docs/remediation/before-after-comparison.md`

```markdown
# Before/After Test Results Comparison

## Baseline
- **Before Report:** `/docs/test-cases/execution-report-before-remediation.md`
- **Before Timestamp:** <original execution timestamp>

## After Remediation
- **After Report:** `/docs/test-cases/execution-report.md`
- **After Timestamp:** <regression sweep timestamp>

## Overall Improvement

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Total Test Cases | <N> | <N> | — |
| Passing | <N> | <N> | +<N> |
| Failing | <N> | <N> | -<N> |
| Blocked | <N> | <N> | -<N> |
| Skipped | <N> | <N> | -<N> |
| Pass Rate | <pct>% | <pct>% | +<pct>% |

## Improvement by Role

| Role | Before Pass Rate | After Pass Rate | Change | Tests Fixed |
|------|-----------------|-----------------|--------|-------------|
| Super Admin | <pct>% | <pct>% | +<pct>% | <N> |
| Admin | <pct>% | <pct>% | +<pct>% | <N> |
| Approver | <pct>% | <pct>% | +<pct>% | <N> |
| Purchaser | <pct>% | <pct>% | +<pct>% | <N> |

## Improvement by Layer

| Layer | Before Pass Rate | After Pass Rate | Change |
|-------|-----------------|-----------------|--------|
| UI (Frontend) | <pct>% | <pct>% | +<pct>% |
| API (Backend) | <pct>% | <pct>% | +<pct>% |

## Test Cases Fixed (were FAIL, now PASS)

| Test Case ID | Role | Layer | Fixed By Task |
|-------------|------|-------|---------------|
| TC-<ID> | <role> | UI/API | REM-<ID> |
| TC-<ID> | <role> | UI/API | REM-<ID> |

## Test Cases Still Failing

| Test Case ID | Role | Layer | Related Task | Task Status |
|-------------|------|-------|-------------|-------------|
| TC-<ID> | <role> | UI/API | REM-<ID> | UNRESOLVED |
| TC-<ID> | <role> | UI/API | — | No remediation task existed |

## New Failures (Regressions)

| Test Case ID | Role | Layer | Was | Now | Likely Cause |
|-------------|------|-------|-----|-----|-------------|
| TC-<ID> | <role> | UI/API | PASS | FAIL | REM-<ID> commit `hash` |

## Cross-Layer Discrepancies Resolved

| Feature | Before | After |
|---------|--------|-------|
| <feature> | UI PASS / API FAIL | UI PASS / API PASS |
| <feature> | UI FAIL / API PASS | UI PASS / API PASS |

## Cross-Layer Discrepancies Remaining

| Feature | UI Status | API Status | Related Tasks |
|---------|-----------|------------|---------------|
| <feature> | PASS | FAIL | REM-<ID> (UNRESOLVED) |

## Recommendations for Next Iteration
- <Unresolved tasks that need human investigation>
- <Regressions that need manual resolution>
- <Test cases still failing with no remediation task — may need new test analysis>
- <Areas where additional test coverage is needed>
- <Suggested next run: `/testing/generate-remediation-plan` to create a plan for remaining issues>
```

### Phase 4: Cleanup & Final Summary

1. **Verify all task files** have a `## Resolution` section.

2. **Update the remediation master README** (`/docs/remediation/README.md`):
   - Add a `## Execution History` section at the end:
     ```markdown
     ## Execution History
     
     | Run | Timestamp | Tasks | Fixed | Unresolved | Pass Rate Change |
     |-----|-----------|-------|-------|------------|-----------------|
     | 1 | <timestamp> | <N> | <N> | <N> | <before>% → <after>% |
     ```

3. **Update the Files Modified Tracker** in the master README with actual modification data from all commits.

4. **Print terminal summary:**
   ```
   ============================================
   REMEDIATION EXECUTION COMPLETE
   ============================================
   Tasks: <N> total | <N> fixed | <N> unresolved | <N> skipped
   Fix Rate: <pct>%
   Commits: <N>
   
   Test Results:
     Before: <N> pass / <N> fail (<pct>%)
     After:  <N> pass / <N> fail (<pct>%)
     Improvement: +<N> tests passing (+<pct>%)
   
   Regressions: <N> detected | <N> resolved | <N> open
   
   Reports:
     Results:    /docs/remediation/remediation-results.md
     Comparison: /docs/remediation/before-after-comparison.md
     Before:     /docs/test-cases/execution-report-before-remediation.md
     After:      /docs/test-cases/execution-report.md
   
   Next Steps:
     <If unresolved > 0: "Review unresolved tasks in remediation-results.md">
     <If regressions > 0: "Review open regressions in remediation-results.md">
     <If still failing > 0: "Run /testing/generate-remediation-plan for remaining issues">
     <If all pass: "All test cases passing! No further action needed.">
   ============================================
   ```

## Important Notes

- **ALWAYS read AGENTS.md, claude.md, AND the remediation master README before touching any code.** The execution order and dependency map are critical. Fixing tasks out of order wastes effort and can cause cascading failures.
- **Surgical code changes only.** Do not rewrite entire files. Use targeted edits (str_replace or equivalent). The smaller the change, the lower the regression risk.
- **Follow project conventions religiously.** Every fix must match the existing code style, architecture patterns, and naming conventions from `AGENTS.md`. A fix that works but breaks conventions creates technical debt.
- **Backend and frontend have different dev loops:**
  - Backend: modify → rebuild/restart (if no DevTools) → wait for health check → verify via curl.
  - Frontend: modify → wait for hot reload (~3-5 sec) → verify via browser. Config changes may need full restart.
  - Cross-layer: fix backend first → verify backend → fix frontend → verify frontend → verify both together.
- **Retry is limited to 2 attempts.** If the fix doesn't work after two tries, revert and mark as UNRESOLVED. Do not get stuck in retry loops. A human should investigate.
- **Regressions take priority.** If a scope-group test execution reveals a regression, attempt to resolve it immediately before moving to the next scope group. The goal is to never leave the codebase in a worse state than before.
- **Git commits are atomic per fix.** One commit per successfully fixed task. This allows easy reversion if a regression is discovered later. Never squash multiple fixes into one commit.
- **Do not modify test case files or remediation plan files** (except appending `## Resolution` sections). The remediation fixes the application, not the tests or the plan.
- **If the remediation plan references files that no longer exist** (e.g., another fix renamed or moved them), adapt the fix to the current file structure. Log the adaptation in the Resolution notes.
- **Token management for API verification:** Obtain fresh tokens before verification. Tokens from Phase 0 may have expired during long remediation runs.
- **If a service crashes during remediation:** Restart it using the commands from `AGENTS.md`, wait for health check, then continue. Log the crash in the task's Resolution notes.
- **The before/after comparison is the ground truth.** The final regression sweep execution report is what matters. Individual inline verifications are smoke tests; the full test suite is the definitive validation.
