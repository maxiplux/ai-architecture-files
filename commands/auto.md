---
description: Implement task
---
## Phase Identification
1. Read docs/tasks.md and examine the Progress Summary table.
2. Identify the first phase where "Completed" < "Tasks Total" (has remaining tasks).
3. Verify all previous phases are 100% complete before proceeding.

## Implementation
4. Implement all tasks for the identified phase sequentially.
5. Mark each task as done [x] immediately upon completion.
6. Run the verification command after each task/epic if specified.

## Phase Transition
7. After completing ALL tasks in the current phase:
   - Update the Progress Summary (increment Completed, decrement Remaining)
   - Update Metadata: "Current Phase", "Current Epic", "Current Task" to next pending item
   - Continue to the next phase automatically

## Final Review
8. After all phases show Completed = Tasks Total:
   - Scan entire file for any unmarked tasks [ ]
   - Complete any missed tasks
   - Verify all checkpoints are marked [x]

## Constraints
- Don't read docs/done
- Complete phases in order (no skipping)
- Don't start Phase N+1 until Phase N is 100% complete
