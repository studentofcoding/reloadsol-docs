---
description: Technical audit of reloadSol's pre-existing repository documentation, identified gaps, information architecture redesign, and ongoing maintenance standards.
---

# Documentation Audit & Architecture Blueprint

This page documents the structural audit performed on reloadSol's pre-existing `docs/` library and explains the information architecture decisions behind this GitBook.

---

## 1. Executive Summary of Pre-Existing Docs Audit

Prior to building this GitBook, reloadSol possessed rich, deeply accurate engineering material—5 condensed subsystem docs (`docs/01`–`05`), **132** implementation `SPEC-*.md` files, **72** incident/analysis reports (`docs/reports/`), **13** deep research notes (`docs/research/`), and **26** HTML diagrams—but **no unified, navigable, trader-facing or developer-onboarding documentation site**.

### Key Findings from the Audit

| Audit Dimension | Finding in Legacy `docs/` | Impact on Readers | Resolution in This GitBook |
| :--- | :--- | :--- | :--- |
| **Audience Separation** | Public trader workflows (`/sell`, `/buy`, `/swap`, `/pnl`) were intermixed with internal Postgres migrations, Go cron internals, and bug post-mortems. | Traders struggled to find simple answers on slippage, ATA rent reclaim, or wallet setup; developers had to grep 90+ specs. | Split cleanly into **Getting Started → Public Trading Platform → Risk Management → Dev & Quant Console → Developer Reference**. |
| **Navigation & Structure** | No `SUMMARY.md`, no `.gitbook.yaml`, and no progressive disclosure; 150+ flat files across `docs/`, `docs/specs/`, and repo root (`architecture.md`, `PNL_AUTOMATION_SETUP.md`, empty `OPERATIONS_TRACKING_GUIDE.md`). | High cognitive load; impossible to browse linearly or publish directly to GitBook Git Sync. | Added `.gitbook.yaml` + hierarchical `SUMMARY.md` with **24** focused, cross-linked pages across 6 tracks and zero dead links (verified: 24 refs, 0 missing, 0 broken intra-page links). |
| **Missing Trader Guides** | No step-by-step guides for **Wallet Check + Ask-for-Fund**, **Bulk Buy affordability reserves**, **Robinhood Permit2 1-click setup**, **Dynamic Slippage**, or **In-Flight OHLC 3-Pattern Defense** in plain trading language. | Users misunderstood Phantom simulation behavior, Robinhood approval prompts, and fee drag on small stakes. | Authored dedicated step-by-step guides with GitBook `{% stepper %}`, `{% tabs %}`, `{% hint %}` callouts, and worked fee/payoff tables. |
| **Spec Drift & Retired Systems** | Several older root/archive docs referenced retired systems (`Supabase`, `v2-gate` / `v2-potential` ML heads, `Raptor` as desk executor, `-50%` RH stop fallback). | Risk of onboarding engineers or traders onto deprecated workflows. | Reconciled every page against current production code (`src/config/route-network.ts`, `src/config/api-access.ts`, `docs/01`–`05`, and Nov 2026 specs) and explicitly marked retired components. |
| **Visual Clarity** | Mermaid diagrams and sequence flows were missing from trader-facing docs (existing diagrams were standalone internal HTML files). | Execution lifecycle, BatchExecutor precedence, and 4-phase runner graduation were hard to grasp quickly. | Embedded clean, GitBook-native Mermaid flowcharts and sequence diagrams across Architecture, Execution, Risk, and Lifecycle pages. |
| **Terminology Standardization** | Terms like `ATA`, `OOR`, `MFE/MAE`, `Payoff Ratio`, `SHACL`, `Shadow vs. Enforce`, and `Bound vs. Parent Wallet` were used without centralized definitions. | Steep learning curve for newer traders and newly onboarded devs. | Added a centralized [Trading & Platform Glossary](glossary.md) covering both FinTech/trading and reloadSol-specific terms. |

---

## 2. Information Architecture Map

```mermaid
flowchart TD
    Root["gitbook/README.md<br/>(Platform Overview & Track Selector)"]
    Root --> GS["1. Getting Started<br/>Quickstart · Wallets & Chains · Paper & Safety"]
    Root --> PT["2. Public Trading Platform<br/>Bulk Sell/Reload · Bulk Buy · Single Swap<br/>Execution/Slippage/Fees · Charts/Watchlist · P&L/History"]
    Root --> RM["3. Risk Management & Mechanics<br/>SL/TP & OHLC Defense · Rug/KG Gates · Regime & Sizing"]
    Root --> DC["4. Dev & Quant Console (/dev & /admin)<br/>Overview · Strategies/Algo Tester · DLMM/CLMM/Arb<br/>ML & Admin Eval · Search/RPC/System Health"]
    Root --> DR["5. Developer Reference<br/>System Architecture · REST/SSE API<br/>Contracts & Cron · Ops & Deployment"]
    Root --> RES["6. Resources<br/>Trading Glossary · Documentation Audit"]
```

---

## 3. Publishing & Maintenance Standard

The docs publish through **two hops**, and only the first is a repository concern:

```
gitbook/  ──npm run docs:sync──▶  studentofcoding/reloadsol-docs  ──▶  GitHub Pages (/widgets)
              (this repo)                  (public mirror)      └─▶  GitBook Git Sync (prose)
```

### The one command

**Any change under `gitbook/` ends with:**

```bash
npm run docs:sync      # link gate → mirror → commit → push
npm run docs:check     # same, prints the diff, stops before commit
```

`scripts/sync-gitbook-docs.sh` runs a link gate (every page reachable from `SUMMARY.md`, no orphan, no
broken intra-page link), mirrors with `rsync --delete` so deletions propagate, asserts the two
Pages-critical files survived, then commits and pushes to the mirror. A re-run with no changes prints
*"already up to date"* — it is idempotent.

{% hint style="warning" %}
**Two files are load-bearing, and their absence is a site-wide 404.**
GitHub Pages builds the mirror with the legacy **Jekyll** pipeline, and GitBook's
`{% hint %}` / `{% tabs %}` / `{% embed %}` syntax is **Liquid** to Jekyll. 61 such tags failed every
build — "Page build failed." — so nothing was served, including the self-contained
`/widgets/index.html` interactive lab.

* **`.nojekyll`** (repo root) serves the tree as-is. Markdown is rendered by GitBook, never by Pages.
* **`index.html`** (repo root) is the site landing page; without it the root 404s because there is no
  document to render.

`npm run docs:sync` refuses to publish if either is missing after the mirror.
{% endhint %}

### Rules

1. **Never hand-edit `reloadsol-docs`.** It is a mirror; the next sync overwrites it.
2. **A page must be reachable from `SUMMARY.md`** — an unreferenced page is unpublished, and the gate
   fails rather than silently dropping it.
3. **Do not add a GitHub Actions workflow for this.** Actions is parked on the account's billing limit
   (`ci.yml`, 2026-10-28) and stays that way by choice; the local command is the mechanism.
4. **Keep docs coupled to changes.** When a route is added to `src/config/route-network.ts` or
   `src/config/api-access.ts`, or a worker is registered in `main.go` / `worker_tracker.go`, update the
   corresponding table in `developer-reference/api-reference.md` or `smart-contracts-and-cron.md` in the
   same change.

GitBook's own Git Sync (space settings → Git Sync → `studentofcoding/reloadsol-docs`, branch `main`,
directory `/`) is a **one-time UI setup** on the GitBook side; it needs nothing from this repository and
polls on its own schedule.

### Still outstanding

**Annotated UI screenshots** — deliberately not fabricated; capture into `gitbook/.gitbook/assets/` for
`/` (Wallet Check), `/sell/solana` (dust categories, burn-close modal), `/swap/robinhood` (Permit2
readiness, receipt modal), `/pnl` (inline Fast Sell), and the three consoles (`/dev/algo-tester`,
`/admin/lifecycle`, `/analytics/knowledge-graph`).
