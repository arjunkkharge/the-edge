# The Edge — Claude Code Context

## What This Project Is
ICT prop trading journal and dashboard. Single HTML file app built for a $50K prop firm challenge. Tracks trades, enforces risk rules, syncs to cloud.

## Live App
- URL: https://arjunkkharge.github.io/the-edge
- GitHub: https://github.com/arjunkkharge/the-edge
- Upload: https://github.com/arjunkkharge/the-edge/upload/main

## Tech Stack
- **Single file**: `index.html` — no build step, no framework, no npm
- **Hosting**: GitHub Pages (free, auto-deploys on push)
- **Database**: Supabase (PostgreSQL, real-time, per-user RLS)
- **Auth**: Supabase email/password
- **Charts**: Chart.js (CDN)
- **Fonts**: Inter + DM Mono (Google Fonts)
- **Export**: XLSX, PDF, CSV, JSON (all CDN libraries)

## Supabase Config
- URL: https://aceffksoagyfnpwuccgm.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFjZWZma3NvYWd5Zm5wd3VjY2dtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzMyNDQ3NDQsImV4cCI6MjA4ODgyMDc0NH0._sSwzzOKhAt8xUxFVNXDIbspmvpTGcE929ZZTMm-yM8
- Tables: `trades` (per-user RLS), `user_settings` (per-user config)
- Auth users: arjunkharge@gmail.com (main), khargearjun@gmail.com (test)

## Deploy Workflow
**Option A — GitHub upload (no git):**
1. Go to https://github.com/arjunkkharge/the-edge/upload/main
2. Drag index.html → commit

**Option B — Git push (faster):**
```bash
git add index.html
git commit -m "your message"
git push
```
GitHub Pages auto-deploys in ~60 seconds.

## CSS Color Theme (Original — DO NOT CHANGE)
```css
:root {
  --bg: #080808;       /* main background */
  --bg2: #0f0f0f;      /* cards, sidebar */
  --bg3: #141414;      /* inputs, secondary */
  --b1: #1e1e1e;       /* borders */
  --b2: #2a2a2a;       /* hover borders */
  --b3: #363636;       /* active borders */
  --green: #00e676;    /* PRIMARY ACCENT — everything active/positive */
  --green2: #00c853;   /* green hover */
  --glow: rgba(0,230,118,.12);
  --red: #ff4560;      /* losses, danger */
  --gold: #ffd740;     /* BE trades, warnings */
  --blue: #448aff;     /* info, links */
  --purple: #b388ff;   /* MNQ instrument */
  --t1: #fff;          /* primary text */
  --t2: #ccc;          /* secondary text */
  --t3: #888;          /* muted text */
  --t4: #555;          /* very muted */
}
```
Fonts: `'Inter', sans-serif` (body) + `'DM Mono', monospace` (labels/values)

## Trader Profile (Context for Features)
- $50K ProFirm Challenge | Target: $3,000 (6%) | Max DD: $2,000 (4%)
- Methodology: ICT — FVG/IFVG, liquidity sweeps (SSL/BSL), swing failure
- Instruments: MNQ, MES, MGC
- Sessions: Asia (8PM-12AM NY), London (2-5AM NY), NY (8-11AM NY), London Close (10AM-12PM NY)
- Known weakness: overtrading/revenge trading after losses

## Risk Model
- Trade 1: $200 risk → $400 reward (1:2 RR)
- T1 WIN → Trade 2: $400 risk → $800 reward
- Any LOSS → session locked for the day
- BE (breakeven) → free re-entry, doesn't count toward 2-trade limit
- Max 2 completed (WIN/LOSS) trades per day
- Best day: +$1,200 | Worst day: -$200

## Trade Limit Logic (Option B — already implemented)
```javascript
// Only WIN/LOSS count toward 2-trade limit. BEs are free.
function getTradeLimitState() {
  const todayTrades = trades.filter(t => t.date === today);
  const completed = todayTrades.filter(t => t.result === 'WIN' || t.result === 'LOSS').length;
  const canTrade = completed < 2;
  // ...
}
```

## App Pages (5 total)
1. **Dashboard** — Sessions, Win Rate grid, Future You AI mentor, P&L stats, equity curve, recent trades
2. **Trade Journal** — Full ICT trade log, date filters, stats bar, calendar view, export
3. **Analytics** — Charts by session/model/instrument, scenario projections
4. **Calculator** — Position sizer, daily loss meter, pre-trade checklist
5. **Trading Plan** — Entry process, rules, sessions, risk model

## Key Functions (don't break these)
- `renderAll()` — renders dashboard + journal + calculator
- `renderD()` — dashboard render (includes mentor AI call)
- `renderJ()` — journal render with filters
- `renderA()` — analytics charts
- `saveTrade()` — saves trade + syncs to Supabase
- `getTradeLimitState()` — returns trade limit state
- `getTrailingFloor()` — calculates trailing DD floor
- `sbPullTrades()` — pulls trades from cloud
- `sbPushSettings()` — syncs settings + EOD marks to cloud
- `nav(id)` — navigates between pages (has safety check for invalid ids)

## Bugs Already Fixed (don't revert)
1. BE result forces pnl=0 and rr=0 in saveTrade()
2. nav() has safety check: `if (!_pgEl) return;`
3. EOD marks sync inside user_settings to cloud
4. Checklist state syncs to cloud per user
5. EOD lock triggers immediate sbPushSettings()
6. Trade limit is WARNING ONLY — checkTradeLimit() always returns true (no hard block)

## Risk Model (current — updated 2026-04-07)
- 3 trades per day guideline (warning shown on dashboard, not enforced as hard block)
- $100 flat risk per trade
- Minimum 1:2 RR, no maximum cap
- After any loss → session should stop (warning only, not enforced)
- Best day: +$600 | Worst day: -$100
- DEFAULT_SETTINGS: `{ target: 3000, account: 50000, maxday: 100, maxdd: 2000, risk1: 100, risk2: 100 }`

## localStorage Keys
- `lt_v4` — trades array (local cache)
- `lt_cfg` — challenge settings
- `lt_eod` — EOD balance marks
- `lt_ck3` — checklist state
- `lt_navmode` — sidebar/header/collapsed preference
- `lt_custom_fields` — custom journal field definitions

## AI Mentor Feature
- Powered by Claude API (claude-sonnet-4-20250514)
- Called in renderMentor() on dashboard load
- Builds context from real trade data and sends to API
- Has fallback messages if API fails
- Cache key prevents unnecessary re-calls

## Nav Modes
Three layouts: sidebar (default), collapsed, header
Stored in localStorage as `lt_navmode`
Mobile: bottom nav bar + slide-in sidebar drawer

## Important Rules When Editing
1. **Never change the color theme** — user wants original green #00e676
2. **Always run syntax check** before saving: `node --check` on the JS
3. **Test all 5 nav pages** still work after any change
4. **Keep it a single file** — no splitting into separate CSS/JS files
5. **Don't add build steps** — must stay deployable by drag-and-drop to GitHub
6. **Supabase RLS** — never bypass row-level security
7. **BE trades** must always have pnl=0 and rr=0

## GitHub Pages Issue (known)
Pages sometimes resets to "None" branch. Fix:
1. Go to github.com/arjunkkharge/the-edge/settings/pages
2. Branch → main → / (root) → Save
