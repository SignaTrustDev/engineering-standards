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

### S — Single Responsibility Principle

> One module, one actor (one reason to change)

**Bad (Python):**

```python
class User:
    def save_to_db(self): ...        # persistence concern
    def send_welcome_email(self): ... # notification concern
    def generate_report(self): ...   # reporting concern
```

**Good (Python):**

```python
class User: ...                   # just the domain model
class UserRepository:
    def save(self, user: User): ...
class UserNotifier:
    def send_welcome(self, user: User): ...
```

**Bad (TypeScript):**

```typescript
class User {
  saveToDb() { ... }        // persistence concern
  sendWelcomeEmail() { ... } // notification concern
  generateReport() { ... }  // reporting concern
}
```

**Good (TypeScript):**

```typescript
class User { ... }           // just the domain model
class UserRepository {
  save(user: User): void { ... }
}
class UserNotifier {
  sendWelcome(user: User): void { ... }
}
```

### O — Open/Closed Principle

> Open for extension, closed for modification

**Bad (Python):**

```python
def get_discount(customer_type: str) -> float:
    if customer_type == 'vip': return 0.2
    if customer_type == 'member': return 0.1
    return 0.0  # must edit this function to add new types
```

**Good (Python):**

```python
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self) -> float: ...

class VipDiscount(DiscountStrategy):
    def calculate(self) -> float: return 0.2

class MemberDiscount(DiscountStrategy):
    def calculate(self) -> float: return 0.1

# New customer types = new classes, no edits to existing code
```

**Bad (TypeScript):**

```typescript
function getDiscount(customerType: string): number {
  if (customerType === 'vip') return 0.2;
  if (customerType === 'member') return 0.1;
  return 0; // must edit this function to add new types
}
```

**Good (TypeScript):**

```typescript
interface DiscountStrategy {
  calculate(): number;
}
class VipDiscount implements DiscountStrategy {
  calculate() { return 0.2; }
}
class MemberDiscount implements DiscountStrategy {
  calculate() { return 0.1; }
}
// New customer types = new classes, no edits to existing code
```

### L — Liskov Substitution Principle

> Subtypes must be substitutable for their base types

**Bad (Python):**

```python
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):           # breaks LSP — callers of Rectangle
        self.width = self.height = w  # get surprising behavior
```

**Good (Python):**

```python
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

**Bad (TypeScript):**

```typescript
class Rectangle {
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}
class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // breaks LSP
}
```

**Good (TypeScript):**

```typescript
abstract class Shape {
  abstract area(): number;
}
class Rectangle extends Shape {
  constructor(private w: number, private h: number) { super(); }
  area() { return this.w * this.h; }
}
class Square extends Shape {
  constructor(private s: number) { super(); }
  area() { return this.s ** 2; }
}
```

### I — Interface Segregation Principle

> Don't depend on things you don't use

**Bad (Python):**

```python
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
```

**Good (Python):**

```python
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

**Bad (TypeScript):**

```typescript
interface Worker {
  work(): void;
  eat(): void;   // robots can't eat — forced to implement a no-op
  sleep(): void; // robots can't sleep
}
class Robot implements Worker {
  work() { ... }
  eat() { throw new Error('robots do not eat'); }
  sleep() { throw new Error('robots do not sleep'); }
}
```

**Good (TypeScript):**

```typescript
interface Workable  { work(): void; }
interface Eatable   { eat(): void; }
interface Sleepable { sleep(): void; }

class Human implements Workable, Eatable, Sleepable { ... }
class Robot implements Workable { work() { ... } }
```

### D — Dependency Inversion Principle

> Depend on abstractions, not concretions

**Bad (Python):**

```python
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # hard dependency on a concretion

    def save_order(self, order):
        self.db.save(order)
```

**Good (Python):**

```python
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

**Bad (TypeScript):**

```typescript
class OrderService {
  private db = new MySQLDatabase(); // hard dependency on a concretion

  saveOrder(order: Order) {
    this.db.save(order);
  }
}
```

**Good (TypeScript):**

```typescript
interface Database {
  save(entity: unknown): void;
}

class OrderService {
  constructor(private db: Database) {} // depends on abstraction

  saveOrder(order: Order) {
    this.db.save(order);
  }
}

// Caller injects whichever impl they want:
const service = new OrderService(new MySQLDatabase());
const testService = new OrderService(new InMemoryDatabase()); // for tests
```

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
