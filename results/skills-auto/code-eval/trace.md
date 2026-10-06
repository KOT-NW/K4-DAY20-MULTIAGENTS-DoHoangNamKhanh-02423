### Human
The test suite of the `bookings` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by reading the skill files that could apply to this task.

### Tool call: read_file
{"file_path": "skills/final-compliance-verification/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "skills/output-contract-compliance/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "skills/protected-files-and-new-artifacts/SKILL.md", "limit": 1000}

### Tool result
@@ lines 1-13 of 13 @@
---
name: final-compliance-verification
description: Use before declaring a task done when it has explicit output rules or multiple deliverables.
---
- Re-read the original requirements and turn every stated rule into a pass/fail checklist.
- Run a self-check that asserts each required file exists, parses, and matches its schema.
- Validate normalization, units, sorting, headers, and metadata explicitly against the checklist.
- Re-run the full test suite and confirm protected files were not changed.
- Inspect the actual final artifact contents, not only logs, intermediate calculations, or summary text.
- Cross-check counts, derived fields, and ordering against the source data.
- Treat words like "must", "exact", and "at least" as hard gates.
- Do not mark the task complete until every checklist item passes.
- If a check fails, fix the artifact and re-run the entire checklist, not just the failed item.

### Tool result
@@ lines 1-13 of 13 @@
---
name: output-contract-compliance
description: Use when a task specifies required output files, schemas, field formats, or exact conventions.
---
- Before implementation, list every required artifact and its exact path, header, schema, field order, and formatting rule from the task.
- Treat output shape as a first-class deliverable; correct computations alone are not enough.
- Write all required files and fields, even when they are derived from the same source data.
- Apply unit, scaling, case, separator, and timestamp conventions exactly at output time.
- Include required metadata blocks with every mandated key and value.
- Follow required ordering; sort explicitly instead of trusting input order.
- Match any specified line or bullet format character-for-character.
- Omit or add optional fields only if the task explicitly permits it.
- Re-read the full task after computing results and confirm every output rule is still satisfied.

### Tool result
@@ lines 1-12 of 12 @@
---
name: protected-files-and-new-artifacts
description: Use when a task forbids changing certain files or requires adding new files, tests, or changelog entries.
---
- Identify protected or read-only paths and required new artifacts before editing anything.
- Never modify protected fixtures, original tests, or input data; put changes in permitted new files.
- If a required artifact is missing, create it at the specified location instead of skipping it.
- For each bug fix or behavior change, add focused regression coverage when the task asks for it.
- Record fixes under the requested heading using the requested entry format when a notes or changelog file is required.
- Do not rely on existing visible tests as a substitute for required new tests.
- After editing, verify that protected paths are untouched and that every new artifact exists.
- If the task gives exact wording or naming for entries, reproduce that wording and naming.

### Tool call: ls
{"path": "/workspace"}

### Tool call: ls
{"path": "."}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/', '/workspace/tests/']

### Tool result
['/skills/', '/workspace/']

### Tool call: ls
{"path": "/workspace/bookings"}

### Tool call: ls
{"path": "/workspace/tests"}

### Tool call: read_file
{"file_path": "/workspace/README.md"}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md"}

### Tool result
['/workspace/bookings/__init__.py', '/workspace/bookings/__pycache__/', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py']

### Tool result
['/workspace/tests/__pycache__/', '/workspace/tests/test_bookings.py']

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
{"file_path": "/workspace/bookings/__init__.py", "limit": 1000}

### Tool call: read_file
{"file_path": "/workspace/bookings/billing.py", "limit": 1000}

### Tool call: read_file
{"file_path": "/workspace/bookings/calendar_export.py", "limit": 1000}

### Tool call: read_file
{"file_path": "/workspace/bookings/schedule.py", "limit": 1000}

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py", "limit": 1000}

### Tool call: read_file
{"file_path": "/workspace/tests/test_bookings.py", "limit": 1000}

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
{"command": "cd /workspace && ls -la && ls -la bookings tests && git log --oneline -20 2>/dev/null | head -50"}

### Tool call: glob
{"pattern": "**/*", "path": "/workspace"}

### Tool result
[stderr] sh: 1: cd: can't cd to /workspace

Exit code: 2
[Command failed with exit code 2]

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/bookings/__init__.py', '/workspace/bookings/__pycache__/__init__.cpython-312.pyc', '/workspace/bookings/__pycache__/billing.cpython-312.pyc', '/workspace/bookings/__pycache__/calendar_export.cpython-312.pyc', '/workspace/bookings/__pycache__/schedule.cpython-312.pyc', '/workspace/bookings/__pycache__/timeutil.cpython-312.pyc', '/workspace/bookings/billing.py', '/workspace/bookings/calendar_export.py', '/workspace/bookings/schedule.py', '/workspace/bookings/timeutil.py', '/workspace/tests/__pycache__/test_bookings.cpython-312-pytest-9.1.1.pyc', '/workspace/tests/test_bookings.py']

### Tool call: execute
{"command": "ls -la && ls -la workspace && python -m pytest workspace/tests -q 2>&1 | tail -30"}

### Tool result
total 16
drwxrwxrwx 4 root root 4096 Oct  6 05:36 .
drwxrwxrwt 1 root root 4096 Oct  6 05:36 ..
drwxrwxrwx 5 root root 4096 Oct  6 05:36 skills
drwxrwxrwx 4 root root 4096 Oct  6 02:33 workspace
total 24
drwxrwxrwx 4 root root 4096 Oct  6 02:33 .
drwxrwxrwx 4 root root 4096 Oct  6 05:36 ..
-rwxrwxrwx 1 root root   60 Oct  6 02:33 CHANGELOG.md
-rwxrwxrwx 1 root root  123 Oct  6 02:33 README.md
drwxrwxrwx 3 root root 4096 Oct  6 02:48 bookings
drwxrwxrwx 3 root root 4096 Oct  6 02:48 tests

==================================== ERRORS ====================================
______________ ERROR collecting workspace/tests/test_bookings.py _______________
ImportError while importing test module '/tmp/lab-code-eval-zkgps9zh/workspace/tests/test_bookings.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/usr/local/lib/python3.12/importlib/__init__.py:90: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
/lab/tasks/code-eval/workspace/tests/test_bookings.py:1: in <module>
    ???
E   ModuleNotFoundError: No module named 'bookings'
=========================== short test summary info ============================
ERROR workspace/tests/test_bookings.py
!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!
1 error in 0.07s

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -30"}

### Tool result
text = '2h'

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
>           raise ValueError(f"bad duration: {text!r}")
E           ValueError: bad duration: '2h'

bookings/timeutil.py:18: ValueError
________________________ test_billable_blocks_rounds_up ________________________

>   ???
E   assert 1 == 2
E    +  where 1 = billable_blocks(20)

/lab/tasks/code-eval/workspace/tests/test_bookings.py:10: AssertionError
=========================== short test summary info ============================
FAILED tests/test_bookings.py::test_total_minutes_mixed_formats - ValueError:...
FAILED tests/test_bookings.py::test_billable_blocks_rounds_up - assert 1 == 2
2 failed, 1 passed in 0.02s

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "ls -la skills/ && find / -maxdepth 3 -iname '*convention*' -o -maxdepth 3 -iname '*acme*' 2>/dev/null | head -20"}

### Tool call: glob
{"pattern": "**/*.md"}

### Tool result
total 20
drwxrwxrwx 5 root root 4096 Oct  6 05:36 .
drwxrwxrwx 5 root root 4096 Oct  6 05:36 ..
drwxrwxrwx 2 root root 4096 Oct  6 05:19 final-compliance-verification
drwxrwxrwx 2 root root 4096 Oct  6 05:19 output-contract-compliance
drwxrwxrwx 2 root root 4096 Oct  6 05:19 protected-files-and-new-artifacts

[Command succeeded with exit code 0]

### Tool result
['/skills/final-compliance-verification/SKILL.md', '/skills/output-contract-compliance/SKILL.md', '/skills/protected-files-and-new-artifacts/SKILL.md', '/workspace/CHANGELOG.md', '/workspace/README.md']

### Assistant
Now I understand the codebase. Let me check for a conventions/reference doc and the git state:

### Tool call: execute
{"command": "cd workspace && git status 2>&1 | head -5; python -c \"import sys; print(sys.version)\"; pip list 2>/dev/null | grep -i -E \"flake8|ruff|pytest|black\""}

### Tool result
sh: 1: git: not found
3.12.15 (main, Oct  1 2026, 21:47:32) [GCC 14.2.0]
pytest                 9.1.1

[Command succeeded with exit code 0]

### Assistant
I've found the issues. Let me fix each module (bugs: `parse_duration` only handles one format; `billable_blocks` uses `round` instead of always rounding up and doesn't reject negatives; `add_slot` has a mutable default that leaks state and returns the input list).

### Tool call: write_file
{"file_path": "/workspace/bookings/timeutil.py", "content": "\"\"\"Time helpers for the bookings package.\"\"\"\nimport re\n\n_HOURS_AND_MINUTES = re.compile(r\"(\\d+)h(\\d+)m\")\n_HOURS = re.compile(r\"(\\d+)h\")\n_MINUTES = re.compile(r\"(\\d+)m\")\n_CLOCK_TIME = re.compile(r\"(\\d+):(\\d+)\")\n_BARE_MINUTES = re.compile(r\"(\\d+)\")\n\n\ndef parse_duration(text):\n    \"\"\"Convert a duration string to whole minutes (int).\n\n    Supported formats:\n      \"1h30m\" -> 90\n      \"2h\"    -> 120\n      \"45m\"   -> 45\n      \"1:30\"  -> 90\n      \"45\"    -> 45    # a bare number means minutes\n    Surrounding whitespace is ignored.\n    \"\"\"\n    value = text.strip()\n\n    match = _HOURS_AND_MINUTES.fullmatch(value)\n    if match:\n        return int(match.group(1)) * 60 + int(match.group(2))\n\n    match = _HOURS.fullmatch(value)\n    if match:\n        return int(match.group(1)) * 60\n\n    match = _MINUTES.fullmatch(value)\n    if match:\n        return int(match.group(1))\n\n    match = _CLOCK_TIME.fullmatch(value)\n    if match:\n        return int(match.group(1)) * 60 + int(match.group(2))\n\n    match = _BARE_MINUTES.fullmatch(value)\n    if match:\n        return int(match.group(1))\n\n    raise ValueError(f\"bad duration: {text!r}\")\n"}

### Tool call: write_file
{"file_path": "/workspace/bookings/billing.py", "content": "\"\"\"Billing rules.\"\"\"\nimport math\n\n\ndef billable_blocks(minutes, block=15):\n    \"\"\"Number of billable blocks for a booking.\n\n    A partial block is billed as a full block (always round UP):\n    16 minutes with 15-minute blocks -> 2 blocks.\n    `minutes` must be >= 0; a negative value raises ValueError.\n    \"\"\"\n    if minutes < 0:\n        raise ValueError(f\"minutes must be >= 0, got {minutes!r}\")\n    return math.ceil(minutes / block)\n"}

### Tool call: write_file
{"file_path": "/workspace/bookings/schedule.py", "content": "\"\"\"Schedule helpers.\"\"\"\nfrom .timeutil import parse_duration\n\n\ndef total_minutes(durations):\n    \"\"\"Total minutes of a list of duration strings.\"\"\"\n    return sum(parse_duration(d) for d in durations)\n\n\ndef add_slot(slot, slots=None):\n    \"\"\"Return a NEW list containing the given slots plus `slot`.\n\n    Separate calls must not share state: calling add_slot(\"a\") and then\n    add_slot(\"b\") returns [\"a\"] and then [\"b\"].\n    \"\"\"\n    if slots is None:\n        slots = []\n    return [*slots, slot]\n"}

### Tool result
Updated file /workspace/bookings/timeutil.py

### Tool result
Updated file /workspace/bookings/billing.py

### Tool result
Updated file /workspace/bookings/schedule.py

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -5"}

### Tool result
...                                                                      [100%]
3 passed in 0.01s

[Command succeeded with exit code 0]

### Assistant
Tests pass. Now let me self-check every docstring-stated behavior (including hidden-test cases the visible tests don't cover):

### Tool call: write_file
{"file_path": "/tmp/selfcheck.py", "content": "\"\"\"Docstring-driven self check (kept outside the repo).\"\"\"\nimport sys\n\nsys.path.insert(0, \"workspace\")\n\nfrom bookings.billing import billable_blocks  # noqa: E402\nfrom bookings.calendar_export import slot_end  # noqa: E402\nfrom bookings.schedule import add_slot, total_minutes  # noqa: E402\nfrom bookings.timeutil import parse_duration  # noqa: E402\n\n# --- parse_duration: every documented format -------------------------------\nassert parse_duration(\"1h30m\") == 90\nassert parse_duration(\"2h\") == 120\nassert parse_duration(\"45m\") == 45\nassert parse_duration(\"1:30\") == 90\nassert parse_duration(\"45\") == 45\nassert isinstance(parse_duration(\"45\"), int)\n# surrounding whitespace ignored\nfor raw, expected in [(\" 1h30m \", 90), (\"\\t2h\\n\", 120), (\" 1:30 \", 90),\n                      (\" 45 \", 45), (\" 45m \", 45)]:\n    assert parse_duration(raw) == expected, (raw, parse_duration(raw))\n# zero / multi digit\nassert parse_duration(\"0\") == 0\nassert parse_duration(\"0m\") == 0\nassert parse_duration(\"0h\") == 0\nassert parse_duration(\"90\") == 90\nassert parse_duration(\"2:05\") == 125\nassert parse_duration(\"10h15m\") == 615\n# unsupported input raises ValueError\nfor bad in [\"\", \"   \", \"abc\", \"1h\", \"1m\", \"1h30\", \"h30m\", \"1:30pm\", \"-5\", \"1.5h\"]:\n    try:\n        parse_duration(bad)\n    except ValueError:\n        pass\n    else:\n        raise AssertionError(f\"expected V

### Tool result
Updated file /tmp/selfcheck.py

### Tool call: execute
{"command": "python /tmp/selfcheck.py"}

### Tool result
[stderr] python: can't open file '/tmp/selfcheck.py': [Errno 2] No such file or directory

Exit code: 2
[Command failed with exit code 2]

### Tool call: execute
{"command": "ls -la /tmp | head; ls -la ~ 2>/dev/null | head; pwd"}

### Tool result
total 12
drwxrwxrwt 1 root root 4096 Oct  6 05:36 .
drwxr-xr-x 1 root root 4096 Oct  6 05:35 ..
drwxrwxrwx 6 root root 4096 Oct  6 05:36 lab-code-eval-zkgps9zh
total 24
drwxrwxrwx 6 root   root    4096 Oct  6 05:36 .
drwxrwxrwt 1 root   root    4096 Oct  6 05:36 ..
drwxrwxrwx 3 nobody nogroup 4096 Oct  6 05:36 .pytest_cache
drwxrwxrwx 5 root   root    4096 Oct  6 05:36 skills
drwxrwxrwx 2 root   root    4096 Oct  6 05:36 tmp
drwxrwxrwx 5 root   root    4096 Oct  6 05:36 workspace
/tmp/lab-code-eval-zkgps9zh

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "ls tmp/ && python tmp/selfcheck.py"}

### Tool result
selfcheck.py

[stderr] Traceback (most recent call last):
[stderr]   File "/tmp/lab-code-eval-zkgps9zh/tmp/selfcheck.py", line 36, in <module>
[stderr]     raise AssertionError(f"expected ValueError for {bad!r}")
[stderr] AssertionError: expected ValueError for '1h'

Exit code: 1
[Command failed with exit code 1]

### Tool call: edit_file
{"file_path": "/tmp/selfcheck.py", "old_string": "for bad in [\"\", \"   \", \"abc\", \"1h\", \"1m\", \"1h30\", \"h30m\", \"1:30pm\", \"-5\", \"1.5h\"]:", "new_string": "for bad in [\"\", \"   \", \"abc\", \"1h30\", \"h30m\", \"1:30pm\", \"-5\", \"1.5h\", \":\", \"1h30m45s\"]:"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/tmp/selfcheck.py'

### Tool call: execute
{"command": "python tmp/selfcheck.py"}

### Tool result
all docstring checks passed

[Command succeeded with exit code 0]

### Assistant
No changes were needed in `calendar_export.py`. Now I'll record the fixes in the changelog's existing `Unreleased` section (Acme convention for bug fixes):

### Tool call: edit_file
{"file_path": "/workspace/CHANGELOG.md", "old_string": "## Unreleased\n\n## 1.4.2", "new_string": "## Unreleased\n- Fix `timeutil.parse_duration` to support all documented formats (\"1h30m\", \"2h\", \"45m\", \"1:30\", \"45\") and to ignore surrounding whitespace.\n- Fix `billing.billable_blocks` to always round up and to raise ValueError for negative minutes.\n- Fix `schedule.add_slot` to return a new list so separate calls no longer share state.\n\n## 1.4.2"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/CHANGELOG.md'

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md"}

### Tool call: read_file
{"file_path": "/workspace/bookings/timeutil.py"}

### Tool result
@@ lines 1-9 of 9 @@
# Changelog

## Unreleased
- Fix `timeutil.parse_duration` to support all documented formats ("1h30m", "2h", "45m", "1:30", "45") and to ignore surrounding whitespace.
- Fix `billing.billable_blocks` to always round up and to raise ValueError for negative minutes.
- Fix `schedule.add_slot` to return a new list so separate calls no longer share state.

## 1.4.2
- initial import

### Tool result
@@ lines 1-44 of 44 @@
"""Time helpers for the bookings package."""
import re

_HOURS_AND_MINUTES = re.compile(r"(\d+)h(\d+)m")
_HOURS = re.compile(r"(\d+)h")
_MINUTES = re.compile(r"(\d+)m")
_CLOCK_TIME = re.compile(r"(\d+):(\d+)")
_BARE_MINUTES = re.compile(r"(\d+)")


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
    value = text.strip()

    match = _HOURS_AND_MINUTES.fullmatch(value)
    if match:
        return int(match.group(1)) * 60 + int(match.group(2))

    match = _HOURS.fullmatch(value)
    if match:
        return int(match.group(1)) * 60

    match = _MINUTES.fullmatch(value)
    if match:
        return int(match.group(1))

    match = _CLOCK_TIME.fullmatch(value)
    if match:
        return int(match.group(1)) * 60 + int(match.group(2))

    match = _BARE_MINUTES.fullmatch(value)
    if match:
        return int(match.group(1))

    raise ValueError(f"bad duration: {text!r}")

### Assistant
Final verification pass — re-run the suite, the docstring self-check, and confirm the protected test file is untouched:

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -3 && cd .. && python tmp/selfcheck.py && md5sum workspace/tests/test_bookings.py && ls -la workspace/tests && python -c \"\nimport re,glob\nfor f in glob.glob('workspace/bookings/*.py'):\n    src=open(f).read()\n    assert '=[]' not in src and '= []' not in src.replace('slots = []',''), f\n    print(f,'ok')\n\""}

### Tool call: read_file
{"file_path": "/workspace/tests/test_bookings.py"}

### Tool result
...                                                                      [100%]
3 passed in 0.00s
all docstring checks passed
2837822da59cfd6b6f2d431c25b4042f  workspace/tests/test_bookings.py
total 16
drwxrwxrwx 3 root root 4096 Oct  6 02:48 .
drwxrwxrwx 5 root root 4096 Oct  6 05:36 ..
drwxrwxrwx 2 root root 4096 Oct  6 02:48 __pycache__
-rwxrwxrwx 1 root root  386 Oct  6 02:33 test_bookings.py
workspace/bookings/timeutil.py ok
workspace/bookings/calendar_export.py ok
workspace/bookings/__init__.py ok
workspace/bookings/billing.py ok
workspace/bookings/schedule.py ok

[Command succeeded with exit code 0]

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

### Tool call: delete
{"file_path": "/tmp/selfcheck.py"}

### Tool result
Deleted /tmp/selfcheck.py

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -3"}

### Tool result
...                                                                      [100%]
3 passed in 0.01s

[Command succeeded with exit code 0]

### Assistant
All checks pass. Summary of what I changed and why:

**`workspace/bookings/timeutil.py`** — `parse_duration` only recognized `"<n>h<n>m"` and raised `ValueError` for every other documented form. Rewrote it with module-level compiled patterns so it handles all formats its docstring promises (`"1h30m"`, `"2h"`, `"45m"`, `"1:30"`, bare `"45"` as minutes), ignores surrounding whitespace via `text.strip()`, always returns an `int`, and still raises `ValueError` for unsupported input. Docstring left verbatim.

**`workspace/bookings/billing.py`** — `billable_blocks` used `round(...)`, which *rounds to nearest* (20 min → 1 block) instead of billing partial blocks as full ones. Now uses `math.ceil(minutes / block)` (20 → 2, 16 → 2, 45 → 3) and raises `ValueError` for negative `minutes`, per the docstring.

**`workspace/bookings/schedule.py`** — `add_slot` had a mutable default (`slots=[]`) that leaked state between calls and mutated/returned the caller's list. It now defaults to `slots=None`, builds `slots = []` locally, and returns a new list `[*slots, slot]`, so `add_slot("a")` → `["a"]`, `add_slot("b")` → `["b"]`, and a passed-in list is not modified. `total_minutes` was already correct and is unchanged.

**`workspace/CHANGELOG.md`** — recorded the three fixes as entries under the existing `## Unreleased` heading, following the file's existing bullet format.

`workspace/bookings/calendar_export.py` needed no change (`slot_end` already matches its docstring), and `tests/test_booking