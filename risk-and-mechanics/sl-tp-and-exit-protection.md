---
description: How spot exits work on reloadSol—why there are no exchange-side stop-loss or limit orders on the public app, and how to manage exits safely.
last_reviewed: 2026-11-21
---

# Exit mechanics & protection

**TL;DR:** On the public reloadSol app, your tokens live in your own non-custodial wallet—you exit manually via **Bulk Sell**, **Single Swap**, or **Fast Sell** on `/pnl`. reloadSol does **not** place exchange-side stop-loss, take-profit, or resting limit orders for your wallet.

[Open P&L Fast Sell →](https://reloadsol.app/pnl)

---

## What reloadSol does (and does not) offer

{% hint style="warning" %}
reloadSol public trading surfaces are **spot, fully-collateralised, non-custodial** market execution. There is **no margin, no leverage, no liquidation, no resting limit order, and no exchange-side stop-loss or take-profit order**. Your position is a token in your own wallet and you exit by selling it.
{% endhint %}

---

## Practical exit discipline for spot traders

Because you sign every exit yourself, plan your exit *before* you buy:

| Exit habit | Why it helps |
| :--- | :--- |
| **Decide your invalidation level first** | Avoids freezing during a fast drop—remember a **−50%** loss needs **+100%** to break even. |
| **Take partial profits into strength** | Use the `20%`, `50%`, or `75%` presets on [Bulk Sell](https://reloadsol.app/sell/solana) to lock gains while keeping a runner. |
| **Watch liquidity, not just price** | A token up +100% on thin liquidity can suffer heavy [price impact](../getting-started/trading-terms.md#quote-and-execution) when you sell. |

*Explore how dynamic trailing stops and early breakdown cuts compare to static stops in our illustrative model:* [Open the OHLC Defense Simulator ↗](https://studentofcoding.github.io/reloadsol-docs/widgets/index.html?view=ohlc)

---

## Related

* [Risk basics for spot traders](risk-basics.md)
* [Sweep dust & reclaim rent](../trading/bulk-sell-and-dust-reload.md)
* [P&L, trade history & portfolio](../trading/pnl-history-and-portfolio.md)
