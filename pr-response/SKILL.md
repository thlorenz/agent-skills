---
name: pr-response
description: "Responds to a PR review comment. Evaluates validity, writes a rebuttal or plans+implements a fix. Use when given a PR comment/discussion URL to address."
---

# PR Response

Evaluates a PR review comment, determines if the concern is valid, and either drafts a
rebuttal or plans and implements a fix.

## Input

The user provides:
- A PR review comment URL (required) - e.g.
  `https://github.com/owner/repo/pull/1136#discussion_r3052850552`

## Workflow

### 1. Fetch the Comment

Parse the comment URL to extract owner, repo, PR number, and comment ID.

Determine which API endpoint to use based on the URL fragment:
- URLs containing `#discussion_r` → PR review comment API
- URLs containing `#issuecomment-` → issue comment API

For PR review comments:
```bash
gh api repos/{owner}/{repo}/pulls/comments/{comment_id}
```

For issue comments:
```bash
gh api repos/{owner}/{repo}/issues/comments/{comment_id}
```

Extract:
- The comment body and author
- The file path and line range the comment refers to (`path`, `line`, `start_line`)
- The parent PR number for context

If the comment is a PR review comment, the API response includes `path` and `line` fields
directly. For issue comments, parse file/line references from the comment body.

Also fetch the parent PR for additional context:

```bash
gh pr view {pr_number} --json title,body,baseRefName
```

### 2. Ensure the PR Branch Is Checked Out

- Get the PR's head branch and compare with the current branch:

```bash
git branch --show-current
gh pr view {pr_number} --json headRefName --jq '.headRefName'
```

- **If they match:** continue to step 3.
- **If they do NOT match:**
  - Check if the staging area is clean: `git diff --cached --quiet`
  - **If clean:** tell the user "Checking out branch `{branch}`", then run
    `git checkout {branch}` and continue.
  - **If not clean:** tell the user the staging area has uncommitted changes and prompt
    for OK. Wait for the user to prepare the directory and check out the correct branch.
    Do NOT proceed until the user types OK and the correct branch is checked out.

### 3. Read the Referenced Code

- Read the referenced file locally at the indicated line range
- Read surrounding context to understand the code fully

### 4. Evaluate the Concern

Review the comment and the referenced code to determine if the concern is valid:
- Does the described problem actually exist in the code?
- Is the suggested change appropriate and correct?

**A) If the concern is NOT valid:**

- Write a clear explanation to `/tmp/pr-response-{pr_number}-{discussion_id}.md`
  explaining why the concern is invalid
- Tell the user: "The concern appears invalid. I've written a response to
  `/tmp/pr-response-{pr_number}-{discussion_id}.md`. Please review it."
- STOP here. Do not proceed further.

**B) If the concern IS valid:**

- Continue to step 5.

### 5. Write a Fix Plan

Write a detailed plan to `/tmp/pr-plan-{pr_number}-{discussion_id}.md` describing how to
fix the issue. Include:
- What the problem is
- Which files need to change
- Step-by-step list of changes
- Any options or trade-offs if applicable

Tell the user: "The concern is valid. I've written a fix plan to
`/tmp/pr-plan-{pr_number}-{discussion_id}.md`. Please review it. Type OK to approve, or
describe how to improve the plan."

### 6. Iterate on the Plan

- If the user provides feedback or selects options, update the plan file and prompt again
- Repeat until the user types OK

DO NOT proceed to implementation without the user typing OK.

### 7. Implement the Fix

Once the user approves the plan, implement the changes described in the plan.

### 8. Verify with rs-check

Load and run the `rs-check` skill to verify the implementation:
- Code formats correctly
- Code compiles and lints cleanly
- All tests pass

If rs-check reveals problems, iterate on the fix until all checks pass.

### 9. Wait for Staging and Commit

- DO NOT stage any files - the user handles staging manually
- DO NOT push any changes
- Tell the user: "Implementation complete. Please stage the changes and type OK."
- Wait for the user to type OK
- Once OK, load and run the `git-commit` skill to commit with a good message

## Red Flags - Stop If You Are Thinking

- "I'll implement the fix without waiting for plan approval"
- "I'll stage the files myself"
- "I'll push the changes"
- "The plan is simple enough, I can skip writing it"
- "I'll commit without waiting for the user to stage"

**Always: fetch comment, evaluate, write rebuttal or plan, wait for OK, implement, rs-check, wait for staging, commit. Never stage or push.**
