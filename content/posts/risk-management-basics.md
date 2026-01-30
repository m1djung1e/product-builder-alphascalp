---
title: "Foundations of Trading Risk Management"
date: 2026-01-29
draft: false
tags: ["risk-management", "trading", "systematic-trading"]
categories: ["trading-foundations"]
ShowToc: true
---

## Why Risk Management Comes First

The primary difference between traders who survive and those who don’t
is not strategy selection, but risk control.

A strategy can be statistically sound and still fail if risk is mismanaged.
In real markets, execution errors, regime shifts, and drawdowns are inevitable.
Risk management exists to ensure that no single mistake — or sequence of mistakes —
can end the trading process.

---

## Core Principles

### 1. Fixed Risk per Trade (Not Fixed Size)

Rather than thinking in terms of position size, risk should be defined
as *maximum acceptable loss per trade*.

A common baseline is risking **1% or less of total equity** per idea.
This does not guarantee profitability — it guarantees survivability.

Position Size =
(Account Equity × Risk %) / |Entry Price − Stop Price|


The key is consistency: every trade expresses the same level of risk,
regardless of market conditions or confidence.

---

### 2. Risk–Reward Is Contextual, Not Absolute

Risk–reward ratios are often presented as static rules (e.g. “always trade 1:2”),
but in practice they depend on market regime and execution style.

| Risk–Reward | Required Win Rate |
|------------|-------------------|
| 1 : 1      | 50%               |
| 1 : 2      | 33%               |
| 1 : 3      | 25%               |

Scalping strategies may operate with lower R:R but higher frequency and tighter risk,
while trend or expansion trades may justify asymmetric payoffs.

The objective is not a specific ratio, but **positive expectancy under controlled variance**.

---

### 3. Drawdown Limits Are System Safeguards

Individual trades fail. Clusters of failures happen.
A system must define when *trading stops*.

Common constraints include:
- Daily max loss (e.g. −2% to −3%)
- Weekly drawdown limits
- Strategy-level cooldowns after consecutive losses

Once a limit is hit, execution stops — without discretion.
This prevents emotional overrides and regime mismatch from compounding losses.

---

## Practical Implementation

1. **Stops are mandatory**  
   If a trade cannot define its invalidation level, it should not be taken.

2. **Risk is decided before entry**  
   Position sizing is calculated *before* the order is placed, not adjusted afterward.

3. **Execution errors are assumed**  
   Slippage, partial fills, and latency should be accounted for in risk models.

4. **Small size is a feature, not a weakness**  
   Capital preservation enables iteration, learning, and adaptation.

5. **Logs matter more than PnL**  
   Trade journals and system metrics reveal failure modes faster than profits do.

---

## Closing Thoughts

Risk management is not a defensive layer added after a strategy is built.
It is the structure that allows a strategy to exist in live markets.

Profit is an outcome.
Survival is a requirement.

A system that cannot control downside
does not deserve to participate in upside.
