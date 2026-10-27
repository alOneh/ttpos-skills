---
name: pivot-entry
description: Answers "does it enter now?" for one symbol with the TTPOS Entry Timer (zone reached, closed-candle reversal signal, status, stop) and offers a one-shot alert on the next status. Use when the user asks whether a setup is triggering, whether price is in a zone, or wants to be warned when an entry appears.
license: MIT
---

# Pivot entry (Entry Timer)

Requires the TTPOS MCP server to be connected. The Entry Timer reads the daily desk of a symbol: is price on a zone in the direction of the day, and has a CLOSED intraday candle printed a reversal signal there? It only reads and proposes; the human decides and acts in TTPOS. Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker.

## Workflow

1. Call `get_flight_plan` for the symbol first (pivot-flightplan skill), so the CPR regime, price position, router and levels frame the answer. If the user only asked for the entry status, still read the plan before answering.
2. Call `check_entry` with the `symbol` and, only if the user asks, these options:
   - `tf`: execution candle timeframe in minutes, `5`, `15` (default) or `30`;
   - `zones`: zones to watch among DPZ, GPZ, MTZ, CPR, H3L3, MZ (all when omitted);
   - `signals`: reversal signals to accept among extreme, wick, outside, doji (all when omitted);
   - `side`: `auto` (the execution bias of the day, default), `long` or `short`;
   - `as_of`: an ISO 8601 datetime (UTC if no offset) to replay the status at a past moment, using only candles closed before it. It works for the current session or the 5 previous ones, otherwise it is refused. It costs the same as a live call. The result carries `as_of`: say that the status is a replay, not the live one;
   - `min_score`: 0 to 100, ignores reversal signals whose desk score is below it. `ignored_below_min_score` counts the signals dropped: mention it, so a quiet result is not read as "no signal at all".
   It needs the Abonné plan and costs 1 credit, so call it once per symbol and per question.
3. Explain the `status` as returned:
   - `NOT_ARMED`: no usable direction, zone or data right now; read the `reason` and say so.
   - `FAR`: a zone exists in the direction but price is more than 0.5 ATR away.
   - `APPROACHING`: price is within 0.5 ATR of the zone.
   - `IN_ZONE`: price is in the zone, no reversal signal on a closed candle yet.
   - `TRIGGERED`: a closed candle printed a signal in the zone. Give the zone, signal, setup number, trigger time, price and stop exactly as returned. If `late` is true, the signal is not on the last closed candle: say it already happened and may be stale.
   - `INVALIDATED`: a candle closed beyond the opposite edge of the zone; the zone no longer holds.
   - `EXPIRED`: the session ended.
4. Finish with what would invalidate the entry, and what to validate in TTPOS before any trade.

## Alerts

- An alert fires on a change into its target, never on a status already reached: an entry alert on a status the symbol already has (or has passed) will not fire for that. Offer an entry alert only on the status strictly after the current one: FAR → `APPROACHING`, APPROACHING → `IN_ZONE`, IN_ZONE → `TRIGGERED`. Offer none when the status is `TRIGGERED`, `INVALIDATED`, `EXPIRED` or `NOT_ARMED`.
- The entry alert is `create_alert` with `symbol` and `condition` `{ "kind": "entry", "status": "APPROACHING" | "IN_ZONE" | "TRIGGERED", "side": "auto" | "long" | "short" }`, plus the same `tf`, `zones` and `signals` you passed to `check_entry`, if any. Other shapes exist: `{ "kind": "level", "level": "<desk level>" or "price": <number>, "direction": "above" | "below" }` and `{ "kind": "regime", "tf": "D" | "W" | "M", "regime": "NARROW" | "MODERATE" | "WIDE" }` (screener symbols only). An alert is one-shot, expires after `expires_in_days` (1 to 30, default 7) and only notifies; it never trades.
- Always ask the user and get a clear yes before calling `create_alert` or `delete_alert`. State the symbol, the condition and the expiry first. Never create or delete on your own initiative.
- Call `list_alerts` before creating one (to avoid duplicates) and to find the `alert_id` to delete.
- If a write answers `write_unconfirmed`, the outcome is unknown: call `list_alerts` to check whether it went through before any retry, never retry blindly.
- If the result shows `telegram_linked` false, tell the user that notifications also need the Telegram pairing, done in TTPOS under Mon compte.
- If alerts are capped for the plan, relay the limit and the plan that raises it from the message.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
