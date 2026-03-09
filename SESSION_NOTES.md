# FF Bot — Session Notes
**Last updated:** March 6, 2026 (Session 15)
**Project folder:** `~/Desktop/AI/FF Bot/ff-bot`
**GitHub repo:** https://github.com/teel23/ff-bot (private)

---

## ✅ What's Done

| Item | Status |
|---|---|
| Supabase schema deployed (19 tables, RLS, triggers) | ✅ Done |
| `.env` file created with all API keys | ✅ Done |
| FastAPI server boots cleanly | ✅ Done |
| `scripts/weekly_learning.py` created | ✅ Done |
| `ingestion/historical.py` fully rewritten + roster bug fixed | ✅ Done |
| Players table populated (4,596 rows, 2022–2025 rosters) | ✅ Done |
| Weekly stats populated (16,881 rows, 2022–2024; 2025 not yet published) | ✅ Done |
| Schedules populated (1,139 rows, 2022–2025) | ✅ Done |
| Snap counts populated (106,148 rows, 2022–2025) | ✅ Done |
| NGS populated (5,776 rows) | ✅ Done |
| snap_counts FK dropped in Supabase | ✅ Done |
| Code pushed to GitHub (private) | ✅ Done |
| GitHub Actions secrets set (5 secrets) | ✅ Done |
| Owner profile set in Supabase (`01teel@gmail.com`, role=owner) | ✅ Done |
| handle_new_user trigger fixed (exception handler added) | ✅ Done |
| Sleeper user matching implemented (username lookup + team picker flow) | ✅ Done |
| Migration 001 deployed (`sleeper_username` on profiles, `sleeper_roster_id` on user_leagues) | ✅ Done |
| injuries.py rewritten (single ESPN endpoint, 793 rows live) | ✅ Done |
| projections.py fixed (removed 403 FFDataPros, off-season guard added) | ✅ Done |
| vegas.py verified working (no lines off-season — expected) | ✅ Done |
| weather.py verified working (no schedule off-season — expected) | ✅ Done |
| Next.js frontend scaffolded (next@16.1.6, Tailwind, Supabase SSR) | ✅ Done |
| Full auth flow working (magic link → /auth/callback → dashboard) | ✅ Done |
| Auth backend rewritten — Supabase /auth/v1/user API (dropped PyJWT) | ✅ Done |
| All dashboard pages built: Home, Lineup, Waiver, Trade, Leagues | ✅ Done |
| League connect flow working end-to-end ("Chimp Dynasty", roster_id=9) | ✅ Done |
| LineupEngine.recommend() + WaiverEngine.analyze() user_id param fixed | ✅ Done |
| weekly_projections seeded (30,651 rows — 1,803 players × 17 weeks, 2025) | ✅ Done |
| opportunity_metrics seeded (32,454 rows — 1,803 players × 18 weeks, 2025) | ✅ Done |
| role_change_flags seeded (10 free-agent WR/RB/TE, week 1 2025) | ✅ Done |
| Claude JSON parse fixed (strip markdown fences before json.loads) | ✅ Done |
| Claude MAX_TOKENS fixed (lineup 4096, waiver 2048, trade 512) | ✅ Done |
| Lineup Claude payload limited to starters + top 6 bench (was sending all 33) | ✅ Done |
| LineupEngine verified: real recommendations + Claude reasoning working | ✅ Done |
| WaiverEngine verified: role change alerts, position filtering, Claude reasoning working | ✅ Done |
| TradeEngine verified: ACCEPT/DECLINE/NEUTRAL verdicts with Claude reasoning | ✅ Done |
| Frontend build: clean compile, no TypeScript errors, all 12 pages OK | ✅ Done |
| Full product spec expanded + refined (Session 3 planning) | ✅ Done |
| **Session 4: DB migration 002 for league settings columns** | ✅ Done |
| **Session 4: Auto-import league settings from Sleeper on connect** | ✅ Done |
| **Session 4: `engine/chat.py` — context-aware Claude chat (4 modes)** | ✅ Done |
| **Session 4: `api/routes/chat.py` — POST /chat/message endpoint** | ✅ Done |
| **Session 4: `api/routes/home.py` — GET /home/overview card data + Claude briefing** | ✅ Done |
| **Session 4: LeagueContext (league-context.tsx) — shared active league state** | ✅ Done |
| **Session 4: ChatPanel component — floating AI chat, context-aware per page** | ✅ Done |
| **Session 4: Dashboard layout overhauled — new nav + league switcher + ChatPanel** | ✅ Done |
| **Session 4: Home page rebuilt — card grid (matchup, injuries, depth, waivers, news)** | ✅ Done |
| **Session 4: Roster page — slot-ordered rows, slide-out drawer, grades, trend bars** | ✅ Done |
| **Session 4: War Room page — scout report, A-F grades, action items** | ✅ Done |
| **Session 4: League page — live Sleeper standings, league info strip, sync button** | ✅ Done |
| **Session 4: TypeScript build clean (0 errors)** | ✅ Done |
| **Session 4: Backend imports clean (python3 -c "from api.main import app")** | ✅ Done |
| **Session 5: `api/routes/players.py` — GET /players/search (name autocomplete)** | ✅ Done |
| **Session 5: `leagues/page.tsx` — uses LeagueContext, shows auto-imported settings, no manual scoring picker** | ✅ Done |
| **Session 5: `lineup/page.tsx` — uses LeagueContext, no self-fetch** | ✅ Done |
| **Session 5: `waiver/page.tsx` — uses LeagueContext, shows FAAB budget in header** | ✅ Done |
| **Session 5: `trade/page.tsx` — uses LeagueContext + player name search w/ autocomplete dropdown** | ✅ Done |
| **Session 5: TypeScript build clean (0 errors) — all pages verified** | ✅ Done |
| **Session 5: Backend imports clean — all 9 routes registered** | ✅ Done |
| **Session 6: Lineup engine — dynamic roster slots from `user_leagues.roster_positions`** | ✅ Done |
| **Session 6: Waiver engine — FAAB budget + waiver type passed into Claude prompt** | ✅ Done |
| **Session 6: Trade engine — `league_type` (dynasty/redraft/keeper) passed into Claude prompt** | ✅ Done |
| **Session 6: Off-season empty states — lineup + waiver return graceful responses when no data** | ✅ Done |
| **Session 6: `GET /players/{player_id}` — full player profile endpoint (bio, injury, projection, stats, opp metrics)** | ✅ Done |
| **Session 6: `PlayerProfileDrawer` component — full stats slide-out, reusable from any page** | ✅ Done |
| **Session 6: Roster page — "View Full Profile" button opens PlayerProfileDrawer** | ✅ Done |
| **Session 6: `claude.py` — added `from __future__ import annotations`; trade token budget raised to 768** | ✅ Done |
| **Session 6: TypeScript build clean (0 errors) + backend imports OK** | ✅ Done |
| **Session 7: PlayerProfileDrawer wired into Waiver + Trade pages** | ✅ Done |
| **Session 7: `GET /leagues/power-rankings` — roster strength + recent form algorithm** | ✅ Done |
| **Session 7: League page — real power rankings table (score, strength, form, trend)** | ✅ Done |
| **Session 7: TypeScript build clean (0 errors) + backend imports OK** | ✅ Done |
| **Session 8: Scoring engine JSONB — rec-value adjustment using target_share (PPR→actual)** | ✅ Done |
| **Session 8: `GET /leagues/waiver-priority` — Sleeper waiver_position order** | ✅ Done |
| **Session 8: `GET /players/roster-timeline` — age + peak window data for Gantt** | ✅ Done |
| **Session 8: `GET /notifications/` — injury + role change alerts** | ✅ Done |
| **Session 8: War Room Gantt chart — CSS horizontal bars, color-coded by peak status** | ✅ Done |
| **Session 8: Waiver page — collapsible waiver priority order panel** | ✅ Done |
| **Session 8: Dashboard layout — notification bell with badge count + dropdown** | ✅ Done |
| **Session 8: TypeScript build clean (0 errors) + backend imports OK** | ✅ Done |
| **Session 9: Home page — opponent name in matchup card; injury players clickable → PlayerProfileDrawer** | ✅ Done |
| **Session 9: `home.py` — `player_id` added to injury_alerts; opponent resolved via `fetch_sleeper_users`** | ✅ Done |
| **Session 9: War Room Playoff Push tab — live standings from Sleeper, rank/magic number/playoff %** | ✅ Done |
| **Session 9: War Room Rebuild vs Contend tab — grade-based CONTEND/SELL-HIGH/REBUILD verdict with rationale** | ✅ Done |
| **Session 9: TypeScript build clean (0 errors) + backend imports OK** | ✅ Done |
| **Session 10: `weekly_learning.py` — `detect_current_week()` uses Sleeper API; fallback to date calc** | ✅ Done |
| **Session 10: `weekly_learning.py` — removed dead code bug (`valid_maes` block)** | ✅ Done |
| **Session 10: `layout.tsx` — notifications fetch uses `nflWeek`/`nflSeason` from LeagueContext** | ✅ Done |
| **Session 10: `roster/page.tsx` — fast raw DB load; position groups; click → PlayerProfileDrawer; Lineup Optimizer button** | ✅ Done |
| **Session 10: `lineup/page.tsx` — subtitle clarifying AI start/sit optimizer role** | ✅ Done |
| **Session 10: `lib/api.ts` — `RosterPlayer` type + `getRoster()` added** | ✅ Done |
| **Session 10: `api/routes/espn.py` — ESPN Phase 2 stub + OAuth design notes (SWID+espn_s2, checklist)** | ✅ Done |
| **Session 10: TypeScript build clean (0 errors) + backend imports OK (13 routes)** | ✅ Done |
| **Session 11: Trade page Mode B — League Scan tab; GET /trade/league-scan — fetches all 12 rosters from Sleeper, Claude ranks top 3 partners with targets** | ✅ Done |
| **Session 11: Rookie Draft page — /dashboard/rookie-draft; GET /players/rookies (years_exp=0); dynasty tier algorithm (position+draft slot); position filter tabs** | ✅ Done |
| **Session 11: Draft page — /dashboard/draft; GET /draft/upcoming + GET /draft/picks/{id}; live board slide-out; Phase 2 AI notice** | ✅ Done |
| **Session 11: Boom/bust tier — computed server-side (Elite/High Floor/High Ceiling/Boom-or-Bust/Fade/Value); added to GET /players/{id} response** | ✅ Done |
| **Session 11: PlayerProfileDrawer — boom/bust tier badge added (color-coded pill)** | ✅ Done |
| **Session 11: Nav — Rookie Draft + Draft added** | ✅ Done |
| **Session 11: TypeScript build clean (0 errors, 17 pages) + backend 36 routes** | ✅ Done |
| **Session 12: Trade page Mode B — "Start Trade →" on PartnerCard pre-fills Mode A with hint banner (team name, target players, offer)** | ✅ Done |
| **Session 12: Mobile polish — sidebar fixed→slide-in on mobile, hamburger `☰` in top bar, backdrop overlay, auto-close on route change, responsive padding** | ✅ Done |
| **Session 12: Keeper Optimizer — POST /keepers/analyze; dynasty = value-based (KEEP/TRADE/CUT); keeper leagues = round cost vs value; new `/dashboard/keepers` page** | ✅ Done |
| **Session 12: Nav — Keepers ⚑ added** | ✅ Done |
| **Session 12: `lib/api.ts` — analyzeKeepers(), KeeperRecommendation, KeeperCostInput types; RosterPlayer.age optional** | ✅ Done |
| **Session 12: TypeScript build clean (0 errors, 18 pages) + backend imports OK** | ✅ Done |
| **Session 13: All open architecture decisions resolved (email/draft-sync/ADP/ESPN)** | ✅ Done |
| **Session 13: `engine/email.py` — Resend HTML injury alert emails** | ✅ Done |
| **Session 13: `api/routes/email_alerts.py` — GET /test + POST /send-injury-alerts (dedup via email_alert_log)** | ✅ Done |
| **Session 13: `db/migrations/003_email_alerts.sql` — email_alert_log table (RLS, daily dedup)** | ✅ Done |
| **Session 13: `engine/draft_grade.py` — A-F pick grading vs ADP, weighted GPA, steals/reaches summary** | ✅ Done |
| **Session 13: `GET /draft/adp` — FantasyPros scrape (primary) + Sleeper player ranks (fallback), in-memory cache** | ✅ Done |
| **Session 13: `GET /draft/grade/{draft_id}` — grade any complete draft; filter by roster_id** | ✅ Done |
| **Session 13: `engine/draft_recommend.py` — Claude live pick AI (BPA + positional need + roster slots)** | ✅ Done |
| **Session 13: `POST /draft/recommend` — live AI pick recommendation; auto-fetches picks from Sleeper if draft_id given** | ✅ Done |
| **Session 13: Draft page — 5s polling when live, AI Pick Assistant panel, my-picks highlighted, DraftGradePanel slide-out, Grade Draft button** | ✅ Done |
| **Session 13: beautifulsoup4 + lxml added to requirements.txt** | ✅ Done |
| **Session 13: TypeScript build clean (0 errors, 18 pages) + backend imports OK (15 routes)** | ✅ Done |
| **Session 13: Production deploy — Railway (backend) + Vercel (frontend), both live** | ✅ Done |
| **Session 13: Auth fix — replaced server-side route.ts with client-side callback page.tsx** | ✅ Done |
| **Session 13: Auth fix — onAuthStateChange handles magic link hash + PKCE cross-device** | ✅ Done |
| **Session 13: Auth fix — Supabase redirect URLs updated to include Vercel domain** | ✅ Done |
| **Session 13: Production fix — all localhost:8000 fallbacks replaced with Railway URL (5 files)** | ✅ Done |
| **Session 13: Supabase + Resend keys hardcoded as fallbacks in supabase.ts + api.ts** | ✅ Done |
| **Session 13: email_alert_log migration 003 deployed to Supabase via Management API** | ✅ Done |
| **Session 13: Email test confirmed working — injury alert delivered to 01teel@gmail.com** | ✅ Done |

---

## 🔄 Next Session — Resume Here

### Current State (Session 15 — first real end-to-end test, March 6, 2026)
- **Auth**: Working on desktop. Mobile works when fresh link requested per device.
- **1 league connected**: Chimp Dynasty — was stored as `redraft` (bug fixed — re-sync required)
- **Backend**: LIVE on Railway v1.7.0 — dynasty fix deploying now
- **Frontend**: LIVE on Vercel — nav filter fix deploying now
- **Most pages blank/minimal** — primarily off-season issue + roster sync needed

### Production URLs
| Service | URL |
|---|---|
| Frontend | https://ff-bot-web.vercel.app |
| Backend API | https://web-production-dddf8.up.railway.app |
| API Docs | https://web-production-dddf8.up.railway.app/docs |

### CRITICAL ACTION AFTER DEPLOY — Re-sync the league
Go to **Settings → Connected Leagues → ↻ Sync** on Chimp Dynasty.
This reruns `_parse_league_settings` with the fixed dynasty detection and updates the DB row (`redraft` → `dynasty`, also refreshes roster). All pages that branch on `league_type` will then work correctly.

### What was done (Session 15)
- **Auth desktop login working**: hardcoded Supabase URL + anon key in `lib/supabase.ts` — eliminates empty Vercel env var problem
- **auth/callback redesigned**: clean "Sign-in link expired" screen + "Request a new link" button replaces confusing raw debug dump
- **Dynasty detection FIXED** (`leagues.py`): was reading `league_info.league_type` (null) → now reads `settings.type` (2=dynasty) with fallback on `pick_trading + taxi_slots`
- **Nav filtering FIXED** (`layout.tsx`): Keepers and Rookie Draft only appear for dynasty/keeper leagues
- **Railway SUPABASE_ANON_KEY**: confirmed correct (208 chars — earlier apparent truncation was CLI error output)

### Known Issues — Logged from First Real Use (Session 15)
| # | Issue | Root Cause | Fix |
|---|---|---|---|
| 1 | Dynasty shows as redraft | `_parse_league_settings` read wrong Sleeper field | ✅ Fixed — re-sync needed |
| 2 | Keepers + Rookie Draft nav visible for all leagues | Nav not filtered by league_type | ✅ Fixed |
| 3 | All pages blank (Home, Roster, Lineup, Waiver, Trade, War Room) | Off-season: no live matchup/projections/vegas. Seeded data (2025 wk1) doesn't match current state. Also need roster sync. | P0 next session |
| 4 | Roster data not flowing to other pages | Pages independently fetch from DB; `league_rosters` may be empty until sync runs | Re-sync after deploy |
| 5 | Home page empty | `/home/overview` needs live Sleeper matchup — off-season there is none | Build off-season home: dynasty capital, roster health, upcoming draft |
| 6 | Lineup optimizer blank | Off-season guard returns `{off_season: true}` — need better frontend state | Show "Come back at season start" or dynasty roster analysis |
| 7 | Waiver wire blank | `role_change_flags` seed data doesn't match rostered players; off-season | Better off-season state or target free agents to monitor |
| 8 | Trade analyzer blank | Needs synced roster players | Fix after sync |
| 9 | War Room blank | Depends on roster + Claude analysis | Fix after sync |
| 10 | Scoring format wrong (displays redraft settings) | Same root cause as #1 | Fixed after re-sync |
| 11 | Mobile magic link fails if desktop already used link | Correct behavior — one-time use OTP | ✅ Now shows clear error + "Request new link" button |

### Next Session Priorities
| Priority | Task |
|---|---|
| P0 | Re-sync Chimp Dynasty after deploy lands |
| P0 | Verify `league_rosters` table has players for this user |
| P1 | Build off-season home page for dynasty (dynasty capital, age profile, upcoming draft, trade targets) |
| P1 | Audit every page for off-season empty states — replace blanks with meaningful content |
| P2 | Verify scoring format (PPR) is correct after re-sync |
| P2 | Test trade analyzer end-to-end with real roster |
| P3 | War Room — verify dynasty mode works (age/picks grading) |
| P4 | ESPN Phase 2 integration |

### Remaining Priorities
| Priority | Feature | Notes |
|---|---|---|
| High | Off-season content strategy | Dynasty pages need off-season purpose: rookie prep, trade market, dynasty capital analysis |
| Medium | ESPN full integration | OAuth design written in `api/routes/espn.py`. Ready to implement. |
| Low | Auction / salary cap support | Dollar valuations, live tracker |
| Low | Custom domain | Get a real domain, verify on Resend to fix spam folder |
| Low | Waitlist / self-serve signup | Invite-only beta is fine for now |

### Known Production Issues (ongoing)
| Issue | Status |
|---|---|
| All pages blank off-season | By design — needs off-season content built |
| Email alerts land in spam | Expected — using Resend shared domain. Fix: verify custom domain on Resend. |

### Credentials & Keys
| Service | Key/URL |
|---|---|
| Supabase URL | https://keksjsxvcymsjhzxubxw.supabase.co |
| Supabase Management PAT | sbp_54f17a410ee8e8e1373cb712202061d9041c5440 |
| Railway backend | https://web-production-dddf8.up.railway.app |
| Vercel frontend | https://ff-bot-web.vercel.app |
| GitHub backend | https://github.com/teel23/ff-bot |
| GitHub frontend | https://github.com/teel23/ff-bot-web |
| Resend API key | re_EZ5EJod6_5wDsFrvZBSL3QkqoycfdjLhN (in .env) |

---

## ❓ Deferred Questions (Stop and Note List)

These tasks were stopped due to missing context or decisions needed from user:

| # | Question | Feature blocked |
|---|---|---|
| 1 | ~~Gantt chart library~~ | ✅ Resolved — CSS-based, no library |
| 2 | ~~Power rankings algorithm~~ | ✅ Resolved — 60% roster strength + 40% recent form |
| 3 | ~~Notifications service~~ | ✅ Resolved — **email only** for injury alerts (Resend). Web push deferred. |
| 4 | ~~Boom/bust model~~ | ✅ Resolved — variance-based, server-side tiers |
| 5 | ~~Keeper optimizer cost-basis~~ | ✅ Resolved — user enters round costs manually; Sleeper doesn't expose it |
| 6 | ~~Draft assistant live sync~~ | ✅ Resolved — **polling** every 5–10s (stateless, draft pace is human-speed) |
| 7 | ~~Trade analyzer Mode B~~ | ✅ Resolved — table of all teams + top 3 partner cards + Mode A pre-fill |
| 8 | ~~Historical draft analysis~~ | ✅ Resolved — **Sleeper ADP primary** + scrape FantasyPros as supplement |

---

## 📐 Session 3 — Expanded Product Spec Decisions

All answers confirmed via Q&A interrogation session. These override or extend the original feature roadmap below.

### Platforms
- **Phase 1:** Sleeper (done)
- **Phase 2:** ESPN (OAuth required)
- ESPN is the only confirmed Phase 2 platform. Yahoo/Fleaflicker/MFL skipped for now.
- All platforms normalize into the same internal data model — engines are platform-agnostic.

### League Formats (all four, fully supported + optimized)
| Format | Key behavior |
|---|---|
| Redraft | Full draft every year, no carry-over. War Room = playoff push + roster health. |
| Dynasty | Rosters carry over, rookie drafts, age/pick value in all decisions. (already in plan) |
| Keeper | Fixed # of keepers. App recommends optimal keepers + shows value vs cost + trade-vs-keep. |
| Auction / Salary cap | Dollar valuations pre-draft, live auction tracker, inflation calculator, post-draft grade. |
- **Best ball: not now** — skip until managed leagues are complete.

### Scoring Format
- Auto-import from platform API on connect (exact weights pulled from Sleeper/ESPN).
- User can override individual stat weights after import.
- All engines use the live scoring format automatically — no manual config.

### Live Draft Assistant
- **Full live draft board**: real-time ADP, tier breaks, value alerts, auto-rank by team need as picks happen.
- **Integration**: auto-sync for Sleeper (API), manual pick entry fallback for ESPN and others.
- **Auction mode**: live bid tracker + inflation calculator as budget depletes.
- **Keeper mode**: pre-draft keeper optimizer (which X to keep + cost analysis shown).

### Keeper Rules
- Fixed # of keepers per team (only rule type needed for now).
- App provides: keeper optimizer (best X to keep) + value vs cost display + trade-vs-keep comparison.

### Historical Draft Analysis (Draft History Tab)
- **Data sources**: FantasyPros ADP + Underdog ADP + Sleeper ADP + user's own past drafts from connected leagues.
- **Historical depth**: as far back as data allows — no cutoff.
- **Three views combined in one tab**:
  1. Your draft replayed — each pick graded A–F vs ADP and actual finish
  2. Position by round heatmap — which positions/rounds returned the most value
  3. Player outcome cards — projected vs actual finish, boom/bust verdict per pick

### Boom/Bust Model
- **All four input pillars combined** with research-backed weights (more weight on more correlative factors):
  - Age curve + injury history (position-adjusted decline + recurrence risk)
  - Team context + role security (snap%, target share, depth chart, scheme)
  - ADP vs actual historical finish (track record of same-profile players)
  - Vegas / game script signals (implied team points, game total, pace)
- **Three display modes, all toggleable**:
  1. Tier groupings (Elite / High floor / High ceiling / Boom-or-bust / Fade)
  2. Probability bars (% chance to hit/miss based on composite)
  3. Scatter plot (X=ADP, Y=projected finish, color=bust risk)

### Player Profiles
- **Drawer**: click any player anywhere in the app → slide-out quick view panel.
- **Dedicated page**: full URL, sharable — career stats, age curve, boom/bust history, ADP trend, injury log, trade value history.

### Trade Analyzer (Fully Adaptive per League Type)
- **Dynasty**: age, pick equity, contend/rebuild mode, sell-high/buy-low timing.
- **Redraft**: this-season value only, position scarcity, playoff schedule.
- **Keeper**: future cost factored in (trade-vs-keep comparison built in).
- **Auction/Cap**: cap hit, contract years remaining, salary vs value.
- App detects format from league settings and silently adapts — no manual mode switch.

### GM War Room (Adapts by Format)
- **Dynasty**: scout report + grades (A–F across Age/Depth/Ceiling/Floor/Pick Capital) + Gantt timeline chart.
- **Redraft**: playoff push (magic numbers, % chance to make playoffs, SOS remaining) + roster health grades (sell-high/buy-low windows mid-season) — both combined in one view.

### Waiver Wire — Snake Waiver Priority
- Rolling priority: using a waiver claim drops you to last; not using it moves you up over time.
- App tracks waiver priority order per team per week (new table needed).
- Factors waiver priority into pickup urgency scoring.

### AI Chat (Claude) — All Four Modes
- Trade negotiation coach (why would they accept, what to counter with)
- Start/sit debate (argue both sides, give verdict)
- Waiver wire advisor (should I drop X for Y, how much FAAB, what's the risk)
- Season strategy advisor (contender or rebuilder, what moves to make)
- Floating button (bottom-right), 400px slide-in panel, context-aware per page.

### Notifications (all four types)
- Injury alerts on your players (immediate)
- Waiver wire role change alerts (before deadline)
- Trade offer received (from connected platform)
- Weekly lineup reminder (with recommended lineup included)

### Data Freshness
- Scheduled auto-refresh (daily + pre/post game windows).
- On-demand Sync button — user can force refresh anytime.

### Power Rankings (League Page)
- Driven by: current roster strength + recent form (last 3 weeks) + strength of schedule remaining.
- Dynasty capital (picks + young players) NOT included in power rankings (only in War Room).

### Mobile Priority Features (must work on mobile)
- Lineup setting (before Sunday lock)
- Trade offer review + response (accept/counter)
- Waiver wire claims (browse + submit)
- Home screen briefing

### Onboarding Target
- Under 5 minutes: email → magic link → connect platform username → auto-import → confirm team → ready.
- Invite-only beta now. Waitlist + approval self-serve later. No paywall until product-market fit.

### Social / Privacy
- All analysis is private. No cross-user data. No anonymized aggregates.

---

## ⚠️ Key Corrections from Session 2 Clarification
- **League format: DYNASTY** (not redraft — rosters carry over year to year, rookie drafts)
- **League size: 12 teams**
- **Waiver system: FAAB** (blind bidding, each manager has a budget)
- **Product scope: SaaS for ANY Sleeper league** — not just Chimp Dynasty. Must auto-import all settings.
- **Users: invite-only for beta + waitlist/approval for self-serve later**
- **Multi-team per user:** A single user can connect multiple teams across multiple leagues (and eventually multiple platforms — Sleeper, ESPN, Yahoo, etc.). Sidebar has a league/team switcher. All pages respect the active selection.

---

## 🗺️ Feature Roadmap (Full Spec)

### 0. Multi-Team / Multi-League Architecture
**Core requirement:** One user can have many teams across many leagues (and eventually many platforms).

**Data model:**
- `user_leagues` table already exists — one row per user+league combo
- Each row stores: league_id, platform (sleeper/espn/yahoo), team name, roster_id, all imported settings
- Active league stored in user's session/local state (not in DB — just which one is currently selected)

**UI — League Switcher:**
- Sits at the top of the sidebar, above the nav links
- Shows current team name + league name + platform icon (Sleeper logo for now)
- Click → dropdown of all connected leagues, click to switch
- All pages (lineup, waiver, trade, war room, league) re-fetch for the newly selected league
- "Connect another league" option at bottom of dropdown → connects a new Sleeper league (or ESPN/Yahoo later)

**Platform expansion plan (future):**
- Phase 1: Sleeper only (now)
- Phase 2: ESPN (OAuth, different API shape)
- Phase 3: Yahoo (OAuth)
- All platforms normalize into the same internal data model — engines don't care which platform

---

### 1. Auto-Import League Settings from Sleeper ✅ DONE (Session 4)
**Goal:** Zero manual configuration. Connect a Sleeper username → everything is auto-detected.

On league connect and re-sync, call `GET /v1/league/{id}` to pull:
- Scoring format (PPR / Half-PPR / Standard / custom) ← auto-detected from `scoring_settings.rec`
- Roster slots (QB count, RB, WR, TE, FLEX, SF, K, DEF) ← from `roster_positions`
- Waiver type (FAAB with budget, rolling priority, or free agent pool)
- Playoff weeks, bracket size, trade deadline week
- League name, avatar, commissioner

Store all in `user_leagues` table. Used everywhere automatically — scoring engine, lineup optimizer, waiver display, trade analyzer. **No manual config ever.**

Works for any Sleeper league any user connects — fully generic, not hardcoded to Chimp Dynasty.

---

### 2. Onboarding & Access Control
**Two modes:**

**Invite-only (beta):**
- Owner sends a magic invite link to a specific email
- User clicks link → account auto-created → connected to the invited league
- Existing `POST /invites` + `POST /accept-invite` flow (already built)

**Self-serve with approval gate:**
- Public signup page — user enters email + Sleeper username
- Goes into a waitlist/pending state
- Owner approves from admin panel → user gets magic link access
- User then connects their own Sleeper league (can be any league, not just Chimp Dynasty)

---

### 3. Home Screen — Weekly War Room Briefing ✅ DONE (Session 4)
**Layout: Card grid** (each section = a card, scannable)

**Cards:**
1. **Matchup Result** — This week's score, W/L, opponent, point differential
2. **Injury Report** — Injuries on your roster with status badges
3. **Position Depth** — Color-coded depth chart (deep/ok/thin/critical)
4. **Waiver Wire Alerts** — Top 3 role change pickups right now
5. **League News** — Recent transactions from Sleeper
6. **Quick Actions** — Links to Lineup, Waiver, Trade, War Room

**AI Briefing:** Claude's narrative summary at the top of the page — 3-4 sentences, scout memo style.

---

### 4. Roster Tab ✅ DONE (Session 4)
**Default sort:** Starters first (QB → RB → WR → TE → FLEX), then bench

**Per player row:**
- Position badge (colored by decision)
- Name, projected points, floor-ceiling range, matchup grade (A-F)
- Trend indicator (▲▼—)

**On row click → slide-out drawer:**
- Grade + decision badges
- Full stats (projected, floor, ceiling, composite, confidence)
- Risk indicator
- Claude reasoning

---

### 5. Lineup Optimizer ✅ (Session 3)
**Default view:** Lineup by slot, with start/sit confidence scores and risk flags.

---

### 6. Waiver Wire v2 (Dynasty-aware, FAAB) ✅ Engines done
**UI needs update to use LeagueContext.**

---

### 7. Trade Analyzer v2 ✅ Engines done
**UI needs update to use LeagueContext + player name search.**

---

### 8. GM War Room ✅ DONE basic version (Session 4)
**Three outputs combined:**

**A. Claude Scout Report** ✅
2–3 paragraphs: team archetype, contend vs rebuild analysis, key decisions.

**B. Grades + Action Bullets** ✅
A–F per dimension: Age Profile, Depth, Upside/Ceiling, Floor/Consistency, Pick Capital

**C. Gantt / Timeline Chart** ⏳ DEFERRED (chart library decision needed)

---

### 9. League Page ✅ DONE basic version (Session 4)
**Data source:** Live Sleeper API via frontend fetch (direct, no backend caching needed for now)

**Sections:**
- **League info strip** ✅ — scoring format, type, team count, waiver type
- **Standings** ✅ — W/L/PF/PA from Sleeper API, highlights your team
- **Refresh button** ✅ — re-syncs roster + re-fetches standings
- **Power Rankings** ⏳ — coming soon (algorithm + backend route needed)

---

### 10. Rookie Draft Prep Page (Dynasty Off-Season)
Not yet started.

---

### 11. UI Design System — War Room Aesthetic ✅ Implemented
- A-F grades with colors
- Position depth color coding (deep/ok/thin/critical)
- Slide-out drawer pattern
- League switcher in sidebar
- ChatPanel floating button

---

## 🐛 Session 5 Changes (March 5, 2026)

| Change | Detail |
|---|---|
| `api/routes/players.py` | New — `GET /players/search?q=&limit=&league_id=` — partial name match on players table, marks rostered players with `on_roster: true` |
| `api/main.py` | Registered `/players` route |
| `ff-bot-web/lib/api.ts` | Added `PlayerSearchResult` type + `searchPlayers()` function |
| `ff-bot-web/app/dashboard/leagues/page.tsx` | Removed manual scoring_format picker (now auto-detected). Shows imported settings as pills (type, waivers, FAAB budget, playoff week, team count). Uses `refreshLeagues()` from context instead of self-fetching. |
| `ff-bot-web/app/dashboard/lineup/page.tsx` | Switched from self-fetching leagues to `useLeague()` context. Active team + scoring format shown in header. Removed league dropdown. |
| `ff-bot-web/app/dashboard/waiver/page.tsx` | Same LeagueContext switch. Waiver type + FAAB budget displayed in header bar. |
| `ff-bot-web/app/dashboard/trade/page.tsx` | Full rewrite — LeagueContext + player name search with autocomplete dropdown. Debounced 300ms search-as-you-type. On-roster players flagged in green. No more raw ID text areas. |

---

## 🐛 Session 4 Changes (March 4, 2026)

| Change | Detail |
|---|---|
| `db/migrations/002_league_settings.sql` | New migration — adds league_type, waiver_type, waiver_budget, playoff_start_week, playoff_teams, trade_deadline, roster_positions, scoring_settings to user_leagues |
| `api/routes/leagues.py` | `_parse_league_settings()` — auto-imports all settings from Sleeper API; scoring_format now optional (auto-detected); new response fields |
| `api/routes/leagues.py` | Added `fetch_sleeper_matchups()` + `fetch_sleeper_transactions()` helpers (used by home route) |
| `engine/chat.py` | New — 4 context modes (trade/lineup/waiver/general), conversation history support |
| `api/routes/chat.py` | New — `POST /chat/message` |
| `api/routes/home.py` | New — `GET /home/overview` — aggregates matchup, injuries, team needs, waiver alerts, transactions, Claude briefing |
| `api/main.py` | Registered /home and /chat routes |
| `ff-bot-web/lib/api.ts` | Added HomeOverview types + `getHomeOverview()`, `sendChatMessage()`; League type extended with new settings fields |
| `ff-bot-web/app/dashboard/league-context.tsx` | New — shared active league state via React context + localStorage persistence |
| `ff-bot-web/components/ChatPanel.tsx` | New — floating chat button + 400px slide-in AI chat panel |
| `ff-bot-web/app/dashboard/layout.tsx` | Overhauled — new nav (8 items), league switcher dropdown, ChatPanel added |
| `ff-bot-web/app/dashboard/page.tsx` | Rebuilt — card grid home (matchup, injuries, depth, waivers, news, quick links) + Claude briefing |
| `ff-bot-web/app/dashboard/roster/page.tsx` | New — slot-ordered roster, clickable rows, slide-out drawer with grades + Claude reasoning |
| `ff-bot-web/app/dashboard/war-room/page.tsx` | New — scout report + A-F grades + action items (Claude-generated) |
| `ff-bot-web/app/dashboard/league/page.tsx` | New — live standings from Sleeper, league info strip, sync button |

---

## 🐛 Session 3 Fixes (March 4, 2026)

| Bug | Fix |
|---|---|
| Claude wrapping JSON in ` ```json ` fences → `json.loads()` failed | Added `_parse_json()` in `engine/claude.py` — strips markdown fences before parsing |
| `MAX_TOKENS=1024` too small for 33-player roster → JSON truncated mid-response | Raised: lineup=4096, waiver=2048, trade=512 |
| Lineup sending all 33 players to Claude → token overflow | Now sends starters + top 6 bench only |
| `role_change_flags` seeded with rostered players → waiver alerts always empty | Seed script now fetches test league roster and excludes those player_ids from flags |
| Backup QB opp_score inflated to 95 (no 2024 data → default snap=0.95) | POSITION_DEFAULTS now use conservative backup-tier snap values (QB default: 0.40) |
| `upsert()` on 2nd seed run fails with duplicate key constraint | Added `on_conflict` param to `upsert_batched()` keyed per table |

---

## 🐛 Session 2 Fixes (March 3, 2026)

| Bug | Fix |
|---|---|
| `_find_user_roster()` always returned roster[0] | Rewrote: username lookup → sleeper_user_id → match roster by owner_id; team picker fallback |
| `_sync_roster_to_db` keyed by wrong field | Changed `{p["player_id"]: p}` → `{p["sleeper_id"]: p}` |
| injuries.py FK violation (ESPN ID ≠ player_id) | Skip players not found in players table instead of using raw ESPN ID |
| Stale Supabase session "Invalid token" | Changed `getSession()` → `getUser()` (validates server-side); added `ready` state guard |
| PyJWT local verification failing (base64 secret format) | Dropped PyJWT entirely; now calls Supabase `/auth/v1/user` to validate tokens |
| `recommend() unexpected keyword argument 'user_id'` | Added `user_id: Optional[str] = None` to `LineupEngine.recommend()` + `WaiverEngine.analyze()` |

---

## 🐛 Known Issues / Context

### 2025 Weekly Stats
- nflverse combined parquet only goes to 2024; per-year 2025 parquet returns 404
- **Status:** Not yet published by nflverse — will become available mid-2026
- **Fix in place:** `ingestion/historical.py` tries 3 sources and fails gracefully; will auto-populate once nflverse publishes

### 2025 Rosters (historical.py fix from this session)
- Old code called `_get_year_df()` which didn't exist → caused all roster years to fail
- **Fixed:** 2022–2024 use `nfl.import_seasonal_rosters([yr])`, 2025 uses direct parquet URL

### 2025 Schedules
- No per-year parquet published yet
- **Fix in place:** 2025 schedules extracted from `play_by_play_2025.parquet` (285 games ✓)

### snap_counts FK
- Snap counts use `pfr_player_id`, not GSIS `player_id` (different ID system)
- FK constraint was dropped in Supabase → `ALTER TABLE snap_counts DROP CONSTRAINT snap_counts_player_id_fkey`

### Python version
- Mac uses Python 3.9 (Xcode Command Line Tools) — no brew installed
- All code uses `from __future__ import annotations` for 3.9 compatibility
- Always run from `/Users/carsonteel/Desktop/AI/FF Bot/ff-bot`

### gh CLI
- Not on PATH — downloaded to `/tmp/gh_dir/gh_2.87.3_macOS_amd64/bin/gh` (temp, won't persist)
- For future use: download from https://github.com/cli/cli/releases/latest (macOS amd64 zip)
- Auth token stored in keychain for account `teel23`

---

## 📁 Project Structure

```
~/Desktop/AI/FF Bot/
├── SESSION_NOTES.md          ← you are here
├── Fantasy Football AI Bot Plan.rtf
├── Research.zip
├── ff-bot/                   ← backend (on GitHub: teel23/ff-bot)
│   ├── .env                  ← API keys (never commit)
│   ├── .env.example
│   ├── .gitignore
│   ├── requirements.txt
│   ├── README.md
│   ├── api/                  ← FastAPI routes
│   │   ├── main.py
│   │   ├── dependencies.py   ← auth (Supabase /auth/v1/user — no PyJWT)
│   │   └── routes/
│   │       ├── chat.py       ← NEW: POST /chat/message
│   │       ├── home.py       ← NEW: GET /home/overview
│   │       ├── lineup.py
│   │       ├── waiver.py
│   │       ├── trade.py
│   │       └── leagues.py    ← updated: auto-import league settings
│   ├── db/
│   │   ├── client.py
│   │   ├── schema.sql        ← Supabase schema (already deployed)
│   │   └── migrations/
│   │       ├── 001_sleeper_user_matching.sql
│   │       └── 002_league_settings.sql  ← NEW — run this in Supabase
│   ├── engine/               ← AI logic
│   │   ├── chat.py           ← NEW: context-aware Claude chat
│   │   ├── claude.py
│   │   ├── consensus.py
│   │   ├── lineup.py
│   │   ├── trade.py
│   │   └── waiver.py
│   ├── ingestion/
│   │   ├── historical.py     ← main data pipeline
│   │   ├── injuries.py
│   │   ├── news.py
│   │   ├── projections.py
│   │   ├── vegas.py
│   │   └── weather.py
│   ├── scripts/
│   │   ├── probe_nflverse_urls.py
│   │   ├── seed_test_data.py
│   │   └── weekly_learning.py
│   ├── tests/
│   └── utils/
│       └── helpers.py
└── ff-bot-web/               ← Next.js 16 frontend
    ├── .env.local            ← NEXT_PUBLIC_SUPABASE_*, NEXT_PUBLIC_API_URL
    ├── lib/
    │   ├── supabase.ts
    │   └── api.ts            ← updated: HomeOverview + Chat types + functions
    ├── components/
    │   └── ChatPanel.tsx     ← NEW: floating AI chat button + slide-in panel
    └── app/
        ├── globals.css       ← dark theme CSS variables
        ├── layout.tsx
        ├── page.tsx
        ├── login/page.tsx
        ├── accept-invite/page.tsx
        ├── auth/callback/route.ts
        └── dashboard/
            ├── layout.tsx    ← updated: new nav, league switcher, ChatPanel
            ├── league-context.tsx  ← NEW: shared active league React context
            ├── page.tsx      ← rebuilt: card grid home + Claude briefing
            ├── roster/page.tsx     ← NEW: slot-ordered roster + drawer
            ├── lineup/page.tsx
            ├── waiver/page.tsx
            ├── trade/page.tsx
            ├── war-room/page.tsx   ← NEW: scout report + grades + actions
            ├── league/page.tsx     ← NEW: live standings + league info
            └── leagues/page.tsx    ← existing: connect + team picker
```

---

## 🔑 Key Credentials (stored in .env)
- Supabase project: `keksjsxvcymsjhzxubxw.supabase.co`
- Sleeper league ID: `1328784373125771264`
- Email: `01teel@gmail.com`
- GitHub: https://github.com/teel23/ff-bot
