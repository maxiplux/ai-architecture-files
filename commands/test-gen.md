---
description: Generate tests for implemented code
---
1. Read docs/tasks.md and identify the current phase
2. Read docs/requirements.md for Gherkin scenarios related to this phase
3. Read docs/plan.md for API contracts and validation rules

## For each completed task in the current phase:
- Generate unit tests covering:
  - Happy path
  - Edge cases
  - Error conditions
- Generate integration tests for API endpoints
- Ensure tests reference requirement IDs (FR-xxx) in comments

## Test Naming Convention
`test_[FR-xxx]_[scenario]_[expected_result]`

## Output
Create/update test files following project structure in plan.md Section 5.

Don't read docs/done.
