---
name: hl-to-plan
description: "Converts a high-level plan into a detailed step-by-step implementation plan with post-step instructions (rs-check, git-commit). Use when given a high-level plan file to break down into actionable steps."
argument-hint: path to the high-level plan file
---

# High-Level Plan to Step-by-Step Implementation

Converts a high-level plan into a fully detailed, step-by-step implementation plan with no
remaining decision points. The plan includes post-step verification and commit instructions.

This skill **delegates the step-execution protocol** (post-step block, recursive handoff,
context window rules, plan-file conventions) to the
[`step-handoff`](file:///Users/thlorenz/dev/ai/agent-skills/step-handoff/SKILL.md) skill.
**Load and follow the `step-handoff` skill** when producing the plan and performing the
initial handoff.

## Workflow

1. Read the high-level plan file provided as the skill argument.
2. Analyze the codebase to understand the current state of the code relevant to the plan.
3. Convert the high-level plan into a detailed step-by-step implementation plan:
   - Each step must be **fully specified** — no ambiguity, no decision points remaining.
   - Each step must be **isolated enough to compile correctly** after completion.
   - Include exact file paths, function signatures, type definitions, and code patterns.
   - Order steps so each builds on the previous one without breaking compilation.
   - Each step is sized per the **Step Size** rule in the `step-handoff` skill (one commit
     per step; use sub-steps `1.1`, `1.2`, … if needed).
4. Under each step, include the **mandatory post-step instructions block** exactly as
   specified by the `step-handoff` skill.
5. At the top of the plan, indicate that completed steps marked with ✓ are done and MUST be
   skipped (per the `step-handoff` skill).
6. Decide the handoff mechanism (`/handoff` vs `/clear`) **before** writing the plan and
   hardcode it into the plan instructions, per the `step-handoff` skill.
7. Present the full implementation plan to the user and **wait for approval**.
8. Once approved, perform the initial handoff to the subagent that executes Step 1, following
   the recursive handoff protocol from the `step-handoff` skill.

## Rules

- The plan must eliminate all ambiguity — a different agent should be able to execute it
  without making any design decisions.
- Every step must include the post-step instructions block from the `step-handoff` skill.
- All execution / handoff / commit / context-window rules come from the `step-handoff` skill
  and must not be relaxed here.

## Output

The name of the plan file should be derived from the name of the high-level plan file, e.g.:

- `prompts/11_ksql-init-hl.md` → `prompts/11_ksql-init-plan.md`
- `prompts/12_ksql-query-design.md` → `prompts/12_ksql-query-plan.md`
- `prompts/12_ksql-query.md` → `prompts/12_ksql-query-plan.md`

If the plan file already exists, create a new one with a numbered suffix and inform the user.
