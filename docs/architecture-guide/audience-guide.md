# Documentation by Audience

Different stakeholders need different documentation. This guide organizes documentation by target audience.

---

## For New Team Members (Onboarding Package)

New developers need to quickly understand the system's purpose, structure, and conventions.

### Essential Documents

1. **System Context Document** — Business purpose and why the system exists
2. **System Context Diagram (C4-L1)** — Big picture of the system
3. **Container Diagram (C4-L2)** — Technical building blocks
4. **Package Structure** — How code is organized
5. **Key ADRs (top 5-10)** — Most important decisions and their rationale
6. **Development Setup Guide** — How to get the system running locally
7. **One Key Sequence Diagram** — Main flow through the system

### Goals

- Understand the business context
- See the high-level architecture
- Know how to navigate the codebase
- Understand key decisions
- Get productive quickly

### Suggested Reading Order

```
1. System Context Document (why we exist)
       ↓
2. C4-L1 Diagram (big picture)
       ↓
3. C4-L2 Diagram (technical overview)
       ↓
4. Development Setup Guide (hands-on)
       ↓
5. Package Structure (navigate code)
       ↓
6. Key ADRs (understand decisions)
       ↓
7. Main Sequence Diagram (how it works)
```

---

## For Other Development Teams (Integration Partners)

Teams integrating with your system need to understand interfaces and contracts.

### Essential Documents

1. **System Context Diagram** — What the system does at a high level
2. **API Contract / OpenAPI Spec** — Detailed interface specification
3. **Authentication Flow** — How to authenticate requests
4. **Event Catalog** — Events published (if event-driven)
5. **SLA/SLO Documentation** — Expected availability and performance

### Goals

- Understand how to integrate
- Know the API contracts
- Understand authentication requirements
- Know what events to expect (if applicable)
- Understand reliability expectations

### Key Questions to Answer

- What APIs are available?
- How do I authenticate?
- What data formats are used?
- What events can I subscribe to?
- What uptime can I expect?

---

## For DevOps/SRE

Operations teams need to understand deployment, monitoring, and incident response.

### Essential Documents

1. **Deployment Diagram** — Infrastructure topology
2. **Environment Matrix** — Differences between environments
3. **Runbook** — Operational procedures
4. **Monitoring Strategy** — What to monitor and alert on
5. **Network Diagram** — Network topology and security zones
6. **Disaster Recovery Plan** — Recovery procedures

### Goals

- Deploy and maintain the system
- Monitor system health
- Respond to incidents
- Plan capacity
- Recover from failures

### Key Questions to Answer

- How is the system deployed?
- What infrastructure is needed?
- What should be monitored?
- How do I handle incidents?
- How do I recover from disasters?

---

## For Security/Compliance

Security teams need to understand threats, data handling, and controls.

### Essential Documents

1. **Threat Model** — Attack vectors and mitigations
2. **Data Flow Diagram** — How data moves through the system
3. **Authentication/Authorization Flow** — Identity and access management
4. **Data Classification** — Sensitivity levels of data
5. **Security Controls Matrix** — Controls mapped to requirements
6. **Relevant ADRs** — Security-related decisions

### Goals

- Assess security posture
- Verify compliance
- Understand data handling
- Review access controls
- Audit the system

### Key Questions to Answer

- What are the security risks?
- How is data protected?
- How does authentication work?
- What data is collected and how is it classified?
- What controls are in place?

---

## For Leadership/Stakeholders

Leadership needs high-level understanding and risk visibility.

### Essential Documents

1. **System Context Document** — Business purpose and value
2. **System Context Diagram (C4-L1)** — Visual overview
3. **Risk Register** — Known risks and mitigations
4. **Key ADRs (business-impacting)** — Major technical decisions
5. **Quality Metrics Dashboard** — System health indicators

### Goals

- Understand business value
- Assess risks
- Make informed decisions
- Track system health
- Plan investments

### Key Questions to Answer

- What does this system do?
- What are the major risks?
- What key decisions were made?
- Is the system healthy?
- What investments are needed?

---

## Documentation Matrix

| Document | New Devs | Partners | DevOps | Security | Leadership |
|----------|:--------:|:--------:|:------:|:--------:|:----------:|
| System Context Doc | ✓ | | | | ✓ |
| C4-L1 Diagram | ✓ | ✓ | | | ✓ |
| C4-L2 Diagram | ✓ | | | | |
| C4-L3 Diagram | ✓ | | | | |
| Package Structure | ✓ | | | | |
| Key ADRs | ✓ | | | ✓ | ✓ |
| API Contract | | ✓ | | | |
| Auth Flow | | ✓ | | ✓ | |
| Event Catalog | | ✓ | | | |
| SLA/SLO Docs | | ✓ | ✓ | | |
| Deployment Diagram | | | ✓ | | |
| Environment Matrix | | | ✓ | | |
| Runbook | | | ✓ | | |
| Monitoring Strategy | | | ✓ | | |
| Network Diagram | | | ✓ | ✓ | |
| DR Plan | | | ✓ | | |
| Threat Model | | | | ✓ | |
| Data Flow Diagram | | | | ✓ | |
| Data Classification | | | | ✓ | |
| Security Controls | | | | ✓ | |
| Risk Register | | | | | ✓ |

---

## Tips for Audience-Focused Documentation

### Do

- **Know your audience** — Write for specific readers
- **Provide context** — Don't assume prior knowledge
- **Use appropriate detail** — Match detail level to audience needs
- **Link to more detail** — Let readers dive deeper if needed
- **Keep it current** — Outdated docs are worse than no docs

### Don't

- **Don't mix audiences** — Create separate docs for different needs
- **Don't over-explain** — DevOps doesn't need business context explained
- **Don't under-explain** — New developers need more context
- **Don't create silos** — Make docs discoverable across teams
