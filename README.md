# Claude Spend Approval Limit

A paste-ready instruction for `CLAUDE.md` (or `AGENTS.md`) that gives an AI coding
agent a **spending approval limit**. It can spend up to a set amount on paid APIs
without asking. Above that amount it has to estimate the cost and get your
approval, the same way a person has a signing limit at work.

The goal is to give the agent full autonomy on small spends and keep a human
decision on large ones. The threshold is a variable you tune.

## The snippet

[`snippet/spend-approval-limit.md`](snippet/spend-approval-limit.md). Copy everything
under `## Snippet` into your system prompt. The frontmatter and `## Why` explain the
rule. They aren't part of the prompt.

What it requires of the agent:

| Rule | Effect |
|---|---|
| Spend ≤ limit | Proceed without asking; report spend afterwards |
| Spend > limit | Estimate with basis (requests × tokens × price), offer cheaper route, wait for approval |
| Per-task, not per-call | Splitting a job into sub-limit calls doesn't get around the limit |
| Re-approval margin | Overrunning an approved estimate by more than the margin means stop and re-ask |
| Multiple options | Cost estimate per model/provider when the price gap is material, under or over the limit |
| Unknown price | Treated as above the limit |
| Pilot first | ~100-item pilot and one synchronous request in the real shape before any bulk or batch run |
| Unattended | The limit becomes a hard ceiling. Do what fits, leave an estimate for the rest |
| Report | Spend per service at the end, measured vs estimated |

## Tuning

There are two variables, declared in the frontmatter and stated in the first line of the
snippet:

| Variable | Default | Meaning |
|---|---|---|
| `APPROVAL_LIMIT` | `$5` | Per-task spend the agent may incur without asking |
| `REAPPROVAL_MARGIN` | `25%` | How far actual spend may exceed an approved estimate before re-asking |

Edit the bold line at the top of the snippet. Set the limit to `$0` and every paid
call needs approval. Set it high and the agent has close to full autonomy, though it
still has to estimate and report.

## Status

Written 2026-10-22. Intended to become a habit in
[Habits-Of-Claude](https://github.com/danielrosehill/Habits-Of-Claude); the snippet
file already uses that repo's frontmatter shape (`id`, `title`, `stance`, `added`,
`status`), plus a `variables` block that habits don't have yet.
