# Documentation by Project Phase

This guide outlines which documentation to prioritize at each phase of a project's lifecycle.

---

## Phase 1: Project Inception

Focus on establishing context and making foundational decisions.

| Document | Priority | Notes |
|----------|----------|-------|
| **System Context Document** | Required | Business purpose, stakeholders, constraints |
| **Quality Attribute Requirements** | Required | Non-functional requirements |
| **Initial ADRs (tech stack, patterns)** | Required | Framework, database, key patterns |
| **System Context Diagram (C4-L1)** | Required | Big picture view |
| **Initial Risk Register** | Recommended | Known architectural risks |

### Key Questions to Answer

- What problem are we solving?
- Who are the stakeholders?
- What are the key quality attributes (performance, security, scalability)?
- What are the major constraints?
- What technologies will we use and why?

---

## Phase 2: Design & Early Development

Focus on establishing the architecture and starting implementation.

| Document | Priority | Notes |
|----------|----------|-------|
| **Container Diagram (C4-L2)** | Required | High-level technical components |
| **Component Diagram (C4-L3)** | Required | Internal structure of key containers |
| **Domain Model / Class Diagram** | Required | Core business entities |
| **Package Structure** | Required | Code organization conventions |
| **ERD (initial)** | Required | Database schema |
| **Key Sequence Diagrams** | Required | Main interaction flows |
| **API Contract (draft)** | Required | Interface specifications |
| **ADRs for major decisions** | Required | Document decisions as made |
| **Threat Model (initial)** | Recommended | Security considerations |

### Key Questions to Answer

- What are the major building blocks?
- How do components interact?
- What does the data model look like?
- What are the main flows through the system?
- What are the security implications?

---

## Phase 3: Active Development

Focus on maintaining documentation as the system evolves.

| Document | Priority | Notes |
|----------|----------|-------|
| **ADRs (ongoing)** | Required | Document new decisions |
| **API Contract (maintained)** | Required | Keep in sync with implementation |
| **ERD (updated with migrations)** | Required | Reflect schema changes |
| **Sequence Diagrams (key flows)** | Recommended | Document complex interactions |
| **State Diagrams (if applicable)** | Recommended | For stateful entities |
| **Data Flow Diagrams** | Recommended | How data moves through the system |

### Key Activities

- Update ADRs when significant decisions are made
- Keep API contracts in sync with implementation
- Update ERD with each schema migration
- Add sequence diagrams for new complex flows
- Review and update existing documentation

---

## Phase 4: Pre-Production

Focus on operational readiness.

| Document | Priority | Notes |
|----------|----------|-------|
| **Deployment Diagram** | Required | Infrastructure topology |
| **Environment Matrix** | Required | Configuration differences |
| **Runbook** | Required | Operational procedures |
| **Monitoring Strategy** | Required | What to monitor, alerting |
| **Disaster Recovery Plan** | Required | RTO/RPO, failover procedures |
| **Security Documentation** | Required | Auth flows, threat model |
| **Performance Test Results** | Recommended | Baseline performance data |

### Key Questions to Answer

- How is the system deployed?
- How do environments differ?
- How do we handle incidents?
- What should we monitor?
- How do we recover from disasters?
- Is the system secure?

---

## Phase 5: Production & Maintenance

Focus on keeping documentation current and learning from operations.

| Document | Priority | Notes |
|----------|----------|-------|
| **All diagrams kept current** | Required | Reflect actual system |
| **ADRs for changes** | Required | Document evolution |
| **Incident Post-mortems** | Required | Learn from failures |
| **Capacity Planning** | Recommended | Future scaling needs |
| **Technical Debt Register** | Recommended | Track known issues |

### Key Activities

- Review and update documentation quarterly
- Create ADRs for significant changes
- Document incidents and lessons learned
- Track technical debt
- Plan for capacity and scaling

---

## Documentation Lifecycle Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DOCUMENTATION LIFECYCLE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INCEPTION        DESIGN          DEVELOPMENT     PRE-PROD       PRODUCTION │
│  ─────────        ──────          ───────────     ────────       ────────── │
│                                                                             │
│  Context Doc ────────────────────────────────────────────────────────────▶ │
│  ADRs        ────────────────────────────────────────────────────────────▶ │
│  C4-L1       ────────────────────────────────────────────────────────────▶ │
│              C4-L2, L3 ──────────────────────────────────────────────────▶ │
│              Domain Model ───────────────────────────────────────────────▶ │
│              ERD ────────────────────────────────────────────────────────▶ │
│              API Contract ───────────────────────────────────────────────▶ │
│              Sequence Diagrams ──────────────────────────────────────────▶ │
│                                       Deployment Diagram ────────────────▶ │
│                                       Runbook ───────────────────────────▶ │
│                                       Monitoring Strategy ───────────────▶ │
│                                       DR Plan ───────────────────────────▶ │
│                                                              Post-mortems ▶ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Phase Transition Checklist

### Inception → Design
- [ ] System context document complete
- [ ] Quality attributes defined
- [ ] Initial ADRs written (tech stack)
- [ ] System context diagram created
- [ ] Initial risks identified

### Design → Development
- [ ] C4 diagrams (L1-L3) complete
- [ ] Domain model documented
- [ ] Package structure defined
- [ ] Initial ERD created
- [ ] API contract drafted
- [ ] Key sequence diagrams created

### Development → Pre-Production
- [ ] All documentation updated
- [ ] Deployment diagram created
- [ ] Environment matrix documented
- [ ] Runbook written
- [ ] Monitoring strategy defined
- [ ] Security documentation complete

### Pre-Production → Production
- [ ] All documentation reviewed
- [ ] Runbook tested
- [ ] Monitoring verified
- [ ] DR plan tested
- [ ] Team trained on operations
