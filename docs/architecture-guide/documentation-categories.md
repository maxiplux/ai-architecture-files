# Documentation Categories

This document describes the five main categories of architectural documentation, organized by purpose.

---

## 1. Strategic Documentation (The "Why")

These documents capture business context and high-level decisions.

| Document | Purpose | Audience | Update Frequency |
|----------|---------|----------|------------------|
| **Architecture Decision Records (ADRs)** | Capture significant decisions with context and consequences | Architects, Tech Leads, Future Maintainers | Per decision |
| **System Context Document** | Business purpose, stakeholders, constraints | Everyone | Quarterly review |
| **Quality Attribute Requirements** | Non-functional requirements (performance, security, scalability) | Architects, DevOps | Per release |
| **Technology Radar** | Approved technologies and their status (adopt, trial, hold) | All developers | Quarterly |
| **Risk Register** | Known architectural risks and mitigations | Leadership, Architects | Monthly |

---

## 2. Structural Documentation (The "What")

These documents describe the system's structure at various levels.

| Document/Diagram | Purpose | Audience | Update Frequency |
|------------------|---------|----------|------------------|
| **System Context Diagram (C4-L1)** | System boundaries and external actors | Everyone | When integrations change |
| **Container Diagram (C4-L2)** | High-level technical building blocks | Developers, DevOps | When services/apps added |
| **Component Diagram (C4-L3)** | Internal structure of containers | Developers | When module structure changes |
| **Code Diagram (C4-L4)** | Class-level detail (use sparingly) | Developers | Auto-generated preferred |
| **Package Structure** | Code organization conventions | Developers | When conventions change |
| **Domain Model** | Core business entities and relationships | Developers, Business Analysts | When domain evolves |
| **ERD / Data Model** | Database schema | Developers, DBAs | Per schema migration |

---

## 3. Behavioral Documentation (The "How")

These documents explain runtime behavior and data flow.

| Document/Diagram | Purpose | Audience | Update Frequency |
|------------------|---------|----------|------------------|
| **Sequence Diagrams** | Interaction flows for key scenarios | Developers | When flow logic changes |
| **Data Flow Diagrams** | How data moves through the system | Developers, Security, Compliance | When data paths change |
| **State Diagrams** | Lifecycle of stateful entities | Developers | When states/transitions change |
| **Activity Diagrams** | Complex business process workflows | Business Analysts, Developers | When process changes |
| **API Contracts** | Interface specifications (OpenAPI/Swagger) | API consumers, Developers | Per API change |
| **Event Catalog** | Domain events and their schemas | Developers (event-driven systems) | Per event change |

---

## 4. Operational Documentation (The "Where" and "When")

These documents support deployment, monitoring, and operations.

| Document/Diagram | Purpose | Audience | Update Frequency |
|------------------|---------|----------|------------------|
| **Deployment Diagram** | Infrastructure topology | DevOps, Architects | Per infrastructure change |
| **Network Diagram** | Network zones, firewalls, traffic flow | DevOps, Security | Per network change |
| **Runbook** | Operational procedures (deploy, rollback, incidents) | DevOps, On-call | Per procedure change |
| **Monitoring Strategy** | What to monitor, alerting thresholds | DevOps | Per release |
| **Disaster Recovery Plan** | RTO/RPO, failover procedures | DevOps, Leadership | Annual review |
| **Environment Matrix** | Configuration differences across environments | DevOps, Developers | Per environment change |

---

## 5. Security Documentation (The "Who" and "What If")

These documents address security architecture and compliance.

| Document/Diagram | Purpose | Audience | Update Frequency |
|------------------|---------|----------|------------------|
| **Threat Model** | Attack vectors and mitigations | Security, Architects | Per major feature |
| **Authentication/Authorization Flow** | How identity and access work | Security, Developers | When auth changes |
| **Data Classification** | Sensitivity levels of data handled | Security, Compliance | Annual review |
| **Security Controls Matrix** | Controls mapped to requirements | Security, Auditors | Annual review |
| **Dependency Security Policy** | Rules for third-party dependencies | Developers | Annual review |

---

## Summary

| Category | Focus | Key Question |
|----------|-------|--------------|
| Strategic | Business context & decisions | Why? |
| Structural | System structure | What? |
| Behavioral | Runtime behavior | How? |
| Operational | Deployment & operations | Where & When? |
| Security | Security & compliance | Who & What If? |
