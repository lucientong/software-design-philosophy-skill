# Testing & Quality

Detailed reference for rules `TEST-01` through `TEST-03`.

---

## TEST-01 · Design for Testability

### Definition

Design code so it can be tested in isolation. Inject dependencies through
constructors or parameters; expose seams for test doubles (mocks, stubs, fakes).

### Rationale

Code that creates its own dependencies (hard-coded `new`, global singletons,
static calls to external services) cannot be tested without the real
infrastructure. Testable design is inherently better design — it enforces loose
coupling and clear interfaces.

### Good Example

```
# Dependencies are injected — easy to test with fakes
class OrderService:
    def __init__(self, repo: OrderRepository, payment: PaymentGateway):
        self.repo = repo
        self.payment = payment

    def place_order(self, order):
        self.payment.charge(order.total)
        self.repo.save(order)

# Test
def test_place_order():
    fake_repo = FakeOrderRepository()
    fake_payment = FakePaymentGateway()
    service = OrderService(fake_repo, fake_payment)
    service.place_order(sample_order)
    assert fake_repo.last_saved == sample_order
```

### Bad Example

```
# Hard-coded dependencies — requires real DB and payment gateway to test
class OrderService:
    def place_order(self, order):
        db = PostgresConnection.get_instance()  # singleton — can't swap
        stripe = Stripe(api_key="sk_live_...")   # real payment — can't fake
        stripe.charge(order.total)
        db.execute("INSERT INTO orders ...")
```

### Testability Checklist

| Question | If No → Fix |
|---|---|
| Can you instantiate the class without external services? | Inject dependencies |
| Can you replace each dependency with a fake? | Depend on interfaces, not concretions |
| Can you test the function without file system or network? | Isolate I/O at boundaries |
| Can you test one behavior without triggering others? | Split responsibilities |

### Seams for Testing

A "seam" is a place where you can alter behavior without editing production code:
- **Constructor injection**: pass dependencies as constructor parameters
- **Method injection**: pass dependencies as method parameters
- **Interface substitution**: swap implementations behind an interface
- **Environment configuration**: use config to switch between real and fake services

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Singletons accessed via static methods | Inject the instance; make singleton creation external |
| `datetime.now()` called directly in business logic | Inject a clock abstraction |
| File paths hard-coded in functions | Pass paths as parameters or inject a file system abstraction |
| Direct HTTP calls in domain logic | Inject an HTTP client interface |

### Related Principles

- `ABST-01` DIP (injected dependencies are abstractions)
- `FUNC-04` Pure Functions (pure functions need no test infrastructure)
- `ARCH-01` SRP (testable classes have focused responsibilities)

---

## TEST-02 · Test Pyramid

### Definition

Structure tests as a pyramid: a large base of fast, focused unit tests; a smaller
layer of integration tests; and a minimal cap of end-to-end (E2E) tests.

### Rationale

Unit tests are fast, precise, and cheap. Integration tests verify component
interactions. E2E tests catch system-level issues but are slow, flaky, and
expensive. Inverting the pyramid (many E2E, few unit tests) creates slow feedback
loops and fragile CI pipelines.

### Pyramid Proportions (Guideline)

```
        ╱ E2E ╲         ~5-10%   Slow, expensive, covers critical paths
       ╱ Integ. ╲       ~15-25%  Medium speed, verifies integrations
      ╱  Unit     ╲      ~70-80%  Fast, focused, covers logic
     ─────────────────
```

### Good Example

```
# Unit test — fast, isolated, focused on logic
def test_calculate_discount_gold():
    assert calculate_discount(100, "gold") == 20

# Integration test — verifies repository + database interaction
def test_save_and_retrieve_order(db_session):
    repo = PostgresOrderRepo(db_session)
    repo.save(sample_order)
    retrieved = repo.find(sample_order.id)
    assert retrieved == sample_order

# E2E test — one critical path through the full system
def test_complete_checkout(browser):
    browser.add_to_cart("SKU-001")
    browser.checkout()
    assert browser.sees("Order confirmed")
```

### What to Test at Each Level

| Level | Test What | Don't Test |
|---|---|---|
| Unit | Business logic, calculations, validations | Database, network, file I/O |
| Integration | Repository implementations, API clients, message handlers | Business rules (those belong in unit tests) |
| E2E | Critical user journeys, smoke tests | Every edge case (too slow and flaky) |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| 500 E2E tests, 20 unit tests (inverted pyramid) | Move logic assertions to unit tests; keep only critical E2E paths |
| Integration tests that test pure logic | Extract logic into testable functions; unit test them |
| No integration tests — only unit and E2E | Add integration tests for data access and external service interactions |

### Related Principles

- `TEST-01` Testability (injectable dependencies enable unit testing)
- `TEST-03` Behavior-based tests (unit tests verify behavior)
- `FUNC-04` Pure Functions (pure functions are naturally unit-testable)

---

## TEST-03 · Behavior-Based Tests

### Definition

Write tests that verify behavior and intent (what the code does for the user),
not implementation details (how the code does it internally).

### Rationale

Tests coupled to implementation details break every time the code is refactored
— even when behavior is unchanged. Behavior-based tests remain stable through
refactoring and catch real regressions.

### Good Example

```
# Tests behavior: "given a gold customer, discount is 20%"
def test_gold_customer_gets_20_percent_discount():
    order = Order(customer=gold_customer, items=[Item(price=100)])
    order.apply_discount()
    assert order.total == 80
```

### Bad Example

```
# Tests implementation: asserts on internal method call sequence
def test_discount_calculation(mock_calculator):
    order = Order(customer=gold_customer, items=[Item(price=100)])
    order.apply_discount()
    mock_calculator.apply_rate.assert_called_once_with(0.20)
    # Breaks if we refactor the internal calculation mechanism
```

### Behavior vs. Implementation Tests

| Behavior Test | Implementation Test |
|---|---|
| Asserts on output/state | Asserts on method calls/ordering |
| Survives refactoring | Breaks on refactoring |
| Describes user-visible behavior | Describes internal mechanics |
| "Gold customer pays 80" | "apply_rate called with 0.20" |

### When Implementation Tests Are Acceptable

- Verifying that an external service was called (e.g., payment gateway was charged)
- Confirming a side effect occurred (e.g., email was sent)
- These are behavior from the caller's perspective — the side effect *is* the behavior

### Related Principles

- `TEST-01` Testability (behavior tests need clean interfaces)
- `TEST-02` Test Pyramid (behavior tests are mostly unit-level)
- `FUNC-04` Pure Functions (output-based assertions are behavior tests)

---

## Checkpoint

Quick verification steps for Testing & Quality:

1. **Testability**: Can every class be instantiated in a test without external
   services? If not, inject dependencies.
2. **Pyramid Shape**: Count tests by type. Is the ratio roughly 70% unit / 20%
   integration / 10% E2E?
3. **Behavior Focus**: Do any tests assert on mock call counts or internal method
   sequences? Rewrite to assert on output or observable state.
4. **Speed**: Does the full test suite run in under 5 minutes? If not, move slow
   tests to integration/E2E and add unit test equivalents.
