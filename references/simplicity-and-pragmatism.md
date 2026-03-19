# Simplicity & Pragmatism

Detailed reference for rules `SIMP-01` through `SIMP-04`.

---

## SIMP-01 · KISS (Keep It Simple, Stupid)

### Definition

Choose the simplest solution that fully satisfies the current requirement. Avoid
clever tricks, unnecessary abstractions, and premature optimization.

### Rationale

Complexity is the primary enemy of reliability. Every unnecessary layer, pattern,
or indirection is a future maintenance burden. Simple code is easier to read,
debug, extend, and hand off.

### Good Example

```
# Direct, readable, no unnecessary abstraction
def is_eligible(user):
    return user.age >= 18 and user.is_verified
```

### Bad Example

```
# Over-engineered: strategy pattern for a simple boolean check
class EligibilityStrategy(ABC):
    @abstractmethod
    def check(self, user): ...

class AgeStrategy(EligibilityStrategy):
    def check(self, user): return user.age >= 18

class VerificationStrategy(EligibilityStrategy):
    def check(self, user): return user.is_verified

class EligibilityChecker:
    def __init__(self, strategies):
        self.strategies = strategies
    def is_eligible(self, user):
        return all(s.check(user) for s in self.strategies)

# Usage — far more complex than the requirement demands
checker = EligibilityChecker([AgeStrategy(), VerificationStrategy()])
```

### Decision Heuristic

Ask: "If I remove this abstraction and inline the logic, does the code become
clearer?" If yes, the abstraction is premature.

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Using a framework/library for a task that needs 10 lines of code | Write the 10 lines directly |
| Creating abstract base classes for a single implementation | Use a concrete class; refactor to abstraction when a second implementation appears |
| Configuring a DI container for a small script | Use constructor injection manually |

### Related Principles

- `SIMP-02` YAGNI (don't build what you don't need)
- `ARCH-03` Unix Philosophy (simplicity through focus)
- `FUNC-01` Small functions

---

## SIMP-02 · YAGNI (You Aren't Gonna Need It)

### Definition

Do not build features, abstractions, or infrastructure for speculative future
needs. Build only what is required now.

### Rationale

Speculative code adds cost today (writing, testing, maintaining) for a future
that may never arrive. Studies show that the majority of anticipated features are
never used. Delete what you don't need.

### Good Example

```
# Build only what the current story requires
class UserService:
    def get_user(self, user_id):
        return self.repo.find_by_id(user_id)
    # No get_users_by_zodiac_sign() "just in case"
```

### Bad Example

```
# Speculative: plugin system, event bus, and strategy pattern
# for an app that currently has one workflow
class WorkflowEngine:
    def __init__(self):
        self.plugins = PluginRegistry()
        self.event_bus = EventBus()
        self.strategies = StrategyMap()
    ...
```

### Over-Engineering Anti-Patterns

| Anti-Pattern | Symptom |
|---|---|
| "Future-proof" abstractions | Interfaces with one implementation |
| Premature microservices | Single-team app split into 12 services |
| Configurable everything | YAML files longer than the code they configure |
| Framework before product | Months on infrastructure, zero user features |

### Related Principles

- `SIMP-01` KISS
- `SIMP-03` DRY (but don't abstract prematurely)
- `ABST-05` OCP (extension points are useful — when justified)

---

## SIMP-03 · DRY — Extract Only Proven Duplication

### Definition

Extract shared logic only when duplication is proven (the "rule of three"):
tolerate two occurrences; extract on the third. Do not predict duplication.

### Rationale

Premature abstraction creates the wrong abstraction — one that couples unrelated
callers and resists future change. Duplication is cheaper than the wrong
abstraction.

### Good Example

```
# Two similar functions — tolerate duplication for now
def format_invoice_date(d):
    return d.strftime("%Y-%m-%d")

def format_report_date(d):
    return d.strftime("%Y-%m-%d")

# When a third caller appears, extract:
def format_iso_date(d):
    return d.strftime("%Y-%m-%d")
```

### Bad Example

```
# Premature abstraction: generic "formatter" for two slightly different cases
class DateFormatter:
    def __init__(self, pattern, prefix="", suffix="", locale=None, tz=None):
        ...
# Now both callers depend on this fragile shared class
```

### Decision Heuristic — When to Abstract

1. Is the logic duplicated in **3+ places**?
2. Are the duplicates **truly identical** in behavior and intent?
3. Will they **change for the same reason**?

If all three are YES → extract. Otherwise → tolerate duplication.

### Related Principles

- `SIMP-01` KISS
- `SIMP-02` YAGNI
- `ABST-04` Composition over Inheritance

---

## SIMP-04 · Tracer-Bullet Development

### Definition

Build thin end-to-end slices — a minimal path from input to output that touches
every layer — before filling in details.

### Rationale

A working skeleton validates architectural decisions, exposes integration
problems early, and provides a tangible feedback loop. It is the fastest way to
reduce risk in a new system.

### Good Example

```
# Sprint 1: thin slice — one endpoint, one DB write, one response
POST /orders → validate minimal fields → insert row → return 201

# Sprint 2: enrich — add full validation, pricing, notifications
```

### Bad Example

```
# Spend 3 sprints building the "perfect" validation framework
# before ever writing an endpoint or touching the database
```

### When to Apply

- Starting a new project or major feature
- Integrating with an unfamiliar external system
- Prototyping to validate architecture
- When stakeholders need early visible progress

### Related Principles

- `SIMP-01` KISS
- `SIMP-02` YAGNI
- `TEST-01` Testability (early slices are testable)

---

## Checkpoint

Quick verification steps for Simplicity & Pragmatism:

1. **KISS**: Can a junior developer understand this code in under 5 minutes?
2. **YAGNI**: Is every class, method, and parameter used by the current
   requirements? Remove anything speculative.
3. **DRY**: Are there abstractions with only one caller? Consider inlining them.
4. **Tracer Bullet**: For new features, is there a thin end-to-end path working
   before details are filled in?
5. **Small-Step Refactoring**: Are structural changes separated from behavioral
   changes? Does every refactoring step pass existing tests?
