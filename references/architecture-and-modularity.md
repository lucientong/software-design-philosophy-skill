# Architecture & Modularity

Detailed reference for rules `ARCH-01` through `ARCH-05`.

---

## ARCH-01 · Single Responsibility Principle (SRP)

### Definition

Every module, class, or service should have exactly one reason to change — one
axis of responsibility aligned with one stakeholder or business concern.

### Rationale

When a unit owns multiple responsibilities, a change in one concern can break
another. The module becomes a coupling magnet: every team that touches any of its
responsibilities becomes a coordination bottleneck.

### Good Example

```
# Each class owns one concern
class InvoiceCalculator:
    def calculate_total(self, line_items) -> Decimal:
        return sum(item.price * item.quantity for item in line_items)

class InvoicePrinter:
    def to_pdf(self, invoice) -> bytes:
        ...

class InvoiceRepository:
    def save(self, invoice) -> None:
        ...
```

### Bad Example

```
# God class — calculation, rendering, AND persistence in one place
class Invoice:
    def calculate_total(self): ...
    def to_pdf(self): ...
    def save_to_database(self): ...
    def send_email(self): ...
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| "God class" with 500+ lines touching UI, DB, and logic | Split into focused classes per concern |
| Service method that validates, transforms, persists, and notifies | Extract each step into a dedicated collaborator |
| Utility class that grows unbounded ("StringUtils" with 80 methods) | Group by domain concept; break into focused helpers |

### Related Principles

- `ARCH-02` Separation of Concerns (broader scope)
- `ARCH-03` Unix Philosophy (composability)
- `FUNC-01` Single-task functions (function-level SRP)

---

## ARCH-02 · Separation of Concerns (SoC)

### Definition

Divide a system into distinct sections, each addressing a separate concern (e.g.,
presentation, business logic, data access, infrastructure).

### Rationale

Mixed concerns make it impossible to change one aspect without touching others.
Separation enables parallel development, independent testing, and technology
substitution within a layer.

### Good Example

```
# Layered architecture
src/
  presentation/    # HTTP handlers, CLI, view models
  domain/          # Business rules, entities, value objects
  infrastructure/  # Database adapters, external API clients
```

### Bad Example

```
# Handler does everything
def create_order(request):
    # Validates input (presentation concern)
    if not request.body.get("items"):
        return Response(400)
    # Applies business rule (domain concern)
    discount = calculate_loyalty_discount(request.user)
    # Writes to DB (infrastructure concern)
    db.execute("INSERT INTO orders ...")
    # Sends notification (infrastructure concern)
    email_client.send(...)
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| SQL queries embedded in HTTP handlers | Introduce a repository layer |
| Business rules in database triggers or stored procedures | Move logic to domain layer; keep DB as storage |
| Presentation formatting in domain entities | Use view models or presenters |

### Related Principles

- `ARCH-01` SRP (per-unit focus)
- `ABST-01` DIP (depend on abstractions between layers)
- `ARCH-04` Bounded Contexts (domain-level separation)

---

## ARCH-03 · Unix Philosophy — Do One Thing Well

### Definition

Design each component to do one thing well and compose with others through clear,
simple interfaces.

### Rationale

Small, single-purpose components are easier to understand, test, replace, and
recombine. Composability multiplies the value of each component.

### Good Example

```
# Composable pipeline of small, focused functions
raw_data = read_csv(path)
cleaned = remove_nulls(raw_data)
enriched = add_computed_columns(cleaned)
write_parquet(enriched, output_path)
```

### Bad Example

```
# Monolithic function that does everything
def process_data(path, output_path):
    # reads, cleans, transforms, validates, writes — 400 lines
    ...
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Functions longer than 50 lines doing multiple tasks | Break into composable steps |
| Libraries with entangled features that can't be used independently | Decompose into focused packages |

### Related Principles

- `ARCH-01` SRP
- `FUNC-01` Single-task functions
- `SIMP-01` KISS

---

## ARCH-04 · Domain-Aligned Boundaries (Bounded Contexts)

### Definition

Align module and service boundaries with business domain boundaries. Use the
domain's own vocabulary (ubiquitous language) within each boundary.

### Rationale

When module boundaries cross domain boundaries, concepts leak. A "User" in the
authentication context is different from a "User" in the billing context —
conflating them creates a bloated model that serves nobody well.

### Good Example

```
# Each bounded context owns its own model
auth/
  models.py    # AuthUser: credentials, roles, sessions
billing/
  models.py    # BillingCustomer: payment methods, invoices
shipping/
  models.py    # ShippingRecipient: address, delivery preferences
```

### Bad Example

```
# One "User" model shared everywhere — 40 fields, 15 responsibilities
class User:
    username, password, roles,             # Auth
    credit_card, billing_address,          # Billing
    shipping_address, delivery_prefs,      # Shipping
    preferences, avatar, bio,              # Profile
    ...
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Shared database tables spanning multiple domains | Split into per-context tables with explicit integration |
| A single "models" package used by every service | Give each service its own models; map at boundaries |

### Related Principles

- `NAME-02` Ubiquitous Language
- `ARCH-02` Separation of Concerns
- `ABST-01` DIP (integrate through abstractions)

---

## ARCH-05 · Orthogonality

### Definition

Keep modules orthogonal — changing one should not require changing another. Each
module's behavior is independent of the others.

### Rationale

Orthogonal design reduces the blast radius of changes. When modules are
independent, developers can reason about, test, and deploy each one without
understanding the entire system.

### Good Example

```
# Logging, authentication, and business logic are independent
class OrderService:
    def __init__(self, repo, logger, auth):
        self.repo = repo
        self.logger = logger
        self.auth = auth

# Changing the logger implementation does not affect OrderService logic
```

### Bad Example

```
# Business logic depends on specific logging library internals
class OrderService:
    def create_order(self, data):
        Log4j.getLogger("orders").setLevel(Level.DEBUG)  # tight coupling
        ...
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Global state shared between modules | Use dependency injection; avoid mutable globals |
| Configuration scattered across source files | Centralize config; inject into modules |
| Feature flags embedded in business logic | Extract flag evaluation behind an interface |

### Related Principles

- `ARCH-01` SRP
- `ABST-01` DIP
- `ABST-04` Composition over Inheritance

---

## Checkpoint

Quick verification steps for Architecture & Modularity:

1. **Single Responsibility**: Can you describe each class in one sentence without
   using "and"?
2. **Layer Separation**: Are presentation, domain, and infrastructure in separate
   directories/packages?
3. **Composability**: Can any component be replaced or reused without modifying
   its collaborators?
4. **Domain Alignment**: Do module names match business domain terms?
5. **Orthogonality**: Does changing a logging library require touching business
   logic?
6. **Deep Modules**: Do your modules expose simple interfaces while hiding
   complex internals? Or do callers have to manage many parameters and call
   sequences?
