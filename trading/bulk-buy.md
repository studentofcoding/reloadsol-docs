---
description: Allocate native SOL, ETH, WETH, or USDG across multiple tokens in a single coordinated batch with built-in risk screening and affordability checks.
---

# Bulk Buy & Multi-Token Allocation

**Routes:** `/buy`, `/buy/solana`, `/buy/robinhood`

Bulk Buy lets you deploy capital across multiple tokens simultaneously from a single total spend amount, eliminating repetitive single-swap workflows during fast-moving market windows.

---

## Key Capabilities

* **Multi-Token Batch Cap:** Buy up to **4 tokens per batch** on Solana (`MAX_TRADE_TOKENS_SOL = 4`) and **4 tokens per batch** on Robinhood Chain (`MAX_TRADE_TOKENS_RH = 4`, bounded by the `BatchExecutor` UX) — see `src/utils/trade-ui-limits.ts`. Each token needs at least **`$0.50`** of the total spend (`MIN_BUY_USD_PER_TOKEN`), and the slider floor is derived from that so a 1 % step can never fall under it.
* **Flexible Input Parsing:** Paste one or multiple contract addresses (CAs) separated by spaces, commas, or newlines. ReloadSOL parses and validates each address into interactive token chips with live logos, symbols, and prices.
* **One-Click Append from Discovery:** Tokens from trending boards, signal toasts, or search can be appended directly into your active buy basket.
* **Built-In Risk Analysis (`RiskAnalysis.tsx`):** Before you execute, ReloadSOL surfaces token risk indicators (holder concentration, liquidity depth, rug signals, and metadata anomalies) inline for each token in the basket.
* **Supported Spend Assets:**
  * **Solana:** `SOL`
  * **Robinhood Chain:** `ETH`, `WETH`, or `USDG`

---

## Smart Quoting, Warm-on-Intent & Affordability Gates

To deliver fast execution without exhausting RPC or Jupiter rate limits, Bulk Buy uses two coordinated safeguards:

1. **Warm-on-Intent Quoting (`useWarmOnIntent`):**
   * Rather than hammering the execution quote API on every keystroke, ReloadSOL shows fast display estimates via the shared quote engine (`purpose: 'estimate'`) and only warms execution quotes after `1.5 seconds` of input idle or when your cursor/focus reaches the **Buy** button.
   * Clicking **Buy** always builds fresh `purpose: 'execute'` quotes—stale display estimates are never signed.
2. **Pre-Quote Affordability Gate (`warmAffordability` & `requiredBuySol`):**
   * On Solana, ReloadSOL checks your native SOL balance before warming or signing to ensure you can cover:
     $$\text{Required SOL} = \text{Total Swap Spend} + \text{Priority Fees} + (N_{\text{new ATAs}} \times 0.00203928\text{ SOL}) + \text{Base Tx Fees}$$
   * Empty or underfunded wallets are caught cleanly with clear user messaging (`describeBuyFailure`) instead of triggering cryptic `422 Insufficient funds` aggregator errors.

---

## Execution Flow by Network

{% tabs %}
{% tab title="Solana Mainnet" %}
1. ReloadSOL requests fresh Jupiter Swap V2 `/order` transactions for each token leg in the basket (falling back to Jupiter Lite only if V2 fails).
2. Routes exceeding `SWAP_QUOTE_MAX_IMPACT_PCT` (default `15%`) are rejected to protect you from illiquid traps.
3. All prepared v0 transactions pass `simulateExact` with their live blockhash.
4. Your wallet prompts once via `signAllTransactions`.
5. Signed transactions are submitted through Jupiter `/execute` (managed landing, ~`1.5s–2.1s` end-to-end) or direct Shyft/RPC fallback with WebSocket confirmation.
6. Each leg is tracked independently in `/history` and `/pnl`.
{% endtab %}

{% tab title="Robinhood Chain (4663)" %}
1. **Parent Wallet (Rabby / EIP-6963):**
   * Fetches parallel KyberSwap `/routes` and `/builds` for all legs.
   * **BatchExecutor Mode (Recommended):** Wraps ETH if needed, pulls tokens via Permit2, and executes all `N` swaps inside **one atomic `executeBatch` transaction**.
   * **EIP-5792 Mode:** Uses `wallet_sendCalls` + `waitForCallsStatus` when the wallet reports `atomic: supported/ready` for chain `4663`.
   * **Sequential Fallback:** Prompts per-leg Approve/Swap with explicit index attribution (`RhSequentialWriteError`), ensuring partial fills are accurately recorded and reconcilable.
2. **Bound Wallet (Operator / GMGN):**
   * Executes sequentially through `/api/gmgn/trade/quote` → `/api/gmgn/trade/swap` → `/api/gmgn/trade/order` polling.
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**Dev Shortcut:** When connected with a dev wallet, `/buy` also displays a compact link to `/dev/insight` for the per-network Data-Public Scout and Macro Climate indicator.
{% endhint %}
