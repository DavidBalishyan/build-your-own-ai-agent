# Day 3: Tools (Giving Your Agent Hands)

## Key idea

**Claude does NOT run my tools.** It sends a message *asking* to use one, with the
inputs it chose. My code runs the real function and sends the result back.
Claude is the brain; my code is the hands.

The flow has four steps: **define, Claude asks, I run, send result back.**

## 1. Define the tool (what Claude sees)

```typescript
const calculatorTool: Anthropic.Tool = {
  name: "calculator",
  description:
    "Performs basic arithmetic. Use this whenever you need an exact math " +
    "answer, since language models are unreliable at mental math.",
  input_schema: {
    type: "object",
    properties: {
      a: { type: "number", description: "First number" },
      b: { type: "number", description: "Second number" },
      operation: { type: "string", enum: ["add", "subtract", "multiply", "divide"] },
    },
    required: ["a", "b", "operation"],
  },
};
```

- The `description` is how Claude decides **when** to use the tool, so write it well.
- `input_schema` is JSON Schema; `enum` constrains choices; `required` lists must-haves.

## 2. Write the real function (what my code runs)

```typescript
function runCalculator(input: any): number {
  const { a, b, operation } = input;
  switch (operation) {
    case "add": return a + b;
    case "subtract": return a - b;
    case "multiply": return a * b;
    case "divide": return a / b;
    default: throw new Error("Unknown operation: " + operation);
  }
}
```

> The definition and the function are two separate things. Keep them in sync.

## 3 & 4. Wire it up and send the result back

```typescript
const res = await client.messages.create({ model, max_tokens: 1024, tools: [calculatorTool], messages });

if (res.stop_reason === "tool_use") {
  const toolUse = res.content.find((c) => c.type === "tool_use");
  if (toolUse?.type === "tool_use") {
    const result = runCalculator(toolUse.input);   // MY code runs it

    messages.push({ role: "assistant", content: res.content });   // Claude's request
    messages.push({
      role: "user",
      content: [{ type: "tool_result", tool_use_id: toolUse.id, content: String(result) }],
    });

    // Call again so Claude turns the raw result into a natural answer
    const final = await client.messages.create({ model, max_tokens: 1024, tools: [calculatorTool], messages });
  }
}
```

Details that tripped me up:
- `res.stop_reason === "tool_use"` is the signal Claude wants a tool.
- The tool request is a `tool_use` **content block** inside `res.content`.
- The result goes back as a `tool_result` block with `role: "user"`, matched by
  `tool_use_id`. `content` must be a **string** (`String(result)`).
- You call Claude a **second time** so it can phrase the final answer.

## Multiple tools

Pass an array; Claude picks. Route by `toolUse.name`:

```typescript
tools: [calculatorTool, weatherTool],
// ...
if (toolUse.name === "calculator") result = String(runCalculator(toolUse.input));
else if (toolUse.name === "get_weather") result = getWeather(toolUse.input);
```

If `stop_reason !== "tool_use"`, Claude just answered directly with no tool needed,
for example "tell me a fun fact about cats."

## My own tool (template)

Recipe for any tool: name it, describe when to use it, describe the inputs, write
the function, then let Claude call it. The one we built in class was a `dice_roller`
(`Math.random()` per die, returns the rolls plus the total).

Next up: wrap this in a loop so Claude can use tools again and again, which makes it
a real agent.
