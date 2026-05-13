# Cline SDK Skill

Cline SDK reference docs for AI coding assistants. Covers the Agent runtime, ClineCore sessions, custom tools, plugins, events, LLM providers, scheduling, multi-agent teams, and production deployment.

## Install

Add the skill to your AI coding assistant for richer context:

```bash
npx skills add cline/sdk-skill
```

This works with Claude Code, Codex, Cursor, Gemini CLI, GitHub Copilot, Goose, OpenCode, and Windsurf.

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

API Surfaces: Agent (lightweight stateless loop), ClineCore (full runtime with persistence)

Tools: Custom tool creation with Zod/JSON Schema, built-in tools (bash, editor, read_files, search, apply_patch, fetch_web), tool policies

Plugins: Extension system, hooks, lifecycle stages, distribution

Cross-cutting: Events, LLM providers, scheduling, multi-agent teams, production deployment

## Credits and Inspiration

This skill's structure and patterns are inspired by:

- [opentui-skill](https://github.com/msmps/opentui-skill) by [msmps](https://github.com/msmps) -- decision trees, progressive disclosure, and the multi-file reference pattern.

- [cloudflare-skill](https://github.com/dmmulroy/cloudflare-skill) by [Dillon Mulroy](https://github.com/dmmulroy) -- an excellent example of a platform skill.

## License

Apache 2.0 - see [LICENSE](LICENSE)
