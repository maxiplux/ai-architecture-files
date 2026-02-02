---
description: Apply emergency hotfix outside normal phase flow
---
## Hotfix Protocol

⚠️ Use only for critical bugs blocking progress

1. **Document the issue:**
   - What is broken?
   - What is the impact?
   - Which task/requirement is affected?

2. **Create minimal fix:**
   - Fix ONLY the immediate issue
   - Do NOT scope creep
   - Add test covering the bug

3. **Update tracking:**
   - Add note to tasks.md "Notes & Decisions" section
   - Mark affected task with [!] if incomplete
   - Document in Blocking Issues Log if applicable

4. **Verify:**
   - Run affected tests
   - Run build verification

5. **Resume normal flow** after hotfix is stable

Don't read docs/done.
