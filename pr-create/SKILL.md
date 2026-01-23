---
name: pr-create
description: "Create a pull request based on changes from master branch."
---

- _branch_ refers to target branch which is master or main (whichever is used in this repo)

# Create a PR

## Goal

I want to submit a PR for the current branch onto the _branch_ branch.

I pulled the latest changes for both branches and based the current branch on the _branch_
branch. Thus you can use `git diff` and `git log` in order to see which changes were applied.

## Workflow (checklist)

### 1. Closed Issues

Ask me for issues that this PR closes, I will provide them via comma separated numbers, or
`none` if there is none.

You will add these issues as part of the PR summary section via the _CLOSES_ keyword.
Example:

```markdown
## Summary

<summary of the PR>
CLOSES: #123, #456
```

## 2. Gather Changes

Use `git diff _branch_...HEAD` and `git log _branch_...HEAD` to gather the changes made in this branch compared to
the _branch_ branch.

Do this in the current directory (the one this agent was invokd in).

## 3. PR Body

Please recommend a content for the pull request body based on the changes made in this branch.
The pull request body should have two sections as outlined in the ./references/pr-description-template.md:

Here is an example of a pull request message:

```markdown
## Summary

Adding a full fledged committor service to support the following kind of commits:

- commits up to 10Kb as long as the size increase per commit is <=1Kb
- max accounts per bundle is 3 without lookup tables and  12 with lookup tables
- commit results are persisted into a SQlite database in the ledger directory which makes it
easy to manually retry them or do this automatically (next committor version will support this)

## Details

<General details here>

### magicblock-rpc-client

<specific details>

### magicblock-table-mania

<specific details>
```

When going into details DO NOT EXPLAIN ALL CODE CHANGES TO THE LETTER, i.e. something was added
to a constructor or things like that. Devs can see that from the diff. Instead explain _what_ I
did, not _how_ I did it.

When you are done, write the results to the `/tmp/pr-body.md` file and ask me to edit it and then to type `ok`.
I will review the body and update it and type `ok` when I am done, only then should you go to the
next step (PR Title).


## 4. PR Title

Please recommend a title for the pull request based on the changes made in this branch.

The pull request title should be short and descriptive, ideally no more than 50 characters.

It should have this format:
`<type>: <description>`

Where type is one of the following:

- `feat`: for new features
- `fix`: for bug fixes
- `docs`: for documentation changes
- `chore`: for maintenance tasks, refactors and other changes that don't change the
functionality
- `perf`: for performance improvements
- `test`: for changes to tests

Then ask me to approve the title. If I type `ok` use it, otherwise I will provide a title you
should use.

## 5. Submit PR

Once you have the PR message and title, submit the pull request using the following command:

```bash
gh pr create --title='<title>' --body-file <path to tmp file> --base _branch_
```

Where `<title>` is the title we decided on and `<path to tmp file>` is the path to the file
containing the PR body.
