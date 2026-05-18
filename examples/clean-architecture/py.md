# Clean Architecture — Python Examples

Python ❌ BAD / ✅ GOOD examples for the SOLID principles in
[`CLEAN-ARCHITECTURE.md`](../../CLEAN-ARCHITECTURE.md). Each section is keyed by
principle letter. Read the principle summary in `CLEAN-ARCHITECTURE.md`; the
code below shows the violation and the fix.

---

## S — Single Responsibility Principle

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

---

## O — Open/Closed Principle

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

---

## L — Liskov Substitution Principle

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

---

## I — Interface Segregation Principle

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

---

## D — Dependency Inversion Principle

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
