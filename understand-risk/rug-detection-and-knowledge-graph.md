---
last_reviewed: 2026-11-21T00:00:00.000Z
description: >-
  How reloadSol's inline risk chips and public Knowledge Graph help you spot
  bundled launches, thin liquidity, and risky deployers before buying.
---

# Rug detection & knowledge graph

**TL;DR:** reloadSol surfaces inline risk indicators on `/buy` and a visual **Knowledge Graph** at `/knowledge-graph` so you can check holder concentration, liquidity depth, and deployer history before signing a buy.

[Open Knowledge Graph →](https://reloadsol.app/knowledge-graph)

***

## What you can inspect before buying

| Signal                       | Where you see it                                                                         | What to watch for                                                            |
| ---------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Inline risk chips**        | [Bulk Buy](https://reloadsol.app/buy/solana)                                             | Flags high top-holder concentration, thin liquidity, or metadata anomalies.  |
| **Deployer provenance**      | [Knowledge Graph](https://reloadsol.app/knowledge-graph)                                 | Shows whether a token creator has a history of failed or rugged launches.    |
| **Bundled / sybil clusters** | [Knowledge Graph](https://reloadsol.app/knowledge-graph)                                 | Visualizes wallets funded from the same source or buying together at launch. |
| **Price impact cap**         | [Bulk Buy](https://reloadsol.app/buy/solana) & [Sell](https://reloadsol.app/sell/solana) | Automatically discards routes exceeding **15%** price impact.                |

***

## Honest limits of automated detection

{% hint style="warning" %}
**Detection is advisory—not a guarantee.** Many risk signals on reloadSol run in observational (shadow) mode, and newly launched tokens may have incomplete holder or deployer history (shown honestly as unknown rather than "clean"). Always verify liquidity and never size a trade assuming a filter will save you from a rug pull.
{% endhint %}

***

## Related

* [Charts, watchlist & discovery](../trade/charts-watchlist-and-discovery.md)
* [Stay safe on reloadSol](../start-here/safety-and-security.md)
* [Risk disclosure](../help/risk-disclosure.md)
