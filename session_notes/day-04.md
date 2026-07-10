# Day 4: The Agent Loop in Code

## Key idea

An agent is just a **loop**:

```
while (true):
  call Claude
  if Claude is done -> return the answer
  else -> run the tools it asked for, send results back, loop
```

That's the whole thing. Everything else is detail.

## The core loop

```typescript
async function runAgent(userMessage: string): Promise<string> {
  const messages: Anthropic.MessageParam[] = [{ role: "user", content: userMessage }];

  while (true) {
    const res = await client.messages.create({ model, max_tokens: 1024, tools: allTools, messages });
    messages.push({ role: "assistant", content: res.content });

    // Claude is done
    if (res.stop_reason !== "tool_use") {
      const text = res.content.find((c) => c.type === "text");
      return text && text.type === "text" ? text.text : "";
    }

    // Claude wants tools, so run ALL of them and collect results
    const toolResults: Anthropic.ToolResultBlockParam[] = res.content
      .filter((b): b is Anthropic.ToolUseBlock => b.type === "tool_use")
      .map((b) => ({
        type: "tool_result",
        tool_use_id: b.id,
        content: runTool(b.name, b.input),
      }));

    messages.push({ role: "user", content: toolResults });
  }
}
```

- `stop_reason` is what tells us whether to keep going (`"tool_use"`) or finish.
- Claude can ask for **several tools at once**, so loop over *all* the `tool_use`
  blocks and return one `tool_result` per request in a single `user` message.

## The dispatcher (tools.ts)

Keep tools in one shared file. A **dispatcher** maps a tool name to the real
function. To add a tool: write its function, add it to `allTools`, add a `case`.

```typescript
export const allTools: Anthropic.Tool[] = [calculatorTool, weatherTool, dictionaryTool];

export function runTool(name: string, input: any): string {
  switch (name) {
    case "calculator":  return runCalculator(input);
    case "get_weather": return getWeather(input);
    case "dictionary":  return lookupWord(input);
    default:            return "Error: unknown tool " + name;
  }
}
```

## Guardrails (safe agent)

A raw `while (true)` is fragile. Two guardrails make it trustworthy:

1. **Max turns**, so it can never loop forever.
2. **try/catch around each tool.** A broken tool shouldn't crash the agent, so send
   the error text back to Claude and it can often recover or try another way.

```typescript
async function runAgent(userMessage: string, maxTurns = 10): Promise<string> {
  let turns = 0;
  while (turns < maxTurns) {
    turns++;
    // ...call Claude, check stop_reason...
    for (const block of res.content) {
      if (block.type === "tool_use") {
        let result: string;
        try {
          result = runTool(block.name, block.input);
        } catch (e: any) {
          result = "Tool error: " + e.message;   // feed error back, don't crash
        }
        toolResults.push({ type: "tool_result", tool_use_id: block.id, content: result });
      }
    }
    messages.push({ role: "user", content: toolResults });
  }
  return "Stopped: hit the max turn limit.";
}
```

## Watching it think

The `02-multi-step` script adds logging inside the loop so you can watch each step:
which tool Claude picked, the inputs, the result, then the next decision. Handy for
debugging multi-step tasks like *"What is 25 * 4, and the weather in London?"*

Next up: memory, context, and planning, the last polish before Week 2.
