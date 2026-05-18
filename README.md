# Cline SDK Skill

Cline SDK reference docs for AI coding assistants. Covers the direct Agent runtime, ClineCore sessions, custom tools, plugins, events, LLM providers, scheduling, multi-agent teams, and production deployment.

For full documentation, visit [docs.cline.bot/sdk](https://docs.cline.bot/sdk).

## Install

Add the skill to your AI coding assistant for richer context:

```bash
npx skills add cline/sdk-skill
```

This works with Cline CLI, Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot, Goose, OpenCode, and Windsurf.

<details>
<summary>Manual install with git</summary>

If you prefer not to use `npx skills`, you can clone the repo and symlink the skill directory into your agent's skill folder.

For Claude Code:

```bash
git clone https://github.com/cline/sdk-skill ~/sdk-skill
mkdir -p ~/.claude/skills
ln -s ~/sdk-skill/skill/cline-sdk ~/.claude/skills/cline-sdk
```

For Codex CLI:

```bash
git clone https://github.com/cline/sdk-skill ~/sdk-skill
mkdir -p ~/.codex/skills
ln -s ~/sdk-skill/skill/cline-sdk ~/.codex/skills/cline-sdk
```

For Amp:

```bash
git clone https://github.com/cline/sdk-skill ~/sdk-skill
mkdir -p ~/.config/amp/tools
ln -s ~/sdk-skill/skill/cline-sdk ~/.config/amp/tools/cline-sdk
```

For Droid (Factory):

```bash
git clone https://github.com/cline/sdk-skill ~/sdk-skill
mkdir -p ~/.factory/skills
ln -s ~/sdk-skill/skill/cline-sdk ~/.factory/skills/cline-sdk
```

For pi-coding-agent:

```bash
git clone https://github.com/cline/sdk-skill ~/sdk-skill
mkdir -p ~/.pi/agent/skills
ln -s ~/sdk-skill/skill/cline-sdk ~/.pi/agent/skills/cline-sdk
```

You can also use project-level paths instead of user-level paths for per-project installation.

</details>

## Structure

```
skill/cline-sdk/
+-- SKILL.md                  # Main manifest + decision trees
+-- references/               # API and concept subdirectories
    +-- agent/                # Agent runtime (4-file pattern)
    +-- clinecore/            # ClineCore full runtime (4-file pattern)
    +-- tools/                # Built-in and custom tools
    +-- plugins/              # Extension system
    +-- events/               # Streaming events
    +-- providers/            # LLM provider configuration
    +-- production/           # Deployment and security
    +-- scheduling/           # Cron and automation
    +-- multi-agent/          # Teams and sub-agents
```

### Decision Trees

The main `SKILL.md` contains decision trees for:
- Choosing an API surface (Agent vs ClineCore)
- Creating and configuring tools
- Setting up plugins and hooks
- Configuring LLM providers
- Streaming events
- Multi-agent coordination
- Scheduling and automation
- Production deployment
- Troubleshooting

## Topics Covered

API Surfaces: Agent (lightweight in-memory loop), ClineCore (full runtime with persistence)

Tools: Custom tool creation with Zod/JSON Schema, built-in tools (`read_files`, `search_codebase`, `run_commands`, `fetch_web_content`, `apply_patch`, `editor`, `skills`, `ask_question`, `submit_and_exit`), tool policies

Plugins: Extension system, hooks, lifecycle stages, distribution

Cross-cutting: Events, LLM providers, scheduling, multi-agent teams, production deployment

## License

[Apache 2.0 © 2026 Cline Bot Inc.](./LICENSE)
