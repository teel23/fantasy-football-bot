# Fantasy Football AI Bot — Project Notes
> Reference for jumping back into this project.
> Last updated: March 16, 2026 (Session 28 complete — Competitive UX Redesign, Sprints 1–4)

## Session 28 — March 16, 2026 (Competitive UX Redesign — Sprints 1–4)

**Goal:** Make the frontend competitive with FantasyPros, Sleeper, ESPN, and 4for4. "We have the data. We just don't show it." Pure Tailwind + CSS/SVG, no new npm packages, build clean after every sprint.

### Sprint 1 — Shared Primitives ✅

**New files:**
- `lib/constants.ts` — `POS_COLOR`, `GRADE_COLOR`, `BOOM_BUST_BORDER` shared color maps (single source of truth)
- `components/StatChip.tsx` — micro inline stat chip: 9px muted uppercase label + 13px semibold value
- `components/TierDivider.tsx` — edge-to-edge 28px tier separator with `--tier-divider` background
- `components/PlayerInlineRow.tsx` — universal dense player row: `[POS badge] [name+team+injury dot] [chips] [trailing badge]`. Renders as `<button>` if onClick, else `<div>`. INJ dot map (Out/Doubtful/Questionable/IR).
- `app/globals.css` — 5 new CSS vars: `--surface3`, `--tier-divider`, `--green-dim`, `--yellow-dim`, `--red-dim`

### Sprint 2 — Waiver + Lineup ✅

**`waiver/page.tsx`** — Full rewrite:
- Sort modes: `rank | proj | opp | role` with sort button row below position filters
- FAAB banner when `waiver_type==="faab"` and budget > 0
- `<details>` collapsed reasoning (replaces always-visible block)
- Drop candidate reduced to inline single line: "Drop: [Name] +N more"
- Player rows → `PlayerInlineRow` with Proj / Opp (color-coded: ≥7 green / 4-6 yellow / <4 red) / Role chips

**`lineup/page.tsx`** — Full rewrite:
- `RangeBar` component: 80px CSS bar, blue fill from floor to ceiling, white dot at projected
- `groupByTier()` groups recommendations by `sit_start_tier`
- `TIER_ORDER`: Definite Start → Lean Start → Toss-Up → Lean Sit → Sit
- Full tier-grouped list with `TierDivider` between groups
- `PlayerRow` uses `PlayerInlineRow` + `<details>` reasoning + 3 chips (Proj, Matchup, Conf) + `RangeBar` as trailingBadge
- Optimal lineup strip: shows `result.optimal_lineup` with position cross-referenced from recommendations

### Sprint 3 — All Remaining Pages ✅

**`roster/page.tsx`** — Full rewrite:
- `HealthStrip`: 4-segment bar (QB/RB/WR/TE) colored by healthy ratio (>0.8 green / >0.5 yellow / else red)
- `SlotBadge` component for slot type
- `"position"` sort mode with `positionGroups` via useMemo
- Position-grouped view uses `TierDivider` with position name + POS_COLOR
- `PlayerInlineRow` with DV chip, age chip (≤24 green / 25-27 yellow / 28-30 orange / ≥31 red), SlotBadge trailing, BOOM_BUST_BORDER left border

**`schedule/page.tsx`** — Full rewrite:
- Playoff seeding banner using `myRank` + `activeLeague.playoff_teams`
- Record + Sparkline header card: large W-L + PF/PA totals + CSS bar chart per week
- Table adds PF, PA (hidden sm), +/- (hidden sm) columns with header row

**`trade/page.tsx`** — Edited:
- DV Balance Meter: fetches all rosters, computes give/recv dynasty value, proportional green/red bar, "Overpaying by X DV" / "Getting X DV surplus" label
- Verdict card: centered `text-3xl font-black`, trajectory line, `<hr>` before reasoning

**`war-room/page.tsx`** — Edited:
- `RadarChart` SVG component (220×220 viewBox): 5 axes (Age Profile / Upside / Pick Capital / Floor / Depth), 5 grid rings, data polygon (blue fill + accent stroke), grade labels at vertices
- Replaced flat grades grid with flex layout: radar chart (max 260px) + grades column

**`dynasty-capital/page.tsx`** — Edited:
- Capital Balance bar: green = owned / (owned+owed), red remainder, owned/net/owed labels
- Pick Timeline grid: unique seasons × rounds 1-3, green circle = owned, red = owed, grey = neither

**`league/page.tsx`** — Edited:
- Power Rankings: composite number → visual bar (score label + thin accent progress bar, capped at 100%)
- Standings: added +/- column (6-column grid, `hidden sm:block`)
- Removed local `POS_COLOR` (was wrong QB=#ef4444), now imports from constants.ts

### Sprint 4 — Layout Polish + Consolidation ✅

**`layout.tsx`** — Edited:
- Top bar context strip: team name + `Wk {nflWeek}` pill (accent, between hamburger and bell)
- Mobile bottom nav: when `nflWeek` truthy, 5th tab = Lineup ▶; when falsy (offseason) = Menu ☰

**POS_COLOR consolidation** — Removed 5 local `const POS_COLOR` definitions:
- `war-room/page.tsx`, `rookie-draft/page.tsx`, `keepers/page.tsx`, `trade/page.tsx`, `draft/page.tsx`
- All import from `@/lib/constants`
- Also removed duplicate `GRADE_COLOR` from `draft/page.tsx`

### Files Changed (ff-bot-web)
| File | Change |
|------|--------|
| `lib/constants.ts` | New — shared POS_COLOR, GRADE_COLOR, BOOM_BUST_BORDER |
| `components/StatChip.tsx` | New — inline stat chip |
| `components/TierDivider.tsx` | New — tier separator row |
| `components/PlayerInlineRow.tsx` | New — universal dense player row |
| `app/globals.css` | 5 new CSS vars |
| `app/dashboard/waiver/page.tsx` | Sort modes, PlayerInlineRow, FAAB banner, collapsed reasoning |
| `app/dashboard/lineup/page.tsx` | RangeBar, tier groups, optimal lineup strip |
| `app/dashboard/roster/page.tsx` | HealthStrip, position groups, age/DV chips |
| `app/dashboard/schedule/page.tsx` | Seeding banner, sparkline, PF/PA/+/- columns |
| `app/dashboard/trade/page.tsx` | DV Balance Meter, upgraded verdict card |
| `app/dashboard/war-room/page.tsx` | SVG radar chart |
| `app/dashboard/dynasty-capital/page.tsx` | Capital balance bar, pick timeline grid |
| `app/dashboard/league/page.tsx` | Composite bar viz, +/- column, POS_COLOR fix |
| `app/dashboard/layout.tsx` | Top bar context strip, conditional Lineup mobile tab |
| `app/dashboard/draft/page.tsx` | Import POS_COLOR+GRADE_COLOR from constants |
| `app/dashboard/keepers/page.tsx` | Import POS_COLOR from constants |
| `app/dashboard/rookie-draft/page.tsx` | Import POS_COLOR from constants |
| `app/dashboard/page.tsx` | ActionBar component (day-of-week nudge), position depth mini bars |
| `app/dashboard/agent/page.tsx` | nflWeek passed to agent chat |
| `lib/api.ts` | week param in sendAgentChat |

### Pushed
- ff-bot-web commit: `806889b`

---

## Session 27 — March 16, 2026 (Full Audit + 10 Bug Fixes)

**Goal:** Full read-through of every file in the codebase (all routes, engine, ingestion, dashboard pages, components). Write `BROKEN_FEATURES.md`, fix everything found in priority order, write `STATUS_REPORT_S27.md`.

### Audit Output ✅
- `ff-bot/BROKEN_FEATURES.md` — 10 issues catalogued across 3 severity tiers (3 critical, 3 major, 4 minor)
- `ff-bot/STATUS_REPORT_S27.md` — 7-section status report

### Critical Fixes ✅

**C1 — `api/routes/players.py`** (`get_player_dynasty_profile`)
Sync `client.messages.create()` called directly in `async def` handler — blocked entire server on every Player Profile Drawer open. Fixed: wrapped in `run_in_executor`. Added `import asyncio`.

**C2 — `engine/waiver.py`** (`WaiverEngine.analyze()`)
`_get_all_drop_candidates()` — sync method with Claude call — was invoked from async `analyze()` without executor. Blocked server on every `/waiver/analyze` for logged-in users. Fixed: wrapped call in `run_in_executor`.

**C3 — `engine/consensus.py`** (matchup pillar dead weight)
Matchup pillar (12% weight) had no ingestion source and always returned 50.0 — a phantom constant in every composite score. Fixed: redistributed weight to active pillars (opportunity 35%→42%, projection 25%→30%). Zeroed out all matchup position deltas. Fixed `get_weights()` floor logic so 0-weight pillars don't snap to 0.03.

### Major Fixes ✅

**M1 — `api/routes/health.py`**
`select("id")` — wrong column on players table (PK is `player_id`). DB health check always errored. Fixed: `select("player_id")`.

**M2 — `api/routes/viz.py`** (`get_player_trend`)
Queried `user_leagues.current_week` column that doesn't exist in schema — always defaulted to week 18, chart always showed weeks 10–18. Fixed: removed DB lookup, added `current_week: int = Query(default=18)` param. Frontend passes `nflWeek`.

**M3 — `api/routes/agent.py`** + frontend
`league.get("current_week")` always `None` (column doesn't exist) — agent chat always used week 1 for all data queries. Fixed: added `week: Optional[int]` to `AgentChatRequest`. Updated `lib/api.ts` type + `agent/page.tsx` to pass `nflWeek` in request body.

### Minor Fixes ✅

**m1 — `app/dashboard/schedule/page.tsx`** + new `api/routes/schedule.py`
19 direct browser → Sleeper API calls per page load. Fixed: new `GET /schedule/matchups` backend endpoint fetches all 17 weeks + users + rosters in one `asyncio.gather` batch. Frontend now makes a single authenticated call. Mounted in `main.py` at `/schedule`.

**m2 — `tasks/lineup_reminder.py`**
Function signature was `run_lineup_reminders()` (no args) but route calls it with `week=` and `season=`. Also used wrong table (`users` instead of `profiles`). Also returned `None` instead of a dict. Fixed all three.

**m3 — `api/routes/home.py`** (`_resolve_player_names`)
Opened a fresh `get_db()` connection instead of using passed-in `db`. Fixed: added `db=None` parameter. Both callers updated.

**m4 — `api/routes/home.py`** (`_build_offseason_calendar`)
All NFL offseason dates hardcoded to 2026. Fixed: year now computed from `date.today()` — current year if before July, next year otherwise. Stays correct across seasons without code changes.

### Files Changed
| File | Change |
|------|--------|
| `api/routes/players.py` | `import asyncio`; sync Claude wrapped in `run_in_executor` |
| `engine/waiver.py` | `_get_all_drop_candidates` call wrapped in `run_in_executor` |
| `engine/consensus.py` | matchup weight 0.12→0.00; redistributed to opportunity/projection; floor fix |
| `api/routes/health.py` | `select("id")` → `select("player_id")` |
| `api/routes/viz.py` | Removed phantom `current_week` DB lookup; `current_week` as query param |
| `api/routes/agent.py` | `week` field added to `AgentChatRequest`; `current_week = request.week or 1` |
| `api/routes/home.py` | `_resolve_player_names` accepts `db` param; offseason calendar year computed |
| `api/routes/schedule.py` | New — proxies 17 Sleeper week fetches in one batch |
| `api/main.py` | `schedule` router imported + mounted at `/schedule` |
| `tasks/lineup_reminder.py` | Fixed signature, `profiles` table, return dict |
| `ff-bot-web/lib/api.ts` | `week?: number` added to `sendAgentChat` payload type |
| `ff-bot-web/app/dashboard/agent/page.tsx` | Destructure `nflWeek`; passes `week: nflWeek` |
| `ff-bot-web/app/dashboard/schedule/page.tsx` | Replaced 19 Sleeper calls with single backend call |

### Pending — Push to main
- [ ] **Commit + push `ff-bot` changes** to Railway
- [ ] **Commit + push `ff-bot-web` changes** to Vercel

---

## Session 26 — March 15, 2026 (Audit + Bugs + Open Registration + Agent + Sync UX)

**Goal:** Full pipeline audit, fix all known bugs, open registration (remove invite-only), upgrade agent to use real data context, add usePageCache + SyncButton to all 8 data pages.

### AUDIT_S26.md ✅
- Full read-only audit across all 5 layers: ingestion → engine → API routes → frontend → auth
- 7 bugs identified and fixed in this session

### Bug Fixes ✅
1. **`ingestion/weather.py`** — `precip_pct` was using `clouds.all` (cloud coverage %) not actual precipitation probability. Added TODO comment — fix requires switching to `/forecast` endpoint (has `pop` field).
2. **`engine/consensus.py`** — `matchup` pillar (12% weight) is dead — no ingestion source. Added `# INACTIVE` comment disclosing this.
3. **`engine/trade.py`** + **`engine/lineup.py`** — Added `logging` + warning log when `dynasty_value` fallback fires (was silent).
4. **`engine/lineup.py`** — `week=1` was querying `opportunity_metrics` for `week=0` (returns empty). Fixed: `.eq("week", max(1, week - 1))`.
5. **`ingestion/weekly_stats.py`** — `_compute_opportunity_score` was deflating scores ~3× — stored pre-weighted components then divided by `len(scores)` (count, not weight sum). Rewrote with explicit `weight_sum` normalization.
6. **`api/routes/home.py`** — 3 sync Claude calls in async handlers were blocking the event loop. All wrapped in `asyncio.get_event_loop().run_in_executor(None, lambda: ...)`.
7. **`supabase/005_advanced_metrics.sql`** — Created safe re-runnable migration (`IF NOT EXISTS` / `ADD COLUMN IF NOT EXISTS`). **Still needs to be run manually in Supabase SQL Editor.**

### Open Registration ✅
- **`api/routes/auth.py`** — Added `POST /auth/register` endpoint. Idempotent: returns existing user if email already exists, creates new user with `role=member` if not. No invite required.
- **`app/enter/page.tsx`** — Two-step flow: email input → `/auth/lookup` → if found, sign in as usual; if not found → show confirmation step ("We'll create your account") → POST `/auth/register` → localStorage → `/dashboard`. Replaces old "Contact Carson for an invite" dead end.

### Agent Chat Upgrade ✅
- **`api/routes/agent.py`** — New `POST /agent/chat` endpoint. Assembles 11 data sources into Claude context: roster, weekly projections, opportunity metrics, injuries, recent news (7 days), Vegas lines, weather, top 10 waiver FAs, pillar weights, accuracy log (last 3 MAE weeks), league settings. Context truncated at 32,000 chars. Claude call in `run_in_executor`. Returns `{reply, data_freshness: {projections_week, news_latest, injuries_updated}}`.
- **`api/main.py`** — Registered `agent.router` at `/agent`.
- **`lib/api.ts`** — Added `sendAgentChat()` function + `AgentChatResponse` type.
- **`app/dashboard/agent/page.tsx`** — Full rewrite. Now calls `POST /agent/chat`. localStorage history persistence (`agent_history:{leagueId}`). Inline markdown renderer (no new npm packages: handles `**bold**`, `*italic*`, `` `code` ``, headers, lists). 4 quick-action chips. Data freshness footnote ("Data: news Xm ago · Week N projections"). Mobile safe-area padding. Clear chat button.

### /health/ping ✅
- **`api/routes/health.py`** — Added `GET /health/ping` returning `{ok: true, ts: "..."}`. Used by cold-start banner.

### Viz Endpoints ✅
- **`api/routes/viz.py`** — 5 pure-DB chart-ready endpoints (no Claude, all <500ms):
  - `GET /viz/player-trend/{player_id}` — weekly actual vs projected trend
  - `GET /viz/roster-health` — injury status counts by position
  - `GET /viz/scoring-breakdown` — pillar scores for starters
  - `GET /viz/league-standings` — W/L/PF standings via Sleeper
  - `GET /viz/waiver-opportunity` — top FA opportunity scores by position
- **`api/main.py`** — Registered `viz.router` at `/viz`.

### SyncButton + usePageCache ✅
- **`hooks/usePageCache.ts`** — New hook. localStorage-backed stale-while-fresh caching with configurable TTL. On mount: immediately returns cached data, always fetches fresh in background. On error: keeps cached data.
- **`components/SyncButton.tsx`** — New component. Green dot = fresh (within TTL), orange dot = stale. Spinner during active fetch. "Synced Xm ago" label, auto-updates every minute.
- **`app/dashboard/layout.tsx`** — Cold-start banner: pings `/health/ping` on mount, shows yellow dismissible banner if >5s or timeout.
- SyncButton added to all 8 data pages:
  - **Dashboard** — shows when `/home/dashboard` was last fetched; clicking re-fetches
  - **Roster** — shows when roster was last loaded; clicking re-fetches
  - **Draft** — shows when draft list was last loaded; clicking re-fetches
  - **War Room** — shows when standings+timeline were last loaded; clicking refreshes background data
  - **Waiver** — shows when waiver priority was last fetched; clicking re-fetches priority
  - **Keepers** — shows when roster was last loaded; clicking re-fetches
  - **Lineup** — shows when last analysis was run (only visible after first analysis); clicking re-runs
  - **Trade** — user-triggered only, SyncButton skipped

### Both repos pushed to main ✅
- ff-bot commit: `db6562c`
- ff-bot-web commit: `7b8f859`

### Pending — Push to main (S27)
- [ ] **Commit + push `ff-bot`** (13 files changed)
- [ ] **Commit + push `ff-bot-web`** (3 files changed: agent/page.tsx, schedule/page.tsx, lib/api.ts)

### Pending — Manual Action Required
- [ ] **Run `supabase/005_advanced_metrics.sql` in Supabase SQL Editor** — adds `racr`, `wopr`, `air_yards`, `epa` cols + indexes. Safe to re-run.

---

## Session 25 — March 15, 2026 (Infrastructure + Feature Sprint)

**Goal:** Confirm all infrastructure in place, implement lineup START/SIT tiers, trade opponent roster panel, schedule page polish, and a UX pass.

### Infrastructure ✅
- **005_advanced_metrics.sql** — Run in Supabase SQL Editor. Adds `racr`, `wopr`, `air_yards`, `epa` cols to `opportunity_metrics`; creates `espn_ownership` table with RLS policy and indexes.
- **Railway env vars** — All confirmed present: `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `SUPABASE_ANON_KEY`, `ANTHROPIC_API_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL`, `SLEEPER_LEAGUE_ID`, `SUPABASE_JWT_SECRET`, `APP_ENV`, `NFL_SEASON`.
- **GitHub Secrets** — Not yet set but Railway already has all needed vars. Actions will fail if triggered but not blocking current work.

### Track 1 — Lineup START/SIT Tiers ✅
- **`engine/lineup.py`** — Added `_sit_start_tier(composite_score: float) -> str` static method. Thresholds: ≥80=Definite Start, ≥65=Lean Start, ≥50=Toss-Up, ≥35=Lean Sit, else=Sit. Added `"sit_start_tier"` to per-player recommendation dict.
- **`lib/api.ts`** — Added `sit_start_tier?` to `PlayerRec` interface.
- **`app/dashboard/lineup/page.tsx`** — Added `TIER_STYLE` map (bg/color per tier). Badge renders next to player name in every recommendation card.
- Commit: `f5fe43f` (ff-bot), part of `190b9ab` (ff-bot-web)

### Track 2 — Trade Opponent Roster Panel ✅
- **`lib/api.ts`** — Added `RosterTeamPlayer`, `RosterTeam` interfaces and `getAllRosters(token, leagueId)` function (calls `/leagues/all-rosters`).
- **`app/dashboard/trade/page.tsx`** — Added `POS_COLOR` constant and full `OpponentRosterPanel` component: collapsible, lazy-loads on first open, team selector dropdown, players grouped by position with position badges, click-to-fill into "You Receive" field. Wired below evaluate button in Mode A. Mobile: full-width, collapsed by default.
- Commit: `190b9ab` (ff-bot-web)

### Track 5 — UX Polish ✅
- **`app/dashboard/schedule/page.tsx`** — Removed orphaned Sleeper fetch (result was discarded). Added animated skeleton loader (10 rows, `animate-pulse`) — replaces empty state during Sleeper data load.
- **`app/dashboard/page.tsx`** — Fixed stale link: `href="/login"` → `href="/enter"` in 401 error handler.

### Both repos pushed to main
- ff-bot commit: `f5fe43f`
- ff-bot-web commit: `190b9ab`

### Pending from this session
- [ ] **Add GitHub Secrets**: `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY` — needed if GitHub Actions ingestion ever runs
- [ ] **Add `FRONTEND_URL` to Railway env vars** — for email alert links back to app

---

## Session 24 — March 13, 2026 (Auth Overhaul + Feature Sprint)

**Goal:** Remove Supabase JWT auth entirely (Track A), fix 5 broken features (C1–C5), add Schedule page (Track D), add all-team rosters view (Track E).

### Track A — Auth Replacement ✅
Replaced Supabase JWT with simple email-in-localStorage identity. No more OTP, no invite flow.
- **`api/dependencies.py`** — `get_current_user` reads `X-User-Email` header, looks up `profiles` table. No JWT.
- **`api/routes/auth.py`** — Down to 2 endpoints: `GET /auth/lookup?email=` (public) + `GET /auth/me` (auth'd).
- **`app/enter/page.tsx`** — New entry page: email input → `/auth/lookup` → store `{email, userId}` in `localStorage["ff_user"]` → `/dashboard`.
- **`app/dashboard/league-context.tsx`** — Reads email from localStorage as `token`. Passed as `X-User-Email` on every API call.
- **`lib/api.ts`** — All `Authorization: Bearer` headers replaced with `"X-User-Email": token`.
- Old auth pages (`/login`, `/accept-invite`, `/auth/callback`, `/auth/confirm`) now redirect to `/enter`.

### Track C3 — Trade Engine Dynasty Value ✅
- **`engine/trade.py`** — `_load_player_ros_data()` now uses `player_profiles.dynasty_value` (KTC normalized 0–100) as ROS proxy when `blended_projection` is NULL (offseason). Formula: `dv/100 × 12 × remaining_weeks × age_factor`. Age factor: `-5% per year over 28`, min 0.5.

### Track C2 — Waiver Position Filters ✅
- **`app/dashboard/waiver/page.tsx`** — Position filter pills (QB/RB/WR/TE/K/DEF). Selected positions passed to `analyzeWaivers()`.

### Track C4 — Keepers Dynasty UI ✅
- **`api/routes/keepers.py`** — Brute-force combination optimizer: C(33,3)=5,456 combos, finds best 3 with max 1 per position. Score: `dynasty_value×0.6 + pts_score×0.3 + youth_bonus - age_penalty`.
- **`app/dashboard/keepers/page.tsx`** — Dynasty mode shows numbered KEEP badges (1/2/3), AI narrative, left-on-table section.

### Track C5 — League Connect Redirect ✅
- **`app/dashboard/leagues/page.tsx`** — After successful connect or team select, auto-redirects to `/dashboard` after 1.5s. Synced_at shows date+time.

### Track D — Schedule Page ✅
- **`app/dashboard/schedule/page.tsx`** — New page. Fetches all 17 weeks of matchup data directly from Sleeper (client-side, no backend). Resolves team names via users + rosters endpoints. Shows W/L/T per week, your score, opponent score, record summary cards at top, current week highlighted in blue, playoff weeks labeled.
- **`app/dashboard/layout.tsx`** — Schedule added to `NAV_BASE`.

### Track E — All-Team Rosters ✅
- **`api/routes/leagues.py`** — New `GET /leagues/all-rosters` endpoint: fetches Sleeper rosters+users in parallel (`asyncio.gather`), enriches with player names/positions/ages from `players` table (by `sleeper_id`), dynasty values from `player_profiles` (KTC normalized). Returns all teams sorted by roster_id (user's team first).
- **`app/dashboard/league/page.tsx`** — Added Standings / Power Rankings / Rosters tab bar. Rosters tab: expandable team cards grouped by position with position-colored headers and dynasty value column. Auto-expands user's own team. Lazy-loads on first tab visit.

### Both repos pushed to main
- ff-bot commits: `e1a39b8` (C3 trade + E backend)
- ff-bot-web commits: `f817342` (D schedule + nav), `6868fd0` (E rosters tab)

### Pending from this session
- [ ] **Run `005_advanced_metrics.sql` in Supabase SQL Editor** — adds advanced cols + espn_ownership table
- [ ] **Add GitHub Secrets**: `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`
- [ ] **Add Railway env vars**: `RESEND_API_KEY`, `ALERT_FROM_EMAIL`, `FRONTEND_URL`

---

## Session 23 — March 13, 2026 (Engine Audit + Auth Fixes)

**Goal:** Full read-only pipeline audit, fix broken wiring, produce DATA_FLOW.md diagram, fix two auth UX bugs.

### Engine Fixes (ff-bot)
- **`engine/waiver.py`** — Critical fix: `waiver opportunity_score` was always 0.0 because `weekly_projections.opportunity_score` is always NULL (projections.py never writes it). Now merges from `opportunity_metrics` (week N-1) after projection query. Also added `recent_news` fetch + pass-through to Claude waiver call.
- **`engine/trade.py`** — Critical fix: `get_trade_reasoning()` was called synchronously, blocking FastAPI's event loop. Wrapped in `asyncio.get_event_loop().run_in_executor(None, lambda: ...)`.
- **`db/migrations/005_advanced_metrics.sql`** — Fixed `CREATE POLICY IF NOT EXISTS` syntax error (PostgreSQL doesn't support IF NOT EXISTS on policies). Replaced with `DO $ BEGIN IF NOT EXISTS (...) THEN CREATE POLICY ... END IF; END $;`.
- **`ingestion/espn_ownership.py` + `weekly_stats.py`** — Fixed log messages referencing wrong migration file name (`migration_v4.sql` → `005_advanced_metrics.sql`).
- **`docs/DATA_FLOW.md`** — New: comprehensive 7-layer ASCII pipeline diagram covering all ingestion sources, DB tables, scoring engine, Claude prompts, API routes, and frontend pages. Includes Known Gaps table.

### Auth Fixes (ff-bot-web)
- **`app/login/page.tsx`** — Added `inviteOnly` state + `isNoAccountError()` helper. Supabase errors containing "not allowed"/"not found"/"signup"/"disabled"/"user not" now show a friendly "No account found — invite-only" UI block with link to /accept-invite, instead of raw error text.
- **`app/accept-invite/page.tsx`** — Full rewrite. Added `extractToken()` helper that parses full invite URL or raw token string. `manualInput` state shown only when no URL token. `resolvedToken = urlToken || extractToken(manualInput)`. Submit button not disabled for missing URL token. Fixes mobile invite link stripping (iMessage/SMS strip query params).

### Both repos pushed to main (Railway + Vercel auto-deploy)
Commits: `a101bc7` (ff-bot), `a15b615` (ff-bot-web)

### Action Items
- [ ] **Run `005_advanced_metrics.sql` in Supabase SQL Editor** — adds racr/wopr/air_yards/epa to opportunity_metrics, creates espn_ownership table (migration now has correct DO block syntax)
- [ ] **Test invite flow end-to-end** with fresh email after Vercel deploy

---

## Session 22 — March 13, 2026 (Data Audit + Pipeline Expansion)

**Goal:** Full audit of all data sources, fix broken pipes, add new free data sources, verify all wiring end to end.

### Root Cause Found
"Blended projection model (Sleeper 65% / FantasyPros 35%)" was a lie — FantasyPros was a stub with no real HTTP call. The model was running on Sleeper data only. News was being ingested but Claude never saw it. Weekly stats were frozen at 2022–2025 historical load.

### New Files Added (backend)
- `ingestion/weekly_stats.py` — nfl-data-py weekly pull every Tuesday after games; writes actual stats + snap counts + air yards to `weekly_stats` + `opportunity_metrics`
- `ingestion/espn_ownership.py` — ESPN unofficial Fantasy API for pct_owned, pct_started, trending adds/drops; writes to new `espn_ownership` table
- `ingestion/sleeper_news.py` — Sleeper `/v1/players/nfl` for practice status + depth chart notes; merges into `news_items`
- `engine/learning.py` — weekly MAE comparison (projections vs actuals); auto-adjusts `source_weights` table; self-improving model

### Files Fixed (backend)
- `ingestion/projections.py` — FantasyPros scraping is now REAL (requests + BeautifulSoup); 65/35 blend is live
- `utils/agent_context.py` — `news_items` now injected into every Claude call (last 5 days, roster players only)
- `engine/claude.py` — `recent_news` block added to all prompt templates (lineup, waiver, trade, war room, briefing)

### GitHub Actions Wiring
- `daily_ingestion.yml` — 6am ET daily: injuries, sleeper_news, projections, vegas, weather, news (RSS), espn_ownership
- `weekly_learning.yml` — Tuesday 10am ET: weekly_stats pull + learning engine weight update

### Database
- Ran `db/migrations/005_advanced_metrics.sql` — adds `racr`, `wopr`, `air_yards`, `epa` to `opportunity_metrics`; creates `espn_ownership` table with RLS

### Auth Issues Found (✅ FIXED in Session 23)
- **Bug 1:** Friend (willsbrody03@gmail.com) hit login page directly → "Error sending magic link email" → Fixed: shows friendly invite-only UI with link to /accept-invite
- **Bug 2:** Invite page showed "Invalid invite link — no token found" — invite URL token was stripped when forwarded on mobile → Fixed: manual paste input + extractToken() helper

### Action Items (still open)
- [ ] **Set GitHub Actions secrets** in `github.com/teel23/fantasy-football-bot` → Settings → Secrets:
  - `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`
  - All values in `.env.local`

---

## Session 21 — March 12, 2026 (Bug Fix Sprint)

**Root cause of all data bugs identified:**
`curl https://api.sleeper.app/v1/state/nfl` returns `{"season":"2026","week":0,"season_type":"off","previous_season":"2025"}` during the NFL offseason. This single fact caused every data-related bug.

**Fixes shipped:**

### Offseason API handling (`league-context.tsx`)
- Use `previous_season` for data queries during offseason (`season_type === "off" || "pre"`)
- Clamp `nflWeek` to `Math.max(1, week)` — Sleeper returns `week=0` during offseason
- Result: `nflSeason=2025`, `nflWeek=17` instead of broken `2026` / `0`

### Loading state (`league-context.tsx` + all dashboard pages)
- Added `isLoadingLeagues: boolean` to LeagueContext
- All 6 pages (home, roster, lineup, waiver, trade, war-room) show skeleton loading while `isLoadingLeagues=true`
- Prevents "Connect a Sleeper league" flash during Railway cold start (~25s)

### Home page roster fallback (`api/routes/home.py`)
- If no roster found for current season, falls back to `season - 1`
- Fixes "roster not synced" when `nflSeason=2026` but rosters stored under 2025

### Auth / OTP login (`app/login/page.tsx`)
- Fixed `maxLength=6` → `maxLength=8` (Supabase sends 8-digit codes, not 6)
- Added 60-second cooldown on Resend button
- Added warning banner after first resend: "Only the most recent code works"
- Better error message on failed OTP
- Auto-focus OTP input after email sent

### War Room 400 error (`api/routes/war_room.py`)
- Removed `{"role": "assistant", "content": "{"}` prefill — not supported by `claude-sonnet-4-6`
- Fixed: conversation must end with user message

### Event loop blocking — all AI routes (`api/routes/war_room.py`, `trade.py`, `keepers.py`, `chat.py`, `engine/lineup.py`, `engine/waiver.py`)
- All synchronous `anthropic.Anthropic().messages.create()` calls were blocking FastAPI's event loop
- Wrapped all 6 in `asyncio.get_event_loop().run_in_executor(None, fn)`
- Fix: AI calls now run in thread pool — other requests are not blocked

### Power Rankings (`api/routes/leagues.py`)
- Added dynasty_value fallback chain: `weekly_projections` → `weekly_stats` → `player_profiles.dynasty_value`
- Offseason form check: use weeks 15-17 when `week <= 1` (was checking weeks 0, -1, -2)
- Fixed SCORE=20 for all teams (was caused by empty data → `strength=0` → `0*0.6 + 50*0.4 = 20`)

### Dynasty Capital (`api/routes/leagues.py`)
- Moved Claude call to `run_in_executor` with 12s timeout
- Was stuck loading because sync Claude call blocked event loop for 15-30s

### Railway cold start (`dashboard/layout.tsx`)
- Added warmup ping `fetch(`${API_URL}/`).catch(() => {})` on mount
- Fires before auth check — Railway has time to wake up while auth is processing

---

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
🟢 Core app working — ~87% complete (Session 26 complete Mar 15, 2026)

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

### Infrastructure
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`) — needed for GitHub Actions ingestion + email workflows
- [ ] **Add `FRONTEND_URL` to Railway env vars** — email alert links will be broken without this
- [ ] **Re-run `python -m ingestion.historical`** when nflverse publishes `player_stats_2025.parquet` (currently 404)
- [ ] Add VERCEL_TOKEN to Vercel env vars → unlocks Deployments page
- [ ] Add option to delete connected Sleeper leagues
- [ ] Add `/admin/ingest-status` endpoint showing last run timestamps for all ingestion jobs
- [ ] Investigate Sleeper live game data API for 2026 in-season scoring (live scores, player points mid-game)
- [ ] Set up cron to auto-refresh `dynasty_value` from KTC periodically (currently static seed)

### Features — In Progress
- [ ] **Standings page** — `/dashboard/standings` with W/L/PF/PA/streak, user’s team highlighted
- [ ] **Player comparison tool** — `/dashboard/compare`, side-by-side composite score + dynasty value + projections
- [ ] **Draft board page** — `/dashboard/draft` show owned picks, estimated value, trade scenarios
- [ ] **Settings page** — `/dashboard/settings` display name, notification prefs, disconnect league
- [ ] **Email alerts** — wire `RESEND_API_KEY` to actually send emails (infra + templates exist, send logic incomplete)
- [ ] **C1 Lineup: tier filter pills** — filter lineup view to show only "Toss-Up" or specific tier
- [ ] **C3 Trade: multi-select from panel** — clicking in opponent panel should append to You Receive, not replace
- [ ] **RACR/WOPR scoring** — incorporate `racr` + `wopr` (now in opportunity_metrics) into receiver composite score
- [ ] **Notifications page polish** — mark-all-read, empty state illustration, group by date
- [ ] **Dynasty capital drill-down** — show per-player breakdown of how dynasty capital score is calculated
- [ ] **Dynasty value refresh script** — ingest latest KTC values into player_profiles.dynasty_value
- [ ] **Taxi squad recommender** — who to promote from taxi to roster
- [ ] Previous draft analysis — ADP vs actual production grades (boom/sleeper/bust)
- [ ] Mobile UX audit pass — 390px viewport, fix overflow/truncation on all 8 pages

### Features — Complete ✅
- [x] Full codebase audit → BROKEN_FEATURES.md + STATUS_REPORT_S27.md (Session 27)
- [x] 10 bug fixes — players.py dynasty profile, waiver drop candidates, matchup pillar weight, health.py column, viz.py current_week, agent.py current_week, schedule proxy, lineup_reminder, home.py db conn, offseason calendar year (Session 27)
- [x] Schedule page proxied through backend (no more 19 direct Sleeper calls from browser) (Session 27)
- [x] Matchup pillar weight (12%) redistributed to active pillars — no more phantom constant in scores (Session 27)
- [x] Open registration — anyone can join with email, no invite required (Session 26)
- [x] POST /agent/chat — 11-source real data context + inline markdown + localStorage history (Session 26)
- [x] usePageCache hook + SyncButton on all 8 data pages (Session 26)
- [x] Cold-start banner in layout (pings /health/ping) (Session 26)
- [x] 5 GET /viz/* chart-ready endpoints (Session 26)
- [x] 7 bug fixes — opp_score formula, home.py run_in_executor, week=1 clamp, logging (Session 26)
- [x] Lineup START/SIT tier badges — 5-tier color system (Session 25)
- [x] Trade opponent roster panel — lazy-load, grouped by position, click-to-fill (Session 25)
- [x] Schedule skeleton loader + orphaned fetch cleanup (Session 25)
- [x] 005_advanced_metrics.sql — racr/wopr/air_yards/epa + espn_ownership table (Session 25)
- [x] Schedule tab — week-by-week W/L/T (Session 24)
- [x] League — see other teams’ rosters in Rosters tab (Session 24)
- [x] Keepers analysis — dynasty rules (keep 3, max 1 per position, no pick cost) (Session 24)
- [x] Waiver wire — position filter pills (Session 24)
- [x] Settings — synced_at timestamp shown per league (Session 24)
- [x] Auth — removed Supabase JWT, simple email entry (Session 24)
- [x] Trade evaluator — dynasty_value fallback for offseason (Session 24)
- [x] War room error fixed — prefill removed (Session 21)
- [x] Roster page — slot groupings, dynasty value + injury badges (Session 20)
- [x] Drop suggestions paired with waiver pickups (Session 20)
- [x] Dynasty offseason calendar card on home dashboard (Session 20)
- [x] Connect to Supabase for persistence
- [x] Deploy web interface via Vercel
