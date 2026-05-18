# Clean Architecture Rules — Extracted from Robert C. Martin

Enforceable architecture rules — each one can be checked against concrete code.
Advisory design philosophy from *Clean Architecture* has been removed; only
mechanically verifiable rules are kept here.

---

## The Dependency Rule

> Source code dependencies must point only inward, toward higher-level policy

The four layers run from Entities (innermost, most stable) outward to
Frameworks/Drivers (outermost, most volatile). An inner layer must know nothing
about any outer layer.

```text
Entities          ← Enterprise Business Rules (most stable)
Use Cases         ← Application Business Rules
Interface Adapters← Controllers, Presenters, Gateways
Frameworks/Drivers← Web, DB, UI (most volatile)
```

**How to classify a module** (needed for the detection signal below):

- **Entities** — domain types and business rules; import no other layer
- **Use Cases** — application workflows; import Entities only
- **Interface Adapters** — controllers, presenters, gateways, ORM models
- **Frameworks/Drivers** — web framework, DB driver, UI code

**Detection signal:** classify each module into a layer, then inspect its
imports. A violation is any `import` edge pointing from an inner layer to an
outer layer.

- `entities/` importing anything in `use_cases/`, `adapters/`, or `frameworks/`
- `use_cases/` importing `adapters/` or `frameworks/`
- A framework type (ORM model, HTTP request, UI widget) referenced inside an
  entity or use case

**Fix:** invert the dependency (DIP) — declare an interface in the inner layer
and have the outer layer implement it — or move the misplaced code to the layer
it belongs in.

**Enforcement:**

- If a change you are making would *introduce* an inner → outer import,
  restructure before finishing — never commit one.
- For a *pre-existing* violation you encounter: refactor it if it touches files
  or modules already in scope for the current task; otherwise add a
  `TODO [ARCH:dependency-rule]` comment and surface it to the user.

**Relationship to ADP:** the Dependency Rule constrains edge *direction* against
the layer hierarchy; ADP constrains the *shape* of the whole graph (no cycles).
Obeying the Dependency Rule rules out inter-layer cycles, but not intra-layer
cycles — the two checks are complementary, not redundant.

---

## SOLID Principles

| Principle | Rule |
|-----------|------|
| **SRP** | A module should be responsible to one, and only one, actor |
| **OCP** | A software artifact should be open for extension but closed for modification |
| **LSP** | Subtypes must be substitutable for their base types |
| **ISP** | Don't depend on things you don't use |
| **DIP** | Source code dependencies should refer only to abstractions, not concretions |

> **TypeScript examples**
> Each principle below shows a Python example inline. The equivalent TypeScript
> examples live in
> [`examples/clean-architecture/ts.md`](examples/clean-architecture/ts.md),
> keyed by principle letter.

**Enforcement (applies to all five principles):** each principle pairs a
❌ BAD pattern with a ✅ GOOD pattern. When you see the BAD pattern:

- Replace it with the GOOD pattern if it touches files or modules already in
  scope for the current task.
- Otherwise add a `TODO [SOLID:<letter>]` comment (e.g. `TODO [SOLID:D]`) naming
  the principle, and surface it to the user.

Never write new code that matches a BAD pattern.

### S — Single Responsibility Principle

> One module, one actor (one reason to change)

**Detection signal:** a class with methods spanning multiple concerns —
persistence (`save`/`load`), notification (`send_email`), reporting, validation
— so more than one actor could force it to change.

**Fix:** split into one class per concern (domain model, repository, notifier,
validator).

```python
# ❌ BAD — one class, three reasons to change
class User:
    def save_to_db(self): ...         # persistence concern
    def send_welcome_email(self): ... # notification concern
    def generate_report(self): ...    # reporting concern

# ✅ GOOD — one actor per class
class User: ...                       # just the domain model

class UserRepository:
    def save(self, user: User): ...

class UserNotifier:
    def send_welcome(self, user: User): ...
```

### O — Open/Closed Principle

> Open for extension, closed for modification

**Detection signal:** an `if`/`elif` chain or `switch` dispatching on a type
string or enum to choose behavior, where adding a new case means editing the
function.

**Fix:** introduce a polymorphic abstraction (abstract base / Strategy); each
case becomes its own class, so new cases add code instead of editing it.

```python
# ❌ BAD — must edit this function to add a new customer type
def get_discount(customer_type: str) -> float:
    if customer_type == 'vip': return 0.2
    if customer_type == 'member': return 0.1
    return 0.0

# ✅ GOOD — new customer types = new classes, no edits to existing code
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self) -> float: ...

class VipDiscount(DiscountStrategy):
    def calculate(self) -> float: return 0.2

class MemberDiscount(DiscountStrategy):
    def calculate(self) -> float: return 0.1
```

### L — Liskov Substitution Principle

> Subtypes must be substitutable for their base types

**Detection signal:** a subclass that overrides a base method to raise
`NotImplementedError`, reject inputs the base accepts, or change behavior that
callers of the base type rely on.

**Fix:** drop the inheritance; make the types siblings under a shared
abstraction. Don't subclass purely to reuse code.

```python
# ❌ BAD — Square.set_width gives callers of Rectangle surprising behavior
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):           # breaks LSP
        self.width = self.height = w

# ✅ GOOD — siblings under a shared abstraction, no broken inheritance
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Rectangle(Shape):
    def __init__(self, w, h): self.w, self.h = w, h
    def area(self): return self.w * self.h

class Square(Shape):
    def __init__(self, s): self.s = s
    def area(self): return self.s ** 2
```

### I — Interface Segregation Principle

> Don't depend on things you don't use

**Detection signal:** an interface or ABC whose implementers leave methods
empty, `pass` them, or raise `NotImplementedError` — they are forced to depend
on methods they don't use.

**Fix:** split the fat interface into role-specific interfaces; each implementer
composes only the ones that apply.

```python
# ❌ BAD — Robot forced to implement methods it cannot support
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...   # robots can't eat
    @abstractmethod
    def sleep(self): ... # robots can't sleep

class Robot(Worker):
    def work(self): ...
    def eat(self): raise NotImplementedError('robots do not eat')
    def sleep(self): raise NotImplementedError('robots do not sleep')

# ✅ GOOD — narrow interfaces; implement only what applies
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Eatable(ABC):
    @abstractmethod
    def eat(self): ...

class Human(Workable, Eatable):
    def work(self): ...
    def eat(self): ...

class Robot(Workable):
    def work(self): ...
```

### D — Dependency Inversion Principle

> Depend on abstractions, not concretions

**Detection signal:** a class that constructs a concrete collaborator
(`MySQLDatabase()`, `new ConcreteRepo()`) inside its constructor or methods
instead of receiving an abstraction.

**Fix:** depend on an interface / `Protocol` and inject the concrete
implementation through the constructor.

```python
# ❌ BAD — hard dependency on a concretion
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()

    def save_order(self, order):
        self.db.save(order)

# ✅ GOOD — depend on an abstraction; caller injects the implementation
class Database(Protocol):              # abstraction (interface)
    def save(self, entity) -> None: ...

class OrderService:
    def __init__(self, db: Database):  # depends on abstraction
        self.db = db

    def save_order(self, order):
        self.db.save(order)

# Caller injects whichever impl they want:
service = OrderService(db=MySQLDatabase())
service = OrderService(db=InMemoryDatabase())  # for tests
```

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

**Relationship to the Dependency Rule:** ADP constrains the *shape* of the graph
(no cycles anywhere); the Dependency Rule constrains edge *direction* against the
layer hierarchy. A lone inner → outer import breaks the Dependency Rule without
forming a cycle, so run both checks.

**Tooling:** `import-linter` / `pydeps` (Python); `madge --circular` or ESLint
`import/no-cycle` (TypeScript).
