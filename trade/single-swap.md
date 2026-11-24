---
last_reviewed: 2026-11-21T00:00:00.000Z
description: >-
  Swap a single token pair on Solana with Jupiter Terminal or on Robinhood Chain
  with KyberSwap and one-click Permit2 batching.
---

# Single swap

**TL;DR:** Use `/swap` for focused single-pair trades—**Jupiter Terminal** on Solana Mainnet (an embedded aggregator routing across Solana DEXs) and **RhSwap** (KyberSwap + Permit2) on Robinhood Chain.

[Open Single Swap →](https://reloadsol.app/swap/solana)

{% tabs %}
{% tab title="Desktop (1440×900)" %}
![Single Swap Paper mode safety guard on Solana on desktop](../.gitbook/assets/swap-solana.png)
{% endtab %}

{% tab title="Mobile (390×844)" %}
![Single Swap Paper mode safety guard on Solana on mobile](<../.gitbook/assets/swap-solana (1).png>)
{% endtab %}
{% endtabs %}

***

## Swap a single pair in 3 steps

{% stepper %}
{% step %}
### Open Single Swap

Go to [https://reloadsol.app/swap/solana](https://reloadsol.app/swap/solana) (or [https://reloadsol.app/swap/robinhood](https://reloadsol.app/swap/robinhood) on Robinhood Chain).
{% endstep %}

{% step %}
### Pick input and output tokens

Select a quick preset (**SOL / USDC** on Solana; **ETH / WETH / USDG** on Robinhood Chain) or paste any token address, then enter your amount.
{% endstep %}

{% step %}
### Review route and confirm

Inspect the quoted output, [price impact](../start-here/trading-terms.md#quote-and-execution), and [slippage](../start-here/trading-terms.md#quote-and-execution), then approve the prompt in your wallet.
{% endstep %}
{% endstepper %}

***

## Execution by network

{% tabs %}
{% tab title="Solana Mainnet" %}
* Powered by **Jupiter Terminal**, routing across Raydium, Orca, Meteora, Pump.fun, and other Solana DEXs.
* Supports native-to-token, token-to-native, and token-to-token swaps.
* In **Paper mode**, Jupiter Terminal is deliberately unmounted so it cannot trigger a real wallet signature—use [Bulk Buy](bulk-buy.md) or [Bulk Sell](bulk-sell-and-dust-reload.md) for paper fills instead.
{% endtab %}

{% tab title="Robinhood Chain (4663)" %}
* Powered by **KyberSwap Aggregator** through reloadSol's in-house swap panel.
* **Precedence:** One-click atomic **BatchExecutor** (after one-time Permit2 setup) → **EIP-5792** (`wallet_sendCalls`) → **Sequential** (Approve then Swap).
* Every mined receipt is checked—reverted transactions are always marked **Failed**, never **Confirmed**.
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
**No resting limit orders:** `/swap` executes spot market swaps at current pool liquidity. reloadSol does not place resting limit orders or exchange-side stop orders.
{% endhint %}

***

## Related

* [Order execution, slippage & fees](order-execution-and-mechanics.md)
* [Practice with Paper mode](../start-here/paper-mode.md)
* [Wallets & Supported Networks](../start-here/wallets-and-networks.md)
