---
title: "Trading Log — Connecting the Pipeline (Day 4)"
date: 2026-02-02
description: "Clarifying the boundary between n8n and Python. Building a clean adapter layer and testing the full signal flow end-to-end."
---

Today was about connecting the pieces and defining clear responsibilities between systems.

---

## Morning — Clarifying the Architecture Boundary

The day started with a fundamental question:

> If n8n is only restructuring JSON, why not send signals directly from TradingView to the trading engine?

After revisiting the pipeline, the boundary became clear.

- n8n is not a decision-maker
- n8n is a control plane and safety buffer
- All judgment, risk, and intelligence must live inside the Python trading engine

This reframed the system into two clean responsibilities:

```
n8n = input control + normalization
Python = judgment + execution + accountability
```

Once this boundary was accepted, many design doubts disappeared.

---

## Late Morning — Finalizing n8n's Minimal Role

I intentionally reduced n8n's scope.

Removed from n8n:

- Symbol filtering (daily universe changes too often)
- Price handling
- Any strategy logic

Kept in n8n:

- Payload validation
- Enable / disable switch
- Time window gating
- Forwarding a fixed, minimal JSON contract

This made n8n boring — and that's exactly what I wanted.

---

## Midday — Choosing the Engine Design

I committed to a clear plan:

- Use `actionability_integration.py` as a pure calculation engine
- Keep all signal interpretation and orchestration outside that file
- Treat Claude-generated code as domain logic, not glue code

Key realization:

> `CoreSignal` creation must NOT live inside `actionability_integration.py`.

That file must remain:

- Stateless
- Deterministic
- Independent of signal source (TradingView, replay, backtest, etc.)

So the responsibility split became:

- **FastAPI (main.py)** → builds `CoreSignal`
- **actionability_integration.py** → computes final parameters
- **narrative_impact_mapper.py** → optional adjustment layer

---

## Afternoon — Connecting Technical Signals to Actionability

Next goal: translate familiar trading intuition into a single scalar.

> `actionability ∈ [0, 1]`

I deliberately avoided over-engineering and started with what already exists in the signal:

- `atr_ratio` → volatility context
- `rel_volume` → participation / interest
- `side` → directional bias (already decided upstream)

Instead of hard rules, I built soft scores:

- Volatility score
- Volume participation score
- Trend confidence proxy

Each score was mapped conservatively, then combined via a weighted average.

This achieved something important:

- No single indicator can force an entry
- Actionability rises and falls smoothly with market conditions
- The system becomes explainable and extensible

---

## Late Afternoon — CoreSignal as the System Adapter

I implemented `build_core_signal()` inside `main.py`.

This function became the adapter between:

- External signals (TradingView / n8n)
- Internal decision engines (Actionability, Narrative, Risk)

This was a key architectural win.

Now:

- Changing signal sources does not affect the engine
- Adding EMA/VWAP later only touches this function
- Backtesting and replay become trivial

---

## Evening — Local End-to-End Testing

With the engine running locally via FastAPI:

- Started the server with `uvicorn`
- Used n8n's HTTP Request node to call `/api/signal`
- Verified responses contained calculated actionability, final size / TP / SL parameters
- No hard dependency on AI or execution

Seeing the full pipeline respond correctly was the first moment the system felt real.

---

## End of Day — What Changed

By the end of today:

- The system has a clear control boundary
- n8n is intentionally dumb and stable
- Python owns all intelligence
- Claude-generated modules are properly reused, not tangled
- Technical indicators are compressed into a clean, testable signal strength metric

Most importantly, the architecture now feels safe to grow.

---

## Reflection

Tomorrow's work is already clear:

- Replace proxy trend logic with real EMA/VWAP data
- Introduce blocking thresholds on low actionability
- Begin wiring narrative intelligence without letting it override discipline

The foundation is finally solid.
