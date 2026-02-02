# ai-architecture-files

Project overview
- A collection of architecture guidance, templates, guides, and reference files for designing, documenting, and reviewing AI and software architectures.  
- Purpose: help architects, engineers, and reviewers produce consistent documentation (diagrams, ADRs, checklists) and follow standardized review practices.

Quick start
1. Clone the repo:
   git clone https://github.com/maxiplux/ai-architecture-files.git
2. Browse documentation:
   - Open docs/architecture-guide/README.md for the main architectural documentation guide.
   - Review mapstruct-guide.md for a detailed technology-specific guide included in this repo.
3. Contribute:
   - Read code-review-guide-lines.md to align PRs and reviews with repository expectations.
   - Use the Checklists in docs/architecture-guide to validate documentation completeness.

File structure (high-level)
- README.md — this file
- LICENSE — repository license (see file for terms)
- code-review-guide-lines.md — repository code review expectations and workflow
- mapstruct-guide.md — a large, detailed technology guide included in the repo
- docs/
  - architecture-guide/
    - README.md — Architectural Documentation Guide (overview and philosophy)
    - documentation-categories.md
    - c4-model.md
    - adr-guide.md
    - project-phases.md
    - audience-guide.md
    - essential-documentation.md
    - architecture-styles.md
    - tools.md
    - anti-patterns.md
    - checklists.md

### Workflow Phases

| Phase | Document | Purpose | Output |
|-------|----------|---------|--------|
| **1. Discovery** | `transform-requirements.md` | Extract and structure requirements from stakeholders | `docs/requirements.md` |
| **2. Validation** | `clarify-questions.md` | Identify gaps and validate understanding with stakeholders | Requirements refinement |
| **3. Planning** | `plan.md` | Design overall architecture and approach | `docs/plan.md` |
| **4. Requirement-Plan Sync** | `loop-req.md` | Cross-validate requirements against plan for consistency | Validation report |
| **5. Task Breakdown** | `task.md` | Decompose plan into actionable development tasks | `docs/tasks.md` |
| **6. Plan-Task Alignment** | `task-check.md` | Verify tasks align with plan and cover all requirements | Alignment matrix |
| **7. Implementation** | `auto.md` | Execute all phases: code, docs, diagrams, ADRs | Implementation artifacts |
| **8. Test Generation** | `test-gen.md` | Generate missing test cases for gaps in coverage | Test suite |
| **9. Code Review** | `review.md` | Review implementation, documentation, and ADRs | Review findings |
| **10. Quality Assurance** | `qa.md` | Final validation and sign-off | QA report |

### How to Use This Workflow

- **Start at Phase 1** for new projects or major features
- **Jump to Phase 5** if you already have solid requirements and architecture
- **Use Phases 8-10** as gates before merging to main/production
- **Loop back to Phase 4-6** if new requirements emerge during implementation

### Quick Reference


Usage examples
- Produce an Architecture Decision Record (ADR)
  1. Read docs/architecture-guide/adr-guide.md to see the ADR template and examples.
  2. Create a new ADR file under docs/adr/ or a project-specific docs/ folder with the recommended template.
  3. Link the ADR from system documentation and include the rationale and alternatives.

- Create and review architecture documentation
  1. Start with docs/architecture-guide/documentation-categories.md to choose which documents you need.
  2. Use docs/architecture-guide/c4-model.md for diagram guidance (context, container, component, code).
  3. Validate with docs/architecture-guide/checklists.md before PR.

- Technology-specific reference (example)
  - Consult mapstruct-guide.md for mapping patterns and examples relevant to the codebase or integrations that use MapStruct-like mapping approaches.

License
- See LICENSE at the repository root for the full license text and permissions.

Contributing
- Read code-review-guide-lines.md before opening a pull request — it contains expectations for review, commit messages, and testing.
- Use small, focused PRs that:
  - Explain why the change is needed (link related ADRs or issue).
  - Include documentation updates for any design or API changes.
  - Follow the doc structure and include diagrams or references as appropriate.
- When adding documentation:
  - Prefer placing project-specific docs in a docs/ folder inside the project.
  - Link to central docs/architecture-guide where relevant.
- If you want a contribution checklist or PR template added, open an issue describing the desired template.

Contact
- Repository owner: https://github.com/maxiplux
- For questions or contributions, open an issue in this repository or create a pull request.

Notes
- This README is a concise entry point. The docs/architecture-guide contains the detailed guide, checklists, and templates you will likely use for authoring and reviewing architecture artifacts.
