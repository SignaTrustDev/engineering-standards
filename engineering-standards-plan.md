# Engineering Standards Repository — Build Plan

## Overview

A standalone repository that serves as the single source of truth for software
engineering standards, clean architecture rules, and GoF design patterns. It
doubles as a Claude Code skill library — skills are symlinked globally so every
project benefits without any per-project setup.

**Intended audience:** Software engineers and AI coding assistants working on any
project in the organisation.

**Primary consumers:**

- Claude Code (via skills and CLAUDE.md references)
- Human engineers (as a reference and onboarding resource)

---

## Repository Identity

| Field       | Value                                               |
| ----------- | --------------------------------------------------- |
| Repo name   | `engineering-standards`                             |
| Visibility  | Private (organisation)                              |
| Main branch | `main`                                              |
| Clone path  | `~/engineering-standards` (global, once per machine)|

---

## Full Directory Structure

```text
engineering-standards/
│
├── README.md                          ← What this repo is and how to use it
├── CLAUDE.md                          ← Instructions for Claude when working IN this repo
│
├── CLEAN-CODE.md                      ← Robert C. Martin clean code rules (40 rules, with TS/Python examples)
├── CLEAN-ARCHITECTURE.md              ← Uncle Bob clean architecture + Hexagonal/Ports & Adapters
│
├── skills/
│   ├── README.md                      ← How to install and use the skills
│   ├── clean-review.md                ← /clean-review
│   ├── apply-pattern.md               ← /apply-pattern <pattern-name>
│   ├── architecture-check.md          ← /architecture-check
│   └── refactor.md                    ← /refactor
│
└── patterns/
    ├── README.md                      ← Index + when-to-use decision tree
    ├── creational/
    │   ├── factory-method.md
    │   ├── abstract-factory.md
    │   ├── builder.md
    │   ├── prototype.md
    │   └── singleton.md
    ├── structural/
    │   ├── adapter.md
    │   ├── bridge.md
    │   ├── composite.md
    │   ├── decorator.md
    │   ├── facade.md
    │   ├── flyweight.md
    │   └── proxy.md
    └── behavioral/
        ├── chain-of-responsibility.md
        ├── command.md
        ├── iterator.md
        ├── mediator.md
        ├── memento.md
        ├── observer.md
        ├── state.md
        ├── strategy.md
        ├── template-method.md
        └── visitor.md
```

---

## File Specifications

### `README.md`

- What the repo is
- Prerequisites (Claude Code CLI installed)
- One-time global setup instructions (clone + symlink skills)
- How to reference from a project's CLAUDE.md
- How to contribute (add a pattern, update a rule)
- Links to external sources (Refactoring Guru, Clean Code book, Clean Architecture book)

---

### `CLAUDE.md`

Instructions for Claude when working **inside this repo**:

- All patterns must include a ❌ BAD and ✅ GOOD TypeScript example
- Examples must be self-contained (runnable without imports from a framework)
- Pattern files must follow the standard template (see below)
- Skill files must reference pattern files using `~/engineering-standards/` absolute paths
- No framework-specific code in pattern files (patterns are framework-agnostic)
- CLEAN-CODE.md and CLEAN-ARCHITECTURE.md apply to any code written in examples

---

### `CLEAN-CODE.md`

Already written. Contains 40 rules from Robert C. Martin's *Clean Code*, each with:

- Rule name and number
- ❌ BAD example (Python and/or TypeScript)
- ✅ GOOD example (Python and/or TypeScript)
- Summary checklist at the end

**Source:** Migrate from `Signatrust_v4/CLEAN-CODE.md`.

---

### `CLEAN-ARCHITECTURE.md`

To be written. Must cover:

#### Section 1 — The Core Idea

- Separation of concerns across concentric layers
- The **Dependency Rule**: source code dependencies must point inward only
- Inner layers know nothing about outer layers

#### Section 2 — The Four Layers

```text
┌─────────────────────────────────────────┐
│           Frameworks & Drivers          │  ← Next.js, Prisma, Express, React
├─────────────────────────────────────────┤
│         Interface Adapters              │  ← API routes, controllers, presenters
├─────────────────────────────────────────┤
│           Application / Use Cases       │  ← Business workflows, orchestration
├─────────────────────────────────────────┤
│           Entities / Domain             │  ← Core business rules, domain models
└─────────────────────────────────────────┘
```

For each layer:

- What belongs here
- What does NOT belong here
- TypeScript example showing the layer in isolation

#### Section 3 — Ports & Adapters (Hexagonal Architecture)

- Ports: interfaces defined by the domain
- Adapters: implementations in the outer layers
- How this maps to the `lib/` (business logic) vs `services/` (infrastructure) split
- Example: `StoragePort` interface → `S3Adapter`, `DropboxAdapter`, `ConsoleAdapter`

#### Section 4 — Practical Rules

- Never import a framework directly into a use case
- Never import Prisma into domain entities
- Interfaces belong to the layer that uses them, not the layer that implements them
- Use cases return domain objects, not framework responses (e.g. not `NextResponse`)

#### Section 5 — Dependency Injection in TypeScript

- Constructor injection pattern
- Factory functions as an alternative to DI containers
- How to wire adapters at the application boundary

#### Section 6 — What to Test at Each Layer

| Layer              | Test type   | Mock?                          |
| ------------------ | ----------- | ------------------------------ |
| Entities           | Unit        | Nothing                        |
| Use Cases          | Unit        | Mock ports (interfaces)        |
| Interface Adapters | Integration | Real use cases, mock framework |
| Frameworks         | E2E         | Nothing mocked                 |

---

### Pattern File Template

Every file in `patterns/` must follow this structure:

```markdown
# Pattern Name

**Category:** Creational / Structural / Behavioral
**Refactoring Guru:** https://refactoring.guru/design-patterns/<name>

## Intent

One sentence: what problem does this pattern solve?

## When to Use

- Bullet list of specific situations that call for this pattern
- Be concrete — name the type of code smell or problem it solves

## When NOT to Use

- Bullet list of situations where this pattern is overkill or wrong

## Structure

Diagram or description of participants and their relationships.

## TypeScript Example

### ❌ Without the Pattern

```typescript
// Code that has the problem this pattern solves
```

### ✅ With the Pattern

```typescript
// Clean implementation of the pattern
```

## Real-World Analogy

One paragraph connecting the pattern to something non-technical.

## Related Patterns

- **Pattern A** — how it differs or combines
- **Pattern B** — when to choose this over that
```

---

### `patterns/README.md` — Decision Tree

The most important file in `patterns/`. Helps Claude (and engineers) choose the
**right** pattern rather than just *a* pattern.

Structure:

#### By Problem Type

| Problem | Pattern(s) to consider |
|---|---|
| Creating objects without specifying exact class | Factory Method, Abstract Factory |
| Building complex objects step by step | Builder |
| Sharing a single instance across the system | Singleton |
| Making incompatible interfaces work together | Adapter |
| Simplifying a complex subsystem | Facade |
| Adding behaviour to objects without subclassing | Decorator |
| Defining a family of algorithms, making them interchangeable | Strategy |
| Notifying multiple objects about state changes | Observer |
| Passing requests along a chain of handlers | Chain of Responsibility |
| Encapsulating a request as an object | Command |
| Defining a skeleton algorithm, letting subclasses fill in steps | Template Method |
| Allowing an object to alter its behaviour when state changes | State |

#### By Code Smell

| Code smell | Pattern that fixes it |
|---|---|
| `switch`/`if-else` on type discriminators | Strategy or Factory Method |
| Objects that know too much about other objects | Facade or Mediator |
| Adding features via subclass explosion | Decorator |
| God class doing everything | Split with Command + Facade |
| Tight coupling to a third-party SDK | Adapter |
| Hard-coded `new SomeConcreteClass()` everywhere | Factory Method |
| Repeated algorithm with varying steps | Template Method |

---

### Skill File Specifications

Each skill file lives in `skills/` and is symlinked to `~/.claude/commands/`.

---

#### `skills/clean-review.md` → `/clean-review`

**Purpose:** Review the current file (or selection) against CLEAN-CODE.md rules.

**What it does:**

1. Reads `~/engineering-standards/CLEAN-CODE.md`
2. Reads the target file
3. Checks each of the 40 rules
4. Reports violations with: rule number, line number, what's wrong, suggested fix
5. Prioritises by severity: naming and function size first, comments last

**Invocation:**

```
/clean-review
/clean-review src/lib/signerOtp.ts
```

---

#### `skills/apply-pattern.md` → `/apply-pattern`

**Purpose:** Apply a named GoF pattern to selected code.

**What it does:**

1. Reads the pattern file from `~/engineering-standards/patterns/<category>/<name>.md`
2. Reads the current file or selection
3. Explains why the pattern fits (or doesn't)
4. If it fits: produces the refactored code following the pattern's ✅ GOOD template
5. Explains each participant in the pattern and where it maps to the existing code

**Invocation:**

```
/apply-pattern strategy
/apply-pattern factory-method
/apply-pattern observer
```

---

#### `skills/architecture-check.md` → `/architecture-check`

**Purpose:** Verify the current file respects clean architecture layer boundaries.

**What it does:**

1. Reads `~/engineering-standards/CLEAN-ARCHITECTURE.md`
2. Determines which layer the file belongs to (based on path)
3. Greps all imports
4. Flags any import that violates the Dependency Rule (inner layer importing outer)
5. Suggests how to introduce a port/interface to fix each violation

**Invocation:**

```
/architecture-check
/architecture-check src/services/storage/s3Provider.ts
```

---

#### `skills/refactor.md` → `/refactor`

**Purpose:** General-purpose refactor that applies CLEAN-CODE.md + stepdown rule +
guard clause extraction in one pass.

**What it does:**

1. Reads CLEAN-CODE.md rules 19–27 (function rules + structure)
2. Reads the target file
3. Applies in order:
   - Extract inline guard/validation blocks into named functions
   - Enforce one level of abstraction per function
   - Apply stepdown rule (high-level above low-level)
   - Extract magic numbers to named constants
   - Remove flag arguments (split into two functions)
4. Produces the refactored file with a diff summary

**Invocation:**

```
/refactor
/refactor src/app/api/internal/envelopes/route.ts
```

---

## GoF Pattern Content Guide

All 23 patterns, grouped, with priority for TypeScript/web relevance:

### Creational (5)

| Pattern | Web relevance | Priority |
|---|---|---|
| Factory Method | High — creating service instances | P1 |
| Abstract Factory | High — provider families (storage, notifications) | P1 |
| Builder | High — complex object construction (query builders, request objects) | P1 |
| Singleton | Medium — database clients, loggers | P2 |
| Prototype | Low — rarely needed in TypeScript | P3 |

### Structural (7)

| Pattern | Web relevance | Priority |
|---|---|---|
| Adapter | High — wrapping third-party SDKs | P1 |
| Facade | High — simplifying complex subsystems | P1 |
| Decorator | High — adding behaviour (logging, caching, auth) | P1 |
| Composite | Medium — tree structures, middleware chains | P2 |
| Proxy | Medium — lazy loading, access control | P2 |
| Bridge | Low — separating abstraction from implementation | P3 |
| Flyweight | Low — memory optimisation, rarely applied | P3 |

### Behavioral (11)

| Pattern | Web relevance | Priority |
|---|---|---|
| Strategy | Very high — swappable algorithms and providers | P1 |
| Observer | Very high — event systems, webhooks, pub/sub | P1 |
| Command | High — encapsulating requests, undo/redo, queues | P1 |
| Chain of Responsibility | High — middleware, validation pipelines | P1 |
| Template Method | High — shared algorithm skeletons | P1 |
| State | Medium — workflow state machines | P2 |
| Iterator | Medium — custom collection traversal | P2 |
| Mediator | Medium — decoupling components | P2 |
| Memento | Low — snapshots, rarely needed | P3 |
| Visitor | Low — operations on object trees | P3 |
| Interpreter | Very low — DSLs, skip unless specifically needed | P3 |

**Build P1 patterns first.** P3 patterns can be stubs that link to Refactoring Guru.

---

## One-Time Machine Setup

```bash
# 1. Clone to a stable global location
git clone git@github.com:<org>/engineering-standards.git ~/engineering-standards

# 2. Create Claude's global commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# 3. Symlink all skills
ln -s ~/engineering-standards/skills/clean-review.md ~/.claude/commands/clean-review.md
ln -s ~/engineering-standards/skills/apply-pattern.md ~/.claude/commands/apply-pattern.md
ln -s ~/engineering-standards/skills/architecture-check.md ~/.claude/commands/architecture-check.md
ln -s ~/engineering-standards/skills/refactor.md ~/.claude/commands/refactor.md

# 4. Verify
ls -la ~/.claude/commands/
```

---

## Per-Project Integration

Add one block to the project's `CLAUDE.md`:

```markdown
## Engineering Standards

All code must comply with the organisation engineering standards:

- **Clean Code rules:** `~/engineering-standards/CLEAN-CODE.md`
- **Architecture rules:** `~/engineering-standards/CLEAN-ARCHITECTURE.md`
- **Design patterns:** `~/engineering-standards/patterns/README.md`

Use the available skills to enforce these:
- `/clean-review` — check a file against clean code rules
- `/apply-pattern <name>` — apply a GoF pattern to selected code
- `/architecture-check` — verify layer boundary compliance
- `/refactor` — general-purpose clean code refactor
```

---

## Build Order

### Phase 1 — Foundation (Day 1)

- [ ] Create repo, push to GitHub
- [ ] Write `README.md`
- [ ] Write `CLAUDE.md` (instructions for working in this repo)
- [ ] Migrate `CLEAN-CODE.md` from Signatrust
- [x] Write `CLEAN-ARCHITECTURE.md` ✅ — extracted from Clean Architecture PDF; includes SOLID examples (Python + TypeScript)

### Phase 2 — P1 Patterns (Day 2–3)

- [ ] `patterns/README.md` (decision tree — do this first)
- [x] `patterns/creational/factory-method.md` ✅
- [x] `patterns/creational/abstract-factory.md` ✅
- [x] `patterns/creational/builder.md` ✅
- [x] `patterns/structural/adapter.md` ✅
- [x] `patterns/structural/facade.md` ✅
- [x] `patterns/structural/decorator.md` ✅
- [x] `patterns/behavioral/strategy.md` ✅
- [x] `patterns/behavioral/observer.md` ✅
- [x] `patterns/behavioral/command.md` ✅
- [x] `patterns/behavioral/chain-of-responsibility.md` ✅
- [x] `patterns/behavioral/template-method.md` ✅

> **Source:** Extracted from GoF Design Patterns PDF (`articulo.pdf`), cross-checked against Refactoring Guru.
> Each file follows the standard template with TypeScript + Python bad/good examples.

### Phase 3 — Skills (Day 4)

- [ ] `skills/README.md`
- [ ] `skills/clean-review.md`
- [ ] `skills/apply-pattern.md`
- [ ] `skills/architecture-check.md`
- [ ] `skills/refactor.md`

### Phase 4 — P2 Patterns (Day 5)

- [x] `patterns/creational/singleton.md` ✅
- [x] `patterns/structural/composite.md` ✅
- [x] `patterns/structural/proxy.md` ✅
- [x] `patterns/behavioral/state.md` ✅
- [x] `patterns/behavioral/iterator.md` ✅
- [x] `patterns/behavioral/mediator.md` ✅

### Phase 5 — P3 Patterns (backlog)

Stubs only — each file contains the intent, a link to Refactoring Guru, and a
note that a full TypeScript example is pending.

- [x] `patterns/creational/prototype.md` ✅ stub
- [x] `patterns/structural/bridge.md` ✅ stub
- [x] `patterns/structural/flyweight.md` ✅ stub
- [x] `patterns/behavioral/memento.md` ✅ stub
- [x] `patterns/behavioral/visitor.md` ✅ stub
- [ ] `patterns/behavioral/interpreter.md` (not created — very low priority)

---

## Success Criteria

- [ ] Cloning the repo and running the symlink commands takes under 2 minutes
- [ ] `/clean-review` correctly identifies at least 5 distinct rule violations in a
  real route file
- [ ] `/apply-pattern strategy` correctly refactors a `switch`-on-type block into
  a strategy pattern without being told which file to look at
- [ ] `/architecture-check` correctly flags a use case file that imports `NextResponse`
- [ ] All P1 pattern files follow the standard template exactly
- [ ] A new engineer can read `patterns/README.md` and choose the correct pattern
  for a given problem without reading any individual pattern file first

---

## Reference Sources

| Resource | URL |
|---|---|
| Refactoring Guru — Design Patterns | https://refactoring.guru/design-patterns |
| Refactoring Guru — TypeScript examples | https://refactoring.guru/design-patterns/typescript |
| Clean Code (Martin) | ISBN 978-0132350884 |
| Clean Architecture (Martin) | ISBN 978-0134494166 |
| Hexagonal Architecture (Cockburn) | https://alistair.cockburn.us/hexagonal-architecture |
