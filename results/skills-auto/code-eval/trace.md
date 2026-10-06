### Human
The test suite of the `bookings` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by reading the SKILL.md files that could apply to this task.

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

### Tool call: glob
{"pattern": "workspace/**/*"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/', '/workspace/tests/']

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/__init__.py', '/workspace/bookings/__pycache__/__init__.cpython-312.pyc', '/workspace/bookings/__pycache__/billing.cpython-312.pyc', '/workspace/bookings/__pycache__/calendar_export.cpython-312.pyc', '/workspace/bookings/__pycache__/schedule.cpython-312.pyc', '/workspace/bookings/__pycache__/timeutil.cpython-312.pyc', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py', '/workspace/tests/__pycache__/test_bookings.cpython-312-pytest-9.1.1.pyc', '/workspace/tests/test_bookings.py']

### Tool call: read_file
{"file_path": "workspace/README.md", "limit": 400}

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md", "limit": 400}

### Tool result
@@ lines 1-5 of 5 @@
# bookings

Small package. Run the tests from the `workspace/` folder:

    cd workspace && python -m pytest tests -q

### Tool result
@@ lines 1-6 of 6 @@
# Changelog

## Unreleased

## 1.4.2
- initial import

### Tool call: read_file
{"file_path": "workspace/bookings/__init__.py", "limit": 400}

### Tool call: read_file
{"file_path": "workspace/bookings/billing.py", "limit": 400}

### Tool call: read_file
{"file_path": "workspace/bookings/calendar_export.py", "limit": 400}

### Tool call: read_file
{"file_path": "workspace/bookings/schedule.py", "limit": 400}

### Tool call: read_file
{"file_path": "workspace/bookings/timeutil.py", "limit": 400}

### Tool result
@@ lines 1-1 of 1 @@
__version__ = "1.4.2"

### Tool result
@@ lines 1-11 of 11 @@
"""Billing rules."""


def billable_blocks(minutes, block=15):
    """Number of billable blocks for a booking.

    A partial block is billed as a full block (always round UP):
    16 minutes with 15-minute blocks -> 2 blocks.
    `minutes` must be >= 0; a negative value raises ValueError.
    """
    return round(minutes / block)

### Tool result
@@ lines 1-7 of 7 @@
"""Calendar export."""
from .timeutil import parse_duration


def slot_end(start_minute, duration_text):
    """Minute of the day at which a slot ends."""
    return start_minute + parse_duration(duration_text)

### Tool result
@@ lines 1-17 of 17 @@
"""Schedule helpers."""
from .timeutil import parse_duration


def total_minutes(durations):
    """Total minutes of a list of duration strings."""
    return sum(parse_duration(d) for d in durations)


def add_slot(slot, slots=[]):
    """Return a NEW list containing the given slots plus `slot`.

    Separate calls must not share state: calling add_slot("a") and then
    add_slot("b") returns ["a"] and then ["b"].
    """
    slots.append(slot)
    return slots

### Tool result
@@ lines 1-19 of 19 @@
"""Time helpers for the bookings package."""
import re


def parse_duration(text):
    """Convert a duration string to whole minutes (int).

    Supported formats:
      "1h30m" -> 90
      "2h"    -> 120
      "45m"   -> 45
      "1:30"  -> 90
      "45"    -> 45    # a bare number means minutes
    Surrounding whitespace is ignored.
    """
    m = re.fullmatch(r"(\d+)h(\d+)m", text.strip())
    if not m:
        raise ValueError(f"bad duration: {text!r}")
    return int(m.group(1)) * 60 + int(m.group(2))

### Tool call: read_file
{"file_path": "workspace/tests/test_bookings.py", "limit": 400}

### Tool result
@@ lines 1-15 of 15 @@
from bookings.billing import billable_blocks
from bookings.schedule import add_slot, total_minutes


def test_total_minutes_mixed_formats():
    assert total_minutes(["1h30m", "2h"]) == 210


def test_billable_blocks_rounds_up():
    assert billable_blocks(20) == 2
    assert billable_blocks(45) == 3


def test_add_slot_single_call():
    assert add_slot("a") == ["a"]

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -40"}

### Tool call: ls
{"path": "/"}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool result
['/skills/', '/workspace/']

### Tool call: execute
{"command": "python3 -c \"print(1)\""}

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: ls
{"path": "/skills"}

### Tool call: glob
{"pattern": "workspace/.*"}

### Tool call: glob
{"pattern": "skills/**/*"}

### Tool result
['/skills/deliver-all-artifacts/', '/skills/recover-from-tool-failure/', '/skills/verify-output-contract/']

### Tool result
No files found

### Tool result
['/skills/deliver-all-artifacts/SKILL.md', '/skills/recover-from-tool-failure/SKILL.md', '/skills/verify-output-contract/SKILL.md']

### Assistant
The shell tool is broken at the environment level (`runuser: not found`), so I'll use static analysis and file tools per the recovery skill.

### Tool call: write_file
{"file_path": "workspace/bookings/timeutil.py", "content": "\"\"\"Time helpers for the bookings package.\"\"\"\n\nimport re\n\nMINUTES_PER_HOUR = 60\n\n_DURATION_PATTERNS = (\n    re.compile(r\"(?P<hours>\\d+)h(?P<minutes>\\d+)m\"),\n    re.compile(r\"(?P<hours>\\d+)h\"),\n    re.compile(r\"(?P<minutes>\\d+)m\"),\n    re.compile(r\"(?P<hours>\\d+):(?P<minutes>\\d+)\"),\n    re.compile(r\"(?P<minutes>\\d+)\"),\n)\n\n\ndef parse_duration(text: str) -> int:\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n\n    Args:\n        text: The duration string to parse.\n\n    Returns:\n        The duration as a whole number of minutes.\n\n    Raises:\n        ValueError: If `text` does not use one of the supported formats.\n    \"\"\"\n    value = text.strip()\n    for pattern in _DURATION_PATTERNS:\n        match = pattern.fullmatch(value)\n        if match:\n            groups = match.groupdict()\n            hours = int(groups.get(\"hours\") or 0)\n            minutes = int(groups.get(\"minutes\") or 0)\n            return hours * MINUTES_PER_HOUR + minutes\n    raise ValueError(f\"bad duration: {text!r}\")\n"}

### Tool call: write_file
{"file_path": "workspace/bookings/billing.py", "content": "\"\"\"Billing rules.\"\"\"\n\nimport math\n\nDEFAULT_BLOCK = 15\n\n\ndef billable_blocks(minutes: int, block: int = DEFAULT_BLOCK) -> int:\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n\n    Args:\n        minutes: Length of the booking in minutes; must be >= 0.\n        block: Length of one billable block in minutes; must be > 0.\n\n    Returns:\n        The number of whole blocks the booking is billed for.\n\n    Raises:\n        ValueError: If `minutes` is negative or `block` is not positive.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError(f\"minutes must be >= 0, got {minutes!r}\")\n    if block <= 0:\n        raise ValueError(f\"block must be > 0, got {block!r}\")\n    return math.ceil(minutes / block)\n"}

### Tool call: write_file
{"file_path": "workspace/bookings/schedule.py", "content": "\"\"\"Schedule helpers.\"\"\"\n\nfrom collections.abc import Iterable, Sequence\n\nfrom .timeutil import parse_duration\n\n\ndef total_minutes(durations: Iterable[str]) -> int:\n    \"\"\"Total minutes of a list of duration strings.\n\n    Args:\n        durations: An iterable of duration strings, e.g. ``[\"1h30m\", \"2h\"]``.\n\n    Returns:\n        The sum of all durations as a whole number of minutes.\n    \"\"\"\n    return sum(parse_duration(duration) for duration in durations)\n\n\ndef add_slot(slot: str, slots: Sequence[str] | None = None) -> list[str]:\n    \"\"\"Return a NEW list containing the given slots plus `slot`.\n\n    Separate calls must not share state: calling add_slot(\"a\") and then\n    add_slot(\"b\") returns [\"a\"] and then [\"b\"].\n\n    The list passed as `slots` is never modified.\n\n    Args:\n        slot: The slot to append.\n        slots: Optional existing slots; defaults to no slots at all.\n\n    Returns:\n        A new list with the existing slots followed by `slot`.\n    \"\"\"\n    existing = list(slots) if slots is not None else []\n    return [*existing, slot]\n"}

### Tool call: write_file
{"file_path": "workspace/bookings/calendar_export.py", "content": "\"\"\"Calendar export.\"\"\"\n\nfrom .timeutil import parse_duration\n\n\ndef slot_end(start_minute: int, duration_text: str) -> int:\n    \"\"\"Minute of the day at which a slot ends.\n\n    Args:\n        start_minute: Minute of the day at which the slot starts.\n        duration_text: The duration string of the slot.\n\n    Returns:\n        The minute of the day at which the slot ends.\n    \"\"\"\n    return start_minute + parse_duration(duration_text)\n"}

### Tool result
Updated file /workspace/bookings/timeutil.py

### Tool result
Updated file /workspace/bookings/billing.py

### Tool result
Updated file /workspace/bookings/schedule.py

### Tool result
Updated file /workspace/bookings/calendar_export.py

### Tool call: write_file
{"content": "\"\"\"Regression tests for the fixes in the bookings package.\n\nThese tests cover behaviour described in the source docstrings that the\noriginal test module does not exercise.\n\"\"\"\n\nimport pytest\n\nfrom bookings.billing import billable_blocks\nfrom bookings.calendar_export import slot_end\nfrom bookings.schedule import add_slot, total_minutes\nfrom bookings.timeutil import parse_duration\n\n\n@pytest.mark.parametrize(\n    (\"text\", \"expected\"),\n    [\n        (\"1h30m\", 90),\n        (\"2h\", 120),\n        (\"45m\", 45),\n        (\"1:30\", 90),\n        (\"45\", 45),\n        (\"0m\", 0),\n        (\" 1h30m \", 90),\n    ],\n)\ndef test_parse_duration_supported_formats(text, expected):\n    assert parse_duration(text) == expected\n\n\ndef test_parse_duration_returns_whole_minutes():\n    result = parse_duration(\"1h30m\")\n    assert isinstance(result, int)\n    assert result == 90\n\n\n@pytest.mark.parametrize(\"text\", [\"\", \"   \", \"abc\", \"1h30\", \"30m1h\", \"-5m\", \"1:2:3\"])\ndef test_parse_duration_rejects_bad_input(text):\n    with pytest.raises(ValueError):\n        parse_duration(text)\n\n\ndef test_billable_blocks_rounds_up_partial_blocks():\n    assert billable_blocks(20) == 2\n    assert billable_blocks(16) == 2\n    assert billable_blocks(45) == 3\n\n\ndef test_billable_blocks_exact_multiples_and_zero():\n    assert billable_blocks(30) == 2\n    assert billable_blocks(0) == 0\n\n\ndef test_billable_blocks_custom_block_size():\n

### Tool call: write_file
{"content": "# Changelog\n\n## Unreleased\n- `timeutil.parse_duration`: support `\"Nh\"`, `\"Nm\"`, `\"H:MM\"` and bare-minute\n  duration strings in addition to `\"NhNm\"`, and reject any other format with\n  `ValueError`.\n- `billing.billable_blocks`: round partial blocks UP to the next whole block\n  instead of to the nearest one, and raise `ValueError` for negative minutes.\n- `schedule.add_slot`: build and return a NEW list instead of mutating a shared\n  default argument, so separate calls never share state.\n\n## 1.4.2\n- initial import\n", "file_path": "workspace/CHANGELOG.md"}

### Tool result
Updated file /workspace/tests/test_regressions.py

### Tool result
Updated file /workspace/CHANGELOG.md

### Tool call: read_file
{"file_path": "workspace/bookings/timeutil.py", "limit": 100}

### Tool call: execute
{"command": "cd workspace && python3 -m pytest tests -q"}

### Tool result
@@ lines 1-44 of 44 @@
"""Time helpers for the bookings package."""

import re

MINUTES_PER_HOUR = 60

_DURATION_PATTERNS = (
    re.compile(r"(?P<hours>\d+)h(?P<minutes>\d+)m"),
    re.compile(r"(?P<hours>\d+)h"),
    re.compile(r"(?P<minutes>\d+)m"),
    re.compile(r"(?P<hours>\d+):(?P<minutes>\d+)"),
    re.compile(r"(?P<minutes>\d+)"),
)


def parse_duration(text: str) -> int:
    """Convert a duration string to whole minutes (int).

    Supported formats:
      "1h30m" -> 90
      "2h"    -> 120
      "45m"   -> 45
      "1:30"  -> 90
      "45"    -> 45    # a bare number means minutes
    Surrounding whitespace is ignored.

    Args:
        text: The duration string to parse.

    Returns:
        The duration as a whole number of minutes.

    Raises:
        ValueError: If `text` does not use one of the supported formats.
    """
    value = text.strip()
    for pattern in _DURATION_PATTERNS:
        match = pattern.fullmatch(value)
        if match:
            groups = match.groupdict()
            hours = int(groups.get("hours") or 0)
            minutes = int(groups.get("minutes") or 0)
            return hours * MINUTES_PER_HOUR + minutes
    raise ValueError(f"bad duration: {text!r}")

### Tool result
[stderr] /bin/sh: 1: runuser: not found

Exit code: 127
[Command failed with exit code 127]

### Tool call: read_file
{"file_path": "workspace/bookings/billing.py", "limit": 100}

### Tool call: read_file
{"file_path": "workspace/bookings/schedule.py", "limit": 100}

### Tool call: read_file
{"file_path": "workspace/bookings/calendar_export.py", "limit": 100}

### Tool call: read_file
{"file_path": "workspace/tests/test_regressions.py", "limit": 200}

### Tool result
@@ lines 1-29 of 29 @@
"""Billing rules."""

import math

DEFAULT_BLOCK = 15


def billable_blocks(minutes: int, block: int = DEFAULT_BLOCK) -> int:
    """Number of billable blocks for a booking.

    A partial block is billed as a full block (always round UP):
    16 minutes with 15-minute blocks -> 2 blocks.
    `minutes` must be >= 0; a negative value raises ValueError.

    Args:
        minutes: Length of the booking in minutes; must be >= 0.
        block: Length of one billable block in minutes; must be > 0.

    Returns:
        The number of whole blocks the booking is billed for.

    Raises:
        ValueError: If `minutes` is negative or `block` is not positive.
    """
    if minutes < 0:
        raise ValueError(f"minutes must be >= 0, got {minutes!r}")
    if block <= 0:
        raise ValueError(f"block must be > 0, got {block!r}")
    return math.ceil(minutes / block)

### Tool result
@@ lines 1-36 of 36 @@
"""Schedule helpers."""

from collections.abc import Iterable, Sequence

from .timeutil import parse_duration


def total_minutes(durations: Iterable[str]) -> int:
    """Total minutes of a list of duration strings.

    Args:
        durations: An iterable of duration strings, e.g. ``["1h30m", "2h"]``.

    Returns:
        The sum of all durations as a whole number of minutes.
    """
    return sum(parse_duration(duration) for duration in durations)


def add_slot(slot: str, slots: Sequence[str] | None = None) -> list[str]:
    """Return a NEW list containing the given slots plus `slot`.

    Separate calls must not share state: calling add_slot("a") and then
    add_slot("b") returns ["a"] and then ["b"].

    The list passed as `slots` is never modified.

    Args:
        slot: The slot to append.
        slots: Optional existing slots; defaults to no slots at all.

    Returns:
        A new list with the existing slots followed by `slot`.
    """
    existing = list(slots) if slots is not None else []
    return [*existing, slot]

### Tool result
@@ lines 1-16 of 16 @@
"""Calendar export."""

from .timeutil import parse_duration


def slot_end(start_minute: int, duration_text: str) -> int:
    """Minute of the day at which a slot ends.

    Args:
        start_minute: Minute of the day at which the slot starts.
        duration_text: The duration string of the slot.

    Returns:
        The minute of the day at which the slot ends.
    """
    return start_minute + parse_duration(duration_text)

### Tool result
@@ lines 1-91 of 91 @@
"""Regression tests for the fixes in the bookings package.

These tests cover behaviour described in the source docstrings that the
original test module does not exercise.
"""

import pytest

from bookings.billing import billable_blocks
from bookings.calendar_export import slot_end
from bookings.schedule import add_slot, total_minutes
from bookings.timeutil import parse_duration


@pytest.mark.parametrize(
    ("text", "expected"),
    [
        ("1h30m", 90),
        ("2h", 120),
        ("45m", 45),
        ("1:30", 90),
        ("45", 45),
        ("0m", 0),
        (" 1h30m ", 90),
    ],
)
def test_parse_duration_supported_formats(text, expected):
    assert parse_duration(text) == expected


def test_parse_duration_returns_whole_minutes():
    result = parse_duration("1h30m")
    assert isinstance(result, int)
    assert result == 90


@pytest.mark.parametrize("text", ["", "   ", "abc", "1h30", "30m1h", "-5m", "1:2:3"])
def test_parse_duration_rejects_bad_input(text):
    with pytest.raises(ValueError):
        parse_duration(text)


def test_billable_blocks_rounds_up_partial_blocks():
    assert billable_blocks(20) == 2
    assert billable_blocks(16) == 2
    assert billable_blocks(45) == 3


def test_billable_blocks_exact_multiples_and_zero():
    assert billable_blocks(30) == 2
    assert billable_blocks(0) == 0


def test_billable_blocks_custom_block_size():
    assert billable_blocks(61, block=30) == 3


def test_billable_blocks_rejects_negat

### Tool call: edit_file
{"file_path": "workspace/bookings/billing.py", "old_string": "DEFAULT_BLOCK = 15\n\n\ndef billable_blocks(minutes: int, block: int = DEFAULT_BLOCK) -> int:", "new_string": "def billable_blocks(minutes: int, block: int = 15) -> int:"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/bookings/billing.py'

### Tool call: read_file
{"file_path": "workspace/bookings/billing.py", "limit": 40}

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md", "limit": 40}

### Tool result
@@ lines 1-26 of 26 @@
"""Billing rules."""

import math

def billable_blocks(minutes: int, block: int = 15) -> int:
    """Number of billable blocks for a booking.

    A partial block is billed as a full block (always round UP):
    16 minutes with 15-minute blocks -> 2 blocks.
    `minutes` must be >= 0; a negative value raises ValueError.

    Args:
        minutes: Length of the booking in minutes; must be >= 0.
        block: Length of one billable block in minutes; must be > 0.

    Returns:
        The number of whole blocks the booking is billed for.

    Raises:
        ValueError: If `minutes` is negative or `block` is not positive.
    """
    if minutes < 0:
        raise ValueError(f"minutes must be >= 0, got {minutes!r}")
    if block <= 0:
        raise ValueError(f"block must be > 0, got {block!r}")
    return math.ceil(minutes / block)

### Tool result
@@ lines 1-13 of 13 @@
# Changelog

## Unreleased
- `timeutil.parse_duration`: support `"Nh"`, `"Nm"`, `"H:MM"` and bare-minute
  duration strings in addition to `"NhNm"`, and reject any other format with
  `ValueError`.
- `billing.billable_blocks`: round partial blocks UP to the next whole block
  instead of to the nearest one, and raise `ValueError` for negative minutes.
- `schedule.add_slot`: build and return a NEW list instead of mutating a shared
  default argument, so separate calls never share state.

## 1.4.2
- initial import

### Tool call: edit_file
{"file_path": "workspace/bookings/billing.py", "old_string": "import math\n\ndef billable_blocks", "new_string": "import math\n\n\ndef billable_blocks"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/bookings/billing.py'

### Assistant
I've completed the fixes. The sandbox shell is unavailable (`/bin/sh: 1: runuser: not found` on every command, including a trivial one), so I verified everything by static reasoning and by re-reading each file against its docstring contract.

## What I changed and why

`workspace/bookings/timeutil.py` — `parse_duration` only accepted `"NhNm"`, so `"2h"`, `"45m"`, `"1:30"` and bare `"45"` all raised `ValueError`. It now matches all five documented formats (surrounding whitespace stripped, still `ValueError` for anything else) and always returns an `int`, with named regex groups and full type hints/Google-style docstring.

`workspace/bookings/billing.py` — `billable_blocks` used `round()`, which rounds to *nearest* (16 min → 1 block, contradicting the docstring's "always round UP"); it now uses `math.ceil`, returns `int`, and raises `ValueError` for negative `minutes` as documented (plus a guarded non-positive `block`, documented in the docstring).

`workspace/bookings/schedule.py` — `add_slot` used a mutable default argument (`slots=[]`), so calls shared state and mutated the caller's list. It now takes `slots: Sequence[str] | None = None` and returns a new list (`[*existing, slot]`) without touching the input; `total_minutes` gained type hints/docs (logic was already correct).

`workspace/bookings/calendar_export.py` — logic was already correct (`start_minute + parse_duration(...)`); added type annotations and Args/Returns docs only.

`workspace/CHANGELOG.md` — added one bull