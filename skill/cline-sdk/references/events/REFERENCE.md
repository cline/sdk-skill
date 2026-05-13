# Events

The Cline SDK emits real-time events during agent execution. Events are used for streaming UI, usage tracking, and observability.

## Event Sources

There are three event layers, each at a different abstraction level:

| Layer | Type | Source |
|-------|------|--------|
| Agent Runtime | `AgentRuntimeEvent` | `agent.subscribe()` or `onEvent` callback |
| ClineCore Session | `CoreSessionEvent` | `cline.subscribe()` |
| Plugin Hooks | Hook callbacks | Plugin `hooks` object |

## Agent Runtime Events (AgentRuntimeEvent)

Low-level events from the browser-compatible agent loop.

### Content Events

```typescript
// Text or tool content started
{ type: "content_start", contentType: "text" | "tool", toolName?: string }

// Text delta or tool input delta
{ type: "content_update", contentType: "text", text: string }
{ type: "content_update", contentType: "tool", toolName: string, input: string }

// Content block finished
{ type: "content_end", contentType: "text" | "tool" }
```

### Iteration Events

```typescript
// Agent loop iteration started/ended
{ type: "iteration_start", iteration: number }
{ type: "iteration_end", iteration: number }
```

### Usage Events

```typescript
{
  type: "usage",
  inputTokens: number,
  outputTokens: number,
  cacheReadTokens?: number,
  cacheWriteTokens?: number,
  cost?: number,
  // Cumulative totals for the session:
  totalInputTokens: number,
  totalOutputTokens: number,
}
```

### Notice Events

```typescript
{ type: "notice", message: string }
```

### Completion Events

```typescript
{
  type: "done",
  status: "completed" | "aborted" | "failed",
  reason?: string,
}

{ type: "error", error: Error }
```

### Done Status Values

| Status | Meaning |
|--------|---------|
| `"completed"` | Agent finished normally |
| `"aborted"` | Cancelled via `abort()` |
| `"failed"` | Unrecoverable error |

## Subscribing to Agent Events

### Via Constructor

```typescript
const agent = new Agent({
  ...config,
  onEvent: (event) => {
    if (event.type === "content_update" && event.contentType === "text") {
      process.stdout.write(event.text)
    }
  },
})
```

### Via subscribe()

```typescript
const unsubscribe = agent.subscribe((event) => {
  switch (event.type) {
    case "content_start":
      if (event.contentType === "tool") console.log(`\n[${event.toolName}]`)
      break
    case "content_update":
      if (event.contentType === "text") process.stdout.write(event.text)
      break
    case "usage":
      console.log(`tokens: ${event.inputTokens} in, ${event.outputTokens} out`)
      break
    case "done":
      console.log(`\nFinished: ${event.status}`)
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
  if (event.type === "content_update" && event.contentType === "text") {
    process.stdout.write(event.text)
  }
})
```

### Usage Tracking

```typescript
let totalCost = 0

agent.subscribe((event) => {
  if (event.type === "usage" && event.cost) {
    totalCost += event.cost
    console.log(`Running cost: $${totalCost.toFixed(4)}`)
  }
})
```

### Tool Call Logging

```typescript
agent.subscribe((event) => {
  if (event.type === "content_start" && event.contentType === "tool") {
    console.log(`Tool started: ${event.toolName}`)
  }
  if (event.type === "content_end" && event.contentType === "tool") {
    console.log(`Tool finished`)
  }
})
```

## See Also

- `../agent/REFERENCE.md` - Agent runtime overview
- `../clinecore/REFERENCE.md` - ClineCore session management
- `../plugins/REFERENCE.md` - Plugin hooks for lifecycle events
- `../production/REFERENCE.md` - Observability in production
