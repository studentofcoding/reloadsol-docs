---
description: Essential risk management for spot traders: drawdown recovery math, payoff ratios, break-even win rate, and fixed-fee drag on small stakes.
last_reviewed: 2026-11-21
---

# Risk basics for spot traders

**TL;DR:** On-chain micro-cap tokens move fast in both directions. Controlling position size, understanding drawdown recovery math, and avoiding tiny-stake fee drag matter more than predicting every winner.

[Open reloadSol P&L →](https://reloadsol.app/pnl)

---

## What reloadSol does (and does not) do

{% hint style="warning" %}
reloadSol public trading surfaces are **spot, fully-collateralised, non-custodial** market execution. There is **no margin, no leverage, no liquidation, no resting limit order, and no exchange-side stop-loss or take-profit order**. Your position is a token in your own wallet and you exit by selling it.
{% endhint %}

---

## Drawdown recovery math

Losses and gains are asymmetric. The deeper a [drawdown](../getting-started/trading-terms.md#risk-metrics) gets, the harder it is to recover:

| Drawdown from entry | Gain required to break even |
| :--- | :--- |
| **−10%** | **+11.1%** |
| **−25%** | **+33.3%** |
| **−50%** | **+100.0%** |
| **−80%** | **+400.0%** |

Cutting losing trades early keeps your capital intact for the next setup.

---

## Payoff ratio and break-even win rate

You do not need to win more than 50% of your trades to be profitable—what matters is your **Payoff Ratio (`R = Avg Win ÷ |Avg Loss|`)** and **Break-Even Win Rate (`1 ÷ (1 + R)`)**:

| Avg Win | Avg Loss | Payoff Ratio (R) | Break-Even Win Rate |
| :--- | :--- | :--- | :--- |
| **+20%** | **−20%** | `1.00×` | **50.0%** |
| **+45%** | **−15%** | `3.00×` | **25.0%** |
| **+60%** | **−10%** | `6.00×` | **14.3%** |

A trader winning only **40%** of trades with a `3.00×` payoff ratio is solidly profitable; a trader winning **60%** of trades with small wins and one **−80%** rug loses money.

---

## Fixed-cost drag on tiny positions

Network and priority fees are **flat per transaction**, not a percentage of your trade. On very small stakes (for example, `0.002 SOL`), flat round-trip fees can eat **10%+** of your position before price even moves. Size positions high enough that fixed fees stay a small fraction of your trade.

*Try the math interactively:* [Open the Fee-Drag & Payoff Calculator →](https://studentofcoding.github.io/reloadsol-docs/widgets/index.html?view=calculator)

---

## Related

* [Stop-Loss, Take-Profit & Exit Mechanics](sl-tp-and-exit-protection.md)
* [Rug Detection & Knowledge Graph](rug-detection-and-knowledge-graph.md)
* [Risk disclosure](../help/risk-disclosure.md)
