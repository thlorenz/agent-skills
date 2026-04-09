---
name: issue-from-comment
description: "Creates a GitHub issue from a linked GitHub comment. Use when asked to create an issue from a GitHub comment URL."
---

# Issue from GitHub Comment

Creates a GitHub issue from a GitHub comment, using the `gh` CLI.

## Input

The user provides:
- A GitHub comment URL (required) — e.g. `https://github.com/owner/repo/pull/123#issuecomment-456`
  or `https://github.com/owner/repo/issues/42#issuecomment-789`
- Optional additional context to include in the issue

## Workflow

### 1. Fetch the Comment

Use `gh` to fetch the comment body from the provided URL.

Extract the comment body, author, and the PR/issue it belongs to for context.

For issue comments use:
```bash
gh api repos/{owner}/{repo}/issues/comments/{comment_id}
```

For PR review comments use:
```bash
gh api repos/{owner}/{repo}/pulls/comments/{comment_id}
```

Parse the comment URL to determine which API endpoint to use:
- URLs containing `#issuecomment-` → issue comment API
- URLs containing `#discussion_r` → PR review comment API

### 2. Prompt for Target Repo

DO NOT SKIP THIS STEP. PROMPTING FOR TARGET REPO IS VERY IMPORTANT.

Ask the user which repo to create the issue in.

Say: "Which repo should I create the issue in? (default: magicblock-labs/magicblock-validator)"

If the user just confirms or provides empty input, use `magicblock-labs/magicblock-validator`.

### 3. Prompt for Assignee

DO NOT SKIP THIS STEP. PROMPTING FOR ASSIGNEE IS VERY IMPORTANT.

Ask the user who to assign the issue to.

Say: "Who should be assigned? (default: thlorenz)"

If the user just confirms or provides empty input, use `thlorenz`.

### 4. Compose Issue

Compose a well-structured GitHub issue in Markdown and write it to `/tmp/issue.md`.

The issue file should have this structure:

```markdown
# <concise title describing the issue>

## Context

Originated from [this comment](<original comment URL>) by @<comment author>
on <PR/issue reference>.

## Description

<Rewrite the comment content into a clear issue description. Do not just copy-paste —
restructure it into actionable issue format. Include the user's additional context if provided.>

## Tasks

- [ ] <break down into actionable tasks if possible>
```

Then tell the user: "I've written the issue to `/tmp/issue.md`. Please review and edit it, then type `ok` when ready."

Wait for the user to type `ok` before proceeding.

### 5. Create the Issue

After the user confirms, re-read `/tmp/issue.md` to pick up any edits.

Extract the title from the first `# ` heading line. The body is everything after that first line.

Create the issue:
```bash
gh issue create --repo <repo> --title '<title>' --body-file /tmp/issue.md --assignee <assignee>
```

Note: before running, strip the title line from `/tmp/issue.md` so it isn't duplicated in the body.
Write the body-only content to `/tmp/issue-body.md` and use that as `--body-file`.

### 6. Link Back to Comment

After the issue is created, reply to the original comment with a link to the new issue.

For issue comments:
```bash
gh api repos/{owner}/{repo}/issues/{issue_number}/comments -f body="Created issue: <issue_url>"
```

Where `{issue_number}` is the issue/PR number from the **original** comment URL (not the newly
created issue).

If the reply fails (e.g. permissions), inform the user and provide the issue URL so they can
link it manually.
