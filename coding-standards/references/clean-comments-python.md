---
description: "Python-specific comment cleanup standards (PEP 8 / PEP 257). Active for **/*.py files."
---

# Rule: Clean Comments — Python

Extends `clean-comments.md` with Python-specific rules derived from PEP 8 and PEP 257. When this file conflicts with the general rule, this file wins.

---

## 1. Block Comments (PEP 8 §Comments)

Block comments precede the code they describe, at the same indentation level.

**Rules:**
- Start with `# ` (hash + single space). Never `#text` or `#  text` (double space).
- Each line of a multi-line block comment begins with `# `.
- Separate paragraphs within a block comment with a line containing a single `#`.
- One blank line above a block comment (unless it is at the top of a block).

```python
# bad — no space after hash
#check connection before sending

# bad — double space
#  check connection before sending

# good
# Check the connection before sending. The socket may have been
# closed by the server during an idle period.
#
# See: https://example.com/issue/42
if not conn.is_alive():
    conn.reconnect()
```

---

## 2. Inline Comments (PEP 8 §Inline Comments)

Inline comments appear on the same line as a statement.

**Rules:**
- At least **two spaces** between the statement and the `#`.
- One space after `#`.
- Use sparingly — only when the line is genuinely non-obvious.
- Never use them to restate what the code does.

```python
# bad — no separation
x = x + 1# increment x

# bad — restates the code
x = x + 1  # increment x

# good — explains why, not what
x = x + 1  # offset by 1; index is 0-based but API expects 1-based
```

---

## 3. Docstrings (PEP 257)

Docstrings are the canonical documentation for modules, classes, and functions. They are read by `help()`, IDEs, and documentation generators.

### 3a. When to write a docstring

| Construct | Docstring required? |
|-----------|-------------------|
| Public module | Yes |
| Public class | Yes |
| Public method / function | Yes |
| `__init__` | Yes — if the class docstring does not cover construction |
| Private / internal (`_name`) | Optional — only if non-obvious |
| One-liner helper whose name is self-documenting | No |

### 3b. One-line docstrings

Use for simple, self-contained functions. The entire docstring fits on one line.

**Rules:**
- Opening and closing `"""` on the **same line**.
- Ends with a **period**.
- Describes the effect as an **imperative phrase** ("Do X"), not a description ("Does X").
- No blank line before or after.

```python
# bad — missing period
def double(x):
    """Return twice the value of x"""

# bad — describes instead of commands
def double(x):
    """Returns twice the value of x."""

# bad — triple quotes on separate lines for a one-liner
def double(x):
    """
    Return twice the value of x.
    """

# good
def double(x):
    """Return twice the value of x."""
```

### 3c. Multi-line docstrings

Use when the summary line alone is insufficient.

**Structure (in order):**
1. **Summary line** — one-line imperative sentence ending with a period. On the same line as the opening `"""`.
2. **Blank line.**
3. **Body** — elaboration, usage notes, algorithm description, or references.
4. **Args / Parameters section** (if applicable).
5. **Returns section** (if applicable).
6. **Raises section** (if applicable).
7. Closing `"""` on its **own line**.

```python
# bad — summary on second line
def fetch_user(user_id: int) -> dict:
    """
    Fetch a user record from the database.
    """

# bad — no blank line between summary and body
def fetch_user(user_id: int) -> dict:
    """Fetch a user record from the database.
    Raises KeyError if the user does not exist.
    """

# good
def fetch_user(user_id: int) -> dict:
    """Fetch a user record from the database.

    Raises KeyError if no record exists for user_id. Callers must
    handle this case; the function does not return None.

    Args:
        user_id: The primary key of the user to retrieve.

    Returns:
        A dict with keys: id, name, email, created_at.

    Raises:
        KeyError: If user_id is not found.
        DatabaseError: If the connection is unavailable.
    """
```

### 3d. Class docstrings

Document the class, not `__init__`. Describe what the class _represents_ and its primary invariants. Construction arguments go in the class docstring (or `__init__` if it has materially different content).

```python
# bad — documents __init__ instead of the class
class RateLimiter:
    """Initialize with max_calls and period."""

# good
class RateLimiter:
    """Token-bucket rate limiter for outbound API calls.

    Allows up to max_calls requests per period (in seconds). Thread-safe
    for concurrent use; uses a threading.Lock internally.

    Args:
        max_calls: Maximum number of calls allowed per period.
        period: Length of the rate window in seconds.
    """
```

### 3e. Module docstrings

Place at the very top of the file, before any imports. Describe what the module provides and any important usage notes.

```python
"""Utilities for normalising and validating postal addresses.

Supports AU, NZ, and GB address formats. Uses the Australia Post
PAF dataset for AU validation; see docs/data-sources.md for details.
"""

import re
...
```

---

## 4. Docstring Format Style

Use **Google style** for Args/Returns/Raises sections (readable in plain text without a renderer). Do not mix styles within a project.

```python
def divide(a: float, b: float) -> float:
    """Divide a by b and return the result.

    Args:
        a: The dividend.
        b: The divisor. Must not be zero.

    Returns:
        The result of a / b.

    Raises:
        ZeroDivisionError: If b is zero.
    """
    if b == 0:
        raise ZeroDivisionError("divisor must not be zero")
    return a / b
```

---

## 5. Python-Specific Anti-Patterns to DELETE

### Docstrings on private internals that restate the name

```python
# bad
def _build_cache(self):
    """Build the cache."""
```

### Type annotations duplicated in docstrings

With modern type hints, do not repeat types in Args/Returns. The signature is the source of truth.

```python
# bad — type repeated in docstring
def get_user(user_id: int) -> dict:
    """Fetch a user.

    Args:
        user_id (int): The user ID.

    Returns:
        dict: The user record.
    """

# good — type annotations in signature only
def get_user(user_id: int) -> dict:
    """Fetch a user record by primary key.

    Args:
        user_id: The primary key of the user to retrieve.

    Returns:
        A dict with keys: id, name, email, created_at.
    """
```

### Encoding declarations (Python 3)

Python 3 files are UTF-8 by default. The `# -*- coding: utf-8 -*-` header is unnecessary noise.

```python
# bad — redundant in Python 3
# -*- coding: utf-8 -*-
```

### Shebang lines in non-executable modules

Only entry-point scripts need a shebang. Library modules do not.

```python
# bad — in a library module
#!/usr/bin/env python
```

---

## 6. Cleanup Checklist (Python)

Apply _in addition to_ the checklist in `clean-comments.md`:

| Check | Action |
|-------|--------|
| Block comment missing space after `#`? | Fix: add space |
| Inline comment fewer than 2 spaces from code? | Fix: add spaces |
| One-line docstring missing period? | Fix: add period |
| One-line docstring uses `"""` on separate lines? | Fix: collapse to one line |
| Multi-line docstring summary not on opening `"""` line? | Fix: move up |
| Multi-line docstring missing blank line after summary? | Fix: add blank line |
| Closing `"""` not on its own line? | Fix: move to own line |
| Args section duplicates type annotations? | Delete the type names |
| Docstring on a private helper that restates the name? | Delete |
| `# -*- coding: utf-8 -*-` header? | Delete |
| Docstring on `__init__` when class docstring covers it? | Delete |

---

## 7. Reference

- [PEP 8 — Comments](https://peps.python.org/pep-0008/#comments)
- [PEP 257 — Docstring Conventions](https://peps.python.org/pep-0257/)
- [Google Python Style Guide — Docstrings](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings)
