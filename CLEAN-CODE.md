# Clean Code Standards (Robert C. Martin)

Rules for writing clean code, organized by chapter. Each rule has ❌ BAD and ✅ GOOD examples in **both Python and TypeScript** unless it is language-specific. Page references are to *Clean Code* (Martin, 2008).

> **How to use this file when reviewing code**
> Scan top-to-bottom. Each rule has a one-line summary. If code violates the rule, the ❌ BAD example will look familiar. Fix it to match the ✅ GOOD example.

---

## Chapter 1 — Clean Code

### 1. Boy Scout Rule — leave code cleaner than you found it *(p. 14)*

Remove dead code, fix obvious naming, add missing type hints when you touch a file.

#### Python

```python
# ❌ BAD — left as-is when you touched this function
def proc(x, y):
    return x*y

# ✅ GOOD — cleaned up while you were here
def calculate_total_price(unit_price: float, quantity: int) -> float:
    return unit_price * quantity
```

#### TypeScript

```typescript
// ❌ BAD — left as-is
function proc(x: number, y: number) { return x * y; }

// ✅ GOOD — cleaned up while you were here
function calculateTotalPrice(unitPrice: number, quantity: number): number {
  return unitPrice * quantity;
}
```

---

## Chapter 2 — Meaningful Names

### 2. Use descriptive, unambiguous names *(p. 18)*

Names must communicate intent without requiring the reader to look elsewhere.

#### Python

```python
# ❌ BAD — single-letter names say nothing
def proc(d, f):
    return d * f

# ✅ GOOD
def calculate_total_cost(unit_price: Decimal, quantity: int) -> Decimal:
    return unit_price * quantity
```

#### TypeScript

```typescript
// ❌ BAD
function proc(d: number, f: number): number { return d * f; }

// ✅ GOOD
function calculateTotalCost(unitPrice: number, quantity: number): number {
  return unitPrice * quantity;
}
```

---

### 3. Use pronounceable names *(p. 21)*

If you can't say a name aloud, you can't discuss it with colleagues.

#### Python

```python
# ❌ BAD
genymdhms = datetime.now()
modymdhms = datetime.now()

# ✅ GOOD
generation_timestamp  = datetime.now()
modification_timestamp = datetime.now()
```

#### TypeScript

```typescript
// ❌ BAD
const genymdhms = new Date();
const modymdhms = new Date();

// ✅ GOOD
const generationTimestamp  = new Date();
const modificationTimestamp = new Date();
```

---

### 4. Use searchable names — avoid magic numbers *(p. 22)*

A bare literal gives grep nothing to find and the reader nothing to understand.

#### Python

```python
# ❌ BAD — what does 7 mean?
if days > 7:
    send_reminder()

# ✅ GOOD
REMINDER_THRESHOLD_DAYS = 7
if days > REMINDER_THRESHOLD_DAYS:
    send_reminder()
```

#### TypeScript

```typescript
// ❌ BAD
if (days > 7) sendReminder();

// ✅ GOOD
const REMINDER_THRESHOLD_DAYS = 7;
if (days > REMINDER_THRESHOLD_DAYS) sendReminder();
```

---

### 5. Avoid encodings and type noise *(p. 23)*

Type prefixes (`str_`, `lst_`, `i`) are redundant when the type system already knows.

#### Python

```python
# ❌ BAD
str_user_name = "Jonathan"
i_user_age    = 51
lst_orders    = []

# ✅ GOOD
user_name = "Jonathan"
user_age  = 51
orders: list[Order] = []
```

#### TypeScript

```typescript
// ❌ BAD
const strUserName = "Jonathan";
const iUserAge    = 51;
const lstOrders: Order[] = [];

// ✅ GOOD
const userName = "Jonathan";
const userAge  = 51;
const orders: Order[] = [];
```

---

### 6. Be consistent — same pattern everywhere *(p. 26)*

Choose one style for a concept and use it everywhere. Mixed conventions force readers to decode instead of read.

#### Python

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

#### TypeScript

```typescript
// ❌ BAD
function fetchUserById(id: number) { ... }
function get_user_email(userId: number) { ... }
function loadProfile(uid: number) { ... }

// ✅ GOOD
function getUserById(userId: number): User { ... }
function getUserByEmail(email: string): User { ... }
function getUserProfile(userId: number): UserProfile { ... }
```

---

## Chapter 3 — Functions

### 7. Functions should be small and do ONE thing *(p. 35)*

If you can extract another function from a function with a name that is not merely a restatement of its implementation, the function is doing more than one thing.

#### Python

```python
# ❌ BAD — validates, creates, and notifies in one function
def register_user(data: dict) -> None:
    if not data.get("email"):
        raise ValueError("Email required")
    user = User(**data)
    db.add(user)
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

#### TypeScript

```typescript
// ❌ BAD — validates, creates, and notifies in one function
function registerUser(data: Record<string, string>): void {
  if (!data.email) throw new Error("Email required");
  const user = new User(data);
  db.add(user);
  sendWelcomeEmail(user.email);
}

// ✅ GOOD — each function has one job
function validateRegistrationData(data: Record<string, string>): void {
  if (!data.email) throw new Error("Email required");
}

function createUser(data: Record<string, string>): User {
  const user = new User(data);
  db.add(user);
  return user;
}

function registerUser(data: Record<string, string>): void {
  validateRegistrationData(data);
  const user = createUser(data);
  sendWelcomeEmail(user.email);
}
```

---

### 8. One level of abstraction per function *(p. 36)*

All statements inside a function must sit at the same conceptual level. Mixing high-level intent with low-level detail makes readers unable to tell what is essential and what is noise.

#### Python

```python
# ❌ BAD — high-level validate_user sits beside raw SQL cursor detail
def process_payment(payment: dict) -> Receipt:
    validate_user(payment["user_id"])
    conn = db.connect()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO payments (amount, user_id) VALUES (%s, %s)",
        (payment["amount"], payment["user_id"]),
    )
    conn.commit()
    return Receipt(id=cursor.lastrowid)

# ✅ GOOD — all statements at business-intent level
def process_payment(payment: dict) -> Receipt:
    validate_user(payment["user_id"])
    return persist_payment(payment)

def persist_payment(payment: dict) -> Receipt:
    conn = db.connect()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO payments (amount, user_id) VALUES (%s, %s)",
        (payment["amount"], payment["user_id"]),
    )
    conn.commit()
    return Receipt(id=cursor.lastrowid)
```

#### TypeScript

```typescript
// ❌ BAD — high-level validateUser() beside raw SQL string
async function processPayment(payment: PaymentInput): Promise<Receipt> {
  validateUser(payment.userId);
  const row = await db.query(
    "INSERT INTO payments (amount, user_id) VALUES ($1, $2) RETURNING *",
    [payment.amount, payment.userId],
  );
  return { id: row.rows[0].id };
}

// ✅ GOOD — every statement at the same abstraction level
async function processPayment(payment: PaymentInput): Promise<Receipt> {
  validateUser(payment.userId);
  return persistPayment(payment);
}

async function persistPayment(payment: PaymentInput): Promise<Receipt> {
  const row = await db.query(
    "INSERT INTO payments (amount, user_id) VALUES ($1, $2) RETURNING *",
    [payment.amount, payment.userId],
  );
  return { id: row.rows[0].id };
}
```

---

### 9. Stepdown Rule — file reads top-to-bottom at decreasing abstraction levels *(p. 37)*

Every function is followed by those at the next level of abstraction. The top-level function names each step as plain English; sub-functions contain the implementation detail. A reader understands the full flow without reading any sub-function body.

#### TypeScript

```typescript
// ❌ BAD — orchestration and raw implementation interleaved
export async function POST(request: Request) {
  const body = await request.json();
  if (!body.title || !body.signers?.length) throw new Error("Invalid input");
  const existing = await db.query(
    "SELECT COUNT(*) FROM envelopes WHERE user_id = $1", [userId]
  );
  if (existing.rows[0].count >= 10) throw new Error("Plan limit reached");
  const envelope = await db.query(
    "INSERT INTO envelopes (title, user_id) VALUES ($1, $2) RETURNING *",
    [body.title, userId],
  );
  for (const signer of body.signers) await sendEmail(signer.email, "Please sign");
  return Response.json(envelope.rows[0]);
}

// ✅ GOOD — top-level reads as a plain-English sequence; detail lives one level down
export async function POST(request: Request) {
  const data = await parseRequestBody(request);
  await validateCreateEnvelopeInput(data);
  await enforceEnvelopePlanLimit(auth.userId);
  const envelope = await createEnvelopeWithSigners(data, auth.userId);
  await dispatchSignerNotifications(envelope);
  return envelopeCreatedResponse(envelope);
}

async function parseRequestBody(request: Request): Promise<CreateEnvelopeInput> { ... }
async function validateCreateEnvelopeInput(data: CreateEnvelopeInput): Promise<void> { ... }
async function enforceEnvelopePlanLimit(userId: string): Promise<void> { ... }
async function createEnvelopeWithSigners(data: CreateEnvelopeInput, userId: string): Promise<Envelope> { ... }
async function dispatchSignerNotifications(envelope: Envelope): Promise<void> { ... }
function envelopeCreatedResponse(envelope: Envelope): Response { ... }
```

#### Python

```python
# ❌ BAD — orchestration and implementation mixed at the same level
def create_order(data: dict) -> Order:
    if not data.get("customer_id") or not data.get("items"):
        raise ValueError("customer_id and items required")
    rows = db.execute("SELECT COUNT(*) FROM orders WHERE customer_id = ?", data["customer_id"])
    if rows[0][0] >= 50:
        raise ValueError("Order limit reached")
    order = Order(customer_id=data["customer_id"])
    for item in data["items"]:
        order.lines.append(OrderLine(sku=item["sku"], qty=item["qty"]))
    db.save(order)
    email_client.send(order.customer.email, "Your order was placed")
    return order

# ✅ GOOD — top-level names each step; sub-functions immediately follow
def create_order(data: dict) -> Order:
    validated = validate_order_input(data)
    enforce_order_limit(validated.customer_id)
    order = build_order_with_lines(validated)
    notify_customer(order)
    return order

def validate_order_input(data: dict) -> OrderInput: ...
def enforce_order_limit(customer_id: int) -> None: ...
def build_order_with_lines(input: OrderInput) -> Order: ...
def notify_customer(order: Order) -> None: ...
```

---

### 10. Prefer fewer arguments — use parameter objects when needed *(p. 40)*

Functions with four or more arguments are hard to call correctly. Group related arguments into a typed object.

#### Python

```python
# ❌ BAD — six positional arguments are easy to transpose
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

#### TypeScript

```typescript
// ❌ BAD — six positional arguments
function createReport(
  title: string, author: string,
  startDate: Date, endDate: Date,
  includeCharts: boolean, format: string,
): Report { ... }

// ✅ GOOD
interface ReportConfig {
  title: string;
  author: string;
  startDate: Date;
  endDate: Date;
  includeCharts?: boolean;
  format?: "pdf" | "csv";
}

function createReport(config: ReportConfig): Report { ... }
```

---

### 11. Don't use flag arguments — split into separate functions *(p. 41)*

A boolean argument loudly declares that the function does more than one thing.

#### Python

```python
# ❌ BAD — caller must know what True/False means
def render_page(include_header: bool) -> str:
    if include_header:
        return render_with_header()
    return render_without_header()

# ✅ GOOD
def render_page_with_header() -> str: ...
def render_page() -> str: ...
```

#### TypeScript

```typescript
// ❌ BAD
function renderPage(includeHeader: boolean): string {
  return includeHeader ? renderWithHeader() : renderWithoutHeader();
}

// ✅ GOOD
function renderPageWithHeader(): string { ... }
function renderPage(): string { ... }
```

---

### 12. No side effects — a function should do what its name says and nothing else *(p. 44)*

A function named `get_user` must not also write to the database. Hidden mutations destroy trust in function names.

#### Python

```python
# ❌ BAD — name says "read", but function also writes
def get_user(user_id: int) -> User:
    user = db.query(user_id)
    user.last_accessed = datetime.now()   # hidden side effect
    db.save(user)
    return user

# ✅ GOOD — separate read from write
def get_user(user_id: int) -> User:
    return db.query(user_id)

def record_access(user: User) -> None:
    user.last_accessed = datetime.now()
    db.save(user)
```

#### TypeScript

```typescript
// ❌ BAD — name says "read", but function also writes
async function getUser(userId: string): Promise<User> {
  const user = await db.find(userId);
  user.lastAccessed = new Date();   // hidden side effect
  await db.save(user);
  return user;
}

// ✅ GOOD
async function getUser(userId: string): Promise<User> {
  return db.find(userId);
}

async function recordAccess(user: User): Promise<void> {
  user.lastAccessed = new Date();
  await db.save(user);
}
```

---

## Chapter 4 — Comments

### 13. Explain yourself in code, not in comments *(p. 55)*

If you need a comment to explain what the code does, rename it or extract a function.

#### Python

```python
# ❌ BAD — comment re-states what well-named code would say
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

#### TypeScript

```typescript
// ❌ BAD
// Check if the user is old enough to purchase alcohol
if (user.age >= 21) allowPurchase();

// ✅ GOOD
const MINIMUM_ALCOHOL_PURCHASE_AGE = 21;

function isEligibleToPurchaseAlcohol(user: User): boolean {
  return user.age >= MINIMUM_ALCOHOL_PURCHASE_AGE;
}

if (isEligibleToPurchaseAlcohol(user)) allowPurchase();
```

---

### 14. Use comments to explain INTENT, WARNINGS, and non-obvious decisions *(p. 55)*

The only good comments are ones that explain *why* — hidden constraints, performance trade-offs, or safety warnings that the code itself cannot express.

#### Python

```python
# ❌ BAD — restates the obvious
# Add item to list
items.append(item)

# ✅ GOOD — explains a non-obvious performance trade-off
# Sorting on every insert (O(n log n)) rather than at read time because
# reads outnumber writes 100:1 in the order processing pipeline.
items.append(item)
items.sort(key=lambda x: x.priority)

# ✅ GOOD — safety warning
# WARNING: Permanently deletes the user and all associated data.
# No soft-delete. Caller must present a confirmation dialog first.
def hard_delete_user(user_id: int) -> None:
    db.execute("DELETE FROM users WHERE id = ?", user_id)
```

#### TypeScript

```typescript
// ❌ BAD
// Add item to array
items.push(item);

// ✅ GOOD — explains a non-obvious trade-off
// Sorting on every insert rather than at read time because reads
// outnumber writes 100:1 in the order processing pipeline.
items.push(item);
items.sort((a, b) => a.priority - b.priority);

// ✅ GOOD — safety warning
// WARNING: Permanently deletes the user and all associated data.
// No soft-delete. Caller must present a confirmation dialog first.
async function hardDeleteUser(userId: string): Promise<void> {
  await db.execute("DELETE FROM users WHERE id = $1", [userId]);
}
```

---

### 15. Never comment out dead code — delete it *(p. 68)*

Commented-out code rots: it calls functions that no longer exist, uses names that have changed, and distracts every reader. Source control remembers everything.

#### Python

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

#### TypeScript

```typescript
// ❌ BAD
async function processPayment(amount: number): Promise<Receipt> {
  // await oldChargeService.charge(amount);
  // const legacyReceipt = oldReceiptService.create();
  return newPaymentService.charge(amount);
}

// ✅ GOOD
async function processPayment(amount: number): Promise<Receipt> {
  return newPaymentService.charge(amount);
}
```

---

## Chapter 5 — Formatting

### 16. Declare variables close to their usage *(p. 80)*

A variable declared 30 lines before its first use forces readers to scroll and remember. Declare it just above its first use.

#### Python

```python
# ❌ BAD — tax_rate declared far from use
def process_order(order: Order) -> None:
    tax_rate = Decimal("0.08")
    # ... 30 lines of unrelated logic ...
    subtotal = sum(item.price for item in order.items)
    total = subtotal * (1 + tax_rate)

# ✅ GOOD
def process_order(order: Order) -> None:
    # ... other logic ...
    TAX_RATE = Decimal("0.08")
    subtotal = sum(item.price for item in order.items)
    total    = subtotal * (1 + TAX_RATE)
```

#### TypeScript

```typescript
// ❌ BAD
function processOrder(order: Order): void {
  const TAX_RATE = 0.08;
  // ... 30 lines of unrelated logic ...
  const subtotal = order.items.reduce((s, i) => s + i.price, 0);
  const total    = subtotal * (1 + TAX_RATE);
}

// ✅ GOOD
function processOrder(order: Order): void {
  // ... other logic ...
  const TAX_RATE = 0.08;
  const subtotal = order.items.reduce((s, i) => s + i.price, 0);
  const total    = subtotal * (1 + TAX_RATE);
}
```

---

### 17. Place caller above callee — high-level first *(p. 84)*

Source files should read like a newspaper: headline at the top, detail below. Every function should appear above the functions it calls.

#### Python

```python
# ✅ GOOD — public API at top, private helpers below
class ReportService:
    def generate_report(self, config: ReportConfig) -> Report:
        data    = self._fetch_data(config)
        summary = self._summarize(data)
        return self._format(summary, config)

    def _fetch_data(self, config: ReportConfig) -> list: ...
    def _summarize(self, data: list) -> dict: ...
    def _format(self, summary: dict, config: ReportConfig) -> Report: ...
```

#### TypeScript

```typescript
// ✅ GOOD — public method at top, private helpers below
class ReportService {
  generateReport(config: ReportConfig): Report {
    const data    = this.fetchData(config);
    const summary = this.summarize(data);
    return this.format(summary, config);
  }

  private fetchData(config: ReportConfig): Row[] { ... }
  private summarize(data: Row[]): Summary { ... }
  private format(summary: Summary, config: ReportConfig): Report { ... }
}
```

---

## Chapter 6 — Objects and Data Structures

### 18. Hide internal structure — expose behavior, not data *(p. 93)*

Objects hide their data behind abstractions and expose functions that operate on that data. Exposing raw fields lets callers bypass invariants.

#### Python

```python
# ❌ BAD — caller bypasses business rules
class BankAccount:
    balance: Decimal = Decimal(0)

account.balance += Decimal("100")   # no validation

# ✅ GOOD
class BankAccount:
    def __init__(self) -> None:
        self._balance = Decimal(0)

    def deposit(self, amount: Decimal) -> None:
        if amount <= 0:
            raise ValueError("Deposit must be positive")
        self._balance += amount

    @property
    def balance(self) -> Decimal:
        return self._balance
```

#### TypeScript

```typescript
// ❌ BAD — caller bypasses business rules
class BankAccount { balance = 0; }
account.balance += 100;   // no validation

// ✅ GOOD
class BankAccount {
  private _balance = 0;

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Deposit must be positive");
    this._balance += amount;
  }

  get balance(): number { return this._balance; }
}
```

---

### 19. Follow the Law of Demeter — talk only to direct neighbors *(p. 97)*

A function should only call methods on: itself, its parameters, any objects it creates, and its direct component objects. Train-wrecks (`a.b().c().d()`) expose internal structure and create brittle coupling.

#### Python

```python
# ❌ BAD — chains through two levels of internals
def print_customer_city(order: Order) -> None:
    print(order.customer.address.city)

# ✅ GOOD — Order exposes a method for what callers need
class Order:
    def customer_city(self) -> str:
        return self.customer.address.city

def print_customer_city(order: Order) -> None:
    print(order.customer_city())
```

#### TypeScript

```typescript
// ❌ BAD — chains through two levels of internals
function printCustomerCity(order: Order): void {
  console.log(order.customer.address.city);
}

// ✅ GOOD
class Order {
  customerCity(): string { return this.customer.address.city; }
}

function printCustomerCity(order: Order): void {
  console.log(order.customerCity());
}
```

---

## Chapter 7 — Error Handling

### 20. Always find the root cause — don't suppress errors silently *(p. 103)*

Swallowing an exception hides failures. Callers cannot distinguish "record not found" from "database is down".

#### Python

```python
# ❌ BAD — silent swallow masks the real problem
def get_user(user_id: int):
    try:
        return db.query(user_id)
    except Exception:
        return None

# ✅ GOOD — log and re-raise so callers and ops can see the failure
def get_user(user_id: int) -> User:
    try:
        return db.query(user_id)
    except DatabaseError as e:
        logger.error("DB failure fetching user %s: %s", user_id, e)
        raise
```

#### TypeScript

```typescript
// ❌ BAD — silent catch hides the real problem
async function getUser(userId: string): Promise<User | null> {
  try {
    return await db.find(userId);
  } catch {
    return null;
  }
}

// ✅ GOOD
async function getUser(userId: string): Promise<User> {
  try {
    return await db.find(userId);
  } catch (err) {
    logger.error({ userId, err }, "DB failure fetching user");
    throw err;
  }
}
```

---

## Chapter 9 — Unit Tests

### 21. One logical assertion per test *(p. 130)*

Testing multiple concepts in one test hides which concept failed. One concept per test makes failures self-diagnosing.

#### Python

```python
# ❌ BAD — four concepts, but the name says "creation"
def test_user_creation():
    user = create_user("jonathan@example.com", "Jonathan")
    assert user.email == "jonathan@example.com"
    assert user.name == "Jonathan"
    assert user.is_active is True
    assert user.created_at is not None

# ✅ GOOD — one concept each
def test_user_stores_correct_email():
    assert create_user("jonathan@example.com", "Jonathan").email == "jonathan@example.com"

def test_new_user_is_active_by_default():
    assert create_user("jonathan@example.com", "Jonathan").is_active is True
```

#### TypeScript

```typescript
// ❌ BAD
it("creates a user", () => {
  const user = createUser("jonathan@example.com", "Jonathan");
  expect(user.email).toBe("jonathan@example.com");
  expect(user.isActive).toBe(true);
});

// ✅ GOOD
it("stores the correct email", () => {
  expect(createUser("jonathan@example.com", "Jonathan").email).toBe("jonathan@example.com");
});

it("activates new users by default", () => {
  expect(createUser("jonathan@example.com", "Jonathan").isActive).toBe(true);
});
```

---

### 22. Fast — tests should run in milliseconds; no real I/O *(F.I.R.S.T., p. 132)*

Slow tests don't get run. No real databases, no real HTTP, no real files.

#### Python

```python
# ❌ BAD — hits real database, brittle and slow
def test_get_user():
    user = UserService().get_user(1)
    assert user.name == "Jonathan"

# ✅ GOOD — mocked dependency, millisecond execution
def test_get_user(mocker):
    mock_repo = mocker.Mock()
    mock_repo.find_by_id.return_value = User(id=1, name="Jonathan")
    service = UserService(repo=mock_repo)
    assert service.get_user(1).name == "Jonathan"
```

#### TypeScript

```typescript
// ❌ BAD — hits real database
it("gets a user", async () => {
  const user = await new UserService().getUser("1");
  expect(user.name).toBe("Jonathan");
});

// ✅ GOOD
it("gets a user", async () => {
  const repo = { findById: jest.fn().mockResolvedValue({ id: "1", name: "Jonathan" }) };
  const user = await new UserService(repo).getUser("1");
  expect(user.name).toBe("Jonathan");
});
```

---

### 23. Independent — tests must not depend on each other *(F.I.R.S.T., p. 132)*

Each test must set up its own state. Shared state causes cascade failures that are hard to diagnose.

#### Python

```python
# ❌ BAD — test_total fails if run without test_add_item
shared_cart = ShoppingCart()

def test_add_item():
    shared_cart.add(Item("book", 10))

def test_total():
    assert shared_cart.total() == 10

# ✅ GOOD — each test owns its setup
def test_total():
    cart = ShoppingCart()
    cart.add(Item("book", 10))
    assert cart.total() == 10
```

#### TypeScript

```typescript
// ❌ BAD — shared state between tests
const sharedCart = new ShoppingCart();
it("adds an item", () => sharedCart.add({ name: "book", price: 10 }));
it("totals items",  () => expect(sharedCart.total()).toBe(10));   // fails if run alone

// ✅ GOOD
it("totals items", () => {
  const cart = new ShoppingCart();
  cart.add({ name: "book", price: 10 });
  expect(cart.total()).toBe(10);
});
```

---

### 24. Readable — test names describe behavior, not implementation *(F.I.R.S.T., p. 132)*

Test names are documentation. Name them so the failure message tells you what broke.

#### Python

```python
# ❌ BAD
def test_func1():
    assert calc(2, 3) == 5

# ✅ GOOD
def test_add_returns_sum_of_two_positive_integers():
    assert calculator.add(2, 3) == 5

def test_add_with_negative_addend_returns_correct_sum():
    assert calculator.add(-1, 5) == 4
```

#### TypeScript

```typescript
// ❌ BAD
it("test1", () => expect(calc(2, 3)).toBe(5));

// ✅ GOOD
it("returns the sum of two positive integers", () => {
  expect(calculator.add(2, 3)).toBe(5);
});

it("handles a negative addend correctly", () => {
  expect(calculator.add(-1, 5)).toBe(4);
});
```

---

## Chapter 10 — Classes

### 25. Single Responsibility — classes should have one reason to change *(p. 138)*

If a class handles persistence, notifications, and validation, then any one of those three domains can force it to change.

#### Python

```python
# ❌ BAD — three reasons to change
class User:
    def save(self): ...         # changes when persistence changes
    def send_email(self): ...   # changes when email changes
    def validate(self): ...     # changes when rules change

# ✅ GOOD
class User: ...                        # domain model only
class UserRepository: ...              # persistence
class UserNotificationService: ...     # email/notifications
class UserValidator: ...               # business rules
```

#### TypeScript

```typescript
// ❌ BAD
class User {
  save(): void { ... }
  sendEmail(): void { ... }
  validate(): void { ... }
}

// ✅ GOOD
class User { ... }
class UserRepository { ... }
class UserNotificationService { ... }
class UserValidator { ... }
```

---

## Chapter 11 — Systems

### 26. Use dependency injection — don't instantiate collaborators inside classes *(p. 154)*

Hard-coding `new ConcreteRepo()` inside a class couples it to that implementation and makes tests impossible without the real database.

#### Python

```python
# ❌ BAD — hard-coded, untestable
class OrderService:
    def __init__(self):
        self.repo = PostgresOrderRepository()

# ✅ GOOD — injected, mockable
class OrderService:
    def __init__(self, repo: OrderRepository) -> None:
        self.repo = repo
```

#### TypeScript

```typescript
// ❌ BAD — hard-coded, untestable
class OrderService {
  private repo = new PostgresOrderRepository();
}

// ✅ GOOD — injected, mockable
class OrderService {
  constructor(private readonly repo: OrderRepository) {}
}
```

---

## Chapter 13 — Concurrency

### 27. Separate multi-threading / async code from business logic *(p. 177)*

Business logic must be testable without concurrency infrastructure. Keep locks and queues in a thin wrapper layer.

#### Python

```python
# ❌ BAD — business rule (sum) tangled with lock management
def process_order(order: Order) -> None:
    lock.acquire()
    try:
        total = sum(item.price for item in order.items)
        db.save(order, total)
    finally:
        lock.release()

# ✅ GOOD — pure function for business logic; async wrapper handles concurrency
def calculate_order_total(order: Order) -> Decimal:
    return sum(item.price for item in order.items)

async def process_order_async(order: Order) -> None:
    async with db_lock:
        total = calculate_order_total(order)
        await db.save(order, total)
```

#### TypeScript

```typescript
// ❌ BAD — business rule tangled with mutex
async function processOrder(order: Order): Promise<void> {
  await mutex.acquire();
  try {
    const total = order.items.reduce((s, i) => s + i.price, 0);
    await db.save(order, total);
  } finally {
    mutex.release();
  }
}

// ✅ GOOD — pure function for business logic; wrapper handles concurrency
function calculateOrderTotal(order: Order): number {
  return order.items.reduce((s, i) => s + i.price, 0);
}

async function processOrder(order: Order): Promise<void> {
  await mutex.runExclusive(async () => {
    const total = calculateOrderTotal(order);
    await db.save(order, total);
  });
}
```

---

## Additional Principles

*(Not tied to a single chapter; applied throughout the book.)*

### 28. Keep it simple (KISS) — reduce complexity

Never solve the simple problem with a complex solution.

#### Python

```python
# ❌ BAD — approximates a known formula with unnecessary iteration
def calculate_circle_area(radius: float) -> float:
    area = 0.0
    for i in range(360):
        area += (math.pi / 180) * radius * radius
    return area

# ✅ GOOD
def calculate_circle_area(radius: float) -> float:
    return math.pi * radius ** 2
```

#### TypeScript

```typescript
// ❌ BAD
function calculateCircleArea(radius: number): number {
  let area = 0;
  for (let i = 0; i < 360; i++) area += (Math.PI / 180) * radius * radius;
  return area;
}

// ✅ GOOD
function calculateCircleArea(radius: number): number {
  return Math.PI * radius ** 2;
}
```

---

### 29. Prefer value objects over primitives

Raw strings and numbers carry no invariants. A value object validates on construction, making illegal states unrepresentable.

#### Python

```python
# ❌ BAD — nothing stops the caller from passing age where email is expected
def create_account(email: str, age: int) -> None: ...

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

#### TypeScript

```typescript
// ❌ BAD — nothing stops the caller from transposing email and age
function createAccount(email: string, age: number): void { ... }

// ✅ GOOD — branded types prevent transposition at compile time
type Email = string & { readonly __brand: "Email" };
type Age   = number & { readonly __brand: "Age" };

function parseEmail(raw: string): Email {
  if (!raw.includes("@")) throw new Error(`Invalid email: ${raw}`);
  return raw as Email;
}

function parseAge(raw: number): Age {
  if (raw < 0) throw new Error("Age cannot be negative");
  return raw as Age;
}

function createAccount(email: Email, age: Age): void { ... }
```

---

### 30. Rigidity — don't make changes that cascade everywhere

A change to one requirement should ripple through as few modules as possible. If adding a payment type means editing five files, the design is too rigid.

#### Python

```python
# ❌ BAD — adding "ACH" means editing this function and re-testing everything
def process_payment(payment_type: str, amount: Decimal) -> None:
    if payment_type == "credit":
        credit_processor.charge(amount)
    elif payment_type == "crypto":
        crypto_processor.charge(amount)

# ✅ GOOD — open for extension, closed for modification (OCP)
class PaymentProcessor(Protocol):
    def charge(self, amount: Decimal) -> None: ...

def process_payment(processor: PaymentProcessor, amount: Decimal) -> None:
    processor.charge(amount)
# Adding ACH = new class only, no edits to existing code
```

#### TypeScript

```typescript
// ❌ BAD — every new payment type requires editing this switch
function processPayment(type: string, amount: number): void {
  if (type === "credit") creditProcessor.charge(amount);
  else if (type === "crypto") cryptoProcessor.charge(amount);
}

// ✅ GOOD
interface PaymentProcessor {
  charge(amount: number): void;
}

function processPayment(processor: PaymentProcessor, amount: number): void {
  processor.charge(amount);
}
```

---

### 31. Opacity — deeply nested code is hard to read; flatten it

Each nesting level forces the reader to hold more context. Guard clauses (early returns) eliminate nesting by handling the exceptional cases first.

#### Python

```python
# ❌ BAD — four levels of nesting to reach the action
def process_invoice(invoice):
    if invoice:
        if invoice.amount > 0:
            if invoice.status == "pending":
                if invoice.customer:
                    charge(invoice)

# ✅ GOOD — guard clauses; the happy path is obvious
def process_invoice(invoice: Invoice | None) -> None:
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

#### TypeScript

```typescript
// ❌ BAD
function processInvoice(invoice: Invoice | null): void {
  if (invoice) {
    if (invoice.amount > 0) {
      if (invoice.status === "pending") {
        if (invoice.customer) charge(invoice);
      }
    }
  }
}

// ✅ GOOD — guard clauses
function processInvoice(invoice: Invoice | null): void {
  if (!invoice) return;
  if (invoice.amount <= 0) return;
  if (invoice.status !== "pending") return;
  if (!invoice.customer) return;
  charge(invoice);
}
```

---

### 32. Needless complexity — don't over-engineer

Don't introduce abstractions for hypothetical future needs. Three similar lines is better than a premature abstraction.

#### TypeScript

```typescript
// ❌ BAD — Strategy pattern for a single, fixed greeting that will never vary
interface GreetingStrategy { execute(name: string): string; }
class FormalGreeting implements GreetingStrategy {
  execute(name: string) { return `Good day, ${name}.`; }
}
class GreetingContext {
  constructor(private strategy: GreetingStrategy) {}
  greet(name: string) { return this.strategy.execute(name); }
}

// ✅ GOOD
function greet(name: string): string { return `Good day, ${name}.`; }
```

#### Python

```python
# ❌ BAD — abstract base class for a single, fixed greeting
from abc import ABC, abstractmethod

class GreetingStrategy(ABC):
    @abstractmethod
    def execute(self, name: str) -> str: ...

class FormalGreeting(GreetingStrategy):
    def execute(self, name: str) -> str:
        return f"Good day, {name}."

# ✅ GOOD
def greet(name: str) -> str:
    return f"Good day, {name}."
```

---

## TypeScript-Specific Clean Code

### 33. Prefer discriminated unions over boolean flags

Boolean flags multiply ambiguous states. Discriminated unions make every state explicit and exhaustively checkable.

```typescript
// ❌ BAD — isLoading and isError can both be true; data can exist with an error
type ApiResponse = {
  data?: User;
  error?: string;
  isLoading: boolean;
  isError: boolean;
};

// ✅ GOOD — exactly one state is possible at a time
type ApiResponse =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error";   error: string };
```

---

### 34. Use `unknown` over `any` — force explicit narrowing

`any` disables the type checker entirely. `unknown` requires the caller to prove the type before using the value.

```typescript
// ❌ BAD — runtime explosion if raw lacks a numeric timeout
function parseConfig(raw: any) {
  return raw.timeout * 1000;
}

// ✅ GOOD — the type checker enforces validation before use
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

## Chapter 17 — Smells and Heuristics (Reference)

A complete list of Martin's named smells and heuristics. Rules with full examples above are cross-referenced. Use this list as a final checklist when reviewing a file.

### Comments

| Code | Smell | Action |
|------|-------|--------|
| C1 | Inappropriate Information — change history, author metadata, ticket numbers in comments | Move to source control / issue tracker |
| C2 | Obsolete Comment — comment no longer matches the code | Update or delete |
| C3 | Redundant Comment — explains what the code already says | Delete |
| C4 | Poorly Written Comment — grammatically unclear, rambling | Rewrite clearly or delete |
| C5 | Commented-Out Code | Delete immediately — see **Rule 15** |

### Environment

| Code | Smell | Action |
|------|-------|--------|
| E1 | Build Requires More Than One Step | Automate to one command |
| E2 | Tests Require More Than One Step | Automate to one command |

### Functions

| Code | Smell | Action |
|------|-------|--------|
| F1 | Too Many Arguments — more than three is very questionable | See **Rule 10** |
| F2 | Output Arguments — arguments used as outputs, not inputs | Return a value instead |
| F3 | Flag Arguments — boolean argument selects behavior | See **Rule 11** |
| F4 | Dead Function — method never called | Delete it |

### General

| Code | Smell | Action |
|------|-------|--------|
| G1 | Multiple Languages in One Source File | Minimize; aim for one language per file |
| G2 | Obvious Behavior Is Unimplemented — function ignores expected edge cases | Implement what callers would reasonably expect |
| G3 | Incorrect Behavior at the Boundaries — untested edge cases | Write a test for every boundary condition |
| G4 | Overridden Safeties — disabled compiler warnings, skipped tests | Re-enable; fix root cause |
| G5 | Duplication | See **Rule 28** (DRY) |
| G6 | Code at Wrong Level of Abstraction — high-level and detail mixed | See **Rule 8** (One Level of Abstraction) |
| G7 | Base Classes Depending on Their Derivatives | See **Rule 29** (Base classes) |
| G8 | Too Much Information — fat interfaces, many public fields | Narrow the public surface |
| G9 | Dead Code — unreachable code, unused variables | Delete it |
| G10 | Vertical Separation — variable used far from its declaration | See **Rule 16** |
| G11 | Inconsistency — similar things done in dissimilar ways | See **Rule 6** (Consistency) |
| G12 | Clutter — unused variables, dead functions, meaningless comments | Remove |
| G13 | Artificial Coupling — unrelated things coupled for convenience | Move each to its natural home |
| G14 | Feature Envy — method uses another class's data more than its own | Move the method or the data |
| G15 | Selector Arguments — boolean/enum arg that selects function behavior | See **Rule 11** (Flag Arguments) |
| G16 | Obscured Intent — run-on expressions, Hungarian notation, magic numbers | See **Rules 2–5** (Naming) |
| G17 | Misplaced Responsibility — code is in a surprising place | Move it where readers would naturally look |
| G18 | Inappropriate Static — static method that should be polymorphic | Make it an instance method or virtual |
| G19 | Use Explanatory Variables | See **Rule 30** |
| G20 | Function Names Should Say What They Do — `rename` that also verifies? | Rename or split |
| G21 | Understand the Algorithm — code that works by accident | Replace with code you can reason about |
| G22 | Make Logical Dependencies Physical — implicit call-order requirements | Pass the result of one call as the argument to the next |
| G23 | Prefer Polymorphism to If/Else or Switch/Case | See **Rule 31** |
| G24 | Follow Standard Conventions | See **Rule 32** |
| G25 | Replace Magic Numbers with Named Constants | See **Rule 4** |
| G26 | Be Precise — vague types, ambiguous returns | Use the most specific type; be explicit about failure |
| G27 | Structure Over Convention — naming conventions that can be violated | Enforce via types and compiler, not just naming |
| G28 | Encapsulate Conditionals — `if (timer.hasExpired() && !timer.isRecurrent())` | Extract to `if (shouldBeDeleted(timer))` |
| G29 | Avoid Negative Conditionals | See **Rule 33** |
| G30 | Functions Should Do One Thing | See **Rule 7** |
| G31 | Hidden Temporal Couplings — functions must be called in a specific order, but nothing enforces it | Chain return values so out-of-order calls are impossible |
| G32 | Don't Be Arbitrary — code structure that has no apparent reason | Document the reason or restructure |
| G33 | Encapsulate Boundary Conditions — `+1` and `-1` scattered everywhere | See **Rule 34** |
| G34 | Functions Should Descend Only One Level of Abstraction | See **Rule 8** (One Level of Abstraction) |
| G35 | Keep Configurable Data at High Levels | See **Rule 35** |
| G36 | Avoid Transitive Navigation (Train Wrecks) — `a.getB().getC()` | See **Rule 19** (Law of Demeter) |

### Names

| Code | Smell | Action |
|------|-------|--------|
| N1 | Choose Descriptive Names | See **Rule 2** |
| N2 | Choose Names at the Appropriate Level of Abstraction — `phoneNumber` on a generic connection interface | Use `connectionLocator` |
| N3 | Use Standard Nomenclature — use `Decorator`, `Factory`, `Iterator` in names when the pattern is applied | See **Rule 32** (Conventions) |
| N4 | Unambiguous Names — `rename` that also verifies? | Name it `renameOrVerify` or split |
| N5 | Use Long Names for Long Scopes — single-letter names for loop variables only when the scope is tiny | Expand names as scope grows |
| N6 | Avoid Encodings | See **Rule 5** |
| N7 | Names Should Describe Side Effects — a function that creates and returns a singleton should not be called `getObjectManager` | Call it `createObjectManagerIfAbsent` |

### Tests

| Code | Smell | Action |
|------|-------|--------|
| T1 | Insufficient Tests — "it seems like enough" is not a test strategy | Test every condition that could possibly break |
| T2 | Use a Coverage Tool | Uncovered lines are untested assumptions |
| T3 | Don't Skip Trivial Tests — they document behavior cheaply | Write them |
| T4 | An Ignored Test Is a Question about an Ambiguity | Resolve the ambiguity; then either delete or enable the test |
| T5 | Test Boundary Conditions | See **Rule 34** (Encapsulate Boundary Conditions) |
| T6 | Exhaustively Test Near Bugs — where you find one bug, look for more | Write tests for all neighboring behavior |
| T7 | Patterns of Failure Are Revealing — tests that fail together hint at a shared cause | Read the pattern; don't just fix individual failures |
| T8 | Test Coverage Patterns Can Be Revealing — untested lines that are always executed together | Consider extracting a unit |
| T9 | Tests Should Be Fast | See **Rule 22** |

---

## Summary Checklist

Before committing, verify:

- [ ] Names are descriptive — no abbreviations, no magic numbers
- [ ] Functions do ONE thing and are under ~20 lines
- [ ] All statements in a function are at the same level of abstraction
- [ ] File reads top-to-bottom: caller appears above callee (Stepdown Rule)
- [ ] No flag arguments (split into separate functions)
- [ ] No side effects that aren't obvious from the function name
- [ ] No commented-out code
- [ ] No deeply nested conditionals (use guard clauses)
- [ ] No duplication (DRY)
- [ ] Configurable values are constants at the top, not buried inline
- [ ] Tests are fast, independent, and named after behavior
- [ ] Classes have a single reason to change (SRP)
- [ ] Dependencies are injected, not instantiated inside classes
- [ ] No `any` in TypeScript without an explicit suppression comment explaining why
