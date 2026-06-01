---
description: "pytest testing standards (PEP 8 / PEP 257 aligned). Active for Python test files."
---

# Rule: Testing — Python (pytest)

Extends `clean-comments-python.md` and `clean-comments.md`. All rules here apply to any file discovered by pytest (`test_*.py`, `*_test.py`).

---

## 1. File and Function Naming

| Target | Pattern | Example |
|--------|---------|---------|
| Test files | `test_*.py` or `*_test.py` | `test_payment.py` |
| Test functions | `test_*` | `test_charge_returns_receipt` |
| Test classes | `Test*` (capital T) | `TestPaymentProcessor` |
| Methods in classes | `test_*` | `test_charge_returns_receipt` |

Test names must describe the **behavior under test**, not just mirror the function name.

```python
# bad — tells you nothing
def test_calculate():

# good — names the contract
def test_calculate_returns_zero_for_empty_list():
def test_calculate_raises_value_error_for_negative_input():
```

**Preferred naming style:** `test_<what>_<when>_<expected>` or `test_<verb>_<condition>`.

```python
def test_withdraw_when_balance_insufficient_raises_overdraft_error():
def test_fetch_user_returns_none_for_unknown_id():
```

---

## 2. Test Discovery Configuration

Configure in `pyproject.toml`. For all new projects, use `importlib` import mode — it avoids `sys.path` manipulation and works cleanly with `src/` layouts.

```toml
[tool.pytest.ini_options]
minversion = "7.0"
addopts = [
    "-ra",
    "--strict-markers",
    "--strict-config",
    "--import-mode=importlib",
    "--tb=short",
]
testpaths = ["tests"]
pythonpath = ["src"]
xfail_strict = true
```

### Canonical project layout

```
project/
├── src/
│   └── mypackage/
│       └── core.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── conftest.py
│   │   └── test_core.py
│   └── integration/
│       ├── conftest.py
│       └── test_api.py
├── pyproject.toml
└── README.md
```

Install the package in editable mode so `src/` is importable:

```bash
pip install -e .
```

---

## 3. Assertions

**Always use plain `assert`.** Never use `unittest`-style assertions (`assertEqual`, `assertIn`, `assertRaises`). pytest rewrites `assert` at import time to show the actual values on failure.

```python
# bad
self.assertEqual(result, {"key": "value"})
self.assertIn("foo", my_list)

# good
assert result == {"key": "value"}
assert "foo" in my_list
```

On failure, pytest automatically produces a diff — no helper needed.

### Custom failure messages

Add a message only when it genuinely adds context the diff cannot provide:

```python
assert user.is_active, f"Expected user {user.id!r} to be active, got {user.status!r}"
```

### Floating-point comparisons — always use `pytest.approx`

Never use raw `==` with floats.

```python
# bad
assert 0.1 + 0.2 == 0.3

# good
assert 0.1 + 0.2 == pytest.approx(0.3)

# with explicit tolerance
assert computed == pytest.approx(expected, rel=1e-3)   # 0.1% relative
assert computed == pytest.approx(expected, abs=0.001)  # absolute ±0.001

# works on sequences and dicts
assert [0.1 + 0.2, 0.4 + 0.2] == pytest.approx([0.3, 0.6])
assert {"ratio": 0.1 + 0.2} == pytest.approx({"ratio": 0.3})
```

---

## 4. `pytest.raises`, `pytest.warns`

### `pytest.raises`

Use for every code path that must raise. Never use a bare `try/except` in a test.

```python
# bad
try:
    withdraw(amount=-50)
    assert False, "expected ValueError"
except ValueError:
    pass

# good
with pytest.raises(ValueError):
    withdraw(amount=-50)

# good — assert on message
with pytest.raises(ValueError, match=r"amount must be positive"):
    withdraw(amount=-50)

# good — inspect the exception object
with pytest.raises(HTTPError) as exc_info:
    fetch("/not-found")
assert exc_info.value.status_code == 404
```

Put **only the single line that should raise** inside the `with` block.

```python
# bad — ambiguous which call raises
with pytest.raises(ValueError):
    result = parse(data)
    store(result)
    report(result)

# good
result = parse(data)
with pytest.raises(ValueError):
    store(result)
```

### `pytest.warns`

Use when testing that code emits a specific warning.

```python
def test_deprecated_function_warns():
    with pytest.warns(DeprecationWarning, match="use new_func instead"):
        old_func()
```

---

## 5. Fixtures

### Scopes

Choose the narrowest scope that satisfies the test's needs.

| Scope | Lifetime | Typical use |
|-------|----------|-------------|
| `function` | One per test (default) | Any mutable state |
| `class` | One per `Test*` class | Shared read-only setup |
| `module` | One per `.py` file | Expensive but reusable setup |
| `session` | Entire test run | DB engine, network client |

### Teardown with `yield`

Code before `yield` = setup. Code after `yield` = teardown. Teardown runs even if the test fails.

```python
@pytest.fixture
def tmp_file(tmp_path):
    path = tmp_path / "data.json"
    path.write_text('{"key": "value"}')
    yield path
    path.unlink(missing_ok=True)
```

### Transaction rollback pattern for databases

```python
@pytest.fixture(scope="session")
def engine():
    eng = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(eng)
    yield eng
    Base.metadata.drop_all(eng)

@pytest.fixture
def db(engine):
    """Each test gets a transaction that rolls back on teardown."""
    connection = engine.connect()
    transaction = connection.begin()
    yield connection
    transaction.rollback()
    connection.close()
```

### Fixture factories

When a test needs multiple independent instances:

```python
@pytest.fixture
def make_user():
    created = []

    def _make(name, role="reader"):
        user = User(name=name, role=role)
        db.session.add(user)
        created.append(user)
        return user

    yield _make

    for user in created:
        db.session.delete(user)
    db.session.commit()
```

### `autouse` — use sparingly

Reserve `autouse=True` for infrastructure concerns invisible to the test body (resetting global state, disabling real network calls). If a fixture changes observable behavior, make it an explicit parameter.

```python
# good use of autouse — infrastructure, not behavior
@pytest.fixture(autouse=True)
def no_real_http(monkeypatch):
    """Block accidental real HTTP calls in unit tests."""
    monkeypatch.setattr("httpx.get", lambda *a, **kw: (_ for _ in ()).throw(
        RuntimeError("Real HTTP calls are forbidden in tests")
    ))
```

---

## 6. `conftest.py`

### What belongs here

**Yes:** fixtures shared across multiple test files, pytest hook implementations, `pytest_plugins` declarations, global `autouse` fixtures.

**No:** test functions, fixtures used by only one file (keep those in the test file), business logic, utility functions (put those in `tests/helpers/`).

### Structure

```python
# tests/conftest.py

import pytest
from mypackage.app import create_app
from mypackage.db import get_engine, Base

# Enable assertion rewriting for shared helper modules
pytest.register_assert_rewrite("tests.helpers.assertions")

@pytest.fixture(scope="session")
def engine():
    eng = get_engine("sqlite:///:memory:")
    Base.metadata.create_all(eng)
    yield eng
    Base.metadata.drop_all(eng)

@pytest.fixture
def db(engine):
    connection = engine.connect()
    transaction = connection.begin()
    yield connection
    transaction.rollback()
    connection.close()

@pytest.fixture
def app():
    return create_app(testing=True)

@pytest.fixture
def client(app):
    return app.test_client()
```

### Fixture lookup order

pytest searches for a fixture named `foo` in this order:

1. The test module itself
2. `conftest.py` files from the test's directory upward to `rootdir`
3. Plugin-provided fixtures (lowest priority)

Tests can only search **upward** — a test in `tests/unit/` cannot use a fixture defined only in `tests/integration/conftest.py`.

---

## 7. Test Organization: Functions vs. Classes

**Prefer standalone functions.** pytest was designed to replace class-based unittest boilerplate.

Use `Test*` classes only when:
- Grouping closely related tests for a single subsystem makes navigation easier
- You need class-scoped fixtures shared across several methods
- You want to apply a `pytestmark` to a logical group

```python
# good — class groups checkout-specific tests and shares a fixture
@pytest.mark.integration
class TestOrderCheckout:
    @pytest.fixture(autouse=True)
    def setup(self, db):
        self.cart = Cart(db)
        self.cart.add_item("widget", qty=2)

    def test_total_reflects_quantity(self):
        assert self.cart.total() == 20.0

    def test_checkout_creates_pending_order(self):
        order = self.cart.checkout(user_id=1)
        assert order.status == "pending"
```

**Rules for `Test*` classes:**
- No `__init__` method — pytest will not collect the class if it has one.
- Do not inherit from `unittest.TestCase` unless you specifically need `setUp`/`tearDown` lifecycle.

---

## 8. Markers

### Register all custom markers

```toml
# pyproject.toml
[tool.pytest.ini_options]
markers = [
    "slow: tests that take more than ~1s",
    "integration: tests requiring external services (db, network)",
    "unit: pure unit tests with no I/O",
    "smoke: minimal passing criteria for a build",
]
```

Always run with `--strict-markers` (included in the `addopts` above). A typo in a marker name becomes a hard error, not a silent no-op.

### Usage

```python
@pytest.mark.slow
@pytest.mark.integration
def test_bulk_import_10k_records(db):
    ...
```

Apply a marker to every test in a file at module level:

```python
# tests/integration/test_payments.py
pytestmark = [pytest.mark.integration, pytest.mark.slow]
```

Run by marker expression:

```bash
pytest -m "not slow"
pytest -m "integration and not slow"
pytest -m "smoke"
```

---

## 9. Parametrize

### Basic form

```python
@pytest.mark.parametrize("value, expected", [
    (1,   True),
    (0,   False),
    (-1,  False),
    (100, True),
])
def test_is_positive(value, expected):
    assert is_positive(value) == expected
```

### Use explicit `ids` for readable output

```python
@pytest.mark.parametrize("n, label", [
    (0,   "zero"),
    (1,   "one"),
    (999, "many"),
], ids=["zero", "one", "large"])
def test_describe_number(n, label):
    assert describe(n) == label
```

### Mark individual parameter sets

```python
@pytest.mark.parametrize("x, y, expected", [
    (2, 3, 5),
    pytest.param(0, 0, 0, id="both-zero"),
    pytest.param(
        None, 1, None,
        marks=pytest.mark.xfail(reason="None input not yet handled"),
    ),
])
def test_add(x, y, expected):
    assert add(x, y) == expected
```

### Dataclass pattern for complex cases

Avoids positional tuple confusion when each case has many fields.

```python
from dataclasses import dataclass, field

@dataclass
class AddCase:
    a: int
    b: int
    expected: int
    id: str = field(default="")

cases = [
    AddCase(1,  2,  3,  "positive"),
    AddCase(-1, -2, -3, "negative"),
    AddCase(0,  0,  0,  "zeros"),
]

@pytest.mark.parametrize("case", cases, ids=lambda c: c.id)
def test_add(case: AddCase):
    assert add(case.a, case.b) == case.expected
```

### Stacked decorators for combinatorial coverage

```python
@pytest.mark.parametrize("serializer", ["json", "msgpack"])
@pytest.mark.parametrize("compression", [None, "gzip"])
def test_roundtrip(serializer, compression):
    # runs 2 × 2 = 4 combinations
    ...
```

---

## 10. Mocking

### Tool hierarchy

| Tool | Use when |
|------|----------|
| `mocker` (pytest-mock) | Default — auto-cleans up, pytest-native |
| `monkeypatch` (built-in) | Simple attribute / env var / `sys.path` replacement |
| `unittest.mock.patch` (decorator/context manager) | Only when `mocker` is unavailable |

### Cardinal rule: patch where the name is used, not where it is defined

```python
# mypackage/service.py
import requests

def fetch_data(url):
    return requests.get(url).json()
```

```python
# bad — patches the source, not what service.py sees
mocker.patch("requests.get", return_value=mock_response)

# good — patches the name bound in service.py's namespace
mocker.patch("mypackage.service.requests.get", return_value=mock_response)
```

### Always use `autospec=True`

```python
# bad — accepts any signature, masks bugs
mock_fn = mocker.patch("mypackage.core.calculate")

# good — enforces the real signature; wrong call raises TypeError
mock_fn = mocker.patch("mypackage.core.calculate", autospec=True)
```

### Common patterns

```python
# Patch a function and assert on calls
def test_send_email(mocker):
    mock_send = mocker.patch("mypackage.email.smtp_send", autospec=True)
    send_welcome_email("user@example.com")
    mock_send.assert_called_once_with(to="user@example.com", subject="Welcome!")

# Patch a method on an object in scope
def test_fetch_user(mocker, db):
    mocker.patch.object(db, "query", return_value=fake_user, autospec=True)
    user = get_user(db, user_id=1)
    assert user.name == "Alice"

# Spy — track calls without replacing behavior
def test_cache_hit(mocker):
    spy = mocker.spy(cache, "get")
    fetch_user(1)
    fetch_user(1)
    assert spy.call_count == 2

# monkeypatch for env vars and attributes
def test_reads_env(monkeypatch):
    monkeypatch.setenv("DATABASE_URL", "sqlite:///:memory:")
    assert get_db_url() == "sqlite:///:memory:"
```

### Common mock assertions

```python
mock.assert_called_once()
mock.assert_called_once_with(arg, kwarg=val)
mock.assert_called_with(arg)          # most recent call only
mock.assert_any_call(arg)             # any call matches
mock.assert_has_calls([call(1), call(2)], any_order=False)
mock.assert_not_called()
assert mock.call_count == 3
```

---

## 11. Test Isolation

Every test must pass in any order and in isolation. Violations produce flaky tests.

**Isolation checklist:**

| Concern | Correct approach |
|---------|-----------------|
| Mutable global state | Use `function`-scoped fixtures; never module-level mutable variables |
| Database changes | Roll back in teardown; never commit in a test |
| Filesystem | Use `tmp_path` (built-in fixture) |
| Time | Freeze with `freezegun` or `monkeypatch` |
| Network | Block with `autouse` fixture; use `pytest-httpx` or `responses` for HTTP |
| Environment variables | Use `monkeypatch.setenv()` — reverts automatically |
| Order dependence | Run `pytest-randomly` in CI to detect and fix |

```bash
# Detect isolation violations by randomising test order
pip install pytest-randomly
pytest -p randomly
```

---

## 12. `skip` and `xfail`

### Decision matrix

| Situation | Use |
|-----------|-----|
| Missing optional dependency | `pytest.importorskip("numpy")` |
| Platform-specific test | `@pytest.mark.skipif(sys.platform == "win32", reason=...)` |
| Known bug tracked in issue | `@pytest.mark.xfail(reason="GH-NNN: ...")` |
| Feature not yet implemented | `@pytest.mark.xfail(raises=NotImplementedError)` |
| Temporarily broken, must fix | `@pytest.mark.xfail(strict=True)` |
| Crashes the interpreter | `@pytest.mark.xfail(run=False)` |

### `skip` examples

```python
# unconditional
@pytest.mark.skip(reason="endpoint not yet deployed")
def test_new_endpoint(): ...

# conditional
@pytest.mark.skipif(sys.platform == "win32", reason="POSIX only")
def test_symlinks(): ...

# runtime check inside the test
def test_optional():
    pytest.importorskip("numpy", minversion="1.20")
    ...

# skip an entire module
pytestmark = pytest.mark.skipif(
    sys.version_info < (3, 11),
    reason="ExceptionGroup requires Python 3.11+",
)
```

### `xfail` examples

```python
# known bug
@pytest.mark.xfail(reason="GH-1234: parser fails on unicode input")
def test_unicode_parsing(): ...

# specific expected exception
@pytest.mark.xfail(raises=NotImplementedError)
def test_unimplemented_feature(): ...

# strict: XPASS (unexpected pass) becomes a test failure
@pytest.mark.xfail(strict=True, reason="must fix before v2.0")
def test_known_regression(): ...
```

Enable `xfail_strict = true` globally in `pyproject.toml` (already in the template above) so that any test that starts passing forces removal of the marker.

---

## 13. Anti-Patterns to Avoid

### Testing implementation, not behavior

```python
# bad — breaks on refactor even when behavior is correct
def test_calls_internal_helper(mocker):
    spy = mocker.spy(service, "_internal_helper")
    service.do_thing()
    spy.assert_called_once()

# good — tests the observable output
def test_do_thing_returns_processed_result():
    assert service.do_thing() == expected_output
```

### Shared mutable state between tests

```python
# bad — order-dependent
items = []

def test_append():
    items.append(1)
    assert len(items) == 1

# good — each test gets a fresh list via fixture
@pytest.fixture
def items():
    return []
```

### Testing the mock, not the code

```python
# bad — proves nothing about real behavior; just echoes mock return value
def test_payment(mocker):
    mocker.patch("stripe.charge", return_value={"id": "ch_123"})
    result = process_payment(100)
    assert result == {"id": "ch_123"}
```

### Overly broad `pytest.raises`

```python
# bad — catches any exception, including bugs you didn't intend
with pytest.raises(Exception):
    do_thing()

# good
with pytest.raises(ValueError, match="amount must be positive"):
    do_thing()
```

### Asserting on auto-generated IDs

```python
# bad — environment-dependent
assert record.id == 1

# good — assert the invariant
assert record.id is not None
assert isinstance(record.id, int)
```

### Putting tests in `conftest.py`

`conftest.py` is for fixtures and hooks only. pytest does not collect test functions from it.

### Duplicating type annotations in parametrize tuples

For large test tables, use the dataclass pattern (section 9) instead of positional tuples with many fields.

---

## 14. Coverage

```toml
[tool.coverage.run]
source = ["src/mypackage"]
branch = true
omit = ["tests/*", "**/__main__.py"]

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
    "@(abc\\.)?abstractmethod",
    "\\.\\.\\.",
]
```

Run:

```bash
pytest --cov=src/mypackage --cov-report=term-missing --cov-branch
```

Use `# pragma: no cover` sparingly and only for code that is genuinely untestable (interactive helpers, platform-specific branches verified by CI on the other platform).

High line coverage is necessary but not sufficient. Enable branch coverage (`branch = true`) and focus tests on **behaviors**, not lines.

---

## Reference

- [pytest — Good Integration Practices](https://docs.pytest.org/en/stable/explanation/goodpractices.html)
- [pytest — How to Use Fixtures](https://docs.pytest.org/en/stable/how-to/fixtures.html)
- [pytest — How to Parametrize](https://docs.pytest.org/en/stable/how-to/parametrize.html)
- [pytest — Skip and xfail](https://docs.pytest.org/en/stable/how-to/skipping.html)
- [pytest — Assertions](https://docs.pytest.org/en/stable/how-to/assert.html)
- [pytest — Markers](https://docs.pytest.org/en/stable/how-to/mark.html)
- [pytest-mock — Usage](https://pytest-mock.readthedocs.io/en/latest/usage.html)
- [pytest-cov — Configuration](https://pytest-cov.readthedocs.io/en/latest/config.html)
