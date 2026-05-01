---
name: comments-fix
description: Verifies and fixes a list of inline review comments/findings one at a time, with recursive handoff between subagents. Use when given a set of code review findings to verify and fix step-by-step.
argument-hint: list of inline review comments/findings (each finding becomes its own step)
---

# Comments-Fix Skill

Processes a list of inline review comments / findings by treating **each finding as a
separate step**.

This skill **delegates the step-execution protocol** (post-step block, recursive handoff,
context window rules, verification-only steps, plan-file conventions) to the
[`step-handoff`](file:///Users/thlorenz/dev/ai/agent-skills/step-handoff/SKILL.md) skill.
**Load and follow the `step-handoff` skill** when producing the plan and performing the
initial handoff.

## Workflow

1. Read the input list of findings.
2. Parse the findings into a numbered list of steps — each finding (one bullet referencing a
   file + line range) becomes one step. Preserve file path and line context for each.
3. Write the plan to `prompts/comments-fix-plan.md` with numbered headers per step. Include
   the verbatim original finding text under each header so the executing subagent has full
   context.
   - At the top of the plan, indicate that completed steps marked with ✓ are done and MUST
     be skipped (per the `step-handoff` skill).
   - Under each step, include the **mandatory post-step instructions block** exactly as
     specified by the `step-handoff` skill.
4. Hand off to a subagent (via `/handoff`) to execute Step 1, following the recursive
   handoff protocol from the `step-handoff` skill.

If the plan file already exists, create a new one with a numbered suffix
(`comments-fix-plan-2.md`, etc.) and inform the user.

## Per-Step Execution (what each subagent does)

For its assigned step, a subagent MUST:

1. **Verify the finding against the current code.** Read the referenced file(s) and lines.
   Determine if the issue still exists. Only fix it if needed.
   - If the finding is already addressed / no longer applies, treat the step as a
     **verification-only step** as defined by the `step-handoff` skill (skip post-step items
     1–3, mark ✓, write a skip note to `/tmp/comments-fix-skip-<step-number>.md`, then hand
     off).
2. If a fix is needed, implement it exactly as the finding describes.
3. Execute the **post-step instructions block** in full (per the `step-handoff` skill).
4. Mark the step as completed (✓) in the plan file.
5. Hand off to the next subagent for the next step (recursive handoff per the `step-handoff`
   skill).

## Rules

- Always verify against current code first; do not blindly apply fixes.
- All execution / handoff / commit / context-window rules come from the `step-handoff` skill
  and must not be relaxed here.

## Plan File Format

```markdown
# Comments-Fix Plan

> Steps marked with ✓ are already completed and MUST be skipped.

> For each step: verify the finding against the current code first. Only fix it if needed.

## Step 1: <short description, e.g. "Fix init-config perl regex in geyser-plugin/Makefile">

**File:** `geyser-plugin/Makefile`
**Lines:** 21-25

**Finding:**
<verbatim copy of the original finding text for this item>

> **⛔ MANDATORY — After completing this step (NEVER skip, NEVER defer, NEVER batch):**
> 1. Use the `rs-check` skill — fix any issues until it passes cleanly.
> 2. Stage only files added/changed/removed in this step (nothing unrelated).
> 3. Use the `git-commit` skill to commit with a message that describes the step just completed. Respect the commit message format from that skill. Do NOT ask for commit message approval.
> 4. Do NOT move to the next step until rs-check passes and the commit is done.
> 5. **Continuous implementation does NOT override these constraints.** You must stop and execute every post-step block before writing any more code.

## Step 2: ...
```
