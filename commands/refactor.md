---
description: Refactor code while maintaining requirements compliance
---
## Refactoring Protocol

1. **Pre-refactor check:**
   - Run full test suite: `[PROJECT_TEST_COMMAND]`
   - Capture baseline: all tests must pass

2. **Scope identification:**
   - What code needs refactoring?
   - Which requirements (FR-xxx) does it implement?
   - What is the goal? (performance, readability, DRY, etc.)

3. **Safe refactoring rules:**
   - Do NOT change public interfaces without updating plan.md
   - Do NOT change behavior (tests should still pass)
   - Do NOT remove requirement traceability comments

4. **Execute refactoring:**
   - Make incremental changes
   - Run tests after each significant change

5. **Post-refactor verification:**
   - All original tests pass
   - No new linting errors
   - Architecture tests pass

Don't read docs/done.
