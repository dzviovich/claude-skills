---
description: "Language-agnostic comment cleanup principles. Always active."
---

# Rule: Clean Comments

## Principle

Comments are liabilities, not assets. Every comment must earn its place by conveying information the code cannot express on its own. When cleaning up comments, apply ruthless scrutiny: if a comment adds no information, delete it.

---

## What to DELETE

### Noise comments
Comments that restate what the code already says. They rot as code changes and actively mislead.

```
# bad — says nothing the code doesn't already say
x = x + 1  # increment x

# bad — restates the function signature
def get_user(id):  # get a user by id
```

### Commented-out code
Dead code that was left "just in case." It is the responsibility of version control to preserve history — not comments.

```
# bad
# old_result = compute_v1(data)
result = compute_v2(data)
```

### TODO/FIXME left from development
Temporary scaffolding that was never cleaned up. If the work is real, it belongs in an issue tracker — not inline.

```
# bad
# TODO: fix this later
# FIXME: this sometimes crashes
```

### Banner and section dividers
Visual decoration that substitutes for good structure. If code needs a banner to be navigable, it should be split into smaller units.

```
# bad
# ============================================================
# SECTION: Data Processing
# ============================================================
```

### Changelog comments
Version history preserved in source code rather than in git. These are always stale and confusing.

```
# bad
# 2024-01-15 jsmith: updated for new API
# 2023-11-02 ajones: initial implementation
```

### Closed-over state comments
Comments that explain _what_ the surrounding code does at the expression or statement level.

```
# bad
# check if user is active
if user.status == "active":
```

---

## What to KEEP

### Why, not what
Comments that explain the _reason_ behind a decision — especially when the code is correct but non-obvious.

```
# good — explains the reason, not the action
# Retry up to 3 times: the upstream service occasionally returns 503
# on the first attempt during cold starts.
for attempt in range(3):
    ...
```

### Non-obvious invariants and preconditions
Constraints that callers must satisfy that cannot be encoded in the type system or enforced at runtime without significant cost.

```
# good — constraint the compiler cannot enforce
# Requires: list is sorted in ascending order
def binary_search(lst, target):
    ...
```

### Intentionally surprising code
Code that looks wrong but is correct. Without a comment, the next reader will "fix" it.

```
# good — prevents well-intentioned breakage
# Intentionally using integer division here; fractional seconds
# would cause the downstream timer to spin.
interval = duration // step_size
```

### References to external context
Links to specs, bug reports, or standards that motivated a specific implementation choice.

```
# good — anchors the code to its external source of truth
# Per RFC 7231 §6.5.4, respond 404 (not 403) to avoid
# leaking resource existence to unauthenticated clients.
return 404
```

### Public API documentation
Top-level docstrings/doc-comments on public functions, classes, and modules — in whatever format the language convention dictates. These describe _what_ the unit does, its parameters, return values, and any raised exceptions.

---

## Cleanup Checklist

When reviewing or modifying a file, apply each check to every comment:

| Check | Action |
|-------|--------|
| Does it restate the code? | Delete |
| Is it commented-out code? | Delete |
| Is it a TODO/FIXME? | Delete (file a ticket if needed) |
| Is it a banner/divider? | Delete |
| Is it a changelog entry? | Delete |
| Does it explain *why*? | Keep |
| Does it document a non-obvious invariant? | Keep |
| Does it prevent a reader from "fixing" correct code? | Keep |
| Does it link to an external spec or issue? | Keep |
| Is it a public API docstring? | Keep |

---

## Tone and Style

- Write in **imperative present tense**: "Return the cached value" not "Returns the cached value" or "Returned the cached value."
- Use **complete sentences** with punctuation for multi-line comments.
- Keep inline comments **short** (≤ 80 characters total including code).
- Do not pad or align inline comments across multiple lines — this creates noisy diffs when code changes.
- Comments are addressed to **the next developer** (who may be you, six months from now).

---

## Language-Specific Rules

This file governs general principles. Language-specific standards (formatting, docstring conventions, linting rules) are in dedicated rule files:

- Python → `clean-comments-python.md`
- Go → *(add when needed)*
- TypeScript → *(add when needed)*
