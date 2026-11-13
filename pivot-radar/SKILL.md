---
name: pivot-radar
description: Runs the Pivot Master radar from the TTPOS MCP server, a four-step funnel (sort, arm, fire, review) that reduces the market to at most three trend-aligned symbols, arms one Telegram entry alert per symbol, and checks an entry only when an alert fires. Use when the user asks for their radar, what to arm today, wants to be alerted when the best opportunities appear, or comes back with a fired alert.
license: MIT
---

# Pivot radar (sort, arm, fire, review)

Requires the TTPOS MCP server to be connected and the Abonné plan. The radar adds no indicator: it removes candidates until at most three remain, each with one direction, then lets the alerts do the watching. Every rule is binary: a symbol passes or is dropped, and the reason is stated. Pass symbols to the tools exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`); never rebuild or shorten a ticker.

Pick the step from the request: "radar", "what do I arm" means steps 1 and 2; "my alert fired", "does it enter" means step 3; "weekly review" means step 4. A full daily run costs about 9 credits (three flight plans, three entry checks): say so once.

## Step 1: sort (context)

1. Call `get_profile` and pick the market as the pivot-desk skill does (`metaux`, `majeures`, `croisees` mean `forex`; `crypto` means `crypto`). Default to `forex`.
2. Call `scan_screener` with `tf` D and `bias` [`STRONG BUY`, `STRONG SELL`], then `scan_screener` with `tf` W (limit 50). On `forex`, also call `get_currency_strength` with timeframe `1D`.
3. Keep a symbol only if all three hold:
   - daily Trend X is `STRONG BUY` with position `ABOVE TC`, or `STRONG SELL` with position `BELOW BC`;
   - weekly Trend X is not on the opposite side (`MIXED` passes);
   - the user did not exclude it.
4. The direction is fixed by the daily Trend X: long for `STRONG BUY`, short for `STRONG SELL`. One direction per symbol, for the whole session.
5. Rank by `score`, highest first; break ties with the most compressed daily CPR (`width_pctl`, lowest first). Keep the first three. If none passes, say "no radar today" and stop: that is a valid result.

## Step 2: arm (alerts)

1. Call `get_calendar` for the next 12 hours.
2. Call `get_flight_plan` (`grain` D) on each kept symbol. Drop the symbol if any of these is true, and say which:
   - `router.execution_bias` is not on the side fixed in step 1;
   - `adr.used_pct` is 70 or more;
   - a High-impact event on one of its currencies falls within the next 2 hours (offer to arm it after the release).
3. Call `check_entry` on each survivor with `side` set to the fixed direction (`long` or `short`), never `auto`.
4. Call `list_alerts` to avoid duplicates.
5. Propose one entry alert per survivor, on the status strictly after the current one (FAR → `APPROACHING`, APPROACHING → `IN_ZONE`, IN_ZONE → `TRIGGERED`; none when `TRIGGERED`, `INVALIDATED`, `EXPIRED` or `NOT_ARMED`), with the same `side` and `expires_in_days` 1. List every alert (symbol, side, status awaited, expiry) and wait for a clear yes before calling `create_alert`. A yes covers only the alerts listed.
6. Reply with one table: symbol, direction, why it passed, entry status, alert armed or not. Then one line per dropped symbol with its reason.

## Step 3: fire (only on an alert)

1. Call `check_entry` on the symbol with the same `side`.
2. The entry qualifies only if `status` is `TRIGGERED`, `late` is false and the side is the one fixed in step 1. Then give the zone, signal, setup number, trigger price, stop and targets exactly as returned, and call `get_risk_status` if the plan allows it.
3. If it qualifies, offer the pivot-propose skill so the trade is linked to a TTPOS decision. If it does not, say "no entry" with the reason, and offer the next alert only if the status allows one.
4. The radar's rule, to state when the user asks about a trade outside it: no alert, no trade; no stop, no trade. The decision stays with the user, in TTPOS.

## Step 4: review (weekly)

Call `get_journal_stats` (`days` 7) and `get_trade_history` on the account the user names (pivot-journal skill). Report the headline numbers, the weakest slice, how many trades have `origin` `ttpos` versus `manual`, and how many have no R (no stop). Describe the past only.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date and level name (CPR, TC, BC, H3, L4, POC, NARROW, WIDE...) unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned, with its `source`. If a tool did not return it, say you do not know.
- Closed candles only. Never reason on the candle still forming.
- No orders. These tools read and propose; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- Alerts notify, never trade. Never create or delete one without a clear yes. If a write answers `write_unconfirmed`, call `list_alerts` before any retry. If `telegram_linked` is false, say the Telegram pairing is done in TTPOS under Mon compte.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short. Never ask for candles or OHLCV series; the tools do not return them.
