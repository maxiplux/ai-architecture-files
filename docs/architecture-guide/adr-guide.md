# Architecture Decision Records (ADRs)

ADRs are arguably the most valuable documentation an architect produces. They capture the *why* behind decisions.

---

## Why ADRs Matter

- **Preserve context** — Decisions make sense when made but context fades over time
- **Enable learning** — New team members understand why things are the way they are
- **Prevent rehashing** — Avoid re-debating settled decisions without new information
- **Support evolution** — Know what assumptions to revisit when circumstances change

---

## ADR Template

```markdown
# ADR-{NUMBER}: {TITLE}

## Status
{Proposed | Accepted | Deprecated | Superseded by ADR-XXX}

## Context
What is the issue that we're seeing that is motivating this decision?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or more difficult because of this change?

### Positive
- ...

### Negative
- ...

### Neutral
- ...

## Alternatives Considered
What other options were evaluated?

### Option A: {Name}
- Pros: ...
- Cons: ...

### Option B: {Name}
- Pros: ...
- Cons: ...

## References
- Links to relevant documentation, discussions, or research
```

---

## ADR Status Lifecycle

```
Proposed → Accepted → [Active]
                   ↓
              Deprecated
                   ↓
           Superseded by ADR-XXX
```

| Status | Meaning |
|--------|---------|
| **Proposed** | Decision is under discussion |
| **Accepted** | Decision has been approved and is in effect |
| **Deprecated** | Decision is no longer relevant |
| **Superseded** | Decision has been replaced by a newer ADR |

---

## What to Document in ADRs

### Technical Decisions

- Choice of framework (Spring Boot vs Quarkus vs Micronaut)
- Database selection (PostgreSQL vs MongoDB vs both)
- Authentication strategy (OAuth2, JWT structure, session management)
- API versioning strategy
- Caching strategy
- Messaging patterns (sync vs async, event sourcing)
- Error handling approach
- Logging and observability strategy

### Architectural Decisions

- Module/service boundaries
- Communication patterns between services
- Data ownership and boundaries
- Deployment strategies
- Scaling approaches

### Process Decisions

- Testing strategies
- Code review requirements
- Documentation standards
- Release processes

---

## ADR Examples

### Example 1: Database Selection

```markdown
# ADR-001: Use PostgreSQL as Primary Database

## Status
Accepted

## Context
We need to select a primary database for our e-commerce platform.
The system needs to handle transactional data with ACID compliance,
support complex queries for reporting, and scale to millions of products.

## Decision
We will use PostgreSQL as our primary database.

## Consequences

### Positive
- Strong ACID compliance for transactional integrity
- Excellent support for complex queries and joins
- Rich ecosystem of tools and extensions
- Strong community support and documentation
- Team has existing PostgreSQL expertise

### Negative
- Horizontal scaling requires additional tooling (Citus, read replicas)
- Schema migrations require careful planning
- Not ideal for document-style data (may need supplementary store later)

### Neutral
- Standard operational practices apply
- Backup and recovery processes are well-established

## Alternatives Considered

### Option A: MongoDB
- Pros: Flexible schema, easy horizontal scaling, good for documents
- Cons: Weaker transactional guarantees, less suitable for complex joins

### Option B: MySQL
- Pros: Widely used, good performance, team familiarity
- Cons: Less feature-rich than PostgreSQL, licensing considerations

## References
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- Internal capacity planning document
```

### Example 2: API Versioning

```markdown
# ADR-002: URL Path Versioning for REST APIs

## Status
Accepted

## Context
We need a strategy for versioning our REST APIs to allow evolution
while maintaining backward compatibility for existing clients.

## Decision
We will use URL path versioning (e.g., /api/v1/resources, /api/v2/resources).

## Consequences

### Positive
- Clear and explicit version in every request
- Easy to route different versions to different deployments
- Simple to understand for API consumers
- Easy to deprecate old versions

### Negative
- URL changes between versions (not purely RESTful)
- Potential for code duplication between versions
- Clients must update URLs when migrating

### Neutral
- Standard approach used by many major APIs
- Documentation must clearly indicate version differences

## Alternatives Considered

### Option A: Header Versioning
- Pros: Clean URLs, more RESTful
- Cons: Harder to test in browser, less visible

### Option B: Query Parameter Versioning
- Pros: URLs remain similar, easy to test
- Cons: Can be cached incorrectly, feels hacky

## References
- API Design Guidelines document
- [REST API Versioning Best Practices](https://example.com)
```

---

## Best Practices

### Writing ADRs

1. **Write at decision time** — Don't wait; context fades quickly
2. **Be concise** — Focus on the essential information
3. **Include alternatives** — Show you considered other options
4. **Document consequences honestly** — Include negatives and trade-offs
5. **Link to related ADRs** — Show how decisions connect

### Managing ADRs

1. **Number sequentially** — ADR-001, ADR-002, etc.
2. **Never delete** — Mark as deprecated or superseded instead
3. **Store with code** — Keep in version control (e.g., `docs/adr/`)
4. **Review periodically** — Assumptions may need revisiting
5. **Make discoverable** — Index or link from main documentation

### ADR File Structure

```
docs/
└── adr/
    ├── README.md           # Index of all ADRs
    ├── 0001-use-postgresql.md
    ├── 0002-api-versioning.md
    ├── 0003-authentication-strategy.md
    └── ...
```

---

## Tools for ADRs

| Tool | Description |
|------|-------------|
| **adr-tools** | Command-line tools for managing ADRs |
| **Log4brains** | ADR management with web UI |
| **Markdown** | Simple, portable, version-controlled |

---

## Common Mistakes

1. **Writing ADRs after the fact** — Context is lost, decisions seem arbitrary
2. **Too much detail** — ADRs become hard to read and maintain
3. **No alternatives** — Looks like the decision wasn't thought through
4. **Missing consequences** — Especially negative ones
5. **Not updating status** — Deprecated decisions appear current
