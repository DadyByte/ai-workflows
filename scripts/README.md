# Workflow scripts

Three multi-agent review scripts for the Workflow tool in Claude Code. Each starts with a plain `meta` literal and then uses `agent()`, `log()` and `args`. Between them they also use `pipeline()`, `parallel()` and `phase()`: adversarial-code-review uses `pipeline()`, pre-commit-privacy-review uses `pipeline()` and `parallel()`, and pre-push-secret-scan uses `parallel()` and `phase()`.

These are cleaned copies of scripts I ran on my own repositories. I removed my personal rules, paths, repository names and counts, and replaced them with `args` fields and generic rules. Where a public copy behaves differently from the version I ran, page 07 says so, for example where the push scan now fails closed when a lens does not finish.

| Script | What it does | Input in `args` |
|---|---|---|
| [pre-push-secret-scan.workflow.js](pre-push-secret-scan.workflow.js) | Four finder lenses read an unpushed diff; each non-info finding gets two verifiers; one `real_secret` verdict blocks the push. | `diffPath`, `repoRoot`; optional `baseRef`, `repoDescription`, `neverCopy`, `acceptedContent`, `configFiles` |
| [pre-commit-privacy-review.workflow.js](pre-commit-privacy-review.workflow.js) | Three lenses (personal data, the owner's own rules, secrets) read staged diffs; a refute-by-default skeptic checks each finding within a shared budget; the rest come back as unverified. | `repos` (array of `label`, `root`, `diffPath`); optional `accepted`, `customRules`, `verifyBudget`, `stamp` |
| [adversarial-code-review.workflow.js](adversarial-code-review.workflow.js) | Five lenses for a small web service, each a finder followed by a skeptic who must reproduce the finding. | `repoRoot`, `scratchDir`, `brief`, `parts`; optional `tradeoffs`, `devUrl`, `forbidden` |

Every `agent()` call sets a model, and the model for each stage can be overridden through `args`: `finderModel` and `verifierModel` (pre-push-secret-scan, defaults sonnet and opus), `reviewModel` and `verifyModel` (pre-commit-privacy-review, defaults sonnet and opus), `finderModel` and `skepticModel` (adversarial-code-review, both default opus).

What these scripts find depends entirely on the repository they run on and on the rules you pass in. They record no results of their own, and a clean run is not proof that a change is safe. The case studies [07](../workflows/07-multi-agent-review.md) and [08](../workflows/08-delivery-cycle.md) explain the design.
