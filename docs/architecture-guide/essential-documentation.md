# Essential vs. Nice-to-Have Documentation

This guide defines what documentation is essential versus optional, with recommended folder structures.

---

## Always Required (Any Project)

Every project, regardless of size, should have these documents.

```
├── README.md
│   ├── Project overview
│   ├── Quick start / Setup
│   └── Links to detailed docs
│
├── docs/
│   ├── architecture/
│   │   ├── system-context.md          # Business context
│   │   ├── c4-diagrams.md             # L1, L2, L3 diagrams
│   │   ├── domain-model.md            # Core entities
│   │   └── package-structure.md       # Code organization
│   │
│   ├── adr/
│   │   ├── 0001-use-spring-boot.md
│   │   ├── 0002-postgresql-database.md
│   │   └── ...
│   │
│   ├── api/
│   │   └── openapi.yaml               # API contract
│   │
│   └── database/
│       └── erd.md                     # Entity-Relationship Diagram
```

### Checklist

- [ ] README with project overview and setup instructions
- [ ] System context document explaining business purpose
- [ ] C4 diagrams (at least L1 and L2)
- [ ] Domain model documenting core entities
- [ ] Package structure showing code organization
- [ ] At least 3 ADRs (framework, database, key pattern)
- [ ] API contract (OpenAPI/Swagger)
- [ ] ERD for database schema

---

## Required for Production Systems

Production systems need operational documentation in addition to the basics.

```
├── docs/
│   ├── operations/
│   │   ├── deployment.md              # Deployment diagram & process
│   │   ├── runbook.md                 # Operational procedures
│   │   ├── monitoring.md              # What to monitor
│   │   └── environments.md            # Environment matrix
│   │
│   └── security/
│       ├── threat-model.md
│       └── auth-flow.md
```

### Checklist

All of the above, plus:

- [ ] Deployment diagram showing infrastructure topology
- [ ] Runbook with operational procedures
- [ ] Monitoring strategy with alerting thresholds
- [ ] Environment matrix documenting configuration differences
- [ ] Threat model identifying security risks
- [ ] Authentication/authorization flow documentation

---

## Recommended for Complex Systems

Complex systems benefit from additional documentation.

```
├── docs/
│   ├── architecture/
│   │   ├── sequence-diagrams/         # Key flows
│   │   ├── data-flow.md
│   │   └── state-diagrams/            # Stateful entities
│   │
│   ├── integration/
│   │   └── event-catalog.md           # For event-driven
│   │
│   └── quality/
│       ├── performance-benchmarks.md
│       └── technical-debt.md
```

### Checklist

All of the above, plus:

- [ ] Sequence diagrams for complex flows
- [ ] Data flow diagrams showing data movement
- [ ] State diagrams for stateful entities
- [ ] Event catalog (for event-driven systems)
- [ ] Performance benchmarks and test results
- [ ] Technical debt register

---

## Complete Folder Structure

For a fully-documented enterprise system:

```
project/
├── README.md
│
├── docs/
│   ├── architecture/
│   │   ├── system-context.md
│   │   ├── c4-diagrams.md
│   │   ├── domain-model.md
│   │   ├── package-structure.md
│   │   ├── sequence-diagrams/
│   │   │   ├── user-registration.md
│   │   │   ├── order-processing.md
│   │   │   └── ...
│   │   ├── state-diagrams/
│   │   │   ├── order-lifecycle.md
│   │   │   └── ...
│   │   └── data-flow.md
│   │
│   ├── adr/
│   │   ├── README.md                  # Index of ADRs
│   │   ├── 0001-use-spring-boot.md
│   │   ├── 0002-postgresql-database.md
│   │   ├── 0003-jwt-authentication.md
│   │   └── ...
│   │
│   ├── api/
│   │   ├── openapi.yaml
│   │   └── postman-collection.json
│   │
│   ├── database/
│   │   ├── erd.md
│   │   └── migrations/
│   │       └── ...
│   │
│   ├── integration/
│   │   ├── event-catalog.md
│   │   └── external-systems.md
│   │
│   ├── operations/
│   │   ├── deployment.md
│   │   ├── runbook.md
│   │   ├── monitoring.md
│   │   ├── environments.md
│   │   └── disaster-recovery.md
│   │
│   ├── security/
│   │   ├── threat-model.md
│   │   ├── auth-flow.md
│   │   ├── data-classification.md
│   │   └── security-controls.md
│   │
│   └── quality/
│       ├── performance-benchmarks.md
│       ├── technical-debt.md
│       └── sla-slo.md
│
└── src/
    └── ...
```

---

## Documentation Tiers

### Tier 1: Minimum Viable Documentation (MVP)

For small projects, proofs of concept, or early-stage startups.

| Document | Status |
|----------|--------|
| README.md | Required |
| System Context (brief) | Required |
| C4-L1 Diagram | Required |
| 1-2 Key ADRs | Required |
| ERD (if using database) | Required |

### Tier 2: Production-Ready Documentation

For production systems with a dedicated team.

| Document | Status |
|----------|--------|
| All Tier 1 | Required |
| C4-L2 and L3 Diagrams | Required |
| Domain Model | Required |
| Package Structure | Required |
| API Contract | Required |
| 5+ ADRs | Required |
| Deployment Diagram | Required |
| Runbook | Required |
| Monitoring Strategy | Required |
| Security Documentation | Required |

### Tier 3: Enterprise-Grade Documentation

For large organizations with compliance requirements.

| Document | Status |
|----------|--------|
| All Tier 2 | Required |
| Complete ADR History | Required |
| Sequence Diagrams | Required |
| Data Flow Diagrams | Required |
| State Diagrams | As needed |
| Event Catalog | If event-driven |
| Threat Model | Required |
| Data Classification | Required |
| Security Controls Matrix | Required |
| Disaster Recovery Plan | Required |
| Technical Debt Register | Recommended |
| Performance Benchmarks | Recommended |
| SLA/SLO Documentation | Required |

---

## When to Add Documentation

### Add Immediately

- New ADR when making significant decisions
- API contract changes when API changes
- ERD updates when schema changes

### Add Before Production

- Deployment documentation
- Runbook
- Monitoring strategy
- Security documentation

### Add When Complexity Warrants

- Sequence diagrams for complex flows
- State diagrams for stateful entities
- Data flow diagrams when data paths are complex

### Review Periodically

- System context document (quarterly)
- C4 diagrams (quarterly)
- Risk register (monthly)
- Technical debt register (monthly)
