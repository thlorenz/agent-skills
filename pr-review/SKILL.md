---
name: pr-review
description: "Reviews GitHub pull requests using gh CLI. Creates a review plan, gets user approval, then posts batched pending review comments with code suggestions. Use when asked to review a PR."
allowed-tools: AskUserQuestion
---

# GitHub PR Review

Reviews GitHub pull requests using `gh api` to create pending reviews with code suggestions.
Always uses pending reviews to batch comments.

## Input

The user provides:
- A PR number or URL (required)
- Optionally: focus areas, event type preference, or specific concerns

## Output

When the skill completes, behavior depends on whether output is piped:

**If piped (e.g., `pr-review <url> | pr-post`):**
- Do NOT post the review to GitHub
- Output only the PR number or URL so downstream skills can pick it up
- Skip section 4 (Post the Review) entirely

**If run standalone:**
- Post the review to GitHub as usual (section 4)
- Tell the user the review has been posted

## Prerequisites

Before starting, verify the gh CLI is available:

```bash
gh --version
```

If gh is not installed, inform the user:

```
The GitHub CLI (gh) is required but is not installed.
Install from: https://cli.github.com/
After installing, authenticate with: gh auth login
```

Do not proceed until gh is installed.

## Workflow

### 1. Verify the PR Branch Is Checked Out

- Check that the PR branch is checked out locally:

```bash
git branch --show-current
gh pr view <PR_NUMBER> --json headRefName --jq '.headRefName'
```

- Compare the current branch with the PR's head branch.
- If they do NOT match do one of the below:
  - check if the staging area is clean (untracked files are ok) and if so, check out the correct branch and continue
  - if the staging area is dirty tell the user: "Please check out the PR branch and respond with OK."
- Do NOT proceed until the branches match.

### 2. Analyze the PR and Prepare a Review Plan

- Fetch PR metadata (title, body, base branch, files changed):

```bash
gh pr view <PR_NUMBER> --json commits,files,title,body,baseRefName
```

- Get the latest commit SHA:

```bash
gh pr view <PR_NUMBER> --json commits --jq '.commits[-1].oid'
```

- Get repository info:

```bash
gh repo view --json owner,name --jq '{owner: .owner.login, name: .name}'
```

- Get the list of changed files and the diff summary:

```bash
gh pr view <PR_NUMBER> --json files --jq '.files[].path'
git diff $(gh pr view <PR_NUMBER> --json baseRefName --jq '.baseRefName')...HEAD --stat
```

- Read the changed files locally to review the code - do NOT download the full diff from GitHub.
  Use the file paths from above to read each file directly from the working tree.

- Analyze the changes and draft review comments.

- Choose the appropriate event type:

| Event Type | When to Use |
|------------|-------------|
| `APPROVE` | Non-blocking suggestions, PR is ready to merge |
| `REQUEST_CHANGES` | Blocking issues that must be fixed (security, bugs) |
| `COMMENT` | Neutral feedback, questions, observations |

- Compute the diff hash for each changed file (used in GitHub links):

```bash
# For each changed file, compute its SHA-256 diff anchor hash:
echo -n "<file_path>" | sha256sum | cut -c1-64
```

  The GitHub link format is:
  - Single line: `https://github.com/<owner>/<repo>/pull/<PR_NUMBER>/files#diff-<hash>R<LINE>`
  - Multi-line: `https://github.com/<owner>/<repo>/pull/<PR_NUMBER>/files#diff-<hash>R<START_LINE>-R<END_LINE>`

- Write the review plan to `/tmp/pr-<PR_NUMBER>-review-plan.md` in this format:

```markdown
# PR #<PR_NUMBER> Review Plan

**Event Type:** COMMENT | APPROVE | REQUEST_CHANGES

**Overall Review Summary (for manual posting):**
<summary message for the review - this will NOT be posted automatically;
use it when submitting your review on GitHub>

---

## Comment 1

**File:** `path/to/file.ts`
**Line:** 20
**Side:** RIGHT
**Link:** [path/to/file.ts#L20](https://github.com/<owner>/<repo>/pull/<PR_NUMBER>/files#diff-<sha256_of_path>R20)

```
<relevant code snippet for context with line numbers prepended>
```

**Comment:**
<comment text>

```suggestion
// suggested replacement code if applicable
```

---

## Comment 2

**File:** `path/to/other-file.ts`
**Line:** 35-40
**Side:** RIGHT
**Link:** [path/to/other-file.ts#L35-L40](https://github.com/<owner>/<repo>/pull/<PR_NUMBER>/files#diff-<sha256_of_path>R35-R40)

```
<relevant code snippet for context with line numbers prepended>
```

**Comment:**
<comment text>

---
```

- Tell the user: "I've written the review plan to `/tmp/pr-<PR_NUMBER>-review-plan.md`. Please review it, remove any comments you don't want, and respond with OK."

### 3. Wait for User Approval

- Wait for the user to respond with OK.
- Do NOT proceed until the user explicitly responds.

### 4. Post the Review (Conditional)

**Check if output is piped:**

```bash
[ -t 1 ]
```

If `[ -t 1 ]` returns true (not piped):
- Proceed with posting the review to GitHub (steps below)
- Tell the user the code comments have been posted

If `[ -t 1 ]` returns false (piped to another command):
- **Skip all GitHub posting steps**
- Output the PR number (or URL) on its own line
- Stop here; let the piped command (e.g., `pr-post`) handle the plan file

**If posting (not piped):**

- Read `/tmp/pr-<PR_NUMBER>-review-plan.md` to get the remaining comments after the user's edits.
- Parse the plan file to extract all remaining comments and the event type.
- Do NOT post the overall review summary - it is included in the plan file for the user to submit manually on GitHub.

**Create a pending review with all code comments and submit it in one call:**

```bash
gh api repos/<owner>/<repo>/pulls/<PR_NUMBER>/reviews \
  -X POST \
  -f commit_id="<COMMIT_SHA>" \
  -f event="<EVENT_TYPE>" \
  -f body="" \
  -f 'comments[][path]=path/to/file.ts' \
  -F 'comments[][line]=<LINE_NUMBER>' \
  -f 'comments[][side]=RIGHT' \
  -f 'comments[][body]=Comment text

```suggestion
// suggested code here
```

Additional explanation...' \
  --jq '{id, state}'
```

For multi-line suggestions, include `start_line`:

```bash
  -F 'comments[][start_line]=<START_LINE>' \
  -F 'comments[][line]=<END_LINE>' \
```

- Tell the user the code comments have been posted and remind them to submit the overall review summary manually using the text from the plan file.

## Syntax Rules

- Use single quotes around parameters with `[]`: `'comments[][path]'`
- Use `-f` for string values
- Use `-F` for numeric values (line numbers)
- Use triple backticks with `suggestion` identifier for code suggestions
- Code suggestions replace the entire line or line range

### Edge Case: Nested Code Blocks in Suggestions

When suggesting changes to markdown or files containing triple backticks, use 4 backticks or tildes:

`````markdown
````suggestion
```javascript
const example = "value";
```
````
`````

## Writing Style

- NEVER use the em dash character (—) anywhere in comments, summaries, or review text. Use a regular hyphen (-), a comma, or rephrase instead.

## Red Flags - Stop If You Are Thinking

- "I'll skip the plan and post directly"
- "Only one comment so no need for the plan file"
- "User already told me what to review so I'll skip the plan step"
- "I'll post the overall summary as the review body"
- "I'll post it and then tell them what I posted"
- "gh is probably installed, no need to check"

**Always: check gh first, write the plan, wait for OK, then post code comments only (no overall summary) using a pending review.**
