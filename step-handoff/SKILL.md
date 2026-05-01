---
name: step-handoff
description: Provides the recursive subagent handoff protocol for executing multi-step plans one step per thread. Use when a skill needs to execute a numbered plan step-by-step with isolated context windows, post-step verification, and commits between steps.
---

# Step-Handoff Skill

Defines the **recursive handoff protocol** used to execute multi-step plans where each step
runs in its own isolated subagent thread. This keeps the context window small, enforces
verification + commit between steps, and ensures the plan runs to completion without an agent
stopping prematurely.

Other skills (e.g. `hl-to-plan`, `comments-fix`) reference this skill instead of duplicating
the protocol.

## When To Use

Load this skill from another skill when that skill produces (or is given) a numbered plan and
needs each step executed in isolation. The calling skill is responsible for:

- Producing the plan file (with one step per section).
- Including the post-step instructions block under each step.
- Performing the **initial handoff** to the subagent that executes Step 1.

This skill defines what goes into those steps and how subagents chain to one another.

## Context Window

- The context window must stay small.
- Therefore each step is executed in its **own subagent thread**.
- **DO NOT EXECUTE MULTIPLE STEPS IN THE SAME THREAD.**

### Choosing the Handoff Mechanism

Decide the handoff mechanism **before writing the plan** and hardcode it into the plan
instructions:

- If `/handoff` is available in the agent (e.g. amp), instruct the agent to hand off to a
  subagent to execute each step.
- Otherwise (e.g. codex), instruct the user to run `/clear` after each step to start a new
  thread, then continue with the next step.

## ⛔ MANDATORY Post-Step Instructions Block

Every step in the plan MUST include this block verbatim (adjust language only if the project
is not Rust — e.g. swap `rs-check` for the equivalent check skill):

```markdown
> **⛔ MANDATORY — After completing this step (NEVER skip, NEVER defer, NEVER batch):**
> 1. Use the `rs-check` skill — fix any issues until it passes cleanly.
> 2. Stage only files added/changed/removed in this step (nothing unrelated).
> 3. Use the `git-commit` skill to commit with a message that describes the step just completed. Respect the commit message format from that skill. Do NOT ask for commit message approval.
> 4. Do NOT move to the next step until rs-check passes and the commit is done.
> 5. **Continuous implementation does NOT override these constraints.** You must stop and execute every post-step block before writing any more code.
```

These are **hard execution constraints, NOT suggestions.** An executing agent MUST NOT skip,
defer, or deprioritize them for any reason — including "prioritizing continuous
implementation" or "maintaining momentum". Skipping a post-step block is a protocol
violation.

### Verification-Only Steps

If a step requires no code change (e.g. a finding that no longer applies after verification):

- Skip post-step items 1–3 (nothing to check / stage / commit).
- Mark the step as completed (✓) and proceed directly to the recursive handoff.
- Write a short explanation of why the step was skipped to
  `/tmp/<skill-name>-skip-<step-number>.md`.

## Recursive Handoff

When handing off, the handoff prompt MUST clearly instruct the receiving subagent that, after
completing its assigned step and all post-step instructions, it must:

1. Mark the step as completed (✓) in the plan file.
2. If there are more steps remaining, hand off to the next subagent to execute the next step.
3. If there are no more steps, the plan is complete and the agent stops.

The handoff prompt MUST very clearly indicate:

- **MARKING COMPLETED STEPS IS VERY IMPORTANT.**
- **EACH AGENT NEEDS TO KEEP HANDING OFF TO THE NEXT ONE UNTIL ALL STEPS ARE COMPLETE. IT
  SHOULD NOT STOP FOR NO REASON.**
- The agent must **NOT** execute any step other than the one it was assigned.
- When handing off, the agent must **switch to the new thread** so the user can see its
  progress (do not run it in the background).

## Plan Conventions

- At the **top of the plan**, indicate that completed steps marked with a checkmark (✓) are
  already done and MUST be skipped.
- Each step gets its own numbered section with a short descriptive title.
- Each step contains all context the executing subagent needs (file paths, line ranges,
  finding text, code patterns) — the subagent will not have the calling agent's context.
- Never batch multiple steps into a single commit.
- Never stage files unrelated to the current step.

## Step Size

Each step should be small enough that exactly **one commit** is needed after completing it.
If you would commit after only part of a step, the step is too big — break it down further.
Use sub-step numbering (`1.1`, `1.2`, …) and include the post-step block (with its own
commit) after each sub-step.

## Commits

NEVER commit prompt or plan files (anything inside `./prompts/**`).

If a file inside `./prompts/**` was staged, **immediately stop and unstage it** before doing
anything else.
