---
name: pivot-desk
description: Runs the Pivot Master morning briefing from the TTPOS MCP server, chaining announcements, currency strength, screener tops and three flight plans, then lists what to validate in TTPOS. Use when the user asks for a morning briefing, a daily market prep, or what to look at today.
license: MIT
---

# Pivot desk (morning briefing)

Requires the TTPOS MCP server to be connected. This skill chains the other Pivot Master skills in a fixed order. If the server offers a `morning_briefing` prompt, it follows the same steps.

## Workflow

0. Call `get_profile` first. Pick the market: if the user named one, use it; otherwise from `answers.markets` (`metaux`, `majeures` or `croisees` mean `forex`; `crypto` means `crypto`; if both appear, brief `forex` and offer `crypto` afterwards). With no usable market, use `forex`; `america` only on request. Pick the depth from `answers.time` (minutes available a day): 15 means steps 1 to 3; 30 or variable means steps 1 to 4; 60 means steps 1 to 5. If `get_profile` fails (for example 402 or 404) or `answers` is empty, do not stop: use `forex` and the 30 depth. If the user gave tickers, use them and skip the shortlist of step 3.
1. Call `get_calendar` for the next 12 hours (pivot-calendar skill).
2. Call `get_currency_strength` (pivot-calendar skill).
3. Call `get_screener_tops` for the market and keep its first three symbols (pivot-screener skill). Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker. If `get_screener_tops` answers `plan_required`, fall back to `scan_screener` with the NARROW regime.
4. Call `get_flight_plan` (`grain` D) on the three symbols (pivot-flightplan skill).
   - If `get_flight_plan` answers `plan_required`, call it no more: in the top-picks section, list the three symbols with their screener regime and position only, and say the flight plans need the plan named in the message, with its pricing link.
   - If it answers `warming` for a symbol, retry once after `retry_after` seconds; if it still fails, say the plan of that symbol is not available yet and go on with the others. Never fill a missing plan with your own levels.
5. Call `check_entry` on the three symbols (pivot-entry skill) and give each status as returned.
6. Optionally call `get_risk_status` (pivot-risk skill) so the user knows their remaining budget.
7. Propose one entry alert on the best candidate (`create_alert` with condition kind `entry`), saying which status it waits for. Never create it without the user's agreement: ask first and wait for a yes. Only when the depth reached step 5.
8. Reply with these sections, skipping those beyond the chosen depth: (a) announcements to avoid, (b) currency strength, (c) top picks, (d) the three flight plans in method order (CPR regime, price position, router, levels as returned), (e) entry status, (f) the proposed alert, (g) what to validate in TTPOS before any trade.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
