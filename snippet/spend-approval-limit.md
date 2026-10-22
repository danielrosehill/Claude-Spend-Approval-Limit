---
id: spend-approval-limit
title: Spend within your approval limit
stance: practice
added: 2026-10-22
status: active
variables:
  APPROVAL_LIMIT: "$5"        # per task; spend up to this without asking
  REAPPROVAL_MARGIN: "25%"    # overrun on an approved estimate that triggers a re-ask
---

## Snippet

### Spending approval limit

**Approval limit: $5 per task. Re-approval margin: 25%.**

Any call billed per use is real money: LLM APIs, search and scraping APIs,
image, audio and video generation, GPU time, batch jobs. Flat-rate subscriptions
that are already paid for don't count. You hold delegated spending authority,
the way an employee holds a signing limit. Inside the limit you act. Above it you
ask.

The point is to avoid surprises and runaway costs, not to be cheap. Choose the model
that gets good results, and say what it will cost.

- **Up to the limit: go ahead.** Don't ask. Still estimate before you start, and
  report what you spent at the end.
- **Above the limit: estimate first and ask.** Give the figure and its basis
  (requests × tokens × unit price, or units × rate), and the cheaper route if there
  is one. Wait for an answer before spending.
- **The limit is per task, not per call.** Twenty calls under the limit that add
  up to more than it are above the limit. Don't split a job to stay under it.
- **An approval covers its estimate, not a blank cheque.** If actual spend is on
  track to pass the approved figure by more than the re-approval margin, stop,
  report spend so far, and ask again.
- **When there's more than one way to do it, price each one.** If two or more models
  or providers could do the job, and the price gap is material, list each
  option with its estimated cost and what you'd lose by picking the cheaper one.
  Do this below the limit too: there, pick one and say why. Above the limit, it
  goes into the question.
- **Know the price before you spend.** Look up the current unit price. Don't quote it
  from memory. If you can't establish it, treat the job as above the limit.
- **Measure, don't guess.** Count tokens. Before any bulk job, run a small pilot
  (around 100 items) and use its real cost per item to estimate the full run.
  Reasoning models bill mostly on output, so budget output and thinking tokens as
  well as input.
- **Prove the request shape cheaply.** Send one synchronous request in the exact shape
  you plan to use before submitting a batch. A failed batch wastes the money and
  the time.
- **When quality is equal, take the cheaper route:** batch APIs over realtime,
  small fast models for bulk labelling, and a local model when one is available.
  Don't trade away a good result to save money that was within the limit.
- **Unattended, the limit is a hard ceiling.** With no one to ask, do the part that
  fits within the limit, stop, and leave the estimate for the rest.
- **Report spend per service** at the end of every task that spent anything, and say
  which figures you measured and which you estimated.

## Why

An agent with API keys spends money at machine speed. Without a stated threshold
it either asks about every cent, which defeats the point of delegating, or asks
about nothing, and a prepaid balance is gone within a day. A numeric limit settles
the question in advance. Small spends happen without interrupting anyone, and large
ones always get a human decision backed by an estimate. Change the number and you
change how much autonomy the agent has, without rewriting the rule.
