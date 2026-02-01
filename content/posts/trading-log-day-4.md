---
title: "Trading Log — Building the Trading Pipeline (Day 4)"
date: 2026-02-02
description: "Turning ideas into a working signal pipeline. No strategies, no optimization—just making sure the system can receive, validate, and route signals correctly."
---

Today was about turning ideas into a working signal pipeline.
No strategies, no optimization—just making sure the system can receive, validate, and route signals correctly. This was an infrastructure day.

---

## Morning — Clarifying the Goal

The goal for today was simple but strict:

- Receive TradingView webhook signals reliably
- Normalize the payload into a clean internal schema
- Reject invalid or out-of-scope signals early
- Log every decision point for later analysis
- Do nothing strategy-related yet

I intentionally avoided:

- Position sizing logic
- ATR or volume thresholds
- AI narrative judgment

This stage is about plumbing, not alpha.

---

## Step 1 — TradingView → n8n Webhook Connection

I started by wiring TradingView alerts to an n8n Webhook node.

Key decisions:

- **HTTP Method:** POST
- **Response mode:** Immediately
- **No authentication** (for now)
- **JSON payload only**

After a few iterations and fixing an `Unused Respond to Webhook` error, the webhook finally received live test data correctly.

At this point, I could clearly see the raw payload:

```json
{
  "symbol": "ETHUSD",
  "tf": "1",
  "trigger": "VOL",
  "atr_ratio": 0.12,
  "rel_volume": 2.12,
  "direction": "SHORT"
}
```

This confirmed that TradingView → n8n transport was stable.

---

## Step 2 — Defining a Canonical Signal Schema

Instead of passing raw webhook data downstream, I decided to standardize everything immediately.

I updated the existing `Extract Signal Fields` node to produce a clean, minimal schema:

- `symbol` (string)
- `side` (string)
- `timeframe` (string)
- `trigger` (string)
- `atr_ratio` (number)
- `rel_volume` (number)
- `source` = "tradingview"
- `timestamp` = current time

All headers, params, and irrelevant metadata were intentionally dropped.

This was an important architectural decision:

> Every downstream component should depend only on the canonical schema, not on webhook internals.

---

## Step 3 — Data Validation as a Hard Gate

Next, I connected the output to `Validate Required Fields`.

The philosophy here is strict:

- Validation checks existence and basic sanity only
- No market judgment
- No strategy assumptions

Examples:

- `symbol` must exist
- `side` must be LONG or SHORT
- `atr_ratio > 0`
- `rel_volume > 0`

If validation fails:

- The signal is logged as invalid
- The webhook still returns 200 OK (to avoid TradingView retries)
- The signal never reaches execution logic

This creates a clean separation between "bad data" and "bad trades."

---

## Step 4 — Enable Flag & Time Window Gates

After validation, I added operational controls:

- Global enable/disable flag
- Trading hours window check

These gates do not judge signal quality.
They only answer:

> "Are we allowed to act right now?"

Rejected signals are logged with explicit reasons:

- Disabled
- Outside time window

Again, everything is recorded.

---

## Step 5 — Logging Before Execution

Before forwarding anything to the Python trading engine, I ensured logging was in place:

- Valid signals
- Invalid signals
- Disabled signals
- Time-window rejections

At this stage, execution is not the priority.
**Observability is.**

If something breaks later, I want to know why, not guess.

---

## End of Day — What Exists Now

By the end of today:

- TradingView webhook is live and stable
- Signal payloads are normalized
- Validation gates are enforced
- Operational controls are active
- Logs capture every decision path

What does not exist yet:

- Strategy logic
- AI narrative integration
- Risk sizing
- Execution decisions

That is intentional.

---

## Reflection

Today wasn't about profits.
It was about building a system that doesn't lie to me later.

Once signals flow cleanly and predictably, strategy becomes a modular layer—not a tangled mess.

Tomorrow, the real decisions begin.
