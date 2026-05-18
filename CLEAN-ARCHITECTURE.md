# Clean Architecture Rules — Extracted from Robert C. Martin

## Core Goal

> "The goal of software architecture is to minimize the human resources required to build and maintain the required system."

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

## Component Cohesion Principles

- **REP** (Reuse/Release Equivalence): The granule of reuse is the granule of release
- **CCP** (Common Closure): Gather into components those classes that change for the same reasons and at the same times
- **CRP** (Common Reuse): Don't force users of a component to depend on things they don't need

---

## Component Coupling Principles

- **ADP** (Acyclic Dependencies): Allow no cycles in the component dependency graph
- **SDP** (Stable Dependencies): Depend in the direction of stability
- **SAP** (Stable Abstractions): A component should be as abstract as it is stable

---

## Architecture Rules

### Boundaries

- Draw lines between things that change at **different rates and for different reasons**
- Architectural boundaries should point dependencies toward **higher-level policy**
- Database, UI, web, frameworks = **details** — keep them behind boundaries
- The GUI is a detail. The web is a detail. The database is a detail.

### Keeping Options Open

> "A good architect maximizes the number of decisions not made."

- Defer decisions about databases, web servers, frameworks as long as possible
- Good architecture allows you to defer framework choice until much later

### The Main Sequence

Components should plot near the line connecting (I=1, A=0) and (I=0, A=1):

- **Zone of Pain** (0,0): Stable + Concrete = rigid, hard to change
- **Zone of Uselessness** (1,1): Unstable + Abstract = useless
- Target: balance abstractness with stability

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

---

## Screaming Architecture

- Your architecture should **scream the business domain**, not the framework
- "When looking at the top-level structure, it should scream 'Health Care System' not 'Rails'"
- Frameworks are tools, not architectures

---

## Testing Rules

- Tests follow the **Dependency Rule** — they are the outermost circle
- Nothing in the system depends on tests
- Fragile Tests Problem = tests coupled to volatile UI or structure
- Create a **Testing API** that lets you bypass UI to test business rules directly
- Design for testability: "Don't depend on volatile things"

---

## Humble Object Pattern

Split behaviors into:

- **Humble object**: Hard-to-test behaviors (Views, DB implementations)
- **Testable object**: Easy-to-test behaviors (Presenters, Interactors)

Applied at: Presenter/View, Database Gateways, Service Listeners

---

## Service Architecture Rules

- Services do **not** automatically define architecture
- Services that simply separate behaviors are "expensive function calls"
- Services can be coupled by **shared data** — the decoupling is often illusory
- Architectural boundaries run **through** services, not between them

---

## Key Heuristics

1. **Only the way to go fast is to go well** — messy code is always slower, even short-term
2. **Making messes is always slower than staying clean**, at every time scale
3. **A good architecture leaves options open** — defer irreversible decisions
4. **If component A should be protected from B, then B should depend on A**
5. **Don't marry a framework** — treat it as a plugin to your core
6. **The database is not the data model** — separate them

---

## Practical Architecture Decisions (from "The Missing Chapter")

- Prefer **package by component** over package by layer — bundle business logic + persistence behind a clean interface
- Use **access modifiers** (package-private, internal) to enforce architectural boundaries at compile time
- Making all types `public` collapses all four code organization styles into the same flat architecture
- Use the **compiler to enforce** your architecture, not just discipline and code reviews
