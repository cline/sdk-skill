---
description: Load Cline SDK skill and get contextual guidance for building AI agents
---

Load the Cline SDK skill and help with any AI agent development task.

## Workflow

### Step 1: Check for --update-skill flag

If $ARGUMENTS contains `--update-skill`:

1. Determine install location by checking which exists:
   - Local: `.opencode/skill/cline-sdk/`
   - Global: `~/.config/opencode/skill/cline-sdk/`

2. Run the appropriate install command:
   ```bash
   # For local installation
   curl -fsSL https://raw.githubusercontent.com/cline/sdk-skill/main/install.sh | bash

   # For global installation
   curl -fsSL https://raw.githubusercontent.com/cline/sdk-skill/main/install.sh | bash -s -- --global
   ```

3. Output success message and stop (do not continue to other steps).

### Step 2: Load cline-sdk skill

```
skill({ name: 'cline-sdk' })
```

### Step 3: Identify task type from user request

Analyze $ARGUMENTS to determine:
- API surface needed (Agent lightweight or ClineCore full runtime)
- Task type (new agent, custom tools, plugins, streaming, deployment, scheduling, multi-agent)

Use decision trees in SKILL.md to select correct reference files.

### Step 4: Read relevant reference files

Based on task type, read from `references/<area>/`:

| Task | Files to Read |
|------|---------------|
| Create a simple agent | `agent/REFERENCE.md` + `agent/api.md` |
| Build a full app with sessions | `clinecore/REFERENCE.md` + `clinecore/api.md` |
| Create custom tools | `tools/REFERENCE.md` |
| Add plugins or hooks | `plugins/REFERENCE.md` |
| Configure LLM providers | `providers/REFERENCE.md` |
| Stream events | `events/REFERENCE.md` |
| Deploy to production | `production/REFERENCE.md` |
| Schedule recurring agents | `scheduling/REFERENCE.md` |
| Multi-agent orchestration | `multi-agent/REFERENCE.md` |
| Debug/troubleshoot | `agent/gotchas.md` or `clinecore/gotchas.md` |
| Common patterns | `agent/patterns.md` or `clinecore/patterns.md` |

### Step 5: Execute task

Apply Cline SDK-specific patterns and APIs from references to complete the user's request.

### Step 6: Summarize

```
=== Cline SDK Task Complete ===

API Surface: <Agent | ClineCore>
Files referenced: <reference files consulted>

<brief summary of what was done>
```

<user-request>
$ARGUMENTS
</user-request>
