---
name: prompt-process
description: Processes prompt (/tmp/prompt.md) (1) inline (2) handoff (3) new thread.
---

# Prompt Process Skill

Reads and processes prompt files with configurable execution context.

## Usage

Invoke with option number 1-3:

```
prompt-process 1
prompt-process 2
prompt-process 3
```

If no number provided, you will be asked to select one.
If a file path is provided use that as the prompt file instead of the default.

## Options

**Option 1: Same Thread**
- Read prompt file from `/tmp/prompt.md`
- Process and execute in current thread
- Results visible immediately
- Direct execution context

**Option 2: Handoff to Agent**
- Read prompt file from `/tmp/prompt.md`
- Delegate processing to sub-agent via Task tool
- Sub-agent handles execution independently
- Returns summary when complete

**Option 3: New Thread**
- Read prompt file from `/tmp/prompt.md`
- Create new Amp thread for processing
- Full isolated execution context
- Separate conversation thread

## Behavior

- **ALWAYS reread** `/tmp/prompt.md` before processing, even if previously read in session
- If invoked without option number, ask user to provide one
- Respect user's chosen execution mode
- Handle file not found errors gracefully
