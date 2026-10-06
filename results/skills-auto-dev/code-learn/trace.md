### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by reading the skill files that apply to this task.

### Tool call: read_file
{"file_path": "skills/deliver-all-artifacts/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "skills/verify-output-contract/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "skills/recover-from-tool-failure/SKILL.md", "limit": 1000}

### Tool result
@@ lines 1-12 of 12 @@
---
name: deliver-all-artifacts
description: When a task lists multiple deliverables such as code fixes, tests, changelog entries, and output data files.
---
- At the start, enumerate every required deliverable and its exact format.
- Apply all stated coding standards (type annotations, docstrings, public API rules) to the relevant functions.
- Add new test files for fixes; do not modify provided tests or input data unless explicitly allowed.
- For each fix, add a changelog entry using the exact required pattern, bullets, and headings.
- Create every required output file, and include every required field or block.
- If any required artifact is missing, stop and create it before finishing.
- Re-read the task requirements and compare them to the final files.
- Prefer adding new files over editing protected or provided files.

### Tool result
@@ lines 1-12 of 12 @@
---
name: verify-output-contract
description: When a task specifies an output file, JSON schema, data format, or required fields.
---
- Before writing, list every explicit output requirement: filename, top-level keys, required subfields, units, ordering, normalization rules, and header/version values.
- Write output that matches exactly; do not substitute your own schema, field names, or units.
- If integer units are required, convert floating values before serializing.
- Normalize text (case, separators, whitespace) and timestamps to the required format before sorting.
- Sort collections by every specified key in the specified direction.
- Include required metadata/header blocks even if they feel redundant.
- After writing, re-open the output and check each checklist item one by one.
- Do not omit a required file or field because another part of the task is done.

### Tool result
@@ lines 1-10 of 10 @@
---
name: recover-from-tool-failure
description: When a sandbox command or tool fails repeatedly and blocks progress.
---
- After one or two identical failures, stop retrying the broken tool.
- Identify fallback tools such as read, glob, grep, write, or manual/static analysis.
- Do not spend turns exploring the filesystem for files the prompt never mentions.
- Continue with available tools to produce the required outputs.
- Verify outputs by reading them back and checking against the task spec.
- If a limitation remains, state it in the final summary but still deliver best-effort complete output.

### Tool call: ls
{"path": "workspace"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool call: ls
{"path": "workspace/inventory"}

### Tool call: ls
{"path": "workspace/tests"}

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/__pycache__/', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py']

### Tool result
['/workspace/tests/__pycache__/', '/workspace/tests/test_report.py']

### Tool call: read_file
{"file_path": "workspace/inventory/__init__.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "workspace/tests/test_report.py"}

### Tool call: read_file
{"file_path": "workspace/README.md"}

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md"}

### Tool result
@@ lines 1-1 @@
System reminder: File exists but has empty contents

### Tool result
@@ lines 1-12 of 12 @@
"""Export helpers."""
from .pricing import parse_price


def to_csv_row(item):
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    price = parse_price(item["price"])
    return f'{item["name"]},{price:.2f},{item["qty"]}'

### Tool result
@@ lines 1-25 of 25 @@
"""Price utilities for the inventory package."""
from decimal import Decimal


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "")
    return Decimal(cleaned)


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"))

### Tool result
@@ lines 1-25 of 25 @@
"""Inventory reports."""
from decimal import Decimal

from .pricing import apply_discount, parse_price


def total_value(items):
    """Total value of `items` after per-item discounts.

    Each item is a dict: {"name": str, "price": str, "qty": int, "discount": percent (optional)}.
    """
    total = Decimal("0")
    for item in items:
        price = parse_price(item["price"])
        price = apply_discount(price, item.get("discount", 0))
        total += price * item["qty"]
    return total


def low_stock(items, threshold=5):
    """Names of the items whose quantity is strictly below `threshold`.

    The names are sorted alphabetically, ignoring case.
    """
    return [item["name"] for item in items if item["qty"] <= threshold]

### Tool result
@@ lines 1-32 of 32 @@
from decimal import Decimal

from inventory.export import to_csv_row
from inventory.pricing import apply_discount
from inventory.report import low_stock, total_value


def test_total_value_plain():
    items = [{"name": "pen", "price": "$2.50", "qty": 4}]
    assert total_value(items) == Decimal("10.00")


def test_total_value_thousands_separator():
    items = [{"name": "laptop", "price": "$1,299.50", "qty": 2}]
    assert total_value(items) == Decimal("2599.00")


def test_discount_rounding_half_up():
    assert apply_discount(Decimal("10.05"), 10) == Decimal("9.05")


def test_total_value_with_discount():
    items = [{"name": "desk", "price": "100.00", "qty": 1, "discount": 15}]
    assert total_value(items) == Decimal("85.00")


def test_low_stock_basic():
    assert low_stock([{"name": "pen", "qty": 2}, {"name": "ink", "qty": 9}]) == ["pen"]


def test_csv_row_plain_name():
    assert to_csv_row({"name": "pen", "price": "2.5", "qty": 4}) == "pen,2.50,4"

### Tool result
@@ lines 1-5 of 5 @@
# inventory

Small package. Run the tests from the `workspace/` folder:

    cd workspace && python -m pytest tests -q

### Tool result
@@ lines 1-6 of 6 @@
# Changelog

## Unreleased

## 0.3.1
- initial import

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -40"}

### Tool call: glob
{"pattern": "**/*.md"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool result
['/skills/deliver-all-artifacts/SKILL.md', '/skills/recover-from-tool-failure/SKILL.md', '/skills/verify-output-contract/SKILL.md', '/workspace/CHANGELOG.md', '/workspace/README.md']

### Tool call: execute
{"command": "python3 --version"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: ls
{"path": "/"}

### Tool result
['/skills/', '/workspace/']

### Tool call: glob
{"pattern": "**/__pycache__/*"}

### Tool result
['/workspace/inventory/__pycache__/__init__.cpython-312.pyc', '/workspace/inventory/__pycache__/export.cpython-312.pyc', '/workspace/inventory/__pycache__/pricing.cpython-312.pyc', '/workspace/inventory/__pycache__/report.cpython-312.pyc', '/workspace/tests/__pycache__/test_report.cpython-312-pytest-9.1.1.pyc']

### Assistant
The shell is failing with `runuser: not found`. Let me probe once with a non-Python command while I start the static analysis, and read the sources I'll change.

### Tool call: execute
{"command": "echo hi"}

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool result
@@ lines 1-25 of 25 @@
"""Price utilities for the inventory package."""
from decimal import Decimal


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "")
    return Decimal(cleaned)


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"))

### Assistant
The sandbox shell is completely non-functional (even `echo hi` fails), so I'll use the fallback approach: static analysis, careful fixes, and manual verification by re-reading. I identified four defects against the docstrings:

1. `parse_price` ignores thousands separators (`$1,299.50` raises) and accounting parentheses.
2. `apply_discount` uses the default banker's rounding instead of "round half up".
3. `low_stock` uses `<=` instead of "strictly below", and never sorts.
4. `to_csv_row` never applies the RFC 4180 quoting it documents.

### Tool call: write_file
{"file_path": "workspace/inventory/pricing.py", "content": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import ROUND_HALF_UP, Decimal\n\n\ndef parse_price(text: str) -> Decimal:\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\").replace(\",\", \"\")\n    if cleaned.startswith(\"(\") and cleaned.endswith(\")\"):\n        return -Decimal(cleaned[1:-1].strip())\n    return Decimal(cleaned)\n\n\ndef apply_discount(price: Decimal, percent: int) -> Decimal:\n    \"\"\"Return `price` after a percentage discount.\n\n    The result is a Decimal rounded to 2 decimal places using the\n    commercial rule \"round half up\" (0.005 -> 0.01).\n    \"\"\"\n    discounted = price * (100 - percent) / 100\n    return discounted.quantize(Decimal(\"0.01\"), rounding=ROUND_HALF_UP)\n"}

### Tool call: write_file
{"file_path": "workspace/inventory/report.py", "content": "\"\"\"Inventory reports.\"\"\"\nfrom __future__ import annotations\n\nfrom decimal import Decimal\n\nfrom .pricing import apply_discount, parse_price\n\n\ndef total_value(items: list[dict]) -> Decimal:\n    \"\"\"Total value of `items` after per-item discounts.\n\n    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n    \"\"\"\n    total = Decimal(\"0\")\n    for item in items:\n        price = parse_price(item[\"price\"])\n        price = apply_discount(price, item.get(\"discount\", 0))\n        total += price * item[\"qty\"]\n    return total\n\n\ndef low_stock(items: list[dict], threshold: int = 5) -> list[str]:\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    names = [item[\"name\"] for item in items if item[\"qty\"] < threshold]\n    return sorted(names, key=str.casefold)\n"}

### Tool call: write_file
{"file_path": "workspace/inventory/export.py", "content": "\"\"\"Export helpers.\"\"\"\nfrom __future__ import annotations\n\nfrom .pricing import parse_price\n\n\ndef to_csv_row(item: dict) -> str:\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    price = parse_price(item[\"price\"])\n    return f\"{name},{price:.2f},{item['qty']}\"\n"}

### Tool call: write_file
{"file_path": "workspace/inventory/__init__.py", "content": "\"\"\"Inventory package: price parsing, reports and CSV export helpers.\"\"\"\nfrom .export import to_csv_row\nfrom .pricing import apply_discount, parse_price\nfrom .report import low_stock, total_value\n\n__all__ = [\n    \"apply_discount\",\n    \"low_stock\",\n    \"parse_price\",\n    \"to_csv_row\",\n    \"total_value\",\n]\n"}

### Tool result
Updated file /workspace/inventory/pricing.py

### Tool result
Updated file /workspace/inventory/report.py

### Tool result
Updated file /workspace/inventory/export.py

### Tool result
Updated file /workspace/inventory/__init__.py

### Assistant
Now the changelog entries (per the deliver-all-artifacts skill) and a new test file covering the docstring specs. My new tests only assert behaviour documented in the docstrings, and I checked each expected value by hand:

### Tool call: write_file
{"content": "# Changelog\n\n## Unreleased\n- Fix `parse_price` so it strips thousands separators (`$1,299.50`) and treats\n  accounting-style parentheses (`(12.00)`) as a negative amount.\n- Fix `apply_discount` to round half up (`ROUND_HALF_UP`) instead of the decimal\n  context default, so `Decimal(\"10.05\")` with a 10 percent discount yields `9.05`.\n- Fix `low_stock` to select quantities strictly below the threshold and to sort the\n  names alphabetically while ignoring case.\n- Fix `to_csv_row` to quote names containing a comma or a double quote and to double\n  the embedded double quotes (RFC 4180).\n- Add `tests/test_fixes.py` covering the documented behaviour of the fixed helpers.\n\n## 0.3.1\n- initial import\n", "file_path": "workspace/CHANGELOG.md"}

### Tool call: write_file
{"content": "\"\"\"Regression tests for behaviour documented in the package docstrings.\"\"\"\nfrom decimal import Decimal\n\nfrom inventory.export import to_csv_row\nfrom inventory.pricing import apply_discount, parse_price\nfrom inventory.report import low_stock, total_value\n\n\ndef test_parse_price_thousands_separator():\n    assert parse_price(\"$1,299.50\") == Decimal(\"1299.50\")\n\n\ndef test_parse_price_surrounding_whitespace():\n    assert parse_price(\"  12 \") == Decimal(\"12\")\n\n\ndef test_parse_price_plain_decimal():\n    assert parse_price(\"1299.5\") == Decimal(\"1299.5\")\n\n\ndef test_parse_price_accounting_parentheses_are_negative():\n    assert parse_price(\"(12.00)\") == Decimal(\"-12.00\")\n\n\ndef test_apply_discount_rounds_half_up():\n    assert apply_discount(Decimal(\"10.05\"), 10) == Decimal(\"9.05\")\n\n\ndef test_apply_discount_rounds_down_below_half():\n    assert apply_discount(Decimal(\"10.04\"), 10) == Decimal(\"9.04\")\n\n\ndef test_apply_discount_zero_percent_keeps_value():\n    assert apply_discount(Decimal(\"12.34\"), 0) == Decimal(\"12.34\")\n\n\ndef test_total_value_with_accounting_negative_price():\n    items = [{\"name\": \"refund\", \"price\": \"(5.00)\", \"qty\": 2}]\n    assert total_value(items) == Decimal(\"-10.00\")\n\n\ndef test_low_stock_below_threshold_is_strict():\n    items = [{\"name\": \"pen\", \"qty\": 5}, {\"name\": \"ink\", \"qty\": 4}]\n    assert low_stock(items, threshold=5) == [\"ink\"]\n\n\ndef test_low_stock_sor

### Tool result
Updated file /workspace/CHANGELOG.md

### Tool result
Updated file /workspace/tests/test_fixes.py

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/__init__.py"}

### Tool result
@@ lines 1-27 of 27 @@
"""Price utilities for the inventory package."""
from decimal import ROUND_HALF_UP, Decimal


def parse_price(text: str) -> Decimal:
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "").replace(",", "")
    if cleaned.startswith("(") and cleaned.endswith(")"):
        return -Decimal(cleaned[1:-1].strip())
    return Decimal(cleaned)


def apply_discount(price: Decimal, percent: int) -> Decimal:
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)

### Tool result
@@ lines 1-28 of 28 @@
"""Inventory reports."""
from __future__ import annotations

from decimal import Decimal

from .pricing import apply_discount, parse_price


def total_value(items: list[dict]) -> Decimal:
    """Total value of `items` after per-item discounts.

    Each item is a dict: {"name": str, "price": str, "qty": int, "discount": percent (optional)}.
    """
    total = Decimal("0")
    for item in items:
        price = parse_price(item["price"])
        price = apply_discount(price, item.get("discount", 0))
        total += price * item["qty"]
    return total


def low_stock(items: list[dict], threshold: int = 5) -> list[str]:
    """Names of the items whose quantity is strictly below `threshold`.

    The names are sorted alphabetically, ignoring case.
    """
    names = [item["name"] for item in items if item["qty"] < threshold]
    return sorted(names, key=str.casefold)

### Tool result
@@ lines 1-17 of 17 @@
"""Export helpers."""
from __future__ import annotations

from .pricing import parse_price


def to_csv_row(item: dict) -> str:
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    name = item["name"]
    if "," in name or '"' in name:
        name = '"' + name.replace('"', '""') + '"'
    price = parse_price(item["price"])
    return f"{name},{price:.2f},{item['qty']}"

### Tool result
@@ lines 1-12 of 12 @@
"""Inventory package: price parsing, reports and CSV export helpers."""
from .export import to_csv_row
from .pricing import apply_discount, parse_price
from .report import low_stock, total_value

__all__ = [
    "apply_discount",
    "low_stock",
    "parse_price",
    "to_csv_row",
    "total_value",
]

### Assistant
Files are written. Since my shell is dead, let me try once to get the suite executed through a subagent (it may have a working runner) — verification is the core of this task.

### Tool call: task
{"description": "Run the test suite in the sandbox and report raw output.\n\nWorking directory contents: the sandbox root contains `skills/` and `workspace/`. The `workspace/` folder has `inventory/` (package: __init__.py, pricing.py, report.py, export.py), `tests/` (test_report.py and test_fixes.py), README.md, CHANGELOG.md.\n\nDo exactly this:\n1. Use the execute tool with the command: `cd workspace && python -m pytest tests -q 2>&1 | tail -40`\n2. If that fails with an error like \"/bin/sh: 1: runuser: not found\", try these alternatives in order and report which one worked:\n   - `python3 -m pytest workspace/tests -q`\n   - `pytest workspace/tests -q`\n   - `python -c \"import sys; print(sys.version)\"`\n   - `sh -c 'echo test'`\n3. If ALL commands fail, say clearly \"ALL SHELL COMMANDS FAILED\" and report the exact error text.\n\nReport back: the exact command(s) you tried, the exact stdout/stderr, and if pytest ran, the list of PASSED/FAILED tests and full failure tracebacks if any. Do not modify any files.", "subagent_type": "general-purpose"}

### Tool result
## ALL SHELL COMMANDS FAILED

Every shell command — including all five specified alternatives — failed with the identical error before any of them could execute. **No test results were produced.** I did not modify any files.

---

### 1. Exact commands tried, in order, with raw output

| # | Command | stdout | stderr | Exit code |
|---|---------|--------|--------|-----------|
| 1 | `cd workspace && python -m pytest tests -q 2>&1 \| tail -40` | *(empty)* | `/bin/sh: 1: runuser: not found` | `127` |
| 2 | `python3 -m pytest workspace/tests -q` | *(empty)* | `/bin/sh: 1: runuser: not found` | `127` |
| 3 | `pytest workspace/tests -q` | *(empty)* | `/bin/sh: 1: runuser: not found` | `127` |
| 4 | `python -c "import sys; print(sys.version)"` | *(empty)* | `/bin/sh: 1: runuser: not found` | `127` |
| 5 | `sh -c 'echo test'` | *(empty)* | `/bin/sh: 1: runuser: not found` | `127` |

**Exact error text (verbatim, all five commands):**

```
/bin/sh: 1: runuser: not found
```

```
Exit code: 127
[Command failed with exit code 127]
```

**Which alternative "worked": none. All five failed identically.**

### Root cause

The sandbox's command wrapper unconditionally invokes `runuser` to drop privileges *before* handing the command to `/bin/sh`. `runuser` is not installed in this container, so the shell exits `127` immediately. This is not a pytest problem, a path problem, or a problem with any of my commands — commands #4 (`python -c "import sys; print(sys.version)"`) and #5 (`sh -c 'echo 

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md"}

### Tool call: read_file
{"file_path": "workspace/tests/test_fixes.py"}

### Tool result
@@ lines 1-15 of 15 @@
# Changelog

## Unreleased
- Fix `parse_price` so it strips thousands separators (`$1,299.50`) and treats
  accounting-style parentheses (`(12.00)`) as a negative amount.
- Fix `apply_discount` to round half up (`ROUND_HALF_UP`) instead of the decimal
  context default, so `Decimal("10.05")` with a 10 percent discount yields `9.05`.
- Fix `low_stock` to select quantities strictly below the threshold and to sort the
  names alphabetically while ignoring case.
- Fix `to_csv_row` to quote names containing a comma or a double quote and to double
  the embedded double quotes (RFC 4180).
- Add `tests/test_fixes.py` covering the documented behaviour of the fixed helpers.

## 0.3.1
- initial import

### Tool result
@@ lines 1-68 of 68 @@
"""Regression tests for behaviour documented in the package docstrings."""
from decimal import Decimal

from inventory.export import to_csv_row
from inventory.pricing import apply_discount, parse_price
from inventory.report import low_stock, total_value


def test_parse_price_thousands_separator():
    assert parse_price("$1,299.50") == Decimal("1299.50")


def test_parse_price_surrounding_whitespace():
    assert parse_price("  12 ") == Decimal("12")


def test_parse_price_plain_decimal():
    assert parse_price("1299.5") == Decimal("1299.5")


def test_parse_price_accounting_parentheses_are_negative():
    assert parse_price("(12.00)") == Decimal("-12.00")


def test_apply_discount_rounds_half_up():
    assert apply_discount(Decimal("10.05"), 10) == Decimal("9.05")


def test_apply_discount_rounds_down_below_half():
    assert apply_discount(Decimal("10.04"), 10) == Decimal("9.04")


def test_apply_discount_zero_percent_keeps_value():
    assert apply_discount(Decimal("12.34"), 0) == Decimal("12.34")


def test_total_value_with_accounting_negative_price():
    items = [{"name": "refund", "price": "(5.00)", "qty": 2}]
    assert total_value(items) == Decimal("-10.00")


def test_low_stock_below_threshold_is_strict():
    items = [{"name": "pen", "qty": 5}, {"name": "ink", "qty": 4}]
    assert low_stock(items, threshold=5) == ["ink"]


def test_low_stock_sorts_names_ignoring_case():
    items = [
        {"name": "Pen", "qty": 1},
        {"name": "app

### Assistant
All artifacts are in place. Summary below.

## What I changed and why

**`workspace/inventory/pricing.py`**
- `parse_price`: now strips thousands separators (`.replace(",", "")`) so `"$1,299.50"` → `Decimal("1299.50")` instead of raising `InvalidOperation`, and handles accounting-style parentheses so `"(12.00)"` → `Decimal("-12.00")` (previously it raised).
- `apply_discount`: quantize with `rounding=ROUND_HALF_UP`. The default context uses banker's rounding, so `Decimal("10.05")` at 10% gave `9.04` instead of the documented commercial `9.05`.

**`workspace/inventory/report.py`**
- `low_stock`: uses strict `< threshold` (was `<=`), and sorts the names with `key=str.casefold` ("sorted alphabetically, ignoring case") — it previously returned them unsorted.
- `total_value`: unchanged in behaviour (it was correct); now carries type annotations and delegates to the fixed `parse_price`/`apply_discount`. `total_value_plain`, `_thousands_separator` and `_with_discount` now evaluate correctly.

**`workspace/inventory/export.py`**
- `to_csv_row`: implements the RFC 4180 quoting its docstring specifies — a name containing a comma or a double quote is wrapped in quotes and embedded quotes are doubled (`'6" ruler'` → `"6"" ruler"`). Previously no quoting was applied.

**`workspace/inventory/__init__.py`** — added a module docstring and re-exported the public helpers with `__all__`, so the package exposes a single public API surface (the module was empty).

**`workspace/tests/test_fixes.py