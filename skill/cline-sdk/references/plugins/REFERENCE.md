# Plugins

Plugins package reusable capabilities (tools, hooks, rules, commands) into a single distributable unit.

## Plugin Structure

```typescript
import type { AgentPlugin } from "@cline/sdk"

const myPlugin: AgentPlugin = {
  name: "my-plugin",
  manifest: {
    capabilities: ["tools", "hooks"],
  },
  setup(api, ctx) {
    api.registerTool(myTool)
    api.registerRule({ content: "Always explain your reasoning." })
  },
  hooks: {
    beforeRun() {
      console.time("agent-run")
    },
    beforeTool({ toolCall }) {
      console.log(`Calling: ${toolCall.toolName}`)
    },
    afterRun({ result }) {
      console.timeEnd("agent-run")
      console.log(`${result.iterations} iterations`)
    },
  },
}
```

## Plugin Capabilities

Declare what your plugin provides in `manifest.capabilities`:

| Capability | What It Does |
|-----------|--------------|
| `"tools"` | Registers custom tools |
| `"hooks"` | Intercepts lifecycle stages |
| `"commands"` | Adds slash commands |
| `"rules"` | Adds system prompt rules |
| `"messageBuilders"` | Transforms messages before sending |
| `"providers"` | Registers custom LLM providers |
| `"automationEvents"` | Declares custom automation event types |

## Extension API (setup)

The `setup(api, ctx)` function receives the extension API for registering contributions:

```typescript
setup(api: AgentExtensionApi, ctx: PluginSetupContext) {
  api.registerTool(tool)           // register a tool
  api.registerCommand(command)     // register a slash command
  api.registerRule(rule)           // add a system prompt rule
  api.registerMessageBuilder(fn)  // transform outgoing messages
  api.registerProvider(provider)   // register an LLM provider
  api.registerAutomationEventType(eventType) // declare event types
}
```

### PluginSetupContext

```typescript
interface PluginSetupContext {
  session?: AgentExtensionSessionContext
  client?: ClientContext
  user?: UserContext
  workspaceInfo?: WorkspaceInfo
  automation?: AgentExtensionAutomationContext
  logger?: BasicLogger
  telemetry?: ITelemetryService
}
```

## Hook Stages

Plugins can intercept these lifecycle stages:

```typescript
hooks: {
  beforeRun?(context): AgentStopControl | undefined
  afterRun?(context): void
  beforeModel?(context): AgentBeforeModelResult | undefined
  afterModel?(context): AgentStopControl | undefined
  beforeTool?(context): AgentBeforeToolResult | undefined
  afterTool?(context): AgentAfterToolResult | undefined
  onEvent?(event: AgentRuntimeEvent): void | Promise<void>
}
```

### Hook Policies

For file-based hooks (`.cline/hooks/`), you can set execution policies:

| Policy | Options |
|--------|---------|
| `mode` | `"blocking"` or `"async"` |
| `timeoutMs` | Maximum execution time |
| `retries` | Number of retry attempts |
| `retryDelayMs` | Delay between retries |
| `failureMode` | `"fail_open"` or `"fail_closed"` |
| `maxConcurrency` | Max parallel executions |

## Using Plugins with Agent

```typescript
import { Agent } from "@cline/sdk"

const agent = new Agent({
  providerId: "anthropic",
  modelId: "claude-sonnet-4-6",
  tools: [],
  plugins: [myPlugin, anotherPlugin],
})
```

## Using Plugins with ClineCore

### Programmatic Loading

```typescript
const cline = await ClineCore.create({ clientName: "my-app" })

await cline.start({
  prompt: "...",
  config: {
    providerId: "anthropic",
    modelId: "claude-sonnet-4-6",
    tools: [],
  },
  extensions: [myPlugin],
})
```

### File-Based Discovery

Place plugin files in:
- Global: `~/.cline/plugins/`
- Workspace: `.cline/plugins/`

ClineCore auto-discovers and loads them.

### Plugin Path Loading

```typescript
await cline.start({
  config: {
    pluginPaths: ["./plugins/my-plugin.ts"],
    extensionLoading: "direct",  // or "isolated" for subprocess
  },
})
```

## Extension Loading Modes

| Mode | Description |
|------|-------------|
| `"direct"` | Loaded in-process, faster, shares memory |
| `"isolated"` | Spawned as subprocess, safer, separate memory |

## Plugin Design Guidelines

1. Use factory functions for configurable plugins:

```typescript
export function createGitHubPlugin(config: { token: string }): AgentPlugin {
  return {
    name: "github",
    manifest: { capabilities: ["tools", "hooks"] },
    setup(api) {
      api.registerTool(createListIssuesTool(config.token))
      api.registerTool(createCreatePRTool(config.token))
    },
    hooks: {
      afterRun({ result }) {
        trackUsage(result)
      },
    },
  }
}
```

2. Keep `setup()` synchronous and fast
3. Register tools in `setup()`, not in hooks
4. Use hooks for observation (logging, metrics), not behavior modification
5. Handle errors in hooks gracefully to avoid crashing the agent

## Distribution

### Via npm

Create a package with a `cline` field in `package.json`:

```json
{
  "name": "@myorg/cline-plugin-github",
  "cline": {
    "plugins": ["./dist/index.js"]
  }
}
```

Install with:
```bash
cline plugin install --npm @myorg/cline-plugin-github
```

### Via Git

```bash
cline plugin install --git https://github.com/myorg/cline-plugin-github
```

### Via Local Path

```bash
cline plugin install ./path/to/plugin
```

## Plugin Examples from SDK

The SDK repo includes these example plugins:

| Plugin | Description |
|--------|-------------|
| `weather-metrics.ts` | Tool registration + lifecycle metrics |
| `mac-notify.ts` | macOS Notification Center alerts |
| `custom-compaction.ts` | Custom message compaction |
| `background-terminal.ts` | Detached shell job management |
| `automation-events.ts` | Plugin-emitted automation events |
| `gitignore-read-files-guard.ts` | File access policy enforcement |
| `web-search.ts` | Web search via Exa API |
| `typescript-lsp/` | TypeScript Language Service tools |
| `agents-squad/` | Multi-agent team orchestration |

## See Also

- `../tools/REFERENCE.md` - Tool creation
- `../events/REFERENCE.md` - Event system
- `../agent/REFERENCE.md` - Using plugins with Agent
- `../clinecore/REFERENCE.md` - Using plugins with ClineCore
