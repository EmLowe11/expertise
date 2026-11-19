# Vending Business Agent Team

Ten specialist Claude Code subagents live in `.claude/agents/`. Each one has its own expertise, a model matched to the job, and hard limits so no agent runs away.

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

**Model split:** 3 Opus (judgment and high stakes), 6 Sonnet (skilled specialist work), 1 Haiku (simple scanning).

## Runaway protections

1. **`maxTurns` on every agent.** Claude Code stops the agent at the limit and returns partial output, which can be resumed.
2. **Search and fetch caps** in every prompt, with instructions to stop and report what's still unknown.
3. **Word limits** on every final report (250–700 words), so reports are concise.
4. **No nesting.** Every agent has `Agent` in `disallowedTools`, and `.claude/settings.json` sets `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH=1`. Agents can't spawn more agents.
5. **At most 4 agents running at once** (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS=4`).
6. **Least-privilege tools.** Only the app builder can write files. Advisors are read and research only. The executive has no web access at all. Bash is limited to calculations for finance and accounting. Nobody can `git push`.
7. **Stay in your lane.** Each agent has a required **Handoffs** section. Instead of doing another expert's job, it says which agent should weigh in. The main session routes those handoffs, which is how the agents "talk to each other."
8. **Disclaimers.** The legal, financial and accounting agents remind you when to bring in a licensed attorney, CPA or lender.

## How to use

- **Name the agent:** "Use vending-location-scout to find 10 warehouse leads near the Sanford airport."
- **Or let Claude pick:** describe the task, and Claude matches it to an agent's description.
- **Multi-agent decisions:** ask the main session to collect specialist reports and send them to `vending-executive` for the final call.
