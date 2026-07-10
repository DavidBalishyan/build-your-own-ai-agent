# Session Notes

My notes from the TUMO **Build Your Own AI Agent** workshop.
The workshop material lives in [rhovagimian/tumo-ai-agents](https://github.com/rhovagimian/tumo-ai-agents).

## Week 1: the building blocks

| # | Session | Notes |
|---|---------|-------|
| 1 | What Are AI Agents? | [day-01.md](./session_notes/day-01.md) |
| 2 | Talking to an LLM Programmatically | [day-02.md](./session_notes/day-02.md) |
| 3 | Tools: Giving Your Agent Hands | [day-03.md](./session_notes/day-03.md) |
| 4 | The Agent Loop in Code | [day-04.md](./session_notes/day-04.md) |
| 5 | Memory, Context & Planning | [day-05.md](./session_notes/day-05.md) |

## The agent recipe

Every agent is four ingredients:

1. Tools: functions Claude can request
2. Loop: call Claude, run tools, repeat until done
3. Prompt: plan, personality, and rules
4. Memory: remember what matters

## The one thing to remember

Claude has no memory and no hands. My program supplies both. The `messages` array
is the memory, and my code is what actually runs the tools. Claude is only the brain
that decides what to do next.
