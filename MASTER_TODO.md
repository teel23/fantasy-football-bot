# FF AI Bot — Master TODO
**Last updated:** July 8, 2026 (audit pass — S40/41 shipped most HIGH items)
**Source of truth for all open work. Update this every session.**

---

## ✅ FIXED Session 19 (Mar 11, 2026) — App Is Now Working

These were the critical blockers. All resolved.

- [x] **Production API URL** — Vercel was pointing to localhost. Fixed via `next.config.ts` + removed bad env var from Vercel dashboard.
- [x] **CORS blocking frontend** — Added `allow_origin_regex` for `*.vercel.app` in FastAPI.
- [x] **League sync / slot_type** — All 33 Chimp Dynasty players synced with correct starter/bench/taxi/ir slots.
- [x] **Player ages** — 4,329 of 5,303 players now have age, years_exp, college, draft data.
- [x] **War Room** — Using real roster data, JSON parse fixed with pre-fill assistant turn.
- [x] **Off-season home page** — Mode=offseason, 33 players, 5 AI priorities, 3 trade targets.
- [x] **Waiver wire** — Returning pickups (Chris Godwin top, with Claude reasoning).
- [x] **Trade analyzer** — Real verdict + reasoning.
- [x] **Role change flags** — 8,154 rows backfilled for 2025 season.
- [x] **user_leagues settings** — league_type, waiver_budget, scoring_settings all populated.

---

## 🔴 MUST DO (before emails / next season)

- [ ] **Add Railway env vars** — `RESEND_API_KEY`, `ALERT_FROM_EMAIL`, `FRONTEND_URL=https://ff-bot-web.vercel.app`. Email alerts are coded but won't send without these.
- [ ] **Add GitHub Secrets** — `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL`. Required for the daily injury alert and lineup reminder workflows.
- [ ] **Re-run 2025 ingestion** — `python -m ingestion.historical` — once nflverse publishes `player_stats_2025.parquet` (currently 404). Will populate `weekly_stats` for 2025 season and unlock scoring history features.

---

## 🟠 HIGH — Core UX Gaps (real users will notice these)

### Roster Page
- [x] **Group by slot type** *(shipped S40/41)* — roster page currently groups by position. Needs Starters / Bench / Taxi Squad / IR grouping to match Sleeper's display. `slot_type` column is populated — just needs the frontend to use it.
- [ ] **Inline dynasty value** — show computed KTC value per player row (already available from `/players/{id}/profile`). Add as a subtle badge.
- [ ] **Inline injury badge** — if a player has an entry in `injuries` table, show status (Out/Doubtful/Q) on their row in roster.

### Waiver Wire
- [x] **Drop suggestion pairing** *(shipped — waiver cards show Drop: candidate)* — every pickup recommendation should include a "Drop candidate: [player]" to complete the move. Use roster depth by position to suggest who to drop.
- [x] **Filter by position** — shipped; persists in localStorage (`ff_waiver_pos_filter`).

### Dynasty Off-Season Hub
- [ ] **Dynasty offseason calendar** — card on home dashboard showing current phase: Free Agency (March), NFL Draft (April), Rookie Draft (May–July), Training Camp (Aug), Season start (Sep).
- [ ] **Sell-high / buy-low targets** — use age + opportunity trend + dynasty value to flag players to trade now vs players to target. Surface on home dashboard off-season layout.

---

## 🟡 IMPORTANT — Dynasty Features

### Prior Year Draft Analysis (highest-value next feature)
- [ ] **Historical draft grade** — for any past Sleeper draft, show pick-by-pick grades vs actual 2024 production. Already have `weekly_stats` 2022–2024 data. Just need to build the endpoint and UI.
- [ ] **"Where are they now" view** — draft pick → ADP → actual dynasty value today → hit/miss verdict.
- [ ] **Draft class performance** — aggregate by year. Which RBs from 2022 draft class hit? Visualize as a grid.

### Boom/Bust Profiles (needed for player pages to be useful)
- [ ] **Build boom/bust engine** — use `weekly_stats` variance, age/position decline curve, opportunity score trend. Output tier: Elite / Safe Floor / High Ceiling / Boom-Bust / Aging / Bust Risk.
- [ ] **Surface on player pages** — boom/bust tier badge on `PlayerProfileDrawer` with one-line explanation.
- [ ] **Draft board filter** — sort rookie draft board by boom/bust tier, not just rank.

### League-Settings-Aware Rankings
- [ ] **Scoring format weighting** — `scoring_settings` JSONB is now stored on `user_leagues` for Chimp Dynasty. Use it to re-weight player values. PPR shifts WR/TE up; SF/2QB shifts QB value heavily.

---

## 🟡 IMPORTANT — Data Pipeline

- [ ] **2025 weekly_stats** — nflverse `player_stats_2025.parquet` currently returns 404. Check monthly. Run `python -m ingestion.historical` when available.
- [ ] **snap_counts ID format** — snap_counts uses pfr-format IDs (`GholWi00`). Cannot join to players table (GSIS format). If snap-based features need player names, build a pfr→GSIS crosswalk table or use `player_name` string matching.
- [ ] **Weekly pipeline schedule** — GitHub Actions `daily_ingestion.yml` exists but secrets aren't set. Once Railway env vars + GitHub secrets are added, ingestion will run automatically on schedule.
- [ ] **Live scoring (in-season)** — Sleeper `GET /v1/stats/nfl/regular/{season}/{week}` for live fantasy points during games. Not needed until September.

---

## 🟡 IMPORTANT — Notifications & Email

- [ ] **Resend domain fix** — emails currently send from `onboarding@resend.dev` (or unset). Will land in spam. Need: verify a real domain in Resend, configure SPF/DKIM, set `ALERT_FROM_EMAIL=alerts@yourdomain.com`.
- [ ] **Email test** — once Railway env vars set, run `python -m scripts.test_email` to confirm Resend is working.
- [ ] **Role change alerts** — notify when a player on your roster triggers a role_change_flag. Already have 8,154 flags populated. Need the alert logic to cross-reference against league_rosters.

---

## 🟢 MEDIUM — Polish

- [ ] **Player search on roster page** — add search box to quickly look up any player's dynasty profile without going to trade page.
- [ ] **War Room — "Rebuild vs Contend" tab** — works when report is generated. Standalone tab needs real data before generate is clicked (show current roster age profile, no report required).
- [ ] **Mobile auth** — magic link fails on mobile if desktop session already active. Known issue. Workaround: OTP fallback (already built). Root fix: clear old sessions on new link request.
- [ ] **League switcher loading** — switching leagues flashes stale data briefly. Add loading state in LeagueContext.

---

## 🔵 LATER — Phase 2+ Features

- [ ] **Self-learning loop** — `weekly_learning.py` is coded but inert. Activate once 2025 actual vs projected data is available.
- [ ] **Live scoring integration** — Sleeper real-time stats during games. Deferred until in-season.
- [ ] **ESPN integration** — OAuth design exists in `api/routes/espn.py`. Implement post-Sleeper.
- [ ] **Custom domain** — needed for Resend email fix and branding.
- [ ] **Waitlist / self-serve signup** — currently invite-only. Add when ready for broader beta.
- [ ] **Web push notifications** — deferred. Email first.

---

## ✅ Done (confirmed working as of Mar 11, 2026)

- Supabase schema (19+ tables, RLS, all Session 18/19 migrations applied)
- FastAPI 2.0.0 backend on Railway — all routes registered and responding
- Next.js 16 frontend on Vercel — Railway URL hardcoded in `next.config.ts`
- CORS — all `*.vercel.app` origins allowed
- Magic link auth + 6-digit OTP fallback + "use on this device" warning
- Sleeper league connect + auto-import settings (dynasty detection, scoring, waiver type)
- `user_leagues` settings populated (PPR, dynasty, FAAB $1000, 12 teams)
- `league_rosters` — 33 players with correct slot_type (starter/bench/taxi/ir)
- Player bios — age, years_exp, college, draft_round, draft_pick (4,329 players)
- `player_profiles` cache table in Supabase
- `build_agent_context()` — real roster data passed to every Claude call
- Home dashboard — offseason mode with AI priorities, dynasty capital, roster summary
- War Room — real grades from actual roster, Gantt chart with player ages, JSON parse fix
- Dynasty Capital page — picks owned/owed, net score, AI analysis
- Roster Timeline — 33 players with correct ages and prime window chart
- Power Rankings — 12 teams ranked, off-season fallback using 2024 stats
- Lineup Optimizer — 33 recommendations with start/sit decisions
- Waiver Wire — pickups from projections + opportunity_metrics fallback, Claude reasoning
- Trade Analyzer — real verdict with player-specific reasoning
- Notifications — 457 alerts loading
- Role change flags — 8,154 rows backfilled for 2025 season
- Off-season states — all 6 pages (roster, lineup, waiver, trade, draft, keepers) handle off-season
- SkeletonCard loading states across home and war room
- Email system coded (Resend) — needs Railway env vars to activate
- GitHub Actions workflows — daily_alerts, lineup_reminder, daily_ingestion, weekly_learning
- Invite-only beta access flow
- Mobile responsive layout (sidebar hamburger)
