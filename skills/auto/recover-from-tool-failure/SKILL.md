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