# Clean Architecture — TypeScript Examples

TypeScript ❌ BAD / ✅ GOOD examples for the SOLID principles in
[`CLEAN-ARCHITECTURE.md`](../../CLEAN-ARCHITECTURE.md). Each section is keyed by
principle letter. Read the principle summary in `CLEAN-ARCHITECTURE.md`; the
code below shows the violation and the fix.

---

## S — Single Responsibility Principle

```typescript
// ❌ BAD — one class, three reasons to change
class User {
  saveToDb() { ... }         // persistence concern
  sendWelcomeEmail() { ... } // notification concern
  generateReport() { ... }   // reporting concern
}

// ✅ GOOD — one actor per class
class User { ... }           // just the domain model

class UserRepository {
  save(user: User): void { ... }
}

class UserNotifier {
  sendWelcome(user: User): void { ... }
}
```

---

## O — Open/Closed Principle

```typescript
// ❌ BAD — must edit this function to add a new customer type
function getDiscount(customerType: string): number {
  if (customerType === 'vip') return 0.2;
  if (customerType === 'member') return 0.1;
  return 0;
}

// ✅ GOOD — new customer types = new classes, no edits to existing code
interface DiscountStrategy {
  calculate(): number;
}
class VipDiscount implements DiscountStrategy {
  calculate() { return 0.2; }
}
class MemberDiscount implements DiscountStrategy {
  calculate() { return 0.1; }
}
```

---

## L — Liskov Substitution Principle

```typescript
// ❌ BAD — Square.setWidth gives callers of Rectangle surprising behavior
class Rectangle {
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}
class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // breaks LSP
}

// ✅ GOOD — siblings under a shared abstraction, no broken inheritance
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

---

## I — Interface Segregation Principle

```typescript
// ❌ BAD — Robot forced to implement methods it cannot support
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

// ✅ GOOD — narrow interfaces; implement only what applies
interface Workable  { work(): void; }
interface Eatable   { eat(): void; }
interface Sleepable { sleep(): void; }

class Human implements Workable, Eatable, Sleepable { ... }
class Robot implements Workable { work() { ... } }
```

---

## D — Dependency Inversion Principle

```typescript
// ❌ BAD — hard dependency on a concretion
class OrderService {
  private db = new MySQLDatabase();

  saveOrder(order: Order) {
    this.db.save(order);
  }
}

// ✅ GOOD — depend on an abstraction; caller injects the implementation
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
