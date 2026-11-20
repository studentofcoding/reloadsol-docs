---
description: How ReloadSOL manages dual-chain switching between Solana Mainnet and Robinhood Chain (EVM 4663), wallet authentication, and account modes.
---

# Wallets & Supported Networks

ReloadSOL operates across two distinct execution environments—**Solana Mainnet (`sol`)** and **Robinhood Chain (`robinhood`, EVM Chain ID `4663`)**—while keeping balances, watchlists, trading records, and strategies strictly isolated by `(wallet, chain)`.

---

## Network Switching (`AppNetwork`)

Your active chain is stored in browser `localStorage` under `reloadsol.appNetwork` (`'sol' | 'robinhood'`) and enforced across both UI routes and backend API queries:

* **Route Gating:** Every trading and dev route is registered in the trade-route registry (`src/config/route-network.ts`). Attempting to open a Solana-only route (such as `/dev/rpc-tester` or `/dev/arbitrage`) while on Robinhood redirects safely to the active chain's default surface.
* **Data Isolation:** Watchlists, trading history, P&L calculations, and strategy definitions are all stamped with a `chain` column (`sol` or `robinhood`). Solana and Robinhood trade histories never bleed into each other.

{% hint style="warning" %}
**Robinhood Chain Access Gating:** Robinhood Chain (`4663`) is currently gated to authorized wallets. Access is automatically enabled when an EVM browser provider (such as Rabby) is detected, or when your connected Solana/EVM address is on the `RH_WHITELIST` / dev-wallet allowlist. Otherwise, selecting Robinhood automatically reverts to `sol`.
{% endhint %}

---

## Wallet Modes by Chain

### Solana Mainnet (`sol`)

* **Standard Browser Wallets:** Phantom, Solflare, Backpack, and any wallet supported by `@jup-ag/wallet-adapter`.
* **Session Authentication:** Connecting a Solana wallet establishes a signed wallet session cookie (`WALLET_SESSION_SECRET`), authorizing wallet-tier API endpoints (`/api/rpc`, `/api/jupiter/*`, `/api/trading/records`, `/api/watchlist`, `/api/operations/*`).
* **Phantom In-App Browser Reliability:** ReloadSOL includes an explicit connect-intent self-healing hook (`useCompleteConnectIntent`) to prevent connection stalls when tapping already-installed wallets inside mobile in-app browsers.

### Robinhood Chain (`robinhood` — EVM `4663`)

Robinhood Chain supports two distinct execution wallet modes, automatically detected by `useRhWalletMode`:

| Wallet Mode | How It Signs | Swap Aggregator | Batching Capability |
| :--- | :--- | :--- | :--- |
| **Parent Wallet (Browser / Rabby / EIP-6963)** | User signs directly in browser wallet | **KyberSwap Aggregator** | Precedence: **BatchExecutor** (1 atomic tx) → **EIP-5792** (`wallet_sendCalls`) → **Sequential** (Approve + Swap per leg) |
| **Bound Wallet (Operator / GMGN)** | Server-signed via configured bound EVM key (`GMGN_PRIVATE_KEY`) | **GMGN Trade API** | Sequential per-leg quote → swap → order status poll; `from` strictly enforced to bound address |

{% hint style="info" %}
**Dual-Auth for EVM-Only Visitors:** Users connected only with an EVM `0x...` address on Robinhood Chain do not need a Solana session cookie to manage watchlists, view charts, or execute parent-wallet swaps. Watchlist and RH API routes accept verified `0x` wallet addresses directly.
{% endhint %}

---

## Token Holdings Discovery Pipeline

To keep balances fast and resilient against single-provider rate limits, ReloadSOL uses tiered holdings resolution:

### Solana Holdings (`/api/shyft/wallet/all_tokens`)
1. **Primary:** Cached Shyft `all_tokens` (15-second fresh TTL / 120-second stale-while-revalidate; immediately bypassed with `fresh=1` right after a confirmed trade).
2. **Fallback 1:** Jupiter Portfolio API (`/api/jupiter/portfolio`).
3. **Fallback 2:** Direct RPC `fetchUserTokens` scan.

### Robinhood Chain Holdings (`/api/rh/wallet-tokens`)
1. **Primary:** Normalized GMGN holdings.
2. **Fallback 1:** Blockscout ERC-20 token index (`robinhoodchain.blockscout.com/api/v2/address/.../tokens`).
3. **Fallback 2:** Direct RPC ERC-20 `balanceOf` reads.
4. **Quote Injection:** Core quote assets (`ETH`, `WETH`, `USDG`) are always injected so you can trade immediately even with a `0` balance on one of them.

---

## Wallet Preview & "Ask for Fund" Gas Recovery

If a Solana trader visits `/` with a wallet that holds **sellable tokens or closable empty token accounts** but lacks **native SOL** to pay transaction fees:

1. Pasting the address on `/` calls `GET /api/wallet/preview`, which returns `nativeSol`, `sellableSol`, `reclaimSol`, and `sellableCount`.
2. When `nativeSol` is below the execution threshold while `sellableSol + reclaimSol > 0`, ReloadSOL surfaces the **Ask for Fund** flow (`POST /api/operations/ask-for-fund`).
3. **Server-Side Safeguards:**
   * Eligibility is **always re-derived server-side** from live Jupiter Portfolio data (client values are never trusted).
   * Protected by per-IP rate limiting, per-wallet cooldowns, a global daily cap, and proof-of-recoverable-value.
   * Requests are logged to `token_operations` and routed to the operations Telegram desk for manual gas assistance (there is no automated payout bot).
