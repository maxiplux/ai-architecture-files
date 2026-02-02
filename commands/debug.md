---
description: Debug and fix failing tests/builds
---
## Troubleshooting Protocol

1. Run the build/test command and capture the error
2. Identify the failing component:
   - Which test/file is failing?
   - What is the error message?
   - What was the last change made?

3. Trace back to requirements:
   - Which task does this relate to?
   - What are the acceptance criteria?

4. Analyze and fix:
   - Check if implementation matches plan.md specification
   - Check if test expectations match requirements.md
   - Fix the root cause (not symptoms)

5. Verify fix:
   - Re-run failing test
   - Run related tests to avoid regression

6. Document in tasks.md Notes section if significant

Don't read docs/done.
