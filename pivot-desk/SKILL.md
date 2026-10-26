---
name: pivot-desk
description: Runs the Pivot Master morning briefing from the TTPOS MCP server, chaining announcements, currency strength, screener tops and three flight plans, then lists what to validate in TTPOS. Use when the user asks for a morning briefing, a daily market prep, or what to look at today.
license: MIT
---

# Pivot desk (morning briefing)

Requires the TTPOS MCP server to be connected. This skill chains the other Pivot Master skills in a fixed order. If the server offers a `morning_briefing` prompt, it follows the same steps.

## Workflow

1. Pick the market: `forex` by default, or `crypto` or `america`. If the user gave tickers, use them and skip step 4's shortlist.
2. Call `get_calendar` for the next 12 hours (pivot-calendar skill).
3. Call `get_currency_strength` (pivot-calendar skill).
4. Call `get_screener_tops` for the market, then `get_flight_plan` (`grain` D) on the first three symbols (pivot-screener and pivot-flightplan skills). Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker. If `get_screener_tops` answers `plan_required`, fall back to `scan_screener` with the NARROW regime.
   - If `get_flight_plan` answers `plan_required`, call it no more: in section (c), list the three symbols with their screener regime and position only, and say the flight plans need the plan named in the message, with its pricing link.
   - If it answers `warming` for a symbol, retry once after `retry_after` seconds; if it still fails, say the plan of that symbol is not available yet and go on with the others. Never fill a missing plan with your own levels.
5. Before proposing a plan, optionally call `get_risk_status` (pivot-risk skill) so the user knows their remaining budget.
6. Reply with exactly four sections: (a) announcements to avoid, (b) currency strength, (c) the three flight plans in method order (CPR regime, price position, router, levels as returned), (d) what to validate in TTPOS before any trade.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
