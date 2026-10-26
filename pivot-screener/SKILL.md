---
name: pivot-screener
description: Finds trading candidates with the Pivot Master method (Central Pivot Range width, regime and price position) through the TTPOS MCP server. Use when the user asks what to watch today, which assets have a narrow or wide CPR, which pairs are compressed, or wants a shortlist on crypto, forex (FX, metals, indices) or US stocks.
license: MIT
---

# Pivot screener

Requires the TTPOS MCP server to be connected. Method: the CPR (Central Pivot Range) of the previous closed period, then its width regime, then the price position versus the CPR. Width % = (TC - BC) / PP x 100. A NARROW CPR (in the screener, a width rank, the percentile in the symbol's own history, at or below 25) tends to precede a trend day; a WIDE CPR (width rank at or above 75) tends to precede a range day. The regime describes the expected character of the session, never its direction. Read the `ttpos://glossary` resource for exact definitions.

## Workflow

1. Pick the market: `crypto` (BloFin), `forex` (Vantage: FX, metals, indices) or `america` (US stocks). Ask only if it is truly ambiguous; default to `forex`.
2. Call `scan_screener` with `market`, the period `tf` (D, W or M, default D) and, if the user cares, `regime` (NARROW, MODERATE, WIDE) and `position` (ABOVE TC, INSIDE, BELOW BC). Rows come sorted from the most compressed CPR.
3. For a quick shortlist ranked by conviction, call `get_screener_tops` (five angles, five symbols each, score 0 to 4). It needs the Abonné plan.
4. Present the few best rows: symbol, regime, position, CPR width, and the Trend X bias and score only when the result contains them. Open with the tool's `summary`.
5. Offer the next step: a flight plan on one symbol (the pivot-flightplan skill, tool `get_flight_plan`). Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker.

## Notes

- Free plans: the NARROW and WIDE filters work; the MODERATE regime, the bias filter and `get_screener_tops` need the Abonné plan.
- A narrow or wide CPR is a context, not a signal. Say what it makes more likely, then point to the flight plan and to TTPOS for validation.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
