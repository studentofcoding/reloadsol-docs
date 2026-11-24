---
description: Plain-language definitions of spot swaps, slippage, price impact, ATA rent, P&L, drawdowns, and how reloadSol compares to a CEX or MetaTrader.
last_reviewed: 2026-11-21
---

# Trading terms in 5 minutes

**TL;DR:** Every trading term used across reloadSol in one place. reloadSol executes **non-custodial spot swaps**—there is no margin, no leverage, no liquidation, and no resting limit orders.

[Open reloadSol →](https://reloadsol.app/)

---

## Quote and execution

| Term | Meaning | Why it matters here |
| :--- | :--- | :--- |
| **Spot swap** | Exchanging one token directly for another at the current pool price. | You own the underlying token in your own wallet. |
| **Aggregator** | Software that splits or routes your swap across DEX pools for the best price. | reloadSol routes Solana swaps via Jupiter and Robinhood Chain swaps via KyberSwap. |
| **Quote** | A live price estimate showing expected output before you sign. | Display estimates update fast; execution quotes are always fetched fresh at click time. |
| **Slippage** | Maximum price move you accept between quote time and block confirmation. | Dynamically scaled between **0.50%** (50 bps) and **10.00%** (1,000 bps). |
| **Price impact** | How much your own order size shifts a pool's price due to limited liquidity. | Separate from slippage; routes above **15%** impact are rejected before signing. |
| **Priority fee** | Small extra network fee paid to Solana validators for faster inclusion. | Paid in SOL alongside the base transaction fee. |
| **Platform fee** | Optional 0.25% (25 bps) transfer fee on supported swaps. | Always fail-open—a fee error never blocks your swap. |
| **SPL / Token-2022** | Solana's standard and extended fungible token programs. | Both token standards are supported for bulk buys, sells, and ATA rent reclaim. |
| **Permit2** | Signature-based ERC-20 approval standard on EVM chains. | Enables one-click batched swaps on Robinhood Chain (`4663`) after one-time setup. |
| **RPC / SSE** | Remote Procedure Call (chain node) and Server-Sent Events (live push). | Pre-sign simulation runs against live RPC; **Confirming → Confirmed** updates push via SSE. |

---

## Portfolio and balances

| Term | Meaning | Why it matters here |
| :--- | :--- | :--- |
| **Dust** | Tiny leftover token balances that are tedious to sell one by one. | Use [Bulk Sell](../trading/bulk-sell-and-dust-reload.md) to sweep dust into SOL or ETH in one batch. |
| **ATA rent** | The `~0.00203928 SOL` locked in every Solana Associated Token Account. | Closing empty or burned token accounts returns this SOL to your wallet. |
| **Market cap (mcap)** | Circulating supply multiplied by current token price. | Used to gauge token size; never confuse mcap growth with your personal P&L. |
| **Liquidity (TVL)** | Total value locked in a token's trading pool. | Thin liquidity causes high price impact and makes exits difficult. |
| **Realized P&L** | Locked-in profit or loss from completed buys and sells. | Tracked per wallet and per chain on [P&L](../trading/pnl-history-and-portfolio.md). |
| **Unrealized P&L** | Live mark-to-market gain or loss on tokens you still hold. | Changes continuously with live prices until you sell. |
| **Confirmation (receipt-gated)** | A trade is marked **Confirmed** only after an on-chain receipt succeeds. | Reverted transactions are always marked **Failed**, never confirmed. |

---

## Risk metrics

| Term | Meaning | Why it matters here |
| :--- | :--- | :--- |
| **Drawdown** | Peak-to-trough percentage drop in a position or portfolio. | A **−50%** drop requires a **+100%** gain just to break even. |
| **Payoff ratio (R)** | Average winning trade divided by average losing trade. | Higher payoff ratios let a trader stay profitable even with a win rate below 50%. |
| **Break-even win rate** | Minimum win rate needed to break even: `1 / (1 + R)`. | See [Risk basics](../risk-and-mechanics/risk-basics.md) for worked examples. |

---

## Order types & execution comparison

| Order / Account Concept | Traditional CEX / MetaTrader | On reloadSol (Public Spot) |
| :--- | :--- | :--- |
| **Market order / Spot swap** | Fills against central order book | **Supported** — routes across on-chain DEX pools (Jupiter V2 / KyberSwap) |
| **Batch basket order** | Usually single-ticker only | **Supported** — bulk buy up to **4 tokens** or sweep dust in one coordinated batch |
| **Limit / Stop-Loss / Trailing order** | Resting exchange-side trigger | **Not offered** — non-custodial wallet; exit manually via Bulk Sell, Swap, or Fast Sell |
| **Margin / Leverage / Liquidation** | Borrowed collateral with liquidation price | **Not offered** — 100% fully-collateralized spot ownership; zero liquidation risk |
| **Simulated / Paper trading** | Demo brokerage account | **Supported** — browser-local **Paper mode** (`1 SOL + 1 ETH/day`, no wallet needed) |

*Want to see how entry cost, partial exits, flat fees, ATA rent, and drawdown add up in practice?* Read the [Worked trade P&L example](../trading/pnl-history-and-portfolio.md#worked-trade-example-pl-fees--drawdown).

---

## Coming from a CEX, MetaTrader, or TradingView

| CEX / MT term | On reloadSol | What changes |
| :--- | :--- | :--- |
| **Market order** | **Spot swap** | Executes against on-chain DEX pools via an aggregator. |
| **Limit / stop order** | **Not offered** | No exchange-side resting orders; you exit by selling from your wallet. |
| **Spread** | **Price impact + pool fee** | Depends on pool depth and your trade size. |
| **Account balance** | **Wallet balance** | Non-custodial—reloadSol never holds your funds. |
| **Margin / leverage** | **None** | 100% spot collateral; zero liquidation risk. |

{% hint style="warning" %}
**What reloadSol does not offer:** reloadSol public trading surfaces are **spot, fully-collateralised, non-custodial** market execution. There is **no margin, no leverage, no liquidation, no resting limit order, and no exchange-side stop-loss or take-profit order**. Your position is a token in your own wallet and you exit by selling it.
{% endhint %}

---

## Related

* [Quickstart Guide](quickstart.md)
* [Risk basics for spot traders](../risk-and-mechanics/risk-basics.md)
* [Order Execution, Slippage & Fees](../trading/order-execution-and-mechanics.md)
