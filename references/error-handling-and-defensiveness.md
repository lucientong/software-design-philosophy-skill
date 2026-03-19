# Error Handling & Defensiveness

Detailed reference for rules `ERRH-01` through `ERRH-04`.

---

## ERRH-01 · Fail Fast

### Definition

Detect and report errors at the earliest possible point. Do not let invalid state
propagate through the system.

### Rationale

Silent failures are the most expensive kind of bug. When an error is detected
early, the stack trace points directly to the cause. When it propagates, the
symptom appears far from the source, turning a 5-minute fix into a day-long
investigation.

### Good Example

```
def transfer_funds(from_account, to_account, amount):
    if amount <= 0:
        raise ValueError(f"Transfer amount must be positive, got {amount}")
    if from_account.balance < amount:
        raise InsufficientFunds(from_account.id, amount, from_account.balance)

    # Only proceed if all preconditions are met
    from_account.debit(amount)
    to_account.credit(amount)
```

### Bad Example

```
def transfer_funds(from_account, to_account, amount):
    # Silently does nothing on invalid input — caller never knows
    if amount > 0 and from_account.balance >= amount:
        from_account.balance -= amount
        to_account.balance += amount
    # No error, no feedback — the bug surfaces hours later in reconciliation
```

### Fail-Fast Strategies

| Strategy | When to Use |
|---|---|
| Precondition checks (assertions / guards) | At function entry |
| Type systems and type hints | At compile time / static analysis |
| Schema validation | At API boundaries |
| Null-object pattern rejection | When `None` is not a valid value |
| Constructor validation | When creating value objects or entities |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Returning `None` on error instead of raising | Raise a specific exception |
| Silently defaulting on invalid input | Validate and reject invalid input |
| Catching generic `Exception` and logging without re-raising | Catch specific exceptions; re-raise or handle explicitly |
| Ignoring return codes from system calls | Check and handle every return code |

### Related Principles

- `ERRH-02` Boundary Validation (validate inputs early)
- `ERRH-03` Explicit Error Handling (never swallow errors)
- `FUNC-05` Guard Clauses (fail fast at function entry)

---

## ERRH-02 · Boundary Validation

### Definition

Validate all external inputs at system boundaries before processing. Treat
everything from outside the current trust boundary as untrusted.

### Rationale

Unvalidated input is the root cause of injection attacks, data corruption, and
unexpected crashes. The system boundary (API endpoints, file parsers, message
consumers) is the only reliable place to enforce input contracts.

### Good Example

```
# Validate at the API boundary — internal code can trust clean data
@app.post("/orders")
def create_order(request):
    schema = OrderSchema()
    validated = schema.load(request.json)  # Validate here, once
    return order_service.create(validated)  # Service receives clean data
```

### Bad Example

```
# No validation at boundary — checks scattered (or missing) inside
@app.post("/orders")
def create_order(request):
    return order_service.create(request.json)  # Raw, untrusted data flows in

class OrderService:
    def create(self, data):
        # Maybe validates some fields, maybe not — inconsistent
        if "items" in data:  # Partial, fragile check
            ...
```

### Boundary Validation Patterns

| Pattern | Use Case |
|---|---|
| Schema validation (JSON Schema, Pydantic, Zod) | API request/response validation |
| Input sanitization | User-supplied strings (HTML, SQL, paths) |
| Allowlist / enum checks | Constrained values (status codes, types) |
| Range / length checks | Numeric bounds, string lengths |
| Type coercion with validation | Converting string inputs to typed values |

### Trust Boundaries

```
┌─ UNTRUSTED ──────────────────────────────────────┐
│  User input, HTTP requests, file uploads,        │
│  message queues, third-party API responses,      │
│  environment variables, config files             │
└──────────────────────────────────────────────────┘
         │ VALIDATE HERE
         ▼
┌─ TRUSTED ────────────────────────────────────────┐
│  Domain logic, services, repositories            │
│  (can assume clean data)                         │
└──────────────────────────────────────────────────┘
```

### Related Principles

- `ERRH-01` Fail Fast (reject bad input immediately)
- `APID-01` Least Surprise (APIs reject invalid input clearly)
- `ERRH-03` Explicit Errors (validation failures are explicit)

---

## ERRH-03 · Explicit, Typed Error Handling

### Definition

Use explicit, typed error handling. Never swallow exceptions, return ambiguous
`null`/`None`, or use error codes without checking them.

### Rationale

Swallowed exceptions hide bugs. Ambiguous `null` returns shift error discovery to
callers who are unprepared for it. Explicit errors make failure visible and force
callers to handle it.

### Good Example

```
# Explicit error type — caller knows exactly what can go wrong
class UserNotFound(Exception):
    def __init__(self, user_id):
        self.user_id = user_id
        super().__init__(f"User {user_id} not found")

def get_user(user_id):
    user = repo.find(user_id)
    if user is None:
        raise UserNotFound(user_id)
    return user
```

### Bad Example

```
# Returns None — caller must guess if None means "not found" or "error"
def get_user(user_id):
    try:
        return repo.find(user_id)
    except Exception:
        return None  # Swallowed! Was it a DB error? Network timeout? Not found?
```

### Error Handling Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Catch-all `except Exception: pass` | Hides every possible failure | Catch specific exceptions |
| Returning `None` for errors | Ambiguous — is `None` a valid value or an error? | Raise a typed exception |
| Returning error codes without checking | Callers forget to check | Use exceptions or Result types |
| Logging and re-raising with `raise Exception(...)` | Destroys the original stack trace | Use `raise` without arguments or `raise ... from ...` |
| Wrapping everything in try/except at every layer | Duplicate handling; confusing flow | Handle at the appropriate layer only |

### Design by Contract Perspective

- **Precondition**: What the caller must provide (validate at boundary)
- **Postcondition**: What the function guarantees (never return invalid state)
- **Invariant**: What must always be true (enforce in constructors and setters)

### Related Principles

- `ERRH-01` Fail Fast
- `ERRH-04` Structured Error Types
- `ABST-02` LSP (subtypes must honor error contracts)

---

## ERRH-04 · Structured, Domain-Specific Error Types

### Definition

Define structured, domain-specific error types with enough context for callers to
make decisions. Include the error category, relevant identifiers, and a
human-readable message.

### Rationale

Generic errors (`Exception("something went wrong")`) force callers to parse
strings or guess the cause. Structured errors enable precise recovery logic,
clear logging, and meaningful user-facing messages.

### Good Example

```
# Rich, structured error
class PaymentDeclined(DomainError):
    def __init__(self, order_id, reason, decline_code):
        self.order_id = order_id
        self.reason = reason
        self.decline_code = decline_code
        super().__init__(
            f"Payment for order {order_id} declined: {reason} ({decline_code})"
        )

# Caller can act precisely
try:
    charge(order)
except PaymentDeclined as e:
    if e.decline_code == "insufficient_funds":
        suggest_alternative_payment(e.order_id)
    else:
        escalate(e)
```

### Bad Example

```
# Generic error — caller has no structured information
raise Exception("Payment failed")

# Caller must parse a string to understand the error
try:
    charge(order)
except Exception as e:
    if "insufficient" in str(e):  # Fragile string matching
        ...
```

### Error Type Hierarchy Pattern

```
DomainError (base)
├── ValidationError
│   ├── MissingFieldError
│   └── InvalidFormatError
├── NotFoundError
│   ├── UserNotFoundError
│   └── OrderNotFoundError
├── ConflictError
│   └── DuplicateOrderError
└── ExternalServiceError
    ├── PaymentGatewayError
    └── ShippingProviderError
```

### Related Principles

- `ERRH-03` Explicit Error Handling (typed errors are explicit)
- `APID-01` Least Surprise (error types are part of the API contract)
- `ERRH-01` Fail Fast (structured errors carry diagnostic context)

---

## Checkpoint

Quick verification steps for Error Handling & Defensiveness:

1. **Fail Fast**: Are there any code paths that silently ignore invalid state?
   Every invalid condition should raise or return an error.
2. **Boundary Validation**: Are external inputs (API, file, config) validated at
   the entry point? Is internal code free of redundant re-validation?
3. **No Swallowing**: Search for `except:`, `except Exception: pass`, and
   functions that return `None` on error. Eliminate them.
4. **Structured Errors**: Are error types specific to the domain? Do they carry
   enough context for callers to act without string parsing?
5. **Define Errors Out of Existence**: Are there string or int fields that
   should be enums? Are there optional fields that should be required? Can any
   runtime validation be replaced by type-safe design?
