---
description: Exact ReloadSOL design tokens, dark-mode palette, logo assets, and GitBook Site Customization settings so docs match the ReloadSOL trading terminal.
---

# Brand & GitBook Customization Guide

Use these exact design tokens (extracted from `src/app/globals.css`) in **GitBook → Site Settings → Customize** so `docs.reloadsol.app` matches the ReloadSOL trading app.

---

## 1. ReloadSOL Color Palette

| Token | Hex / Value | Usage in GitBook |
| :--- | :--- | :--- |
| **Primary Accent (`--accent`)** | `#A3E635` | Primary brand color, active links, buttons, callout highlights |
| **Strong Accent (`--accent-strong`)** | `#84CC16` | Hover states & light-mode primary tint |
| **Background (`--background`)** | `#000000` | Default **Dark Mode** canvas |
| **Surface (`--surface`)** | `#111113` | Sidebar & card surface |
| **Surface Elevated (`--surface-2`)** | `#1A1A1E` | Code blocks & interactive panels |
| **Border (`--border-soft`)** | `#2A2A2E` | Subtle dividers |
| **Positive / Profit (`--positive`)** | `#4ADE80` | Confirmed / profit states |
| **Negative / Loss (`--negative`)** | `#F87171` | Failed / veto / stop-loss states |
| **Warning (`--warning`)** | `#FBBF24` | Pending / warn states |

---

## 2. GitBook `Customize` Checklist

1. **Theme:** Set **Default color mode** to **Dark** (`#000000` / `#111113`), **Primary color** to **`#A3E635`**, and **Tint** to **Subtle**.
2. **Logo & Favicon:** Upload `.gitbook/assets/logo.png` as the site logo and `.gitbook/assets/toke_Icon.png` as the favicon.
3. **Header Links:**
   * **Launch App** → `https://reloadsol.app` (Primary CTA button)
   * **Bulk Sell** → `https://reloadsol.app/sell`
   * **P&L Tracker** → `https://reloadsol.app/pnl`
4. **Interactive OpenAPI:** Under **Integrations → OpenAPI**, link `openapi.yaml` from the repository root to activate the live **"Test it"** API playground.
