# Engineering Standards — Claude Instructions

This file governs Claude's behaviour both when working **inside this repository** and when this
repository is **referenced from a project's CLAUDE.md**.

---

## Part 1 — Working Inside This Repo

### Pattern File Rules

- Every pattern file in `patterns/` must follow the template in `engineering-standards-plan.md`
- TypeScript and Python examples are both required for P1 and P2 patterns
- Examples must be self-contained — no imports from application frameworks (Next.js, FastAPI, etc.)
- Examples must use realistic domain names (orders, users, payments) not abstract names (A, B, ConcreteStrategyA)
- The ❌ Without section must show a real problem (not just "here is the bad way")
- P3 patterns use the stub template — do not add full examples until explicitly asked

### Standards File Rules

- `CLEAN-CODE.md` — do not change rule numbering; keeps rule text only. Code examples live in
  `examples/clean-code/py.md` and `examples/clean-code/ts.md`, keyed by rule number (`## Rule N`).
  When you add or change a rule, update the matching `## Rule N` section in both example files.
- `CLEAN-ARCHITECTURE.md` — keeps principle text only; layer diagram must stay intact. SOLID code
  examples live in `examples/clean-architecture/py.md` and `examples/clean-architecture/ts.md`,
  keyed by principle letter (`## S`, `## O`, …). When you add or change a principle, update the
  matching section in both example files.

### Adding a New Pattern

1. Choose the correct category directory (`creational/`, `structural/`, `behavioral/`)
2. Copy the template from `engineering-standards-plan.md`
3. Extract Intent and Applicability from the GoF book PDF (`articulo.pdf`, offset: book page + 19 = 0-based PDF index)
4. Cross-check against `https://refactoring.guru/design-patterns/<name>` for structural accuracy
5. Write both TypeScript and Python bad/good examples with domain-specific code
6. Update `patterns/README.md` decision tables
7. Mark the item `[x]` in `engineering-standards-plan.md`

---

## Part 2 — Standing Instructions for All Projects

When this file is referenced from a project's `CLAUDE.md`, the following rules are **always active**
during any code reading, writing, or reviewing task.

### Automatic Pattern Detection

While reading or writing code, silently check for the following signals. Do not narrate the check —
only surface findings when there is a clear match.

**Pattern opportunity signals (code that SHOULD use a pattern but doesn't):**

| Signal in code | Pattern to suggest |
|---|---|
| `if/else` or `switch` that dispatches on a type string/enum to call different implementations | Strategy |
| Multiple classes with nearly identical method bodies differing only in a few lines | Template Method |
| Hard-coded `new ConcreteClass()` scattered through business logic | Factory Method or Abstract Factory |
| Constructor with 4+ parameters, many optional | Builder |
| Cross-cutting behaviour (logging, caching, auth, retry) added via subclass | Decorator |
| Code calling 3+ subsystems to complete one user-facing operation | Facade |
| Calling a third-party SDK directly in business logic | Adapter |
| Objects polling or checking state of another object | Observer |
| Long chain of `if/else` where each handler either processes or passes | Chain of Responsibility |
| Operations that need queuing, undo, or replay | Command |

**Pattern misuse signals (code using a pattern incorrectly):**

| Signal | Issue |
|---|---|
| Strategy with only one ConcreteStrategy that never changes | Unnecessary abstraction — remove |
| Decorator that also changes the interface (not just wraps) | That's an Adapter, not a Decorator |
| Singleton with mutable shared state | Threading/test hazard — flag |
| Observer with no unsubscribe mechanism in a long-lived process | Memory leak risk |
| Abstract Factory where all concrete factories produce the same types | Collapse to Factory Method |

### Flagging Convention

When you identify a pattern opportunity or misuse in **existing** code (not code you just wrote),
add a structured TODO comment at the relevant location:

```typescript
// TODO [PATTERN:strategy]: Replace switch-on-type with Strategy pattern.
// Each payment provider should be a ConcreteStrategy implementing PaymentGateway.
// Reference: patterns/behavioral/strategy.md (in engineering-standards repo)
```

```python
# TODO [PATTERN:factory-method]: Hard-coded `new EmailNotifier()` should go behind
# a factory method so callers depend on the Notifier interface, not the concrete class.
# Reference: patterns/creational/factory-method.md (in engineering-standards repo)
```

Rules for TODOs:
- Format: `TODO [PATTERN:<pattern-name>]: <one sentence describing the fix>`
- Always include the `Reference:` line pointing to the pattern file
- Place the comment on the line immediately above the offending code
- Do not add a TODO if the pattern is already correctly applied
- Do not add a TODO if the code is simple enough that the pattern would be overkill

### Code Quality Standards

All code written or modified in any project must comply with:

- **Clean Code rules** — `~/engineering-standards/CLEAN-CODE.md`
- **Architecture rules** — `~/engineering-standards/CLEAN-ARCHITECTURE.md`
- **SOLID principles** — see SOLID section in `CLEAN-ARCHITECTURE.md`

Key non-negotiables (check these on every function or class you write):

1. Functions do one thing — no side-effect + return value mixing
2. No magic numbers — extract named constants
3. Dependency Rule — inner layers never import outer layers
4. Depend on abstractions (interfaces/protocols), not concretions
5. No `any` in TypeScript without an explicit suppression comment explaining why

### When Asked to Refactor

When the user asks you to refactor a file or function:

1. Identify all `TODO [PATTERN:*]` comments already present — address them first
2. Scan for new pattern opportunities using the detection table above
3. Apply patterns bottom-up: fix the most fundamental violation before the higher-level ones
4. For each pattern applied, briefly state which pattern and why in your response — not as a code comment
5. Run a final Clean Code pass (naming, function size, guard clauses) after structural changes

---

## Part 3 — Available Skills

Install with: `ln -s ~/engineering-standards/skills/<name>.md ~/.claude/commands/<name>.md`

| Skill | Command | Purpose |
|---|---|---|
| `skills/clean-review.md` | `/clean-review` | Check a file against all 32 CLEAN-CODE rules + flag pattern opportunities |
| `skills/apply-pattern.md` | `/apply-pattern <name>` | Apply a named GoF pattern to the current file or selection |
| `skills/architecture-check.md` | `/architecture-check` | Verify the file respects Clean Architecture layer boundaries |
| `skills/refactor.md` | `/refactor` | Full refactor pass: patterns + clean code + architecture |

---

## Reference Paths

Paths below are relative to the root of the engineering-standards repository.

| Resource | Location |
|---|---|
| Clean Code rules | `CLEAN-CODE.md` |
| Clean Architecture rules | `CLEAN-ARCHITECTURE.md` |
| Pattern index + decision tree | `patterns/README.md` |
| Individual patterns | `patterns/<category>/<name>.md` |
