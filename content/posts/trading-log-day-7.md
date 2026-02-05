---
title: "Trading Log — Human-in-the-Loop & Training Data (Day 7)"
date: 2026-02-06
description: "Adding the missing layer: structured training data + Telegram user input. Turning the pipeline into an auditable learning system, not just an execution system."
---

Day 6 fixed the system’s *internal communication* by upgrading the Python engine contract.
Day 7 was about fixing the next weak link: **the system still had no memory, no labels, and no way to ask a human.**

The pipeline could judge signals, but it couldn’t:

* store clean, ML-friendly rows for later learning, or
* capture the user’s decision in real time (approve/reject/override).

That meant the “why not trade” principle was still incomplete. It existed in text logs, not in a dataset.

---

## Morning — Realizing What Was Missing

The MCP-driven n8n updates were strong: workflow cloning, node updates, validation, and tests.
But when I looked at what was actually saved, I saw the gap immediately.

We had logs, but not a **training table**.

n8n was recording outcomes like:

* “Rejected Signal”
* “Trade Filled”
* “Error Alert”

Those are useful operationally, but they don’t form a consistent dataset that can be queried, modeled, or used to improve decision boundaries.

The uncomfortable realization:

> The system could explain itself, but it couldn’t **learn from itself**.

---

## Late Morning — The Dataset Design Shift

Instead of treating Google Sheets as a “log sink”, the goal became:

> Make Sheets a **form-like structured dataset**, where every signal becomes one row with fixed columns.

So I defined a strict schema that could support both:

* future ML training (features + labels), and
* immediate operational review (audit + filtering + search).

The dataset needed three categories of fields:

1. **Raw technical features** (from TradingView & engine output)
2. **Model judgments** (Gemini + OpenAI)
3. **Human labels** (approve/reject/override + reason codes)

This is where the system became a loop, not a line.

---

## Midday — Human-in-the-Loop as a First-Class Gate

The second missing piece was obvious:

> I need a clean approval interface that can interrupt execution safely.

Telegram is the fastest “control surface” because it’s:

* low friction,
* real-time,
* and naturally supports quick binary decisions.

The design principle was simple:

* **EARLY is always preview-only** (never execute).
* **CONFIRM can execute only if allowed by the policy and optionally approved by the user.**
* Even if the user approves, **Hard-Block and risk gates still win.**

So I planned HITL as two workflows:

1. **Main Judge Workflow**: compute + log + notify + queue the review
2. **Telegram Review Handler**: receive user input + update dataset + trigger execution (only if eligible)

That separation matters: it keeps the judge flow deterministic and the review flow event-driven.

---

## Afternoon — Converting Logs into an Auditable Loop

The system’s purpose is not “more trades.”
It’s **fewer bad trades with perfect accountability**.

So every action needed to be trackable through:

* `idempotency_key` as the universal join key
* `status` transitions (AWAITING_USER → APPROVED → FILLED / CANCELLED / FAILED)
* consistent storage across tabs:

  * `signals_log` (full snapshots, verbose)
  * `training_dataset` (flat ML-friendly row)
  * `review_queue` (approval state machine)

This is where the pipeline became something I can actually iterate on:

* filter by outcome,
* compare false positives,
* identify which flags dominated,
* and tighten rules without guessing.

---

## End of Day — What Day 7 Changed

Day 7 wasn’t about adding more AI.
It was about adding the parts AI cannot do alone: **labels and accountability.**

What the system gained:

* A **training dataset** schema that can evolve into modeling or rule tuning.
* A **Telegram control plane** to capture human decisions without breaking safety.
* A clear separation between:

  * orchestration (n8n),
  * truth (engine),
  * narrative judgment (Gemini),
  * risk gate + execution policy (OpenAI),
  * and the final human veto.

The new guiding rule:

> If it can’t be written as a row, it didn’t happen.

Tomorrow’s focus is clear:

* implement the sheets schema + queue loop cleanly,
* enforce the final gates,
* and make the dataset “query-first” so improvements are driven by evidence, not intuition.
