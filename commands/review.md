---
description: Review code against requirements and plan
---
## Code Review Protocol

For the current phase/epic, verify:

### Compliance Checks
- [ ] Code matches plan.md directory structure
- [ ] Public APIs match plan.md Section 4 contracts
- [ ] Error handling matches plan.md Section 4.3
- [ ] Requirement IDs (FR-xxx) referenced in code comments

### Quality Checks
- [ ] No hardcoded values (use config)
- [ ] Proper error handling (no swallowed exceptions)
- [ ] Logging at appropriate levels
- [ ] No TODO/FIXME without linked task

### Test Checks
- [ ] Unit tests exist for business logic
- [ ] Integration tests exist for APIs
- [ ] Edge cases covered
- [ ] Test names are descriptive

### Output
Generate review report with:
- ✅ Compliant items
- ⚠️ Warnings (minor issues)
- ❌ Blockers (must fix before proceeding)

Don't read docs/done.
```

---

## 📋 Suggested Workflow Order
```
1. transform-requeriments.md  → Creates docs/requirements.md
2. clarify-questions.md       → Validates understanding
3. plan.md                    → Creates docs/plan.md
4. loop-req.md (NEW)          → Validates requirements → plan
5. task.md                    → Creates docs/tasks.md
6. task-check.md              → Validates plan → tasks
7. auto.md                    → Implements all phases
8. test-gen.md (NEW)          → Generates missing tests
9. review.md (NEW)            → Reviews implementation
10. qa.md                     → Final QA validation
