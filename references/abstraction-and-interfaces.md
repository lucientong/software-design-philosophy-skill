# Abstraction & Interfaces

Detailed reference for rules `ABST-01` through `ABST-05`.

---

## ABST-01 · Dependency Inversion Principle (DIP)

### Definition

High-level modules should not depend on low-level modules. Both should depend on
abstractions (interfaces or protocols). Details depend on abstractions, not the
reverse.

### Rationale

When business logic depends on a concrete database client or HTTP library, it
cannot be tested in isolation or swapped without rewriting. Abstractions create
stable seams.

### Good Example

```
# High-level module depends on an interface
class OrderService:
    def __init__(self, repo: OrderRepository):  # abstraction
        self.repo = repo

    def place_order(self, order):
        self.repo.save(order)

# Low-level module implements the interface
class PostgresOrderRepository(OrderRepository):
    def save(self, order): ...
```

### Bad Example

```
# High-level module depends on concrete low-level detail
class OrderService:
    def __init__(self):
        self.db = psycopg2.connect("host=localhost dbname=orders")

    def place_order(self, order):
        self.db.execute("INSERT INTO orders ...")
```

### Dependency Direction Guidance

```
┌─────────────────┐
│  Domain Layer    │  ← depends on nothing external
│  (interfaces)    │
├─────────────────┤
│  Application     │  ← depends on Domain interfaces
│  (use cases)     │
├─────────────────┤
│  Infrastructure  │  ← implements Domain interfaces
│  (DB, API, I/O)  │
└─────────────────┘
```

Arrows of dependency always point **inward** — from infrastructure toward domain.

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Importing a database driver in domain logic | Define a repository interface in domain; implement in infrastructure |
| Hard-coded HTTP client in a service | Inject an abstract client; mock in tests |
| Direct file system access in business rules | Use a storage abstraction |

### Related Principles

- `ARCH-02` Separation of Concerns
- `TEST-01` Testability (DIP enables test doubles)
- `ABST-03` ISP (keep injected interfaces minimal)

---

## ABST-02 · Liskov Substitution Principle (LSP)

### Definition

Subtypes must be substitutable for their base types without altering the
correctness of the program. A subclass should honor the base class's contract.

### Rationale

If a subtype changes behavior in unexpected ways (throws new exceptions, ignores
preconditions, changes return semantics), callers using the base type will break
silently.

### Good Example

```
class Bird:
    def move(self): ...

class Sparrow(Bird):
    def move(self):
        self.fly()  # move() contract: relocate the bird — honored

class Penguin(Bird):
    def move(self):
        self.walk()  # move() contract still honored, different mechanism
```

### Bad Example

```
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self):
        raise NotImplementedError("Penguins can't fly")
        # Violates LSP — callers of Bird.fly() crash
```

### Violation Signals

- Subclass raises exceptions not declared in the base class
- Subclass ignores or weakens preconditions
- Subclass changes the semantic meaning of a method
- Subclass requires callers to do `isinstance` checks

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| `NotImplementedError` in a subclass method | Redesign hierarchy — the method doesn't belong on the base |
| Subclass silently ignores a method call | Either implement the contract or use a different abstraction |
| Square subclassing Rectangle with different `set_width` semantics | Use composition or separate interfaces |

### Related Principles

- `ABST-01` DIP (abstractions must be trustworthy)
- `ABST-03` ISP (split interfaces so subtypes don't need to fake methods)
- `ABST-04` Composition over Inheritance (avoid fragile hierarchies)

---

## ABST-03 · Interface Segregation Principle (ISP)

### Definition

Design interfaces to be client-specific and minimal. No client should be forced
to depend on methods it does not use.

### Rationale

Fat interfaces create unnecessary coupling: implementors must provide methods
they don't care about, and clients depend on a surface area they don't use.
Smaller interfaces improve focus and reduce the impact of changes.

### Good Example

```
# Segregated interfaces — each client gets only what it needs
class Readable:
    def read(self) -> bytes: ...

class Writable:
    def write(self, data: bytes): ...

class ReadWriteStream(Readable, Writable): ...
```

### Bad Example

```
# Fat interface — forces all implementors to handle everything
class Stream:
    def read(self) -> bytes: ...
    def write(self, data: bytes): ...
    def seek(self, pos: int): ...
    def truncate(self): ...
    def flush(self): ...
    # Read-only implementations must stub write/seek/truncate/flush
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Interface with 10+ methods where most implementors stub half | Split into role-based interfaces |
| A "Repository" interface with `find`, `save`, `delete`, `bulk_import`, `export` | Separate read and write interfaces |

### Related Principles

- `ABST-01` DIP (smaller interfaces are easier to abstract)
- `ARCH-01` SRP (interface-level single responsibility)
- `ABST-02` LSP (smaller interfaces reduce LSP violation surface)

---

## ABST-04 · Composition over Inheritance

### Definition

Extend behavior through composition (has-a) and delegation rather than deep
inheritance hierarchies (is-a).

### Rationale

Inheritance creates fragile, tightly coupled hierarchies. A change in a base
class can break all subclasses. Composition provides flexibility, explicit
dependencies, and the ability to swap behaviors at runtime.

### Good Example

```
# Compose behaviors
class Logger:
    def log(self, msg): ...

class NotificationService:
    def __init__(self, sender, logger):
        self.sender = sender   # composition: can swap email/sms/push
        self.logger = logger

    def notify(self, user, message):
        self.sender.send(user, message)
        self.logger.log(f"Notified {user}")
```

### Bad Example

```
# Deep inheritance — fragile and hard to understand
class BaseNotifier:
    def log(self, msg): ...

class EmailNotifier(BaseNotifier):
    def send(self, user, msg): ...

class LoggingEmailNotifier(EmailNotifier):
    def send(self, user, msg):
        super().send(user, msg)
        self.log(f"Sent to {user}")

class RetryLoggingEmailNotifier(LoggingEmailNotifier):
    # 4 levels deep — a change in BaseNotifier ripples everywhere
    ...
```

### When to Use Inheritance vs Composition

| Use Inheritance When | Use Composition When |
|---|---|
| True "is-a" relationship (e.g., `Dog` is an `Animal`) | Behavior varies independently (e.g., notification channel vs. logging) |
| Shared implementation is stable and unlikely to change | You need to mix and match behaviors at runtime |
| Framework requires it (e.g., Django views, Java servlets) | The inheritance depth exceeds 2 levels |

### Related Principles

- `ABST-01` DIP (injected dependencies are composed)
- `ARCH-05` Orthogonality (composed behaviors are independent)
- `ABST-02` LSP (composition avoids LSP pitfalls)

---

## ABST-05 · Open-Closed Principle (OCP)

### Definition

Software entities should be open for extension but closed for modification.
Achieve new behavior by adding new code, not by changing existing tested code.

### Rationale

Modifying existing, tested modules risks introducing regressions. Extension
points (strategy pattern, plugins, hooks) allow growth without touching stable
code.

### Good Example

```
# Open for extension via new strategy implementations
class PricingStrategy:
    def calculate(self, order) -> Decimal: ...

class RegularPricing(PricingStrategy): ...
class PremiumPricing(PricingStrategy): ...
class HolidayPricing(PricingStrategy): ...  # new — no existing code changed

class OrderService:
    def __init__(self, pricing: PricingStrategy):
        self.pricing = pricing

    def total(self, order):
        return self.pricing.calculate(order)
```

### Bad Example

```
# Closed for extension — must modify existing code for every new case
class OrderService:
    def total(self, order):
        if order.type == "regular":
            return self._regular_price(order)
        elif order.type == "premium":
            return self._premium_price(order)
        # Adding "holiday" requires modifying this method
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Growing `if/elif/else` chains for new types | Use polymorphism or a registry pattern |
| Hard-coded format handlers (JSON, XML, CSV) | Define a handler interface; register new formats |

### Related Principles

- `ABST-01` DIP (extension through abstraction)
- `ABST-04` Composition (compose new behaviors)
- `SIMP-02` YAGNI (add extension points only when a second case appears)

---

## Checkpoint

Quick verification steps for Abstraction & Interfaces:

1. **DIP**: Do high-level modules import low-level concrete classes directly? If
   yes, introduce an abstraction.
2. **LSP**: Does any subclass raise `NotImplementedError` or change base method
   semantics? Redesign the hierarchy.
3. **ISP**: Does any interface have methods that some implementors stub or
   ignore? Split it.
4. **Composition**: Is the inheritance depth > 2? Consider flattening with
   composition.
5. **OCP**: Are there growing `if/elif` chains for type dispatch? Use
   polymorphism.
