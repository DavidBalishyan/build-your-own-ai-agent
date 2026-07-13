# PLAN: growing the starter into a full agent

Right now the agent has the four pieces from Day 5: tools, a loop, a prompt, and
memory. That already counts as an agent, just a small one. Here is what to add to
make it closer to something like Claude Code.

Each item says what it is, why it's worth doing, and which file it touches. Work
down the list roughly in order, since the early ones make the later ones easier.

## Tier 1: make it useful day to day

### Streaming output (done)
The agent streams Claude's text to the screen as it's generated instead of
waiting for the whole turn. `runAgent` in `src/agent.ts` uses
`client.messages.stream(...)`, writes each `stream.on("text", ...)` delta to
stdout, and reads the assembled turn from `await stream.finalMessage()` so the
tool-calling logic is unchanged.

### Real tools (done)
Added `read_file`, `list_dir`, `write_file`, and `run_command` in `src/tools.ts`,
alongside the original demo tools. File paths are scoped to the project
directory (anything resolving outside it is rejected), and `run_command` runs
through the confirmation gate. Still worth adding later: `fetch_url`, a real
weather API, a web search. Once there are a few more, move them into
`src/tools/`.

### Tool confirmation (done)
Tools that change something on disk are listed in `DANGEROUS_TOOLS` in
`src/tools.ts`. Before running one, `runAgent` asks the front-end through a
`confirmTool` handler and skips the tool if the answer is no. The CLI shows a
`y/N` prompt; the GUI turns its input bar into a `[y] yes [n] no` prompt, passing
the decision back over the JSON bridge.

## Tier 2: smarter memory and context

### Better long-term memory
`memory.json` is a flat list of strings today. Give each fact a timestamp and a
category (name, preference, fact), skip duplicates, and add a `forget` tool so the
agent can drop stale entries. This lives in `src/tools.ts`.

### Context-window management
Long chats eventually run past the model's context limit. The API returns a
`usage` field on every response, so track it, and once `messages` gets big,
summarize the oldest turns into a single note and drop them. In `src/agent.ts`.

### Save and resume conversations (done)
`src/sessions.ts` writes the `messages` array to `sessions/<name>.json` and reads
it back, replacing the live array in place so the agent resumes with full
context. Driven by `/save <name>`, `/load <name>`, and `/sessions` in
`src/commands.ts`. `/load` also re-renders the restored transcript so you can see
what came back.

## Tier 3: polish

### Usage and cost readout
Add up `res.usage` across the session and show tokens (and a rough dollar figure)
per turn or behind a `/usage` command. Totals go in `src/agent.ts`, the display in
`src/commands.ts`.

### Retries and error handling
Wrap the API call in a retry with backoff for rate limits and transient failures,
and print a friendly message instead of a stack trace when something breaks. In
`src/agent.ts`.

### Config file
Pull the model list, system prompt, and turn limit out of the code and into
`src/config.ts` (or a JSON file) so you can change them without touching the logic.

### Tests
Cover the pure parts with unit tests: `runTool`, the calculator, memory load and
save, command parsing. Use something like vitest, with files named `src/*.test.ts`.

## Stretch
- Sub-agents: a tool that spins up a second agent to handle a focused subtask.
- MCP servers: connect to tools you didn't write.
- A personality: rewrite `SYSTEM_PROMPT` so the agent has a real character and job.
- Prompt caching: cache the system prompt and tool definitions to cut cost and latency.

## A reasonable order for class
1. ~~Streaming, for the quick payoff.~~ done
2. ~~`read_file` and `write_file`, your first real tools.~~ done
3. ~~Tool confirmation, so those file tools are safe to use.~~ done
4. ~~Save and resume conversations.~~ done
5. Pick a stretch goal and make it your own. ← next
