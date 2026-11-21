---
description: Sweep wallet dust into SOL, ETH, or any custom token, burn illiquid tokens, and reclaim locked Solana ATA rent in one batch.
last_reviewed: 2026-11-21
---

# Sweep dust & reclaim rent

**TL;DR:** Select multiple tokens on `/sell/solana` (or `/sell/robinhood`), swap sellable [dust](../getting-started/trading-terms.md#portfolio-and-balances) back into SOL/ETH or a custom token, and close empty Solana accounts to reclaim `~0.00203928 SOL` per account.

[Open Bulk Sell →](https://reloadsol.app/sell/solana)

![Bulk Sell screen on Solana in Paper mode](../.gitbook/assets/screens/desktop/sell-solana.png "Bulk Sell · Solana (Paper mode)")

---

## Sweep dust in 4 steps

{% stepper %}
{% step %}
### Open the Sell screen

Go to [https://reloadsol.app/sell/solana](https://reloadsol.app/sell/solana) (or [https://reloadsol.app/sell/robinhood](https://reloadsol.app/sell/robinhood) on Robinhood Chain).
{% endstep %}

{% step %}
### Filter and select tokens

Use **Dust only**, **Select All**, or **Show zero balance** to pick tokens and empty rent-bearing accounts. Adjust individual amounts with `10%`, `20%`, `50%`, `75%`, or `100%` presets.
{% endstep %}

{% step %}
### Choose your output token

Keep native **SOL** (or **ETH**), or choose any custom output token. Any selected token matching your output token is skipped automatically.
{% endstep %}

{% step %}
### Review quotes and confirm

Wait for live quotes to load, click **Sell / Reload**, and approve the single wallet batch prompt. Watch the confirmation modal move from **Confirming…** to **Confirmed** with Solscan or Blockscout links.
{% endstep %}
{% endstepper %}

---

## How your tokens are categorized

| Category | What it means | Action |
| :--- | :--- | :--- |
| **Sellable** | Active DEX route under the **15%** [price impact](../getting-started/trading-terms.md#quote-and-execution) ceiling. | Swap to SOL/ETH/custom token + auto-close ATA on 100% sell (Solana). |
| **Zero balance (Solana)** | Empty SPL or Token-2022 account locking `~0.00203928 SOL`. | **Close & Reclaim Rent** directly—no swap needed. |
| **No route / too little liquidity** | Pool is drained or has no viable route (`Unsellable`). | On Solana: optional **Burn & Close** to reclaim ATA rent. |
| **Frozen / NFT** | Frozen token account or NFT. | Protected and excluded from bulk selection by default. |

{% hint style="warning" %}
**Burning is permanent:** Using **Burn & Close** permanently destroys the token balance in exchange for the `~0.00203928 SOL` ATA rent. Only burn tokens you have verified are worthless.
{% endhint %}

---

## Related

* [What is checked before you sign](../getting-started/paper-trading-and-safety.md)
* [Order execution, slippage & fees](order-execution-and-mechanics.md)
* [Troubleshooting & FAQ](../help/troubleshooting-and-faq.md)
