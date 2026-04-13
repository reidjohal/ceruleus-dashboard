# Ceruleus Capital — Morning Dashboard
## Project Memory for Claude Code

---

## What This Is
A personal morning trading dashboard for Reid Johal (Ceruleus Capital family office).
Live at: **dashboard.ceruleuscapital.com**
GitHub: **github.com/reidjohal/ceruleus-dashboard**
Deployed via: GitHub Pages (auto-deploys on push to main)

---

## Infrastructure
- **Hosting:** GitHub Pages — push to main = live in ~60 seconds
- **API proxy:** Cloudflare Worker at `ceruleus-api.reidjohal.workers.dev`
  - Holds the Anthropic API key server-side (env var: `ANTHROPIC_API_KEY`)
  - Proxies Claude API calls from the dashboard
- **Domain:** ceruleuscapital.com on Cloudflare DNS (nameservers: byron + raina)
- **Other subdomains:**
  - `journal.ceruleuscapital.com` → Railway (trade journal, Node/Express/PostgreSQL)
  - `ceruleuscapital.com` → Wix (main public site, not yet built)

---

## File Structure
Single file app — everything lives in `index.html`.
```
ceruleus-dashboard/
├── index.html        ← entire app (HTML + CSS + JS)
├── CLAUDE.md         ← this file
└── worker.js         ← Cloudflare Worker source (reference only, deployed separately)
```

---

## Design System
**Fonts:** DM Mono (data/labels), Cormorant Garamond (headlines), Space Grotesk (body)
**Background:** `#0d0f14`
**Text:** `#e8e6e0`
**Accent colors:**
- Teal: `#4dba87` (bull/positive)
- Blue: `#6e9ef5` (neutral)
- Red: `#e07070` (bear/negative)
- Purple: `#a97de8` (tail risk)
- Gold: `#f5c35a` (high-impact events)
- Orange: `#f0a040` (cumulative lines)

**Key rules:**
- NO rounded corners anywhere (`border-radius: 0` throughout)
- Borders: `0.5px solid rgba(255,255,255,0.08)` for cards
- Top accent border on scenario cards: `2.5px solid [color]`
- Rainbow bar at very top: gradient across all accent colors
- Nav: logo left, nav items right, both at 28px padding from edge

---

## App Structure (Two Pages)

### Page 1: News Feed (`#page-news`)
- **Top nav pill:** "Scenarios + Calendar + Drivers · 1/1" (left) + trading week (right)
- **Headline:** Large Cormorant Garamond, colored accent spans
- **Subhead:** Small muted context text
- **Regenerate Scenarios button:** Full width, square, calls Cloudflare Worker → Claude API
- **4 scenario cards (2x2 grid):**
  - bull (teal top border)
  - neutral (blue top border)
  - bear (red top border)
  - tail (purple top border)
  - Each has: title, probability badge (bold, no ~ symbol), trigger, description, asset row (S&P · BTC · Oil)
  - Asset labels AND values are bold and match card accent color
- **Calendar strip:** 5 columns, Mon–Fri of current trading week
  - Classes: `has-event`, `high-impact` (gold border), `war-event` (red border)
- **Drivers row:** 2 columns — Equity Drivers + Bitcoin Drivers
  - Title: white, bold, DM Mono uppercase
  - ▲ up arrows: teal, ▼ down arrows: red
- **Footer:** `@ceruleus_capital` left, "Scenarios + Calendar · Ceruleus Capital" right

### Page 2: Market Data (`#page-data`)
All 5 charts on one page. Charts only initialise when user first visits the page.
Layout: 2 charts side by side (Row 1), 2 charts side by side (Row 2), 1 full-width chart (Row 3).

**Charts:**
1. **Quarterly Basis** — bar chart showing annualized basis, exchange tabs (Deribit/OKX/Binance), live API data
   - Deribit: Fetches up to 3 quarterly contracts (Jun/Sep/Dec style naming)
   - OKX: Quarterly + Biquarterly contracts via BTC-USD futures
   - Binance: Quarterly + Biquarterly via COIN-margined delivery futures
   - Shows spot price + annualized basis calculation: `((futures - spot) / spot) * (365 / days) * 100`
2. **Market Spread** — line chart, BTC/ETH asset toggle + time tabs 1D/1W/1M
   - Colors: $1K = white `#e8e6e0`, $25K = light blue `#93c5fd`, $100K = blue `#3b82f6`
   - ETH spreads use tighter base values than BTC
3. **Volume by Contract** — stacked bar + cumulative line, BTC spot/perp/quarterly, teal/blue/gold bars + orange line
4. **Volume Outliers** — bar chart, outlier bars highlighted orange, cumulative overlay, time tabs: 3M/6M/1Y
5. **Volume by Region** — stacked bar + cumulative line, Asia/US/EU, full width, teal/blue/gold + orange line

**Chart card anatomy:**
- Header: title + subtitle (left), tabs (right)
- Stat cards row: border-left connected boxes, DM Mono values
- Canvas: Chart.js, dark grid, y-axis on right, DM Mono tick labels
- Legend: small colored lines/dots with labels

**Chart defaults:** dark tooltips, right y-axis, no legend, `rgba(255,255,255,0.04)` grid lines

---

## Scenario Generation (Claude API)
- Prompt instructs Claude to return strict JSON with: headline, subhead, 4 scenarios, equity_drivers, btc_drivers
- Probabilities: no ~ symbol, just "X%"
- Headline: 2 lines split by `|`, supports `[gold]`, `[teal]`, `[red]` color tags
- Trading week is computed once on load and locked — never changes on regenerate
- On failed generation: shows error banner, restores button text

---

## What's Done
- [x] News dashboard fully designed and live
- [x] Scenario generation via Claude API working
- [x] Custom domain dashboard.ceruleuscapital.com live
- [x] Nav with News Feed + Market Data dropdowns
- [x] Market Data page with all 5 charts (random data)
- [x] Spread chart has BTC/ETH toggle and blue color scheme
- [x] Quarterly Basis wired to live APIs (Deribit, OKX, Binance) with exchange toggle

## What's Next
- [ ] Wire Spread to live Binance order book API
- [ ] Wire Volume by Contract to live Binance API
- [ ] Wire Volume Outliers to live Binance OHLCV API
- [ ] Wire Volume by Region to live multi-exchange APIs
- [ ] Add web search to Worker so scenarios auto-pull current news context
- [ ] Add Google Calendar OAuth so calendar strip pulls real events

---

## Conventions
- Always `border-radius: 0` — no exceptions
- Git workflow: `git add . && git commit -m "message" && git push`
- Never expose API keys in index.html — all API calls go through the Cloudflare Worker
- Chart.js loaded from cdnjs.cloudflare.com CDN
- Charts use `chartsInitialised` flag — only init on first Market Data visit
