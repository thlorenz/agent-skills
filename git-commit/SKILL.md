---
name: git-commit
description: Recommends a commit title based on staged changes and commits. Suggests type and description in format "<type> <description>"
---

# Git Commit Skill

Analyzes staged changes and recommends a conventional commit message title and uses it to commit the staged changes.

## Workflow

1. **Analyze staged changes**
   - Run `git diff --staged` to get the changes
   - Analyze the type of changes (new feature, bug fix, docs, chore, perf, test)
   - If you have further context from previous interactions, use it to improve the recommendation

2. **Generate title recommendation**
   - Format: `<type>: <description>`
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
   - DO NOT COMMIT ANYTHING YOURSELF
   - DO NOT RUN `git add` OR ANY OTHER COMMAND THAT ADDS MORE FILES TO THE STAGING AREA

## Commit Message Types

- **feat**: New features
- **fix**: Bug fixes
- **docs**: Documentation changes
- **chore**: Maintenance tasks, refactors, non-functional changes
- **perf**: Performance improvements
- **test**: Changes to tests
