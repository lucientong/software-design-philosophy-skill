# Modern Patterns

Detailed reference for rules `MODN-01` through `MODN-03`.

---

## MODN-01 · Immutability

### Definition

Prefer immutable data structures. Instead of modifying existing objects, create new copies with the desired changes.

### Rationale

Immutability eliminates race conditions, unexpected mutations, and stale-reference issues. Immutable objects are thread-safe and easier to reason about.

### Good Example

```
# Python — frozen dataclass
@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str
    def add(self, other):
        return Money(self.amount + other.amount, self.currency)

# JavaScript — spread operator
const updated = { ...user, name: "Alice" };
```

### Bad Example

```
class ShoppingCart:
    items = []  # Shared mutable class variable!
    def add(self, item):
        self.items.append(item)  # Mutates shared state across all instances
```

### Immutability Techniques by Language

| Language | Technique |
|---|---|
| Python | `frozen=True` dataclass, `tuple`, `frozenset` |
| JavaScript/TS | `const`, `Object.freeze()`, spread `...`, `readonly` |
| Java | `final` fields, unmodifiable collections, records |
| Rust | Immutable by default; `mut` is opt-in |

### When Mutability Is Acceptable

- Performance-critical inner loops where allocation cost matters
- Builder pattern during construction (freeze after build)
- Framework requirements (e.g., ORM managed entities)

### Related Principles

- `FUNC-04` Pure Functions · `MODN-02` Minimal Side Effects · `ERRH-01` Fail Fast

---

## MODN-02 · Minimal Side Effects

### Definition

Keep computations pure and push I/O, state mutation, and external interactions to the system edges.

### Rationale

Side-effect-free code is easier to test, cache, parallelize, and compose. The "Functional Core / Imperative Shell" pattern concentrates side effects in a thin outer layer.

### Good Example

```
# Pure core
def calculate_total(items, tax_rate):
    subtotal = sum(i.price * i.quantity for i in items)
    return subtotal + subtotal * tax_rate

# Impure shell
def handle_checkout(request):
    order = db.get_order(request.order_id)
    total = calculate_total(order.items, get_tax_rate(order.region))
    gateway.charge(total)
    db.update_order(order.id, total=total)
```

### Bad Example

```
def calculate_total(order_id):
    order = db.get_order(order_id)        # DB read in "calculate"
    subtotal = sum(i.price for i in order.items)
    audit_log.write(f"Subtotal: {subtotal}")  # Logging in "calculate"
    return subtotal
```

### Side Effect Inventory — Isolate To

| Side Effect | Boundary Layer |
|---|---|
| Database reads/writes | Repository |
| HTTP/API calls | Client adapters |
| File I/O | Storage adapters |
| Clock / randomness | Injected dependencies |

### Related Principles

- `FUNC-04` Pure Functions · `MODN-01` Immutability · `TEST-01` Testability

---

## MODN-03 · Declarative over Imperative

### Definition

Prefer declarative style (expressing *what* to compute) over imperative style (expressing *how*) when the language supports it.

### Rationale

Declarative code is more concise, expresses intent directly, and reduces off-by-one errors. It often enables runtime/compiler optimizations.

### Good Example

```
active_users = [u for u in users if u.is_active]
total = sum(order.amount for order in orders)
```

### Bad Example

```
active_users = []
for i in range(len(users)):
    if users[i].is_active:
        active_users.append(users[i])
```

### Declarative Techniques

| Context | Imperative | Declarative |
|---|---|---|
| Filtering | `for` + `if` + append | comprehension, `filter()` |
| Mapping | manual loop + transform | `map()`, comprehension |
| Aggregation | accumulator loop | `sum()`, `reduce()` |
| UI rendering | DOM manipulation | JSX, templates |
| Infrastructure | scripts | Terraform, K8s YAML |

### When Imperative Is Better

- Complex stateful algorithms (e.g., graph traversal with backtracking)
- Performance-critical code where allocation patterns matter
- When declarative syntax obscures rather than clarifies

### Related Principles

- `SIMP-01` KISS · `FUNC-04` Pure Functions · `NAME-03` Self-documenting code

---

## Checkpoint

Quick verification steps for Modern Patterns:

1. **Immutability**: Are data classes mutable? Can they be frozen? Are internal collections returned by reference?
2. **Side Effects**: Are calculation functions free of I/O? Can you test them without mocking?
3. **Declarative**: Are there manual loops that could be replaced with comprehensions, `map`, or `filter`?
