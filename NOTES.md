# Fantasy Football AI Bot — Project Notes
> Reference for jumping back into this project.
> Last updated: March 24, 2026 (Session 39 — Mobile Audit, News Feed, Stats-Driven AI)

## Session 39 — March 24, 2026

### Backend Changes ✅ (`ff-bot` — commit 8b11e47)

**Team News Endpoint** (`api/routes/lineup.py`)
- New `GET /lineup/team-news?league_id=&season=` endpoint
- Fetches all roster players from `league_rosters`, queries `news_items` (last 300) and `injuries` tables
- Merges both sources: injury rows get priority, news_items filtered to exclude duplicates
- Returns `{ news: [...], player_count: N }` — top 50 items sorted newest-first
- No N+1 queries: single bulk fetch for news + single for injuries, matched in Python

**Player Profile News** (`api/routes/players.py`)
- `GET /players/{player_name}` now queries `news_items` by player_name (case-insensitive ilike)
- Falls back to live Sleeper API `/v1/players/nfl` if no stored news (off-season)
- Returns `news_items: []` field in response for ProfileDrawer to render

**Stats-Driven Claude Prompts** (`engine/claude.py`)
- `LINEUP_SYSTEM_PROMPT`: requires leading with `"Projected {pts}pts ({grade} matchup)"`, bans vague phrases, requires floor/ceiling in close calls, mandates injury status exact wording
- `WAIVER_SYSTEM_PROMPT`: requires `"Opp score X/100, proj Y pts"` lead, specific % metrics (target share, snap%, carries/gm), specific FAAB $ bid ranges
- `TRADE_SYSTEM_PROMPT`: requires ROS projected pts for both sides, explicit `+/-N` value delta, positional ROS rank numbers; bans hedging phrases

### Frontend Changes ✅ (`ff-bot-web` — commit 1b062e1)

**Mobile/iPhone Audit** (`app/globals.css`, `app/dashboard/layout.tsx`)
- `html` + `body`: `overflow-x: hidden` to eliminate horizontal scroll
- All interactive elements: `touch-action: manipulation`, `-webkit-tap-highlight-color: transparent`
- Safe area insets: bottom nav, main content padding, mobile drawer all use `env(safe-area-inset-bottom)`
- Bottom nav tap targets: `minHeight: 44`
- Notification dropdown: `max-w-[calc(100vw-2rem)]` to prevent off-screen overflow
- `.label-caps` font fixed to 11px on ≤640px screens

**PlayerProfileDrawer** (`components/PlayerProfileDrawer.tsx`)
- Body scroll lock while drawer is open (useEffect adds `overflow: hidden` to document.body)
- Close button enlarged to 44×44px with background color for tap target compliance
- Action buttons minimum height 48px
- Recent games table: CSS grid → flex to prevent mobile horizontal overflow
- **Recent News section**: renders `profile.news_items`, injury items highlighted red, time-ago display, up to 3 items; graceful "No recent news" fallback
- Off-season empty stats: shows informational message instead of blank table

**Roster Page Team News Feed** (`app/dashboard/roster/page.tsx`)
- "Team News" collapsible section (open by default) above player list
- Async fetch via `getTeamNews()` on league/week change
- Skeleton shimmer (3 rows) while loading
- Each news item: position badge, clickable player name (opens profile drawer), team chip, time-ago, headline (red border for injuries), summary text
- Empty state message when no news found

**API Client** (`lib/api.ts`)
- `PlayerNewsItem` interface: `{ title, summary, published, source }`
- `TeamNewsItem` interface: `{ player_id, player_name, position, team, slot_type, headline, summary, published, source }`
- `news_items?: PlayerNewsItem[]` added to `PlayerProfile` interface
- `getTeamNews(leagueId, season)` function: `GET /lineup/team-news`

**Dashboard Home** (`app/dashboard/page.tsx`)
- "AI Offseason Priorities" → "Dynasty Priorities"
- "Projections-based" badge added to Week Outlook card header

**Data Connections Audit** (across all pages)
- Audited all 6 live data sources: Sleeper, FantasyPros, ESPN, Vegas (Odds API), Weather (OpenWeather), Supabase
- Added error states and fallback UI for each: network failure, missing credentials, empty response
- Lineup, Waiver Wire, Trade Analyzer, War Room, Weekly Briefing: replaced vague AI verbiage with stats-driven copy (projected pts, snap%, target share, carry counts, matchup scores)

### Build Status ✅
- `npx tsc --noEmit`: 0 errors
- `npm run build`: 26/26 pages compiled, exit 0

### Deploy Status ✅
- `ff-bot-web` pushed to `origin/main` → Vercel auto-deploying
- `ff-bot` pushed to `origin/main` → Railway auto-deploying

### Open Items
- [ ] Add VERCEL_TOKEN to Vercel env vars (vercel.com/account/tokens) → unlocks Deployments page
- [ ] Build weighting engine v1
- [ ] Define data sources (free tier: ESPN API, Sleeper API, etc.)
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
- [ ] Roster page — show slot groupings (Starters / Bench / Taxi / IR) instead of position groups
- [ ] Dynasty offseason calendar card on home dashboard
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL`) — needed for GitHub Actions email workflows
- [ ] Drop suggestions paired with waiver pickups ("Pick up X → Drop Y")
- [ ] Roster page — show dynasty value + injury badge inline per player
- [ ] **League Scan match_score → visual bar** — the compatibility score column renders as a raw number (e.g. `5`) with no context. Replace with a thin accent-colored progress bar proportional to max match_score in the list.
- [ ] **Render per-player breakdown from API response** — backend returns `giving_evaluation.players[]` and `receiving_evaluation.players[]` with `ros_projected_points`, `dynasty_value`, `age`, `blended_projection` per player. Frontend `TradeResult` interface only reads `verdict/reasoning/recommendation` — all per-player data is silently dropped. Add a small breakdown table under the verdict showing what drove the decision.
- [ ] **Fix PlayerTag `onRemove` no-op in PlayerSearch** — `onRemove={() => {}}` at line 85 in `trade/page.tsx` means removing a player from the search selection does nothing. Removal only works from the secondary list below.
- [ ] **League Scan result cached in localStorage** — cache scan result with timestamp, load instantly on re-visit, "Refresh" button to re-run
- [ ] **Agent chat export** — "Copy conversation" button that puts full Q&A on clipboard as plain text
- [ ] **Send `roster_needs` from frontend** — backend `TradeEngine` applies a +10% bonus for receiving players that match `roster_needs`, but the frontend `handleEvaluate()` call never passes this field. Pull weak positions from the existing roster data already loaded.
- [ ] **Keeper cost efficiency sort** — sort keepers by `dynasty_value / keeper_cost` ratio, surfacing best value keeps first
- [ ] **Per-player opportunity score trend in drawer** — show how opp score moved over last 4 weeks using `opportunity_metrics` table
- [ ] **Trade suggested offer text → player chips** — `partner.what_to_offer` text auto-searches your roster and shows matching players as quick-add chips
- [ ] **In-app session changelog page** — `/dashboard/changelog` renders the most recent NOTES.md session entry as formatted HTML
- [ ] **Draft round/pick estimator** — show estimated overall pick number (e.g., "Your 2026 2nd ≈ Overall #16–20") on rookie draft page
- [ ] **Schedule opponent difficulty color coding** — opponent name gets faint red/green/neutral bg based on their power ranking
- [ ] **Standings page** — `/dashboard/standings` with W/L/PF/PA/streak, user’s team highlighted
- [ ] **Roster slot utilization badge on home** — single-line "10/10 starters set · 3 taxi open" strip so you see roster completeness at a glance
- [ ] **Trade offseason mode note upgrade** — replace generic "Trading is open" banner with "Evaluating via dynasty value — projection data unavailable until Week 1"
- [ ] **Branded 404 page** — unknown routes show a branded error page with links back to Dashboard instead of default Next.js blank
- [ ] **War room magic number tracker** — how many wins to clinch playoff spot shown as filled/unfilled circles (earned vs needed)
- [ ] **Supabase row count in health check** — add `data_health` to `/health/ping` response showing row counts for projections/injuries/news tables
- [ ] **Home page "Top priority today" card** — single top-of-page action card surfacing the #1 most time-sensitive item (e.g., "Waiver closes in 6h — RB need")
- [ ] **Stash/cut recommender** — when picking up a waiver player, suggest who to cut from taxi/bench to make room. Engine already has dynasty_value + age for all players.
- [ ] Mobile UX audit pass — 390px viewport, fix overflow/truncation on all 8 pages
- [ ] **C1 Lineup: tier filter pills** — filter lineup view to show only "Toss-Up" or specific tier
- [ ] Previous draft analysis — ADP vs actual production grades (boom/sleeper/bust)
- [ ] **RACR/WOPR scoring** — incorporate `racr` + `wopr` (now in opportunity_metrics) into receiver composite score
- [ ] **Inline news on lineup page** — `news_items` table has last 48h Sleeper news. Show a snippet next to player name on lineup page if recent news exists.
- [ ] **Taxi squad recommender** — who to promote from taxi to roster
- [ ] **Positional DV breakdown in League Rosters tab** — each team card already has dynasty values loaded. Add a QB/RB/WR/TE DV sum row per team card.
- [ ] **Remove Lineup tab from nav** — redundant with the START/SIT functionality that should live on the Roster page. Remove from sidebar and bottom nav.
- [ ] **Notifications page polish** — mark-all-read, empty state illustration, group by date
- [ ] **Dynasty value refresh script** — ingest latest KTC values into player_profiles.dynasty_value
- [ ] Investigate Sleeper live game data API for 2026 in-season scoring (live scores, player points mid-game)
- [ ] **Mobile PWA manifest** — `manifest.json` + app icons for "Add to Home Screen" on iOS/Android. No framework changes needed, just static files.
- [ ] **Playoff bracket preview** — for teams in contention, show simulated bracket for weeks 15-17 based on current standings and seeding rules.
- [ ] **Sticky tier headers on lineup scroll** — "Definite Start", "Lean Start" etc. become sticky on mobile scroll so you always know which tier you're in
- [ ] **Positional color bottom border on player rows** — 2px bottom border in POS_COLOR per player row; subtle visual rhythm
- [ ] **Power ranking trend chart** — power rankings are computed weekly but no historical movement chart exists. Visualize your rise/fall over the season in the War Room or League page.
- [ ] **Trade history log** — persist evaluated trades to Supabase so you can review past analyses and see if Claude's verdicts were right.
- [ ] **Bye week visualizer** — show which weeks starters have byes on a small calendar grid. Pure client-side from roster + schedule data already loaded.
- [ ] **FAAB bid amount recommender** — $1000 FAAB dynasty league + waiver engine scores players. Add a suggested bid amount based on player opportunity score + remaining budget.
- [ ] **War room peak window timeline — be harsher** — current logic turns too many players green (prime). Tighten the rules: prime window should only apply to players with DV ≥ 60 AND age 23-27. Everyone else should be yellow (developing) or red (declining/aging). Add a "Superstars" callout section highlighting your top 3 players by DV with their value score prominent.
- [ ] **Home page load speed** — page takes too long to load. Implement `usePageCache` with localStorage stale-while-fresh caching so cached data renders immediately and refreshes in background. Consider parallelizing backend calls if currently sequential.
- [ ] **League tab — clickable team cards** — clicking any team in Standings, Power Rankings, or Rosters tab should open a dedicated team detail view showing that team's full roster, record, PF/PA, and dynasty value breakdown.
- [ ] **Settings — add delete league option** — add a "Remove league" button per connected league in the Settings page. Confirm dialog before deleting from `user_leagues`.
- [ ] **Roster export button** — "Copy roster" outputs full 33-player list (Name · POS · Team · DV) as plain text for Discord/Slack
- [ ] Add VERCEL_TOKEN to Vercel env vars → unlocks Deployments page
- [ ] Set up cron to auto-refresh `dynasty_value` from KTC periodically (currently static seed)
- [ ] **Draft pick value calculator** — "What is a 2026 1st worth vs a 2027 2nd?" Uses existing dynasty_value data. Standalone tool page.
- [ ] **League/team names — use team name not username** — everywhere a league or team is referenced in the UI (nav, top bar, league context, dropdowns), use `team_name` first, falling back to `league_name`. Never show the Sleeper username (e.g., "ctteel") as the primary identifier.
- [ ] **Global style overhaul** — the overall visual design needs a refresh. Log this as a dedicated session to define a new direction before touching code.
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`) — needed for GitHub Actions ingestion + email workflows
- [ ] **Learning engine dashboard** — `engine/learning.py` auto-adjusts `source_weights` each week but there's no UI showing current weights or MAE history. Add a section in War Room.
- [ ] Add `/admin/ingest-status` endpoint showing last run timestamps for all ingestion jobs
- [ ] **Score differential color gradient in schedule** — `+/-` color scales from bright green → muted → bright red rather than binary green/red
- [ ] **Playoff odds meter on war room** — needle-gauge SVG showing estimated playoff probability (0–100%) based on record + remaining schedule
- [ ] **Player comparison tool** — `/dashboard/compare`, side-by-side composite score + dynasty value + projections
- [ ] **Dynasty capital AI narrative collapsed by default** — collapse AI analysis behind `<details>` like reasoning blocks elsewhere
- [ ] **Compact/full density toggle on roster page** — switch shrinks `PlayerInlineRow` from 56px → 36px to show all 33 players without scrolling
- [ ] **Trade scan auto-fill target players** — clicking "Start Trade →" from League Scan should pre-fill "You Receive" with Claude's named target players, not just show a banner
- [ ] **Trade dynasty vs redraft toggle** — toggle switches evaluation between dynasty mode (DV-heavy) and redraft mode (projection-heavy)
- [ ] **Settings page** — `/dashboard/settings` display name, notification prefs, disconnect league
- [ ] **Last sync timestamp per data type** — SyncButton tooltip shows "Projections: 6h ago · Injuries: 2h ago · News: 45m ago" granularity
- [ ] **Player profile start % stat** — `espn_ownership.pct_started` exists in DB but not shown in `PlayerProfileDrawer`. Add as a chip.
- [ ] **Dynasty capital drill-down** — show per-player breakdown of how dynasty capital score is calculated
- [ ] **Roster page — Sleeper-style layout** — completely rework to match Sleeper's visual format: labeled starter slots (QB, RB1, RB2, WR1, WR2, TE, FLEX, etc.) at the top, then bench below clearly separated, each player color-coded by position, with START/SIT options visible per player on this page. Currently all players look the same with no slot labels.
- [ ] **Waiver wire — show all available players in the league** — currently only shows AI-ranked suggestions. Add a full "Free Agency" section showing all players not on a roster in the league (call Sleeper `/league/{id}/rosters` to derive available players). Add a Sync button to refresh on demand. Rename tab to "Waivers / Free Agency".
- [ ] **Schedule page — fix broken state** — page is not loading/working. Diagnose root cause (likely the backend proxy endpoint from S27 or auth header issue) and restore to working state.
- [ ] **Dynasty Capital — 4 rounds per year** — Chimp Dynasty rookie draft has 4 rounds, not 3. Update the pick timeline grid to show rounds 1-4. Also move AI analysis to the top of the page and expand it — current output is too generic/programmatic.
- [ ] **Conditional nav by league type** — Rookie Draft tab only visible for dynasty leagues. Draft tab only visible for redraft leagues. Keepers tab only visible for keeper leagues. Gate by `activeLeague.league_type` in `layout.tsx`.
- [ ] **Weather alerts on lineup page** — `ingestion/weather.py` runs daily but weather data never appears in lineup recommendations. Show a weather chip inline on affected players.
- [ ] **Positional need badge on waiver** — waiver engine already knows your roster composition. Surface an "Addresses Need" flag on pickups that match a weak position.
- [ ] **League activity feed** — pull Sleeper transaction history (adds/drops/trades across all 12 teams). Sleeper has `GET /league/{id}/transactions/{round}`.
- [ ] **Roster age distribution chart** — bar or donut showing roster by age bucket (≤23 / 24-26 / 27-29 / 30+). Add to War Room alongside radar chart.
- [ ] **Animated DV balance meter fill** — trade balance bar animates from center outward using CSS `width` transition (0 → final, 400ms) on render
- [ ] **Injury dot pulse for active injuries** — Out/IR dots have continuous slow pulse animation; Questionable dots are static yellow
- [ ] **Bulk player bio sync endpoint** — `POST /players/sync-bios` refreshes age/team/injury for all 33 roster players in one Sleeper batch, triggered from Settings
- [ ] **Waiver sort button filled active state** — active sort = filled accent pill (white text); inactive = ghost. Currently too subtle to read.
- [ ] **"Already rostered" warning in trade search** — if searched player is on another team's roster, show "Rostered by [Team]" in the dropdown
- [ ] **Lineup "Optimal vs Actual" projected gap** — show optimal lineup score vs what you have set: "Leaving 4.3 pts on the bench"
- [ ] **Player profile "comparable players"** — inside `PlayerProfileDrawer`, show 2-3 players with similar age, position, and dynasty value as a comp reference.
- [ ] **Position scarcity overlay on draft page** — sticky strip showing how many QBs/RBs/WRs/TEs remain in top 50 available while viewing draft board
- [ ] Add option to delete connected Sleeper leagues
- [ ] **Notification preferences per alert type** — Settings toggle for each alert type (injury, waiver close, lineup reminder) independently
- [ ] **ESPN ownership trends page** — `espn_ownership` table is ingested daily but never surfaced in the UI. Add a section showing pct_owned, pct_started, trending adds/drops per player.
- [ ] **Add `FRONTEND_URL` to Railway env vars** — email alert links will be broken without this
- [ ] **Show `confidence_pct` and `value_delta` in verdict card** — both are returned by `/trade/evaluate` but not in the `TradeResult` type and never rendered. `value_delta` (e.g. `+14.3 pts`) and `confidence_pct` (e.g. `80%`) should appear in the verdict card.
- [ ] **Dynasty value sparkline in PlayerProfileDrawer** — small 8-week KTC value trend line inside the drawer
- [ ] **Left-to-right shimmer on skeleton cards** — replace `animate-pulse` opacity flash with a directional sweep shimmer
- [ ] **Email alerts** — wire `RESEND_API_KEY` to actually send emails (infra + templates exist, send logic incomplete)
- [ ] **C3 Trade: multi-select from panel** — clicking in opponent panel should append to You Receive, not replace
- [ ] **Trade page search overhaul** — current search requires exact name match. Fix: (1) search by first name, last name, or any prefix using `ILIKE` or fuzzy match on backend; (2) show dynasty value / projected score next to each player in search results AND in the selected player list; (3) add AI analysis section at top of page before you even run a trade; (4) add "My Team" selector panel mirroring the opponent roster panel so you can click-to-fill "You Give" from your own roster.
- [ ] **Dynasty capital dot sizing by round** — R1 circle = 24px, R2 = 20px, R3 = 16px. Communicates pick value visually without labels.
- [ ] **In-app changelog** — "What's New" slide-up shown once per deploy, driven by static JSON. No backend needed.
- [ ] **"Last time you faced this opponent" on schedule** — show last matchup result above each week row
- [ ] **Standings row highlight pulse on load** — user's own row flashes a single accent pulse on page load to draw the eye to their position
- [ ] **Trade verdict badge entrance animation** — ACCEPT/DECLINE/NEUTRAL badge scales in from 0→1 with a spring-style CSS animation on load
- [ ] **Tab underline slide animation on mode switch** — active underline slides horizontally on trade page tab switch instead of appearing/disappearing
- [ ] **Projected final standings simulation** — use power rankings + remaining schedule to estimate final seed. Show "on track for #3 seed" type output.
- [ ] **Draft board page** — `/dashboard/draft` show owned picks, estimated value, trade scenarios
- [ ] **Re-run `python -m ingestion.historical`** when nflverse publishes `player_stats_2025.parquet` (currently 404)
- [ ] **Historical H2H record on schedule page** — expand any matchup row to show all-time head-to-head record vs that opponent. Sleeper has full matchup history.
- [ ] **Multi-league quick-switch** — if user has multiple leagues connected, a visible dropdown on every page (not buried in the leagues screen) to swap active league.
- [ ] **Player card hover tooltip with recent stats** — desktop hover on `PlayerInlineRow` slides in mini tooltip showing last 3 weeks of actual points as small bars
- [ ] **War room radar chart entrance animation** — polygon animates from center outward using SVG `stroke-dashoffset` on page load
- [ ] **Confidence arc on verdict card** — SVG semicircle arc from 0–100% replacing or augmenting the raw confidence_pct number
- [ ] **Pulsing dot on current week in schedule** — animated green pulse dot next to the active week row to visually anchor "you are here"
- [ ] **Gradient row background on waiver by opp score** — row bg shifts faint red→green based on opp score (≤4 red-dim, ≥7 green-dim); no chip needed
- [ ] **Color-blended roster health strip** — segment fills use gradient blend between healthy/injured colors rather than flat fills
- [ ] **Waiver wire claim deadline countdown** — live countdown to Tuesday FAAB deadline shown at top of waiver page when within 24 hours
- [ ] **Dynasty value history seeding script** — write script to seed monthly KTC snapshots into `dynasty_value_history` table for real trend data in drawer
- [ ] **Animated tier transitions on lineup page** — smooth CSS `max-height` transition on tier group expand/collapse instead of instant show/hide
- [x] Connect to Supabase for persistence
- [x] Deploy web interface via Vercel

## Session 38 — March 17, 2026 (25-Task Sprint)

### Backend Changes ✅
**A1 — GET /home/week-outlook** (`api/routes/home.py`)
- Claude-powered 2-3 sentence weekly briefing (matchup difficulty, key injuries, weather)
- 4-hour in-memory cache per `league_id:week:season` key — avoids re-calling Claude on every refresh
- Returns `{ outlook: string }`

**C1 — GET /lineup/drop-candidates** (`api/routes/lineup.py`)
- Fetches bench players (`slot_type='bench'`) from `league_rosters`
- Scores: `projected_pts * 0.6 + (dynasty_value/100) * 40.0`
- Bottom 3 sent to Claude for `cut | trade | hold` recommendation + one-line reason
- Returns `{ candidates: [{ player_id, player_name, position, dynasty_value, projected_pts, recommendation, reason }] }`

**E2 Backend — weekly_history in /players/{id}** (`api/routes/players.py`)
- Joins `weekly_stats` + `weekly_projections` for the player's season
- Returns last 10 weeks with actual scores as `{ week, actual_pts, projected_pts }`

**J1 — FantasyPros scraper hardening** (`ingestion/projections.py`)
- 3 retries per position with exponential backoff (2s, 4s)
- Rotating User-Agent strings (3 browser UAs, cycling per request)
- On all-retries-fail: falls back to last stored `fantasypros_projection` from `weekly_projections` table

**J2 — 006_rls_audit.sql** (`db/migrations/006_rls_audit.sql`)
- Idempotent RLS `SELECT` policies for: `players`, `weekly_stats`, `weekly_projections`, `news_items`, `schedules`, `opportunity_metrics`
- Each policy guarded by `pg_policies` check — safe to re-run
- Does NOT touch `user_leagues`, `league_rosters`, `profiles` (auth-gated by FastAPI layer)
- **Do NOT run automatically — manual Supabase SQL Editor run required**

### Frontend Changes ✅ (11 files, 865 insertions)

**A1 — Week Outlook card (dashboard/page.tsx)**
- Parallel fetch inside `InseasonLayout` via `useLeague()` token
- Skeleton while loading; full-width `md:col-span-2` card at bottom of in-season grid
- Only renders when response arrives (no flash on offseason)

**A2 — Trade pending indicator (ActionBar)**
- `ActionBar` now accepts `leagueId` + `rosterId` props
- Fetches Sleeper transactions rounds 1–2, filters `type=trade && status=pending && roster_ids includes mine`
- If pending trades found → urgent item injected at top of action list

**B1 — FAAB budget tracker (waiver/page.tsx)**
- Remaining budget stored in `ff_faab_remaining_${leagueId}` localStorage key
- Initializes from localStorage; falls back to `activeLeague.waiver_budget`
- Inline edit mode: pencil (✎) → number input → confirm (✓) → persists to localStorage
- Shows `$remaining / $total remaining`

**B2 — Waiver tier grouping**
- AI Picks tab groups `sortedPickups` into Must Add (≥7), Consider (4–6), Stash (<4) by `opportunity_score`
- Each tier has a labeled divider with color-coded line; global rank numbering preserved across tiers

**B3 — Gradient row background**
- Each waiver row has left-to-right tinted gradient (green for Must Add, amber for Consider, transparent for Stash)

**C1 — Cut Candidates collapsible panel (roster/page.tsx)**
- Fetches `GET /lineup/drop-candidates` on league/week change
- Collapsible section with count badge, skeleton, recommendation pills (red=cut, yellow=trade, green=hold)
- Player name clickable → opens `PlayerProfileDrawer`

**C2 — Copy Roster button**
- Builds plain text `POSITION Name (Team)` list, sorted starter→bench→taxi→IR
- `navigator.clipboard.writeText()` + 2s "Copied!" feedback flash

**D1 — Trade verdict SVG arc (trade/page.tsx)**
- 160×80 half-donut using `stroke-dasharray` on an SVG arc path
- Background arc + filled arc driven by `confidence_pct / 100 * π * r`
- 0.6s ease transition; % value + "confidence" label as SVG text center
- Removed old confidence pill badge (replaced by arc)

**D2 — Proportional match_score bar**
- Computes `maxMatchScore` from all teams; bar width = `match_score / max * 100%`
- Top team always fills bar; all others scale proportionally

**D3 — Trade history tab**
- Third tab "History" added to trade page
- Fetches all weeks 1–`nflWeek` from Sleeper transactions API in parallel
- Filters `type=trade`, sorted newest first; shows week badge, roster IDs, date, received players, FAAB budget

**E4 — Player photo (PlayerProfileDrawer.tsx)**
- Sleeper CDN: `https://sleepercdn.com/content/nfl/players/thumb/{player_id}.jpg`
- 48×48 rounded circle; `onError` → colored fallback circle with position initial
- Skeleton circle during loading; `imgError` resets on `playerId` change

**E2 — Weekly history bar chart**
- Added `weekly_history` field to `PlayerProfile` interface in `lib/api.ts`
- Bar chart after Recent Games: last 10 weeks, proportional height (max 64px)
- Filled bars (actual): green ≥15, yellow ≥8, red <8
- Outline bars (projected): muted border, 0.5 opacity
- Week numbers as tiny labels below each column

**E3 — PlayerInlineRow hover tooltip (PlayerInlineRow.tsx)**
- `hovered` state + `onMouseEnter`/`onMouseLeave` on inner container
- Tooltip appears above row (`bottom-full mb-1`), `hidden sm:block` (desktop only)
- Shows chip label + value in compact dark bubble with border

**F1 — Pulsing dot on current week (schedule/page.tsx)**
- Replaced `◈` symbol with 8×8 animated green dot
- `@keyframes pulse-dot` injected via `<style>` tag at top of return (scale 1→1.5, opacity fade)

**G1 — Collapsible tier groups (lineup/page.tsx)**
- `collapsedTiers: Set<string>` state + `toggleTier()` function
- Each tier header wrapped in clickable `flex` row with `▶`/`▼` icon
- Content div uses `maxHeight: isCollapsed ? 0 : 2000` + `0.3s ease` transition

**H1 — Standings W/L sparkline (league/page.tsx)**
- Fetches all matchups weeks 1–`nflWeek` from Sleeper in parallel
- Groups by `matchup_id`, resolves W/L/T per roster per week
- `SparkLine` component: up to 8 tiny 6×6px squares (green=W, red=L, yellow=T)
- Renders below team name in standings rows

**I1 — Radar chart entrance animation (war-room/page.tsx)**
- `animated` state (false → true after 50ms) drives `strokeDasharray`
- Animates from `0 {perimeter}` → `{perimeter} 0` with 0.6s ease transition
- Fill also animates from transparent → blue simultaneously

**K1 — Offseason calendar phase CTA (dashboard/page.tsx)**
- `PHASE_CTA` map: phase name → `{ label, href }` for 7 common phases
- Active phase renders a styled `<Link>` button below the tips list
- Phase matched by case-insensitive `includes` check

**K2 — Empty state SVG illustrations (dashboard/page.tsx)**
- 4 empty states replaced with centered SVG icon + concise text:
  - Injury Report: shield + checkmark (green)
  - Waiver Alerts: pulse waveform (muted)
  - League News: info circle (muted)
  - Current Matchup: four-squares grid (muted)

### Commits
- Backend: `de7df62`
- Frontend: `6e2eea9`

### Roadmap Items Completed This Session
From the 50-item roadmap (Session 36): items 7, 11, 13, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 29, 30, 32, 33, 36, 37, 46, 48, 49 — 24 of 50 items done.

### Open Items (Post Session 38)
- [ ] Run `006_rls_audit.sql` manually in Supabase SQL Editor
- [ ] Test drop-candidates endpoint live with Chimp Dynasty (bench scoring + Claude reasoning)
- [ ] Test FAAB tracker on a FAAB league (Chimp Dynasty is WAIVERS — need a second league for full test)
- [ ] Trade history tab shows player IDs not names (Sleeper transactions don't include player names — would need a player ID→name lookup map, or call `/players/nfl` once)
- [ ] Radar chart perimeter calculation uses `useState` but imports need to include `useState`+`useEffect` — verify both are in war-room imports after Session 38

---

## Session 37 — March 17, 2026 (10-Feature Sprint)

### Changes Shipped ✅

**Task 1 — Railway Cold Start UX**
- Trigger at 3s (was 5s), text "Server is waking up — first load takes ~25 seconds. Hang tight."
- Animated progress bar 0→100% over 25s via CSS `@keyframes coldstart-fill`
- Auto-dismiss when health/ping responds; manual X button

**Task 2 — Nav Surface Area Reduction**
- Removed `viz` and `compare` from NAV_ADVANCED and mobile menu drawer (routes kept)
- Added "Analytics" tab to league/page.tsx → Link to `/dashboard/viz?league_id=...`

**Task 3 — Live Matchup Score**
- `getLiveMatchup(leagueId, week, myRosterId)` added to `lib/api.ts` (calls Sleeper directly)
- InseasonLayout polls every 60s on Thu/Sun/Mon when `nflSeason >= 2026 && nflWeek > 0`
- Shows live points + green LIVE badge on matchup card when polling active

**Task 4 — Waiver Deadline Countdown**
- Banner shows within 36h of Tuesday 8pm ET, counts down every minute
- Dismissable X; auto-hides after deadline passes

**Task 5 — Position Filter Persist on Waiver**
- `availPos` initialized from `localStorage['ff_waiver_pos_filter']`; written on every change

**Task 6 — Sync Roster Button**
- "↻ Sync" button added next to SyncButton on roster page
- Calls `syncLeague(token, activeLeague.id)` then re-fetches roster
- Shows spinner → "Synced!" / "Sync failed" inline for 2s

**Task 7 — Lineup Lock Warning**
- Backend: `game_time` (ISO ET string from `schedules.gameday + gametime`) added to each recommendation
- Frontend: `parseETDateTime()` converts ET→UTC; checks starters within 2h; amber banner with countdown; updates every minute

**Task 8 — Multi-League Quick-Switch (mobile)**
- Compact league name + chevron in mobile header bar when `leagues.length > 1`
- Dropdown lists all leagues with type badge; clicking calls `setActiveLeague`

**Task 9 — Trade Player Search**
- PlayerSearch result rows now show "Free agent" label when `!p.on_roster` (was nothing)

**Task 10 — League-Type Hardcoding**
- `keepers.py`: `num_keepers` and `max_per_pos` read from `user_leagues.settings`; defaults 3/1
- `rookie-draft` nav item: `leagueTypes: ['dynasty', 'keeper']`

### Commits
- Frontend: `e5e2b9d`
- Backend: `4ed5b27`

---

## Session 36 — March 17, 2026 (Project Report + Roster Health + Roadmap)

### Changes Shipped ✅

**Roster Health Strip on Home**
- Added `RosterHealthStrip` (in-season): horizontal QB/RB/WR/TE/K bars, healthy/total counts, color by status
- Added `RosterHealthStripOffseason`: position depth count badges, red tint on positions with ≤1 player
- Both render at top of their respective layout sections before the grid
- Removed redundant "Position Depth" grid card (same data, now covered by strip)
- Frontend commit: `ac3c943`

### Decisions Made

**nflverse vs Sleeper for weekly stats:**
Sleeper is now primary. `weekly_stats` was already backfilled via Sleeper (`/v1/stats/nfl/regular/{season}/{week}`) — confirmed working for all 18 weeks of 2025. nflverse is supplemental for advanced metrics (snap_pct, air_yards, RACR, WOPR) only when published. No action needed.

**Surface area reduction plan:**
14+ nav routes are too many. Strategy: demote secondary pages off the sidebar and embed them as slide-overs or modals from the relevant primary page:
- Compare → embed in PlayerProfileDrawer (done S35) + keep as hidden route
- Dynasty Capital → accessible from Roster page sidebar panel
- War Room → accessible from Home "GM Mode" button
- Keepers / Rookie Draft → accessible from Roster page tabs
- Analytics/Viz → accessible from League page tab
- Journal → remove from nav entirely or fold into AI Agent

**Live game scores plan:**
Sleeper endpoint: `GET /v1/league/{id}/matchups/{week}` — returns `points` per player during live games.
Frontend: poll every 60s on matchup card when `nflSeason = current` and it's a game day (Thu/Sun/Mon).
Backend: no changes needed — Sleeper proxies live.
Timeline: ready to implement before 2026 season start (September).

### Slipped Items — Remediation Plans

| Item | Plan |
|---|---|
| `GET/PATCH /auth/preferences` | Add to `api/routes/auth.py`; store theme + notifications in `profiles` table; settings page reads from API not localStorage |
| GitHub Actions secrets | Manual: go to github.com/teel23/fantasy-football-bot → Settings → Secrets. Add: `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL` |
| 2025 weekly_stats | Already backfilled from Sleeper in S32. nflverse is supplemental only. No blocking action. |
| espn_ownership RLS (005_advanced_metrics.sql) | Verify in Supabase dashboard → Table Editor. If `espn_ownership` table exists, migration ran. If not, re-run the SQL from S22 NOTES. |
| Keeper logic hardcoded | Read `num_keepers` + per-position limits from `user_leagues.settings` JSON; remove hardcoded 3/1-per-pos |

### Railway Cold Start Fix Plan
- Option A (quick): Replace blank loading state with a "Waking up the server..." banner + animated progress bar counting 0→100% over 25s. Shows only when API call takes >3s.
- Option B (real fix): Upgrade Railway to Hobby plan ($5/mo) — removes auto-sleep entirely.
- Implement A now; recommend B to user.

### Multi-League / League-Type Hardcoding Fixes
- `num_keepers` read from `user_leagues.settings->>'num_keepers'` not hardcoded 3
- Keeper per-position limit: read from settings, default 1 if not set
- `dynasty_value` labels: show only when `league_type === 'dynasty'`
- Dynasty Capital / Rookie Draft pages: gate with `leagueTypes: ['dynasty']` in NAV_ADVANCED
- In-season home already gates on `data.mode`; extend this pattern to all advanced features

---

## 50-Item Roadmap (Session 36)

### High Impact — Core Experience (target: next sprint)
1. Railway cold start UX — "Waking up..." progress bar (count 0→25s, shows only when API slow)
2. Live matchup score — poll Sleeper `/v1/league/{id}/matchups/{week}` every 60s on game days
3. Waiver wire claim deadline countdown — live countdown at top of waiver page within 24h of Tuesday deadline
4. Position filter persist on waiver — save last-used filter to localStorage, restore on mount
5. Sync roster button on roster page — direct "Re-sync from Sleeper" CTA, no redirect to leagues
6. Lineup lock warning — banner when a starter's game starts within 2 hours
7. Trade pending indicator on home — surface pending Sleeper trades as action item on home dashboard
8. Dynasty value trend sparkline — 4-week trend line in PlayerProfileDrawer (not just up/down arrow)
9. Multi-league quick-switch dropdown — visible on every page header, not buried in leagues screen
10. Trade page player search — type-to-search across all players when adding to a trade proposal

### Medium Impact — Feature Completeness
11. In-season home AI Week Outlook card — 2-3 sentence Claude read on the week (matchup difficulty, key players, weather alerts)
12. Schedule win probability — AI-estimated win % for future matchups based on opponent's projected pts allowed
13. Waiver budget tracker — remaining FAAB ($) shown visually on waiver page header for FAAB leagues
14. Dynasty value history seeding script — monthly KTC snapshots into `dynasty_value_history` for real trend data
15. Boom/bust history chart — actual weekly fantasy pts vs projection bar chart in PlayerProfileDrawer
16. Drop suggestion on bench — "Weakest bench player" card on roster page with keep/trade/cut reasoning
17. Trade value difference visual — balance scale or +/- number on the trade verdict card
18. League Scan match_score bar — replace raw number with colored progress bar proportional to max in list
19. Roster export — "Copy Roster to Clipboard" for dynasty trade discussions
20. Offseason calendar active phase CTA — direct action button on current phase (e.g., "View Trade Targets")

### Medium Impact — Polish
21. Animated tier transitions on lineup — CSS max-height transition on tier group expand/collapse
22. Confidence arc on trade verdict — SVG semicircle arc for confidence_pct instead of raw number
23. Player card hover tooltip — desktop hover on PlayerInlineRow shows last 3 weeks of fantasy pts
24. Pulsing dot on current week in schedule — green pulse dot next to active week row
25. Gradient row background on waiver by opp score — row bg shifts faint red→green based on opportunity score
26. War room radar chart entrance animation — polygon animates from center outward via SVG stroke-dashoffset
27. Offseason depth strip red flags — flag positions with 0 starters (e.g., no K after cuts)
28. Position Depth card consolidation — DONE (removed in S36, covered by RosterHealthStrip)
29. Loading skeleton on PlayerProfileDrawer — skeleton rows while fetching instead of blank panel
30. Empty state illustrations — small SVG icon + one-line context per empty state instead of plain text

### Lower Impact — Nice to Have
31. Waiver wire "Add All Watched" — batch-submit all watchlisted players as FAAB bids
32. Trade history log — pull past accepted trades from Sleeper, display as timeline
33. Standings mini-chart on league page — wins trend over season as a line chart per team
34. Age histogram on dynasty capital page — age distribution vs league average
35. Draft grade detail view — click player in rookie draft to see boom/bust tier + usage trend
36. Custom FAAB budget input — let user set remaining budget manually for accurate bid suggestions
37. Waiver wire tier grouping — "Must Add / Consider / Stash" tiers like lineup page
38. Weekly digest email — Tuesday morning: W/L recap, top 3 waiver targets, lineup preview (deferred)
39. PWA push notifications — injury alerts within minutes of news (deferred)
40. Google OAuth — replace email-only login, reduce mobile friction (deferred)

### Infrastructure / Data
41. Set GitHub Actions secrets — SUPABASE_URL, SUPABASE_SERVICE_KEY, ODDS_API_KEY, OPENWEATHER_API_KEY, RESEND_API_KEY, ALERT_FROM_EMAIL (manual step)
42. Verify 005_advanced_metrics.sql ran — check Supabase for espn_ownership table + racr/wopr columns
43. 2025 weekly_stats — Sleeper is now primary source (backfilled S32). nflverse is supplemental only.
44. snap_counts ID format — pfr IDs can't join to players table; build crosswalk or switch source
45. auth/preferences endpoint — build GET/PATCH in auth.py; persist to profiles table
46. FantasyPros scraper rate limit protection — add backoff + retry, currently fails silently if blocked
47. Railway paid plan — $5/mo Hobby removes auto-sleep, eliminates cold start permanently
48. Supabase RLS audit — verify news_items, weekly_stats, players tables have appropriate policies
49. Player photo integration — ESPN/Sleeper CDN headshot URLs in PlayerProfileDrawer
50. Second user test — sign up with redraft league, document every broken edge case; app only tested with Chimp Dynasty

---

## Session 35 — March 17, 2026 (Roster Redesign + Quick Wins + Audit)

### P1 — Roster Page Redesign ✅

**Problem:** Old page grouped all starters under one header — no indication of which player is in which slot (QB, RB1, RB2, WR1, FLEX, etc.).

**Solution:** Slot-based layout using greedy best-fit assignment:
- Fetch `starter_slots` from `GET /lineup/slots` (e.g. `["QB","RB","RB","WR","WR","WR","TE","FLEX","SUPER_FLEX"]`)
- Sort eligible players by `dynasty_value DESC`
- Assign best available player to each slot in order (FLEX = best RB/WR/TE not yet used)
- Group slots into sections: Quarterback / Running Backs / Wide Receivers / Tight End / Flex / Utility / Kicker / Defense
- Empty slots show "— Empty —" placeholder row
- Remaining unassigned players → Bench section
- Taxi and IR as separate sections below bench
- Fallback to slot_type grouping if `starter_slots` not yet loaded
- `SlotRow` custom component (slot badge + name + DV color-coded + injury)
- Removed: sort mode toggle pills — replaced by section headers

**File:** `ff-bot-web/app/dashboard/roster/page.tsx` — full rewrite of render section

### P2 — Waiver Watchlist → Supabase ✅

**Problem:** Watchlist stored in `localStorage['ff_watchlist']` — lost on device switch.

**Solution:** New `waiver_watchlist` Supabase table + 3 endpoints:
- `GET /waiver/watchlist?league_id=` — load saved player_ids
- `POST /waiver/watchlist` — upsert player
- `DELETE /waiver/watchlist/{player_id}?league_id=` — remove

Frontend: optimistic UI toggle + background sync to Supabase.

⚠️ **MANUAL ACTION REQUIRED:** Run this migration in Supabase dashboard:
```sql
CREATE TABLE waiver_watchlist (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES profiles(id) ON DELETE CASCADE,
  league_id text NOT NULL,
  player_id text NOT NULL,
  player_name text,
  position text,
  added_at timestamptz DEFAULT now(),
  UNIQUE(user_id, league_id, player_id)
);
ALTER TABLE waiver_watchlist ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users manage own watchlist" ON waiver_watchlist
  FOR ALL USING (user_id = auth.uid());
```

**Files:** `ff-bot/api/routes/waiver.py` (3 new endpoints), `ff-bot-web/app/dashboard/waiver/page.tsx`, `ff-bot-web/lib/api.ts` (3 new functions)

### P3 — Compare from PlayerProfileDrawer ✅

- "Compare [player]" button at bottom of PlayerProfileDrawer → sets `localStorage['ff_compare_prefill']`
- Compare page reads prefill on mount, pre-populates first player slot

**Files:** `ff-bot-web/components/PlayerProfileDrawer.tsx`, `ff-bot-web/app/dashboard/compare/page.tsx`

### Audit Findings (no code changes)

Gaps found in codebase audit (92% verified):
1. `GET/PATCH /auth/preferences` — not in auth.py; settings page uses localStorage/context only. Deferred feature.
2. FantasyPros active scraping — projections.py exists but may fall back silently. Verify in Railway logs.
3. `espn_ownership` RLS migration (005_advanced_metrics.sql) — may not have been manually run in Supabase. Verify in Supabase dashboard.

### Commits
- Frontend: `beeac71` — Session 35: roster redesign, watchlist persistence, compare drawer
- Backend: `ab19d26` — Session 35: waiver watchlist endpoints

---

## Session 34 — March 17, 2026 (Bug Fixes + UX Polish)

**Context:** Executed the full S33 plan — phase 1 bug fixes + phase 2 UX polish. All 9 items completed.

### Bug Fixes ✅

**B1 — War Room generate error (backend)**
- Root cause: `build_agent_context()` called outside try/except in `war_room.py` — any DB/HTTP error surfaced as unhelpful 500
- Fix: wrapped in try/except, returns clean "Failed to load roster data: ... Try syncing your league first." message

**B2 — Schedule page broken state**
- Root cause 1: frontend ignored `data.error` field when HTTP 200 — showed empty table with no explanation
- Root cause 2: backend returned early if `sleeper_roster_id` was null (no fallback)
- Fix: added Sleeper API fallback to look up roster_id by team_name matching; frontend now catches and shows friendly error

**B3 — Keepers** — Already correct from S20 (dynasty mode, 3 players, 1 per position, no draft pick cost). Skipped.
**B4 — Team name display** — Already using `team_name ?? league_name` everywhere since S28. Skipped.

### UX Polish ✅

**U1 — Auto-load Lineup Analyzer**
- `lineup/page.tsx`: calls `runAnalysis()` on mount when `activeLeague` + `token` ready and not offseason
- One-shot guard via `hasAutoRun` state to prevent re-run on re-renders
- Button label: "Re-analyze" when results already loaded

**U2 — This Week's Actions card (home)**
- Replaced `ActionBar` single-link banner with a proper card section
- Day-of-week and hour aware: waiver deadline warnings (Tue/Wed), start/sit decisions (Thu/Fri), game-day (Sun), recap (Mon), default roster scouting
- Urgent items shown in accent color with "Now →" badge

**U3 — Floating Ask AI button**
- Fixed position FAB on all dashboard pages except `/dashboard/agent`
- Mobile: `bottom: calc(56px + 1rem)`, Desktop: `bottom: 1.5rem`, `right: 1rem/1.5rem`
- 48×48px accent circle with chat SVG icon

**U4 — Sidebar cleanup**
- `NAV_PRIMARY` (7 items always visible): Home, Roster, Waiver, Trade, Schedule, League, AI Agent
- `NAV_ADVANCED` (9 items collapsed by default): War Room, Dynasty Capital, Keepers, Rookie Draft, Analytics, Compare, Settings, Leagues, Draft
- Collapsed/expanded state persisted to `localStorage['ff_sidebar_expanded']`
- Auto-expands Advanced when current page is in that section

**U5 — Ask about this player**
- `PlayerProfileDrawer.tsx`: "Ask AI about [player]" button at bottom → sets `localStorage['ff_agent_prefill']`, closes drawer, routes to `/dashboard/agent`
- `agent/page.tsx`: reads `ff_agent_prefill` on mount, pre-fills input with "Tell me about [name]", clears key, focuses textarea

### Commits
- Frontend: `b0c835e` — Session 34: bug fixes + UX polish
- Backend: `b5cd414` — Session 34: backend bug fixes

---

## Session 33 — March 16, 2026 (UX Review — Average FF Player Perspective)

**Context:** Full UX audit from the lens of a casual-to-intermediate fantasy player. No code changes this session — review and planning only.

### What Works Well ✅
- Mobile 4-tab bar (Home / Roster / Waiver / Trade) — correct priority order
- AI Picks on Waiver — most trusted feature for casual players
- Trade League Scan — genuinely differentiated vs. competitors
- 5-tier lineup system (Definite Start → Sit) — clear and actionable
- Injury pulse animations, PlayerProfileDrawer slide-out, dark/light mode

### Problems Identified ❌

1. **Sidebar overload** — 14+ routes overwhelm casual players; advanced pages (Viz, War Room, Dynasty Capital, Keepers, Rookie Draft, Journal, Compare) should be collapsed behind a "More" section
2. **Lineup Analyzer requires button click** — should auto-load on page mount with skeleton shimmer; kill the manual trigger
3. **AI Agent is buried** — the most approachable feature for casual players sits at position 8+ in sidebar; needs a floating "Ask AI" button visible on every page
4. **No "What do I do today?" prompt** — add "This Week's Actions" card to home (lineup deadline, waiver closes, pending trades)
5. **Email-only auth** — no Google/Apple OAuth; friction on mobile signup
6. **Cold start on Railway** — 30s spinner is a bad first impression; needs a more prominent loading state with progress indication
7. **Waiver watchlist in localStorage** — lost on device switch; should persist to Supabase
8. **Compare page is orphaned** — no natural entry point; should be embedded as "Compare with..." in PlayerProfileDrawer

### Additions to Build 🆕
- **"Ask about this player" button** in PlayerProfileDrawer → opens Agent pre-filled with player name
- **Weekly Tuesday digest email** — W/L recap, top waiver targets, lineup preview
- **PWA push notifications** — injury alerts are time-sensitive; email is too slow
- **Roster health summary on home** — 5-position health bar (QB/RB/WR/TE/K) at a glance

### Removals / Demotions 🗑️
- **Viz/Analytics** → demote to "Advanced" section (too niche for casual players)
- **Compare (/compare)** → repurpose as embedded flow in PlayerProfileDrawer, not standalone page
- **Journal (/journal)** → FF players don't journal; remove or hide deep in "More"

### Priority Build Order (Next Session)
1. Auto-load Lineup Analyzer (remove button, load on mount)
2. "This Week's Actions" card on home dashboard
3. Floating "Ask AI" button across all pages
4. Sidebar cleanup — collapse advanced routes into "More" section
5. "Ask about this player" in PlayerProfileDrawer
6. Watchlist → Supabase persistence
7. Google OAuth (Supabase)
8. Weekly digest email

---

## Session 32 — March 16, 2026 (Audit Fixes + Live Data Pipeline)

### Part 1 — AUDIT_S31 Fixes

**Backend (`ff-bot` — commit `5a10f9a`):**
- **`engine/lineup.py`** — added `_get_matchup_score()` helper that logs `logger.warning` when `matchup_score` is null (rather than silently defaulting to 50.0); added weather fetch block in `recommend()` — fetches `weather` table rows for home teams, builds a text block, prepends to `recent_news` before the Claude call
- **`api/routes/viz.py`** — removed stale `# NOTE: currently always 50` comment on matchup_score fallback
- **`AUDIT_S31.md`** — full 5-pillar audit report (2 Critical, 2 High, 7 Medium, 6 Low findings)

**Frontend (`ff-bot-web` — commit `ea3cd6e`):**
- **`lib/api.ts`** — added 7 typed wrapper functions and interfaces: `getAvailablePlayers`, `getWaiverPriority`, `getTradeRosterNeeds`, `getPowerRankings`, `comparePlayers`, `getPreferences`, `updatePreferences`; all go through `request()` helper (auto `X-User-Email` header)
- **`waiver/page.tsx`** — migrated 2 direct fetch calls to api.ts wrappers; added AI picks skeleton shimmer
- **`trade/page.tsx`** — migrated roster-needs fetch to api.ts; added league scan skeleton shimmer
- **`league/page.tsx`** — parallelized standings + power rankings via `Promise.all`; migrated to api.ts wrappers; removed local `PowerRankRow` interface
- **`compare/page.tsx`** — removed local interfaces + `POS_COLOR`; imported from api.ts + constants; added loading skeleton
- **`settings/page.tsx`** — migrated all prefs fetches to api.ts; replaced "Loading preferences…" text with skeleton shimmer placeholders

**Operational tasks completed:**
- `FRONTEND_URL=https://ff-bot-web.vercel.app` set in Railway env vars
- Dynasty values refreshed: `python3 -m scripts.refresh_dynasty_values --season 2024` (2025 data not yet available at that point)

---

### Part 2 — 2025 Live Data Pipeline (Sleeper Stats Ingestion)

**Problem:** `weekly_stats` only had 2022–2024 data. nflverse hasn't published `player_stats_2025.parquet`. Dynasty values, recent form, and avg pts were all 2024-based.

**Solution:** Sleeper free stats API (`/v1/stats/nfl/regular/{season}/{week}`) has complete 2025 regular season data. Built a full ingestion pipeline around it.

**Backend (`ff-bot` — commit `0fcb44e`):**
- **`ingestion/weekly_stats.py`** — added `_fetch_from_sleeper(week, season, db)` as attempt 4 in the fallback chain (after nfl_data_py + 2 nflverse parquet attempts fail). Maps Sleeper player IDs → GSIS player IDs via `players.sleeper_id`. Derives std_pts as `half_ppr - rec * 0.5` so `_upsert_weekly_stats` recomputes correctly. Updated `run_weekly_stats_ingestion` to call it when nflverse returns nothing.
- **`scripts/backfill_sleeper_stats_2025.py`** (new) — one-time script; loops weeks 1–18 calling `run_weekly_stats_ingestion`; supports `--weeks 1-18` and `--season` args
- **`scripts/refresh_dynasty_values.py`** — fixed `weekly_stats.position` column error (column doesn't exist; removed `.in_("position", ...)` filter)
- **`.github/workflows/weekly_learning.yml`** — added dynasty value refresh step after weekly learning; Tuesday GitHub Action now: pulls weekly stats (nflverse → Sleeper fallback) → runs learning engine → refreshes dynasty values

**Backfill results:**
- 18 weeks × ~500 active players ingested into `weekly_stats` and `opportunity_metrics` for 2025
- Dynasty values refreshed from 2025 performance: Caleb Williams #3 QB, Joe Burrow #4, Tee Higgins top WR
- Going forward: Tuesday GitHub Action automatically ingests each completed week via Sleeper fallback until nflverse publishes 2025 parquet

**Key technical note — correct Sleeper stats URL:**
- ❌ `https://api.sleeper.app/v1/stats/nfl/{season}/{week}?season_type=regular` — returns rankings only (no actual stats)
- ✅ `https://api.sleeper.app/v1/stats/nfl/regular/{season}/{week}` — returns full stats with `pts_half_ppr`, `rec`, `rush_yd`, etc.

---

## Session 31 — March 16, 2026 (UX Polish Sprint)

**Frontend (`ff-bot-web` — commit `24a95d8`):**
- **PlayerProfileDrawer** — full rewrite: POS_COLOR badge, bio strip (height/weight/college/exp), season summary, recent games table with score color coding, skeleton loading states, backdrop click to close
- **Roster page** — fetches `/lineup/slots`, renders slot summary pills (Nx QB/RB/WR/TE/bench), START/SIT Analysis button → modal calling POST `/lineup/recommend` with tier badges + AI summary
- **League page** — clicking rows in Standings/Power Rankings opens right-side slide-over panel (320px, `translateX` transition) showing record, power rank badge, trend, roster by position using `PlayerInlineRow`
- **Settings page** — delete league button per row with inline confirm ("Remove" → "Confirm/Cancel" with disclaimer text), DELETE `/leagues/{id}` call, redirect to `/enter` if no leagues remain
- **Waiver page** — added "AI Picks | All Available" tab bar; All Available tab: position filter pills, sort (DV/Proj/Opp), star watchlist (localStorage `ff_watchlist`), `PlayerInlineRow` rows
- **Dynasty Capital** — 4-round × 4-year CSS grid timeline (owned ✓ green / owed ✗ red / — own), AI Capital Assessment card at top with Capital Rich/Balanced/Capital Poor badge
- **War Room** — Dynasty Cornerstones section (DV≥80) at top of identity tab with harsh age colors; `playerAgeColor()` with green/blue/orange/red prime window rules
- **Trade** — `PlayerTag onRemove` fix; search results show DV + proj; AI Roster Needs card (calls `/trade/roster-needs`); per-player breakdown columns; confidence + value_delta badges; match_score CSS bar in league scan
- **layout.tsx** — Lineup removed from all nav surfaces (sidebar + mobile drawer), conditional `leagueTypes` complete for War Room/Dynasty Capital/Keepers/Rookie Draft

**Backend (`ff-bot` — commit `603f0ff`):**
- **`api/routes/lineup.py`** — added `GET /lineup/slots?league_id=` — returns `{ starter_slots: string[], bench_count: int }` from cached `user_leagues.roster_positions` or live Sleeper fetch
- **`api/routes/trade.py`** — added `GET /trade/roster-needs?league_id=&roster_id=` — position depth vs league avg, Claude 2-sentence summary, returns `{ needs, surplus, my_counts, avg_counts, summary }`
- **`api/routes/waiver.py`** — added `GET /waiver/available?league_id=&position=` — all unrostered players enriched with `dynasty_value`, `blended_projection`, `opportunity_score`, `injury_status`
- **`lib/utils.ts`** (new) — `getTeamName(roster, usersMap)` helper: `metadata.team_name` → `usersMap[owner_id].display_name` → `owner_id`

---

## Session 30 — March 16, 2026 (9-Feature Sprint)

**Frontend (`ff-bot-web` — commit `5644aed`):**
- **Lineup tier filter pills** — All / Definite Start / Lean Start / Toss-Up / Lean Sit / Sit
- **Trade click-to-fill** — opponent roster panel already appended; confirmed + mobile grid fix applied
- **`app/dashboard/viz/page.tsx`** — new 4-tab analytics page: Trends (SVG line chart, player search), Health (injury/snap grid), Breakdown (stacked pillar bars), Standings + Waiver Opps (table + CSS bars). No npm packages.
- **`app/dashboard/compare/page.tsx`** — player comparison tool: search up to 4, side-by-side stats table with best-value highlights
- **`app/dashboard/settings/page.tsx`** — profile (display name), notifications (email_alerts + frequency), connected leagues, theme toggle
- **layout.tsx** — viz/compare/settings added to nav, icons, page titles, mobile drawer
- **Mobile UX pass** — `overflow-x-auto` on tables/pill rows (waiver/roster/war-room/league), `grid-cols-1 sm:grid-cols-2` on trade, `flex-wrap` on lineup controls

**Backend (`ff-bot` — commit `9442e4b`, includes unpushed S27 + S29):**
- **`ingestion/weather.py`** — switched OWM `/weather` → `/forecast` endpoint; now uses real `pop` field for precipitation probability
- **`ingestion/matchup.py`** (new) — computes `matchup_score` from opponent's pts allowed by position (last 4 wks), normalizes 0–100, bulk-updates `weekly_projections.matchup_score`
- **`engine/consensus.py`** — restored matchup weight: 0.00 → 0.12; opp 0.42 → 0.35; proj 0.30 → 0.25
- **`api/routes/players.py`** — added `GET /players/compare?player_ids=&league_id=&season=&week=` (up to 4 players, composite_score, dynasty_value, opp, proj, injury, boom/bust, recent avg)
- **`api/routes/auth.py`** — added `GET/PATCH /auth/preferences` (display_name, email_alerts, alert_frequency)
- **`scripts/refresh_dynasty_values.py`** (new) — batch refreshes dynasty_value for all active players; supports `--dry-run`

---

## Session 29 — March 16, 2026 (Full Visual Redesign)

**Goal:** FantasyPros + ESPN hybrid — sports-forward, data-dense, warm charcoal dark mode + full light mode toggle, C2T Builds branding throughout.

### What changed
- **`public/logo-dark.svg` + `public/logo-light.svg`** — C2T `[C2T]` terminal mark brand assets
- **`app/globals.css`** — full rewrite: dark/light mode via `data-theme="light"` on `<html>`, new palette (`#111214` bg, `#60A5FA` dark accent, `#2563EB` light accent), `.card`, `.stat-hero`, `.stat-large`, `.stat-medium`, `.label-caps`, `.skeleton-shimmer`, `.inj-pulse` utility classes
- **`app/dashboard/layout.tsx`** — theme toggle (☀/☾) persisted to `localStorage["ff_theme"]`; C2T logo in sidebar; ESPN left-border active nav state; full SVG icon set (NAV_ICONS) replacing all Unicode/emoji; dynamic page title + week badge in top bar
- **All 12 dashboard pages** — `font-black tracking-tight` headers, `.card` containers, `stat-hero`/`stat-large`/`stat-medium` on scores/records/values, `label-caps` section labels, `skeleton-shimmer` replacing `animate-pulse`
- **`app/enter/page.tsx`** — C2T logo centered above form, "FF **AI Bot**" title with accent color, `.card` form class
- **`components/PlayerInlineRow.tsx`** — solid filled position badges (white text on color bg), `boxShadow` on clickable rows, `inj-pulse` animation on Out/IR dots

### Commit: `da126ea` — pushed to `teel23/ff-bot-web` main ✅

---

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
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `RESEND_API_KEY`, `ALERT_FROM_EMAIL`) — needed for GitHub Actions email workflows
- [ ] **Re-run `python -m ingestion.historical`** when nflverse publishes 2025 weekly_stats parquet (check `player_stats_2025.parquet` — currently 404)
- [ ] Roster page — show slot groupings (Starters / Bench / Taxi / IR) instead of position groups
- [ ] Drop suggestions paired with waiver pickups ("Pick up X → Drop Y")
- [ ] **Add `RESEND_API_KEY` + `ALERT_FROM_EMAIL` to Railway env vars** — email alerts not functional until set
- [ ] Dynasty offseason calendar card on home dashboard
- [ ] Roster page — show dynasty value + injury badge inline per player
- [ ] **Multi-league quick-switch** — if user has multiple leagues connected, a visible dropdown on every page (not buried in the leagues screen) to swap active league.
- [ ] **Player card hover tooltip with recent stats** — desktop hover on `PlayerInlineRow` slides in mini tooltip showing last 3 weeks of actual points as small bars
- [ ] **War room radar chart entrance animation** — polygon animates from center outward using SVG `stroke-dashoffset` on page load
- [ ] **Confidence arc on verdict card** — SVG semicircle arc from 0–100% replacing or augmenting the raw confidence_pct number
- [ ] **Pulsing dot on current week in schedule** — animated green pulse dot next to the active week row to visually anchor "you are here"
- [ ] **Gradient row background on waiver by opp score** — row bg shifts faint red→green based on opp score (≤4 red-dim, ≥7 green-dim); no chip needed
- [ ] **Color-blended roster health strip** — segment fills use gradient blend between healthy/injured colors rather than flat fills
- [ ] **Waiver wire claim deadline countdown** — live countdown to Tuesday FAAB deadline shown at top of waiver page when within 24 hours
- [ ] **Dynasty value history seeding script** — write script to seed monthly KTC snapshots into `dynasty_value_history` table for real trend data in drawer
- [ ] **Animated tier transitions on lineup page** — smooth CSS `max-height` transition on tier group expand/collapse instead of instant show/hide
- [ ] **League Scan match_score → visual bar** — the compatibility score column renders as a raw number (e.g. `5`) with no context. Replace with a thin accent-colored progress bar proportional to max match_score in the list.
- [ ] **Notification preferences per alert type** — Settings toggle for each alert type (injury, waiver close, lineup reminder) independently
- [ ] **Render per-player breakdown from API response** — backend returns `giving_evaluation.players[]` and `receiving_evaluation.players[]` with `ros_projected_points`, `dynasty_value`, `age`, `blended_projection` per player. Frontend `TradeResult` interface only reads `verdict/reasoning/recommendation` — all per-player data is silently dropped. Add a small breakdown table under the verdict showing what drove the decision.
- [ ] **Fix PlayerTag `onRemove` no-op in PlayerSearch** — `onRemove={() => {}}` at line 85 in `trade/page.tsx` means removing a player from the search selection does nothing. Removal only works from the secondary list below.
- [ ] **League Scan result cached in localStorage** — cache scan result with timestamp, load instantly on re-visit, "Refresh" button to re-run
- [ ] **Agent chat export** — "Copy conversation" button that puts full Q&A on clipboard as plain text
- [ ] **Keeper cost efficiency sort** — sort keepers by `dynasty_value / keeper_cost` ratio, surfacing best value keeps first
- [ ] **Per-player opportunity score trend in drawer** — show how opp score moved over last 4 weeks using `opportunity_metrics` table
- [ ] **Trade suggested offer text → player chips** — `partner.what_to_offer` text auto-searches your roster and shows matching players as quick-add chips
- [ ] **In-app session changelog page** — `/dashboard/changelog` renders the most recent NOTES.md session entry as formatted HTML
- [ ] **Draft round/pick estimator** — show estimated overall pick number (e.g., "Your 2026 2nd ≈ Overall #16–20") on rookie draft page
- [ ] **Schedule opponent difficulty color coding** — opponent name gets faint red/green/neutral bg based on their power ranking
- [ ] **Standings page** — `/dashboard/standings` with W/L/PF/PA/streak, user’s team highlighted
- [ ] **Roster slot utilization badge on home** — single-line "10/10 starters set · 3 taxi open" strip so you see roster completeness at a glance
- [ ] **Trade offseason mode note upgrade** — replace generic "Trading is open" banner with "Evaluating via dynasty value — projection data unavailable until Week 1"
- [ ] **Branded 404 page** — unknown routes show a branded error page with links back to Dashboard instead of default Next.js blank
- [ ] **War room magic number tracker** — how many wins to clinch playoff spot shown as filled/unfilled circles (earned vs needed)
- [ ] **Supabase row count in health check** — add `data_health` to `/health/ping` response showing row counts for projections/injuries/news tables
- [ ] **Home page "Top priority today" card** — single top-of-page action card surfacing the #1 most time-sensitive item (e.g., "Waiver closes in 6h — RB need")
- [ ] **Stash/cut recommender** — when picking up a waiver player, suggest who to cut from taxi/bench to make room. Engine already has dynasty_value + age for all players.
- [ ] Mobile UX audit pass — 390px viewport, fix overflow/truncation on all 8 pages
- [ ] **C1 Lineup: tier filter pills** — filter lineup view to show only "Toss-Up" or specific tier
- [ ] **Taxi squad recommender** — who to promote from taxi to roster
- [ ] **Positional DV breakdown in League Rosters tab** — each team card already has dynasty values loaded. Add a QB/RB/WR/TE DV sum row per team card.
- [ ] **Remove Lineup tab from nav** — redundant with the START/SIT functionality that should live on the Roster page. Remove from sidebar and bottom nav.
- [ ] **Notifications page polish** — mark-all-read, empty state illustration, group by date
- [ ] **Dynasty value refresh script** — ingest latest KTC values into player_profiles.dynasty_value
- [ ] **Email alerts** — wire `RESEND_API_KEY` to actually send emails (infra + templates exist, send logic incomplete)
- [ ] **C3 Trade: multi-select from panel** — clicking in opponent panel should append to You Receive, not replace
- [ ] **Trade page search overhaul** — current search requires exact name match. Fix: (1) search by first name, last name, or any prefix using `ILIKE` or fuzzy match on backend; (2) show dynasty value / projected score next to each player in search results AND in the selected player list; (3) add AI analysis section at top of page before you even run a trade; (4) add "My Team" selector panel mirroring the opponent roster panel so you can click-to-fill "You Give" from your own roster.
- [ ] **Draft board page** — `/dashboard/draft` show owned picks, estimated value, trade scenarios
- [ ] **Re-run `python -m ingestion.historical`** when nflverse publishes `player_stats_2025.parquet` (currently 404)
- [ ] Investigate Sleeper live game data API for 2026 in-season scoring (live scores, player points mid-game)
- [ ] **Mobile PWA manifest** — `manifest.json` + app icons for "Add to Home Screen" on iOS/Android. No framework changes needed, just static files.
- [ ] **Playoff bracket preview** — for teams in contention, show simulated bracket for weeks 15-17 based on current standings and seeding rules.
- [ ] **Sticky tier headers on lineup scroll** — "Definite Start", "Lean Start" etc. become sticky on mobile scroll so you always know which tier you're in
- [ ] **Positional color bottom border on player rows** — 2px bottom border in POS_COLOR per player row; subtle visual rhythm
- [ ] **Score differential color gradient in schedule** — `+/-` color scales from bright green → muted → bright red rather than binary green/red
- [ ] **Playoff odds meter on war room** — needle-gauge SVG showing estimated playoff probability (0–100%) based on record + remaining schedule
- [ ] **Player comparison tool** — `/dashboard/compare`, side-by-side composite score + dynasty value + projections
- [ ] **Dynasty capital AI narrative collapsed by default** — collapse AI analysis behind `<details>` like reasoning blocks elsewhere
- [ ] **Compact/full density toggle on roster page** — switch shrinks `PlayerInlineRow` from 56px → 36px to show all 33 players without scrolling
- [ ] **Trade scan auto-fill target players** — clicking "Start Trade →" from League Scan should pre-fill "You Receive" with Claude's named target players, not just show a banner
- [ ] **Trade dynasty vs redraft toggle** — toggle switches evaluation between dynasty mode (DV-heavy) and redraft mode (projection-heavy)
- [ ] **Settings page** — `/dashboard/settings` display name, notification prefs, disconnect league
- [ ] **Last sync timestamp per data type** — SyncButton tooltip shows "Projections: 6h ago · Injuries: 2h ago · News: 45m ago" granularity
- [ ] **Player profile start % stat** — `espn_ownership.pct_started` exists in DB but not shown in `PlayerProfileDrawer`. Add as a chip.
- [ ] **Send `roster_needs` from frontend** — backend `TradeEngine` applies a +10% bonus for receiving players that match `roster_needs`, but the frontend `handleEvaluate()` call never passes this field. Pull weak positions from the existing roster data already loaded.
- [ ] Previous draft analysis — ADP vs actual production grades (boom/sleeper/bust)
- [ ] **RACR/WOPR scoring** — incorporate `racr` + `wopr` (now in opportunity_metrics) into receiver composite score
- [ ] **Dynasty capital drill-down** — show per-player breakdown of how dynasty capital score is calculated
- [ ] **Inline news on lineup page** — `news_items` table has last 48h Sleeper news. Show a snippet next to player name on lineup page if recent news exists.
- [ ] **Roster page — Sleeper-style layout** — completely rework to match Sleeper's visual format: labeled starter slots (QB, RB1, RB2, WR1, WR2, TE, FLEX, etc.) at the top, then bench below clearly separated, each player color-coded by position, with START/SIT options visible per player on this page. Currently all players look the same with no slot labels.
- [ ] **Waiver wire — show all available players in the league** — currently only shows AI-ranked suggestions. Add a full "Free Agency" section showing all players not on a roster in the league (call Sleeper `/league/{id}/rosters` to derive available players). Add a Sync button to refresh on demand. Rename tab to "Waivers / Free Agency".
- [ ] **Schedule page — fix broken state** — page is not loading/working. Diagnose root cause (likely the backend proxy endpoint from S27 or auth header issue) and restore to working state.
- [ ] **Dynasty Capital — 4 rounds per year** — Chimp Dynasty rookie draft has 4 rounds, not 3. Update the pick timeline grid to show rounds 1-4. Also move AI analysis to the top of the page and expand it — current output is too generic/programmatic.
- [ ] **Conditional nav by league type** — Rookie Draft tab only visible for dynasty leagues. Draft tab only visible for redraft leagues. Keepers tab only visible for keeper leagues. Gate by `activeLeague.league_type` in `layout.tsx`.
- [ ] **Weather alerts on lineup page** — `ingestion/weather.py` runs daily but weather data never appears in lineup recommendations. Show a weather chip inline on affected players.
- [ ] **Positional need badge on waiver** — waiver engine already knows your roster composition. Surface an "Addresses Need" flag on pickups that match a weak position.
- [ ] **League activity feed** — pull Sleeper transaction history (adds/drops/trades across all 12 teams). Sleeper has `GET /league/{id}/transactions/{round}`.
- [ ] **Roster age distribution chart** — bar or donut showing roster by age bucket (≤23 / 24-26 / 27-29 / 30+). Add to War Room alongside radar chart.
- [ ] **Animated DV balance meter fill** — trade balance bar animates from center outward using CSS `width` transition (0 → final, 400ms) on render
- [ ] **Injury dot pulse for active injuries** — Out/IR dots have continuous slow pulse animation; Questionable dots are static yellow
- [ ] **Bulk player bio sync endpoint** — `POST /players/sync-bios` refreshes age/team/injury for all 33 roster players in one Sleeper batch, triggered from Settings
- [ ] **Waiver sort button filled active state** — active sort = filled accent pill (white text); inactive = ghost. Currently too subtle to read.
- [ ] **"Already rostered" warning in trade search** — if searched player is on another team's roster, show "Rostered by [Team]" in the dropdown
- [ ] **Lineup "Optimal vs Actual" projected gap** — show optimal lineup score vs what you have set: "Leaving 4.3 pts on the bench"
- [ ] **Player profile "comparable players"** — inside `PlayerProfileDrawer`, show 2-3 players with similar age, position, and dynasty value as a comp reference.
- [ ] **Power ranking trend chart** — power rankings are computed weekly but no historical movement chart exists. Visualize your rise/fall over the season in the War Room or League page.
- [ ] **Position scarcity overlay on draft page** — sticky strip showing how many QBs/RBs/WRs/TEs remain in top 50 available while viewing draft board
- [ ] **Trade history log** — persist evaluated trades to Supabase so you can review past analyses and see if Claude's verdicts were right.
- [ ] **Bye week visualizer** — show which weeks starters have byes on a small calendar grid. Pure client-side from roster + schedule data already loaded.
- [ ] **FAAB bid amount recommender** — $1000 FAAB dynasty league + waiver engine scores players. Add a suggested bid amount based on player opportunity score + remaining budget.
- [ ] **War room peak window timeline — be harsher** — current logic turns too many players green (prime). Tighten the rules: prime window should only apply to players with DV ≥ 60 AND age 23-27. Everyone else should be yellow (developing) or red (declining/aging). Add a "Superstars" callout section highlighting your top 3 players by DV with their value score prominent.
- [ ] **Home page load speed** — page takes too long to load. Implement `usePageCache` with localStorage stale-while-fresh caching so cached data renders immediately and refreshes in background. Consider parallelizing backend calls if currently sequential.
- [ ] **League tab — clickable team cards** — clicking any team in Standings, Power Rankings, or Rosters tab should open a dedicated team detail view showing that team's full roster, record, PF/PA, and dynasty value breakdown.
- [ ] **Settings — add delete league option** — add a "Remove league" button per connected league in the Settings page. Confirm dialog before deleting from `user_leagues`.
- [ ] **Roster export button** — "Copy roster" outputs full 33-player list (Name · POS · Team · DV) as plain text for Discord/Slack
- [ ] Add option to delete connected Sleeper leagues
- [ ] Add VERCEL_TOKEN to Vercel env vars → unlocks Deployments page
- [ ] Set up cron to auto-refresh `dynasty_value` from KTC periodically (currently static seed)
- [ ] **Draft pick value calculator** — "What is a 2026 1st worth vs a 2027 2nd?" Uses existing dynasty_value data. Standalone tool page.
- [ ] **League/team names — use team name not username** — everywhere a league or team is referenced in the UI (nav, top bar, league context, dropdowns), use `team_name` first, falling back to `league_name`. Never show the Sleeper username (e.g., "ctteel") as the primary identifier.
- [ ] **Global style overhaul** — the overall visual design needs a refresh. Log this as a dedicated session to define a new direction before touching code.
- [ ] **Add GitHub Secrets** (`SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `ODDS_API_KEY`, `OPENWEATHER_API_KEY`) — needed for GitHub Actions ingestion + email workflows
- [ ] **Learning engine dashboard** — `engine/learning.py` auto-adjusts `source_weights` each week but there's no UI showing current weights or MAE history. Add a section in War Room.
- [ ] Add `/admin/ingest-status` endpoint showing last run timestamps for all ingestion jobs
- [ ] **ESPN ownership trends page** — `espn_ownership` table is ingested daily but never surfaced in the UI. Add a section showing pct_owned, pct_started, trending adds/drops per player.
- [ ] **Add `FRONTEND_URL` to Railway env vars** — email alert links will be broken without this
- [ ] **Show `confidence_pct` and `value_delta` in verdict card** — both are returned by `/trade/evaluate` but not in the `TradeResult` type and never rendered. `value_delta` (e.g. `+14.3 pts`) and `confidence_pct` (e.g. `80%`) should appear in the verdict card.
- [ ] **Dynasty value sparkline in PlayerProfileDrawer** — small 8-week KTC value trend line inside the drawer
- [ ] **Left-to-right shimmer on skeleton cards** — replace `animate-pulse` opacity flash with a directional sweep shimmer
- [ ] **Dynasty capital dot sizing by round** — R1 circle = 24px, R2 = 20px, R3 = 16px. Communicates pick value visually without labels.
- [ ] **In-app changelog** — "What's New" slide-up shown once per deploy, driven by static JSON. No backend needed.
- [ ] **"Last time you faced this opponent" on schedule** — show last matchup result above each week row
- [ ] **Standings row highlight pulse on load** — user's own row flashes a single accent pulse on page load to draw the eye to their position
- [ ] **Trade verdict badge entrance animation** — ACCEPT/DECLINE/NEUTRAL badge scales in from 0→1 with a spring-style CSS animation on load
- [ ] **Tab underline slide animation on mode switch** — active underline slides horizontally on trade page tab switch instead of appearing/disappearing
- [ ] **Projected final standings simulation** — use power rankings + remaining schedule to estimate final seed. Show "on track for #3 seed" type output.
- [ ] **Historical H2H record on schedule page** — expand any matchup row to show all-time head-to-head record vs that opponent. Sleeper has full matchup history.
- [x] Deploy web interface via Vercel
- [x] Connect to Supabase for persistence

---

## Deferred to Future Sessions (from Session 40 planning)

- [ ] **Draft page major overhaul** — big changes coming, held for dedicated session
- [ ] **AI Agent refinement** — improve prompts, add more data context, refine UX; `// TODO: Refine AI Agent in future session`
- [ ] **Settings page refinement** — notification preferences, theme options, account management
- [ ] **Playoff Push adjustments** — War Room section kept but will be refined
- [ ] **Standings adjustments** — League page standings kept but will be refined later
- [ ] **Power Rankings** — removed from League page display but logic kept for future views (War Room integration, schedule difficulty, etc.)
- [ ] **Analytics** — removed from League page but logic kept for future dedicated analytics page
- [ ] **Trade History** — removed from Trade page tab but code kept for future trade log feature
- [ ] **Trade engine value calculation deep-dive** — build comprehensive player value model: dynasty value (KTC), projected points, age/prime window, recent trend, positional scarcity, draft capital, team needs, composite scoring. See TODO in `engine/trade.py`.
- [ ] **Waiver ranking weights** — make the composite waiver score weights user-configurable in Settings
- [ ] **Home page load speed** — implement `usePageCache` with stale-while-fresh for home page data
- [ ] **Taxi squad recommender** — who to promote from taxi to roster
- [ ] **Lineup optimizer overhaul** — full rebuild for in-season start/sit recommendations
- [ ] **Trade Team Compare full impl** — Session 40 added shell; Session 41 should build out side-by-side roster browsing with click-to-add
- [ ] **Picks Capital shared component** — Dynasty Capital, War Room, and Home page all reference picks; extract into a shared `<PickCapitalGrid>` component with full 4-round Sleeper traded_picks data
- [ ] **Position rank on player nameplate** — WR12/RB5 etc. based on 2025 weekly_stats totals; requires a rank computation pass
- [ ] **PlayerInlineRow projected pts** — show full-season projected pts offseason, weekly proj in-season; source from weekly_projections.blended_projection
