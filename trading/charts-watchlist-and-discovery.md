---
description: Analyze tokens with live OHLC charts, manage cross-chain watchlists, execute 1-click chart buys, and inspect relational token topology in the Knowledge Graph Explorer.
---

# Charts, Watchlist & Market Discovery

reloadSol pairs its execution tools with real-time charting, cross-chain watchlists, and visual market intelligence so you can research and act without switching tabs.

---

## Token Chart Page (`/chart/[tokenAddress]`)

Opening `/chart/<mintOrContract>` gives you a unified trading view for both **Solana SPL/Token-2022 mints** and **Robinhood Chain 42-character EVM (`0x...`) contracts**:

* **Live Candlestick Charting:** Powered by `lightweight-charts` and embedded GMGN charts, backed by reloadSol's own 1-minute `token_ohlc_bars` sampler when upstream chart feeds miss.
* **Live Metadata & Decimals:** Resolves live USD price, market cap, liquidity, and exact token decimals (`fetchRhChartTokenInfo` on Robinhood; Jupiter/GMGN metadata on Solana).
* **"Your Position" Card:** Automatically displays your live balance, average entry, and unrealized P&L for the token on **both** Solana and Robinhood Chain.
* **Chart Buy Modal (`ChartBuyModal.tsx`):** Keyboard-navigable quick-buy overlay available directly from chart, signal, and trend boards so you can enter a position without leaving the chart.

---

## Cross-Chain Watchlist

Track tokens across both Solana and Robinhood Chain in one persistent watchlist (`wallet_watchlist`, unique per `(wallet, token, chain)`):

* **Live Price Streaming & % Change:** Real-time USD marks and percentage moves since the moment you added the token.
* **Chain-Aware Cost Basis:** Captures initial reference price at watch time via Jupiter/GMGN on Solana and UniV2/DexScreener on Robinhood Chain.
* **One-Click Watch from Search:** Add tokens directly from search results or chart headers.
* **Daily Runners Global Watchlist Bar:** Highlights top daily movers across the platform so active traders can jump straight into charts or bulk buys.

---

## Public Knowledge Graph Explorer (`/knowledge-graph` & `/analytics/knowledge-graph`)

reloadSol exposes an interactive **Ontodia-style Visual Knowledge Graph** that lets traders inspect hidden on-chain relationships before entering a trade:

* **Deployer Provenance ("Dev DNA"):** See whether a token's creator has a history of successful graduations or serial rug pulls.
* **Holder & Cabal Topology:** Visualize clusters of wallets funded by the same source or buying in the same atomic slot (bundler rings).
* **Provenance Honesty:** If live holder or deployer feeds have no data for a freshly discovered token, the explorer explicitly displays an empty-graph provenance banner (`UNKNOWN_DEPLOYER` / `dataQuality`) rather than substituting fake nodes—synthetic graphs appear only when you explicitly pick a demo preset.

---

## Analytics & Blog

* **Market Analytics (`/analytics`):** High-level market metrics, trending flow summaries, and knowledge graph links.
* **Research & Release Blog (`/blog`, `/blog/[slug]`):** Platform announcements, trading mechanics explainers, and empirical market research posts.
