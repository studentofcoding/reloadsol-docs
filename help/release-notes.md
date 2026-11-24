---
last_reviewed: 2026-11-24T00:00:00.000Z
description: >-
  User-facing release notes and documentation updates for reloadSol across
  Solana Mainnet and Robinhood Chain.
---

# Release notes

**TL;DR:** Chronological summary of public platform improvements and documentation milestones on reloadSol.

[Open reloadSol →](https://reloadsol.app/)

***

## 2026-11-24 — Documentation visual & trading-math overhaul

* Added **Desktop (`1440×900`) and Mobile (`390×844`)** screenshot tabs across Quickstart, Bulk Sell, Bulk Buy, Single Swap, P\&L, Trade History, and Knowledge Graph.
* Added **Solana vs. Robinhood Chain (`4663`)** network comparison tabs on Bulk Buy and Bulk Sell.
* Added an end-to-end **worked trade lifecycle example** showing entry cost, partial exits, flat network fees, refundable ATA rent, realized P\&L, and drawdown math in [P\&L, trade history & portfolio](../trade/pnl-history-and-portfolio.md#worked-trade-example-pl-fees--drawdown).
* Added visual Mermaid flowcharts for **Paper vs. Live Execution**, **Dust Sweep & ATA Rent Reclaim**, and **Non-Custodial Signing & Routing Architecture**.
* Published [Routing, architecture & data feeds](architecture-and-data-feeds.md) and this **Release notes** page.

***

## 2026-11-21 — FinTech UX documentation standards (v2)

* Restructured navigation around trader goals (**Start Here**, **Trade**, **Understand Risk**, **Help**) with direct live app links on every page.
* Added [Trading terms in 5 minutes](../start-here/trading-terms.md) (including a CEX / MetaTrader / TradingView comparison table), [Stay safe on reloadSol](../start-here/safety-and-security.md), [Practice with Paper mode](../start-here/paper-mode.md), [Risk basics for spot traders](../understand-risk/risk-basics.md), [Troubleshooting & FAQ](troubleshooting-and-faq.md), and [Risk disclosure](risk-disclosure.md).
* Added automated documentation verification covering broken links, code-backed platform limits (`4` tokens/batch, `$0.50` minimum, `0.50%–10.00%` slippage, `15%` max impact), and strict public/internal separation.

***

## 2026-11-05 — Pre-sign simulation & Robinhood Chain parity

* Upgraded Solana pre-sign checks to simulate **exact transaction bytes** with the real blockhash before opening wallet approval prompts, preventing stale-blockhash and bad-chunk signing warnings.
* Shipped full **Robinhood Chain (`4663`)** trading parity across Bulk Buy, Bulk Sell, Single Swap, Watchlist, Charts, and P\&L Tracker.
* Added signed-out **Wallet Check** and **Ask for Fund** gas-assistance requests on the landing page for Solana wallets with recoverable tokens or ATA rent but zero SOL for gas.

***

## Related

* [Quickstart Guide](../start-here/quickstart.md)
* [Troubleshooting & FAQ](troubleshooting-and-faq.md)
* [Risk disclosure](risk-disclosure.md)
