# Quick Reference Checklists

Use these checklists to assess and track your documentation completeness.

---

## Minimum Viable Documentation

The bare minimum for any project.

- [ ] README with project overview and setup
- [ ] System Context Diagram (C4-L1)
- [ ] Container Diagram (C4-L2)
- [ ] Package structure documented
- [ ] Domain model / Class diagram
- [ ] ERD for database
- [ ] At least 3 ADRs (framework, database, key pattern)
- [ ] API contract (OpenAPI)
- [ ] One sequence diagram for main flow

### When Complete

You should be able to:
- Explain what the system does to a new team member
- Show the major technical components
- Understand why key technology choices were made
- Know the database structure
- Understand the main API

---

## Production-Ready Documentation

Required before going to production.

All of the above, plus:

- [ ] Component Diagram (C4-L3)
- [ ] Deployment Diagram
- [ ] Environment Matrix
- [ ] Runbook
- [ ] Monitoring Strategy
- [ ] Authentication/Authorization Flow
- [ ] Threat Model
- [ ] Disaster Recovery Plan

### When Complete

You should be able to:
- Deploy to any environment
- Handle common operational scenarios
- Respond to security questions
- Recover from failures
- Monitor system health

---

## Enterprise-Grade Documentation

For large organizations with compliance requirements.

All of the above, plus:

- [ ] Complete ADR history
- [ ] Data Flow Diagrams
- [ ] State Diagrams (where applicable)
- [ ] Event Catalog (if event-driven)
- [ ] Security Controls Matrix
- [ ] Technical Debt Register
- [ ] Capacity Planning Document
- [ ] SLA/SLO Documentation

### When Complete

You should be able to:
- Pass compliance audits
- Plan capacity for growth
- Track and manage technical debt
- Guarantee service levels
- Trace data through the system

---

## New Project Checklist

Starting a new project? Complete these in order.

### Week 1: Foundation

- [ ] Create project README
- [ ] Write system context document
- [ ] Create C4-L1 diagram
- [ ] Write first ADR (tech stack)
- [ ] Set up documentation folder structure

### Week 2-4: Architecture

- [ ] Create C4-L2 diagram
- [ ] Document package structure
- [ ] Create initial domain model
- [ ] Write ADRs for major decisions
- [ ] Create initial ERD

### Before First Release

- [ ] Create C4-L3 diagrams for key components
- [ ] Complete API contract
- [ ] Write key sequence diagrams
- [ ] Create deployment diagram
- [ ] Write initial runbook

---

## Documentation Review Checklist

Use during quarterly reviews.

### Accuracy

- [ ] C4 diagrams match actual architecture
- [ ] ERD matches current schema
- [ ] API docs match implementation
- [ ] ADRs reflect current status (not superseded)
- [ ] Runbook procedures are valid

### Completeness

- [ ] New components are documented
- [ ] Recent decisions have ADRs
- [ ] New integrations are in context diagram
- [ ] New environments are in environment matrix

### Quality

- [ ] Documents are findable (good organization)
- [ ] Links between documents work
- [ ] No duplicate/conflicting information
- [ ] Appropriate detail for audience

### Maintenance

- [ ] Last review date is recorded
- [ ] Owners are assigned
- [ ] Stale documents are flagged or removed

---

## Onboarding Documentation Checklist

Can a new team member find:

### Day 1

- [ ] Project overview (what and why)
- [ ] Development environment setup
- [ ] How to run the application locally
- [ ] How to run tests

### Week 1

- [ ] System architecture overview (C4-L1, L2)
- [ ] Key technology decisions (ADRs)
- [ ] Code organization (package structure)
- [ ] Main flow through the system

### Month 1

- [ ] Component details (C4-L3)
- [ ] Database structure (ERD)
- [ ] API documentation
- [ ] Deployment process
- [ ] How to contribute

---

## ADR Checklist

For each significant decision:

- [ ] Context is clearly explained
- [ ] Decision is stated unambiguously
- [ ] Consequences (positive, negative, neutral) are listed
- [ ] Alternatives were considered and documented
- [ ] Status is accurate (proposed/accepted/deprecated)
- [ ] References are included
- [ ] Linked from relevant architecture docs

---

## Diagram Checklist

For each diagram:

- [ ] Title clearly describes the diagram
- [ ] Legend explains notation used
- [ ] Elements have brief descriptions
- [ ] Relationships are labeled
- [ ] Appropriate level of detail for audience
- [ ] Linked to/from related documents
- [ ] Source is in version control (if diagrams-as-code)
- [ ] Last updated date is visible

---

## API Documentation Checklist

- [ ] All endpoints documented
- [ ] Request/response schemas defined
- [ ] Authentication requirements clear
- [ ] Error responses documented
- [ ] Examples provided
- [ ] Versioning strategy documented
- [ ] Rate limits documented (if applicable)
- [ ] Generated from code or kept in sync

---

## Runbook Checklist

- [ ] Deployment procedure
- [ ] Rollback procedure
- [ ] How to check system health
- [ ] Common issues and solutions
- [ ] Escalation contacts
- [ ] Log locations
- [ ] How to access environments
- [ ] Database operations (backup, restore)
- [ ] Incident response steps

---

## Security Documentation Checklist

- [ ] Threat model exists
- [ ] Authentication flow documented
- [ ] Authorization model documented
- [ ] Data classification complete
- [ ] Sensitive data handling documented
- [ ] Security controls mapped to requirements
- [ ] Dependency update policy defined
- [ ] Incident response plan exists

---

## Quick Self-Assessment

Score your documentation (0-2 points each):

| Area | 0 (Missing) | 1 (Partial) | 2 (Complete) | Score |
|------|-------------|-------------|--------------|-------|
| System Context | | | | |
| C4 Diagrams | | | | |
| ADRs | | | | |
| API Docs | | | | |
| ERD | | | | |
| Deployment Docs | | | | |
| Runbook | | | | |
| Security Docs | | | | |
| **Total** | | | | /16 |

### Interpretation

- **0-4**: Critical gaps, high risk
- **5-8**: Basic coverage, needs improvement
- **9-12**: Good coverage, some gaps
- **13-16**: Comprehensive documentation

---

## Monthly Maintenance Tasks

- [ ] Review and update risk register
- [ ] Check for stale ADRs
- [ ] Update technical debt register
- [ ] Review monitoring alerts and thresholds
- [ ] Update capacity projections

## Quarterly Maintenance Tasks

- [ ] Full documentation review
- [ ] Update all C4 diagrams
- [ ] Review and update technology radar
- [ ] Validate disaster recovery plan
- [ ] Review security documentation
- [ ] Update onboarding materials
