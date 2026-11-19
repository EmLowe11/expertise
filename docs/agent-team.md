# Vending Business Agent Team

Thirteen specialist Claude Code subagents live in `.claude/agents/`. Each one has its own expertise, a model matched to the job, and hard limits so no agent runs away.

## Roster

| Agent | Role | Model | Effort | Max turns | Search / fetch cap | Why this model |
|---|---|---|---|---|---|---|
| `vending-legal-compliance` | Licenses, tax receipts, contracts, insurance, scams | **Opus** | high | 20 | 8 / 6 | High stakes: a wrong answer means fines or a bad contract. Needs careful reasoning over statutes. |
| `vending-financial-advisor` | Budgets, financing, ROI, break-even, route valuation | **Opus** | high | 20 | 6 / 4 | Capital-allocation decisions with trade-offs and uncertainty. Mistakes cost real money. |
| `vending-executive` | Weighs all specialists' input, resolves conflicts, decides | **Opus** | high | **8** | none (no web) | Judgment and synthesis across domains. Kept short and offline so it decides instead of researching. |
| `vending-accountant` | Books, FL sales tax by county, 1099s, records | Sonnet | medium | 15 | 6 / 4 | Rules-based, structured, checklist work. Sonnet is accurate here at lower cost. |
| `vending-location-scout` | Finds and ranks specific Sanford/Orlando sites | Sonnet | medium | 25 | 12 / 8 | Search-heavy research. Needs speed and volume more than deep reasoning; capped at 15 leads. |
| `vending-marketing` | Brand, flyers, copy, promotions, standing out | Sonnet | medium | 15 | 6 / 5 | Strong writing at a good price. |
| `vending-sales-closer` | Scripts, objections, negotiation, displacing incumbents | Sonnet | medium | 12 | 4 / 3 | Conversational writing and tactics. Few lookups needed. |
| `vending-operations` | Machines, card readers, suppliers, product mix, routes | Sonnet | medium | 15 | 8 / 5 | Practical, product-specific research. |
| `vending-app-builder` | Builds the lead-tracking app in `app/` | Sonnet | high | 40 | 5 / – | Strong, efficient coder. Highest turn cap because building takes steps, but it can't push or deploy. |
| `vending-bid-watcher` | Quick scan for open government vending RFPs | **Haiku** | – | **10** | 6 / 4 | Simple find-and-report task. Fastest and cheapest model; no analysis allowed. |
| `vending-outreach-strategist` | Finds business contact channels, drafts emails with logo/flyer/pamphlet, picks visit times from your calendar | Sonnet | medium | 20 | 10 / 8 | Research plus personalized writing plus scheduling logic. **Cannot send or book**: a hook blocks send/reply/forward/create-event tools. Max 5 companies per run. |
| `vending-finance-tracker` | Weekly per-machine P&L, commissions owed, sales tax set-aside, cash runway, alerts | Sonnet | medium | 15 | none (no web) | Repeatable CSV number-crunching that has to catch anomalies. Has local memory so it never double-imports a file. Works only in `finance/`, which is git-ignored. |
| `vending-proposal-writer` | RFP compliance matrix, bid/no-bid, full proposal drafts | Sonnet | high | 25 | 6 / 8 | Long, careful, compliance-heavy writing. Sonnet at high effort handles it; the final bid/no-bid call can go to the Opus executive. Never submits. |

**Model split:** 3 Opus (judgment and high stakes), 9 Sonnet (skilled specialist work), 1 Haiku (simple scanning).

## Runaway protections

1. **`maxTurns` on every agent.** Claude Code stops the agent at the limit and returns partial output, which can be resumed.
2. **Search and fetch caps** in every prompt, with instructions to stop and report what's still unknown.
3. **Word limits** on every final report (250–700 words), so reports are concise.
4. **No nesting.** Every agent has `Agent` in `disallowedTools`, and `.claude/settings.json` sets `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH=1`. Agents can't spawn more agents.
5. **At most 4 agents running at once** (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS=4`).
6. **Least-privilege tools.** Only the app builder, finance tracker (`finance/`), proposal writer (`docs/proposals/`) and outreach strategist (`docs/outreach/`) can write files. Advisors are read and research only. The executive has no web access at all. Bash is limited to calculations for finance and accounting. Nobody can `git push`.
7. **Stay in your lane.** Each agent has a required **Handoffs** section. Instead of doing another expert's job, it says which agent should weigh in. The main session routes those handoffs, which is how the agents "talk to each other."
8. **Human approval for anything outbound.** The outreach strategist is hard-blocked from sending email or booking calendar events. It can only create drafts, and the main session sends only after you say yes. The proposal writer never submits bids.
9. **Private data stays local.** `finance/` and local agent memory are in `.gitignore`.
10. **Disclaimers.** The legal, financial and accounting agents remind you when to bring in a licensed attorney, CPA or lender.

## How to use

- **Name the agent:** "Use vending-location-scout to find 10 warehouse leads near the Sanford airport."
- **Or let Claude pick:** describe the task, and Claude matches it to an agent's description.
- **Multi-agent decisions:** ask the main session to collect specialist reports and send them to `vending-executive` for the final call.

## Why no separate "strategist" overlap
- **Outreach strategist** is new because no existing agent finds contact channels, drafts personalized emails or plans visits around your calendar. The location scout finds sites, the sales closer writes scripts, and marketing writes copy.
- **Finance tracker** is new because the accountant sets up books and explains tax rules once, and the financial advisor plans big decisions. Neither keeps the ongoing numbers.
- **Proposal writer** is new because the bid watcher only finds bids and is deliberately not allowed to analyze or write.

## Accounts to connect (not yet connected)
- **Gmail** for drafting outreach emails with attachments.
- **Google Calendar** for reading your free time to pick visit slots.
Until these are connected, the outreach strategist saves drafts to `docs/outreach/` and asks for your availability.
