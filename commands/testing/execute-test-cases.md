# Execute Test Cases

## Objective

Open a browser (via MCP Browser Agent or Playwright) to execute UI test cases and use `curl` plus standard Linux tools to execute API test cases. Authenticate using `credentials.md`, perform all interactions, capture evidence screenshots at key checkpoints, and produce detailed reports at individual, role, and global levels.

## Parameters

This command supports flexible execution scopes via parameters:

- **`$ROLE`** (optional): Execute all test cases for a specific role. Values: `super-admin`, `admin`, `approver`, `purchaser`. If omitted and no `$TEST_CASE_ID` is provided, all roles are executed.
- **`$TEST_CASE_ID`** (optional): Execute a single test case by ID (e.g., `TC-SA-001`). Takes precedence over `$ROLE`.
- **`$ON_FAILURE`** (optional, default: `skip-deps`): Behavior on failure. Values: `stop`, `continue`, `skip-deps`.

**Examples:**
- `/testing/execute-test-cases` → runs all test cases for all roles
- `/testing/execute-test-cases $ROLE=approver` → runs all Approver test cases
- `/testing/execute-test-cases $TEST_CASE_ID=TC-PU-003` → runs a single test case
- `/testing/execute-test-cases $ROLE=admin $ON_FAILURE=stop` → runs Admin tests, stops on first failure

## Instructions

### Phase 0: Environment Setup & Validation

1. **Verify prerequisites:**

   **For UI testing (browser-based):** Detect the best available browser agent in this order of preference:
   - **Option A — MCP Browser Agent:** Check if an MCP browser tool is available in the current session (e.g., `computer use`, `browser_use`, or any connected browser MCP server). This is the preferred option as it allows the agent to see and interact with the browser natively.
   - **Option B — Playwright:** If no MCP browser agent is available, check for Playwright. If not installed, install it: `npm install playwright && npx playwright install chromium`.
   - Store the chosen browser strategy (`mcp-browser` or `playwright`) and use it consistently for all UI test execution.

   **For API testing (terminal-based):** Verify the following Linux tools are available:
   - `curl` — primary HTTP client for API requests.
   - `jq` — for JSON response parsing and assertion validation. If not available, install it: `sudo apt-get install -y jq` or fall back to `grep`/`sed`/`awk` for JSON parsing.
   - Any other standard Linux tools (`grep`, `sed`, `awk`, `diff`, `wc`, `head`, `tail`) can be used as needed for response validation, data extraction, and assertion checking.

   **General:**
   - Confirm `/docs/test-cases/` exists and contains generated test cases. If empty, abort and instruct the user to run `/testing/generate-test-cases` first.

2. **Read configuration:**
   - **Frontend URL:** `http://localhost:3000` (Next.js)
   - **Backend URL:** `http://localhost:8080` (Spring Boot API)
   - If these are defined in `.env`, `application.yml`, or any project config, read from there instead.
   - Verify both services are reachable (quick HTTP GET to each base URL). Report which services are up/down before proceeding.

3. **Read credentials:**
   - Locate and parse `credentials.md` — extract the table: `| Role | Email | Password | Scope |`.
   - Build an internal credentials map:
     ```
     super-admin → { email: superadmin@acme.com, password: password123, scope: "Platform-wide (all organizations)" }
     admin       → { email: admin@acme.com, password: password123, scope: "Organization-level (Acme Corp)" }
     approver    → { email: approver@acme.com, password: password123, scope: "Approvals & Budget (HQ, West Region)" }
     purchaser   → { email: purchaser@acme.com, password: password123, scope: "Ordering (HQ branch)" }
     ```
   - This map is used for both browser login and API authentication throughout execution.

4. **Resolve execution scope:**
   - If `$TEST_CASE_ID` is provided → find the specific test case file, identify its role, execute only that one.
   - If `$ROLE` is provided → collect all `TC-*.md` files in that role's folder, ordered by ID.
   - If neither → collect all `TC-*.md` files across all role folders, ordered by role (super-admin → admin → approver → purchaser), then by ID within each role.
   - Parse each test case file to extract: metadata, preconditions, Given/When/Then steps, negative scenarios, API references, UI references, and cross-role dependencies.

5. **Create output directories:**
   - For each role being executed, ensure a `/docs/test-cases/<role>/screenshots/` folder exists.
   - Prepare report file paths (details in Phase 4).

### Phase 1: Authentication Strategy

For each role that will be tested in this execution run:

**UI Authentication (for test cases with UI References):**

Depending on the browser strategy detected in Phase 0:

**If using MCP Browser Agent:**
1. Open the browser to the login page URL (e.g., `http://localhost:3000/login`).
2. Visually identify the email and password input fields on the page.
3. Type the email and password from the credentials map for the current role.
4. Click the login/submit button.
5. Wait for the page to navigate to the post-login state (dashboard, home, etc.).
6. **Checkpoint screenshot:** Capture the screen as `screenshots/<ROLE>/auth-login-success.png`.
7. The MCP browser session remains active for reuse across all UI test cases for this role.
8. If login fails, mark ALL test cases for this role as `BLOCKED` with reason "Authentication failed" and proceed to the next role.

**If using Playwright:**
1. Launch Playwright browser (Chromium, headed or headless based on environment).
2. Navigate to the login page (e.g., `http://localhost:3000/login` or `/auth/login` — detect the actual login route from the codebase).
3. Fill in the email and password from the credentials map for the current role.
4. Submit the login form.
5. Wait for successful redirect (dashboard, home, or any post-login route).
6. **Checkpoint screenshot:** Capture `screenshots/<ROLE>/auth-login-success.png`.
7. Store the browser session/context for reuse across all UI test cases for this role.
8. If login fails, mark ALL test cases for this role as `BLOCKED` with reason "Authentication failed" and proceed to the next role.

**API Authentication (for test cases with API References):**
1. Execute a `curl` request to the auth/login endpoint and extract the token using `jq`:
   ```bash
   TOKEN=$(curl -s -X POST http://localhost:8080/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email": "<email>", "password": "<password>"}' \
     | jq -r '.token // .access_token // .data.token')
   ```
2. Validate the token is not null/empty. If `jq` is not available, fall back to `grep -oP` for extraction.
3. Store the token for reuse across all API test cases for this role.
4. If auth fails, mark all API test cases for this role as `BLOCKED`.

> **Important:** Detect the actual auth endpoint from the codebase (controllers, security config). The paths above are examples — adapt to the project's actual routes.

### Phase 2: Test Case Execution

Execute each test case following this workflow:

#### 2.1 — UI Test Case Execution (MCP Browser Agent or Playwright)

For each test case that has a **UI Reference** section, use the browser strategy detected in Phase 0:

**If using MCP Browser Agent:**

1. **Pre-action screenshot:** Navigate to the target page/route. Wait for the page to fully render. Capture the screen:
   `screenshots/<ROLE>/TC-<ID>-01-before.png`

2. **Execute steps:** For each step in the **When** section:
   - Visually identify the target elements on the page (buttons, inputs, dropdowns, links, tables, etc.).
   - Perform the interaction: click, type, select, scroll, drag, upload, etc.
   - Wait for the page to update after each interaction (new content visible, loading spinners gone, navigation complete).
   - If a step involves a form submission or destructive action, capture a **mid-action screenshot**:
     `screenshots/<ROLE>/TC-<ID>-02-action-<step-number>.png`

3. **Post-action screenshot:** After all steps are completed, capture the final state:
   `screenshots/<ROLE>/TC-<ID>-03-after.png`

4. **Validate assertions:** For each assertion in the **Then** section:
   - Visually verify text, element presence, URL changes, toast/notification messages, table data, disabled/enabled states, redirects, etc.
   - Record each assertion as `PASS` or `FAIL` with the actual vs. expected values.

5. **Execute negative scenarios:** For each negative scenario in the test case:
   - Perform the unauthorized/invalid action.
   - Verify the expected error or denial is shown on screen.
   - Capture screenshot:
     `screenshots/<ROLE>/TC-<ID>-04-negative-<scenario-number>.png`

**If using Playwright:**

1. **Pre-action screenshot:** Navigate to the target page/route. Wait for page load. Capture screenshot:
   `screenshots/<ROLE>/TC-<ID>-01-before.png`

2. **Execute steps:** For each step in the **When** section:
   - Perform the browser interaction (click, type, select, navigate, drag, upload, etc.) using Playwright selectors.
   - Use Playwright's built-in waits (`waitForSelector`, `waitForNavigation`, `waitForResponse`) to handle async UI updates.
   - If a step involves a form submission or destructive action, capture a **mid-action screenshot**:
     `screenshots/<ROLE>/TC-<ID>-02-action-<step-number>.png`

3. **Post-action screenshot:** After all steps are completed, capture the final state:
   `screenshots/<ROLE>/TC-<ID>-03-after.png`

4. **Validate assertions:** For each assertion in the **Then** section:
   - Check visible text, element presence, URL changes, toast/notification messages, table data, disabled/enabled states, redirects, etc.
   - Record each assertion as `PASS` or `FAIL` with the actual vs. expected values.

5. **Execute negative scenarios:** For each negative scenario in the test case:
   - Perform the unauthorized/invalid action.
   - Verify the expected error or denial is shown.
   - Capture screenshot:
     `screenshots/<ROLE>/TC-<ID>-04-negative-<scenario-number>.png`

**Common to both strategies — On failure:**
   - Capture an **error screenshot:** `screenshots/<ROLE>/TC-<ID>-ERROR.png`
   - Record the failure details (step that failed, actual behavior, expected behavior, console errors if any).
   - If `$ON_FAILURE=skip-deps`: flag any test cases that list this one as a dependency as `SKIPPED (dependency TC-<ID> failed)`.
   - If `$ON_FAILURE=stop`: halt execution and proceed directly to Phase 4 (Report Generation).
   - If `$ON_FAILURE=continue`: log the failure and proceed to the next test case.

#### 2.2 — API Test Case Execution (curl + Linux tools)

For each test case that has an **API Reference** section, use `curl` and standard Linux tools exclusively (no browser).

1. **Build the curl command** from the API Reference:
   ```bash
   curl -s -w "\n%{http_code}\n%{time_total}" -X <METHOD> http://localhost:8080<ENDPOINT> \
     -H "Authorization: Bearer <TOKEN>" \
     -H "Content-Type: application/json" \
     -d '<REQUEST_BODY if applicable>' \
     -o /tmp/tc-<ID>-response.json
   ```

2. **Execute the curl command** and capture:
   - HTTP status code (from `-w "%{http_code}"`)
   - Response body saved to a temp file
   - Response time (from `-w "%{time_total}"`)
   - Response headers if needed: add `-D /tmp/tc-<ID>-headers.txt`

3. **Validate assertions using Linux tools:**
   - **Status code:** Compare HTTP status code against **Expected Status** using simple bash comparison.
   - **JSON response validation with `jq`:**
     ```bash
     # Check a specific field value
     jq -r '.data.status' /tmp/tc-<ID>-response.json
     
     # Verify array length
     jq '.items | length' /tmp/tc-<ID>-response.json
     
     # Check field existence
     jq 'has("id")' /tmp/tc-<ID>-response.json
     
     # Extract nested values
     jq -r '.data.order.totalAmount' /tmp/tc-<ID>-response.json
     ```
   - **Text matching with `grep`/`sed`/`awk`:**
     ```bash
     # Verify response contains expected string
     grep -q "success" /tmp/tc-<ID>-response.json
     
     # Extract values without jq
     grep -oP '"id"\s*:\s*"\K[^"]+' /tmp/tc-<ID>-response.json
     ```
   - **Data comparison with `diff`:**
     ```bash
     # Compare response structure against expected template
     jq -S 'keys' /tmp/tc-<ID>-response.json > /tmp/tc-<ID>-actual-keys.txt
     diff /tmp/tc-<ID>-expected-keys.txt /tmp/tc-<ID>-actual-keys.txt
     ```
   - **Chain validation for workflows:** Use `xargs`, pipes, and variable substitution to extract values from one response and feed them into the next `curl` call:
     ```bash
     # Extract order ID and use in next request
     ORDER_ID=$(jq -r '.data.id' /tmp/tc-<ID>-response.json)
     curl -s -X GET http://localhost:8080/api/v1/orders/$ORDER_ID \
       -H "Authorization: Bearer <TOKEN>"
     ```
   - For negative scenarios: verify `403 Forbidden`, `401 Unauthorized`, `400 Bad Request`, etc.
   - Record each assertion as `PASS` or `FAIL`.

4. **Log the full curl command and response** in the report (sanitize the password, keep the token truncated for readability). Include the `jq`/`grep` commands used for validation.

5. **Clean up temp files** after each test case: `rm -f /tmp/tc-<ID>-*.json /tmp/tc-<ID>-*.txt`

6. **On failure:** Apply same `$ON_FAILURE` behavior as UI tests.

#### 2.3 — Cross-Role Workflow Execution

For test cases with cross-role dependencies (noted in `Notes` section):

1. **Execute the workflow in role sequence.** For example:
   - Login as **Purchaser** → Execute TC-PU-005 (create purchase order) → Capture order ID from the UI/API response.
   - Logout / Switch session.
   - Login as **Approver** → Execute TC-AP-003 (approve purchase order) → Use the order ID from the previous step.

2. **Pass data between steps:** Store intermediate values (IDs, reference numbers, URLs) generated by one role's test case for use in the next role's test case.

3. **Screenshot at each role switch:** Capture `screenshots/<ROLE>/TC-<ID>-cross-role-handoff.png`.

4. **If any step in the chain fails:** Mark all subsequent steps in the chain as `SKIPPED (cross-role dependency failed at TC-<ID>)`.

### Phase 3: Evidence & Screenshot Management

All screenshots are stored under `/docs/test-cases/<role>/screenshots/` with the following naming convention:

| Screenshot Type | Pattern | When Captured |
|---|---|---|
| Auth success | `auth-login-success.png` | After successful login |
| Pre-action | `TC-<ID>-01-before.png` | Before executing steps |
| Mid-action | `TC-<ID>-02-action-<N>.png` | During key interactions |
| Post-action | `TC-<ID>-03-after.png` | After all steps complete |
| Negative scenario | `TC-<ID>-04-negative-<N>.png` | During negative validations |
| Error/failure | `TC-<ID>-ERROR.png` | On any unexpected failure |
| Cross-role handoff | `TC-<ID>-cross-role-handoff.png` | When switching roles in workflows |

**All screenshots MUST be referenced in the corresponding report files using relative paths** (see Phase 4).

### Phase 4: Report Generation

Generate reports at three levels:

#### 4.1 — Individual Test Case Report

**File:** `/docs/test-cases/<role>/TC-<ID>-report.md`

One report per executed test case. Placed alongside the test case file.

```markdown
# Test Report: TC-<ID> — <Test Case Title>

## Summary
- **Status:** PASS | FAIL | BLOCKED | SKIPPED
- **Role:** <Role Name>
- **Executed At:** <ISO 8601 timestamp>
- **Duration:** <execution time in seconds>
- **Environment:** Frontend: http://localhost:3000 | Backend: http://localhost:8080

## Results

### UI Validation
| Step | Action | Expected | Actual | Status | Screenshot |
|------|--------|----------|--------|--------|------------|
| 1    | <action> | <expected> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-02-action-1.png) |
| 2    | <action> | <expected> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-02-action-2.png) |

**Before State:** [screenshot](screenshots/TC-<ID>-01-before.png)
**After State:** [screenshot](screenshots/TC-<ID>-03-after.png)

### API Validation
| Endpoint | Method | Expected Status | Actual Status | Status | Response Time |
|----------|--------|-----------------|---------------|--------|---------------|
| <path>   | <GET/POST/...> | <200> | <actual> | PASS/FAIL | <ms> |

<details>
<summary>curl command</summary>

\```bash
curl -s -X <METHOD> http://localhost:8080<ENDPOINT> \
  -H "Authorization: Bearer <TOKEN_TRUNCATED>..." \
  -H "Content-Type: application/json"
\```

</details>

<details>
<summary>Response body</summary>

\```json
{ ... }
\```

</details>

### Negative Scenarios
| Scenario | Action | Expected | Actual | Status | Screenshot |
|----------|--------|----------|--------|--------|------------|
| <desc>   | <action> | <error/denial> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-04-negative-1.png) |

## Errors
- **Error 1:** <Description of what went wrong>
  - **Step:** <Which step failed>
  - **Expected:** <What should have happened>
  - **Actual:** <What actually happened>
  - **Screenshot:** [error](screenshots/TC-<ID>-ERROR.png)
  - **Console Errors:** <Any browser console errors captured>

## Incidents
- <Any unexpected behavior that didn't cause a failure but is worth noting>
- <E.g., slow response time, UI glitch that self-corrected, deprecation warnings>

## Issues
- <Confirmed defects found during execution>
- <E.g., "Save button remains disabled after valid form input">

## Gaps
- <Missing functionality, untestable steps, or areas where the test case couldn't be fully validated>
- <E.g., "Email notification could not be verified — no mailbox integration available">
- <E.g., "Test case references a 'Reports' page that does not exist yet">
```

#### 4.2 — Role Summary Report

**File:** `/docs/test-cases/<role>/report.md`

One report per role aggregating all test case results.

```markdown
# Test Execution Report: <Role Name>

## Overview
- **Role:** <Role Name>
- **Email:** <email>
- **Scope:** <scope>
- **Executed At:** <ISO 8601 timestamp>
- **Total Duration:** <total time>

## Summary

| Metric | Count |
|--------|-------|
| Total Test Cases | <N> |
| Passed | <N> |
| Failed | <N> |
| Blocked | <N> |
| Skipped | <N> |

**Pass Rate:** <percentage>%

## Results by Test Case

| ID | Title | Priority | Type | Status | Duration | Report |
|----|-------|----------|------|--------|----------|--------|
| TC-<ID> | <title> | <priority> | <type> | PASS/FAIL | <sec> | [report](TC-<ID>-report.md) |
| TC-<ID> | <title> | <priority> | <type> | FAIL | <sec> | [report](TC-<ID>-report.md) |

## All Errors
<Aggregated list of all errors across test cases for this role, with links to individual reports>

## All Incidents
<Aggregated list of all incidents>

## All Issues
<Aggregated list of all confirmed defects>

## All Gaps
<Aggregated list of all gaps and missing functionality>

## Cross-Role Workflows
| Workflow | Roles Involved | Status | Notes |
|----------|---------------|--------|-------|
| <workflow name> | Purchaser → Approver | PASS/FAIL | <details> |
```

#### 4.3 — Global Execution Report

**File:** `/docs/test-cases/execution-report.md`

Single global report for the entire execution run.

```markdown
# Global Test Execution Report

## Execution Metadata
- **Executed At:** <ISO 8601 timestamp>
- **Parameters:** ROLE=<value or "all"> | TEST_CASE_ID=<value or "N/A"> | ON_FAILURE=<value>
- **Environment:** Frontend: http://localhost:3000 | Backend: http://localhost:8080
- **Total Duration:** <total time>

## Global Summary

| Role | Total | Passed | Failed | Blocked | Skipped | Pass Rate |
|------|-------|--------|--------|---------|---------|-----------|
| Super Admin | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Admin | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Approver | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Purchaser | <N> | <N> | <N> | <N> | <N> | <pct>% |
| **Total** | **<N>** | **<N>** | **<N>** | **<N>** | **<N>** | **<pct>%** |

## Critical Failures
<List of all FAIL results with Critical or High priority, with links to individual reports>

## All Errors (aggregated)
<Numbered list of unique errors across all roles>

## All Issues (aggregated)
<Numbered list of confirmed defects across all roles>

## All Incidents (aggregated)
<Numbered list of incidents across all roles>

## All Gaps (aggregated)
<Numbered list of gaps across all roles>

## Cross-Role Workflow Summary
| Workflow | Roles | Status | Link |
|----------|-------|--------|------|
| <name> | <roles> | PASS/FAIL | [details](<role>/TC-<ID>-report.md) |

## Recommendations
<Based on the execution results, provide recommendations:>
- <Defects to prioritize>
- <Missing features flagged as gaps>
- <Test cases that need refinement>
- <Areas that need more test coverage>

## Role-Level Reports
- [Super Admin Report](super-admin/report.md)
- [Admin Report](admin/report.md)
- [Approver Report](approver/report.md)
- [Purchaser Report](purchaser/report.md)
```

### Phase 5: Post-Execution Cleanup

1. **Close all browser sessions:** If using Playwright, close all browser contexts and instances. If using MCP Browser Agent, close/navigate away from the application.
2. **Clean up temp files:** Remove all `/tmp/tc-*` files created during API test execution.
3. **Verify all report files** were written successfully.
3. **Verify all screenshots** referenced in reports actually exist on disk.
4. **Print a terminal summary:**
   ```
   ============================================
   TEST EXECUTION COMPLETE
   ============================================
   Total: <N> | Pass: <N> | Fail: <N> | Blocked: <N> | Skipped: <N>
   Pass Rate: <pct>%
   
   Reports:
     Global:     /docs/test-cases/execution-report.md
     Super Admin: /docs/test-cases/super-admin/report.md (<N> tests)
     Admin:       /docs/test-cases/admin/report.md (<N> tests)
     Approver:    /docs/test-cases/approver/report.md (<N> tests)
     Purchaser:   /docs/test-cases/purchaser/report.md (<N> tests)
   ============================================
   ```

## Important Notes

- **Browser strategy selection is automatic.** The command detects available tools in order: MCP Browser Agent first, Playwright as fallback. Once chosen, the same strategy is used for all UI tests in the run. Log which strategy was selected at the start of execution.
- **API tests are terminal-only.** All API testing uses `curl`, `jq`, `grep`, `sed`, `awk`, `diff`, and any other standard Linux tools. Never open a browser for API tests.
- **Never hardcode passwords in reports.** Reference the role and email, mask passwords as `*****`, and point to `credentials.md`.
- **Truncate JWT tokens in reports** to the first 20 characters followed by `...` for readability and security.
- **curl commands in reports** must sanitize credentials: replace the password in the auth curl with `*****` and truncate Bearer tokens.
- **Screenshots are relative paths** in all reports — do NOT use absolute filesystem paths.
- **If the application is not running** (frontend or backend unreachable), abort execution with a clear error message listing which service is down and how to start it.
- **If a test case file cannot be parsed**, skip it, log a warning in the role report, and continue.
- **Cross-role workflows** are only executed when running in "all roles" mode or when both involved roles are included in the `$ROLE` parameter scope. Otherwise, execute only the current role's part and note the dependency as incomplete.
- **Retry logic:** If a UI interaction fails due to a transient issue (element not found, timeout), retry once with a 3-second wait before marking as failed. If a `curl` call fails with a connection error, retry once after 2 seconds.
