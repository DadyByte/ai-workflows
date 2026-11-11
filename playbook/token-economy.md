# Where the tokens go

> In an agent session, cost follows the number of tool calls more than the size of any one result, so I batch calls, check cheaply before dispatching agents, and cap what searches return.

## Every call re-sends the whole context

Each tool call starts a new request, and each request carries the whole conversation so far. The fixed part of the context is paid again on every call, and it grows as the session goes on.

I measured one session on 2026-08-22. The fixed floor was 65K tokens on the first request and 185K by the end. Across 176 requests, cache reads totalled 22.1M tokens.

The lesson is that one oversized tool result is small next to one extra round trip. Cost scales with the count of calls.

## Batch independent calls

If two searches and a file read do not depend on each other, they go in one message, not three turns. This is written into the global instructions every session loads, under a section on tool-call economy.

Two smaller habits support it:

- Get a command right before running it. Confirm that a flag or a column exists instead of discovering it from an error, because the error costs a full extra request.
- Know the shell. Mine is zsh, which does not split unquoted variables the way bash does.

## Run the cheap local check before dispatching agents

Subagents are useful because they keep heavy reading out of the main thread. They are also easy to send at work that does not need doing.

On 2026-08-22, in the Money Manager expense-filing workflow, filer agents were sent to statements that had already been filed. One offline check against the downloaded expense database would have saved 157K tokens of subagent spend. The rule now is: run the offline check first, and dispatch a filer only for a statement that is not yet filed.

The same design keeps the main thread small. The orchestrator is designed to cost about 200 tokens per statement (a design target, not a measurement), because each statement's PDF text, row lists and verification run inside its own disposable subagent, which returns a receipt of at most 10 lines.

## Cap what searches return

My local search tool, qmd, returns 20 full results by default when asked for JSON. That is about 10 times the tokens of asking for the top 5 with `-n 5`. The retrieval rule every session follows uses `-n 5`.

## Pay for depth only when it is used

- **Skills.** A skill costs about 37 tokens until it is invoked, so a large skill library is cheap to keep loaded.
- **No always-on digest.** Agent OS once injected a digest of about 320 tokens into every session. I retired it on 2026-07-20. The context Agent OS itself adds is now only the memory index and one orienting line. The whole always-on load, with the global instructions, measured 20.5 KB (about 5-6K tokens) on 2026-11-01. Retrieval runs when it is needed.
- **CLI over MCP for search.** I call the search tool from the command line instead of through its MCP server, because MCP tool schemas would cost tokens in every session whether or not they were used.

## Cut overhead, not content

Headless jobs run with settings sources, hooks and MCP servers turned off, and extraction calls get no tools. Each such call still carries about 1.4K tokens of fixed overhead. The nightly transcript job is kept cheap by removing bulk that can be rebuilt (tool output, system reminders, harness overhead) while keeping every user and assistant word (including mid-turn queued messages, which the first version dropped until 2026-11-01). The details are in [the night shift case study](../workflows/03-night-shift.md).

I treat this as a hard line. When a capture step has to get cheaper, it cuts tool output, never conversation content. The incident behind that rule is in [Corrections that became rules](operating-rules.md).

## What these numbers are

| Measure | Value | Date |
|---|---|---|
| Fixed context on the first request of one session | 65K tokens | 2026-08-22 |
| Fixed context by the end of the same session | 185K tokens | 2026-08-22 |
| Requests in that session | 176 | 2026-08-22 |
| Cache reads in that session | 22.1M tokens | 2026-08-22 |
| Subagent spend an offline check would have saved | 157K tokens | 2026-08-22 |
| Search tokens, default JSON output against `-n 5` | about 10x | not dated in source |
| Cost of a skill until invoked | about 37 tokens | not dated in source |
| Fixed overhead per headless extraction call | about 1.4K tokens | not dated in source |

All figures come from my own sessions on my own setup. They show where cost comes from in this kind of work; they are not a benchmark of any tool.

## Related

- [The right model for each step](model-routing.md)
- [Corrections that became rules](operating-rules.md)
