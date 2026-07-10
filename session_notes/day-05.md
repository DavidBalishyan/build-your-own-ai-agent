# Day 5: Memory, Context & Planning

Last Week 1 session. We put together the **complete agent template**, which is the
starting point for Week 2.

## Two kinds of memory

- **Short-term memory** is the `messages` array. It lives only while the program runs.
- **Long-term memory** is a **file** (`memory.json`) that survives restarts.
- **Context window:** Claude can read a lot, but not infinitely, so you can't just
  keep stuffing everything in forever. Store facts and inject the relevant ones.

## File-based long-term memory

Save facts to a JSON file, load them back, and **inject them into the system prompt**.

```typescript
import * as fs from "fs";
const MEMORY_FILE = "memory.json";

function loadMemory(): string[] {
  if (!fs.existsSync(MEMORY_FILE)) return [];
  return JSON.parse(fs.readFileSync(MEMORY_FILE, "utf-8"));
}

function remember(fact: string): void {
  const memory = loadMemory();
  memory.push(fact);
  fs.writeFileSync(MEMORY_FILE, JSON.stringify(memory, null, 2));
}

// inject into the system prompt so Claude "remembers" across runs
const facts = loadMemory();
const system = "You are a friendly tutor. Things you remember: " + facts.join(" ");
```

On the next run Claude greets the student by name because the facts persisted.
Delete `memory.json` to start fresh.

## Planning is just a system prompt

The lesson that stuck: to make the agent plan, the code didn't change at all. Only
the prompt did.

```typescript
const SYSTEM = `You are a helpful task agent with tools.

IMPORTANT — how to work:
1. First, briefly outline your plan in 1-2 sentences.
2. Then use your tools one step at a time.
3. After each tool result, decide the next step.
4. When the task is fully done, give a clear final answer.

Always think about WHICH tool fits before calling it.`;
```

## The complete agent (`agent.ts`): fork this for Week 2

Brings together everything from Week 1:
- **tools** (from `tools.ts`) for the abilities
- **the safe agent loop** for the engine (max-turns plus try/catch)
- **a planning system prompt** for the brain's instructions
- **file-based memory** so it remembers across runs
- **an interactive chat loop** (`readline`) so you can talk to it in the terminal

Memory gets folded into the prompt each turn:

```typescript
function buildSystemPrompt(): string {
  const facts = loadMemory();
  if (facts.length === 0) return SYSTEM_PROMPT;
  return SYSTEM_PROMPT + "\n\nThings you remember: " + facts.join(" ");
}
// runAgent(...) calls system: buildSystemPrompt() on every turn
```

## The agent recipe (all four ingredients together)

1. Tools: functions Claude can request
2. Loop: call, run tools, repeat
3. Prompt: plan, personality, and rules
4. Memory: remember what matters

## Make it my own (Week 2)

1. Edit `SYSTEM_PROMPT` in `agent.ts` to give the agent a job and a personality.
2. Add tools in `tools.ts`: function, definition, and dispatcher case.
3. Run `npm run agent` and chat with it.

## Week 2 project ideas

_(to fill in)_. Fork the Day 5 agent and build something of my own.
