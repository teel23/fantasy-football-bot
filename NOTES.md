# Fantasy Football AI Bot — Project Notes
> Reference for jumping back into this project.
> Last updated: March 12, 2026 (Session 20 complete)

## Session 20 — March 12, 2026

**Track A — Email Infrastructure ✅**
- `send_injury_alerts_to_all_users()` already existed — verified wired in daily_alerts.yml
- Added `POST /email-alerts/send-lineup-reminders` to `email_alerts.py`
- Created `SETUP_EMAILS.md` with Railway + GitHub secrets guide

**Track B — Roster Page Overhaul ✅**
- Backend: `GET /lineup/roster/{league_id}` returns dynasty_value, injury_status, age, boom_bust_tier
- Frontend: 4 slot sections (Starters/Bench/Taxi/IR), search bar, sort toggle (DV/Age), boom/bust + injury badges

**Track C — Waiver Wire Drop Pairing ✅**
- Backend: `WaiverEngine._get_all_drop_candidates()` — 3 weakest bench players per position, reasons via single Claude batch call
- Frontend: "Consider dropping:" section under each waiver pickup card

**Track D — Home Dashboard New Cards ✅**
- Backend: `_build_offseason_calendar()` (5 phases), `get_sell_buy_targets()` (sell-high/buy-low via Claude)
- Frontend: `OffseasonCalendarCard` + `SellBuyCard` in offseason layout

**Track E — Dynasty Intelligence ✅**
- E1: `engine/boom_bust.py` — 6-tier boom/bust + `POST /players/boom-bust` endpoint
- E2: `GET /draft/grades/{league_id}` — actual production grades; draft history section in draft/page.tsx
- E3: `engine/rankings.py` — `get_league_settings_weights()`; applied to WaiverEngine opportunity scoring

---

## What It Is
An AI-powered fantasy football assistant. Custom weighting engine, self-learning capabilities, free data stack. Analyzes matchups, suggests waiver pickups, improves week over week. Currently in active development.

## GitHub
📁 **https://github.com/teel23/fantasy-football-bot**
- Backend: `teel23/ff-bot` (private) → Railway
- Frontend: `teel23/ff-bot-web` (private) → Vercel

## Local Path
`/COMPUTER/AI/Projects/fantasy-football-bot/`

---

## Tech Stack
Python (FastAPI) · Next.js 16 · Supabase · Claude API (claude-sonnet-4-6)

## Folder Structure
```
fantasy-football-bot/
├── ff-bot/         ← FastAPI backend (teel23/ff-bot) → Railway
├── ff-bot-web/     ← Next.js frontend (teel23/ff-bot-web) → Vercel
├── NOTES.md
├── MASTER_TODO.md
├── AUDIT_REPORT_S17.md
└── CLAUDE_CODE_PROMPT_S19.md
```

---

## Status
🟢 Core app working — ~72% complete (Sessions 17–19 complete Mar 11, 2026)

## Portfolio
Shown in the **Coming Soon** section of the portfolio. Links to GitHub repo.

---

## Infrastructure

| Service | Project | URL |
|---|---|---|
| Railway (backend) | web-production-dddf8 | `https://web-production-dddf8.up.railway.app` |
| Vercel (frontend) | teel23/ff-bot-web | auto-deploy from main |
| Supabase | keksjsxvcymsjhzxubxw | `https://keksjsxvcymsjhzxubxw.supabase.co` |

**CORS:** All `*.vercel.app` origins allowed via regex in `api/main.py`.
**API URL:** `next.config.ts` hardcodes Railway URL as default — Vercel does NOT need `NEXT_PUBLIC_API_URL` set in dashboard (removed).

---

## Connected League

| Field | Value |
|---|---|
| Name | Chimp Dynasty |
| Sleeper League ID | `1328784373125771264` |
| League Type | Dynasty, PPR, FAAB $1000 |
| Teams | 12 |
| Roster | 33 players (10 starters, 16 bench, 3 taxi, 4 IR) |
| Playoff Start | Week 15, 6 teams |

---

## Data State (as of Mar 11, 2026)

| Table | Rows | Notes |
|---|---|---|
| `players` | 5,303 | 4,329 have age populated (Sleeper bio sync) |
| `weekly_stats` | 16,881 | Seasons 2022–2024 only — 2025 not yet on nflverse |
| `weekly_projections` | 30,651 | 2025 season, weeks 1–17 |
| `opportunity_metrics` | 32,454 | 2025 season |
| `snap_counts` | 106,148 | 2022–2025 (pfr ID format — doesn't join to players table) |
| `role_change_flags` | 8,154 | 2025 season (backfilled Mar 11) |
| `injuries` | 1,000 | Populated |
| `league_rosters` | 33 | Chimp Dynasty, slot_type set correctly |
| `schedules` | 1,139 | 2022–2025 |

**Known data gap:** `weekly_stats` has no 2025 rows. nflverse `player_stats_2025.parquet` returns 404 — not yet published. Re-run `python -m ingestion.historical` when it becomes available.

**Known schema note:** `snap_counts` uses pfr-format player IDs (`GholWi00` style). Cannot join to `players` table (GSIS format). `opportunity_metrics` uses GSIS IDs and is the source of truth for opportunity data.

---

## Recent Changes

| Date | Session | What Changed |
|---|---|---|
| Mar 11, 2026 | Session 19 | Full audit + rebuild — all core features working |
| Mar 11, 2026 | Session 18 | Schema migrations, email fix, backfill scripts |
| Mar 11, 2026 | Session 17 | War Room, off-season home, auth fix, dynasty capital |

---

## Session 19 — March 11, 2026 (Full Audit + Rebuild)

### Root Causes Fixed
- **`NEXT_PUBLIC_API_URL=http://localhost:8000`** in Vercel — was making every API call fail in production. Fixed by: removing from Vercel dashboard + hardcoding Railway URL in `next.config.ts`.
- **CORS blocking Vercel** — added `allow_origin_regex=r"https://.*\.vercel\.app"` to FastAPI middleware.
- **All roster players showing as `bench`** — upsert was hitting column DEFAULT. Rewrote resync to hard-delete then INSERT. `scripts/fix_roster_slots.py`.
- **0 players with age** — `_ingest_rosters` never mapped bio fields. Built `scripts/populate_player_bios.py` — fetches from Sleeper API, updates 4,429 players.
- **`user_leagues` missing settings columns** — `league_type`, `waiver_budget`, etc. never added. Applied via management API, then synced Chimp Dynasty settings from Sleeper.
- **War Room JSON parse error** — Claude returned report text with quotes inside JSON strings. Fixed with pre-filled assistant turn (`{"`) to force JSON mode + fallback regex parser.
- **Waiver engine returning 0 results** — `weekly_projections.blended_projection` is NULL for most players. Added fallback to `opportunity_metrics` when projection data is sparse.
- **Role change flags nearly empty** — threshold was 15pp but data is stored as 0–1 decimal (max trend ~0.05). Corrected thresholds, backfilled 8,154 flags.

### Scripts Added
- `scripts/fix_roster_slots.py` — re-syncs all leagues from Sleeper with correct slot_type
- `scripts/populate_player_bios.py` — populates age/years_exp/college/draft data from Sleeper
- `scripts/backfill_opportunity.py` — builds opportunity_metrics from snap_counts + weekly_stats
- `scripts/backfill_role_flags.py` — builds role_change_flags from opportunity_metrics trends

### Verified Working (live Railway endpoints)
- ✅ Home dashboard — offseason mode, 33 players, 5 AI priorities, 3 trade targets
- ✅ Roster timeline (War Room Gantt) — 33 players with ages, chart 2025–2033
- ✅ Dynasty Capital — 4 owned / 4 owed / net -250 / AI analysis
- ✅ Power Rankings — 12 teams ranked by composite score
- ✅ Lineup Optimizer — 33 recommendations, starters identified
- ✅ Waiver Wire — Chris Godwin top pickup with Claude reasoning
- ✅ Trade Analyzer — DECLINE verdict with specific player reasoning
- ✅ Notifications — 457 alerts
- ✅ War Room Scout Report — fix deployed (pre-fill JSON mode)

---

## Session 18 — March 11, 2026

- Added `slot_type` to `league_rosters`, player bio columns, `player_profiles` cache table (Supabase migration `004_session18_schema.sql`)
- Fixed `engine/email.py` — production URL (`FRONTEND_URL` env var), `send_injury_alerts_to_all_users()` function
- Power rankings off-season fallback to prior season `weekly_stats`
- Backfill scripts for opportunity_metrics and role_change_flags

---

## Open Items
- [ ] Add VERCEL_TOKEN to Vercel env vars (vercel.com/account/tokens) → unlocks Deployments page
- [ ] Define data sources (free tier: ESPN API, Sleeper API, etc.)
- [ ] Build weighting engine v1
- [ ] Sync leave isn’t working or transferring to home page
- [ ] Connect ai assistant to live data in website
- [ ] What is trade targets? Maybe team needs instead?
- [ ] Roster page to show starting line up, bench, taxi and IR if applicable
- [ ] Taxi squad recommender 
- [ ] Lineup optimizer: no weekly data found, need a full overhaul, want start sit recommendations based on players I select 
- [ ] Waiver wire is not working, I love this so far add in a feature for teams needs maybe just an indicator
- [ ] Trade evaluator is not working, no player value built, need to add this based on league settings
- [ ] War room is getting an error when generating
- [ ] League what to be able to see other teams rosters apply this in the trade page too
- [ ] Settings add a time for time synced 
- [ ] Add option to delete sleeper teams
- [ ] On top in menu have league and team name show rather than username it will be the same for all
- [ ] Only show the draft page that applies to the league dynasty = rookie, redraft = draft, keepers = keepers draft have these change based on what league is selected
- [ ] Keepers analysis is not working and I do mine different not hy losing a draft pick, we get to keep 3 players no more than 1 from each position
- [ ] NEED 2025 data and a reliable source for near instant data for 2026 season in a bad way 
- [ ] Need previous draft player profile analysis, all players drafted, get their average draft position and tell me what kinds of players are likely to boom(find the sleeper pick) and what ones bust(find the picks that don’t pan out or are not valuable) 
- [ ] Schedule tab 
- [ ] I am not sure what data we can pull from sleeper but maybe we pull all players drafted stats and live games form there as well maybe as a backup or main if it works 
- [ ] **Re-run `python -m ingestion.historical`** when nflverse publishes 2025 weekly_stats parquet (check `player_stats_2025.parquet` — currently 404)
- [ ] **Add `RESEND_API_KEY` + `ALERT_FROM_EMAIL` to Railway env vars** — email alerts not functional until set
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL`) — needed for GitHub Actions email workflows
- [ ] Roster page — show slot groupings (Starters / Bench / Taxi / IR) instead of position groups
- [ ] Roster page — show dynasty value + injury badge inline per player
- [ ] Drop suggestions paired with waiver pickups ("Pick up X → Drop Y")
- [ ] Dynasty offseason calendar card on home dashboard
- [x] Connect to Supabase for persistence
- [x] Deploy web interface via Vercel
