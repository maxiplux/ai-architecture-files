# Diagram Selection by Architecture Style

Different architecture styles benefit from different types of diagrams. This guide maps diagram importance to common architecture patterns.

---

## Layered Architecture

Traditional n-tier architecture with horizontal layers.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Component Diagram (showing layers)** | Critical | Show layer boundaries and dependencies |
| **Package Structure** | Critical | Demonstrate layer organization in code |
| **Class Diagram (per layer)** | High | Show structure within each layer |
| **Sequence Diagram (cross-layer)** | High | Show how requests flow through layers |
| **Dependency Diagram** | Medium | Verify dependencies flow downward |

### Key Documentation Focus

- Layer responsibilities and boundaries
- Allowed dependencies between layers
- Cross-cutting concerns handling
- Package naming conventions

### Example Structure

```
┌─────────────────────────────────────────┐
│           Presentation Layer            │
│         (Controllers, Views)            │
├─────────────────────────────────────────┤
│           Application Layer             │
│         (Services, Use Cases)           │
├─────────────────────────────────────────┤
│            Domain Layer                 │
│       (Entities, Business Logic)        │
├─────────────────────────────────────────┤
│         Infrastructure Layer            │
│    (Repositories, External Services)    │
└─────────────────────────────────────────┘
```

---

## Hexagonal / Clean Architecture

Architecture emphasizing dependency inversion and ports & adapters.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Component Diagram (ports & adapters)** | Critical | Show ports, adapters, and core |
| **Dependency Flow Diagram** | Critical | Prove dependencies point inward |
| **Package Structure (showing boundaries)** | Critical | Demonstrate boundary enforcement |
| **Sequence Diagram (showing dependency direction)** | High | Show how adapters use ports |
| **Class Diagram (interfaces/ports)** | High | Document port contracts |

### Key Documentation Focus

- Port definitions and contracts
- Adapter implementations
- Dependency direction rules
- How external concerns are isolated

### Example Structure

```
┌───────────────────────────────────────────────────────┐
│                      Adapters                         │
│  ┌─────────────┐                    ┌─────────────┐  │
│  │   Web API   │                    │  Database   │  │
│  │  (Driving)  │                    │ (Driven)    │  │
│  └──────┬──────┘                    └──────┬──────┘  │
│         │                                  │         │
│         ▼                                  ▼         │
│  ┌─────────────┐  ┌───────────┐  ┌─────────────┐    │
│  │   Input     │  │           │  │   Output    │    │
│  │   Ports     │──│   Core    │──│   Ports     │    │
│  │(Interfaces) │  │ (Domain)  │  │(Interfaces) │    │
│  └─────────────┘  └───────────┘  └─────────────┘    │
└───────────────────────────────────────────────────────┘
```

---

## Microservices

Distributed system with independently deployable services.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Container Diagram (service map)** | Critical | Overview of all services |
| **Service Communication Diagram** | Critical | How services interact |
| **Data Ownership Map** | Critical | Which service owns which data |
| **Deployment Diagram** | Critical | Infrastructure topology |
| **Event Flow Diagram** | High | If using events for communication |
| **Saga/Choreography Diagrams** | High | For distributed transactions |
| **Network Diagram** | High | Service mesh, API gateway |
| **Sequence Diagrams (cross-service)** | Medium | Key business flows |

### Key Documentation Focus

- Service boundaries and responsibilities
- Data ownership and consistency boundaries
- Communication patterns (sync vs async)
- Service discovery and routing
- Failure handling and resilience

### Example Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                         API Gateway                               │
└───────────────────────────────┬──────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│    User       │      │    Order      │      │   Inventory   │
│   Service     │      │   Service     │      │   Service     │
├───────────────┤      ├───────────────┤      ├───────────────┤
│   User DB     │      │   Order DB    │      │  Inventory DB │
└───────────────┘      └───────┬───────┘      └───────────────┘
                               │
                               ▼
                    ┌───────────────────┐
                    │   Message Queue   │
                    │     (Events)      │
                    └───────────────────┘
```

---

## Event-Driven Architecture

System where components communicate through events.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Event Flow Diagram** | Critical | How events flow through system |
| **Event Catalog** | Critical | All events with schemas |
| **Sequence Diagram (async flows)** | Critical | Show event production and consumption |
| **State Diagrams** | High | Event-driven state transitions |
| **Container Diagram** | High | Producers, consumers, brokers |
| **Data Consistency Diagram** | High | Eventual consistency boundaries |

### Key Documentation Focus

- Event definitions and schemas
- Event producers and consumers
- Event ordering and guarantees
- Eventual consistency handling
- Idempotency requirements

### Example Structure

```
┌─────────────┐                              ┌─────────────┐
│  Producer   │                              │  Consumer   │
│  Service A  │──┐                      ┌───▶│  Service X  │
└─────────────┘  │                      │    └─────────────┘
                 │    ┌────────────┐    │
┌─────────────┐  │    │            │    │    ┌─────────────┐
│  Producer   │──┼───▶│   Event    │────┼───▶│  Consumer   │
│  Service B  │  │    │   Broker   │    │    │  Service Y  │
└─────────────┘  │    │            │    │    └─────────────┘
                 │    └────────────┘    │
┌─────────────┐  │                      │    ┌─────────────┐
│  Producer   │──┘                      └───▶│  Consumer   │
│  Service C  │                              │  Service Z  │
└─────────────┘                              └─────────────┘
```

---

## Monolithic Architecture

Single deployable unit containing all functionality.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Component Diagram** | Critical | Internal module structure |
| **Package Structure** | Critical | Code organization |
| **Class Diagram (per module)** | High | Module internals |
| **Sequence Diagrams** | High | Key flows within the monolith |
| **Database ERD** | High | Single database schema |
| **Deployment Diagram** | Medium | Simpler than distributed |

### Key Documentation Focus

- Module boundaries within the monolith
- Internal coupling management
- Database schema organization
- Potential future decomposition points

---

## CQRS (Command Query Responsibility Segregation)

Separate models for reading and writing data.

| Diagram | Importance | Notes |
|---------|------------|-------|
| **Component Diagram (read/write split)** | Critical | Show command and query sides |
| **Data Flow Diagram** | Critical | How data flows between sides |
| **Sequence Diagrams (commands)** | High | Command processing flow |
| **Sequence Diagrams (queries)** | High | Query processing flow |
| **Event Flow (if event-sourced)** | High | Event projection |
| **Data Model (both sides)** | High | Write model vs read model |

### Key Documentation Focus

- Command and query separation
- Synchronization mechanisms
- Consistency guarantees
- Read model projection logic

### Example Structure

```
                    ┌─────────────────┐
                    │     Client      │
                    └────────┬────────┘
                             │
            ┌────────────────┴────────────────┐
            │                                 │
            ▼                                 ▼
    ┌───────────────┐                 ┌───────────────┐
    │   Command     │                 │    Query      │
    │   Handler     │                 │   Handler     │
    └───────┬───────┘                 └───────┬───────┘
            │                                 │
            ▼                                 ▼
    ┌───────────────┐    sync/async   ┌───────────────┐
    │  Write Model  │────────────────▶│  Read Model   │
    │   (Source)    │                 │  (Projection) │
    └───────────────┘                 └───────────────┘
```

---

## Diagram Selection Matrix

| Diagram Type | Layered | Hexagonal | Microservices | Event-Driven | Monolith | CQRS |
|--------------|:-------:|:---------:|:-------------:|:------------:|:--------:|:----:|
| C4-L1 (Context) | ✓ | ✓ | ✓✓ | ✓ | ✓ | ✓ |
| C4-L2 (Container) | ✓ | ✓ | ✓✓✓ | ✓✓ | ✓ | ✓✓ |
| C4-L3 (Component) | ✓✓✓ | ✓✓✓ | ✓ | ✓ | ✓✓✓ | ✓✓ |
| Package Structure | ✓✓✓ | ✓✓✓ | ✓ | ✓ | ✓✓✓ | ✓✓ |
| Class Diagram | ✓✓ | ✓✓ | ✓ | ✓ | ✓✓ | ✓ |
| Sequence Diagram | ✓✓ | ✓✓ | ✓✓ | ✓✓✓ | ✓✓ | ✓✓✓ |
| Data Flow | ✓ | ✓ | ✓✓ | ✓✓✓ | ✓ | ✓✓✓ |
| Deployment | ✓ | ✓ | ✓✓✓ | ✓✓ | ✓ | ✓✓ |
| Event Catalog | - | - | ✓✓ | ✓✓✓ | - | ✓✓ |
| State Diagram | ✓ | ✓ | ✓ | ✓✓✓ | ✓ | ✓✓ |

Legend: ✓ = Useful, ✓✓ = Important, ✓✓✓ = Critical, - = Not applicable
