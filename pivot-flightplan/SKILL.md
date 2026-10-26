---
name: pivot-flightplan
description: Builds and explains the Pivot Master flight plan of one symbol (CPR, width regime, price position, router, key levels, confluence zones, triggered setups, ADR, Initial Balance forecast) from the TTPOS MCP server. Use when the user asks for a flight plan, a pre-market plan, the levels of a symbol, whether a setup fits today, or how a setup has performed.
license: MIT
---

# Pivot flight plan

Requires the TTPOS MCP server to be connected. This skill uses the MCP tools only: the computation is done by the TTPOS engine, never locally, and it replaces any local pivot script. Symbols are TradingView tickers with their exchange prefix (for example `OANDA:EURUSD`, `OANDA:XAUUSD`). Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker.

## Workflow

1. Call `get_flight_plan` with `symbol`, `grain` (the pivot period: D by default, W or M on request) and, to replay a past period, `offset` (0 is the current one, 1 the previous). It needs the Abonné plan and costs credits, so call it once per symbol and per question.
2. Read the result in method order, and say so in that order:
   1. the CPR (TC, PP, BC) of the period;
   2. its width regime: NARROW (trend day expected), MODERATE (no forced reading), WIDE (range day expected);
   3. the price position: ABOVE TC, INSIDE or BELOW BC, and the distance (at contact, near, far);
   4. the router, which says which setups the day allows (NARROW with an open outside the previous range: breakout setups; WIDE confirmed: reversal setups; a forecast rejected by the open means caution);
   5. only then the levels, the confluence zones (DPZ, MTZ, GPZ), the ADR budget and the triggered setups with entry, stop and targets, exactly as returned.
3. Optionally add `get_ib_forecast` with `symbol` and `anchor` (Initial Balance forecast: an amplitude band, not a direction; the `nyse` anchor is the one with real skill).
4. To judge a setup, call `scan_setups` with `symbols` (up to 10), `grain` and `sessions` (number of past periods, 2 to 60, default 20) to see what fired; it costs credits. Then call `get_setup_performance` with `symbol`, `grain` and `sessions` for the measured record of each setup on the symbol. Quote the record as returned and never extrapolate it. Setup numbers and names: `ttpos://setups`.
5. Finish with what would invalidate the plan, and what to validate in TTPOS before any trade.

## Notes

- Levels come from the previous CLOSED period. Never mix in the candle still forming.
- ADR consumption: below 70 % expansion is still possible, from 90 % it is a reversal zone. Use only the figures returned.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
