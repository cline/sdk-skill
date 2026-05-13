# Events

The Cline SDK emits real-time events during agent execution. Events are used for streaming UI, usage tracking, and observability.

## Event Sources

There are three event layers, each at a different abstraction level:

| Layer | Type | Source |
|-------|------|--------|
| Agent Runtime | `AgentRuntimeEvent` | `agent.subscribe()` |
| ClineCore Session | `CoreSessionEvent` | `cline.subscribe()` |
| Plugin Hooks | Hook callbacks | Plugin `hooks` object |

## Agent Runtime Events (AgentRuntimeEvent)

Events from the agent loop, emitted via `agent.subscribe()`. Every event includes a `snapshot` field with the current `AgentRuntimeStateSnapshot`.

### Run Lifecycle Events

```typescript
// Run started
{ type: "run-started", snapshot }

// Run completed
{ type: "run-finished", snapshot, result: AgentRunResult }
```

### Turn Events

```typescript
// A turn (iteration) started
{ type: "turn-started", snapshot, iteration: number }

// A turn finished
{ type: "turn-finished", snapshot, iteration: number, toolCallCount: number }
```

### Content Events

```typescript
// Streaming text delta from the assistant
{ type: "assistant-text-delta", snapshot, iteration: number, text: string, accumulatedText: string }

// Complete assistant message received
{ type: "assistant-message", snapshot, iteration: number, message: AgentMessage, finishReason: string }

// A message (user or assistant) was added to the conversation
{ type: "message-added", snapshot, message: AgentMessage }
```

### Usage Events

```typescript
{
  type: "usage-updated",
  snapshot,
  usage: {
    inputTokens: number,
    outputTokens: number,
    cacheReadTokens?: number,
    cacheWriteTokens?: number,
    totalCost?: number,
  },
}
```

### Run Result Status Values

| Status | Meaning |
|--------|---------|
| `"completed"` | Agent finished normally |
| `"aborted"` | Cancelled via `abort()` |
| `"failed"` | Unrecoverable error |

## Subscribing to Agent Events

Use `agent.subscribe()` to listen for streaming events. Register the listener before calling `run()` to avoid missing early events.

```typescript
const unsubscribe = agent.subscribe((event) => {
  switch (event.type) {
    case "assistant-text-delta":
      process.stdout.write(event.text)
      break
    case "usage-updated":
      console.log(`tokens: ${event.usage.inputTokens} in, ${event.usage.outputTokens} out`)
      break
    case "run-finished":
      console.log(`\nFinished: ${event.result.status}`)
      break
  }
})
```

## ClineCore Session Events (CoreSessionEvent)

Higher-level events from ClineCore's session orchestration:

```typescript
type CoreSessionEvent =
  | { type: "started"; sessionId: string }
  | { type: "chunk"; payload: SessionChunkEvent }
  | { type: "tool"; payload: SessionToolEvent }
  | { type: "ended"; payload: SessionEndedEvent }
  | { type: "team_progress"; payload: SessionTeamProgressEvent }
```

### SessionChunkEvent

```typescript
interface SessionChunkEvent {
  type: "text" | "reasoning"
  text: string
  sessionId: string
}
```

### SessionToolEvent

```typescript
interface SessionToolEvent {
  toolName: string
  status: "started" | "completed" | "failed"
  input?: unknown
  output?: unknown
  sessionId: string
}
```

### SessionEndedEvent

```typescript
interface SessionEndedEvent {
  sessionId: string
  finishReason: "completed" | "max_iterations" | "aborted" | "mistake_limit" | "error"
  result?: AgentResult
}
```

### Subscribing to ClineCore Events

```typescript
cline.subscribe((event) => {
  switch (event.type) {
    case "started":
      console.log(`Session started: ${event.sessionId}`)
      break
    case "chunk":
      process.stdout.write(event.payload.text)
      break
    case "tool":
      console.log(`Tool ${event.payload.toolName}: ${event.payload.status}`)
      break
    case "ended":
      console.log(`Finished: ${event.payload.finishReason}`)
      break
    case "team_progress":
      console.log(`Teammate update: ${event.payload}`)
      break
  }
})
```

Filter by session:

```typescript
cline.subscribe(handler, { sessionId: "specific-session-id" })
```

## Common Patterns

### Streaming Text to Console

```typescript
agent.subscribe((event) => {
  if (event.type === "assistant-text-delta") {
    process.stdout.write(event.text)
  }
})
```

### Usage Tracking

```typescript
agent.subscribe((event) => {
  if (event.type === "usage-updated" && event.usage.totalCost) {
    console.log(`Running cost: $${event.usage.totalCost.toFixed(4)}`)
  }
})
```

### Tool Call Logging

```typescript
agent.subscribe((event) => {
  if (event.type === "turn-finished" && event.toolCallCount > 0) {
    console.log(`Turn ${event.iteration} made ${event.toolCallCount} tool calls`)
  }
})
```

## See Also

- `../agent/REFERENCE.md` - Agent runtime overview
- `../clinecore/REFERENCE.md` - ClineCore session management
- `../plugins/REFERENCE.md` - Plugin hooks for lifecycle events
- `../production/REFERENCE.md` - Observability in production
