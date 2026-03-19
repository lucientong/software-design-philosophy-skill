# Software Design Philosophy Skill

> **39 battle-tested design rules** · **3 severity levels** · **9 categories** · **Reviewer pattern**

An Agent Skill that distills the best modern software design principles into a
compact, severity-graded rule set. Built on the **Reviewer design pattern** — the
Agent applies rules during code generation and runs a self-review checklist
afterward to catch violations before delivering output.

---

## Why This Skill?

AI coding agents produce better code when guided by explicit design principles.
Without guidance, agents tend to:

- Over-engineer solutions with speculative abstractions
- Mix concerns within single functions or classes
- Swallow errors silently or return ambiguous nulls
- Ignore testability and create tightly coupled code

This skill provides a structured rulebook that agents can load, follow, and
self-verify against — turning design knowledge into consistent, high-quality
output.

---

## Design Pattern: Reviewer

This skill adopts the **Reviewer** pattern from Google's
[5 Agent Skill Design Patterns](https://google.github.io/adk-docs/skills/):

```
┌──────────────────────────────────────────────────────────┐
│  1. LOAD    →  Agent loads SKILL.md rules at task start  │
│  2. APPLY   →  Follow rules during code generation       │
│  3. REVIEW  →  Run Self-Review Checklist post-generation  │
│  4. REPORT  →  Output violations in structured format     │
│  5. FIX     →  Auto-remediate CRITICAL violations         │
│  6. DELIVER →  Clean output                               │
└──────────────────────────────────────────────────────────┘
```

The skill also incorporates **Tool Wrapper** elements — domain expertise
(references/) is loaded on demand when the Agent needs deeper guidance.

---

## Severity Levels

| Tag | Meaning | Agent Behavior |
|---|---|---|
| `[CRITICAL]` | Violation causes bugs, security holes, or architectural decay | Must comply; auto-fix before output |
| `[RECOMMENDED]` | Strongly advised; improves quality and maintainability | Follow unless a documented trade-off justifies deviation |
| `[OPTIONAL]` | Situational; beneficial when context supports it | Adopt when it improves the specific solution |

**Distribution:** 16 CRITICAL · 15 RECOMMENDED · 8 OPTIONAL

---

## Covered Design Principles

| Source Principle | Mapped To Category |
|---|---|
| SRP, Separation of Concerns, Unix Philosophy, Bounded Contexts, Orthogonality, Deep Modules | Architecture & Modularity |
| KISS, YAGNI, DRY, Pragmatic Programmer, Small-Step Refactoring | Simplicity & Pragmatism |
| OCP, LSP, ISP, DIP, Composition over Inheritance | Abstraction & Interfaces |
| Clean Code Naming, Ubiquitous Language, Self-Documenting Code | Naming & Readability |
| Small Functions, SLAP, Law of Demeter, Pure Functions | Functions & Control Flow |
| Fail-Fast, Design by Contract, Defensive Programming, Define Errors Out of Existence | Error Handling & Defensiveness |
| Test Pyramid, TDD, Testability-Driven Design | Testing & Quality |
| Least Surprise, API Consistency, Backward Compatibility | API Design & Compatibility |
| Immutability, Minimal Side Effects, Declarative Style | Modern Patterns |

---

## Design Lineage

This skill is intentionally a synthesis rather than a book summary.

| Source | Reflected in the rule set |
|---|---|
| *Refactoring* | Guard clauses, smaller functions, testability before change, late extraction of abstractions, **small-step behavior-preserving refactoring** (`SIMP-05`) |
| *A Philosophy of Software Design* | **Complexity = dependencies + obscurity** (`SIMP-01`), **deep modules** (`ARCH-06`), **define errors out of existence** (`ERRH-05`), reduced coupling, minimal interfaces, simpler APIs |
| *Clean Code* | Intention-revealing names, self-documenting code, one abstraction level per function |
| *The Pragmatic Programmer* | Tracer bullets, orthogonality, pragmatic trade-offs over ceremony |
| *Domain-Driven Design* | Ubiquitous language and domain-aligned boundaries |

---

## Rule Index

| # | Category | Rules | C | R | O | Reference |
|---|---|---|---|---|---|---|
| 1 | Architecture & Modularity | `ARCH-01`–`06` | 3 | 2 | 1 | [reference](references/architecture-and-modularity.md) |
| 2 | Simplicity & Pragmatism | `SIMP-01`–`05` | 2 | 2 | 1 | [reference](references/simplicity-and-pragmatism.md) |
| 3 | Abstraction & Interfaces | `ABST-01`–`05` | 2 | 2 | 1 | [reference](references/abstraction-and-interfaces.md) |
| 4 | Naming & Readability | `NAME-01`–`04` | 2 | 1 | 1 | [reference](references/naming-and-readability.md) |
| 5 | Functions & Control Flow | `FUNC-01`–`05` | 2 | 2 | 1 | [reference](references/functions-and-control-flow.md) |
| 6 | Error Handling & Defensiveness | `ERRH-01`–`05` | 3 | 2 | 0 | [reference](references/error-handling-and-defensiveness.md) |
| 7 | Testing & Quality | `TEST-01`–`03` | 1 | 1 | 1 | [reference](references/testing-and-quality.md) |
| 8 | API Design & Compatibility | `APID-01`–`03` | 1 | 1 | 1 | [reference](references/api-design-and-compatibility.md) |
| 9 | Modern Patterns | `MODN-01`–`03` | 0 | 2 | 1 | [reference](references/modern-patterns.md) |
| | **Total** | **39** | **16** | **15** | **8** | |

*C = CRITICAL · R = RECOMMENDED · O = OPTIONAL*

---

## Installation

### One-command install (recommended)

```bash
# Uses the skills CLI — auto-detects your agents and installs to the right path
npx skills add lucientong/software-design-philosophy-skill
```

### Alternative methods

```bash
# Clone directly into Claude Code personal skills
git clone --depth=1 https://github.com/lucientong/software-design-philosophy-skill \
  ~/.claude/skills/software-design-philosophy

# Or use npx degit for a zero-history copy
npx degit lucientong/software-design-philosophy-skill \
  ~/.claude/skills/software-design-philosophy
```

For **project-level** install (shared with the team via version control):

```bash
mkdir -p .claude/skills
git clone --depth=1 https://github.com/lucientong/software-design-philosophy-skill \
  .claude/skills/software-design-philosophy
```

---

## Usage

Once installed, the Agent will:

1. **Auto-activate** on any coding task (writing, reviewing, refactoring)
2. **Apply rules** from SKILL.md during code generation
3. **Self-review** using the built-in checklist after generation
4. **Report violations** in structured format (severity / rule ID / location /
   issue / fix)
5. **Auto-fix** CRITICAL violations before delivering output
6. **Load references** on demand for deeper guidance on specific rules

### Manual Activation

If the Agent does not auto-activate, prompt it with:

> "Use the software-design-philosophy skill for this task."

---

## Directory Structure

```
software-design-philosophy-skill/
├── SKILL.md                          # Core skill file (rules + checklist)
├── references/                       # Detailed references (loaded on demand)
│   ├── architecture-and-modularity.md
│   ├── simplicity-and-pragmatism.md
│   ├── abstraction-and-interfaces.md
│   ├── naming-and-readability.md
│   ├── functions-and-control-flow.md
│   ├── error-handling-and-defensiveness.md
│   ├── testing-and-quality.md
│   ├── api-design-and-compatibility.md
│   └── modern-patterns.md
└── README.md                         # This file
```

---

## Three-Tier Progressive Disclosure

The skill follows the Agent Skills progressive disclosure model:

| Tier | Content | Loaded When | Size |
|---|---|---|---|
| **Tier 1** | YAML frontmatter (name + description) | Always — skill discovery | ~60 words |
| **Tier 2** | SKILL.md body (rules + checklist) | Skill activates | ~3000 words |
| **Tier 3** | references/ (detailed guides) | Agent needs deep guidance | Unlimited |

This design minimizes token consumption while ensuring the Agent has access to
full detail when needed.

---

## Contributing

Contributions are welcome! To add or modify rules:

1. **Rule format**: `**\`RULE-ID\`** \`[SEVERITY]\` Imperative statement.` +
   one-line rationale
2. **Reference format**: Each principle section needs Definition, Rationale,
   Good/Bad Examples, Common Violations & Fixes, Related Principles
3. **Checkpoint**: Every reference file ends with a Checkpoint section
4. **Consistency**: Update the Rule Index in this README when adding/removing rules
5. **Translations**: Update Chinese translations to match

---

## License

MIT
