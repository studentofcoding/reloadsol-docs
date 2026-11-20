---
description: Monitor open positions, calculate realized and unrealized P&L, execute inline Fast Sells, and audit every confirmed or failed transaction in real time.
---

# P&L Tracker, Trade History & Portfolio

**Routes:** `/pnl` and `/history`

reloadSol maintains a receipt-verified ledger of every manual and simulated trade, strictly scoped by **`wallet:chain`** so your Solana (`SOL`) and Robinhood Chain (`ETH`) performance never mix.

---

## P&L Tracker (`/pnl`)

The P&L Tracker (`PnLTracker.tsx`) reconstructs your portfolio directly from confirmed `trading_records` legs and live on-chain balances:

* **Open vs. Completed Positions:**
  * **Open Positions** are continuously pruned against your live on-chain token holdings (`pruneOpenPositionsByHoldings`), ensuring tokens you transferred out or sold elsewhere do not linger as phantom positions.
  * Native and quote assets (`SOL`, `ETH`, `WETH`, `USDG`) are automatically suppressed from the open-positions candidate list.
* **Realized vs. Unrealized P&L:**
  * **Realized P&L:** Matched across confirmed buy, sell, and close legs in native units (`SOL` / `ETH`) and USD using trade-time native prices (`solPriceUsd`), so historical USD records remain accurate even when SOL or ETH prices move later.
  * **Unrealized P&L:** Marked to live prices from the shared GMGN + Redis + SSE feed (with Jupiter / DexScreener / UniV2 fallbacks).
* **Real vs. Sim Filters:** Toggle cleanly between live on-chain executions and paper/simulated trades.
* **Inline Fast Sell:** Close an open position directly from `/pnl` without navigating to `/sell`:
  * **Solana:** Executes a native-SOL sell via Jupiter Swap V2.
  * **Robinhood Chain:** Executes an inline KyberSwap sell (`executeRhParentKyberSell`) via the universal `OpenSellModal`.

---

## Trade History (`/history`)

The History feed (`TradingHistory.tsx`) provides a full audit log of every `buy`, `sell`, and `close` operation:

| Column / Indicator | Meaning |
| :--- | :--- |
| **Operation Type** | `buy`, `sell`, or `close` (ATA rent reclaim / burn-close). |
| **Settlement Badge (`txStatus`)** | **Amber spinner ("Confirming…")** = `pending`; **Green** = `confirmed` (on-chain receipt succeeded); **Red** = `failed` (reverted, rejected, or batch non-success). |
| **Leg Breakdown** | `successCount / totalTokens` (e.g., `4 confirmed · 1 failed`) with per-token amounts, prices, and symbols. |
| **Financials** | Native amount spent/received, fees paid, and trade-time USD valuation. |
| **Explorer Links** | Direct links to **Solscan** (Solana) or **Blockscout** (Robinhood Chain) for every transaction signature or GMGN order hash. |

---

## Real-Time Updates (SSE + Offline Resilience)

You never need to refresh `/pnl` or `/history` after a trade:

1. Every confirmed or failed trade update invalidates the wallet's Redis cache (`records:<wallet>:*`) and broadcasts a **Server-Sent Event (SSE)** (`trade_update`, `pnl_update`, or `balance_update`) over `GET /api/trading/subscribe?wallet=<addr>`.
2. The client maintains a single deduplicated SSE stream per wallet with a `60-second` heartbeat staleness check, exponential backoff, and an automatic `8-second` polling fallback if SSE disconnects.
3. If your browser goes offline mid-session, `tradingTracker` queues unsynced records to a `wallet:chain`-scoped `localStorage` buffer and reconciles them automatically on reconnect.

{% hint style="info" %}
**Daily Automated P&L Rollup:** A scheduled background worker (`pnl_update`, daily at `02:00 UTC`) recomputes 30-day matched wallet P&L into `token_operations.trade_pnl` so long-term portfolio summaries stay fast and consistent.
{% endhint %}
