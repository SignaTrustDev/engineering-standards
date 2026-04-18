# Clean Code Standards (Robert C. Martin)

When writing or reviewing code, apply the following rules. Each rule includes a ❌ BAD and ✅ GOOD example in Python and/or TypeScript.

---

## General Rules

### 1. Follow standard conventions
Use the language's idiomatic style (PEP 8 for Python, ESLint/Prettier for TypeScript).

**Python**
```python
# ❌ BAD
class order_service:
    def PlaceOrder(self,o):
        self.repo.add(o)

# ✅ GOOD
class OrderService:
    def place_order(self, order: Order) -> None:
        self.repo.add(order)
```

**TypeScript**
```typescript
// ❌ BAD
class order_service {
  placeorder(o: any) { this.repo.add(o) }
}

// ✅ GOOD
class OrderService {
  placeOrder(order: Order): void {
    this.repo.add(order);
  }
}
```

---

### 2. Keep it simple (KISS) — reduce complexity
```python
# ❌ BAD
def calculate_area(radius: float) -> float:
    area = 0.0
    for i in range(360):
        area += (math.pi / 180) * radius * radius
    return area

# ✅ GOOD
def calculate_area(radius: float) -> float:
    return math.pi * radius ** 2
```

---

### 3. Boy Scout Rule — leave code cleaner than you found it
Always clean up before you commit: remove dead code, fix obvious naming, add a missing type hint.

```python
# ❌ BAD — left as-is when you touched this function
def proc(x, y):  # what is x? what is y?
    return x*y

# ✅ GOOD — cleaned up while you were here
def calculate_total_price(unit_price: float, quantity: int) -> float:
    return unit_price * quantity
```

---

### 4. Always find the root cause — don't suppress errors silently
```python
# ❌ BAD
def get_user(user_id: int):
    try:
        return db.query(user_id)
    except Exception:
        return None   # silent swallow — callers can't distinguish "not found" from "DB down"

# ✅ GOOD
def get_user(user_id: int) -> User:
    try:
        return db.query(user_id)
    except DatabaseError as e:
        logger.error("DB failure fetching user %s: %s", user_id, e)
        raise
```

---

## Design Rules

### 5. Keep configurable data at high levels
```python
# ❌ BAD — magic value buried deep
def connect():
    socket.settimeout(30)   # where does 30 come from?

# ✅ GOOD
DEFAULT_TIMEOUT_SECONDS = 30   # high-level config

def connect(timeout: int = DEFAULT_TIMEOUT_SECONDS) -> None:
    socket.settimeout(timeout)
```

**TypeScript**
```typescript
// ❌ BAD
function connect() { socket.setTimeout(30000); }

// ✅ GOOD
const DEFAULT_TIMEOUT_MS = 30_000;
function connect(timeoutMs = DEFAULT_TIMEOUT_MS): void {
  socket.setTimeout(timeoutMs);
}
```

---

### 6. Prefer polymorphism over if/else or switch/case
```python
# ❌ BAD
def send_notification(type: str, message: str) -> None:
    if type == "email":
        send_email(message)
    elif type == "sms":
        send_sms(message)
    elif type == "push":
        send_push(message)

# ✅ GOOD
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None: ...

class EmailNotification(Notification):
    def send(self, message: str) -> None:
        send_email(message)

class SmsNotification(Notification):
    def send(self, message: str) -> None:
        send_sms(message)
```

---

### 7. Separate multi-threading / async code from business logic
```python
# ❌ BAD — business logic and concurrency tangled together
def process_order(order: Order) -> None:
    lock.acquire()
    try:
        total = sum(item.price for item in order.items)  # business logic
        db.save(order, total)
    finally:
        lock.release()

# ✅ GOOD — pure business logic
def calculate_order_total(order: Order) -> Decimal:
    return sum(item.price for item in order.items)

# Concurrency wrapper is separate
async def process_order_async(order: Order) -> None:
    async with db_lock:
        total = calculate_order_total(order)
        await db.save(order, total)
```

---

### 8. Use dependency injection — don't instantiate collaborators inside classes
```python
# ❌ BAD
class OrderService:
    def __init__(self):
        self.repo = PostgresOrderRepository()   # hard-coded, untestable

# ✅ GOOD
class OrderService:
    def __init__(self, repo: OrderRepository) -> None:
        self.repo = repo   # injected, mockable
```

---

### 9. Follow the Law of Demeter — talk only to direct neighbors
```python
# ❌ BAD — chaining through internals
def print_customer_city(order: Order) -> None:
    print(order.customer.address.city)   # violates LoD

# ✅ GOOD — Order exposes what callers need
class Order:
    def customer_city(self) -> str:
        return self.customer.address.city

def print_customer_city(order: Order) -> None:
    print(order.customer_city())
```

---

## Understandability Tips

### 10. Be consistent — same pattern everywhere
```python
# ❌ BAD — three styles for the same concept
def fetch_user_by_id(id): ...
def getUserEmail(user_id): ...
def get_profile(uid): ...

# ✅ GOOD
def get_user_by_id(user_id: int) -> User: ...
def get_user_by_email(email: str) -> User: ...
def get_user_profile(user_id: int) -> UserProfile: ...
```

---

### 11. Use explanatory variables
```python
# ❌ BAD
if (order.status == 2 and datetime.now() - order.created_at > timedelta(days=30)):
    cancel(order)

# ✅ GOOD
is_pending = order.status == OrderStatus.PENDING
is_stale   = datetime.now() - order.created_at > timedelta(days=30)

if is_pending and is_stale:
    cancel(order)
```

---

### 12. Encapsulate boundary conditions
```python
# ❌ BAD — off-by-one logic scattered everywhere
if index >= 0 and index <= len(items) - 1:
    ...

# ✅ GOOD
def is_valid_index(index: int, items: list) -> bool:
    return 0 <= index < len(items)

if is_valid_index(index, items):
    ...
```

---

### 13. Prefer value objects over primitives
```python
# ❌ BAD
def create_account(email: str, age: int) -> None: ...
# Nothing stops caller from passing age as email, or a raw invalid string

# ✅ GOOD
@dataclass(frozen=True)
class Email:
    value: str
    def __post_init__(self):
        if "@" not in self.value:
            raise ValueError(f"Invalid email: {self.value}")

@dataclass(frozen=True)
class Age:
    value: int
    def __post_init__(self):
        if self.value < 0:
            raise ValueError("Age cannot be negative")

def create_account(email: Email, age: Age) -> None: ...
```

---

### 14. Avoid negative conditionals
```python
# ❌ BAD
if not user.is_inactive:
    grant_access(user)

# ✅ GOOD
if user.is_active:
    grant_access(user)
```

---

## Naming Rules

### 15. Use descriptive, unambiguous names
```python
# ❌ BAD
def proc(d, f):
    return d * f

# ✅ GOOD
def calculate_total_cost(unit_price: Decimal, quantity: int) -> Decimal:
    return unit_price * quantity
```

---

### 16. Use pronounceable names
```python
# ❌ BAD
genymdhms = datetime.now()   # "gen why em dee aitch em ess"?
modymdhms = datetime.now()

# ✅ GOOD
generation_timestamp = datetime.now()
modification_timestamp = datetime.now()
```

---

### 17. Use searchable names — avoid magic numbers
```python
# ❌ BAD
if days > 7:
    send_reminder()

# ✅ GOOD
REMINDER_THRESHOLD_DAYS = 7

if days > REMINDER_THRESHOLD_DAYS:
    send_reminder()
```

---

### 18. Avoid encodings and type noise
```python
# ❌ BAD
str_user_name = "Jonathan"
i_user_age    = 51
lst_orders    = []

# ✅ GOOD
user_name = "Jonathan"
user_age  = 51
orders    = []
```

---

## Function Rules

### 19. Functions should be small and do ONE thing
```python
# ❌ BAD — does three things
def register_user(data: dict) -> None:
    # validate
    if not data.get("email"):
        raise ValueError("Email required")
    # create
    user = User(**data)
    db.add(user)
    # notify
    send_welcome_email(user.email)

# ✅ GOOD — each function has one job
def validate_registration_data(data: dict) -> None:
    if not data.get("email"):
        raise ValueError("Email required")

def create_user(data: dict) -> User:
    user = User(**data)
    db.add(user)
    return user

def register_user(data: dict) -> None:
    validate_registration_data(data)
    user = create_user(data)
    send_welcome_email(user.email)
```

---

### 20. Prefer fewer arguments — use parameter objects when needed
```python
# ❌ BAD
def create_report(title, author, start_date, end_date, include_charts, format):
    ...

# ✅ GOOD
@dataclass
class ReportConfig:
    title: str
    author: str
    start_date: date
    end_date: date
    include_charts: bool = True
    format: str = "pdf"

def create_report(config: ReportConfig) -> Report:
    ...
```

---

### 21. No side effects — a function should do what its name says and nothing else
```python
# ❌ BAD — name implies read-only, but mutates state
def get_user(user_id: int) -> User:
    user = db.query(user_id)
    user.last_accessed = datetime.now()   # hidden side effect!
    db.save(user)
    return user

# ✅ GOOD — separate concerns
def get_user(user_id: int) -> User:
    return db.query(user_id)

def record_access(user: User) -> None:
    user.last_accessed = datetime.now()
    db.save(user)
```

---

### 22. Don't use flag arguments — split into separate functions
```python
# ❌ BAD
def render_page(include_header: bool) -> str:
    if include_header:
        return render_with_header()
    return render_without_header()

# ✅ GOOD
def render_page_with_header() -> str: ...
def render_page() -> str: ...
```

---

## Comment Rules

### 23. Explain yourself in code, not in comments
```python
# ❌ BAD
# Check if the user is old enough to purchase alcohol
if user.age >= 21:
    allow_purchase()

# ✅ GOOD
MINIMUM_ALCOHOL_PURCHASE_AGE = 21

def is_eligible_to_purchase_alcohol(user: User) -> bool:
    return user.age >= MINIMUM_ALCOHOL_PURCHASE_AGE

if is_eligible_to_purchase_alcohol(user):
    allow_purchase()
```

---

### 24. Use comments to explain INTENT, WARNINGS, and non-obvious decisions — not the obvious
```python
# ❌ BAD — states the obvious
# Add item to list
items.append(item)

# ✅ GOOD — explains WHY, not WHAT
# We sort on every insert (O(n log n)) rather than at read time
# because reads vastly outnumber writes in this workflow.
items.append(item)
items.sort(key=lambda x: x.priority)
```

```python
# ✅ GOOD — warning comment
# WARNING: This permanently deletes the user and all associated data.
# There is no soft-delete. Caller must confirm via UI before invoking.
def hard_delete_user(user_id: int) -> None:
    db.execute("DELETE FROM users WHERE id = ?", user_id)
```

---

### 25. Never comment out dead code — delete it
```python
# ❌ BAD
def process_payment(amount: Decimal) -> Receipt:
    # old_charge(amount)
    # legacy_receipt = old_receipt_service.create()
    return new_payment_service.charge(amount)

# ✅ GOOD — git history preserves the old code
def process_payment(amount: Decimal) -> Receipt:
    return new_payment_service.charge(amount)
```

---

## Source Code Structure

### 26. Declare variables close to their usage
```python
# ❌ BAD
def process_order(order: Order) -> None:
    subtotal = Decimal(0)    # declared far from use
    tax_rate = Decimal("0.08")
    # ... 30 lines of unrelated logic ...
    subtotal = sum(item.price for item in order.items)
    total = subtotal * (1 + tax_rate)

# ✅ GOOD
def process_order(order: Order) -> None:
    # ... other unrelated logic ...
    TAX_RATE = Decimal("0.08")
    subtotal = sum(item.price for item in order.items)
    total    = subtotal * (1 + TAX_RATE)
```

---

### 27. Place caller above callee (newspaper structure — high-level first)
```python
# ✅ GOOD — public API at top, helpers below
class ReportService:
    def generate_report(self, config: ReportConfig) -> Report:
        data    = self._fetch_data(config)
        summary = self._summarize(data)
        return self._format(summary, config)

    def _fetch_data(self, config: ReportConfig) -> list: ...
    def _summarize(self, data: list) -> dict: ...
    def _format(self, summary: dict, config: ReportConfig) -> Report: ...
```

---

## Objects and Data Structures

### 28. Hide internal structure — expose behavior, not data
```python
# ❌ BAD — callers manipulate internals
class BankAccount:
    balance: Decimal = Decimal(0)

account.balance += Decimal("100")   # bypasses business rules

# ✅ GOOD
class BankAccount:
    def __init__(self) -> None:
        self._balance = Decimal(0)

    def deposit(self, amount: Decimal) -> None:
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount

    def balance(self) -> Decimal:
        return self._balance
```

---

### 29. Single Responsibility — classes should have one reason to change
```python
# ❌ BAD — does everything
class User:
    def save(self): ...         # persistence concern
    def send_email(self): ...   # notification concern
    def validate(self): ...     # validation concern

# ✅ GOOD — split by responsibility
class User: ...                          # domain model only
class UserRepository: ...                # persistence
class UserNotificationService: ...       # notifications
class UserValidator: ...                 # validation
```

---

### 30. Base class should not know about derived classes
```python
# ❌ BAD
class Shape:
    def area(self) -> float:
        if isinstance(self, Circle):    # base knows about subclass!
            return math.pi * self.radius ** 2
        if isinstance(self, Square):
            return self.side ** 2

# ✅ GOOD — polymorphism
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Circle(Shape):
    def area(self) -> float:
        return math.pi * self.radius ** 2

class Square(Shape):
    def area(self) -> float:
        return self.side ** 2
```

---

## Tests (F.I.R.S.T.)

### 31. Fast — tests should run in milliseconds; no real I/O
```python
# ❌ BAD — hits actual database
def test_get_user():
    user = UserService().get_user(1)   # slow, brittle
    assert user.name == "Jonathan"

# ✅ GOOD — mocked dependency
def test_get_user(mocker):
    mock_repo = mocker.Mock()
    mock_repo.find_by_id.return_value = User(id=1, name="Jonathan")
    service = UserService(repo=mock_repo)
    assert service.get_user(1).name == "Jonathan"
```

---

### 32. Independent — tests must not depend on each other
```python
# ❌ BAD — test 2 depends on test 1 having run first
shared_cart = ShoppingCart()

def test_add_item():
    shared_cart.add(Item("book", 10))

def test_total():
    assert shared_cart.total() == 10   # fails if run in isolation

# ✅ GOOD — each test owns its setup
def test_total():
    cart = ShoppingCart()
    cart.add(Item("book", 10))
    assert cart.total() == 10
```

---

### 33. Readable — test names describe behavior, not implementation
```python
# ❌ BAD
def test_func1():
    assert calc(2, 3) == 5

# ✅ GOOD
def test_add_returns_sum_of_two_positive_integers():
    assert calculator.add(2, 3) == 5

def test_add_with_negative_number_returns_correct_sum():
    assert calculator.add(-1, 5) == 4
```

---

### 34. One logical assertion per test
```python
# ❌ BAD — multiple concerns, hard to pinpoint failure
def test_user_creation():
    user = create_user("jonathan@example.com", "Jonathan")
    assert user.email == "jonathan@example.com"
    assert user.name == "Jonathan"
    assert user.is_active is True
    assert user.created_at is not None

# ✅ GOOD — focused tests
def test_user_has_correct_email():
    user = create_user("jonathan@example.com", "Jonathan")
    assert user.email == "jonathan@example.com"

def test_new_user_is_active_by_default():
    user = create_user("jonathan@example.com", "Jonathan")
    assert user.is_active is True
```

---

## Code Smells to Avoid

### 35. Rigidity — don't make changes that cascade everywhere
```python
# ❌ BAD — adding a new payment type requires touching this function AND every caller
def process_payment(payment_type: str, amount: Decimal) -> None:
    if payment_type == "credit":
        credit_processor.charge(amount)
    elif payment_type == "crypto":
        crypto_processor.charge(amount)
    # Adding "ACH" means editing this function, re-testing everything

# ✅ GOOD — open for extension, closed for modification (OCP)
class PaymentProcessor(Protocol):
    def charge(self, amount: Decimal) -> None: ...

def process_payment(processor: PaymentProcessor, amount: Decimal) -> None:
    processor.charge(amount)
# Adding ACH = new class only, no edits here
```

---

### 36. Needless repetition (DRY)
```python
# ❌ BAD
def validate_order(order: Order) -> None:
    if not order.customer_id:
        raise ValueError("customer_id required")
    if not order.items:
        raise ValueError("items required")

def validate_quote(quote: Quote) -> None:
    if not quote.customer_id:
        raise ValueError("customer_id required")   # duplicated
    if not quote.items:
        raise ValueError("items required")         # duplicated

# ✅ GOOD
def validate_has_customer_and_items(obj: Any) -> None:
    if not obj.customer_id:
        raise ValueError("customer_id required")
    if not obj.items:
        raise ValueError("items required")
```

---

### 37. Opacity — deeply nested code is hard to read; flatten it
```python
# ❌ BAD
def process_invoice(invoice):
    if invoice:
        if invoice.amount > 0:
            if invoice.status == "pending":
                if invoice.customer:
                    charge(invoice)

# ✅ GOOD — guard clauses (early return)
def process_invoice(invoice: Invoice) -> None:
    if not invoice:
        return
    if invoice.amount <= 0:
        return
    if invoice.status != "pending":
        return
    if not invoice.customer:
        return
    charge(invoice)
```

---

### 38. Needless complexity — don't over-engineer
```typescript
// ❌ BAD — strategy pattern for two cases that will never grow
interface GreetingStrategy { execute(name: string): string; }
class FormalGreeting implements GreetingStrategy {
  execute(name: string) { return `Good day, ${name}.`; }
}
class GreetingContext {
  constructor(private strategy: GreetingStrategy) {}
  greet(name: string) { return this.strategy.execute(name); }
}

// ✅ GOOD
function greet(name: string): string {
  return `Good day, ${name}.`;
}
```

---

## TypeScript-Specific Clean Code

### 39. Prefer discriminated unions over boolean flags
```typescript
// ❌ BAD
type ApiResponse = {
  data?: User;
  error?: string;
  isLoading: boolean;
  isError: boolean;
};

// ✅ GOOD
type ApiResponse =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error";   error: string };
```

---

### 40. Use `unknown` over `any` — force explicit narrowing
```typescript
// ❌ BAD
function parseConfig(raw: any) {
  return raw.timeout * 1000;  // runtime explosion waiting to happen
}

// ✅ GOOD
function parseConfig(raw: unknown): Config {
  if (typeof raw !== "object" || raw === null) {
    throw new Error("Config must be an object");
  }
  const r = raw as Record<string, unknown>;
  if (typeof r.timeout !== "number") {
    throw new Error("Config.timeout must be a number");
  }
  return { timeout: r.timeout * 1000 };
}
```

---

## Summary Checklist

Before committing, verify:

- [ ] Names are descriptive — no abbreviations, no magic numbers
- [ ] Functions do ONE thing and are under ~20 lines
- [ ] No flag arguments (split into separate functions)
- [ ] No side effects that aren't obvious from the function name
- [ ] No commented-out code
- [ ] No deeply nested conditionals (use guard clauses)
- [ ] No duplication (DRY)
- [ ] Configurable values are constants at the top, not buried inline
- [ ] Tests are fast, independent, and named after behavior
- [ ] Classes have a single reason to change (SRP)
