---
description: AI Agent Guide: Loop Validation - Requirements to Plan
---
## Document Purpose
Validate that `plan.md` fully addresses all items in `requirements.md`.

**Critical Rule:** 100% coverage required. Every FR, NFR, and Gherkin scenario must trace to plan.md components.

## Validation Checklist
1. Read docs/requirements.md completely
   - Extract: All FR-xxx, NFR-xxx, Gherkin scenarios, data requirements
   
2. Read docs/plan.md completely
   - Extract: Components, APIs, architectural decisions, phases

3. Build traceability matrix:
   | Requirement | Plan Component | Status |
   |-------------|----------------|--------|
   | FR-001      | Component X    | FOUND/GAP |

4. Verify each FR has:
   - [ ] At least one component implementing it
   - [ ] API endpoint (if user-facing)
   - [ ] Covered in a phase

5. Verify each NFR has:
   - [ ] Architectural decision supporting it
   - [ ] Implementation approach defined

6. Generate gap report if coverage < 100%

Don't read docs/done.
