# Day 2: Talking to an LLM Programmatically

## Key idea

**Claude has no memory between API calls.** The `messages` array *is* the memory,
and **my program** keeps it alive by appending every new message (both mine and
Claude's replies).

## Multi-turn conversation

Keep a growing `messages` array. After each reply, push it back before the next turn.

```typescript
const messages: Anthropic.MessageParam[] = [
  { role: "user", content: "Hi! My name is Ani." },
];

let res = await client.messages.create({ model, max_tokens: 1024, messages });
let reply = res.content[0].type === "text" ? res.content[0].text : "";

// Save Claude's reply so it "remembers" next turn
messages.push({ role: "assistant", content: reply });

messages.push({ role: "user", content: "What's my name?" });
// -> now Claude can answer "Ani", because the history contains it
```

> Proof: comment out the `push({ role: "assistant", ... })` line and Claude forgets
> your name. The memory isn't magic. It's just us keeping the list.

## Interactive chat loop (terminal)

Use `readline/promises` and a `while (true)` loop. Push user input, call Claude,
push the reply, repeat.

```typescript
import * as readline from "readline/promises";
const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
const messages: Anthropic.MessageParam[] = [];

while (true) {
  const userInput = await rl.question("You: ");
  if (userInput.trim().toLowerCase() === "exit") break;

  messages.push({ role: "user", content: userInput });
  const res = await client.messages.create({ model, max_tokens: 1024, messages });
  const reply = res.content[0].type === "text" ? res.content[0].text : "";
  console.log("Claude: " + reply);
  messages.push({ role: "assistant", content: reply });
}
rl.close();
```

## Prompt engineering

- Be specific, show examples, and state the exact output format you want.

### Few-shot prompting

Instead of long instructions, **show a couple of examples** and let Claude match
the pattern:

```
Classify the sentiment as POSITIVE or NEGATIVE.

Review: "This movie was amazing!"
Sentiment: POSITIVE

Review: "Total waste of time."
Sentiment: NEGATIVE

Review: "Best pizza I've ever had!"
Sentiment:
```

Claude completes with `POSITIVE`. Keep `max_tokens` small for one-word answers.

## Structured output (JSON I can use in code)

Two parts:
1. A **system prompt** that specifies the exact shape and says *"ONLY valid JSON,
   no markdown, no extra text."*
2. `JSON.parse()` on the reply, **always wrapped in try/catch** so a stray word
   from Claude doesn't crash the program.

```typescript
system:
  "You extract event details. Respond with ONLY valid JSON, no extra text. " +
  'Shape: { "title": string, "date": string, "location": string }',
// ...
try {
  const data = JSON.parse(text);
} catch {
  console.error("Could not parse JSON:", text);
}
```

## Takeaways

- I own the conversation state; the SDK is stateless.
- Being strict about format in the prompt is what makes the output reliable and
  parseable.

Next up: give Claude **tools**, so it can actually DO things.
