# Corrections that became rules

> Nine rules my agents now follow, each born from a dated mistake, and each enforced by something stronger than a note wherever I could manage it.

When I correct an agent, the correction becomes a feedback memory in one store (`~/.claude/projects/<project>/memory/`) whose index every session loads. Where a memory alone kept failing, I moved the rule into a hook, a check or a scheduled job. Every defect found in three audits of my setup had the same shape: a rule correct in the documentation and not true in practice.

## 1. Verify the surface the user opens, not the source you edited

**Incident (2026-09-15).** The agent added a section to a local dashboard, passed its self-test and reported it live. I could not see it. The page came from a long-running process started the evening before, so it could not serve the new code. A first check had looked green only because the phrase it searched for already appeared elsewhere on the page. Later the same day a button rendered correctly and still opened the wrong page when clicked: the open action resolved a file path against the page address, so it showed the dashboard again instead of the board. The next day another local dashboard turned out to run from a process nine days old: the new route returned 404 while every test passed.

**Enforced by.** A memory with concrete probes: restart any long-running process after an edit, request the new route and treat a 404 as the tell, search for a string unique to the change, exercise the control rather than its markup, and say "unverified" when the behaviour cannot be exercised.

## 2. Check every reader before changing shared state

**Incident (2026-09-17).** In another project, one session produced three regressions. It said a scheduled job had retried and recovered; the job definition had no retry, so the log showed five separate failed mornings. It moved a shared setting out of the scheduler config without searching for readers; a second script depended on it. A live test wrote a fake event into the real log, and a dashboard showed a false alarm. I asked how I could trust the agent if it kept introducing bugs.

**Enforced by.** A memory: search the repo for every reader before moving a value, read a scheduler's definition before saying what it does, patch every side effect in a live test, and verify before reporting.

## 3. Query what is already known before claiming a defect or asking for a decision

**Incident (2026-08-03).** An audit reported that local search had fallen back to the CPU, reading a log line cold. A dated page in the vault had already shown the line was noise, with GPU measurements. On 2026-09-24 a session closed by asking me to decide a question that a vault page had settled weeks earlier, and written down so no session would reopen it.

**Enforced by.** The written rule was skipped four times (2026-08-03, 2026-09-14, 2026-09-18 and 2026-09-24). So I stopped writing rules and added a prompt hook that puts matching vault page titles in front of the model before it answers. It uses keyword search, no model and no network. Its worst case was 0.24s at build and 0.4s across 100 real prompts on 2026-11-01 (median 0.105s). See [the session loop](../workflows/01-session-loop.md) and [the evaluation](../workflows/06-evaluating-my-own-system.md).

## 4. A check someone must remember becomes a scheduled job that reports

**Incident (2026-10-17).** A nightly job in another project had been dead for six days and nobody knew until I happened to ask. It was not the first time: in Agent OS the retrieval benchmark once went 18 days unrun, and a scheduled job in another project stayed dead for 16 days before anyone noticed. I said the check should happen automatically.

**Enforced by.** Scheduled jobs that act and then report: a daily summary mail for that project, and for Agent OS a weekly gate-plus-audit job (`com.example.agent-os-gate`). Jobs leave heartbeats, and a nudge appears at session start when one goes stale.

## 5. A long plan needs a scoreboard of real outcomes and a "what this buys" line per task

**Incident (2026-09-17).** A weeks-long plan an agent drew up showed almost no real-world movement, while every automated part kept running. Every task was clear; none showed movement toward the goal. A second draft was rejected as hard for a person to follow.

**Enforced by.** The rebuilt plan's self-test fails if any week loses its "Buys you" cell. Plans now split in two: plain weekly tasks, with the reasoning in a separate file.

## 6. Finish what you flag in the same turn

**Incident (2026-09-14).** In one session I had to say "do it" five times for work the agent had already identified: committing finished work, pinning a rule an installed skill would overwrite, rebuilding a stale page, refreshing two graphs. I said I felt the flywheel was not working because I had to correct the input again and again. On 2026-10-17 an agent added something I had not asked for to a deliverable, called it harmless and left it.

**Enforced by.** A memory: small, clearly correct fixes next to the current work get done before reporting, and anything unrequested in a deliverable is removed. The read-and-write loop moved into a global rule file loaded in every session.

## 7. Never cut content to save cost; cut tool output instead

**Incident (2026-10-24).** The first plan for nightly transcript capture fed a cheaper model a capped excerpt of each session. Measured afterwards, the cap dropped 92%, 84%, 71% and 68% of the conversation text on the four biggest sessions of that week, which were the most valuable ones. I rejected it.

**Enforced by.** The [night shift](../workflows/03-night-shift.md) builds a spine of each transcript: every user and assistant word, one line per tool call. Until 2026-11-01 it dropped messages I typed mid-turn (25 in 14 days) without the coverage check noticing, as [page 06](../workflows/06-evaluating-my-own-system.md) explains; it now keeps queued messages, attachments and subagent results. Savings come from removing tool output and headless overhead, and the memory requires choosing the model by a scored trial rather than by price alone.

## 8. A record of finished work is progress, not history

**Incident (2026-10-26).** The agent drew up a "final information only" list for the whole vault and marked dated sections and a finished build log as history. I stopped it: "wiki still should show progress even if the work got done or is duplicated." The re-check cut the plan from 3 moves and 23 rewrites to 7 small fixes, and found 8 pages already moved to history that were progress records.

**Enforced by.** Code, since 2026-10-26: the page mover reads each page's kind (frontmatter, else its title) and refuses to move a progress record or any page without a named successor. Every page has declared a `kind:` since the 2026-10-28 backfill; the audit fails if history holds a progress record; the retrieval benchmark has a progress guard.

## 9. Give each extraction agent the page's old node count

**Incident (2026-10-04).** A knowledge-graph refresh sent 20 changed pages to agents at one to three files each. The graph still shrank from 2676 to 2440 nodes; one dense page fell from 161 to 65. Re-running the seven shrunk pages with the old count and an "every named entity gets its own node" rule brought all seven back above their old count (161 to 334, 147 to 368, 87 to 273), and the graph reached 3773 nodes. On 2026-11-02 a size hint in the prompt read as a cap, and agents stopped near it; the same day, agents sent past the limit of 20 running at once were dropped.

**Enforced by.** The scheduled map refresh puts each page's old node count in every prompt, says no agent has a node limit, and dispatches agents in waves of at most 18. It backs up the graph and restores it if nodes fall more than 2% (edges 5%) beyond what deleted pages owned, or if a re-extracted page keeps under half its node density. A gate check fails if a reinstall of the Graphify skill drops the density-chunking or wave rules.
