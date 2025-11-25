# Documentation Anti-Patterns to Avoid

This guide identifies common documentation mistakes and how to avoid them.

---

## 1. Documentation Graveyards

### The Problem

Documents created once and never updated. They become increasingly inaccurate over time, eventually becoming misleading.

### Signs

- Last modified date is months or years old
- Content doesn't match current system
- Team members don't trust the documentation
- "Just ask someone" becomes the norm

### Solution

- **Link documentation updates to PR reviews** — Include docs in definition of done
- **Automate where possible** — Generate API docs from code
- **Review quarterly** — Schedule documentation reviews
- **Delete stale content** — Outdated docs are worse than no docs

### Prevention Checklist

- [ ] Documentation updates in PR template
- [ ] Quarterly review calendar reminders
- [ ] Clear ownership for each document
- [ ] Automated staleness detection

---

## 2. Over-Documentation

### The Problem

Documenting obvious things, volatile details, or information that's better found in code.

### Signs

- Documentation duplicates what code clearly shows
- Excessive detail on implementation that changes frequently
- More time maintaining docs than the code
- Developers ignore docs because signal-to-noise is low

### Examples to Avoid

```markdown
# BAD: Over-documented
## getUserById Method
This method takes a userId parameter of type Long and returns
a User object. It queries the database using JPA...

# GOOD: Document the why, not the what
## User Lookup Strategy
Users are cached for 5 minutes to reduce database load.
Cache invalidation occurs on user update events.
See ADR-015 for caching decision.
```

### Solution

- **Focus on decisions and rationale** — The "why", not the "what"
- **Document stable abstractions** — Not volatile implementation details
- **Trust the code** — Well-written code is self-documenting for "what"
- **Use ADRs** — Capture decisions, not descriptions

### What TO Document

- Architectural decisions and rationale
- Non-obvious behaviors and edge cases
- Cross-cutting concerns
- Integration points and contracts
- Operational procedures

### What NOT to Document

- Implementation details visible in code
- Every method signature
- Temporary workarounds (use code comments)
- Obvious behaviors

---

## 3. Under-Documentation

### The Problem

"The code is the documentation" taken to an extreme. Critical context, decisions, and knowledge exist only in people's heads.

### Signs

- New team members struggle for weeks/months
- Same questions asked repeatedly
- Fear of changing code due to unknown impacts
- Key knowledge leaves when people leave

### Solution

- **Document during design** — Not after the fact
- **Capture decisions** — Write ADRs when deciding
- **Onboarding test** — If a new person can't understand, document more
- **Exit interviews** — Capture knowledge before people leave

### Minimum Documentation

Every system needs at minimum:

- [ ] System context (what and why)
- [ ] Architecture overview (how it's structured)
- [ ] Key decisions (why this approach)
- [ ] Setup instructions (how to run it)
- [ ] Main flows (how it works)

---

## 4. Wrong Abstraction Level

### The Problem

Documentation that's too detailed for the audience, or too vague to be useful.

### Signs

- Business stakeholders confused by technical details
- Developers wanting more specifics
- Diagrams with mixed levels of detail
- "This doesn't help me" feedback

### Examples

```markdown
# BAD: Wrong level for executives
The system uses React 18 with Redux Toolkit for state management,
PostgreSQL 14 with pgBouncer for connection pooling...

# GOOD: Right level for executives
The system processes customer orders and integrates with
our shipping and payment partners. It handles 10,000
orders per day with 99.9% availability.

# BAD: Wrong level for developers
The system has a frontend and backend that talk to a database.

# GOOD: Right level for developers
The API server (Spring Boot 3.x) exposes REST endpoints
consumed by the React SPA. Authentication uses JWT tokens
with a 1-hour expiry. See ADR-003 for auth details.
```

### Solution

- **Know your audience** — Write for specific readers
- **Use C4 levels** — Match detail to zoom level
- **Create multiple views** — Different docs for different audiences
- **Get feedback** — Ask if docs meet reader needs

---

## 5. Diagram-Only Architecture

### The Problem

Pretty pictures without the reasoning behind them. Diagrams show structure but not why that structure was chosen.

### Signs

- Diagrams without accompanying text
- No explanation of alternatives considered
- Diagrams that don't explain trade-offs
- "Why is it like this?" questions persist

### Solution

- **Always pair diagrams with ADRs** — Explain the decisions
- **Add context to diagrams** — Brief descriptions of each element
- **Document alternatives** — What you didn't choose and why
- **Link diagrams to decisions** — Reference relevant ADRs

### Template for Diagram Documentation

```markdown
## [Diagram Name]

### Overview
[Brief description of what this diagram shows]

### Key Elements
- **Element A**: Purpose and responsibility
- **Element B**: Purpose and responsibility

### Key Decisions
- Why A communicates with B this way: See ADR-XXX
- Why we chose this boundary: See ADR-YYY

### [Diagram Here]
```

---

## 6. Copy-Paste Documentation

### The Problem

Same content duplicated in multiple places. Updates happen in some places but not others, leading to inconsistency.

### Signs

- Same information in multiple documents
- Conflicting information across docs
- Uncertainty about which version is correct
- Updates require changing multiple files

### Solution

- **Single source of truth** — One place for each piece of information
- **Link, don't duplicate** — Reference other docs instead of copying
- **Use includes** — If your tooling supports it
- **Audit regularly** — Find and eliminate duplication

### Example

```markdown
# BAD: Duplicating auth flow everywhere
## User Service
The authentication flow works by...
[500 words about auth]

## Order Service
The authentication flow works by...
[Same 500 words, slightly different]

# GOOD: Single source, referenced
## User Service
For authentication, see [Authentication Flow](../security/auth-flow.md)

## Order Service
For authentication, see [Authentication Flow](../security/auth-flow.md)
```

---

## 7. Tribal Knowledge

### The Problem

Critical information exists only in people's heads, not written down anywhere.

### Signs

- "Ask Sarah, she knows how that works"
- Long onboarding times
- Bus factor of 1 for critical systems
- Knowledge lost when people leave

### Solution

- **Document during design** — Capture knowledge when it's fresh
- **Pair programming** — Spread knowledge through collaboration
- **Rotation** — Rotate people through systems
- **Explicit handoffs** — Document before vacation/departure
- **Record decisions** — Write ADRs in real-time

### Knowledge Capture Triggers

Document when:

- [ ] Making a significant decision
- [ ] Completing a complex feature
- [ ] Fixing a non-obvious bug
- [ ] Someone asks "how does this work?"
- [ ] Before someone goes on vacation
- [ ] During onboarding (what's missing?)

---

## Anti-Pattern Detection Checklist

| Anti-Pattern | Warning Signs | Quick Fix |
|--------------|---------------|-----------|
| Graveyards | Last update > 6 months | Quarterly reviews |
| Over-documentation | More doc than code | Focus on "why" |
| Under-documentation | "Ask someone" culture | Minimum viable docs |
| Wrong level | Confused readers | Audience-specific docs |
| Diagram-only | No rationale | ADRs with diagrams |
| Copy-paste | Conflicting info | Single source of truth |
| Tribal knowledge | Bus factor = 1 | Document while doing |

---

## Recovery Steps

If you're suffering from documentation anti-patterns:

1. **Audit current state** — What exists? What's accurate?
2. **Delete the dead** — Remove clearly outdated content
3. **Identify gaps** — What's missing that causes pain?
4. **Prioritize** — Focus on highest-impact documentation
5. **Establish process** — Prevent future decay
6. **Iterate** — Documentation is never "done"
