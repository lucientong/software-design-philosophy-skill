# Functions & Control Flow

Detailed reference for rules `FUNC-01` through `FUNC-05`.

---

## FUNC-01 · Small, Focused Functions

### Definition

Keep functions short and focused — each function performs one logical task. A
function should be readable at a glance and fit on one screen (~20-30 lines as a
guideline, not a hard rule).

### Rationale

Long functions hide bugs, resist testing, mix abstraction levels, and are
difficult to name. When a function does one thing, it is easy to understand, test,
reuse, and replace.

### Good Example

```
def process_order(order):
    validate_order(order)
    total = calculate_total(order.items)
    payment = charge_payment(order.customer, total)
    receipt = generate_receipt(order, payment)
    send_confirmation(order.customer, receipt)
```

### Bad Example

```
def process_order(order):
    # 200 lines: validates fields, calculates prices, applies discounts,
    # checks inventory, charges payment, generates PDF, sends 3 different
    # emails, updates analytics, and logs everything
    ...
```

### Size Heuristics

| Signal | Action |
|---|---|
| Function > 30 lines | Look for extraction opportunities |
| Function > 50 lines | Almost certainly doing too much — split it |
| Function needs more than 3-4 parameters | Extract a parameter object or split the function |
| Function has multiple `# section` comments | Each section is a candidate for extraction |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| "God function" that orchestrates everything | Extract each step into a named function |
| Functions with boolean flags that change behavior | Split into two functions with clear names |
| Copy-pasted logic within a function | Extract shared logic into a helper |

### Related Principles

- `FUNC-02` Single Level of Abstraction
- `ARCH-01` SRP (function-level)
- `NAME-01` Intention-revealing names (small functions have clear names)

---

## FUNC-02 · Single Level of Abstraction Principle (SLAP)

### Definition

Maintain a single level of abstraction within each function. Do not mix
high-level orchestration with low-level implementation details.

### Rationale

When high-level steps are interleaved with low-level operations, readers must
constantly context-switch. Functions that stay at one abstraction level read like
a story.

### Good Example

```
# High-level orchestration — each call is at the same abstraction level
def deploy_application(config):
    build_artifacts(config)
    run_tests(config)
    push_to_registry(config)
    update_infrastructure(config)
    verify_health(config)
```

### Bad Example

```
# Mixed levels: orchestration + low-level file operations + string formatting
def deploy_application(config):
    os.makedirs(f"/tmp/build-{config.version}", exist_ok=True)
    subprocess.run(["npm", "build"], check=True)
    run_tests(config)
    with open("Dockerfile", "r") as f:
        content = f.read().replace("{{VERSION}}", config.version)
    subprocess.run(["docker", "push", f"registry/{config.name}:{config.version}"])
    verify_health(config)
```

### Detection Heuristic

Read the function top-to-bottom. If some lines describe *what* to do while others
describe *how* to do it, the function mixes abstraction levels.

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Business logic mixed with string formatting | Extract formatting into a helper |
| Orchestration function with embedded SQL queries | Move queries to a repository |
| Controller that validates, transforms, and persists | Split into validator, transformer, repository calls |

### Related Principles

- `FUNC-01` Small functions (extraction creates right-sized functions)
- `ARCH-02` Separation of Concerns
- `NAME-01` Named functions document each abstraction level

---

## FUNC-03 · Law of Demeter (Principle of Least Knowledge)

### Definition

A method should only call methods on:
1. Its own object (`self`)
2. Objects passed as parameters
3. Objects it creates
4. Its direct component objects

Avoid chained calls through intermediaries (`a.b().c().d()`).

### Rationale

Long chains create hidden coupling: the caller depends not just on its direct
collaborator but on the entire chain of objects behind it. A change anywhere in
the chain can break the caller.

### Good Example

```
# Ask, don't dig
class Order:
    def get_shipping_cost(self):
        return self.shipping_address.calculate_cost(self.weight)

# Caller
cost = order.get_shipping_cost()
```

### Bad Example

```
# Train wreck — caller knows the entire object graph
cost = order.get_customer().get_address().get_city().get_shipping_zone().get_rate()
```

### Detection Pattern

Count the dots. If a single expression has more than 2 dots (excluding fluent
builders and stream operations), it likely violates the Law of Demeter.

**Exception**: Fluent APIs and builder patterns are designed for chaining and are
not violations.

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| `user.getProfile().getSettings().getTheme()` | Add `user.getTheme()` that delegates internally |
| Accessing deeply nested config values | Provide a focused accessor method |
| Extracting data through 3+ layers of objects | Use a query method on the owner |

### Related Principles

- `ARCH-05` Orthogonality (hidden coupling reduces orthogonality)
- `ABST-01` DIP (depend on direct abstractions)
- `ABST-04` Composition (well-composed objects expose needed behavior)

---

## FUNC-04 · Pure Functions and Side-Effect Isolation

### Definition

Prefer pure functions (same input → same output, no side effects) for
computations. Isolate side effects (I/O, state mutation, network calls) at system
boundaries.

### Rationale

Pure functions are predictable, testable, cacheable, and safely parallelizable.
Side effects are unavoidable but should be concentrated at the edges (entry
points, I/O adapters) to keep the core logic clean.

### Good Example

```
# Pure: calculation logic
def calculate_discount(price, loyalty_tier):
    rates = {"gold": 0.20, "silver": 0.10, "bronze": 0.05}
    return price * rates.get(loyalty_tier, 0)

# Impure: side effects at the boundary
def apply_discount_to_order(order_id):
    order = db.get(order_id)                  # side effect: DB read
    discount = calculate_discount(            # pure
        order.total, order.customer.tier
    )
    order.total -= discount
    db.save(order)                            # side effect: DB write
    notify(order.customer, discount)          # side effect: notification
```

### Bad Example

```
# Mixed: pure logic entangled with side effects
def calculate_discount(order_id):
    order = db.get(order_id)        # side effect embedded in "calculation"
    rate = 0.20 if order.tier == "gold" else 0.10
    discount = order.total * rate
    audit_log.write(discount)       # another side effect in a "calculation"
    return discount
```

### Functional Core / Imperative Shell Pattern

```
┌─────────────────────────────────┐
│  Imperative Shell (boundaries)  │  ← I/O, state, side effects
│  ┌───────────────────────────┐  │
│  │  Functional Core (pure)   │  │  ← business logic, no side effects
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

### Related Principles

- `MODN-01` Immutability (pure functions don't mutate)
- `MODN-02` Minimal side effects
- `TEST-01` Testability (pure functions need no mocks)

---

## FUNC-05 · Low Cyclomatic Complexity

### Definition

Reduce cyclomatic complexity — flatten nested conditionals with early returns,
guard clauses, or polymorphism. Aim for flat, linear control flow.

### Rationale

High nesting depth (3+ levels of `if/else/for/while`) increases cognitive load
exponentially. Each additional branch doubles the number of paths to reason about.

### Good Example

```
# Guard clauses — flat, easy to follow
def process_payment(payment):
    if not payment:
        raise ValueError("Payment required")
    if payment.amount <= 0:
        raise ValueError("Amount must be positive")
    if payment.is_expired():
        raise PaymentExpired()

    return gateway.charge(payment)
```

### Bad Example

```
# Deeply nested — hard to trace the happy path
def process_payment(payment):
    if payment:
        if payment.amount > 0:
            if not payment.is_expired():
                result = gateway.charge(payment)
                if result.success:
                    return result
                else:
                    raise PaymentFailed()
            else:
                raise PaymentExpired()
        else:
            raise ValueError("Amount must be positive")
    else:
        raise ValueError("Payment required")
```

### Complexity Reduction Techniques

| Technique | When to Use |
|---|---|
| Guard clauses (early return) | Validate preconditions upfront |
| Extract method | Complex condition becomes a named boolean function |
| Polymorphism | Replace type-switching `if/elif/else` chains |
| Lookup table / dictionary dispatch | Replace value-based `switch` statements |
| Pipeline / chain | Replace sequential `if` checks |

### Related Principles

- `FUNC-01` Small functions (split complex functions)
- `ERRH-01` Fail Fast (guard clauses are fail-fast)
- `ABST-05` OCP (polymorphism replaces growing conditionals)

---

## Checkpoint

Quick verification steps for Functions & Control Flow:

1. **Size**: Are all functions under 30 lines? Any over 50?
2. **SLAP**: Does each function operate at one abstraction level? Read it as a
   story — any jumps between "what" and "how"?
3. **Train Wrecks**: Any expression with 3+ chained dots (excluding fluent
   APIs)?
4. **Purity**: Are calculation functions free of I/O and state mutation?
5. **Nesting**: Any function with 3+ levels of indentation? Flatten with guard
   clauses.
