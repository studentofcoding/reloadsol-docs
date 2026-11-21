---
description: Buy up to 4 tokens at once on Solana or Robinhood Chain from a single spend amount, with $0.50/token minimum, inline risk checks, and affordability guards.
last_reviewed: 2026-11-21
---

# Bulk buy up to 4 tokens

**TL;DR:** Paste up to **4 token addresses** on `/buy/solana` or `/buy/robinhood`, set one total spend amount (minimum **$0.50 per token**), review inline risk indicators, and execute in a single coordinated batch.

[Open Bulk Buy →](https://reloadsol.app/buy/solana)

![Bulk Buy screen with 4-token cap in Paper mode](../.gitbook/assets/screens/desktop/buy-solana.png "Bulk Buy · Solana 4-token cap (Paper mode)")

---

## Execute a bulk buy in 4 steps

{% stepper %}
{% step %}
### Open Bulk Buy

Visit [https://reloadsol.app/buy/solana](https://reloadsol.app/buy/solana) (or [https://reloadsol.app/buy/robinhood](https://reloadsol.app/buy/robinhood) on Robinhood Chain).
{% endstep %}

{% step %}
### Add up to 4 tokens

Paste up to **4** contract addresses separated by spaces, commas, or newlines, or click a token from trending/search to append it to your basket.
{% endstep %}

{% step %}
### Set spend amount and review risk

Choose your spend asset (**SOL** on Solana; **ETH**, **WETH**, or **USDG** on Robinhood Chain) and enter a total spend of at least **$0.50 per token**. Review the inline risk chips for each token.
{% endstep %}

{% step %}
### Sign and verify settlement

Click **Buy**. reloadSol fetches fresh execution quotes, checks your SOL reserve (including ATA rent for new accounts), simulates all legs, and opens one wallet approval.
{% endstep %}
{% endstepper %}

---

## Execution limits and behavior

| Setting | Default | Details |
| :--- | :--- | :--- |
| **Max tokens per batch** | **4 tokens** | Applies to both Solana and Robinhood Chain. |
| **Minimum spend** | **$0.50 per token** | Prevents dust-sized buys that lose value to flat network fees. |
| **Max price impact** | **15%** | Routes above 15% [price impact](../getting-started/trading-terms.md#quote-and-execution) are discarded before signing. |
| **Solana balance reserve** | Spend + fees + ATA rent | Reserves `~0.00203928 SOL` per new token account automatically. |

{% hint style="warning" %}
**Partial batch fills on Solana:** Each token leg in a Solana bulk buy is an independent swap transaction signed together. If 3 legs confirm and 1 leg reverts due to sudden slippage, the 3 confirmed legs settle and the 1 failed leg is clearly flagged in [History](https://reloadsol.app/history).
{% endhint %}

---

## Related

* [Practice with Paper mode](../getting-started/paper-mode.md)
* [Order execution, slippage & fees](order-execution-and-mechanics.md)
* [P&L, trade history & portfolio](pnl-history-and-portfolio.md)
