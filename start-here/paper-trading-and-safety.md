---
last_reviewed: 2026-11-21T00:00:00.000Z
description: >-
  What reloadSol checks and simulates before opening your wallet signature
  prompt on Solana and Robinhood Chain.
---

# What is checked before you sign

**TL;DR:** Before your wallet opens a signature prompt, reloadSol simulates the exact serialized transaction bytes with a live blockhash, checks your SOL reserve for ATA rent, and isolates reverting dust accounts automatically.

[Open reloadSol Sell →](https://reloadsol.app/sell/solana)

***

## Five pre-sign checks on Solana

1. **Exact-bytes simulation:** Every swap leg, account close, and rent reclaim is simulated using the exact serialized bytes and the live recent blockhash—matching what Phantom or Solflare simulates.
2. **Stale blockhash auto-rebuild:** If a blockhash expires before signing, reloadSol rebuilds the transaction once automatically rather than handing you a stale prompt.
3. **Preflight always on:** Every RPC broadcast runs with preflight checks enabled.
4. **Split-on-revert before signing:** When closing multiple empty or dust accounts, if one account reverts during simulation, reloadSol isolates it _before_ opening your wallet so the healthy accounts still close in one approval.
5. **SOL reserve check (Bulk Buy):** Before preparing a buy batch, reloadSol checks that your SOL balance covers swap spend, priority fees, base fees, and **ATA rent (`~0.00203928 SOL` per new token account)**.

***

## Pre-sign safety on Robinhood Chain (`4663`)

* **Atomic BatchExecutor:** Browser-wallet batches wrap ETH, pull tokens via canonical Permit2, and execute all swaps inside a single atomic transaction (`executeBatch`).
* **One-time Permit2 setup:** Approving Permit2 does not move funds; every trade still requires your explicit signature.
* **Strict receipt checks:** Every mined receipt's status is verified—a mined-but-reverted transaction is always recorded as **Failed**, never **Confirmed**.

{% hint style="warning" %}
**Simulation is not a guarantee:** A clean pre-sign simulation catches broken routes, stale blockhashes, and reverting accounts, but on-chain pool prices can still move before your transaction lands in a block.
{% endhint %}

***

## Related

* [Practice with Paper mode](paper-mode.md)
* [Stay safe on reloadSol](safety-and-security.md)
* [Order execution, slippage & fees](../trade/order-execution-and-mechanics.md)
