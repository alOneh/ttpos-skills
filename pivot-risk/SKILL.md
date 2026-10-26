---
name: pivot-risk
description: Reads the user's TTPOS trading accounts, today's risk gauge and trade decision journal through the TTPOS MCP server (read only). Use when the user asks whether they can still trade today, how much loss budget remains, which accounts exist, or what was proposed, approved or rejected recently.
license: MIT
---

# Pivot risk

Requires the TTPOS MCP server to be connected. These tools are read only: approving, rejecting or executing a decision happens in TTPOS, never here.

## Workflow

1. Call `get_risk_status` first (Avancé plan). It gives today's losing trades versus the daily maximum, today's and cumulative loss versus the learning budget, the per-trade risk cap and a `healthy` flag.
2. If the gauge is not healthy, say it plainly and recommend not opening new trades; the decision stays with the user.
3. Call `get_accounts` to list the accounts (label, broker, environment, autonomy mode observer, copilot or autopilot, balance, equity, frozen or not). Mention frozen accounts.
4. Call `list_decisions` (Avancé plan) to review what was proposed or taken; filter with `status` (proposed, approved, rejected, expired, executed, closed, vetoed, exit_signal), `symbol` or `limit`. Newest first.
5. Summarize: risk status, accounts, then the decisions that need attention (for example those still proposed). Send the user to TTPOS to approve, reject or settle anything.

## Notes

- Never suggest raising a risk cap or ignoring a veto.
- Quote amounts and counts exactly as returned, in the currency returned.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
