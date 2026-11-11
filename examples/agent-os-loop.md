<!-- Public copy. Personal paths, names and ids are replaced with placeholders. -->
# Agent OS — the loop I run every session

The user's second brain lives in `$VAULT` (projects, decisions, research) and
`~/.claude` (how this setup is wired). It only compounds if each session **reads before
deriving** and **writes back what it learned**. Measured 2026-09-14: 7 of 13 substantial
sessions read from it. That count is unverified, because nobody has checked whether automation
transcripts were excluded from it. It is still the number this rule exists to move.

Full reasoning, and the user's own steps: the Agent OS playbook page in the vault.

## Read — before deriving, not after

1. **A SessionStart line naming a vault project is an instruction, not decoration.**
   If it says this directory is project X, the vault already holds prior decisions on
   what is about to be worked on. Query before touching anything.
2. **One retrieval call before asserting any defect, gap, or fact about this setup or
   a project.** `/recall`, or `cd "$VAULT" && qmd query "<question>" --json -n 5`.
   Relationship or "what connects to X" questions go to `graphify query` instead.
   See the memory about querying recall before claiming a defect for what it cost when skipped.
3. **A flagged hit is not triage until it is checked.** A sweep result, a stale-looking
   claim, a suspicious number — run the one command that settles it or say it is unverified.

Reading a file you are about to edit is not retrieval and needs no query. The rule binds
when the question is "what do we already know / decide about X".

## Write — same session, not next time

4. **Durable output gets filed as it is produced**, into `projects/<name>/`, with the
   index line and the log entry in the same pass. Not batched to the end. If it changes what is true now,
   rewrite that project's `<name>-brief.md` in the same pass — the brief is what the user reads.
5. **A correction from the user gets a memory**, in the one store
   `~/.claude/projects/<project>/memory/` — every folder reads and writes it since
   2026-10-24 (`autoMemoryDirectory` in `~/.claude/settings.json`; before that each start
   folder had its own store and most sessions loaded almost none of it) —
   with `type: feedback` and an index line in `MEMORY.md`. The store went 24 days and 30
   sessions without a write while the same corrections repeated; see
   the memory about finishing flagged work instead of handing it back.
6. **End with the gate**: `bash ~/.claude/skills/wrapup/agent_os_refresh.sh`. It is the
   only cheap proof the index and the retrieval benchmark still hold. A red gate is a
   report — relay it verbatim, never retry it into silence.
7. **Name the steps that are the user's**, every time work is summarised or "what's next"
   is answered. `/wrapup` when the session produced durable output; a graph refresh only
   when the gate or the nudge says the weekly job (Tuesday 03:30, since 2026-10-25) is not
   loaded, missed a run, was restored or failed, or will skip a map as too big; acting on or cutting a nudge that
   has printed for over a week. The user's steps are invisible unless raised; mine are not.
   "Nothing on your side" is also an answer. See the memory about surfacing the user's own loop steps.

## The failure this prevents

Every Agent OS defect found in three audits had the same shape: **a rule correct in the
documentation and not true in practice.** These seven steps are only worth their tokens if
they are executed, so prefer doing the small thing now over noting it for later.
