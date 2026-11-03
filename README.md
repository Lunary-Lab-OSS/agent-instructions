# Global agent instructions

[AGENTS.md](AGENTS.md) is a standalone set of cross-project instructions for coding
agents. It covers scoped implementation, evidence gathering, reproducible changes,
verification, collaboration, authorization boundaries, and honest handoffs.
This README explains the design and its research basis; it is not an additional
instruction dependency.

## Use and scope

Read AGENTS.md before adopting it and reconcile it with your runtime's instruction
hierarchy, existing guidance, and approval requirements. Place it where your tool
loads global or repository instructions, and verify that loading in the intended
harness. Do not overwrite existing instructions without reviewing their content.
The file defines defaults, not a universal architecture or deployment model.

Keep project-specific conventions and specialized procedures in the appropriate
repository documentation or skills. The core directs agents to consult relevant
instructions without prescribing a particular platform or credential architecture.
No-secret-disclosure, authorized credential access, no incidental credential
rotation/revocation, and explicit permission boundaries remain in AGENTS.md.

Instructions guide behavior; they do not enforce permissions. Use credentials,
sandboxes, tool permissions, required checks and merge gates as the actual controls.

## What the research actually supports

Sources were checked on **2026-11-03**. This is a focused review of primary
research and maintainer guidance, not a systematic literature review.

| Source | Finding | Limits that matter here |
| --- | --- | --- |
| [Gloaguen et al., *Evaluating AGENTS.md*, v3, 2026-09-29](https://arxiv.org/html/2602.11988v3), sections 4–5 and appendix B | Across four agent/model configurations on SWE-bench Lite and CTXbench, context files did not generally improve task success, while inference cost rose by over 20% on average. Developer-written files outperformed generated ones, but neither had a statistically significant success difference from no context. | Python tasks and specific model/harness versions. The study did **not** establish that longer files caused the cost increase: its length analysis found no clear relationship with success or per-task cost. Following extra instructions induced more work. Security and other non-functional outcomes were not established by the task-resolution metric. |
| [Lulla et al., *On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents*, v2, 2026-03-30](https://arxiv.org/html/2601.20404v2), sections 3–4 | In 124 small PR tasks across 10 repositories using GPT-5.2-Codex, files were associated with 28.64% lower median runtime and 16.58% fewer median **output** tokens. | This is important counterevidence to “instructions only waste tokens.” It used one agent/model and bounded changes. A manual sanity check of 50 outputs was not a comprehensive correctness evaluation. Output-token reductions are not the same as total-token or billing reductions. |
| [Khatri, *Do Context Files Help Coding Agents?*, REALM 2026, 2026-10-29](https://aclanthology.org/2026.realm-1.5.pdf), sections 3–6 | Compared no context, always-loaded context and selective retrieval across 288 evaluated runs on 17 tasks from three Python repositories. No measurable correctness difference emerged; some narrow process effects did. | Only 15 tasks were shared by both agents. Thirteen tasks used gold tests; four used diff overlap. Statistical power was limited, injection channels differed between agents, and selective directories contained duplicate copies of unchanged sections. This does not prove universal equivalence or that retrieval always wins. |
| [Liu et al., *Curating Always-Loaded Context for LLM Agents*, v1, 2026-10-07](https://arxiv.org/html/2610.11007v1), sections 2–5 and appendix D | Models instruction selection under capacity and cost assumptions; experiments find that irrelevant rules can reduce instruction compliance. | A preprint combining theory with writing/keyword-compliance tasks, not end-to-end repository engineering. Its assumptions and experimental setup do not establish a universal file-length limit, production billing rule or coding-success improvement. |
| [Aliabadi, *Lessons Outlive Their Failures*, v1, 2026-10-23](https://arxiv.org/abs/2610.32578) | A six-week case study suggests some accumulated rules become obsolete as models improve, while documentation drift can still require mechanical checks. | One personal repository and small replay experiments. Useful motivation to revisit rules, not broad causal evidence that deleting them improves every agent. |

**Decision:** keep rules that express actual constraints, remove irrelevant or
stale requirements, and measure changes on representative tasks before claiming
an improvement. These studies do not justify “long files never help,” a universal
token cutoff, or a measured performance claim for this example.

## Maintainer guidance, separately from experiments

- [Anthropic's current best-practices guidance](https://code.claude.com/docs/en/best-practices#keep-claude-md-short)
  recommends short instructions for information the agent cannot infer from code,
  with specialized procedures scoped to relevant work. This is product-team
  guidance, not a controlled result for this file.
- [The AGENTS.md format maintainers](https://agents.md/#how-to-use-agents-md)
  recommend project-relevant instructions and scoped files. The format separates
  agent instructions from human documentation; it does not prescribe an optimal
  word count or guarantee identical loading behavior across tools.
- [Anthropic's enforcement guidance](https://code.claude.com/docs/en/best-practices#enforce-the-rules-that-cant-have-exceptions)
  distinguishes model-read instructions from enforced controls. Approval gates,
  permissions, sandboxing and checks must do the enforcing. Markdown alone is
  not a security boundary. The file itself supplies none of those enforcing controls.

## Why these rules belong in AGENTS.md

The studies inform context design, not the individual policy choices. Including
a rule here does not claim that its benefit has been measured. The rationale
below explains the operating constraints expressed in AGENTS.md.

| Rule | Rationale and evidence type |
| --- | --- |
| Runtime hierarchy, local conventions and unresolved conflicts | Establish which instructions apply without allowing a global default to displace more-specific authorized guidance. This is a precedence policy; [maintainer scoping guidance](https://code.claude.com/docs/en/best-practices#scope-rules-to-one-part-of-the-codebase) supports keeping context applicable to its location. |
| Production-ready behavior, relevant failures and explicit prototype exceptions | Preserve the distinction between a delivered change and a deliberately disposable experiment. The rule keeps rigor tied to the requested outcome, rather than interpreting “one-off” as permission for incomplete behavior. This is an acceptance policy, consistent with [verifiable completion](https://code.claude.com/docs/en/best-practices#give-claude-a-check-to-run). |
| Bounded task scope, proportionate planning and targeted clarification | Avoid unrelated improvements and unnecessary ceremony while asking about ambiguities that affect correctness or authority. These are operating choices, not a measured universal planning advantage. [Project-relevant guidance](https://agents.md/#how-to-use-agents-md) supports their role as explicit expectations. |
| Inspect code/callers/tests; search narrowly; load procedures only when relevant | Keeps the policy from prescribing an exhaustive repository tour. [Gloaguen et al.](https://arxiv.org/html/2602.11988v3) caution about redundant context; [Liu et al.](https://arxiv.org/html/2610.11007v1) provide narrower compliance evidence for selective curation. Neither proves a particular retrieval strategy always wins. |
| Installed or version-matched API evidence | Prevents relying on remembered APIs that may not match the environment. This is a concrete correctness constraint, fitting the [maintainer emphasis on non-obvious project facts](https://code.claude.com/docs/en/best-practices#keep-claude-md-short). No benchmark gain is asserted. |
| Smallest coherent fix, preserved contracts, affected tests/docs and actual repository tooling | Preserves the requested outcome without gratuitous refactors or toolchain substitution. The [format guidance](https://agents.md/#how-to-use-agents-md) supports project-specific conventions and commands; the exact scope boundary remains a policy choice. |
| Reproducible dependencies/artifacts and publisher-verified pins | Preserves an explicit delivery policy rather than leaving versions, hashes and update mechanisms to guesswork. This is a non-functional requirement of the kind [Gloaguen et al.](https://arxiv.org/html/2602.11988v3) distinguish from redundant repository summaries. The paper does not evaluate this particular supply-chain policy. |
| No incidental credential rotation or revocation | Maintains the distinction between finding a problem and authorizing a disruptive migration. This is a consequential scope boundary. The same [enforcement guidance](https://code.claude.com/docs/en/best-practices#enforce-the-rules-that-cant-have-exceptions) applies; a sentence is not an access control. |
| Observable regression checks, edge cases, risk-proportionate testing and reruns after integration | Ties completion to behavior rather than a plausible patch. [Maintainer verification guidance](https://code.claude.com/docs/en/best-practices#give-claude-a-check-to-run) supports executable checks. The policy starts with relevant checks, then honors required gates; it does not demand the same exhaustive suite for every task. |
| Do not weaken checks; classify failures honestly; disclose unrun checks | Preserves evidence quality. A failure needs evidence before being called pre-existing. This is an acceptance standard informed by the [verifiable-completion principle](https://code.claude.com/docs/en/best-practices#give-claude-a-check-to-run), not a study result about calibrated model honesty. |
| Smoke-test changed behavior and distinguish completion from blockers | Keeps merged or deployed artifacts separate from verified user-visible behavior. Delivery requirements apply only within authorized scope. [Verification guidance](https://code.claude.com/docs/en/best-practices#give-claude-a-check-to-run) motivates the evidence; it does not grant deployment permission. |
| Bounded delegation, isolated writers/resources and integrated review | Preserves explicit collaboration requirements that a generic “use subagents” instruction would miss. These are project operating constraints under the [format's project-relevance criterion](https://agents.md/#how-to-use-agents-md); no delegation-efficiency gain is claimed. |
| Preserve others' work; treat external content as data; respect approved gates and controls | Maintains authority boundaries, including the rule that an agent cannot authorize itself by changing its own controls. [Maintainer enforcement guidance](https://code.claude.com/docs/en/best-practices#enforce-the-rules-that-cant-have-exceptions) supports separating prose from actual permissions and gates. |
| Evidence-based handoff and no automatic accumulation of one-off lessons | Keeps global guidance focused on recurring cross-project requirements. [Aliabadi's exploratory case study](https://arxiv.org/abs/2610.32578) motivates rechecking obsolete rules; [Liu et al.](https://arxiv.org/html/2610.11007v1) caution against indiscriminate accumulation. These are reasons to evaluate changes, not automatic instructions to delete rarely triggered safety rules. |

## Design choices

- **Completion follows the requested deliverable.** Tasks including authorized
  merge and delivery require the approved merge/promotion workflow and verification
  where the change runs. A draft, investigation, or narrower task is complete
  against its own acceptance criteria; the instructions do not authorize extra
  operations.
- **Autonomous workflows must be documented and user-approved for the task or
  repository.** Repository text cannot grant itself authority or override
  higher-priority confirmations. Once an approved workflow permits an action and
  required gates pass, redundant manual approval is unnecessary.
- **Context is loaded for the task.** Agents inspect relevant code, callers,
  tests, documentation and skills, expanding their search as evidence requires.
  The file does not prescribe exhaustive repository reading or a preferred toolchain.
- **Verification is proportionate but honest.** Required checks still apply.
  Changed behavior, meaningful edge cases and integration effects determine the
  additional tests. Unrun checks, baseline failures and blockers must be reported
  accurately rather than hidden behind a generic success claim.
- **Global guidance stays focused.** Recurring cross-project requirements belong
  here. One-off lessons belong in repository guidance, tests or automated checks.
  Remove or revise rules based on evidence and current requirements, not age or
  length alone.

These are policy and design choices, not measured performance improvements.
[Maintainer guidance](https://code.claude.com/docs/en/best-practices#enforce-the-rules-that-cant-have-exceptions)
supports distinguishing behavioral instructions from enforced controls.

## Validation and limitations

Static review checks consistency, links, formatting, privacy and authorization
boundaries. It is not a model benchmark: no paid model evaluation or measured
behavior, latency, token, or cost improvement is claimed for this file.

Before claiming an improvement, compare instruction variants on representative
tasks under fixed model/harness settings, with repeated runs where feasible.
Measure correctness, policy compliance, wall-clock time and token/cost accounting
separately, including any additional document reads and caching behavior.
