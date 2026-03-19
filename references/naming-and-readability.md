# Naming & Readability

Detailed reference for rules `NAME-01` through `NAME-04`.

---

## NAME-01 · Intention-Revealing Names

### Definition

Every variable, function, class, and module name should clearly communicate its
purpose, role, and usage without requiring additional context or comments.

### Rationale

A name is read hundreds of times more than it is written. A vague name forces
every reader to stop and decode; a clear name makes the code self-documenting.

### Good Example

```
elapsed_time_in_days = current_date - start_date
overdue_invoices = [inv for inv in invoices if inv.is_past_due()]
max_retry_count = 3
```

### Bad Example

```
d = x - y          # What is d? What are x and y?
lst = [i for i in items if i.flag]  # What flag? What does the list represent?
n = 3               # Magic number with no context
```

### Naming Heuristics

| Element | Heuristic | Example |
|---|---|---|
| Boolean | Reads as a yes/no question | `is_active`, `has_permission`, `can_retry` |
| Function | Verb + noun describing the action | `calculate_total`, `send_notification` |
| Class | Noun describing the concept | `InvoiceCalculator`, `UserRepository` |
| Collection | Plural noun describing contents | `active_users`, `pending_orders` |
| Constant | SCREAMING_SNAKE describing the invariant | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Single-letter variables outside tiny loops | Use descriptive names |
| Generic names: `data`, `info`, `temp`, `result`, `value` | Name by what the data represents |
| Abbreviated names: `usr_mgr`, `calc_amt` | Spell out: `user_manager`, `calculated_amount` |
| Type-encoded names: `strName`, `listItems` | Drop type prefix; let the type system handle it |

### Related Principles

- `NAME-02` Ubiquitous Language (domain-correct names)
- `NAME-03` Self-documenting code (names replace comments)
- `FUNC-01` Small functions (named functions document intent)

---

## NAME-02 · Ubiquitous Language

### Definition

Use domain vocabulary consistently — the same business concept gets the same name
in code, documentation, tests, and conversations. Avoid synonyms.

### Rationale

When code uses "customer" but the business says "client", or when "order" means
different things in different modules, communication breaks down. Ubiquitous
language creates a shared mental model.

### Good Example

```
# Business says "Policy" and "Claim" — code matches
class InsurancePolicy:
    def file_claim(self, claim: Claim) -> ClaimResult: ...
```

### Bad Example

```
# Business says "Policy" but code says "Contract"
# Business says "Claim" but code says "Request"
class Contract:
    def submit_request(self, req: ServiceRequest) -> ProcessResult: ...
# Every conversation requires mental translation
```

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| Using technical jargon instead of domain terms | Adopt the terms domain experts use |
| Same concept with different names in different modules | Establish a glossary; enforce consistency |
| Code names that evolved from an old domain model | Rename to match current business language |

### Related Principles

- `ARCH-04` Bounded Contexts (each context has its own ubiquitous language)
- `NAME-01` Intention-revealing names
- `APID-02` API Consistency

---

## NAME-03 · Self-Documenting Code Over Comments

### Definition

Write code that communicates intent through structure, naming, and small
functions. Add comments only to explain *why* something is done, never *what* is
done.

### Rationale

Comments that restate the code rot quickly — when code changes, comments are
often forgotten. Clear code with intention-revealing names is the most reliable
documentation.

### Good Example

```
# The code explains what; the comment explains why
def retry_with_backoff(fn, max_attempts=3):
    # Exponential backoff avoids thundering herd on the payment gateway
    for attempt in range(max_attempts):
        try:
            return fn()
        except TransientError:
            sleep(2 ** attempt)
    raise MaxRetriesExceeded()
```

### Bad Example

```
# Comment restates the code — adds no value
def process(x):
    # increment x by 1
    x = x + 1
    # check if x is greater than 10
    if x > 10:
        # return true
        return True
```

### Comment Anti-Patterns

| Anti-Pattern | Why It's Harmful |
|---|---|
| Restating code in English | Doubles maintenance; becomes stale |
| Commented-out code blocks | Use version control instead |
| TODO comments that stay for years | Track in issue tracker; delete from code |
| Journal comments ("// Changed by Bob, 2023-01-15") | Version control handles history |
| Mandated doc comments on obvious getters/setters | Noise that dilutes real documentation |

### Related Principles

- `NAME-01` Intention-revealing names (names are inline documentation)
- `FUNC-01` Small functions (each function's name documents a step)
- `SIMP-01` KISS (if code needs extensive comments, simplify the code)

---

## NAME-04 · Language-Ecosystem Conventions

### Definition

Match naming conventions to the language ecosystem: `snake_case` in Python,
`camelCase` in JavaScript/TypeScript, `PascalCase` for classes in most languages,
`SCREAMING_SNAKE_CASE` for constants.

### Rationale

Following community conventions reduces friction for contributors, ensures tool
compatibility (linters, formatters, IDE auto-complete), and makes code feel
idiomatic.

### Convention Quick Reference

| Language | Functions/Variables | Classes | Constants | Modules/Files |
|---|---|---|---|---|
| Python | `snake_case` | `PascalCase` | `UPPER_SNAKE` | `snake_case.py` |
| JavaScript/TS | `camelCase` | `PascalCase` | `UPPER_SNAKE` | `camelCase.ts` or `kebab-case.ts` |
| Go | `camelCase` (unexported), `PascalCase` (exported) | `PascalCase` | `PascalCase` or `UPPER_SNAKE` | `snake_case.go` |
| Java | `camelCase` | `PascalCase` | `UPPER_SNAKE` | `PascalCase.java` |
| Rust | `snake_case` | `PascalCase` | `UPPER_SNAKE` | `snake_case.rs` |

### Common Violations & Fixes

| Violation | Fix |
|---|---|
| `camelCase` variables in Python | Use `snake_case` |
| `snake_case` functions in JavaScript | Use `camelCase` |
| Mixing conventions within the same file | Pick one; enforce with a linter |

### Related Principles

- `APID-02` API Consistency (external APIs follow language conventions)
- `NAME-01` Intention-revealing names
- `SIMP-01` KISS (conventions are simple and predictable)

---

## Checkpoint

Quick verification steps for Naming & Readability:

1. **Intent**: Can you understand what each variable/function does without
   reading its implementation?
2. **Domain**: Do code names match the terms used in requirements and by
   stakeholders?
3. **Comments**: Are there any comments that simply restate the code? Remove
   them. Do remaining comments explain *why*?
4. **Conventions**: Does the code follow the target language's naming
   conventions? Run the ecosystem's linter.
