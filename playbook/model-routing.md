# The right model for each step

> The top model plans and reviews, cheaper models do the work, and the rule is that every agent call names its model, so nothing inherits the most expensive one by accident.

## The rule

I split agent work into two kinds.

- **Thinking work** goes to the top model: planning, architecture, writing the brief for a subagent, reading its report, and reviewing the result.
- **Doing work** goes to the cheapest model that can do it well. Sonnet or Haiku handle mechanical and routine steps such as filing, commits, log entries and liveness checks. Opus is used for a step that is genuinely hard, or after a Sonnet pass has failed.

The top model then checks the output and relays it. It does not do the typing itself.

On 2026-11-04 I reviewed a batch of routine work that had run entirely on the two most expensive models, including simple file and commit steps run directly by the top model. My review: cheaper models do delegated and trivial work; the top model only reviews it. That became a standing rule in the agent's memory, whose index line every session loads.

## Name the model on every call

A workflow agent() call that is not given a model runs on the session's own model. In a session driven by the top model, that means every builder silently becomes a top-model builder.

This happened on 2026-11-10. A build workflow in another project dispatched its builder agents without a `model` setting, so all of them ran on the top model. The run died at a usage limit before it finished, and I had to tell it to try again. The agent's own reading afterwards was that the missing `model` setting caused it; I only saw the failure and asked for a retry.

The second run set a model on every `agent()` call: Sonnet for the builders and the checkers, and Opus for an adversarial review of the sign-in code. That run finished.

The rule now reads: set `model` on every Agent tool call and on every `agent()` call inside a workflow script. It is a feedback memory whose index line every session loads.

## What the split looks like in practice

| Step | Model | Why |
|---|---|---|
| Plan, brief, review | Top model | Judgement and cross-checking |
| Expense statement filing, one subagent per statement | Sonnet | Mechanical extraction and filing |
| Nightly transcript extraction, one call per 40K-token chunk | Sonnet at low effort | High volume, verified by code afterwards |
| Adversarial review of security-sensitive code | Opus | Hard reasoning where a miss is costly |

Two examples from my own systems:

- **Money Manager filing.** In my expense-filing skill, the main thread only orchestrates, and by design that costs about 200 tokens per statement (a design figure, not a measured result). Each statement goes to one filer subagent whose definition pins `model: sonnet` and a short tool list. The earlier version did everything in one chat on the most expensive model, and one statement cycle cost about an entire session budget.
- **The night shift.** The Agent OS night shift splits a changed transcript into 40K-token chunks only when it is longer than that, and makes one trimmed Sonnet call per chunk at low effort. Code then writes an item as fact only if its quote is found in the transcript; the rest go to a proposals file marked unverified, so the cheaper model's output is checked rather than trusted.

## Choose a recurring job's model by a trial, not by price

For a job that runs every night, the model choice compounds. I do not pick it by price alone. I score candidate models against a gold set built from Opus wrap-ups of five sessions before choosing. That gold set is a model's summary, not ground truth. The bake-off for the night shift is written up in [the night shift case study](../workflows/03-night-shift.md).

The 2026-10-24 night-shift review that set this rule also rejected a cheaper plan that dropped conversation content to save tokens. The cost cut has to come from somewhere else; see [Where the tokens go](token-economy.md).

## Limits

- This is one person's setup, and the routing rules come from my own runs, not from a controlled comparison across many users.
- The 2026-11-10 diagnosis is the agent's reading of a single failure, not a measured result.
- Model names and their relative strengths change. The rule that lasts is "decide the model per step and write it down", not any particular mapping.

## Related

- [Where the tokens go](token-economy.md)
- [Corrections that became rules](operating-rules.md)
