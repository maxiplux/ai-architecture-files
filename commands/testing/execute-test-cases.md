# Execute Test Cases

## Objective

Read `AGENTS.md`, `claude.md`, and project documentation to understand the project context. Then execute test cases from `/docs/test-cases/` using two distinct strategies: **browser agent (MCP or Playwright) for UI test cases** and **`curl` + Linux tools for API test cases**. Authenticate using `credentials.md`, perform all interactions, capture evidence screenshots, and produce detailed reports at individual, role, and global levels.

## Parameters

This command supports flexible execution scopes via parameters:

- **`$ROLE`** (optional): Execute all test cases for a specific role. Values: `super-admin`, `admin`, `approver`, `purchaser`. If omitted and no `$TEST_CASE_ID` is provided, all roles are executed.
- **`$TEST_CASE_ID`** (optional): Execute a single test case by ID (e.g., `TC-SA-UI-001` or `TC-PU-API-003`). Takes precedence over `$ROLE`.
- **`$LAYER`** (optional): Filter by layer. Values: `ui`, `api`, `both` (default: `both`). Useful for running only frontend or only backend tests.
- **`$ON_FAILURE`** (optional, default: `skip-deps`): Behavior on failure. Values: `stop`, `continue`, `skip-deps`.

**Examples:**
- `/testing/execute-test-cases` → runs all UI + API test cases for all roles
- `/testing/execute-test-cases $ROLE=approver` → runs all Approver test cases (UI + API)
- `/testing/execute-test-cases $ROLE=admin $LAYER=ui` → runs only Admin UI test cases
- `/testing/execute-test-cases $ROLE=purchaser $LAYER=api` → runs only Purchaser API test cases
- `/testing/execute-test-cases $TEST_CASE_ID=TC-PU-UI-003` → runs a single UI test case
- `/testing/execute-test-cases $TEST_CASE_ID=TC-AD-API-005` → runs a single API test case
- `/testing/execute-test-cases $ROLE=admin $ON_FAILURE=stop` → runs all Admin tests, stops on first failure

## Instructions

### Phase 0: Project Intelligence & Environment Setup

**This phase is mandatory and must be completed before any execution.**

1. **Read agent/project documentation first:**
   - Search for and read the following files (check root, `.claude/`, `/docs/`, and any subdirectory):
     - `AGENTS.md` — understand development patterns, architecture, conventions, folder structure, start commands.
     - `claude.md` or `.claude/claude.md` or `CLAUDE.md` — project-specific instructions, preferences, constraints.
     - `README.md` — setup instructions, tech stack, how to run the project.
   - **Extract critical execution context:**
     - How to start the backend (e.g., `./mvnw spring-boot:run`, `gradle bootRun`, `docker compose up`)
     - How to start the frontend (e.g., `npm run dev`, `pnpm dev`, `yarn dev`)
     - Frontend and backend URLs (ports, base paths)
     - Auth flow specifics (login endpoint, token format, cookie names)
     - Any test-specific instructions or environment setup
     - Known quirks, workarounds, or environmental constraints

2. **Verify prerequisites:**

   **For UI testing (browser-based):** Detect the best available browser agent in this order of preference:
   - **Option A — MCP Browser Agent:** Check if an MCP browser tool is available in the current session (e.g., `computer use`, `browser_use`, or any connected browser MCP server). This is the preferred option as it allows the agent to see and interact with the browser natively.
   - **Option B — Playwright:** If no MCP browser agent is available, check for Playwright. If not installed, install it: `npm install playwright && npx playwright install chromium`.
   - Store the chosen browser strategy (`mcp-browser` or `playwright`) and use it consistently for all UI test execution.

   **For API testing (terminal-based):** Verify the following Linux tools are available:
   - `curl` — primary HTTP client for API requests.
   - `jq` — for JSON response parsing and assertion validation. If not available, install it: `sudo apt-get install -y jq` or fall back to `grep`/`sed`/`awk` for JSON parsing.
   - Any other standard Linux tools (`grep`, `sed`, `awk`, `diff`, `wc`, `head`, `tail`, `xargs`, `sort`, `uniq`) can be used as needed for response validation, data extraction, and assertion checking.

   **General:**
   - Confirm `/docs/test-cases/` exists and contains generated test cases. If empty, abort and instruct the user to run `/testing/generate-test-cases` first.

3. **Read configuration (auto-detect from codebase and docs):**
   - **Frontend URL:** Detect from `AGENTS.md`, `README.md`, `next.config.*`, `.env*`, `package.json` scripts. Default: `http://localhost:3000`.
   - **Backend URL:** Detect from `AGENTS.md`, `README.md`, `application.yml`, `application.properties`, `.env*`. Default: `http://localhost:8080`.
   - Verify both services are reachable:
     ```bash
     # Check frontend
     curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
     # Check backend
     curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/actuator/health
     ```
   - Report which services are up/down before proceeding.
   - **If only one service is up:** Allow execution of the available layer only (e.g., if backend is up but frontend is down, execute only API test cases and mark all UI test cases as `BLOCKED: Frontend unreachable`).

4. **Read credentials:**
   - Locate and parse `credentials.md` — extract the table: `| Role | Email | Password | Scope |`.
   - Build an internal credentials map:
     ```
     super-admin → { email: superadmin@acme.com, password: password123, scope: "Platform-wide (all organizations)" }
     admin       → { email: admin@acme.com, password: password123, scope: "Organization-level (Acme Corp)" }
     approver    → { email: approver@acme.com, password: password123, scope: "Approvals & Budget (HQ, West Region)" }
     purchaser   → { email: purchaser@acme.com, password: password123, scope: "Ordering (HQ branch)" }
     ```
   - This map is used for both browser login and API authentication throughout execution.

5. **Resolve execution scope:**
   - If `$TEST_CASE_ID` is provided → find the specific test case file, identify its role and layer (UI or API), execute only that one.
   - If `$ROLE` is provided → collect all test case files in that role's folder, filtered by `$LAYER`.
   - If neither → collect all test case files across all role folders, filtered by `$LAYER`.
   - **Separate the collected test cases into two queues:**
     - `UI queue`: all `TC-*-UI-*.md` files (executed via browser)
     - `API queue`: all `TC-*-API-*.md` files (executed via curl + Linux tools)
   - Parse each test case file to extract: metadata, preconditions, Given/When/Then steps, negative scenarios, UI details or API details, cross-layer links, cross-role dependencies.

6. **Create output directories:**
   - For each role being executed, ensure a `/docs/test-cases/<role>/screenshots/` folder exists.
   - Prepare report file paths (details in Phase 4).

### Phase 1: Authentication Strategy

For each role that will be tested in this execution run:

#### 1.1 — UI Authentication (for UI queue test cases)

Depending on the browser strategy detected in Phase 0:

**If using MCP Browser Agent:**
1. Open the browser to the frontend login page URL (auto-detected from codebase, e.g., `http://localhost:3000/login`).
2. Visually identify the email and password input fields on the page.
3. Type the email and password from the credentials map for the current role.
4. Click the login/submit button.
5. Wait for the page to navigate to the post-login state (dashboard, home, etc.).
6. **Checkpoint screenshot:** Capture the screen as `screenshots/<ROLE>/auth-login-success.png`.
7. The MCP browser session remains active for reuse across all UI test cases for this role.
8. If login fails, mark ALL UI test cases for this role as `BLOCKED` with reason "UI Authentication failed" and proceed to API tests or next role.

**If using Playwright:**
1. Launch Playwright browser (Chromium, headed or headless based on environment).
2. Navigate to the frontend login page (auto-detected from codebase).
3. Fill in the email and password from the credentials map for the current role.
4. Submit the login form.
5. Wait for successful redirect (dashboard, home, or any post-login route).
6. **Checkpoint screenshot:** Capture `screenshots/<ROLE>/auth-login-success.png`.
7. Store the browser session/context for reuse across all UI test cases for this role.
8. If login fails, mark ALL UI test cases for this role as `BLOCKED` with reason "UI Authentication failed" and proceed to API tests or next role.

#### 1.2 — API Authentication (for API queue test cases)

1. Detect the auth endpoint from the codebase (controllers, security config, `AGENTS.md`).
2. Execute a `curl` request to the auth endpoint and extract the token using `jq`:
   ```bash
   TOKEN=$(curl -s -X POST http://localhost:8080/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email": "<email>", "password": "<password>"}' \
     | jq -r '.token // .access_token // .data.token // .data.access_token')
   ```
3. Validate the token is not null/empty. If `jq` is not available, fall back to `grep -oP` for extraction.
4. Store the token for reuse across all API test cases for this role.
5. If auth fails, mark all API test cases for this role as `BLOCKED` with reason "API Authentication failed".

> **Important:** The auth endpoint path and request/response format above are examples. Always detect the actual paths and formats from the codebase and `AGENTS.md`.

### Phase 2: Test Case Execution

Execute test cases from the two queues. **Within each role, execute UI test cases first, then API test cases** (UI first because some API tests may depend on data created via UI, and UI tests validate the user experience first).

#### 2.1 — UI Test Case Execution (Browser: MCP Agent or Playwright)

For each test case in the **UI queue** (`TC-*-UI-*.md`), use the browser strategy detected in Phase 0:

**If using MCP Browser Agent:**

1. **Pre-action screenshot:** Navigate to the target page/route specified in `UI Details → Page/Route`. Wait for the page to fully render (no loading spinners, content visible). Capture the screen:
   `screenshots/<ROLE>/TC-<ID>-01-before.png`

2. **Execute steps:** For each step in the **When** section:
   - Visually identify the target elements on the page (buttons, inputs, dropdowns, links, tables, modals, etc.).
   - Perform the interaction: click, type, select from dropdown, scroll, drag, upload file, toggle switch, check/uncheck, etc.
   - **For forms:** Fill each field as specified in `UI Details → Form Fields`. Use the Example Values provided. Respect field types (date pickers, number inputs, selects).
   - **For tables:** Interact with sorting headers, filter inputs, pagination controls, row action buttons as specified.
   - **For modals:** Click the trigger element, wait for the modal to animate in, interact with modal content, confirm or cancel.
   - Wait for the page to update after each interaction (new content visible, loading spinners gone, navigation complete, toasts appear).
   - If a step involves a form submission or destructive action, capture a **mid-action screenshot**:
     `screenshots/<ROLE>/TC-<ID>-02-action-<step-number>.png`

3. **Post-action screenshot:** After all steps are completed, capture the final state:
   `screenshots/<ROLE>/TC-<ID>-03-after.png`

4. **Validate assertions:** For each assertion in the **Then** section:
   - Visually verify: text content, element presence/absence, URL in address bar, toast messages, table data changes, button states (enabled/disabled), navigation menu changes, error messages, empty states.
   - Check `UI Details → Loading States`: verify loading indicators appeared during async operations.
   - Check `UI Details → Error States`: if testing error scenarios, verify error UI appeared correctly.
   - Record each assertion as `PASS` or `FAIL` with actual vs. expected values.

5. **Execute negative scenarios:** For each negative scenario in the test case:
   - Perform the unauthorized/invalid action (navigate to forbidden route, submit empty form, enter invalid data).
   - Verify: client-side validation errors, redirect to unauthorized page, disabled buttons remain disabled, error toasts appear.
   - Capture screenshot:
     `screenshots/<ROLE>/TC-<ID>-04-negative-<scenario-number>.png`

**If using Playwright:**

1. **Pre-action screenshot:** Navigate to the target page/route. Wait for network idle and DOM stable. Capture screenshot:
   `screenshots/<ROLE>/TC-<ID>-01-before.png`

2. **Execute steps:** For each step in the **When** section:
   - Use Playwright selectors (CSS, text, role, test-id) to find target elements.
   - Perform interactions via Playwright API: `click()`, `fill()`, `selectOption()`, `check()`, `setInputFiles()`, `dragTo()`, etc.
   - **For forms:** Use `fill()` for text inputs, `selectOption()` for selects, `check()`/`uncheck()` for checkboxes, `setInputFiles()` for file uploads.
   - **For tables:** Use `click()` on column headers for sorting, `fill()` on filter inputs, `click()` on pagination buttons.
   - **For modals:** Use `click()` on trigger, `waitForSelector()` for modal content, interact, then confirm/cancel.
   - Use Playwright's built-in waits: `waitForSelector()`, `waitForNavigation()`, `waitForResponse()`, `waitForLoadState('networkidle')`.
   - Capture mid-action screenshots for submissions/destructive actions:
     `screenshots/<ROLE>/TC-<ID>-02-action-<step-number>.png`

3. **Post-action screenshot:** Capture final state:
   `screenshots/<ROLE>/TC-<ID>-03-after.png`

4. **Validate assertions:** Use Playwright assertions:
   - `expect(page).toHaveURL(...)` for URL checks
   - `expect(locator).toBeVisible()` / `toBeHidden()` for element visibility
   - `expect(locator).toHaveText(...)` for text content
   - `expect(locator).toBeDisabled()` / `toBeEnabled()` for button states
   - `expect(locator).toHaveCount(...)` for table rows
   - Record each assertion as `PASS` or `FAIL`.

5. **Execute negative scenarios:** Same as MCP agent but using Playwright selectors and assertions. Capture screenshots:
   `screenshots/<ROLE>/TC-<ID>-04-negative-<scenario-number>.png`

**Common to both strategies — On failure:**
   - Capture an **error screenshot:** `screenshots/<ROLE>/TC-<ID>-ERROR.png`
   - If using Playwright, capture browser console errors: `page.on('console', ...)` and `page.on('pageerror', ...)`.
   - Record the failure details: step that failed, actual behavior, expected behavior, console errors, network errors.
   - If `$ON_FAILURE=skip-deps`: flag any test cases that list this one as a dependency (in `Notes` or `Related API Test Case`) as `SKIPPED (dependency TC-<ID> failed)`.
   - If `$ON_FAILURE=stop`: halt execution and proceed directly to Phase 4 (Report Generation).
   - If `$ON_FAILURE=continue`: log the failure and proceed to the next test case.

#### 2.2 — API Test Case Execution (curl + Linux tools)

For each test case in the **API queue** (`TC-*-API-*.md`), use `curl` and standard Linux tools exclusively (never open a browser).

1. **Build the curl command** from the `API Details` section of the test case:
   ```bash
   curl -s -w "\n%{http_code}\n%{time_total}" -X <METHOD> http://localhost:8080<ENDPOINT> \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '<REQUEST_BODY if applicable>' \
     -o /tmp/tc-<ID>-response.json \
     -D /tmp/tc-<ID>-headers.txt
   ```

2. **Execute the curl command** and capture:
   - HTTP status code (from `-w "%{http_code}"`)
   - Response body saved to temp file
   - Response headers saved to temp file
   - Response time (from `-w "%{time_total}"`)

3. **Validate assertions using Linux tools:**
   - **Status code validation:**
     ```bash
     ACTUAL_STATUS=$(tail -1 /tmp/tc-<ID>-status.txt)
     [ "$ACTUAL_STATUS" = "200" ] && echo "PASS" || echo "FAIL: expected 200, got $ACTUAL_STATUS"
     ```
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

     # Validate against expected value
     ACTUAL=$(jq -r '.data.name' /tmp/tc-<ID>-response.json)
     [ "$ACTUAL" = "expected_value" ] && echo "PASS" || echo "FAIL"
     ```
   - **Text matching with `grep`/`sed`/`awk`:**
     ```bash
     # Verify response contains expected string
     grep -q "success" /tmp/tc-<ID>-response.json

     # Extract values without jq as fallback
     grep -oP '"id"\s*:\s*"\K[^"]+' /tmp/tc-<ID>-response.json
     ```
   - **Header validation:**
     ```bash
     # Check content type
     grep -qi "content-type: application/json" /tmp/tc-<ID>-headers.txt

     # Check pagination headers
     grep -oP 'X-Total-Count: \K\d+' /tmp/tc-<ID>-headers.txt
     ```
   - **Data comparison with `diff`:**
     ```bash
     # Compare response structure against expected template
     jq -S 'keys' /tmp/tc-<ID>-response.json > /tmp/tc-<ID>-actual-keys.txt
     diff /tmp/tc-<ID>-expected-keys.txt /tmp/tc-<ID>-actual-keys.txt
     ```
   - **Chain validation for workflows:** Use pipes and variable substitution to extract values from one response and feed them into the next `curl` call:
     ```bash
     # Extract order ID and use in next request
     ORDER_ID=$(jq -r '.data.id' /tmp/tc-<ID>-response.json)
     curl -s -X GET "http://localhost:8080/api/v1/orders/$ORDER_ID" \
       -H "Authorization: Bearer $TOKEN"
     ```
   - Record each assertion as `PASS` or `FAIL`.

4. **Execute negative scenarios** from the test case:
   ```bash
   # No auth token → expect 401
   curl -s -o /dev/null -w "%{http_code}" -X <METHOD> http://localhost:8080<ENDPOINT>

   # Wrong role token → expect 403
   curl -s -o /dev/null -w "%{http_code}" -X <METHOD> http://localhost:8080<ENDPOINT> \
     -H "Authorization: Bearer $WRONG_ROLE_TOKEN"

   # Invalid body → expect 400
   curl -s -w "\n%{http_code}" -X POST http://localhost:8080<ENDPOINT> \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"invalid": "data"}'
   ```

5. **Log the full curl command and response** in the report. Include the `jq`/`grep` commands used for validation.

6. **Clean up temp files** after each test case: `rm -f /tmp/tc-<ID>-*.json /tmp/tc-<ID>-*.txt`

7. **On failure:** Apply same `$ON_FAILURE` behavior as UI tests. Additionally, if a linked UI test case exists and it passed, note the discrepancy (UI works but API doesn't, or vice versa) as a high-priority issue.

#### 2.3 — Cross-Role Workflow Execution

For test cases with cross-role dependencies (noted in `Notes` section):

1. **Execute the workflow in role sequence, both layers.** For example:
   - **UI path:** Login as Purchaser in browser → Execute TC-PU-UI-005 (create purchase order via form) → Capture order ID from the screen.
   - **API verification:** Run TC-PU-API-003 via `curl` to confirm the order was created in the backend.
   - **Role switch (UI):** Logout in browser → Login as Approver → Execute TC-AP-UI-003 (approve order via UI).
   - **API verification:** Run TC-AP-API-002 via `curl` to confirm the order status changed in the backend.

2. **Pass data between steps:** Store intermediate values (IDs, reference numbers, URLs) generated by one role's test case for use in the next role's test case. Both UI-extracted values and API-extracted values can feed into subsequent steps.

3. **Screenshot at each role switch:** Capture `screenshots/<ROLE>/TC-<ID>-cross-role-handoff.png`.

4. **Cross-layer verification:** After a UI action, verify the backend state via `curl`. After an API action, verify the UI reflects the change. Note any discrepancies.

5. **If any step in the chain fails:** Mark all subsequent steps in the chain as `SKIPPED (cross-role dependency failed at TC-<ID>)`.

### Phase 3: Evidence & Screenshot Management

All screenshots are stored under `/docs/test-cases/<role>/screenshots/` with the following naming convention:

| Screenshot Type | Pattern | When Captured |
|---|---|---|
| Auth success | `auth-login-success.png` | After successful UI login |
| Pre-action | `TC-<ID>-01-before.png` | Before executing UI steps |
| Mid-action | `TC-<ID>-02-action-<N>.png` | During key UI interactions |
| Post-action | `TC-<ID>-03-after.png` | After all UI steps complete |
| Negative scenario | `TC-<ID>-04-negative-<N>.png` | During negative UI validations |
| Error/failure | `TC-<ID>-ERROR.png` | On any unexpected UI failure |
| Cross-role handoff | `TC-<ID>-cross-role-handoff.png` | When switching roles in workflows |

**Notes:**
- Screenshots are only captured for UI test cases (browser interactions). API test cases log curl commands and responses as text evidence.
- All screenshots MUST be referenced in the corresponding report files using **relative paths**.

### Phase 4: Report Generation

Generate reports at three levels. Reports now clearly separate UI and API results.

#### 4.1 — Individual Test Case Report

**File:** `/docs/test-cases/<role>/TC-<ID>-report.md`

One report per executed test case. Placed alongside the test case file.

**For UI test cases (`TC-*-UI-*`):**

```markdown
# Test Report: TC-<ID> — <Test Case Title>

## Summary
- **Status:** PASS | FAIL | BLOCKED | SKIPPED
- **Layer:** Frontend (UI)
- **Browser Strategy:** MCP Browser Agent | Playwright
- **Role:** <Role Name>
- **Executed At:** <ISO 8601 timestamp>
- **Duration:** <execution time in seconds>
- **Frontend URL:** <URL tested>

## Results

### UI Step Validation
| Step | Action | Expected | Actual | Status | Screenshot |
|------|--------|----------|--------|--------|------------|
| 1    | <action> | <expected> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-02-action-1.png) |
| 2    | <action> | <expected> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-02-action-2.png) |

**Before State:** [screenshot](screenshots/TC-<ID>-01-before.png)
**After State:** [screenshot](screenshots/TC-<ID>-03-after.png)

### Form Validation (if applicable)
| Field | Input Value | Expected Behavior | Actual Behavior | Status |
|-------|-------------|-------------------|-----------------|--------|
| <n>   | <value>     | <expected>        | <actual>        | PASS/FAIL |

### Negative Scenarios
| Scenario | Action | Expected | Actual | Status | Screenshot |
|----------|--------|----------|--------|--------|------------|
| <desc>   | <action> | <error/denial> | <actual> | PASS/FAIL | [screenshot](screenshots/TC-<ID>-04-negative-1.png) |

### Cross-Layer Verification
- **Related API Test Case:** TC-<PREFIX>-API-<NNN>
- **API Status:** <PASS/FAIL/NOT_EXECUTED>
- **Consistency:** <UI and API results match / DISCREPANCY FOUND>

## Errors
- **Error 1:** <Description of what went wrong>
  - **Step:** <Which step failed>
  - **Expected:** <What should have happened>
  - **Actual:** <What actually happened>
  - **Screenshot:** [error](screenshots/TC-<ID>-ERROR.png)
  - **Console Errors:** <Any browser console errors captured>

## Incidents
- <Any unexpected behavior that didn't cause a failure but is worth noting>

## Issues
- <Confirmed defects found during execution>

## Gaps
- <Missing functionality, untestable steps, or areas where the test case couldn't be fully validated>
```

**For API test cases (`TC-*-API-*`):**

```markdown
# Test Report: TC-<ID> — <Test Case Title>

## Summary
- **Status:** PASS | FAIL | BLOCKED | SKIPPED
- **Layer:** Backend (API)
- **Role:** <Role Name>
- **Executed At:** <ISO 8601 timestamp>
- **Duration:** <execution time in seconds>
- **Backend URL:** <URL tested>

## Results

### API Endpoint Validation
| Endpoint | Method | Expected Status | Actual Status | Response Time | Status |
|----------|--------|-----------------|---------------|---------------|--------|
| <path>   | <GET/POST/...> | <200> | <actual> | <ms> | PASS/FAIL |

### Response Body Validation
| Field / Assertion | Expected | Actual | Status |
|-------------------|----------|--------|--------|
| `$.data.status`   | "active" | <actual> | PASS/FAIL |
| `$.items.length`  | > 0      | <actual> | PASS/FAIL |

<details>
<summary>curl command executed</summary>

\```bash
curl -s -X <METHOD> http://localhost:8080<ENDPOINT> \
  -H "Authorization: Bearer <TOKEN_FIRST_20_CHARS>..." \
  -H "Content-Type: application/json" \
  -d '{ ... }'
\```

</details>

<details>
<summary>Response body</summary>

\```json
{ ... }
\```

</details>

<details>
<summary>Validation commands</summary>

\```bash
jq -r '.data.status' response.json  # Result: "active" ✓
jq '.items | length' response.json   # Result: 5 ✓
\```

</details>

### Negative Scenarios
| Scenario | curl Summary | Expected Status | Actual Status | Status |
|----------|-------------|-----------------|---------------|--------|
| No auth  | `curl -X <METHOD> <ENDPOINT>` | 401 | <actual> | PASS/FAIL |
| Wrong role | `curl -H "Auth: Bearer $OTHER"` | 403 | <actual> | PASS/FAIL |
| Invalid body | `curl -d '{"bad":"data"}'` | 400 | <actual> | PASS/FAIL |

### Cross-Layer Verification
- **Related UI Test Case:** TC-<PREFIX>-UI-<NNN>
- **UI Status:** <PASS/FAIL/NOT_EXECUTED>
- **Consistency:** <UI and API results match / DISCREPANCY FOUND>

## Errors
- **Error 1:** <Description>
  - **curl:** <The failing curl command>
  - **Expected:** <Expected status/response>
  - **Actual:** <Actual status/response>

## Incidents
- <Unexpected behavior: slow responses, deprecation warnings, unexpected response fields>

## Issues
- <Confirmed defects: wrong status codes, missing fields, incorrect data, broken auth>

## Gaps
- <Untestable scenarios: side effects like email sending, event publishing, external service calls>
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
- **Browser Strategy:** <MCP Browser Agent | Playwright>

## Summary

| Layer | Total | Passed | Failed | Blocked | Skipped | Pass Rate |
|-------|-------|--------|--------|---------|---------|-----------|
| UI (Frontend) | <N> | <N> | <N> | <N> | <N> | <pct>% |
| API (Backend) | <N> | <N> | <N> | <N> | <N> | <pct>% |
| **Total** | **<N>** | **<N>** | **<N>** | **<N>** | **<N>** | **<pct>%** |

## UI Test Case Results

| ID | Title | Priority | Type | Status | Duration | Related API | Report |
|----|-------|----------|------|--------|----------|-------------|--------|
| TC-<PREFIX>-UI-001 | <title> | <priority> | <type> | PASS/FAIL | <sec> | TC-<PREFIX>-API-<N> | [report](TC-<ID>-report.md) |

## API Test Case Results

| ID | Title | Priority | Type | Status | Duration | Related UI | Report |
|----|-------|----------|------|--------|----------|------------|--------|
| TC-<PREFIX>-API-001 | <title> | <priority> | <type> | PASS/FAIL | <sec> | TC-<PREFIX>-UI-<N> | [report](TC-<ID>-report.md) |

## Cross-Layer Discrepancies
<List any cases where UI passed but API failed, or vice versa — these are high-priority issues>

| UI Test | UI Status | API Test | API Status | Discrepancy |
|---------|-----------|----------|------------|-------------|
| TC-XX-UI-001 | PASS | TC-XX-API-001 | FAIL | <description> |

## All Errors
<Aggregated list of all errors across test cases for this role, with links to individual reports>

## All Incidents
<Aggregated list of all incidents>

## All Issues
<Aggregated list of all confirmed defects>

## All Gaps
<Aggregated list of all gaps and missing functionality>

## Cross-Role Workflows
| Workflow | Roles Involved | UI Status | API Status | Notes |
|----------|---------------|-----------|------------|-------|
| <workflow name> | Purchaser → Approver | PASS/FAIL | PASS/FAIL | <details> |
```

#### 4.3 — Global Execution Report

**File:** `/docs/test-cases/execution-report.md`

Single global report for the entire execution run.

```markdown
# Global Test Execution Report

## Execution Metadata
- **Executed At:** <ISO 8601 timestamp>
- **Parameters:** ROLE=<value or "all"> | LAYER=<value> | TEST_CASE_ID=<value or "N/A"> | ON_FAILURE=<value>
- **Frontend URL:** <URL> (Status: UP/DOWN)
- **Backend URL:** <URL> (Status: UP/DOWN)
- **Browser Strategy:** <MCP Browser Agent | Playwright>
- **Total Duration:** <total time>
- **Docs Read:** AGENTS.md, claude.md, credentials.md, <others>

## Global Summary

| Role | UI Total | UI Pass | UI Fail | API Total | API Pass | API Fail | Overall Pass Rate |
|------|----------|---------|---------|-----------|----------|----------|-------------------|
| Super Admin | <N> | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Admin | <N> | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Approver | <N> | <N> | <N> | <N> | <N> | <N> | <pct>% |
| Purchaser | <N> | <N> | <N> | <N> | <N> | <N> | <pct>% |
| **Total** | **<N>** | **<N>** | **<N>** | **<N>** | **<N>** | **<N>** | **<pct>%** |

## Layer Comparison

| Metric | UI (Frontend) | API (Backend) |
|--------|---------------|---------------|
| Total Test Cases | <N> | <N> |
| Pass Rate | <pct>% | <pct>% |
| Avg Response Time | N/A | <ms> |
| Critical Failures | <N> | <N> |

## Cross-Layer Discrepancies (High Priority)
<Aggregated list of all cases where UI and API results don't match>

## Critical Failures
<List of all FAIL results with Critical or High priority, with links to individual reports>

## All Errors (aggregated)
<Numbered list of unique errors across all roles and layers>

## All Issues (aggregated)
<Numbered list of confirmed defects across all roles and layers>

## All Incidents (aggregated)
<Numbered list of incidents across all roles and layers>

## All Gaps (aggregated)
<Numbered list of gaps across all roles and layers>

## Cross-Role Workflow Summary
| Workflow | Roles | UI Status | API Status | Link |
|----------|-------|-----------|------------|------|
| <name> | <roles> | PASS/FAIL | PASS/FAIL | [details](<role>/TC-<ID>-report.md) |

## Recommendations
<Based on the execution results, provide recommendations:>
- <Cross-layer discrepancies to investigate urgently>
- <Defects to prioritize (grouped by UI vs API)>
- <Missing features flagged as gaps>
- <Test cases that need refinement>
- <Areas needing more test coverage (frontend, backend, or both)>
- <Defense-in-depth findings: UI hides feature but API allows it, or vice versa>

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
4. **Verify all screenshots** referenced in UI reports actually exist on disk.
5. **Verify cross-layer links** in reports point to valid report files.
6. **Print a terminal summary:**
   ```
   ============================================
   TEST EXECUTION COMPLETE
   ============================================
   Layer:    UI: <N> pass / <N> fail | API: <N> pass / <N> fail
   Total:    <N> | Pass: <N> | Fail: <N> | Blocked: <N> | Skipped: <N>
   Pass Rate: <pct>%
   Cross-Layer Discrepancies: <N>
   
   Reports:
     Global:      /docs/test-cases/execution-report.md
     Super Admin: /docs/test-cases/super-admin/report.md (UI: <N>, API: <N>)
     Admin:       /docs/test-cases/admin/report.md (UI: <N>, API: <N>)
     Approver:    /docs/test-cases/approver/report.md (UI: <N>, API: <N>)
     Purchaser:   /docs/test-cases/purchaser/report.md (UI: <N>, API: <N>)
   
   Docs Used: AGENTS.md, claude.md, credentials.md
   ============================================
   ```

## Important Notes

- **ALWAYS read AGENTS.md and/or claude.md FIRST** before any execution. These files contain critical context about how to run the project, auth flows, known issues, and environmental setup.
- **Browser strategy selection is automatic.** MCP Browser Agent first, Playwright as fallback. Once chosen, same strategy for all UI tests. Log which strategy was selected.
- **UI tests use only the browser.** All frontend interactions (navigation, forms, tables, modals, toasts, etc.) are performed through the browser agent or Playwright. Never use `curl` for UI test cases.
- **API tests use only the terminal.** All backend endpoint testing uses `curl`, `jq`, `grep`, `sed`, `awk`, `diff`, and any other standard Linux tools. Never open a browser for API tests.
- **Cross-layer verification is critical.** After UI actions, verify backend state via `curl`. After API actions, verify UI reflects changes. Cross-layer discrepancies are flagged as high-priority issues.
- **If only one service is running:** Execute available layer, mark other layer as `BLOCKED`. This allows partial test runs when developing frontend or backend independently.
- **Never hardcode passwords in reports.** Mask as `*****`, point to `credentials.md`.
- **Truncate JWT tokens in reports** to first 20 characters followed by `...`.
- **Screenshots are relative paths** in all reports — never absolute filesystem paths.
- **If a test case file cannot be parsed**, skip it, log a warning, continue.
- **Cross-role workflows** are only executed when running in "all roles" mode or when both involved roles are in scope. Otherwise, execute only the current role's part.
- **Retry logic:** UI transient failures (element not found, timeout) → retry once after 3 seconds. `curl` connection errors → retry once after 2 seconds. Mark as failed after retry.
- **Defense-in-depth flagging:** If a UI test shows a button/page is hidden for a role BUT the corresponding API test shows the endpoint is accessible → flag as CRITICAL security issue in the report.
