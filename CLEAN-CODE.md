# Clean Code Standards (Robert C. Martin)

Rules for writing clean code, organized by chapter. Page references are to *Clean Code* (Martin, 2008).

> **How to use this file when reviewing code**
> Scan top-to-bottom. Each rule has a one-line summary. If code violates the
> rule, look up the ❌ BAD / ✅ GOOD examples in the language-specific example
> file and fix it to match the ✅ GOOD version.
>
> **Where the examples live**
> Code examples are split into two files, keyed by rule number, so reviewers
> only load the language they need — Python in
> [`examples/clean-code/py.md`](examples/clean-code/py.md) and TypeScript in
> [`examples/clean-code/ts.md`](examples/clean-code/ts.md). To see Rule N in
> action, jump to the `## Rule N` section of the relevant file.

---

## Chapter 1 — Clean Code

### 1. Boy Scout Rule — leave code cleaner than you found it *(p. 14)*

Remove dead code, fix obvious naming, add missing type hints when you touch a file.

---

## Chapter 2 — Meaningful Names

### 2. Use descriptive, unambiguous names *(p. 18)*

Names must communicate intent without requiring the reader to look elsewhere.

---

### 3. Use pronounceable names *(p. 21)*

If you can't say a name aloud, you can't discuss it with colleagues.

---

### 4. Use searchable names — avoid magic numbers *(p. 22)*

A bare literal gives grep nothing to find and the reader nothing to understand.

---

### 5. Avoid encodings and type noise *(p. 23)*

Type prefixes (`str_`, `lst_`, `i`) are redundant when the type system already knows.

---

### 6. Be consistent — same pattern everywhere *(p. 26)*

Choose one style for a concept and use it everywhere. Mixed conventions force readers to decode instead of read.

---

## Chapter 3 — Functions

### 7. Functions should be small and do ONE thing *(p. 35)*

If you can extract another function from a function with a name that is not merely a restatement of its implementation, the function is doing more than one thing.

---

### 8. One level of abstraction per function *(p. 36)*

All statements inside a function must sit at the same conceptual level. Mixing high-level intent with low-level detail makes readers unable to tell what is essential and what is noise.

---

### 9. Stepdown Rule — file reads top-to-bottom at decreasing abstraction levels *(p. 37)*

Every function is followed by those at the next level of abstraction. The top-level function names each step as plain English; sub-functions contain the implementation detail. A reader understands the full flow without reading any sub-function body.

---

### 10. Prefer fewer arguments — use parameter objects when needed *(p. 40)*

Functions with four or more arguments are hard to call correctly. Group related arguments into a typed object.

---

### 11. Don't use flag arguments — split into separate functions *(p. 41)*

A boolean argument loudly declares that the function does more than one thing.

---

### 12. No side effects — a function should do what its name says and nothing else *(p. 44)*

A function named `get_user` must not also write to the database. Hidden mutations destroy trust in function names.

---

## Chapter 4 — Comments

### 13. Explain yourself in code, not in comments *(p. 55)*

If you need a comment to explain what the code does, rename it or extract a function.

---

### 14. Use comments to explain INTENT, WARNINGS, and non-obvious decisions *(p. 55)*

The only good comments are ones that explain *why* — hidden constraints, performance trade-offs, or safety warnings that the code itself cannot express.

---

### 15. Never comment out dead code — delete it *(p. 68)*

Commented-out code rots: it calls functions that no longer exist, uses names that have changed, and distracts every reader. Source control remembers everything.

---

## Chapter 5 — Formatting

### 16. Declare variables close to their usage *(p. 80)*

A variable declared 30 lines before its first use forces readers to scroll and remember. Declare it just above its first use.

---

### 17. Place caller above callee — high-level first *(p. 84)*

Source files should read like a newspaper: headline at the top, detail below. Every function should appear above the functions it calls.

---

## Chapter 6 — Objects and Data Structures

### 18. Hide internal structure — expose behavior, not data *(p. 93)*

Objects hide their data behind abstractions and expose functions that operate on that data. Exposing raw fields lets callers bypass invariants.

---

### 19. Follow the Law of Demeter — talk only to direct neighbors *(p. 97)*

A function should only call methods on: itself, its parameters, any objects it creates, and its direct component objects. Train-wrecks (`a.b().c().d()`) expose internal structure and create brittle coupling.

---

## Chapter 7 — Error Handling

### 20. Always find the root cause — don't suppress errors silently *(p. 103)*

Swallowing an exception hides failures. Callers cannot distinguish "record not found" from "database is down".

---

## Chapter 9 — Unit Tests

### 21. One logical assertion per test *(p. 130)*

Testing multiple concepts in one test hides which concept failed. One concept per test makes failures self-diagnosing.

---

### 22. Fast — tests should run in milliseconds; no real I/O *(F.I.R.S.T., p. 132)*

Slow tests don't get run. No real databases, no real HTTP, no real files.

---

### 23. Independent — tests must not depend on each other *(F.I.R.S.T., p. 132)*

Each test must set up its own state. Shared state causes cascade failures that are hard to diagnose.

---

### 24. Readable — test names describe behavior, not implementation *(F.I.R.S.T., p. 132)*

Test names are documentation. Name them so the failure message tells you what broke.

---

## Chapter 10 — Classes

### 25. Single Responsibility — classes should have one reason to change *(p. 138)*

If a class handles persistence, notifications, and validation, then any one of those three domains can force it to change.

---

## Chapter 11 — Systems

### 26. Use dependency injection — don't instantiate collaborators inside classes *(p. 154)*

Hard-coding `new ConcreteRepo()` inside a class couples it to that implementation and makes tests impossible without the real database.

---

## Chapter 13 — Concurrency

### 27. Separate multi-threading / async code from business logic *(p. 177)*

Business logic must be testable without concurrency infrastructure. Keep locks and queues in a thin wrapper layer.

---

## Additional Principles

*(Not tied to a single chapter; applied throughout the book.)*

### 28. Keep it simple (KISS) — reduce complexity

Never solve the simple problem with a complex solution.

---

### 29. Prefer value objects over primitives

Raw strings and numbers carry no invariants. A value object validates on construction, making illegal states unrepresentable.

---

### 30. Rigidity — don't make changes that cascade everywhere

A change to one requirement should ripple through as few modules as possible. If adding a payment type means editing five files, the design is too rigid.

---

### 31. Opacity — deeply nested code is hard to read; flatten it

Each nesting level forces the reader to hold more context. Guard clauses (early returns) eliminate nesting by handling the exceptional cases first.

---

### 32. Needless complexity — don't over-engineer

Don't introduce abstractions for hypothetical future needs. Three similar lines is better than a premature abstraction.

---

## TypeScript-Specific Clean Code

*(Rules 33–34 are TypeScript-only; examples appear in [`examples/clean-code/ts.md`](examples/clean-code/ts.md) only.)*

### 33. Prefer discriminated unions over boolean flags

Boolean flags multiply ambiguous states. Discriminated unions make every state explicit and exhaustively checkable.

---

### 34. Use `unknown` over `any` — force explicit narrowing

`any` disables the type checker entirely. `unknown` requires the caller to prove the type before using the value.

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
