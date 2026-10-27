---
name: pivot-timemachine
description: Replays a trade plan on 1,000 simulated paths drawn from past sessions of the same symbol with the TTPOS Time Machine, and reads the odds of reaching the first target before the stop, the verdict against the break-even share, the share of paths that end flat and the distribution of R. Use when the user asks whether a trade is worth taking today, what the odds are, or wants to replay a setup. A simulation on past sessions, never a forecast.
license: MIT
---

# Pivot Time Machine (replay a plan)

Requires the TTPOS MCP server to be connected and the Abonné plan. `run_timemachine` takes a trade plan (symbol, side, entry, stop, 1 to 3 targets) and replays it on 1,000 paths built from past sessions of the same symbol, managed with the real stop and target rules. It only computes and reads; it never places an order. Pass symbols exactly as the tools return them (exchange prefix included, for example `VANTAGE:EURJPY`).

## When to use it

Use it when the user asks "is this trade worth it today", "what are the odds" or "replay this setup". Do not use it to guess where price goes next: it describes how a plan would have fared on past sessions.

## Cost: ask first

A run costs 10 credits. The same plan with the same seed is cached for 10 minutes, and a cached repeat is free. Before the first run, tell the user the cost and get a clear yes. Never run on your own initiative, and never rerun with another seed or pool just to look for a better answer.

## Workflow

1. Call `get_flight_plan` for the symbol (pivot-flightplan skill) so the regime, the levels and the setups frame the answer.
2. Pick the plan with the user, quoting only what the tools returned:
   - a setup of the flight plan: take its side, entry, stop and targets as returned and pass its number as `setup`, which adds that setup's recent history to the result; or
   - a decision card: pass `decision_id` alone (Avancé plan). It supplies symbol, side, entry, stop, targets, setup and management, so do not combine it with the plan fields. `at`, `anchor`, `pool` and `seed` still apply.
3. Ask for the yes on the cost, then call `run_timemachine`. Options, only if the user asks or the plan needs them:
   - `targets`: 1 to 3 targets, nearest first, each with a `price` and an optional `partial` share in ]0, 1];
   - `management`: `range_day` (default), `trend_day` or `magnet`;
   - `at`: entry time HH:MM in the anchor's local time, now when omitted; it must be inside the session;
   - `anchor`: the session clock, `nyse`, `london` or `utc0` (default `utc0`). For US indices use `nyse`;
   - `pool`: `all` (default) or `same_regime`, the past sessions with the same CPR width class as today. Offer `same_regime` when the user wants odds for today's kind of day. It needs at least 20 such sessions; otherwise the call is refused with `pool_too_small`, or `regime_unknown` if today's regime cannot be read. Relay the reason and fall back to `all` only if the user agrees;
   - `seed`: reproduces a run on the same day and data (the simulation always runs on today's session, so another day or refreshed candles give another result).
4. The simulation uses 30-minute candles and the daily grain. Say so if the user asks what it is based on.

## Reading the result

- `p_target`, `p_stop` and `p_flat`: the share of paths that reach the first target before the stop, hit the stop first, or end with neither. They add up to 1. Quote them as returned.
- `p_flat` is the "ni cible ni stop" share. It includes exits at the session end, magnet timeouts and profitable trailing exits before the first target. Do not describe it as a loss, and do not describe it as a win.
- `verdict` has a `label` (`lean_target`, `coin_flip` or `lean_stop`), a `threshold`, `win_r`, `loss_r`, a `decided_share` and a `flat` share. It compares the share of decided paths (those that reached the target or the stop) with the break-even `threshold`, derived from the simulated average win and loss: `win_r` is the mean R of the paths that reached the first target (partial exits and later stops included), `loss_r` the mean loss of the paths stopped first, and threshold = loss_r / (win_r + loss_r), the share at which they balance. When one of the two groups is empty, the threshold falls back to 1 / (1 + R1), where R1 is the first target's reward-to-risk ratio (`r1`). When `win_r` is zero or below, no share can break even and the verdict is `lean_stop`. Explain it as such: above the threshold the plan leans to the target, around it it is a coin flip, below it it leans to the stop. Quote the threshold and the decided share next to the label.
- `r_mean`, `r_p10`, `r_p50`, `r_p90` and `histogram` give the distribution of R, a multiple of the initial risk. `exits` breaks down how paths ended. Quote them as returned.
- `pool` and `paths` describe the sample; `as_of` and `anchor` say which moment and clock were used.
- `history`, present when a `setup` was given, shows how that setup fared recently. It is often a small sample: say the number of cases and do not present it as a pattern.
- `seed` reproduces the run on the same day and data: give it if the user wants to rerun the exact same simulation that day.
- `disclaimer` comes with the result: relay it.

## Never a forecast

The result is a simulation on past sessions. It does not tell what will happen today. Never present it as a promise, a prediction or a recommendation to take or skip the trade. You may say where the numbers lean; the decision is the user's, made in TTPOS.

## Rules (always)

- Answer in the user's language: French by default, with the formal "vous". Each tool result has a `summary` written in French; when you reply in French, reuse it word for word. In another language, translate it faithfully and leave every number, price, date, symbol and level name unchanged.
- Never invent a number, level or statistic. Quote only what a tool returned. If a tool did not return it, say you do not know.
- No orders. These tools read and compute; they never place, approve or execute anything. Approving or executing a trade is done by the human in TTPOS.
- This is not investment advice. Keep the line `Pas un conseil en investissement.` in French answers (translate it otherwise) and promise no gain.
- If a result says `warming`, the data is being prepared: wait, then retry once after `retry_after` seconds.
- If a tool answers `plan_required`, tell the user which plan is needed and give the pricing link from the message.
- Keep answers short: the three probabilities, the verdict with its threshold, the R percentiles, one line on the sample. Never ask for candles or paths; the tool does not return them.
