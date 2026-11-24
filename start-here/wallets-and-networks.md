---
last_reviewed: 2026-11-21T00:00:00.000Z
description: >-
  Supported Solana and Robinhood Chain (EVM 4663) wallets, network switching,
  wallet isolation, and 0-SOL Ask-for-Fund gas recovery.
---

# Wallets & Supported Networks

**TL;DR:** reloadSol supports **Solana Mainnet** (open to all Solana wallets) and **Robinhood Chain (`4663`)** (available to allowlisted or EVM-enabled wallets). Balances, watchlists, and trade history stay strictly isolated per wallet and per chain.

[Connect your wallet on reloadSol →](https://reloadsol.app/)

***

## Supported wallets

| Network                      | Supported Wallets                                          | Standard                  |
| ---------------------------- | ---------------------------------------------------------- | ------------------------- |
| **Solana Mainnet**           | Phantom, Solflare, Backpack, and compatible Solana wallets | Solana Wallet Adapter     |
| **Robinhood Chain (`4663`)** | Rabby and compatible EVM browser wallets                   | EIP-6963 browser provider |

* **Solana Wallet Adapter** connects browser and mobile Solana wallets without ever exposing your private keys.
* **EIP-6963** lets your browser detect EVM wallets like Rabby cleanly without wallet conflicts.
* Your active network choice is saved in browser `localStorage` (`reloadsol.appNetwork`).

***

## Network gating & data isolation

* **Solana and Robinhood Chain never mix:** Watchlists, open positions, P\&L, and trade history are stored separately per `(wallet, chain)`.
* **Robinhood Chain access:** If your wallet is not allowlisted and no EVM provider is active, selecting Robinhood Chain automatically returns to Solana Mainnet.
* **Mobile Phantom reliability:** reloadSol includes a connect-intent recovery hook so tapping an already-installed wallet inside Phantom's in-app browser completes cleanly.

{% hint style="warning" %}
**Non-custodial responsibility:** reloadSol never holds your funds or recovery phrase. Switching networks or clearing browser storage does not affect your on-chain tokens—your assets always remain in your wallet.
{% endhint %}

***

## Related

* [Quickstart Guide](quickstart.md)
* [Stay safe on reloadSol](safety-and-security.md)
* [Single swap](../trade/single-swap.md)
