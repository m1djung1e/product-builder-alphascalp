---
title: "Trading Log — Building a System Constitution (Day 5)"
date: 2026-02-04
description: "Formalizing implicit assumptions into explicit, versioned rules. Adding CI enforcement to protect the system from silent breaking changes."
---

Today was not about trading strategies or indicators.
It was about **building rules that protect the system itself**.

I decided to formalize everything that had been living implicitly in my head—assumptions, safety checks, failure handling—into something explicit, versioned, and enforceable.

---

## Morning — Defining the Problem

As the system grew (TradingView → n8n → Python engine → exchange), I realized something uncomfortable:

- Many decisions were *assumed*, not enforced
- Some failures would only be discovered **after** money was at risk
- There was no single source of truth for "what is allowed" vs "what is a bug"

The conclusion was simple:

> Before adding more logic, the system needed a constitution.

---

## Late Morning — Designing the Constitution

I defined a **Trading System Constitution v1.0**:
a set of documents that sit *above* the code.

The scope was intentionally strict:

- Canonical schemas (what data is allowed to exist)
- Idempotency rules (what must never happen twice)
- Retry and circuit breaker policies
- Execution state machine (legal state transitions only)
- Observability requirements (what must be visible)
- Database schema and operational queries

The key rule:

> If it's not in the Constitution, it's not allowed in production.

---

## Midday — Structuring the Repository

I organized the Constitution as first-class assets in the repository:

```
docs/constitution/
├── README.md
├── 01-canonical-schema.md
├── 02-idempotency.md
├── 03-htf-cache.md
├── 04-retry-circuit-breaker.md
├── 05-observability.md
├── 06-state-machine.md
├── 07-database.md
└── schemas/
```

Each section is atomic, referencable, and versionable.

This immediately changed how I thought about future development:
every new feature now has to answer *where it fits* in this structure.

---

## Early Afternoon — Git Initialization and Hard Lessons

This was the messy part.

- Initial `git add` failed because the directory was not a Git repo
- SSH authentication worked, but pushes failed with `Repository not found`
- The root cause turned out **not** to be permissions or shell access

The real issue:
**the SSH key was authenticated to the wrong GitHub account**.

Once the correct account and SSH key alignment were verified, everything clicked into place.

This was a good reminder:

> Git errors are rarely about Git itself—they're about identity.

---

## Afternoon — Constitution Becomes the Source of Truth

With the repository correctly set up, I committed:

- The full Constitution
- JSON Schemas for all signal boundaries

At this point, the Constitution stopped being "documentation" and became **law**.

From now on:

- Design changes happen via Git
- Breaking changes are explicit
- History is preserved

---

## Late Afternoon — Adding CI as an Enforcement Layer

Documentation alone is not enough.
Rules must be enforced automatically.

I added a GitHub Actions workflow:

- Triggers on changes to `docs/constitution/**`
- Validates all JSON Schemas
- Fails the build if schemas are invalid

This turns the Constitution into something actionable:

- Invalid schemas cannot be merged
- Silent breaking changes are blocked
- Mistakes are caught before deployment

The first CI run failed—correctly—due to a directory casing issue:
`.github/workFlows` vs `.github/workflows`.

Fixing that and seeing the workflow re-run was a small but important victory.

---

## End of Day — What Changed

By the end of today, the system had gained something more valuable than a new feature:

- A **single source of truth**
- Automated enforcement of core rules
- Clear separation between:
  - *What the system may do*
  - *What the system must never do*

Most importantly, future work just became safer.

---

## Reflection

Today didn't make the strategy more profitable.
It made the system **harder to break**.

That's a trade I will take every time.

Tomorrow, the Constitution will start shaping the Python engine and execution logic—but now, it will do so on solid ground.
