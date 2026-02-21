# Generate Test Cases by Role

## Objective

Read and understand the entire project, detect its structure (monorepo or single repo), parse `credentials.md` to extract roles and their scopes, analyze the codebase and existing documentation, and generate comprehensive test cases organized by role under `/docs/test-cases/`.

## Instructions

### Phase 1: Project Discovery

1. **Detect project structure:**
   - Check the root directory for monorepo indicators: `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`, multiple `package.json` or `pom.xml` files in subdirectories, or a `settings.gradle` with `include` directives.
   - If monorepo: identify each package/module and its tech stack (Spring Boot, Next.js, etc.).
   - If single repo: identify the tech stack and entry points.
   - Output your findings as a summary before proceeding.

2. **Read credentials:**
   - Locate and read `credentials.md` (check root, `/docs`, `.claude/`, or any common location).
   - Parse the **table format**: `| Role | Email | Password | Scope |`.
   - Extract each role, its associated email, and most importantly, its **Scope** description.
   - These roles define the folder structure and test case boundaries.

3. **Analyze the codebase for features:**
   - **Backend (Spring Boot / API):** Scan controllers, route definitions, `@PreAuthorize` / `@Secured` annotations, endpoint paths, service layer methods, and any role-based access control (RBAC) configuration.
   - **Frontend (Next.js / React):** Scan pages, routes, navigation guards, middleware, role-based component rendering, and protected routes.
   - **Database/Migrations:** Check for role or permission tables, seed data, or migration files that define role hierarchies.
   - **Existing documentation:** Read any files in `/docs`, `README.md`, `AGENTS.md`, requirements specs, user stories, or any `.md` files that describe features, workflows, or acceptance criteria. Extract testable scenarios from them.
   - Build a **Feature Map**: a list of features/modules grouped by which roles have access to them based on the scope and RBAC rules found.

### Phase 2: Folder Structure Creation

Create the following folder structure under `/docs/test-cases/`:

```
docs/
└── test-cases/
    ├── README.md                          # Overview: roles, scope summary, how to use
    ├── super-admin/
    │   ├── README.md                      # Role summary, scope, credentials reference
    │   ├── TC-SA-001-<feature-slug>.md
    │   ├── TC-SA-002-<feature-slug>.md
    │   └── ...
    ├── admin/
    │   ├── README.md
    │   ├── TC-AD-001-<feature-slug>.md
    │   └── ...
    ├── approver/
    │   ├── README.md
    │   ├── TC-AP-001-<feature-slug>.md
    │   └── ...
    └── purchaser/
        ├── README.md
        ├── TC-PU-001-<feature-slug>.md
        └── ...
```

**Naming conventions:**
- Folder names: lowercase, kebab-case (e.g., `super-admin`)
- File prefix codes: `SA` = Super Admin, `AD` = Admin, `AP` = Approver, `PU` = Purchaser
- Feature slug: short kebab-case description (e.g., `organization-management`, `purchase-order-creation`)
- Sequential numbering: `001`, `002`, etc.

### Phase 3: Test Case Generation

For each role, generate test cases covering **all features accessible to that role** based on the Feature Map built in Phase 1.

**Each test case file MUST follow this template:**

```markdown
# TC-<PREFIX>-<NNN>: <Test Case Title>

## Metadata
- **Role:** <Role Name>
- **Scope:** <Scope from credentials>
- **Module/Feature:** <Feature or module name>
- **Priority:** Critical | High | Medium | Low
- **Type:** Functional | Negative | Boundary | Security | Integration
- **Source:** Codebase Analysis | Requirements Doc | Both

## Preconditions
- User is authenticated as `<email>` with role `<Role>`
- <Any other preconditions: data state, dependencies, configurations>

## Test Scenario

### Given
- <Initial context / system state>

### When
- <Action(s) the user performs, step by step>
  1. <Step 1>
  2. <Step 2>
  3. <Step N>

### Then
- <Expected outcome(s)>
  1. <Assertion 1>
  2. <Assertion 2>
  3. <Assertion N>

## Negative Scenarios
- **When** <unauthorized action or invalid input> **Then** <expected error or denial>

## API Reference (if applicable)
- **Endpoint:** `<METHOD> /api/v1/path`
- **Auth:** Bearer Token (role: <Role>)
- **Expected Status:** <200 | 201 | 403 | etc.>

## UI Reference (if applicable)
- **Page/Route:** `/path/to/page`
- **Component:** `<ComponentName>`
- **User Flow:** <Brief description of the navigation>

## Notes
- <Any additional context, edge cases, dependencies on other test cases>
```

**Generation rules:**
- **Source: Codebase Analysis (3a):** For every endpoint, page, or feature detected in the codebase that the role can access, generate at least one positive and one negative test case.
- **Source: Requirements Doc (3b):** For every testable requirement or user story found in existing documentation, generate a test case and mark its source as "Requirements Doc" or "Both" if it also maps to code.
- **Security test cases:** For each role, generate test cases verifying the role **cannot** access features outside its scope (e.g., Purchaser cannot access Admin endpoints).
- **Cross-role test cases:** If a workflow spans roles (e.g., Purchaser creates order → Approver approves), note the dependency in the `Notes` section and create linked test cases in both role folders.

### Phase 4: README Generation

**`/docs/test-cases/README.md`** must include:

1. **Project type** (monorepo / single repo) and detected packages/modules.
2. **Roles summary table** (mirroring credentials.md with scope descriptions).
3. **Feature Map summary** — which features are accessible to which roles.
4. **Test case inventory** — total count per role with a table of IDs and titles.
5. **How to use** — instructions on how to read, update, and extend the test cases.
6. **Generation metadata** — date generated, source files analyzed.

**Each role's `README.md`** must include:
1. Role name, email, scope.
2. List of all test case files in that folder with ID, title, priority, and type.
3. Summary of what this role can and cannot do in the system.

### Phase 5: Validation

Before finishing:
- Verify every detected feature has at least one test case assigned to a role.
- Verify every role has at least one security/negative test case for out-of-scope access.
- Verify all test case IDs are unique and sequential within each role.
- Verify cross-role workflow dependencies are referenced in both directions.
- Report a summary: total test cases generated, per role count, any features without coverage.

## Important Notes

- Do NOT include actual passwords in the generated test case files. Reference the role and email only, and point to `credentials.md` for authentication details.
- If the project is a monorepo, prefix feature names with the package/module they belong to (e.g., `backend: purchase-order-creation`, `frontend: dashboard-view`).
- If no existing requirements documents are found, note this in the README and mark all test cases as "Source: Codebase Analysis".
- Generate test cases incrementally — start with Critical priority features, then High, Medium, and Low.
