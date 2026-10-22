# Beyond the prompt: enforceable spend controls for AI agents

**Checked:** 2026-10-22. Prices, plan tiers and feature names change fast, so re-verify
anything here before you rely on it.

The snippet in this repo is **advisory**. It asks the agent to stop at a threshold
and depends on the agent doing so. This note lists the controls that actually stop
spending, so you can move up a layer when the honour system isn't enough.

**Strength:**
- **Hard**: requests are refused at the limit.
- **Soft-hard**: refused, but the request that crosses the line still finishes, or
  enforcement lags, so you can go a little over.
- **Alert**: you get notified and nothing stops.
- **Advisory**: the model is told about the budget and is trusted to respect it.

**Status:**
- **Confirmed**: read on the vendor's own page.
- **Search only**: a search result pointed to the vendor's docs, but the page itself
  wasn't read.
- **Unverified**: third-party sources only.

## Summary

1. **Use more than one layer.** Provider caps (org, workspace, key) are the floor. A gateway
   adds per-user and per-key budgets. In Claude Code, `--max-budget-usd` /
   `maxBudgetUsd` is the only native **per-task** dollar cap, and it works only in
   print mode and the Agent SDK, not in interactive sessions. Confirmed locally from
   `claude --help` on 2026-10-22: "only works with --print".
2. **Paid third-party tools bypass every LLM-side cap.** A call to an image-generation
   or search API through an MCP tool never passes through an LLM gateway. You can cap
   those calls with per-key limits on the service itself, a prepaid balance, or a
   Claude Code `PreToolUse` hook or `ask` permission rule on the tools.
3. **Nearly every "hard" cap overshoots a little** (OpenAI, Gemini, Cloudflare,
   Vercel, Kong, `--max-budget-usd`). Leave headroom.
4. **Telling a model about its budget improves its pacing but guarantees nothing.**
   The research below supports this. This repo's snippet is in that class, and it
   pairs well with a hard cap underneath it.

## 1. Provider-side caps

| Option | What it does | Strength | Scope | Status |
|---|---|---|---|---|
| [Anthropic tier spend cap](https://platform.claude.com/docs/en/api/rate-limits) | Monthly cap per usage tier; 429 `enforced_spend_limit_reached` until next month | Hard | Org, monthly | Confirmed |
| Anthropic custom spend limit (Console → Billing) | A limit you set below the tier cap; 400 once reached | Hard | Org, monthly | Confirmed |
| Anthropic workspace spend limits ([costs doc](https://code.claude.com/docs/en/costs)) | Per-workspace spend and rate limits, including the auto-created Claude Code workspace; not on the default workspace | Hard | Workspace | Confirmed |
| Claude Teams/Enterprise usage-credit limits | Credits beyond the seat allowance, capped per org, group or member | Hard | Org, group, user | Confirmed |
| [OpenAI spend limits](https://developers.openai.com/api/docs/guides/spend-limits) | 429 `organization_spend_limit_exceeded` / `project_spend_limit_exceeded`; enforcement lags | Soft-hard | Org, project | Confirmed |
| OpenAI notification thresholds | Email alerts; the docs say they "do not enforce a cap" | Alert | Org, project | Confirmed |
| [OpenRouter per-key limit](https://openrouter.ai/docs/api-reference/api-keys/create-api-key) | `limit` (USD) plus `limit_reset` (daily, weekly, monthly or none) per key | Hard | Key | Confirmed |
| [OpenRouter guardrails](https://openrouter.ai/docs/guides/features/guardrails) | Budgets per member and per key; the lower limit wins | Hard | Member, key | Confirmed |
| [OpenRouter workspace budgets](https://openrouter.ai/docs/guides/features/workspaces/workspace-budgets) | 403 when over budget; requests already running finish; Enterprise only | Soft-hard | Workspace | Confirmed |
| [Gemini API project spend cap](https://ai.google.dev/gemini-api/docs/billing) | Monthly cap set in AI Studio; Google calls it "experimental"; overage window of about 10 minutes | Soft-hard | Project | Confirmed |
| Gemini tier caps | At the tier cap, all linked projects pause until next month | Hard | Billing account | Confirmed |
| [Google Cloud budgets](https://docs.cloud.google.com/billing/docs/how-to/budgets) | Alerts only, unless Pub/Sub automation switches billing off; spend-cap budgets for eligible services | Alert, or hard if automated | Project, billing account | Confirmed |
| [Modal budgets](https://modal.com/docs/guide/budgets) | The workspace budget is "the hard outer cap"; the spend limit stops workloads | Hard | Workspace, environment | Confirmed |
| [Exa per-key budget](https://exa.ai/docs/reference/team-management/create-api-key.md) | `budgetCents` per key per billing period; `isOverBudget` flag | Probably hard | Key | Field confirmed; enforcement inferred |
| [fal](https://fal.ai/docs/documentation/model-apis/pricing) | Prepaid credits; account locks below a balance threshold; no per-key cap found | Hard (via balance) | Account | Search only |

**Prepaid credits with auto-reload off** is the simplest universal hard stop: you
can't spend money that isn't in the account. This is inferred, not a documented
control on any of these services.

## 2. LLM gateways and proxies

| Option | What it does | Strength | Scope | Status |
|---|---|---|---|---|
| [Claude apps gateway](https://code.claude.com/docs/en/claude-apps-gateway-spend-limits) (Anthropic, self-hosted, `claude gateway`) | Daily, weekly or monthly caps via admin API; 429 `billing_error`; Claude Code warns at 75% and 95%; fails open if its Postgres store is down unless `fail_closed_on_error` is set; fronts Bedrock, Google's Agent Platform and Microsoft Foundry | Hard | User, IdP group, org | Confirmed |
| [LiteLLM proxy](https://docs.litellm.ai/docs/proxy/users) | `max_budget` plus `budget_duration` on virtual keys, users, teams and members; rejects before calling the provider; cost reservation for concurrent requests. **Without Postgres, budgets aren't enforced** (it only logs a warning) | Hard | Key, user, team, model | Confirmed |
| [Portkey](https://portkey.ai/docs/product/ai-gateway/virtual-keys/budget-limits) | Cost or token limit per virtual key or integration; key expires when used up; optional reset; Enterprise and some Pro plans | Hard | Key, workspace | Search only |
| [Helicone](https://docs.helicone.ai/features/advanced-usage/custom-rate-limits) | Cost-based rate limits through a header, e.g. `500;w=3600;u=cents;s=user` is $5 per hour per user | Hard (rolling window) | User, custom property | Search only |
| [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/features/spend-limits/) | Dollar budgets by model, provider or metadata; 429, or fall back to a cheaper model; eventually consistent; launched 2026-06-05 | Soft-hard | Gateway, model, metadata | Confirmed |
| [Vercel AI Gateway](https://vercel.com/docs/ai-gateway/observability-and-spend/budgets) | Stacked budgets for team, project, key and user; 402 `quota_for_entity_exceeded`; documented as "a soft cap"; BYOK spend not counted | Soft-hard | Team, project, key, user | Confirmed |
| [Kong AI Rate Limiting Advanced](https://developer.konghq.com/plugins/ai-rate-limiting-advanced) | Limits based on token cost; a request's cost only counts against the next request; Enterprise only | Soft-hard | Consumer, provider, model | Search only |

## 3. Claude Code-specific

| Option | What it does | Strength | Scope | Status |
|---|---|---|---|---|
| [`--max-budget-usd`](https://code.claude.com/docs/en/cli-reference) | Ends a `-p` run when estimated spend hits the amount, subagents included; based on a client-side estimate, so spend can go past it | Soft-hard | Per run (per task) | Confirmed, including locally |
| [Agent SDK `maxBudgetUsd`](https://code.claude.com/docs/en/agent-sdk/agent-loop) | Ends with result subtype `error_max_budget_usd`; can overshoot by one model response plus running subagents; resumed sessions don't add to the total | Soft-hard | Query, session | Confirmed |
| [Agent SDK `taskBudget`](https://code.claude.com/docs/en/agent-sdk/typescript) (alpha) | Tells the model its remaining token budget so it can pace itself | Advisory | Task | Confirmed |
| `--max-turns` / `maxTurns` | Caps agent turns; a proxy for cost | Hard (turns, not dollars) | Run | Confirmed |
| `/usage`, status-line cost, `modelPricing` setting | Session cost estimate; `modelPricing` makes figures and caps use contracted rates | Reporting | Session | Confirmed |
| OpenTelemetry export, Claude Code Analytics API | Per-user cost metrics sent to your own monitoring stack | Reporting, or alert if you add rules | User | Confirmed |
| `PreToolUse` hooks and `ask`/`deny` permission rules | A hook can keep a running total and deny paid MCP tools (match `mcp__*<service>*`) above a threshold; `ask` rules force a human prompt. **This is the only way here to gate paid tools that aren't LLM calls.** | Hard, at tool level (you build it) | Call, session | Mechanism confirmed; budget pattern inferred |
| [ccusage](https://github.com/ryoppippi/ccusage) | Reports from local logs: daily usage, 5-hour blocks, status line | Reporting | Local | Confirmed |
| [cost-guardian](https://github.com/Belkins/cost-guardian) | Plugin with warn, ask or block modes, based on *heuristic* size estimates rather than real billing; small project | Soft | Session, day, project | Exists; low maturity |
| [budgetclaw](https://github.com/RoninForge/budgetclaw) | Daemon that tails `~/.claude/projects` logs and SIGTERMs `claude` on a project or branch cap breach | Hard (kills the process, after the fact) | Project, branch, period | Exists |
| AgentFleet | Web wrapper reported to end a session at a dollar or token ceiling | Unknown | Session | Unverified (news item only) |

## 4. Agent payment and spend authorization

These matter when an agent spends money on anything other than API calls, such as
buying things.

| Option | Spend control | Strength | Status |
|---|---|---|---|
| [Stripe Issuing for agents](https://docs.stripe.com/issuing/agents) | Virtual card per agent with `spending_limits`, merchant-category allowlists, single-use cards, real-time approve/decline webhook | Hard (card network) | Confirmed; access by application |
| [Crossmint virtual cards](https://docs.crossmint.com/solutions/ai-agents/agent-wallets/virtual-cards) | Order intent with an allowance and expiry, approved by passkey; enforced only on the `agentic-token` rail | Hard on one rail only | Confirmed; `/unstable/` API |
| Visa Intelligent Commerce | User-set limits, merchant categories and approval rules on tokenized agent credentials | Hard, as claimed | Press material only |
| Mastercard Agent Pay | Agentic tokens with caps, merchant categories and frequency limits; biometric confirmation | Hard, as claimed | Newsroom only |
| [AP2](https://ap2-protocol.org/) | Signed "Open" mandates carry a budget and constraints; proves authorization rather than enforcing it | Depends on implementation | Confirmed |
| [x402](https://www.x402.org/) | HTTP 402 pay-per-request; **no limits in the protocol**, so caps live in the wallet or client | n/a | Confirmed |
| [Skyfire](https://docs.skyfire.xyz/docs/agent-spending-limits) | Per-request and periodic USDC limits per provider; "not publicly available yet" | Hard, as claimed | Page returned 404 when fetched |
| Payman | Per-agent wallets with caps and approval above a threshold | Unknown | Unverified |

## 5. Research on budget-aware agents

| Paper | Point |
|---|---|
| [Budget-Aware Tool-Use Enables Effective Agent Scaling](https://arxiv.org/abs/2511.17006) (Nov 2025) | More tool budget helps only when the agent can see how much is left; introduces a Budget Tracker and BATS. Confirmed |
| [Budget-Constrained Agentic LLMs (INTENT)](https://arxiv.org/abs/2602.11541) | Planner that forecasts tool costs and stays within hard monetary budgets. Confirmed |
| [arXiv:2603.12634](https://arxiv.org/html/2603.12634) | Argues that self-regulation by the model has no formal guarantees. Search only |
| [AllocBench, arXiv:2607.23332](https://arxiv.org/pdf/2607.23332) | Benchmark for how well agents allocate a tool budget. Search only |

## How this was produced

A research subagent read the vendor pages listed above on 2026-10-22. The
`--max-budget-usd` print-mode restriction was re-checked against the local
`claude --help` output. Nothing here was tested end to end: no cap was deliberately
triggered.
