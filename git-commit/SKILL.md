---
name: git-commit
description: Recommends a commit title based on staged changes and commits. Suggests type and description in format "<type> <description>"
---

# Git Commit Skill

Analyzes staged changes and recommends a conventional commit message title and uses it to commit the staged changes.

If you cannot commit due to sandboxing limitations or similar just print the commit message
instead and then wait for the user to perform the commit and respond with OK.

There are two distinct flows depending on context:

## Flow A: Standalone (user invokes skill directly)

1. **Analyze staged changes**
   - Run `git diff --staged` to get the changes
   - Analyze the type of changes (new feature, bug fix, docs, chore, perf, test)
   - If you have further context from previous interactions, use it to improve the recommendation

2. **Generate title recommendation**
   - Format: `<type>: <description>` (NOTE the colon after the type)
   - Keep total length ≤ 50 characters
   - Types: `feat`, `fix`, `docs`, `chore`, `perf`, `test`

3. **Request approval**
   - Present the recommended title
   - Ask user to approve or modify

4. **Handle user response**
   - If user types `ok`: use the recommended title
   - If user types only a type (e.g., `feat`, `chore`): keep the description, change only the prefix
   - If user provides custom text: use the custom title provided
   - Then commit with the final title `git commit -m "<final title>"`
   - DO NOT STAGE ANYMORE FILES
   - DO NOT RUN `git add` OR ANY OTHER COMMAND THAT ADDS MORE FILES TO THE STAGING AREA

## Flow B: Part of a step-by-step plan

Use this flow when the skill is invoked as a step within a larger plan being implemented incrementally (e.g., after completing a plan step that says "commit"). However, if the plan explicitly instructs to stage and commit manually, use Flow A instead.

1. **Stage all changes**
   - Run `git add -A` to stage all changed, added, and removed files

2. **Analyze staged changes**
   - Run `git diff --staged` to get the changes
   - Analyze the type of changes (new feature, bug fix, docs, chore, perf, test)
   - Use context from the plan step being committed to improve the message

3. **Generate title and commit immediately**
   - Format: `<type>: <description>` (NOTE the colon after the type)
   - Keep total length ≤ 50 characters
   - Types: `feat`, `fix`, `docs`, `chore`, `perf`, `test`
   - Run `git commit -m "<title>"` immediately — do NOT ask for approval

## Commit Message Types

- **feat**: New features
- **fix**: Bug fixes
- **docs**: Documentation changes
- **chore**: Maintenance tasks, refactors, non-functional changes
- **perf**: Performance improvements
- **test**: Changes to tests

## Exclude

NEVER COMMIT prompt/plan files, i.e. anything inside a ./prompts directory.

## Description

Only include a commit message aside from the title when it is a very large change, otherwise
leave it empty.

NEVER LIST co-author or agent thread id, i.e. the below should NEVER BE INCLUDED:

Amp-Thread-ID: https://ampcode.com/threads/T-019dce36-f47e-762c-b869-c102743093b0
Co-authored-by: Amp <amp@ampcode.com>

THIS IS VERY IMPORTANT.
STOP IF THE COMMIT MESSAGE INCLUDES EITHER OF THE ABOVE OR SIMILAR, i.e.:

Amp-Thread-ID: ...
Co-authored-by: ...
