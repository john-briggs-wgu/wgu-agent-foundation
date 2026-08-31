---
inclusion: auto
description: Fix root cause, never hide failures, no continue-on-error
---

# No Workarounds

- Never hide failures. Never paper over problems. Fix the root cause.
- No `continue-on-error: true` in CI — NEVER.
- No `|| true` on commands that should fail loudly.
- No hardcoding values that should be variables.
- No commenting out failing policies instead of fixing them.
- No `--force` flags to bypass safety checks.
- No stub implementations returning fake data in production code.
- Every CI step that matters must fail the build if it fails.
- If you can't fix it now, escalate — don't hide it.
- Understand the failure → trace the root cause → fix it.
- A passing CI run is a contract that the code is deployable.
