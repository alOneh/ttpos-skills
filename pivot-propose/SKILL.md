---
name: pivot-propose
description: Proposes a trade decision from the TTPOS flight plan to the human, after the user explicitly agrees, so it appears in TTPOS for their own approval. Use when the user asks to propose, submit or send a setup to their journal or desk. Needs the Avancé plan; a proposal is never an order.
license: MIT
---

# Pivot propose (propose a decision)

Requires the TTPOS MCP server to be connected and the Avancé plan with the `decisions:propose` scope. `propose_decision` puts a candidate setup in front of the human in TTPOS. The proposal waits there for the human, who approves or refuses it, and it expires at the end of the session. Proposing never approves, never places an order and never executes anything. Pass symbols exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`).

## Workflow

1. Call `get_accounts` and pick the account the proposal is for. If there are several, ask the user which one; never guess. A frozen account refuses proposals.
2. Call `get_flight_plan` for the symbol (pivot-flightplan skill). Only the current session counts. Choose a signal with status `forming` or `active`; the candidate is its `occurrence_id`. Never invent or reuse an id from an earlier session or from memory.
3. Explain the setup to the user, quoting only what the tools returned: entry, stop, targets, the account, and what would invalidate it. If a figure is missing, say you do not know it. The indicative size is computed by TTPOS (Risk Sizer) for that account when you call `propose_decision` and is shown in its result. Never state a size before the call.
4. Ask for the user's explicit agreement before calling `propose_decision`. State symbol, side, entry, stop, targets and account, then wait for a clear yes. Never propose on your own initiative, on a vague "ok" about the analysis, or because a previous answer was positive.
5. Call `propose_decision` with `candidate_id`, `account_id` and a `note` (1 to 500 characters). The note must be factual: the setup as returned by the tools and the reason it qualifies. No promises, no hype, no invented numbers. The user will see it labelled "unverified" because an AI wrote it.
6. Give the indicative size: quote it from `summary` (never compute or round it yourself). Tell the user the proposal now waits in TTPOS, where they approve or refuse it, and that it expires at the end of the session. Say that nothing was executed. Never present a proposal as an executed or pending order.

## Refusals

Relay the reason from the message, do not work around it:

- duplicate: the proposal already exists; the result gives the existing `decision_id`. Quote it, do not propose again.
- limit of 5 proposals reached for the account: say so; the human must handle the existing ones in TTPOS first.
- frozen account, unsupported instrument, size below the minimum, compliance veto, kill condition: state the reason; do not retry or try another account to get around it.
- stale or unavailable candidate: call `get_flight_plan` again and, if a `forming` or `active` signal still exists, ask the user again before proposing.
- `write_unconfirmed`: the outcome is unknown, so never retry blindly. Call `list_decisions` first (it shows `account_id`, `proposed_by` and `expires_at`) and look for the proposal; retry only if it is absent and the user agrees again.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
