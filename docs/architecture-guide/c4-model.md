# The C4 Model: A Practical Framework

The C4 model provides a hierarchical approach to architectural diagrams that works for any architecture style.

---

## Overview

The C4 model consists of four levels of abstraction, each serving different audiences and purposes:

```
┌─────────────────────────────────────────────────────────────────────┐
│  LEVEL 1: SYSTEM CONTEXT                                            │
│  ────────────────────────                                           │
│  Scope: The system as a whole                                       │
│  Audience: Everyone (technical and non-technical)                   │
│  Shows: System + external users + external systems                  │
│                                                                     │
│         ┌──────┐          ┌─────────────────┐         ┌──────────┐ │
│         │ User │─────────▶│   Your System   │◀───────▶│ External │ │
│         └──────┘          │   [Software]    │         │  System  │ │
│                           └─────────────────┘         └──────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LEVEL 2: CONTAINER DIAGRAM                                         │
│  ──────────────────────────                                         │
│  Scope: Inside the system boundary                                  │
│  Audience: Technical people                                         │
│  Shows: Applications, databases, message queues, etc.               │
│                                                                     │
│    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│    │   Web    │───▶│   API    │───▶│ Database │    │  Queue   │   │
│    │   App    │    │  Server  │    │          │    │          │   │
│    │ [React]  │    │ [Spring] │    │[Postgres]│    │ [Kafka]  │   │
│    └──────────┘    └──────────┘    └──────────┘    └──────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LEVEL 3: COMPONENT DIAGRAM                                         │
│  ──────────────────────────                                         │
│  Scope: Inside a single container                                   │
│  Audience: Developers working on that container                     │
│  Shows: Major structural building blocks (modules, packages)        │
│                                                                     │
│    ┌──────────────────────────────────────────────────────────┐    │
│    │                    API Server                             │    │
│    │  ┌────────────┐  ┌────────────┐  ┌────────────┐         │    │
│    │  │ Controller │─▶│  Service   │─▶│ Repository │         │    │
│    │  └────────────┘  └────────────┘  └────────────┘         │    │
│    └──────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  LEVEL 4: CODE DIAGRAM (Use Sparingly)                              │
│  ─────────────────────────────────────                              │
│  Scope: Inside a component                                          │
│  Audience: Developers needing detailed understanding                │
│  Shows: Classes, interfaces, relationships                          │
│  Note: Often auto-generated; avoid manual maintenance               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Level 1: System Context Diagram

### Purpose
Shows the big picture of how your system fits into the world around it.

### What to Include
- Your system (as a single box)
- Users/actors who interact with your system
- External systems your system depends on or integrates with
- High-level relationships between these elements

### Audience
- Everyone: developers, architects, business stakeholders, operations

### When to Update
- When new integrations are added
- When user types change
- When system boundaries shift

---

## Level 2: Container Diagram

### Purpose
Zooms into your system to show the high-level technical building blocks.

### What to Include
- Applications (web apps, mobile apps, desktop apps)
- APIs/services
- Databases
- Message queues/brokers
- File systems
- Technology choices for each container

### Audience
- Technical people: developers, architects, DevOps

### When to Update
- When new services/applications are added
- When databases are added or changed
- When new message queues or other infrastructure is introduced

---

## Level 3: Component Diagram

### Purpose
Zooms into a single container to show its internal structure.

### What to Include
- Major structural building blocks (modules, packages, namespaces)
- Key components and their responsibilities
- Relationships and dependencies between components

### Audience
- Developers working on that specific container

### When to Update
- When module structure changes
- When new major components are added
- When dependencies between components change

---

## Level 4: Code Diagram

### Purpose
Shows class-level detail for complex components.

### What to Include
- Classes and interfaces
- Relationships (inheritance, composition, dependencies)
- Key methods and attributes (optional)

### Important Notes
- **Use sparingly** — most code-level detail is better understood by reading the code
- **Prefer auto-generation** — tools like IDE plugins can generate these on demand
- **Focus on complex areas** — only document code structure where it's genuinely helpful

### Audience
- Developers needing detailed understanding of specific components

### When to Update
- Auto-generate when needed rather than maintaining manually

---

## Best Practices

### Do
- Start from Level 1 and zoom in as needed
- Keep diagrams at a consistent level of abstraction
- Include a key/legend explaining notation
- Add brief descriptions to elements
- Store diagram source in version control

### Don't
- Mix abstraction levels in a single diagram
- Include too much detail at higher levels
- Create diagrams that nobody will maintain
- Forget to update diagrams when architecture changes

---

## Tools for C4 Diagrams

| Tool | Description |
|------|-------------|
| **Structurizr** | Purpose-built for C4 with DSL support |
| **PlantUML** | C4 extension available |
| **Mermaid** | Basic C4 support |
| **Draw.io** | C4 shape libraries available |

See [Tools](./tools.md) for more details on diagram tooling.
