### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by reading the skill files that could apply, then explore the workspace.

### Tool call: read_file
{"file_path": "/skills/deliver-all-artifacts/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "/skills/recover-from-tool-failure/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "/skills/verify-output-contract/SKILL.md", "limit": 1000}

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

### Tool call: ls
{"path": "/workspace"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool call: ls
{"path": "/workspace/inventory"}

### Tool call: ls
{"path": "/workspace/tests"}

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/__pycache__/', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py']

### Tool result
['/workspace/tests/__pycache__/', '/workspace/tests/test_report.py']

### Tool call: read_file
{"file_path": "/workspace/README.md", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md", "limit": 200}

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

### Tool call: read_file
{"file_path": "/workspace/inventory/__init__.py", "limit": 200}

### Tool call: read_file
{"file_path": "/workspace/inventory/export.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/inventory/pricing.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/inventory/report.py", "limit": 400}

### Tool call: read_file
{"file_path": "/workspace/tests/test_report.py", "limit": 400}

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

### Tool call: ls
{"path": "/"}

### Tool call: glob
{"pattern": "**/*.md"}

### Tool result
['/skills/', '/workspace/']

### Tool result
['/skills/deliver-all-artifacts/SKILL.md', '/skills/recover-from-tool-failure/SKILL.md', '/skills/verify-output-contract/SKILL.md', '/workspace/CHANGELOG.md', '/workspace/README.md']

### Tool call: execute
{"command": "cd /workspace && ls -la && git log --oneline 2>&1 | head -20; python -V; python -m pytest tests -q 2>&1 | tail -30"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "ls -la /workspace /workspace/inventory /workspace/tests"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: execute
{"command": "echo hello"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Assistant
The shell tool is failing; let me note that and continue with file tools.

### Tool call: ls
{"path": "/workspace/tests/__pycache__"}

### Tool call: ls
{"path": "/workspace/inventory/__pycache__"}

### Tool result
['/workspace/tests/__pycache__/test_report.cpython-312-pytest-9.1.1.pyc']

### Tool result
['/workspace/inventory/__pycache__/__init__.cpython-312.pyc', '/workspace/inventory/__pycache__/export.cpython-312.pyc', '/workspace/inventory/__pycache__/pricing.cpython-312.pyc', '/workspace/inventory/__pycache__/report.cpython-312.pyc']

### Assistant
Shell execution is unavailable in this sandbox, so I'll proceed with static analysis (per the recovery skill). Python version is 3.12 (from `__pycache__`), pytest is present.

Let me analyze the bugs against the docstrings:

### Tool call: write_file
{"file_path": "/workspace/inventory/models.py", "content": "\"\"\"Typed data structures shared by the inventory helpers.\"\"\"\n\nfrom typing import NotRequired, TypedDict\n\n__all__ = [\"Item\"]\n\n\nclass Item(TypedDict):\n    \"\"\"A single inventory item.\n\n    Attributes:\n        name: Display name of the item.\n        price: Price as text, in any format accepted by `inventory.pricing.parse_price`.\n        qty: Number of units.\n        discount: Optional percentage discount subtracted from the unit price.\n    \"\"\"\n\n    name: str\n    price: str\n    qty: int\n    discount: NotRequired[int]\n"}

### Tool call: write_file
{"file_path": "/workspace/inventory/pricing.py", "content": "\"\"\"Price utilities for the inventory package.\"\"\"\n\nfrom decimal import ROUND_HALF_UP, Decimal, InvalidOperation\n\n__all__ = [\"apply_discount\", \"parse_price\"]\n\n_CENTS = Decimal(\"0.01\")\n_PERCENT_BASE = Decimal(100)\n\n\ndef _as_decimal(value: int | float | str | Decimal) -> Decimal:\n    \"\"\"Return `value` as a Decimal.\n\n    Floating point values are converted through their text representation so\n    that e.g. ``0.1`` becomes ``Decimal(\"0.1\")`` instead of the binary\n    approximation.\n    \"\"\"\n    if isinstance(value, Decimal):\n        return value\n    return Decimal(str(value))\n\n\ndef parse_price(text: str) -> Decimal:\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n\n    Args:\n        text: The price as text. Surrounding whitespace, \"$\" currency symbols\n            and thousands separators are ignored. A value wrapped in\n            parentheses is read as a negative amount.\n\n    Returns:\n        The parsed price.\n\n    Raises:\n        ValueError: If `text` is not a supported price format.\n    \"\"\"\n    cleaned = text.strip()\n    negative = cleaned.startswith(\"(\") and cleaned.endswith(\")\")\n    if negative:\n        cleaned

### Tool result
Updated file /workspace/inventory/models.py

### Tool result
Updated file /workspace/inventory/pricing.py

### Tool call: write_file
{"content": "\"\"\"Inventory reports.\"\"\"\n\nfrom collections.abc import Sequence\nfrom decimal import Decimal\n\nfrom .models import Item\nfrom .pricing import apply_discount, parse_price\n\n__all__ = [\"low_stock\", \"total_value\"]\n\n\ndef total_value(items: Sequence[Item]) -> Decimal:\n    \"\"\"Total value of `items` after per-item discounts.\n\n    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n\n    Args:\n        items: The items to total. The optional ``discount`` key is a\n            percentage that is subtracted from the unit price before it is\n            multiplied by ``qty``.\n\n    Returns:\n        The summed value of all items.\n    \"\"\"\n    total = Decimal(\"0\")\n    for item in items:\n        price = parse_price(item[\"price\"])\n        price = apply_discount(price, item.get(\"discount\", 0))\n        total += price * item[\"qty\"]\n    return total\n\n\ndef low_stock(items: Sequence[Item], threshold: int = 5) -> list[str]:\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n\n    Args:\n        items: The items to inspect.\n        threshold: Exclusive upper bound on the quantity.\n\n    Returns:\n        The names of the matching items, sorted case-insensitively.\n    \"\"\"\n    return sorted(\n        (item[\"name\"] for item in items if item[\"qty\"] < threshold),\n        key=str.casefold,\n    )\n", "f

### Tool call: write_file
{"content": "\"\"\"Export helpers.\"\"\"\n\nfrom .models import Item\nfrom .pricing import parse_price\n\n__all__ = [\"to_csv_row\"]\n\n\ndef to_csv_row(item: Item) -> str:\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n\n    Args:\n        item: The item to format; ``price`` is read with `parse_price`.\n\n    Returns:\n        A single CSV row without a trailing newline.\n    \"\"\"\n    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    return f\"{name},{price:.2f},{item['qty']}\"\n", "file_path": "/workspace/inventory/export.py"}

### Tool call: write_file
{"content": "\"\"\"Small inventory package: pricing, reporting and export helpers.\"\"\"\n\nfrom .export import to_csv_row\nfrom .models import Item\nfrom .pricing import apply_discount, parse_price\nfrom .report import low_stock, total_value\n\n__all__ = [\n    \"Item\",\n    \"apply_discount\",\n    \"low_stock\",\n    \"parse_price\",\n    \"to_csv_row\",\n    \"total_value\",\n]\n", "file_path": "/workspace/inventory/__init__.py"}

### Tool result
Updated file /workspace/inventory/report.py

### Tool result
Updated file /workspace/inventory/export.py

### Tool result
Updated file /workspace/inventory/__init__.py

### Tool call: write_file
{"content": "\"\"\"Regression tests for the inventory fixes.\n\nThese tests cover behaviour that the docstrings specify but the original test\nsuite did not exercise.\n\"\"\"\n\nfrom decimal import Decimal\n\nimport pytest\n\nfrom inventory.export import to_csv_row\nfrom inventory.pricing import apply_discount, parse_price\nfrom inventory.report import low_stock, total_value\n\n\ndef test_parse_price_ignores_whitespace_and_currency_symbol():\n    assert str(parse_price(\"  $12.00  \")) == \"12.00\"\n    assert str(parse_price(\"  12 \")) == \"12\"\n\n\ndef test_parse_price_ignores_thousands_separators():\n    assert str(parse_price(\"$1,299.50\")) == \"1299.50\"\n    assert str(parse_price(\"1,299.5\")) == \"1299.5\"\n\n\ndef test_parse_price_reads_accounting_style_negative():\n    assert str(parse_price(\"(12.00)\")) == \"-12.00\"\n    assert str(parse_price(\"($1,299.50)\")) == \"-1299.50\"\n\n\ndef test_parse_price_rejects_invalid_text():\n    with pytest.raises(ValueError):\n        parse_price(\"twelve dollars\")\n\n\ndef test_apply_discount_rounds_half_up():\n    assert apply_discount(Decimal(\"10.05\"), 10) == Decimal(\"9.05\")\n    assert apply_discount(Decimal(\"0.05\"), 10) == Decimal(\"0.05\")\n    assert apply_discount(Decimal(\"1.005\"), 0) == Decimal(\"1.01\")\n\n\ndef test_apply_discount_returns_two_decimals():\n    assert str(apply_discount(Decimal(\"2.50\"), 0)) == \"2.50\"\n    assert str(apply_discount(Decimal(\"100.00\"), 15)) == \"85.00\"\n\n\ndef test_to

### Tool call: write_file
{"content": "# Changelog\n\n## Unreleased\n- Fix `parse_price` to ignore thousands separators (\"$1,299.50\") and to read accounting style negatives (\"(12.00)\").\n- Fix `apply_discount` to round half up (0.005 -> 0.01) instead of banker's rounding.\n- Fix `low_stock` to return items strictly below the threshold, sorted case-insensitively.\n- Fix `to_csv_row` to quote names containing a comma or a double quote and to double embedded quotes (RFC 4180).\n- Add type annotations, `__all__` declarations, `Item` TypedDict and `py.typed` marker for the public API.\n\n## 0.3.1\n- initial import\n", "file_path": "/workspace/CHANGELOG.md"}

### Tool call: write_file
{"content": "\n", "file_path": "/workspace/inventory/py.typed"}

### Tool result
Updated file /workspace/tests/test_regressions.py

### Tool result
Updated file /workspace/CHANGELOG.md

### Tool result
Updated file /workspace/inventory/py.typed

### Assistant
Let me tighten a couple of long lines and verify the final content.

### Tool call: read_file
{"file_path": "/workspace/inventory/models.py"}

### Tool call: execute
{"command": "cd /workspace && python -m pytest tests -q"}

### Tool result
@@ lines 1-21 of 21 @@
"""Typed data structures shared by the inventory helpers."""

from typing import NotRequired, TypedDict

__all__ = ["Item"]


class Item(TypedDict):
    """A single inventory item.

    Attributes:
        name: Display name of the item.
        price: Price as text, in any format accepted by `inventory.pricing.parse_price`.
        qty: Number of units.
        discount: Optional percentage discount subtracted from the unit price.
    """

    name: str
    price: str
    qty: int
    discount: NotRequired[int]

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: edit_file
{"file_path": "/workspace/inventory/models.py", "old_string": "        price: Price as text, in any format accepted by `inventory.pricing.parse_price`.", "new_string": "        price: Price text, in a format accepted by `inventory.pricing.parse_price`."}

### Tool call: edit_file
{"file_path": "/workspace/tests/test_regressions.py", "old_string": "def test_to_csv_row_leaves_plain_name_unquoted():\n    assert to_csv_row({\"name\": \"pen\", \"price\": \"$1,299.50\", \"qty\": 2}) == \"pen,1299.50,2\"", "new_string": "def test_to_csv_row_leaves_plain_name_unquoted():\n    row = to_csv_row({\"name\": \"pen\", \"price\": \"$1,299.50\", \"qty\": 2})\n    assert row == \"pen,1299.50,2\""}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/models.py'

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/tests/test_regressions.py'

### Tool call: read_file
{"file_path": "/workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/__init__.py"}

### Tool call: read_file
{"file_path": "/workspace/tests/test_regressions.py"}

### Tool result
@@ lines 1-69 of 69 @@
"""Price utilities for the inventory package."""

from decimal import ROUND_HALF_UP, Decimal, InvalidOperation

__all__ = ["apply_discount", "parse_price"]

_CENTS = Decimal("0.01")
_PERCENT_BASE = Decimal(100)


def _as_decimal(value: int | float | str | Decimal) -> Decimal:
    """Return `value` as a Decimal.

    Floating point values are converted through their text representation so
    that e.g. ``0.1`` becomes ``Decimal("0.1")`` instead of the binary
    approximation.
    """
    if isinstance(value, Decimal):
        return value
    return Decimal(str(value))


def parse_price(text: str) -> Decimal:
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative

    Args:
        text: The price as text. Surrounding whitespace, "$" currency symbols
            and thousands separators are ignored. A value wrapped in
            parentheses is read as a negative amount.

    Returns:
        The parsed price.

    Raises:
        ValueError: If `text` is not a supported price format.
    """
    cleaned = text.strip()
    negative = cleaned.startswith("(") and cleaned.endswith(")")
    if negative:
        cleaned = cleaned[1:-1]
    cleaned = cleaned.replace("$", "").replace(",", "").strip()
    try:
        value = Decimal(cleaned)
    exce

### Tool result
@@ lines 1-48 of 48 @@
"""Inventory reports."""

from collections.abc import Sequence
from decimal import Decimal

from .models import Item
from .pricing import apply_discount, parse_price

__all__ = ["low_stock", "total_value"]


def total_value(items: Sequence[Item]) -> Decimal:
    """Total value of `items` after per-item discounts.

    Each item is a dict: {"name": str, "price": str, "qty": int, "discount": percent (optional)}.

    Args:
        items: The items to total. The optional ``discount`` key is a
            percentage that is subtracted from the unit price before it is
            multiplied by ``qty``.

    Returns:
        The summed value of all items.
    """
    total = Decimal("0")
    for item in items:
        price = parse_price(item["price"])
        price = apply_discount(price, item.get("discount", 0))
        total += price * item["qty"]
    return total


def low_stock(items: Sequence[Item], threshold: int = 5) -> list[str]:
    """Names of the items whose quantity is strictly below `threshold`.

    The names are sorted alphabetically, ignoring case.

    Args:
        items: The items to inspect.
        threshold: Exclusive upper bound on the quantity.

    Returns:
        The names of the matching items, sorted case-insensitively.
    """
    return sorted(
        (item["name"] for item in items if item["qty"] < threshold),
        key=str.casefold,
    )

### Tool result
@@ lines 1-25 of 25 @@
"""Export helpers."""

from .models import Item
from .pricing import parse_price

__all__ = ["to_csv_row"]


def to_csv_row(item: Item) -> str:
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).

    Args:
        item: The item to format; ``price`` is read with `parse_price`.

    Returns:
        A single CSV row without a trailing newline.
    """
    price = parse_price(item["price"])
    name = item["name"]
    if "," in name or '"' in name:
        name = '"' + name.replace('"', '""') + '"'
    return f"{name},{price:.2f},{item['qty']}"

### Tool result
@@ lines 1-15 of 15 @@
"""Small inventory package: pricing, reporting and export helpers."""

from .export import to_csv_row
from .models import Item
from .pricing import apply_discount, parse_price
from .report import low_stock, total_value

__all__ = [
    "Item",
    "apply_discount",
    "low_stock",
    "parse_price",
    "to_csv_row",
    "total_value",
]

### Tool result
@@ lines 1-87 of 87 @@
"""Regression tests for the inventory fixes.

These tests cover behaviour that the docstrings specify but the original test
suite did not exercise.
"""

from decimal import Decimal

import pytest

from inventory.export import to_csv_row
from inventory.pricing import apply_discount, parse_price
from inventory.report import low_stock, total_value


def test_parse_price_ignores_whitespace_and_currency_symbol():
    assert str(parse_price("  $12.00  ")) == "12.00"
    assert str(parse_price("  12 ")) == "12"


def test_parse_price_ignores_thousands_separators():
    assert str(parse_price("$1,299.50")) == "1299.50"
    assert str(parse_price("1,299.5")) == "1299.5"


def test_parse_price_reads_accounting_style_negative():
    assert str(parse_price("(12.00)")) == "-12.00"
    assert str(parse_price("($1,299.50)")) == "-1299.50"


def test_parse_price_rejects_invalid_text():
    with pytest.raises(ValueError):
        parse_price("twelve dollars")


def test_apply_discount_rounds_half_up():
    assert apply_discount(Decimal("10.05"), 10) == Decimal("9.05")
    assert apply_discount(Decimal("0.05"), 10) == Decimal("0.05")
    assert apply_discount(Decimal("1.005"), 0) == Decimal("1.01")


def test_apply_discount_returns_two_decimals():
    assert str(apply_discount(Decimal("2.50"), 0)) == "2.50"
    assert str(apply_discount(Decimal("100.00"), 15)) == "85.00"


def test_total_value_sums_items_with_and_without_discount():
    items = [
        {"name": "pen", 

### Tool call: edit_file
{"file_path": "/workspace/inventory/report.py", "old_string": "    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n\n    Args:", "new_string": "    Each item is a dict:\n    {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n\n    Args:"}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/report.py'

### Tool result
@@ lines 1-11 of 11 @@
# Changelog

## Unreleased
- Fix `parse_price` to ignore thousands separators ("$1,299.50") and to read accounting style negatives ("(12.00)").
- Fix `apply_discount` to round half up (0.005 -> 0.01) instead of banker's rounding.
- Fix `low_stock` to return items strictly below the threshold, sorted case-insensitively.
- Fix `to_csv_row` to quote names containing a comma or a double quote and to double embedded quotes (RFC 4180).
- Add type annotations, `__all__` declarations, `Item` TypedDict and `py.typed` marker for the public API.

## 0.3.1
- initial import

### Tool call: read_file
{"file_path": "/workspace/inventory/report.py", "offset": 10, "limit": 15}

### Tool call: read_file
{"file_path": "/workspace/inventory/models.py"}

### Tool result
@@ lines 11-25 of 49 | next offset 25 @@

def total_value(items: Sequence[Item]) -> Decimal:
    """Total value of `items` after per-item discounts.

    Each item is a dict:
    {"name": str, "price": str, "qty": int, "discount": percent (optional)}.

    Args:
        items: The items to total. The optional ``discount`` key is a
            percentage that is subtracted from the unit price before it is
            multiplied by ``qty``.

    Returns:
        The summed value of all items.
    """

### Tool result
@@ lines 1-21 of 21 @@
"""Typed data structures shared by the inventory helpers."""

from typing import NotRequired, TypedDict

__all__ = ["Item"]


class Item(TypedDict):
    """A single inventory item.

    Attributes:
        name: Display name of the item.
        price: Price text, in a format accepted by `inventory.pricing.parse_price`.
        qty: Number of units.
        discount: Optional percentage discount subtracted from the unit price.
    """

    name: str
    price: str
    qty: int
    discount: NotRequired[int]

### Assistant
I could not run the suite: every `execute` call in this sandbox fails with `/bin/sh: 1: runuser: not found` (the sandbox shell wrapper is broken), so I verified by static tracing against each docstring instead.

## Changes

**`inventory/pricing.py`**
- `parse_price`: the old code only stripped whitespace and `$`, so `"$1,299.50"` blew up on the thousands separator and `"(12.00)"` on the parentheses. It now strips surrounding whitespace, removes `$` and `,`, and treats an accounting-style `(…)` wrapper as negative. Invalid text raises a documented `ValueError`.
- `apply_discount`: `quantize()` defaulted to `ROUND_HALF_EVEN`, contradicting the docstring's "round half up" (`10.05 - 10% = 9.045` → `9.04`). Now uses `ROUND_HALF_UP` (test expects `9.05`).
- Added a private `_as_decimal` helper so float percents are converted via their text form.

**`inventory/report.py`**
- `low_stock`: used `<=` (docstring says *strictly* below) and returned names unsorted. Now filters `qty < threshold` and sorts case-insensitively via `key=str.casefold`.
- `total_value`: logic was already correct; kept per-unit-discount semantics and added annotations/docstrings.

**`inventory/export.py`**
- `to_csv_row`: RFC 4180 quoting was not implemented. Names containing a comma or a double quote are now wrapped in quotes with embedded quotes doubled.

**Conventions (Acme review bot)**
- Type annotations, `Args/Returns/Raises` docstrings and `__all__` on every module; new `inventory/models.py` with a shared 