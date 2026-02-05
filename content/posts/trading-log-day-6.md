---
title: "Trading Log — The Richer Contract Day (Day 6)"
date: 2026-02-05
description: "A deep dive into upgrading the trading engine's output to provide a richer, structured context for AI narrative analysis. Realizing n8n is the wrong place to invent context."
---

Today was about fixing a design flaw that was hiding in plain sight.
The system was working, but it wasn't **communicating with itself** properly.

The goal: upgrade the contract between the Python engine and the AI layer to be richer, more explicit, and ready for production-level narrative analysis.

---

## Morning — Discovering the Flaw

The day started by reviewing the n8n workflow.
TradingView webhooks were successfully passing through the Python engine and triggering AI analysis.

But the output from the engine was too thin. It looked like this:
- **Status:** `ACTIONABLE`
- **Score:** `0.85`
- **Reason:** `"High volume breakout signal"`

This is fine for a human, but for an AI, it's a black box.
It answers *what* but not *why*. The AI had no context to evaluate narrative risk, volatility, or the underlying assumptions of the trade.

The conclusion was uncomfortable:
> The engine was making decisions in a vacuum, and I was trying to patch the context back on downstream in n8n.

---

## Late Morning — The Wrong Path (and the Correction)

My first instinct was to "fix" it in n8n.
I started adding nodes to fetch more data, enrich the signal, and build the context that the engine had thrown away.

This felt productive for about an hour, until I realized it was a huge architectural mistake.

- It made n8n a second, competing source of truth.
- It broke the clear line of accountability.
- It made the system impossible to reproduce and audit.

The right approach was clear:
> **The trading engine is the source of truth for technical context. It must not delegate that responsibility.**

---

## Midday — Redefining the Contract

I went back to the Python engine and started over, this time with a new goal:
**Create a `v1.1` `ExecutionResult` contract.**

This new contract would be the *single source of truth* for any trade decision, containing everything the AI (or a human) would need to know:

- The original `EnrichedSignal`
- The full technical reasoning (volume profile, volatility state, indicators)
- A structured `PositionAndRiskPlan` (entry, TP, SL, expected hold time)
- The final actionability score and human-readable reason

This wasn't just adding fields; it was a commitment to **exposing the engine's internal state** safely and explicitly.

I extended `core_signal_builder.py` and `compute_final_params_legacy` to retain and pass this context through.

---

## Afternoon — Implementing the New Contract

With the design clear, the implementation was straightforward.
I updated the Python code to generate the new `ExecutionResult v1.1` and modified the n8n workflow to consume it.

The new workflow is much cleaner:
1.  **Webhook:** Receives the raw signal.
2.  **Python Engine:**
    - Evaluates the signal.
    - Generates the rich `ExecutionResult v1.1`.
3.  **n8n:**
    - Receives the `ExecutionResult`.
    - Assembles a prompt for the AI using this rich context.
    - Routes the result for approval or execution.

n8n is no longer inventing context; it is **orchestrating it**.

---

## Late Afternoon — First Impressions of n8n

Having spent a full day working with n8n, my thoughts are solidifying.

- It is incredibly powerful for workflow automation.
- It is also a footgun. It's very easy to build messy, untestable "logic" inside it.
- **The key is to treat n8n as a control plane, not a logic engine.**

The mistakes I made in the morning were valuable. They forced me to clarify the boundaries between my services.

---

## End of Day — What Changed

The system is now more robust.
The connection between the trading engine and the AI layer is no longer a "thin pipe" but a **rich, structured conversation**.

- **Ambiguity is reduced.**
- **Accountability is clear.**
- **The AI now has the context it needs to do its job safely.**

I also set a hard goal:
> **Finish this project in the next 12 days.**

This means a fully functional, auditable, and reliable system from signal to execution.
Today was a critical step in making that happen.
