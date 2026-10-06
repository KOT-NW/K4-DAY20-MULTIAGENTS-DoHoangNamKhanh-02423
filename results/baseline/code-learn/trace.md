### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by exploring the workspace structure.

### Tool call: ls
{"path": "workspace"}

### Tool call: glob
{"pattern": "**/*.py"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py', '/workspace/tests/test_report.py']

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
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -30 && python --version"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "cd workspace && python3 -m pytest tests -q 2>&1 | tail -30; python3 --version"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "echo hi; pwd"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "python3 -c \"print(1+1)\""}

### Tool call: execute
{"command": "/bin/ls workspace"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "bash -lc \"echo ok\"", "timeout": 30}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: ls
{"path": "."}

### Tool call: glob
{"pattern": "*", "path": "/"}

### Tool result
['/workspace/']

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/__init__.py', '/workspace/inventory/__pycache__/__init__.cpython-312.pyc', '/workspace/inventory/__pycache__/export.cpython-312.pyc', '/workspace/inventory/__pycache__/pricing.cpython-312.pyc', '/workspace/inventory/__pycache__/report.cpython-312.pyc', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py', '/workspace/tests/__pycache__/test_report.cpython-312-pytest-9.1.1.pyc', '/workspace/tests/test_report.py']

### Assistant
The shell in this sandbox appears broken (`runuser` missing). Let me try again before falling back to reasoning.

### Tool call: execute
{"command": "whoami"}

### Tool call: execute
{"command": "cd workspace && python3 -m pytest tests -q"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Assistant
Shell execution is unavailable, so I'll make the fixes carefully from the docstrings (the spec) and verify logic by hand.

### Tool call: write_file
{"file_path": "workspace/inventory/pricing.py", "content": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import ROUND_HALF_UP, Decimal\n\n\ndef parse_price(text):\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\").replace(\",\", \"\")\n    if cleaned.startswith(\"(\") and cleaned.endswith(\")\"):\n        return -Decimal(cleaned[1:-1])\n    return Decimal(cleaned)\n\n\ndef apply_discount(price, percent):\n    \"\"\"Return `price` after a percentage discount.\n\n    The result is a Decimal rounded to 2 decimal places using the\n    commercial rule \"round half up\" (0.005 -> 0.01).\n    \"\"\"\n    discounted = price * (100 - percent) / 100\n    return discounted.quantize(Decimal(\"0.01\"), rounding=ROUND_HALF_UP)\n"}

### Tool call: write_file
{"file_path": "workspace/inventory/report.py", "content": "\"\"\"Inventory reports.\"\"\"\nfrom decimal import Decimal\n\nfrom .pricing import apply_discount, parse_price\n\n\ndef total_value(items):\n    \"\"\"Total value of `items` after per-item discounts.\n\n    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n    \"\"\"\n    total = Decimal(\"0\")\n    for item in items:\n        price = parse_price(item[\"price\"])\n        price = apply_discount(price, item.get(\"discount\", 0))\n        total += price * item[\"qty\"]\n    return total\n\n\ndef low_stock(items, threshold=5):\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    return sorted(\n        (item[\"name\"] for item in items if item[\"qty\"] < threshold),\n        key=str.casefold,\n    )\n"}

### Tool call: write_file
{"file_path": "workspace/inventory/export.py", "content": "\"\"\"Export helpers.\"\"\"\nfrom .pricing import parse_price\n\n\ndef to_csv_row(item):\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    return f'{name},{price:.2f},{item[\"qty\"]}'\n"}

### Tool result
Updated file /workspace/inventory/pricing.py

### Tool result
Updated file /workspace/inventory/report.py

### Tool result
Updated file /workspace/inventory/export.py

### Tool call: edit_file
{"file_path": "workspace/inventory/export.py", "old_string": "    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    return f'{name},{price:.2f},{item[\"qty\"]}'", "new_string": "    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        escaped = name.replace('\"', '\"\"')\n        name = f'\"{escaped}\"'\n    return f'{name},{price:.2f},{item[\"qty\"]}'"}

### Tool call: write_file
{"content": "\"\"\"Inventory package.\"\"\"\n", "file_path": "workspace/inventory/__init__.py"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/export.py'

### Tool result
Updated file /workspace/inventory/__init__.py

### Tool call: task
{"description": "Verify that the test suite in workspace/ passes. Run exactly: `cd workspace && python3 -m pytest tests -q` (use the execute/shell tool). Also run a couple of extra sanity checks with python3, e.g.:\npython3 -c \"import sys; sys.path.insert(0,'workspace'); from inventory.pricing import parse_price, apply_discount; print(parse_price('(1,299.50)'), apply_discount(__import__('decimal').Decimal('10.05'),10))\"\npython3 -c \"import sys; sys.path.insert(0,'workspace'); from inventory.export import to_csv_row; print(to_csv_row({'name':'Smith, \\\"J\\\"','price':'2.5','qty':4}))\"\npython3 -c \"import sys; sys.path.insert(0,'workspace'); from inventory.report import low_stock; print(low_stock([{'name':'Banana','qty':4},{'name':'apple','qty':5},{'name':'Cherry','qty':1}]))\"\nReport the exact stdout/stderr and exit codes of each command. Do NOT modify any files. If the shell is completely unavailable (e.g. an error about `runuser` not being found), say so explicitly and clearly.", "subagent_type": "general-purpose"}

### Tool result
## Bottom line: the shell is completely unavailable — I could not execute anything

Every command I attempted through the execute/shell tool failed identically, including the trivial control command `echo hello`. This is exactly the `runuser` failure you mentioned, so I am reporting it explicitly rather than guessing at results.

### Exact observed results (verbatim from the tool)

**Command 1** — `cd workspace && python3 -m pytest tests -q`
```
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]
```
stdout: *(empty — none produced)*

**Command 2** — `python3 -c "import sys; sys.path.insert(0,'workspace'); from inventory.pricing import parse_price, apply_discount; print(parse_price('(1,299.50)'), apply_discount(__import__('decimal').Decimal('10.05'),10))"`
```
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]
```
stdout: *(empty — none produced)*

**Command 3** — `python3 -c "import sys; sys.path.insert(0,'workspace'); from inventory.export import to_csv_row; print(to_csv_row({'name':'Smith, \"J\"','price':'2.5','qty':4}))"`
```
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]
```
stdout: *(empty — none produced)*

**Command 4** — `python3 -c "import sys; sys.path.insert(0,'workspace'); from inventory.report import low_stock; print(low_stock([{'name':'Banana','qty':4},{'name':'apple','qty':5},{'name':'Cherry','qty':1}]))"`
```
[stderr] /bin/sh: 1: runus

### Assistant
The shell is confirmed unavailable sandbox-wide, so I verified statically. Let me update the changelog and do a final review of the files.

### Tool call: edit_file
{"file_path": "workspace/CHANGELOG.md", "old_string": "## Unreleased\n", "new_string": "## Unreleased\n- `parse_price` now handles thousands separators (\"$1,299.50\") and accounting-style\n  negatives (\"(12.00)\"), as documented.\n- `apply_discount` rounds half up instead of using the default half-even rounding.\n- `low_stock` now selects items strictly below the threshold and sorts the names\n  case-insensitively.\n- `to_csv_row` quotes and escapes names containing commas or double quotes (RFC 4180).\n"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/CHANGELOG.md'

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md"}

### Tool result
@@ lines 1-27 of 27 @@
"""Price utilities for the inventory package."""
from decimal import ROUND_HALF_UP, Decimal


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "").replace(",", "")
    if cleaned.startswith("(") and cleaned.endswith(")"):
        return -Decimal(cleaned[1:-1])
    return Decimal(cleaned)


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)

### Tool result
@@ lines 1-16 of 16 @@
"""Export helpers."""
from .pricing import parse_price


def to_csv_row(item):
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    price = parse_price(item["price"])
    name = item["name"]
    if "," in name or '"' in name:
        escaped = name.replace('"', '""')
        name = f'"{escaped}"'
    return f'{name},{price:.2f},{item["qty"]}'

### Tool result
@@ lines 1-28 of 28 @@
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
    return sorted(
        (item["name"] for item in items if item["qty"] < threshold),
        key=str.casefold,
    )

### Tool result
@@ lines 1-12 of 12 @@
# Changelog

## Unreleased
- `parse_price` now handles thousands separators ("$1,299.50") and accounting-style
  negatives ("(12.00)"), as documented.
- `apply_discount` rounds half up instead of using the default half-even rounding.
- `low_stock` now selects items strictly below the threshold and sorts the names
  case-insensitively.
- `to_csv_row` quotes and escapes names containing commas or double quotes (RFC 4180).

## 0.3.1
- initial import

### Assistant
Shell execution is unavailable in this sandbox (`/bin/sh: runuser: not found` for every command, even `echo`), so I verified the fixes by careful hand-tracing against each docstring and the visible tests.

## What was wrong and what I changed

**`inventory/pricing.py`**
- `parse_price`: only stripped whitespace and `$`, so `"$1,299.50"` blew up on the comma-separated digits and `"(12.00)"` failed outright. It now removes `,` as well and treats enclosing parentheses as a negative sign (`-Decimal(...)`), matching all four documented formats.
- `apply_discount`: quantized with the default `ROUND_HALF_EVEN` context, so `10.05` minus 10% gave `9.04` instead of `9.05`. Now passes `rounding=ROUND_HALF_UP` (imported from `decimal`), implementing the documented commercial "round half up" rule.

**`inventory/report.py`**
- `low_stock`: used `qty <= threshold` (violating "strictly below") and ignored the sorting requirement. Now filters with `qty < threshold` and returns `sorted(..., key=str.casefold)` so names are alphabetical ignoring case.

**`inventory/export.py`**
- `to_csv_row`: ignored quoting entirely. Names containing a comma or a double quote are now wrapped in double quotes with every inner quote doubled (RFC 4180), per the docstring.

**`inventory/__init__.py`** – added the package docstring so every module carries a docstring, consistent with the rest of the package.

**`CHANGELOG.md`** – recorded the four fixes under the existing `## Unreleased` heading.

`total_value` nee