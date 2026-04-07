# AGENTS.md — Shared AI Collaboration File
# The Edge | ICT Prop Trading Journal
# For use by: Claude (Anthropic) + Antigravity (Google)

---

## What This Project Is
A personal ICT prop trading journal and dashboard built for a $50K prop firm challenge.
Single HTML file app. No build step. No framework. No npm.

**Owner:** arjunkharge@gmail.com
**Live URL:** https://arjunkkharge.github.io/the-edge
**GitHub:** https://github.com/arjunkkharge/the-edge
**Deploy:** Drag `index.html` to https://github.com/arjunkkharge/the-edge/upload/main OR `git push origin main`

---

## Tech Stack
- **Single file:** `index.html` — all HTML, CSS, JS in one file. Never split.
- **Hosting:** GitHub Pages (auto-deploys ~60s after push to main)
- **Database:** Supabase (PostgreSQL, per-user RLS)
- **Auth:** Supabase email/password
- **Charts:** Chart.js (CDN)
- **Fonts:** Inter + DM Mono (Google Fonts CDN)
- **Export:** XLSX, PDF, CSV, JSON (all CDN)

## Supabase Config
- URL: https://aceffksoagyfnpwuccgm.supabase.co
- Anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFjZWZma3NvYWd5Zm5wd3VjY2dtIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzMyNDQ3NDQsImV4cCI6MjA4ODgyMDc0NH0._sSwzzOKhAt8xUxFVNXDIbspmvpTGcE929ZZTMm-yM8
- Tables: `trades` (per-user RLS), `user_settings`
- Never bypass RLS. Never expose service key.

---

## Current Risk Model (as of 2026-04-05)
- **3 trades max per day**
- **$100 flat risk per trade** (no scaling, all 3 same size)
- **Minimum 1:2 RR** ($200 reward) — no maximum cap (1:10, 1:20+ allowed)
- **After any LOSS → session over, no more trades**
- **BE (breakeven) trades** → free re-entry, do NOT count toward 3-trade limit
- **Best day:** +$600 (3 wins × $200)
- **Worst day:** -$100 (Trade 1 loss → stop)

### Daily P&L Scenarios
| Outcome | P&L |
|---|---|
| Loss on T1 | -$100 → stop |
| Win T1, Loss T2 | +$100 net |
| Win T1+T2, Loss T3 | +$300 net |
| Win all 3 | +$600 |

---

## App Pages (5 total)
| ID | Page | Description |
|---|---|---|
| `pg-dash` | Dashboard | P&L stats, equity curve, session grid, AI mentor, today's trades, alerts |
| `pg-journal` | Trade Journal | Full log, filters, calendar view, export (XLSX/PDF/CSV/JSON) |
| `pg-analytics` | Analytics | Charts by session/model/instrument, scenario projections |
| `pg-calc` | Calculator | Position sizer (3 flat $100 options), daily loss meter, pre-trade checklist |
| `pg-plan` | Trading Plan | Entry rules, sessions, risk model reference, after-loss protocol |

---

## Key JavaScript Functions (do not break)
| Function | Purpose |
|---|---|
| `renderAll()` | Renders dashboard + journal + calculator |
| `renderD()` | Dashboard render (includes AI mentor call) |
| `renderJ()` | Journal render with filters |
| `renderA()` | Analytics charts and projections |
| `saveTrade()` | Saves trade + syncs to Supabase |
| `getTradeLimitState()` | Returns trade limit state (3-trade limit, BE-aware) |
| `checkTradeLimit()` | Called on openModal() — blocks new trade if limit reached |
| `getTrailingFloor()` | Calculates trailing drawdown floor |
| `sbPullTrades()` | Pulls trades from Supabase cloud |
| `sbPushSettings()` | Syncs settings + EOD marks to cloud |
| `nav(id)` | Navigates between pages (has safety check for invalid ids) |
| `calcUpdate()` | Calculator position sizer — 3 flat $100 options |

---

## localStorage Keys
| Key | Content |
|---|---|
| `lt_v4` | trades array (local cache) |
| `lt_cfg` | challenge settings |
| `lt_eod` | EOD balance marks |
| `lt_ck3` | checklist state |
| `lt_navmode` | sidebar/header/collapsed preference |
| `lt_custom_fields` | custom journal field definitions |

## DEFAULT_SETTINGS (current)
```javascript
{ target: 3000, account: 50000, maxday: 100, maxdd: 2000, risk1: 100, risk2: 100 }
```

---

## Design Rules — MUST FOLLOW
1. **Never change the color theme** — locked as below
2. **Keep it a single file** — no splitting into separate CSS/JS files
3. **No build steps** — must stay drag-and-drop deployable to GitHub
4. **Never bypass Supabase RLS**
5. **BE trades** must always have pnl=0 and rr=0
6. **Run syntax check** before committing: extract JS and validate
7. **Test all 5 nav pages** still work after any change

## Color Theme (LOCKED — do not change)
```css
:root {
  --bg: #080808;        /* main background */
  --bg2: #0f0f0f;       /* cards, sidebar */
  --bg3: #141414;       /* inputs, secondary */
  --b1: #1e1e1e;        /* borders */
  --b2: #2a2a2a;        /* hover borders */
  --b3: #363636;        /* active borders */
  --green: #00e676;     /* PRIMARY ACCENT — everything active/positive */
  --green2: #00c853;    /* green hover */
  --glow: rgba(0,230,118,.12);
  --red: #ff4560;       /* losses, danger */
  --gold: #ffd740;      /* BE trades, warnings */
  --blue: #448aff;      /* info, long trades */
  --purple: #b388ff;    /* MNQ instrument */
  --t1: #fff;           /* primary text */
  --t2: #ccc;           /* secondary text */
  --t3: #888;           /* muted text */
  --t4: #555;           /* very muted */
}
```
Fonts: `'Inter', sans-serif` (body) + `'DM Mono', monospace` (labels/values)

---

## Trader Profile
- **Challenge:** $50K PropFirm | Target: $3,000 (6%) | Max DD: $2,000 (4%)
- **Methodology:** ICT — FVG/IFVG, liquidity sweeps (SSL/BSL), swing failure
- **Instruments:** MNQ, MES, MGC
- **Sessions:** Asia (8PM-12AM NY), London (2-5AM NY), NY (8-11AM NY), London Close (10AM-12PM NY)
- **Known weakness:** overtrading/revenge trading after losses

---

## Bugs Already Fixed (do not revert)
1. BE result forces pnl=0 and rr=0 in saveTrade()
2. nav() has safety check: `if (!_pgEl) return;`
3. EOD marks sync inside user_settings to cloud
4. Checklist state syncs to cloud per user
5. EOD lock triggers immediate sbPushSettings()
6. Trade limit enforced on openModal() for new trades

---

## Change Log
| Date | Change | Agent |
|---|---|---|
| 2026-04-05 | Risk model updated: $200/$400 scaling → flat $100/trade, 3 trades/day | Claude |
| 2026-04-05 | Analytics EV formula updated for 3-trade sequential model | Claude |
| 2026-04-05 | All UI references (plan page, calculator, checklist, alerts) updated | Claude |
| 2026-04-07 | Removed hard trade limit block — checkTradeLimit() is warning only, no modal lock | Claude |

---

## How to Collaborate
- **Before making changes:** read this file + CLAUDE.md to understand current state
- **After making changes:** update the Change Log table above with date, change, and your agent name
- **Never override each other's work** without noting it in the log
- **Sync point:** this file is the source of truth for current project state
