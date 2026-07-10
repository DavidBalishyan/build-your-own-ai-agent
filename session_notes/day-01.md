# Day 1: What Are AI Agents?

## Big picture

- **Chatbot vs agent:** a chatbot just talks back. An **agent** can take actions
  (use tools), observe what happened, and keep going until a goal is done.
- **The Agent Loop** is the mental model for the whole course:
  1. **Perceive:** get input or observe the current state
  2. **Think:** the LLM decides what to do next
  3. **Act:** use a tool or take an action
  4. **Observe:** see the result, feed it back in
  Repeat until the task is complete.

## Setup

```bash
cd day-01
npm install
cp .env.example .env   # paste ANTHROPIC_API_KEY into .env
npx tsx src/01-hello-claude.ts
```

- SDK: `@anthropic-ai/sdk`. Uses TypeScript plus `tsx` to run `.ts` directly.
- `import "dotenv/config"` loads `.env` so the key is available.
- `new Anthropic()` **reads `ANTHROPIC_API_KEY` from the env automatically**, so
  you don't pass it in.

## First API call

```typescript
import "dotenv/config";
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-sonnet-4-6",   // which model
  max_tokens: 1024,             // max response length
  messages: [
    { role: "user", content: "What is an AI agent in 2 sentences?" },
  ],
});
```

- The reply is an **array of content blocks**, not a plain string.
  For simple text, grab `response.content[0]`, check `.type === "text"`, read `.text`.

```typescript
const block = response.content[0];
if (block.type === "text") console.log(block.text);
```

## System prompts

`system` sets Claude's personality and rules for the whole call.

```typescript
await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  system: "You are a friendly Armenian tutor. Greet in Armenian, then answer in English. Keep it short.",
  messages: [{ role: "user", content: "What's the best food in Armenia?" }],
});
```

## Takeaways

- Same code with a different `system` prompt gives completely different behavior.
- The content-blocks shape (checking `.type`) matters later, because tool calls
  arrive as *other kinds* of blocks in the same array.

Next up: multi-turn conversations, prompt engineering, structured JSON output.
