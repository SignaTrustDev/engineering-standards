# Clean Architecture Rules — Extracted from Robert C. Martin

Enforceable architecture rules — each one can be checked against concrete code.
Advisory design philosophy from *Clean Architecture* has been removed; only
mechanically verifiable rules are kept here.

---

## The Dependency Rule

**The most important rule in the book:**

- Source code dependencies must **point only inward**, toward higher-level policies
- Nothing in an inner circle can know anything about something in an outer circle
- Layers from outside-in: Frameworks/Drivers → Interface Adapters → Use Cases → Entities

---

## SOLID Principles

| Principle | Rule |
|-----------|------|
| **SRP** | A module should be responsible to one, and only one, actor |
| **OCP** | A software artifact should be open for extension but closed for modification |
| **LSP** | Subtypes must be substitutable for their base types |
| **ISP** | Don't depend on things you don't use |
| **DIP** | Source code dependencies should refer only to abstractions, not concretions |

> **Where the examples live**
> Code examples for each principle are split into two files, keyed by principle
> letter, so reviewers only load the language they need — Python in
> [`examples/clean-architecture/py.md`](examples/clean-architecture/py.md) and
> TypeScript in
> [`examples/clean-architecture/ts.md`](examples/clean-architecture/ts.md). To
> see a principle in action, jump to its `## <letter>` section in the relevant
> file.

### S — Single Responsibility Principle

> One module, one actor (one reason to change)

### O — Open/Closed Principle

> Open for extension, closed for modification

### L — Liskov Substitution Principle

> Subtypes must be substitutable for their base types

### I — Interface Segregation Principle

> Don't depend on things you don't use

### D — Dependency Inversion Principle

> Depend on abstractions, not concretions

---

## Acyclic Dependencies (ADP)

> Allow no cycles in the component dependency graph

- The component dependency graph must be a directed **acyclic** graph
- If component A depends on B, B must not depend (directly or transitively) back on A
- Break a cycle by inverting one dependency (DIP) or extracting a new component
  that both sides depend on

---

## Clean Architecture Layers

```text
Entities          ← Enterprise Business Rules (most stable)
Use Cases         ← Application Business Rules
Interface Adapters← Controllers, Presenters, Gateways
Frameworks/Drivers← Web, DB, UI (most volatile)
```

**Rules for each layer:**

- **Entities**: Encapsulate critical business rules; no knowledge of outer layers
- **Use Cases**: Application-specific rules; unaffected by UI/DB changes
- **Interface Adapters**: Convert data formats between use cases and external agencies
- **Frameworks/Drivers**: All the details go here
