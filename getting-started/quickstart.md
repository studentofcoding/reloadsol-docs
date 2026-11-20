---
description: Go from zero to your first dust reload, bulk buy, or portfolio P&L check on ReloadSOL in under 3 minutes.
---

# Quickstart Guide

This guide walks you through connecting a wallet, checking recoverable value before signing in, executing your first **Dust Reload** or **Bulk Buy**, and verifying settlement in your **P&L Tracker**.

---

## Before You Connect: Free Wallet Check

You do not need to connect a wallet to inspect how much recoverable SOL is sitting in your account.

1. Open the ReloadSOL landing page (`/`).
2. Paste any Solana wallet address into the **Wallet Check** input.
3. ReloadSOL queries `/api/wallet/preview` (backed by Jupiter Portfolio) and displays:
   * **Native SOL balance**
   * **Sellable token value** (tokens with active DEX liquidity)
   * **Reclaimable ATA rent** (~`0.00203928 SOL` per empty Associated Token Account)

{% hint style="tip" %}
**Stuck with 0 SOL for gas?** If your wallet has `0` (or near-zero) native SOL but holds recoverable tokens or closable accounts, the Wallet Check displays an **Ask for Fund** button. Submitting a request notifies the operations desk (rate-limited and verified server-side against your live portfolio snapshot) to assist with initial gas so you can recover your assets.
{% endhint %}

---

## Step-by-Step: Your First Dust Reload

{% stepper %}
{% step %}
### Connect Your Wallet

Click **Connect Wallet** in the top navigation bar.

* **Solana:** Select Phantom, Solflare, Backpack, or another Wallet Adapter wallet.
* **Robinhood Chain (EVM `4663`):** Select Rabby or another EIP-6963 browser wallet (available when your wallet is whitelisted or an EVM provider is active).

Once connected, ReloadSOL automatically routes you to `/sell/solana` (or `/sell/robinhood` based on your active network).
{% endstep %}

{% step %}
### Review Categorized Holdings

ReloadSOL scans your wallet and automatically groups every token into actionable categories:

* **Sellable:** Tokens with an active swap route and sufficient liquidity.
* **Unsellable / Low Liquidity:** Tokens with no viable DEX route or excessive price impact.
* **Zero Balance (Solana):** Empty SPL or Token-2022 accounts holding reclaimable SOL rent.
* **Frozen / NFT:** Protected or non-fungible accounts excluded from accidental liquidation.
{% endstep %}

{% step %}
### Select Tokens & Output Asset

1. Use **Dust only**, **Select All**, or click individual rows (with quick percentage presets: `10%`, `20%`, `50%`, `75%`, `100%`).
2. Keep the default output asset (**SOL** on Solana, **ETH** on Robinhood) or select a custom output token mint/address on the full `/sell` screen.
3. Wait briefly for live quotes to load. ReloadSOL displays estimated output, price impact, and slippage before you sign.
{% endstep %}

{% step %}
### Simulate, Sign & Confirm

1. Click **Sell / Reload**.
2. ReloadSOL simulates every transaction leg against current RPC state and checks for stale blockhashes or reverting accounts.
3. Approve the transaction batch in your wallet:
   * **Solana:** One `signAllTransactions` approval for the batch (and packed close-account chunks).
   * **Robinhood Chain:** One atomic `executeBatch` signature (after one-time Permit2 setup) or sequential Approve + Swap prompts.
4. Watch the confirmation modal transition from **Submitting & confirming on-chain…** to **Confirmed** with direct Solscan / Blockscout transaction links.
{% endstep %}
{% endstepper %}

---

## Quick Navigation Cheat Sheet

| Goal | URL Route | Notes |
| :--- | :--- | :--- |
| **Sweep dust & reclaim rent** | `/sell` (`/sell/solana`, `/sell/robinhood`) | Default post-connect landing page |
| **Buy multiple tokens at once** | `/buy` (`/buy/solana`, `/buy/robinhood`) | Up to 4 tokens per batch (`MAX_TRADE_TOKENS_SOL` / `_RH`), split spend, min `$0.50`/token |
| **Single-pair swap** | `/swap` (`/swap/solana`, `/swap/robinhood`) | Jupiter Terminal (Solana) / RhSwap (Robinhood) |
| **Inspect a token chart & trade** | `/chart/<tokenAddress>` | Live chart, position card, and Quick Buy modal |
| **Track realized & unrealized P&L** | `/pnl` | Supports Real/Sim filters and inline Fast Sell |
| **Audit transaction history** | `/history` | Per-wallet, per-chain settlement log with explorer links |
| **Explore token relationships** | `/knowledge-graph` | Visual deployer, holder, and cabal topology |
| **Open Developer & Quant Tools** | `/dev` and `/admin` | Dev-wallet gated; see [Dev Console Overview](../dev-console/overview-and-access.md) |
