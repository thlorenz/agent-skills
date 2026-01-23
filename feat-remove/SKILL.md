---
name: removing-features
description: Plans and implements feature removal step-by-step with build/test verification. Use for refactoring code to remove deprecated or unwanted features.
---

# Removing Features Skill

Systematically removes features from a codebase in safe, incremental steps that maintain buildability and passing tests.

## Workflow

### Step 1: Planning

Check if a plan file exists at `_planfile_`. If found, skip to Implementation.

Otherwise, devise a step-by-step plan to remove the feature. Each step must:
- Allow the project to build successfully
- Keep unit tests passing
- Follow dependency order (tests → dependents → feature itself)

Example steps:
1. Remove tests depending on the feature
2. Remove usage in specific modules
3. Remove the feature definition

Write the plan to `_planfile_` with numbered headers for each step:

```markdown
## Step 1: Remove Tests
- <which tests to remove and how>

## Step 2: Remove Usage in <module_name>
- <how to remove/adjust usage>

## Step 3: Remove the Feature
- <explanation of code/files to remove>
```

Request review and confirmation (`ok`) before proceeding.

### Step 2: Implementation

After plan approval, read the plan file again.

- Steps with ✅ were already implemented—skip them
- Ask which step to implement (user responds with step number)
- Implement the step, then:
  - Run build
  - Run unit tests
  - Format code
  - **DO NOT** commit, stage, or push

Request user to stage changes and confirm with `ok`.

### Step 3: Commit

Supply a concise one-line commit message (e.g., `chore: remove usage of x in y`).

After user confirms message with `ok`:
1. Run `git commit -m "<agreed message>"`
2. **DO NOT** push
3. Update `_planfile_`:
   - Add ✅ before step header
   - Add commit info below header:
     ```markdown
     Addressed by <commit_hash>
     > <commit message>
     ```

Return to Step 2 for the next implementation step.

## Key Rules

- **Incremental**: Each step builds and passes tests independently
- **No premature commits**: Only commit after user confirms staging
- **No pushing**: User controls final push
- **Clean tracking**: Plan file documents progress with commit hashes
