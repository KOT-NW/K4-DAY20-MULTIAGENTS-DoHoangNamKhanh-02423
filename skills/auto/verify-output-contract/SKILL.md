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