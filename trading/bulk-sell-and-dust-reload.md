---
description: Liquidate multiple tokens, sweep wallet dust into SOL/ETH or any custom token, and reclaim Solana Associated Token Account (ATA) rent in a single workflow.
---

# Bulk Sell, Dust Sweep & Rent Reclaim

**Routes:** `/sell`, `/sell/solana`, `/sell/robinhood` (and default post-connect destination from `/`)

Bulk Sell—also referred to as **Reload**—is reloadSol's flagship public surface. Instead of swapping tokens one by one and manually closing leftover token accounts, you can liquidate an entire portfolio slice and reclaim locked rent in one coordinated flow.

---

## How Token Categorization Works

When `/sell` loads your holdings, every token account is classified into one of five buckets so you never accidentally burn an NFT or waste gas on a dead pool:

| Category | Meaning | Available Actions |
| :--- | :--- | :--- |
| **Sellable** | Positive balance with an active DEX route under the price-impact ceiling (`SWAP_QUOTE_MAX_IMPACT_PCT`, default `15%`). | Swap to SOL/ETH (or custom output token) + auto-close ATA on 100% sell (Solana). |
| **Dust** | Small-value sellable tokens below your dust threshold. | One-click **Dust only** filter to sweep small remainders. |
| **Zero Balance (Solana)** | Empty SPL or Token-2022 accounts (`amount = 0`) still locking `~0.00203928 SOL` in rent-exempt reserves. | **Close & Reclaim Rent** directly—no swap required. |
| **Unsellable / Illiquid** | Tokens whose liquidity pool is drained, rugged, or has no valid quote. | On Solana: optional **Burn & Close** to destroy worthless tokens and reclaim ATA rent. |
| **Frozen / NFT** | Frozen token accounts (cannot be transferred/burned) or NFTs (`decimals = 0, amount = 1`). | Protected and excluded from bulk selection by default. |

---

## Step-by-Step Workflow

1. **Filter & Select:**
   * Toggle **Show zero balance** to include empty rent-bearing accounts.
   * Click **Dust only** or **Select All** from the responsive action bar, or pick individual rows.
   * Adjust sell size per token using quick presets (`10%`, `20%`, `50%`, `75%`, `100%`) or exact input.
2. **Choose Output Asset:**
   * Default output is native **SOL** (Solana) or **ETH** (Robinhood Chain).
   * On the full `/sell` page, you can select a **custom output mint/contract** (e.g., USDC, USDG, WETH, or any valid CA). Any selected token matching the output mint is automatically skipped.
3. **Auto-Quote & Debounce:**
   * Editing token selections or percentages triggers a debounced (`150 ms`) quote refresh keyed by `sellQuoteKey`, with in-flight queueing (`pendingQuoteRunRef`) so rapid edits never display stale quotes.
4. **Execute & Reclaim:**
   * **Solana:** Swap legs execute via Jupiter Swap V2 (`/order` + `/execute`), and close/burn-close instructions are packed by **measured serialized transaction size** (~`10–11` accounts per transaction, supporting both SPL Token and Token-2022 with explicit `setComputeUnitLimit`).
   * **Robinhood Chain:** Parent wallets execute through KyberSwap + `BatchExecutor` (one atomic `executeBatch` tx) → EIP-5792 → sequential fallback; bound wallets execute through GMGN Trade API.

---

## Solana ATA Rent Reclaim Mechanics

Every SPL or Token-2022 token you have ever held on Solana requires an **Associated Token Account (ATA)**, which locks `2,039,280 lamports` (`~0.00203928 SOL`) as rent-exempt reserve.

* **Normal 100% Sell:** When you sell 100% of a token, its ATA becomes empty (`balance = 0`), allowing reloadSol to close the account and return the rent to your wallet.
* **Burn & Close (Illiquid / Rugged Tokens):** For tokens with zero liquidity, reloadSol packs a `burn` + `closeAccount` instruction pair into a measured chunk (Token-2022 burn+close requires ~`5,669 CU`).
* **Split-on-Revert Protection:** Before opening your wallet prompt, `simulateExact` tests every packed close chunk. If one account in a chunk reverts (e.g., a token with transfer hooks or frozen state), reloadSol isolates and removes the failing account *before* signing so the remaining accounts close cleanly in one approval.

{% hint style="warning" %}
**Irreversible Action:** Burning an unsellable token permanently destroys the token balance in order to reclaim the `~0.00204 SOL` ATA rent. Only burn tokens you have verified are worthless or rugged.
{% endhint %}
