---
name: software-design-philosophy
description: >-
  This skill provides battle-tested software design principles organized as an
  actionable rule set with severity levels. It should be used when writing,
  reviewing, or refactoring code to ensure high quality, maintainability, and
  correctness. Covers SOLID, KISS, DRY, YAGNI, clean code, error handling,
  testing, API design, domain-driven design, defensive programming, and modern
  functional patterns. Adopts the Reviewer design pattern — rules are applied
  during code generation and verified via a post-generation self-review
  checklist.
---

# Software Design Philosophy

## Overview

This skill encodes the most impactful modern software design principles into a
compact, severity-graded rule set. It adopts the **Reviewer pattern**: the Agent
applies rules during code generation and runs a Self-Review Checklist afterward
to catch violations before delivering output.

**When to activate:** any task that involves writing, modifying, reviewing, or
refactoring code — including new features, bug fixes, API design, and
architecture decisions.

**Workflow:**
1. Load rules at task start.
2. Follow rules during code generation.
3. Run Self-Review Checklist after generation.
4. Report any violations using the structured format.
5. Auto-remediate CRITICAL violations before delivering output.
6. Consult `references/<category>.md` when deeper guidance is needed.

---

## Severity Levels

| Tag | Meaning | Agent Behavior |
|---|---|---|
| **`[CRITICAL]`** | Violation causes bugs, security holes, or architectural decay. | Must comply. Auto-fix before output. |
| **`[RECOMMENDED]`** | Strongly advised; improves quality and maintainability. | Follow unless a documented trade-off justifies deviation. |
| **`[OPTIONAL]`** | Situational; beneficial when context supports it. | Adopt when it improves the specific solution. |

---

## 1 · Architecture & Modularity

**`ARCH-01`** `[CRITICAL]` Assign each module, class, or service exactly one
reason to change.
— Multiple responsibilities create coupling magnets where changes ripple
unpredictably.

**`ARCH-02`** `[CRITICAL]` Separate concerns into distinct layers or modules
that can evolve independently.
— Mixing UI, business logic, and data access in one unit blocks reuse and
parallel development.

**`ARCH-03`** `[CRITICAL]` Design each component to do one thing well and
compose with others through clear interfaces.
— Small, focused components are easier to understand, test, and replace.

**`ARCH-04`** `[RECOMMENDED]` Align module boundaries with domain boundaries
using ubiquitous language.
— Misaligned boundaries leak domain concepts across modules, increasing
cognitive load.

**`ARCH-05`** `[OPTIONAL]` Keep modules orthogonal — changing one should not
require changing another.
— Orthogonality reduces the blast radius of modifications and simplifies
reasoning.

**`ARCH-06`** `[RECOMMENDED]` Prefer deep modules — simple interfaces that
hide powerful, complex implementations behind them.
— A module's value is measured by the functionality it provides minus the
complexity of its interface. Shallow modules (many parameters, little behavior)
push complexity onto callers rather than absorbing it.

> Detail → `references/architecture-and-modularity.md`

---

## 2 · Simplicity & Pragmatism

**`SIMP-01`** `[CRITICAL]` Choose the simplest solution that fully satisfies
the current requirement. Measure complexity by its two components: dependencies
(what a reader must know) and obscurity (what is not obvious).
— Complexity is the primary enemy of reliability; every unnecessary dependency
or obscure behavior is a future liability.

**`SIMP-02`** `[CRITICAL]` Do not build features, abstractions, or
infrastructure for speculative future needs.
— Speculative code adds cost now and may never be needed; delete what is not
used.

**`SIMP-03`** `[RECOMMENDED]` Extract shared logic only when duplication is
proven (rule of three), not predicted.
— Premature abstraction creates wrong abstractions; duplication is cheaper than
the wrong abstraction.

**`SIMP-04`** `[OPTIONAL]` Use tracer-bullet development: build thin
end-to-end slices before filling in details.
— An early working skeleton validates architecture and uncovers integration
issues.

**`SIMP-05`** `[RECOMMENDED]` Refactor in small, behavior-preserving steps —
never mix structural cleanup with feature changes.
— Large rewrites introduce risk and are hard to review. Each refactoring step
should pass all existing tests; if a test must change, that is a separate commit.

> Detail → `references/simplicity-and-pragmatism.md`

---

## 3 · Abstraction & Interfaces

**`ABST-01`** `[CRITICAL]` Depend on abstractions (interfaces / protocols),
not concrete implementations.
— Concrete dependencies create tight coupling; abstractions allow substitution
and testing.

**`ABST-02`** `[CRITICAL]` Ensure subtypes are fully substitutable for their
base types without altering program correctness.
— Violating substitutability breaks polymorphism and causes subtle runtime
errors.

**`ABST-03`** `[RECOMMENDED]` Design interfaces to be client-specific and
minimal — no forced unused method implementations.
— Fat interfaces impose unnecessary burdens on implementors and obscure intent.

**`ABST-04`** `[RECOMMENDED]` Extend behavior through composition and
delegation rather than deep inheritance hierarchies.
— Inheritance creates fragile coupling; composition provides flexibility and
explicit dependencies.

**`ABST-05`** `[OPTIONAL]` Design modules to be open for extension but closed
for modification.
— Stable modules can gain new behavior through extension points without risking
existing callers.

> Detail → `references/abstraction-and-interfaces.md`

---

## 4 · Naming & Readability

**`NAME-01`** `[CRITICAL]` Use intention-revealing names for all variables,
functions, classes, and modules.
— A name is the first and most-read documentation; vague names force readers to
decode context.

**`NAME-02`** `[CRITICAL]` Use domain vocabulary consistently — the same
concept gets the same name everywhere.
— Inconsistent terminology causes confusion between code and business
requirements.

**`NAME-03`** `[RECOMMENDED]` Write self-documenting code; add comments only
to explain *why*, never *what*.
— Comments that restate code rot quickly; clear code with intent-revealing names
is the best documentation.

**`NAME-04`** `[OPTIONAL]` Match naming conventions to the language ecosystem
(e.g., `snake_case` in Python, `camelCase` in JavaScript).
— Following community conventions reduces friction for contributors and tool
compatibility.

> Detail → `references/naming-and-readability.md`

---

## 5 · Functions & Control Flow

**`FUNC-01`** `[CRITICAL]` Keep functions short and focused — each function
performs one logical task.
— Long functions hide bugs, resist testing, and mix abstraction levels.

**`FUNC-02`** `[CRITICAL]` Maintain a single level of abstraction within each
function.
— Mixing high-level orchestration with low-level detail obscures the function's
intent.

**`FUNC-03`** `[RECOMMENDED]` Limit the knowledge a unit has about other
units — talk only to immediate collaborators.
— Long method chains (`a.b().c().d()`) create hidden coupling and violate
encapsulation.

**`FUNC-04`** `[RECOMMENDED]` Prefer pure functions with no side effects;
isolate side effects at system boundaries.
— Pure functions are predictable, testable, and composable; side effects belong
at the edges.

**`FUNC-05`** `[OPTIONAL]` Reduce cyclomatic complexity — flatten nested
conditionals with early returns or guard clauses.
— High nesting depth increases cognitive load and error probability.

> Detail → `references/functions-and-control-flow.md`

---

## 6 · Error Handling & Defensiveness

**`ERRH-01`** `[CRITICAL]` Fail fast — detect and report errors at the
earliest possible point.
— Silent failures propagate corrupted state; early detection limits blast
radius.

**`ERRH-02`** `[CRITICAL]` Validate all external inputs at system boundaries
before processing.
— Unvalidated input is the root cause of injection, corruption, and crash
vulnerabilities.

**`ERRH-03`** `[CRITICAL]` Use explicit, typed error handling — never swallow
exceptions or return ambiguous null.
— Swallowed exceptions hide bugs; null returns shift error discovery to callers
unprepared for it.

**`ERRH-04`** `[RECOMMENDED]` Define structured, domain-specific error types
with enough context for callers to act.
— Generic errors force callers to guess; structured errors enable precise
recovery and logging.

**`ERRH-05`** `[RECOMMENDED]` Define errors out of existence — design APIs and
data structures so that invalid states are unrepresentable.
— The best error handling is the error that cannot occur. Use type systems,
enums, and constructor validation to make misuse a compile-time or
construction-time failure rather than a runtime surprise.

> Detail → `references/error-handling-and-defensiveness.md`

---

## 7 · Testing & Quality

**`TEST-01`** `[CRITICAL]` Design for testability — inject dependencies and
expose seams for test doubles.
— Code that cannot be tested in isolation hides defects and resists refactoring.

**`TEST-02`** `[RECOMMENDED]` Follow the test pyramid — maximize unit tests,
limit integration tests, minimize E2E tests.
— Unit tests are fast and precise; excessive E2E tests are slow, flaky, and
costly to maintain.

**`TEST-03`** `[OPTIONAL]` Write tests that verify behavior and intent, not
implementation details.
— Tests coupled to implementation break on refactoring without catching real
regressions.

> Detail → `references/testing-and-quality.md`

---

## 8 · API Design & Compatibility

**`APID-01`** `[CRITICAL]` Follow the principle of least surprise — APIs
should behave as callers expect.
— Surprising behavior causes misuse bugs that are hard to trace and debug.

**`APID-02`** `[RECOMMENDED]` Keep API naming, parameter ordering, and return
conventions consistent across the surface.
— Inconsistency forces callers to check documentation for every call;
consistency enables guessing correctly.

**`APID-03`** `[OPTIONAL]` Maintain backward compatibility; use additive
changes and deprecation cycles for breaking changes.
— Breaking callers erodes trust and creates costly migration work.

> Detail → `references/api-design-and-compatibility.md`

---

## 9 · Modern Patterns

**`MODN-01`** `[RECOMMENDED]` Prefer immutable data structures; create new
copies instead of mutating state.
— Immutability eliminates a class of concurrency bugs and simplifies state
reasoning.

**`MODN-02`** `[RECOMMENDED]` Minimize side effects — keep computations pure
and push I/O to the edges.
— Side-effect-free code is easier to test, cache, parallelize, and compose.

**`MODN-03`** `[OPTIONAL]` Prefer declarative over imperative style when the
language and context support it.
— Declarative code expresses intent over mechanism, improving readability and
reducing off-by-one errors.

> Detail → `references/modern-patterns.md`

---

## Self-Review Checklist

Run this checklist after generating code. Each item maps to a CRITICAL rule.

### Architecture
- [ ] Every class / module has a single, clear responsibility? → `ARCH-01`
- [ ] Concerns (UI, logic, data) live in separate layers? → `ARCH-02`
- [ ] Components are small, focused, and compose via interfaces? → `ARCH-03`
- [ ] Modules expose simple interfaces hiding complex internals? → `ARCH-06`

### Simplicity
- [ ] No unnecessary complexity or speculative abstractions? → `SIMP-01`
- [ ] No code built for unconfirmed future requirements? → `SIMP-02`
- [ ] Refactoring changes separated from feature changes? → `SIMP-05`

### Abstraction
- [ ] Dependencies point toward abstractions, not concretions? → `ABST-01`
- [ ] Subtypes are fully substitutable for base types? → `ABST-02`

### Naming
- [ ] All names reveal intent — no `temp`, `data`, `x` for real variables? → `NAME-01`
- [ ] Domain terms are used consistently across the codebase? → `NAME-02`

### Functions
- [ ] Every function does one thing at one abstraction level? → `FUNC-01`, `FUNC-02`

### Error Handling
- [ ] Errors are detected and raised at the earliest point? → `ERRH-01`
- [ ] All external inputs are validated at boundaries? → `ERRH-02`
- [ ] No swallowed exceptions or ambiguous null returns? → `ERRH-03`
- [ ] Invalid states made unrepresentable where practical? → `ERRH-05`

### Testing
- [ ] Dependencies are injectable; no hidden singletons? → `TEST-01`

### API
- [ ] API behavior matches caller expectations (no surprises)? → `APID-01`

---

## Violation Reporting Format

When a design violation is found, report it using this structure:

```
- Severity: [CRITICAL | RECOMMENDED | OPTIONAL]
- Rule: <rule-id> (e.g., ARCH-01)
- Location: <file:line or component name>
- Issue: <one-sentence description of what violates the rule>
- Fix: <one-sentence suggested remediation>
```

For CRITICAL violations, apply the fix automatically before delivering output.
For RECOMMENDED violations, report and suggest the fix.
For OPTIONAL violations, mention only if the improvement is significant.

---

## Reference Lookup Guide

Each category has a dedicated reference file in `references/` with:
- Principle definitions and rationale
- Good vs. bad code examples (language-agnostic pseudocode)
- Common violations and fixes
- Checkpoint — quick verification steps
- Cross-references to related principles

To find details on a specific rule or principle:

```
grep -i "<keyword>" references/
```

Examples:
- `grep -i "single responsibility" references/architecture-and-modularity.md`
- `grep -i "fail fast" references/error-handling-and-defensiveness.md`
- `grep -i "immutab" references/modern-patterns.md`
