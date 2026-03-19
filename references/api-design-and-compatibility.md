# API Design & Compatibility

Detailed reference for rules `APID-01` through `APID-03`.

---

## APID-01 · Principle of Least Surprise

### Definition

APIs should behave as callers expect. Method names, parameter semantics, return
types, and error behavior should be intuitive and consistent with common
conventions.

### Rationale

When an API surprises its callers, they misuse it — leading to bugs that are hard
to trace because the code "looks correct." An unsurprising API reduces the need
for documentation and minimizes integration errors.

### Good Example

```
# Intuitive: "find" returns None when not found; "get" raises when not found
user = repo.find_by_email("alice@example.com")  # Returns None if missing
user = repo.get_by_email("alice@example.com")    # Raises UserNotFound if missing
```

### Bad Example

```
# Surprising: "get" returns -1 on failure (like C's indexOf)
user = repo.get_user(42)
# Returns -1 when not found — callers expect a User or an exception
if user == -1:  # Easy to forget; breaks type expectations
    ...
```

### Surprise Detection Checklist

| Question | If "Yes" → Surprise Risk |
|---|---|
| Does the function name imply one behavior but perform another? | Rename or restructure |
| Does the function mutate input parameters? | Return a new object instead, or document clearly |
| Does the function return different types depending on success/failure? | Use a consistent return type or raise |
| Are required parameters optional (with dangerous defaults)? | Make them required |
| Does the function have hidden side effects (logging, caching, notifications)? | Document or separate |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| `delete()` returns the deleted object (unexpected) | Return void or a boolean success indicator |
| `save()` silently creates or updates based on whether ID exists | Split into `create()` and `update()` |
| Setter method that returns `self` for chaining (unexpected in some ecosystems) | Follow the ecosystem's convention |

### Related Principles

- `NAME-01` Intention-revealing names (names set expectations)
- `APID-02` Consistency (consistent APIs are unsurprising)
- `ERRH-03` Explicit errors (error behavior should not surprise)

---

## APID-02 · API Consistency

### Definition

Keep naming conventions, parameter ordering, return types, and error handling
patterns consistent across the entire API surface.

### Rationale

Consistency enables callers to guess correctly. When `create_user` returns the
created object, callers expect `create_order` to do the same. Inconsistency
forces callers to read documentation for every method.

### Good Example

```
# Consistent CRUD pattern across all resources
class UserService:
    def create_user(self, data) -> User: ...
    def get_user(self, id) -> User: ...        # Raises NotFound
    def update_user(self, id, data) -> User: ...
    def delete_user(self, id) -> None: ...

class OrderService:
    def create_order(self, data) -> Order: ...
    def get_order(self, id) -> Order: ...      # Same pattern
    def update_order(self, id, data) -> Order: ...
    def delete_order(self, id) -> None: ...
```

### Bad Example

```
# Inconsistent: different naming, return types, and error handling per resource
class UserService:
    def create_user(self, data) -> User: ...       # Returns object
    def find_user(self, id) -> Optional[User]: ... # Returns None

class OrderService:
    def new_order(self, data) -> dict: ...         # Different verb, returns dict
    def get_order(self, id) -> Order: ...          # Raises on not found
    def remove_order(self, id) -> bool: ...        # Different verb, returns bool
```

### Consistency Dimensions

| Dimension | Apply Consistently |
|---|---|
| Naming | Use the same verbs: `create/get/update/delete` or `add/find/modify/remove` |
| Parameter order | Always `(id, data)` or always `(data, id)` — never mix |
| Return types | Same operation → same return type across resources |
| Error handling | Same failure → same exception type across resources |
| Pagination | Same parameters and response structure for all list endpoints |
| Null handling | Always `Optional` or always raise — never mix |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Mixed verbs: `create_user` but `new_order` | Standardize to one set of verbs |
| Some methods return `None` on not found, others raise | Pick one strategy; enforce globally |
| Different date formats across endpoints | Standardize to ISO 8601 |

### Related Principles

- `APID-01` Least Surprise (consistency is the foundation of predictability)
- `NAME-02` Ubiquitous Language (consistent domain terms)
- `NAME-04` Language Conventions (follow ecosystem norms)

---

## APID-03 · Backward Compatibility

### Definition

Maintain backward compatibility in public APIs. Introduce changes through
additive modifications and deprecation cycles. Breaking changes require
versioning.

### Rationale

Breaking callers erodes trust and creates costly migration work across
potentially many consumers. Planning for evolution from day one is far cheaper
than retrofitting compatibility.

### Additive Change Strategy

```
# Version 1: original API
def get_user(self, user_id) -> User: ...

# Version 1.1: additive change — new optional parameter, old callers unaffected
def get_user(self, user_id, include_profile=False) -> User: ...

# Version 1.2: new method — old method still works
def get_user_with_orders(self, user_id) -> UserWithOrders: ...
```

### Breaking Change Protocol

1. **Deprecate**: Mark the old API as deprecated with a clear migration path
2. **Grace period**: Maintain both old and new APIs for at least one release cycle
3. **Communicate**: Document the breaking change, timeline, and migration guide
4. **Version**: Use semantic versioning — breaking changes increment the major version

### Versioning Approaches

| Approach | When to Use |
|---|---|
| URL path versioning (`/v1/users`) | REST APIs with external consumers |
| Header versioning (`Accept: application/vnd.api.v2+json`) | APIs needing version flexibility |
| Semantic versioning (semver) | Libraries and packages |
| Additive-only (never break) | Internal APIs with many consumers |

### Idempotency

Design write operations to be safely retryable:

```
# Idempotent: using a client-provided idempotency key
POST /orders
Idempotency-Key: abc-123
# Second call with same key returns the same result, no duplicate order
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Renaming a field in a JSON response | Add the new field; keep the old one (deprecated) |
| Changing a parameter from optional to required | Keep it optional; add validation in new version |
| Removing an endpoint without notice | Deprecate first; remove in next major version |

### Related Principles

- `APID-01` Least Surprise (backward-compatible APIs don't surprise existing callers)
- `APID-02` Consistency (versioning strategy should be consistent)
- `ABST-05` OCP (extension without modification)

---

## Checkpoint

Quick verification steps for API Design & Compatibility:

1. **Least Surprise**: Ask a colleague to predict what each API method does from
   its name alone. If they guess wrong, rename it.
2. **Consistency**: List all CRUD methods across resources. Are naming, parameter
   order, and return types uniform?
3. **Backward Compatibility**: Does the change add fields/methods without
   removing or renaming existing ones? If not, plan a deprecation cycle.
4. **Idempotency**: Are write operations safe to retry?
