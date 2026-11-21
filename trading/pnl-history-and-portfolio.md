---
description: Track realized and unrealized P&L, filter Real vs Sim trades, execute inline Fast Sells, and audit every confirmed or failed transaction in real time.
last_reviewed: 2026-11-21
---

# P&L, trade history & portfolio

**TL;DR:** `/pnl` tracks your open positions and realized/unrealized P&L per wallet and per chain with inline **Fast Sell**; `/history` logs every confirmed, pending, or failed transaction with Solscan/Blockscout links.

[Open P&L Tracker →](https://reloadsol.app/pnl)

![P&L Tracker screen in Paper mode](../.gitbook/assets/screens/desktop/pnl.png "P&L Tracker · Paper mode")

---

## Review positions and Fast Sell

{% stepper %}
{% step %}
### Open your P&L Tracker

Go to [https://reloadsol.app/pnl](https://reloadsol.app/pnl) (or [https://reloadsol.app/history](https://reloadsol.app/history) for the full transaction log).
{% endstep %}

{% step %}
### Filter Real vs. Sim

Use the **Real / Sim** filter to separate live on-chain trades from [Paper mode](../getting-started/paper-mode.md) fills. Solana and Robinhood Chain records never mix.
{% endstep %}

{% step %}
### Close a position with Fast Sell

Click **Fast Sell** on any open token row to exit directly into native **SOL** (Solana) or **ETH** (Robinhood Chain) without navigating to `/sell`.
{% endstep %}
{% endstepper %}

---

## How P&L and status badges work

| Indicator | Meaning |
| :--- | :--- |
| **Realized P&L** | Locked-in profit or loss from matched confirmed buys, sells, and closes, valued at trade-time native prices. |
| **Unrealized P&L** | Live mark-to-market value on tokens still held in your wallet (pruned automatically if you transfer them out). |
| **Confirming… (Amber)** | Transaction submitted and waiting for on-chain receipt. |
| **Confirmed (Green)** | On-chain receipt succeeded; updates push live via Server-Sent Events (no manual refresh needed). |
| **Failed (Red)** | Reverted on-chain or rejected in wallet; never counted as a fill. |

{% hint style="warning" %}
**Entry-time marks vs. current USD:** Historical USD valuations use the SOL/ETH price at the time of the trade so your past P&L does not distort when SOL or ETH moves later. Native assets (`SOL`, `ETH`, `WETH`, `USDG`) are excluded from open-position token rows.
{% endhint %}

---

## Related

* [Sweep dust & reclaim rent](bulk-sell-and-dust-reload.md)
* [Practice with Paper mode](../getting-started/paper-mode.md)
* [Risk basics for spot traders](../risk-and-mechanics/risk-basics.md)
