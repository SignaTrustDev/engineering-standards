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

A component is a module or package — a unit imported as a whole. The import
graph (node = module/package, edge = `import`) must be a directed **acyclic**
graph.

**Detection signal:** a cycle exists when, following `import` edges from
component A, you can return to A.

- Direct cycle — `A` imports `B` and `B` imports `A`
- Indirect cycle — `A` → `B` → `C` → `A`

**Fix:** break the cycle by inverting one edge (DIP — put an interface in the
component that should not depend outward) or extract the shared code into a new
component both sides depend on.

**Enforcement:**

- If a change you are making would *introduce* a cycle, restructure before
  finishing — never commit a new cycle.
- For a *pre-existing* cycle you encounter: refactor it if it touches files or
  modules already in scope for the current task; otherwise add a
  `TODO [ARCH:adp]` comment marking the cycle and surface it to the user.

**Tooling:** `import-linter` / `pydeps` (Python); `madge --circular` or ESLint
`import/no-cycle` (TypeScript).

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
