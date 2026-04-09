---
name: err-fix
description: "Fixes a single provided compiler error. Use when given a specific error to fix. Focuses exclusively on the provided error and ignores all others."
---

# Fix a Single Error

Fix ONLY the specific error provided by the user. Do not fix, address, or investigate any other errors.

## Rules

1. **Fix only the provided error** — even if you encounter other errors in the same file or codebase, ignore them completely.
2. **Leave the code in whatever state results** — it is acceptable (and expected) for the code to remain broken in other ways after your fix.
3. **Do not refactor, clean up, or improve** surrounding code. Change the minimum amount of code required to resolve the provided error.
4. **The task is complete** once the provided error no longer occurs. Do not continue looking for more issues.

## Workflow

1. Read the error message to identify the file, line, and cause.
2. Read the relevant code.
3. Make the smallest edit that resolves the error.
4. Verify the specific error is resolved (e.g. run the compiler and confirm that exact error is gone).
5. Stop. Do not address any remaining errors.
