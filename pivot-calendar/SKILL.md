---
name: pivot-calendar
description: Lists the upcoming economic announcements and the relative strength of the 8 major currencies from the TTPOS MCP server, to avoid high-impact news and pick pairs that oppose a strong currency to a weak one. Use when the user asks what news is coming, whether it is safe to trade now, or which currencies are strong or weak.
license: MIT
---

# Pivot calendar and currency strength

Requires the TTPOS MCP server to be connected. Both tools are available on every plan.

## Workflow

1. Call `get_calendar` with `hours` (1 to 168, default 24; use 12 for a morning briefing) and, if useful, `currencies` (USD, EUR, GBP, JPY, CHF, CAD, AUD, NZD). Events come in chronological order with UTC time, currency, impact (Low, Medium, High) and title.
2. Put the High-impact events first and convert times to the user's timezone only if they gave it; otherwise keep UTC and say so.
3. Call `get_currency_strength` (timeframe 5, 15, 30, 60, 240 or 1D; default 60) for the ranking from strongest to weakest, with zone and trend.
4. Suggest pairs that oppose a strong currency to a weak one, and flag those exposed to a High-impact event in the window. Name only currencies and pairs that appear in the results.
5. Offer the next step: a flight plan on the chosen pair (the pivot-flightplan skill, tool `get_flight_plan`).

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
