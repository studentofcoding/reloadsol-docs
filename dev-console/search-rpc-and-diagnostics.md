---
description: Cross-chain token search, TokenLocateHub Freeview maps, RPC endpoint health & send telemetry, and /admin/system health diagnostics.
---

# Token Search, RPC Diagnostics & System Health

This section covers reloadSol's operator research and infrastructure diagnostic surfaces.

---

## 1. Cross-Chain Token Search & `TokenLocateHub` (`/dev/search-token`)

**Routes:** `/dev/search-token`, `/dev/search-token/solana`, `/dev/search-token/robinhood`, and `/dev/search-token/detail?address=<ca>&view=<mode>`  
*(Legacy `/search-token*` and `/dev/token-search` routes redirect here.)*

* **Multi-Field Discovery:** Search by token name, ticker symbol, or exact contract address across Solana and Robinhood Chain.
* **Direct Actions:** Add to Watchlist, jump to `/chart/<ca>`, or open **TokenLocateHub**.
* **`TokenLocateHub` (`/dev/search-token/detail`):** Provides **Freeview** (visual price/activity map backed by `token_info_detect` and `token_ohlc_bars`) and **List** views for deep inspection of a token's detection history and market-cap trajectory.

---

## 2. RPC Tester & Standardized Endpoint Health (`/dev/rpc-tester`)

**Route:** `/dev/rpc-tester` (Solana-only)  
**APIs:** `GET /api/rpc/health`, `GET /api/rpc/diagnostics`, `GET /api/rpc/diagnostics?send=1`

reloadSol routes all Solana RPC traffic through a same-origin proxy (`/api/rpc`) with ranked failover and strict role separation (`SPEC-rpc-endpoint-standardization-v1`):

| Rank | Provider Role | Capabilities & Health Label |
| :--- | :--- | :--- |
| **Rank 0 (Send Lane)** | **Shyft RPC** | Primary transaction landing lane (measured `100%` landing in interleaved prod tests). Free-plan Shyft keys return `403 IndexNotAllowedOnFreePlan` on indexed reads, so `/api/rpc/diagnostics` accurately labels them **`send-only`** rather than falsely flagging `Error`. |
| **Rank ≥ 1 (Index & Read Lane)** | **Helius Mainnet** | Primary indexed-read and low-latency blockhash failover (~`13 ms` p50 blockhash, ~`52 ms` p50 indexed reads). |

### Proxy-Level Send Telemetry (`?send=1`)
Calling `GET /api/rpc/diagnostics?send=1` (or `?send=90` for a 90-day window) reports live landing rates per target lane (`via:jupiter`, `via:shyft`, etc.) recorded into `swap_tca` (migration `94-rpc-send-telemetry.sql`).

{% hint style="warning" %}
**Measured SolanaTracker RPC Rate Limits:** Do not burst concurrent requests against SolanaTracker RPC (`~4.9 req/s` sustained ceiling; bursts of 6 simultaneous requests return `6× 429` rather than queuing, and JSON-RPC batches count per inner call). Always pace serial requests at `~200 ms` and batch account reads via `getMultipleAccounts` (up to `100` accounts in a single call).
{% endhint %}

---

## 3. System Health (`/admin/system`) & Transactions (`/admin/transactions`)

* **`/admin/system`:** Live operational check across:
  * Container & Postgres / PgBouncer pool counters (`[db-pool]` active/idle/waiting vs. `DATABASE_POOL_MAX = 25`).
  * Go cron worker states (`ok`, `stale`, `error`, `never_run`, `disabled`, `offline`) sourced from `CRON_SERVICE_URL` (`http://cron:8080/workers`).
  * Entry-context snapshot coverage (`token_entry_context`), re-entry guard activity, and stake-floor compliance.
* **`/admin/transactions`:** Platform-wide transaction audit log, swap TCA (Transaction Cost Analysis), and per-user portfolio/session telemetry.
