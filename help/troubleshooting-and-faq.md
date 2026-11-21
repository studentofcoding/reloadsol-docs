---
description: Fix common reloadSol issues: wallet connection stalls, 0 SOL for gas, quote unavailable, pending vs failed swaps, and missing token prices.
last_reviewed: 2026-11-21
---

# Troubleshooting & FAQ

**TL;DR:** Most swap issues come from low SOL gas reserves, high price impact (> 15%), or a stale wallet session. Use the table below to find the exact fix.

[Open reloadSol →](https://reloadsol.app/)

---

## Common symptoms and fixes

| Symptom | Likely cause | What to do |
| :--- | :--- | :--- |
| **Wallet won't connect (Phantom mobile)** | Stale in-app browser session | Refresh once, tap **Connect Wallet**, and wait 2 seconds for auto-reconnect on [reloadsol.app](https://reloadsol.app/). |
| **0 SOL for gas, but wallet holds tokens** | No native SOL to pay transaction fees | Paste your address on [reloadsol.app](https://reloadsol.app/) and click **Ask for Fund**. |
| **Quote unavailable / route rejected** | Pool liquidity too thin or price impact > **15%** | Lower your trade size, or use **Burn & Close** on [Bulk Sell](https://reloadsol.app/sell/solana) if the pool is dead. |
| **Swap stuck on "Confirming…"** | Waiting for on-chain receipt | Do not double-submit; check the Solscan / Blockscout link in [History](https://reloadsol.app/history). |
| **Swap shows "Failed"** | Reverted on-chain, slippage exceeded, or rejected in wallet | No funds were swapped; increase slippage slightly or retry with a fresh quote. |
| **Token price shows `—`** | Live price feed temporarily unavailable | reloadSol never fabricates fallback prices; wait a few seconds and refresh. |
| **Can't switch to Robinhood Chain** | Robinhood Chain is currently allowlist/EVM-gated | Connect a supported EVM wallet (e.g., Rabby) or trade on Solana Mainnet. |

---

## Recovering a wallet with 0 SOL for gas

{% stepper %}
{% step %}
### Open the Wallet Check

Go to [reloadsol.app](https://reloadsol.app/) without connecting a wallet and paste your Solana address.
{% endstep %}

{% step %}
### Check recoverable value

Confirm the preview shows positive **Sellable** token value or **Reclaimable ATA rent** while your native SOL balance is near zero.
{% endstep %}

{% step %}
### Click Ask for Fund

Submit the **Ask for Fund** request. Eligibility is verified server-side and sent to the operations desk for manual gas assistance (rate-limited and cooled-down per wallet).
{% endstep %}
{% endstepper %}

{% hint style="warning" %}
**Never pay anyone for "gas recovery" in DMs.** Official **Ask for Fund** requests happen only on `https://reloadsol.app/` and never ask for your seed phrase or private key.
{% endhint %}

---

## Related

* [Stay safe on reloadSol](../getting-started/safety-and-security.md)
* [What is checked before you sign](../getting-started/paper-trading-and-safety.md)
* [Bulk Sell, Dust Sweep & Rent Reclaim](../trading/bulk-sell-and-dust-reload.md)
