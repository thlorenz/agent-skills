---
name: pr-post
description: "Post-processes pr-review output to humanize AI-generated comments. Reads the review plan, rephrases comments to sound natural and dev-like, and appends humanized versions inline. Use after pr-review via pipe: pr-review <url> | pr-post"
argument-hint: "<PR_NUMBER or PR_URL>"
---

# PR Post-Process (Humanize Review Comments)

Reads the review plan produced by `pr-review` and rephrases each comment to sound like a
human developer wrote it.

## Input

Receives a PR number or URL either:
- Piped from `pr-review` (the skill passes through its argument)
- Directly as an argument

Extract the PR number from whatever form is provided (URL or plain number).

## Workflow

### 1. Read the Review Plan

- Read `/tmp/pr-<PR_NUMBER>-review-plan.md`
- If the file does not exist, tell the user to run `pr-review` first and stop.

### 2. Parse Comments

- Identify each `## Comment N` section in the plan.
- Extract the **Comment:** block text from each section.
- Preserve all other fields (File, Line, Side, Link, suggestion blocks) as-is.

### 3. Humanize Each Comment

Rephrase every comment following these rules:

- **Casual dev tone** - write like a teammate leaving a quick note on a PR, not a formal review.
  Think Slack message about code, not an essay.
- **Keep it short** - strip filler words, hedging ("perhaps", "it might be worth considering"),
  and unnecessary politeness. Get to the point.
- **Retain all substance** - every technical point, concern, and suggestion must survive. Do not
  drop information.
- **Keep code references** - file paths, line numbers, variable names, function names, and
  inline code must stay exactly as-is.
- **Keep suggestion blocks** - ```` ```suggestion ```` blocks are never modified.
- **No AI tells** - avoid patterns that scream AI:
  - No "Great job overall, but..." openers
  - No "I noticed that..." or "It appears that..."
  - No bullet-point-heavy restructuring of a short thought
  - No "Consider..." as a sentence starter (once in a while is fine, every comment is not)
  - No overly balanced "X is good, however Y" formulas
  - No em dash (—) usage
- **Vary sentence structure** - mix short and longer sentences. Start some with the issue
  directly, others with context. Don't follow a template.
- **Use contractions** - "doesn't", "won't", "isn't" etc. Formal uncontracted forms are a
  giveaway.

### 4. Write Humanized Version Inline

For each `## Comment N` section, append a `### Humanized` subsection directly below the
original comment text (but before the `---` separator), containing the rephrased version.

- DO NEVER USE — em dashes since they are a dead giveaway of AI-generated text. Stick to simple punctuation.
- each time a sentence starts put it on a new line so I can easily paste into github


Example output structure:

```markdown
## Comment 1

**File:** `src/handler.ts`
**Line:** 42
**Side:** RIGHT
**Link:** [src/handler.ts#L42](https://github.com/...)

**Comment:**
The error handling here doesn't account for the case where `response` is undefined.
This could lead to a runtime TypeError when accessing `response.status`.

```suggestion
if (response?.status === 200) {
```

### Humanized

This'll blow up if `response` is undefined - you'd get a TypeError on `.status`.
Optional chaining fixes it:

```suggestion
if (response?.status === 200) {
```

---

### 5. Save and Report

- Write the updated plan back to `/tmp/pr-<PR_NUMBER>-review-plan.md`.
- Tell the user: "Humanized comments added to `/tmp/pr-<PR_NUMBER>-review-plan.md`.
  Review the `### Humanized` sections, then when posting pick whichever version you prefer
  for each comment."

## Red Flags - Stop If You Are Thinking

- "I'll rewrite the original comments instead of appending"
- "I'll create a separate file"
- "The suggestion block needs rephrasing too"
- "I should make the tone more professional"
