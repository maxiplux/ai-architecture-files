# Generate Test Cases by Role

## Objective

Read and understand the entire project — both backend and frontend — by first reading `AGENTS.md`, `claude.md`, and all project documentation. Detect the project structure (monorepo or single repo), locate the frontend and backend codebases, parse `credentials.md` to extract roles and their scopes, deeply analyze both codebases, and generate comprehensive test cases organized by role under `/docs/test-cases/` with **separate test cases for UI (frontend) and API (backend)**.

## Instructions

### Phase 0: Project Intelligence Gathering

**This phase is mandatory and must be completed before any analysis or generation.**

1. **Read agent/project documentation first:**
   - Search for and read the following files (check root, `.claude/`, `/docs/`, and any subdirectory):
     - `AGENTS.md` — understand development patterns, architecture decisions, conventions, coding standards, and any agent-specific instructions.
     - `claude.md` or `.claude/claude.md` — understand project-specific Claude instructions, preferences, and constraints.
     - `CLAUDE.md` — same as above, check all casing variants.
     - `README.md` — project overview, setup instructions, tech stack summary.
   - **Extract and internalize:**
     - Architecture pattern (e.g., Vertical Slices, Clean Architecture, MVC, etc.)
     - Tech stack details (frameworks, libraries, versions)
     - Coding conventions and naming patterns
     - Folder structure conventions
     - Any testing-specific instructions or preferences
     - Any role/permission descriptions or business domain context
   - These documents provide the foundational understanding for everything that follows. Treat their content as authoritative project context.

2. **Read all supplementary documentation:**
   - Scan `/docs/` recursively for any `.md` files: requirements specs, user stories, feature descriptions, API documentation, architecture decision records (ADRs).
   - Read `package.json`, `pom.xml`, `build.gradle`, `settings.gradle`, `tsconfig.json`, `next.config.*` — these reveal dependencies, scripts, and project configuration.
   - Read `.env.example`, `.env.local.example`, `application.yml`, `application.properties` — these reveal environment variables, service URLs, feature flags.
   - Store all discovered context for use in subsequent phases.

### Phase 1: Project Discovery

1. **Detect project structure (auto-detect):**
   - Check the root directory for monorepo indicators: `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`, multiple `package.json` or `pom.xml` files in subdirectories, or a `settings.gradle` with `include` directives.
   - If monorepo: identify each package/module, its path, and its tech stack.
   - If single repo: identify the tech stack and entry points.
   - **Critical: Locate the frontend and backend separately:**
     - **Backend:** Look for Spring Boot indicators (`pom.xml` with spring-boot-starter, `src/main/java`, `@RestController`, `@SpringBootApplication`), or Node.js backend (`express`, `fastify`, `nestjs`).
     - **Frontend:** Look for Next.js indicators (`next.config.*`, `app/` or `pages/` directory, `package.json` with `next` dependency), or React indicators (`react-scripts`, `vite.config.*`).
     - Record the exact paths: e.g., `backend: /apps/backend`, `frontend: /apps/frontend` or `backend: /`, `frontend: /client`.
   - Output your findings as a summary before proceeding.

2. **Read credentials:**
   - Locate and read `credentials.md` (check root, `/docs`, `.claude/`, or any common location).
   - Parse the **table format**: `| Role | Email | Password | Scope |`.
   - Extract each role, its associated email, and most importantly, its **Scope** description.
   - These roles define the folder structure and test case boundaries.

3. **Analyze the BACKEND codebase for features:**
   - **Controllers/Endpoints:** Scan all `@RestController`, `@Controller`, route definitions. Extract every endpoint path, HTTP method, request/response DTOs.
   - **Security/RBAC:** Scan `@PreAuthorize`, `@Secured`, `@RolesAllowed` annotations, Spring Security configuration, `SecurityFilterChain`, role hierarchies, permission evaluators.
   - **Service layer:** Identify business logic methods, validation rules, business workflows.
   - **Database/Migrations:** Check for role or permission tables, seed data, Flyway/Liquibase migrations, JPA entities that define role hierarchies.
   - **Error handling:** Identify `@ControllerAdvice`, `ProblemDetail` responses, custom exception handlers — these define expected error responses for negative test cases.
   - **API documentation:** Read any SpringDoc/OpenAPI annotations (`@Operation`, `@ApiResponse`, `@Schema`), Swagger configs, or generated API specs.
   - Build a **Backend Feature Map**: every endpoint grouped by module/feature, annotated with which roles can access it.

4. **Analyze the FRONTEND codebase for features:**
   - **Routing & Pages:**
     - For Next.js App Router: scan `app/` directory for `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, route groups `(group)`, parallel routes `@slot`, intercepting routes.
     - For Next.js Pages Router: scan `pages/` directory for all page files.
     - For React Router: scan route definitions, `<Route>` components, router configuration.
     - Record every user-accessible route/page.
   - **Authentication flow (auto-detect):**
     - Look for login pages/components, auth providers, session management.
     - Detect token storage: `localStorage`, `sessionStorage`, cookies (`js-cookie`, `next-auth`, `iron-session`).
     - Detect auth middleware: Next.js `middleware.ts`, route guards, `useAuth` hooks, `withAuth` HOCs.
     - Detect OAuth/SSO: Keycloak, Auth0, NextAuth providers.
     - Record the full auth flow: login page → token acquisition → storage → injection into requests → refresh → logout.
   - **Frontend-Backend communication (auto-detect):**
     - Scan for API call patterns: `fetch()`, `axios`, `ky`, `swr`, `react-query`/`tanstack-query`, custom API clients.
     - Detect if calls go direct to backend (`localhost:8080`) or through a BFF/proxy (Next.js API routes at `/api/*`, `rewrites` in `next.config.*`).
     - Identify base URLs, interceptors, auth header injection.
   - **Role-based rendering:**
     - Scan for conditional rendering based on role/permissions: `{role === 'admin' && ...}`, `hasPermission()`, `useRole()`, role-based navigation menus.
     - Identify components/pages that are hidden, disabled, or show different content per role.
     - Identify navigation guards and redirects for unauthorized access.
   - **Interactive components:**
     - Forms: identify all form components, validation schemas (Zod, Yup, native), required fields, field types, submission handlers.
     - Tables: identify data tables with pagination, sorting, filtering, row actions.
     - Modals/Dialogs: identify confirmation dialogs, creation/edit modals, deletion confirmations.
     - Notifications: identify toast/notification systems, success/error messages.
     - Dropdowns/Selects: identify complex selectors, multi-selects, search/filter dropdowns.
     - File uploads: identify upload components, file type restrictions, size limits.
   - **State management:** Identify state libraries (Redux, Zustand, Jotai, Context API) and how application state relates to role-based features.
   - **Loading & Error states:** Identify loading skeletons, spinners, error boundaries, fallback UIs, empty states.
   - **Responsive design:** Identify breakpoints, mobile navigation (hamburger menus), responsive layouts that might hide/show features.
   - Build a **Frontend Feature Map**: every page/route grouped by module/feature, annotated with which roles can access it and what interactions are available.

5. **Merge Feature Maps:**
   - Combine the Backend Feature Map and Frontend Feature Map into a **Unified Feature Map**.
   - For each feature, identify:
     - Which API endpoints power it (backend)
     - Which pages/components render it (frontend)
     - Which roles can access it
     - Whether it's backend-only, frontend-only, or full-stack
   - This unified map drives test case generation for both UI and API.

### Phase 2: Folder Structure Creation

Create the following folder structure under `/docs/test-cases/`:

```
docs/
└── test-cases/
    ├── README.md                                    # Overview: roles, scope, unified feature map
    ├── super-admin/
    │   ├── README.md                                # Role summary, scope, test case inventory
    │   ├── TC-SA-UI-001-<feature-slug>.md           # UI (frontend) test case
    │   ├── TC-SA-UI-002-<feature-slug>.md
    │   ├── TC-SA-API-001-<feature-slug>.md          # API (backend) test case
    │   ├── TC-SA-API-002-<feature-slug>.md
    │   └── ...
    ├── admin/
    │   ├── README.md
    │   ├── TC-AD-UI-001-<feature-slug>.md
    │   ├── TC-AD-API-001-<feature-slug>.md
    │   └── ...
    ├── approver/
    │   ├── README.md
    │   ├── TC-AP-UI-001-<feature-slug>.md
    │   ├── TC-AP-API-001-<feature-slug>.md
    │   └── ...
    └── purchaser/
        ├── README.md
        ├── TC-PU-UI-001-<feature-slug>.md
        ├── TC-PU-API-001-<feature-slug>.md
        └── ...
```

**Naming conventions:**
- Folder names: lowercase, kebab-case (e.g., `super-admin`)
- File prefix codes: `SA` = Super Admin, `AD` = Admin, `AP` = Approver, `PU` = Purchaser
- **Test type separator:** `UI` = Frontend/browser test, `API` = Backend/curl test
- Feature slug: short kebab-case description (e.g., `organization-management`, `purchase-order-creation`)
- Sequential numbering: `001`, `002`, etc. — **numbered independently per type** (UI-001, UI-002... and API-001, API-002...)

### Phase 3: Test Case Generation

For each role, generate test cases covering **all features accessible to that role** based on the Unified Feature Map. Generate **separate test case files for UI and API**.

#### 3.1 — UI Test Case Template (Frontend)

For every frontend page, route, form, or interactive component accessible to the role:

```markdown
# TC-<PREFIX>-UI-<NNN>: <Test Case Title>

## Metadata
- **Role:** <Role Name>
- **Scope:** <Scope from credentials>
- **Layer:** Frontend (UI)
- **Module/Feature:** <Feature or module name>
- **Priority:** Critical | High | Medium | Low
- **Type:** Functional | Negative | Boundary | Security | Integration | Usability
- **Source:** Codebase Analysis | Requirements Doc | Both

## Preconditions
- User is authenticated as `<email>` with role `<Role>` via the frontend login page
- <Any other preconditions: data state, feature flags, browser requirements>

## Test Scenario

### Given
- <Initial context / system state>
- <Current page/route the user is on>

### When
- <Action(s) the user performs in the browser, step by step>
  1. Navigate to `<route>`
  2. <Click/Type/Select/Scroll/Drag/Upload action>
  3. <Wait for response/loading>
  4. <Next interaction>

### Then
- <Expected visual and behavioral outcomes>
  1. <Page shows/hides specific element>
  2. <Toast notification appears with message "...">
  3. <Table updates with new row>
  4. <URL changes to "...">
  5. <Form field shows validation error "...">

## Negative Scenarios
- **When** user manually navigates to `<unauthorized-route>` **Then** redirect to `/unauthorized` or show 403 page
- **When** user submits form with empty required fields **Then** client-side validation errors appear for each field
- **When** user submits form with invalid data (e.g., negative quantity) **Then** validation error displayed without API call

## UI Details
- **Page/Route:** `/path/to/page`
- **Components involved:** `<ComponentName>`, `<FormName>`, `<ModalName>`
- **User Flow:** <Step-by-step navigation from dashboard to target page>
- **Form Fields (if applicable):**
  | Field | Type | Required | Validation | Example Value |
  |-------|------|----------|------------|---------------|
  | <n> | <text/select/date/number> | Yes/No | <rules> | <value> |
- **Interactive Elements:**
  - Buttons: <list of key buttons and their expected behavior>
  - Tables: <columns, sortable, filterable, paginated>
  - Modals: <trigger, content, confirm/cancel actions>
- **Loading States:** <What loading indicators appear during async operations>
- **Error States:** <What error UI appears on failure>

## Related API Test Case
- **Linked to:** TC-<PREFIX>-API-<NNN> — <title>
- **Relationship:** This UI test case triggers the API call tested in the linked API test case

## Notes
- <Cross-role dependencies>
- <Edge cases: empty states, first-time user experience, max data scenarios>
- <Responsive behavior differences if applicable>
```

#### 3.2 — API Test Case Template (Backend)

For every backend endpoint accessible to the role:

```markdown
# TC-<PREFIX>-API-<NNN>: <Test Case Title>

## Metadata
- **Role:** <Role Name>
- **Scope:** <Scope from credentials>
- **Layer:** Backend (API)
- **Module/Feature:** <Feature or module name>
- **Priority:** Critical | High | Medium | Low
- **Type:** Functional | Negative | Boundary | Security | Integration
- **Source:** Codebase Analysis | Requirements Doc | Both

## Preconditions
- User is authenticated via API as `<email>` with role `<Role>` (Bearer token)
- <Any other preconditions: data state, dependencies, seed data>

## Test Scenario

### Given
- <Initial API state / database state>

### When
- Execute the following API request:
  ```bash
  curl -s -X <METHOD> http://localhost:8080<ENDPOINT> \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '<REQUEST_BODY>'
  ```

### Then
- <Expected API response>
  1. HTTP status: `<200 | 201 | 204 | etc.>`
  2. Response body contains: `<key fields and expected values>`
  3. Response headers include: `<relevant headers if applicable>`
  4. Database state changed: `<what was created/updated/deleted>`

## Negative Scenarios
- **When** request is sent without auth token **Then** `401 Unauthorized`
- **When** request is sent with a role that lacks permission **Then** `403 Forbidden`
- **When** request body has invalid/missing required fields **Then** `400 Bad Request` with ProblemDetail error
- **When** resource does not exist **Then** `404 Not Found`

## API Details
- **Endpoint:** `<METHOD> /api/v1/path`
- **Auth:** Bearer Token (role: <Role>)
- **Request Headers:**
  | Header | Value |
  |--------|-------|
  | Authorization | Bearer $TOKEN |
  | Content-Type | application/json |
- **Request Body (if applicable):**
  ```json
  {
    "field1": "value1",
    "field2": "value2"
  }
  ```
- **Expected Response:**
  - **Status:** `<HTTP status code>`
  - **Body:**
    ```json
    {
      "data": { ... }
    }
    ```
- **Validation Commands:**
  ```bash
  # Verify status code
  # Verify response field
  jq -r '.data.status' response.json
  # Verify array count
  jq '.data.items | length' response.json
  ```

## Related UI Test Case
- **Linked to:** TC-<PREFIX>-UI-<NNN> — <title>
- **Relationship:** This API endpoint is called by the UI interaction in the linked UI test case

## Notes
- <Cross-role dependencies>
- <Rate limiting considerations>
- <Pagination parameters for list endpoints>
- <Side effects: emails sent, events published, audit logs created>
```

#### 3.3 — Generation Rules

**UI (Frontend) test case rules:**
- For every page/route accessible to the role, generate at least one positive UI test case.
- For every form, generate: one happy-path submission test + one client-side validation test (empty fields, invalid data).
- For every table, generate: one data display test + one interaction test (sort, filter, paginate if applicable).
- For every modal/dialog, generate: one open/confirm test + one cancel/dismiss test.
- For role-based rendering: generate test cases verifying elements that should be **visible** and elements that should be **hidden/disabled** for this role.
- For navigation: generate test cases for the menu items and routes accessible to this role, and negative tests for routes that should redirect or show 403.
- For loading states: at least one test case verifying loading indicators appear during async operations.
- For error states: at least one test case verifying the error UI when the backend returns an error.

**API (Backend) test case rules:**
- For every endpoint accessible to the role, generate at least one positive test case with a valid `curl` command.
- For every endpoint, generate negative test cases: no auth (401), wrong role (403), invalid body (400), not found (404).
- For CRUD operations: generate test cases for Create, Read, Update, Delete where applicable.
- For list endpoints: include pagination, sorting, and filtering parameters.
- For workflows: generate chained `curl` commands that pass data between steps (e.g., create → get by ID → update → delete).
- Include `jq` validation commands in every test case for automated assertion.

**Cross-layer linking:**
- Every UI test case that triggers an API call MUST reference the corresponding API test case in `Related API Test Case`.
- Every API test case that powers a UI feature MUST reference the corresponding UI test case in `Related UI Test Case`.
- If a feature is frontend-only (client-side logic, no API call), set Related API Test Case to "N/A — client-side only".
- If a feature is backend-only (no UI, e.g., internal cron job), set Related UI Test Case to "N/A — backend-only".

**Security test cases:**
- For each role, generate UI test cases verifying the role **cannot** navigate to pages outside its scope.
- For each role, generate API test cases verifying the role **cannot** call endpoints outside its scope.
- Generate at least one defense-in-depth test: verify that even if the frontend hides a button, a direct `curl` call to the endpoint is also denied.

**Cross-role test cases:**
- If a workflow spans roles (e.g., Purchaser creates order → Approver approves), create linked test cases in both role folders with dependencies noted.
- Create both UI and API versions of cross-role workflows.

### Phase 4: README Generation

**`/docs/test-cases/README.md`** must include:

1. **Project intelligence summary** — what was learned from `AGENTS.md`, `claude.md`, `README.md`.
2. **Project type** (monorepo / single repo) and detected packages/modules with their paths.
3. **Tech stack** — backend framework, frontend framework, auth mechanism, state management, API communication pattern.
4. **Roles summary table** (mirroring credentials.md with scope descriptions).
5. **Unified Feature Map** — which features are accessible to which roles, with both backend endpoints and frontend pages listed.
6. **Test case inventory:**

   | Role | UI Test Cases | API Test Cases | Total |
   |------|---------------|----------------|-------|
   | Super Admin | <N> | <N> | <N> |
   | Admin | <N> | <N> | <N> |
   | Approver | <N> | <N> | <N> |
   | Purchaser | <N> | <N> | <N> |
   | **Total** | **<N>** | **<N>** | **<N>** |

7. **How to use** — instructions on how to read, update, and extend the test cases. Explain the UI vs API separation and the cross-layer linking.
8. **Generation metadata** — date generated, source files analyzed (including AGENTS.md, claude.md).

**Each role's `README.md`** must include:
1. Role name, email, scope.
2. **UI Test Cases table:**
   | ID | Title | Priority | Type | Related API |
   |----|-------|----------|------|-------------|
3. **API Test Cases table:**
   | ID | Title | Priority | Type | Related UI |
   |----|-------|----------|------|------------|
4. Summary of what this role can and cannot do — both in the UI (pages visible, hidden menus, disabled buttons) and in the API (endpoints accessible, endpoints denied).
5. Authentication flow for this role (how they log in via UI, what token they get for API).

### Phase 5: Validation

Before finishing:
- Verify every detected backend endpoint has at least one API test case assigned to a role.
- Verify every detected frontend page/route has at least one UI test case assigned to a role.
- Verify every role has at least one UI security test case (unauthorized navigation).
- Verify every role has at least one API security test case (unauthorized endpoint access).
- Verify at least one defense-in-depth test exists (UI hidden + API denied).
- Verify all cross-layer links (Related UI ↔ Related API) are bidirectional and valid.
- Verify all test case IDs are unique and sequential within each type per role.
- Verify cross-role workflow dependencies are referenced in both directions and in both layers.
- Report a summary:
  ```
  ============================================
  TEST CASE GENERATION COMPLETE
  ============================================
  Backend endpoints found:  <N>
  Frontend pages/routes found: <N>
  
  Test Cases Generated:
    UI:  <N> (SA: <N>, AD: <N>, AP: <N>, PU: <N>)
    API: <N> (SA: <N>, AD: <N>, AP: <N>, PU: <N>)
    Total: <N>
  
  Cross-layer links: <N>
  Cross-role workflows: <N>
  Security test cases: <N>
  
  Docs read: AGENTS.md, claude.md, credentials.md, <others>
  ============================================
  ```

## Important Notes

- **ALWAYS read AGENTS.md and/or claude.md FIRST** before any analysis. These files contain critical project context that affects how test cases should be structured and what patterns to expect.
- Do NOT include actual passwords in the generated test case files. Reference the role and email only, and point to `credentials.md` for authentication details.
- If the project is a monorepo, prefix feature names with the package/module they belong to (e.g., `backend: purchase-order-creation`, `frontend: dashboard-view`).
- If no existing requirements documents are found, note this in the README and mark all test cases as "Source: Codebase Analysis".
- Generate test cases incrementally — start with Critical priority features, then High, Medium, and Low.
- **Frontend detection is auto.** Do not assume the frontend location — discover it from the file system, configs, and `AGENTS.md`.
- **Auth flow detection is auto.** Do not assume login page routes or token storage — discover them from the codebase.
- **API communication detection is auto.** Do not assume direct calls vs proxy — discover from frontend code (fetch calls, API clients, Next.js rewrites).
- If `AGENTS.md` or `claude.md` describes features or modules not yet found in code, create placeholder test cases marked as "Source: Requirements Doc" with a note "Feature described in documentation but not yet detected in codebase".
