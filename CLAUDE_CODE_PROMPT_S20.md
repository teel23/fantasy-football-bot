# FF Bot — Claude Code Session 20

> Drop this file into a new Claude Code session running in `Projects/fantasy-football-bot/`
> Run with: `claude --dangerously-skip-permissions`

---

## Project Context

**What it is:** AI-powered dynasty fantasy football assistant for Carson's Chimp Dynasty league (Sleeper).

**Stack:**
- Backend: FastAPI (Python) — deployed on Railway
- Frontend: Next.js 16 + TypeScript + Tailwind CSS — deployed on Vercel
- Database: Supabase (`keksjsxvcymsjhzxubxw.supabase.co`) — separate project from AI Dashboard
- AI: Claude Sonnet (`claude-sonnet-4-6`) via Anthropic SDK

**URLs:**
- Railway backend: `https://web-production-dddf8.up.railway.app`
- Vercel frontend: `https://ff-bot-web.vercel.app`
- Supabase: `keksjsxvcymsjhzxubxw.supabase.co`
- API docs: `https://web-production-dddf8.up.railway.app/docs`

**League:**
- Name: Chimp Dynasty
- Platform: Sleeper
- League ID: `1328784373125771264`
- Type: Dynasty, PPR, 12 teams, 33-player rosters (starter/bench/taxi/IR)
- Carson's roster_id: 9

**Repo structure:**
```
fantasy-football-bot/
├── ff-bot/                  ← FastAPI backend
│   ├── api/
│   │   ├── main.py          ← app init, CORS, router registration
│   │   └── routes/          ← one file per feature
│   ├── engine/              ← analysis + AI logic
│   ├── ingestion/           ← data pipelines
│   ├── db/                  ← Supabase client + schema + migrations
│   └── scripts/             ← one-off backfill scripts
└── ff-bot-web/              ← Next.js frontend
    ├── app/
    │   ├── dashboard/       ← 13 pages (home, roster, waiver, trade, draft, etc.)
    │   └── auth/            ← login, callback, confirm
    ├── components/
    └── lib/                 ← API client, utilities
```

**Key Supabase tables (relevant to this session):**
- `profiles` — user extended data (id, role, email)
- `user_leagues` — connected leagues with settings: `scoring_format`, `roster_positions`, `waiver_budget`, `league_type`
- `league_rosters` — 33 players per team with `slot_type` (starter/bench/taxi/ir), `player_id`, `player_name`, `position`
- `players` — 5,303 players with `birth_date`, `college`, `draft_round`, `draft_pick`, `years_exp`
- `player_profiles` — cache table for dynasty value + analysis (check if `dynasty_value` column exists)
- `weekly_stats` — 16,881 rows, 2022–2024 game-by-game stats (`fantasy_points`, `receptions`, `rushing_yards`, etc.)
- `weekly_projections` — 30,651 rows, 2025 season projections
- `opportunity_metrics` — 32,454 rows (snap share, target share, route participation, air yards — 2025)
- `role_change_flags` — 8,154 rows (downward/upward role changes, week, player_id)
- `injuries` — 1,000 rows (player_name, status: out/doubtful/questionable, description)
- `schedules` — 1,139 games 2022–2025

**IMPORTANT data note:** 2025 `weekly_stats` not yet published by nflverse (404). Use `weekly_projections` + `opportunity_metrics` as source of truth for 2025 player values.

---

## What's Already Working (Session 19 Complete)

All of these are verified working on live endpoints — do NOT rewrite or break them:

- ✅ Magic link auth + OTP fallback
- ✅ Sleeper league connect + auto-import (scoring_format, waiver_budget, roster_positions)
- ✅ League roster sync — all 33 players with correct `slot_type`
- ✅ Home dashboard (offseason + in-season modes) — `GET /home/dashboard`
- ✅ War Room scout report — `POST /war-room/scout-report`
- ✅ Roster timeline Gantt (33 players with ages)
- ✅ Power rankings (12 teams)
- ✅ Lineup optimizer (33 recs)
- ✅ Waiver wire — top 10 pickups with Claude reasoning
- ✅ Trade analyzer — real verdict + player reasoning
- ✅ Draft tools (ADP, pick recs, draft grades)
- ✅ Notifications (457 alerts)
- ✅ CORS: `allow_origin_regex=r"https://.*\.vercel\.app"` in `api/main.py`
- ✅ API URL: hardcoded in `next.config.ts` (do not change to env var)

---

## Session 20 — What To Build

Work through the tracks in order A → B → C → D → E. Commit after each track using `git add` on specific files only (never `git add .` — risk of committing `.env.local`).

---

### TRACK A — Email Infrastructure

**Goal:** Wire the full email alert system so it's ready to go the moment Railway env vars are added.

**A1 — Verify + implement `send_injury_alerts_to_all_users()`**

Check `ff-bot/engine/email.py`. If `send_injury_alerts_to_all_users()` does not exist, implement it:

```python
async def send_injury_alerts_to_all_users():
    """
    For each user:
    - Get their connected leagues from user_leagues
    - Get their starters from league_rosters (slot_type='starter')
    - Cross-reference against injuries table (status IN ('out','doubtful'))
    - If any injured starters found, send a Resend email with the list
    """
```

The function should use the existing Resend client already in `email.py`. If the Resend client isn't set up, add it using the `resend` Python package (`import resend; resend.api_key = os.environ.get("RESEND_API_KEY")`). Do NOT hardcode any API keys.

**A2 — GitHub Actions workflows**

Check if `.github/workflows/` exists in `ff-bot/`. If not, create it. Ensure these two workflow files exist:

`ff-bot/.github/workflows/daily-injury-alert.yml`:
```yaml
name: Daily Injury Alert
on:
  schedule:
    - cron: '0 9 * * *'   # 9 AM UTC daily during season (Sept–Jan)
  workflow_dispatch:
jobs:
  send-alerts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Send injury alerts
        run: |
          curl -X POST "${{ secrets.RAILWAY_URL }}/email-alerts/send-injury-alerts" \
            -H "Authorization: Bearer ${{ secrets.SUPABASE_SERVICE_KEY }}"
```

`ff-bot/.github/workflows/lineup-reminder.yml`:
```yaml
name: Lineup Reminder
on:
  schedule:
    - cron: '0 16 * * 0'  # 4 PM UTC Sunday (noon ET)
  workflow_dispatch:
jobs:
  send-reminders:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Send lineup reminders
        run: |
          curl -X POST "${{ secrets.RAILWAY_URL }}/email-alerts/send-lineup-reminders" \
            -H "Authorization: Bearer ${{ secrets.SUPABASE_SERVICE_KEY }}"
```

Verify the corresponding API endpoints (`POST /email-alerts/send-injury-alerts` and `POST /email-alerts/send-lineup-reminders`) exist in `ff-bot/api/routes/email_alerts.py`. If missing, add them — they should just call the engine functions.

**A3 — Write `SETUP_EMAILS.md`**

Create `SETUP_EMAILS.md` in the `fantasy-football-bot/` root (not inside ff-bot/). This is for Carson to follow manually. Include:

1. Railway dashboard steps: Add these env vars under the `ff-bot` service:
   - `RESEND_API_KEY` — get from resend.com dashboard
   - `ALERT_FROM_EMAIL` — format: `FF Bot <alerts@yourdomain.com>` (or use `onboarding@resend.dev` for testing until custom domain is set up)
   - `FRONTEND_URL` — `https://ff-bot-web.vercel.app`

2. GitHub repo steps: Go to `teel23/ff-bot` → Settings → Secrets and variables → Actions → New repository secret:
   - `SUPABASE_URL` — `https://keksjsxvcymsjhzxubxw.supabase.co`
   - `SUPABASE_SERVICE_KEY` — get from Supabase dashboard → Settings → API → service_role key
   - `RESEND_API_KEY` — same as Railway
   - `ALERT_FROM_EMAIL` — same as Railway
   - `RAILWAY_URL` — `https://web-production-dddf8.up.railway.app`

3. Test command: `curl -X POST https://web-production-dddf8.up.railway.app/email-alerts/send-injury-alerts -H "Authorization: Bearer YOUR_SERVICE_KEY"`

---

### TRACK B — Roster Page Full Overhaul

**Files to modify:**
- `ff-bot/api/routes/leagues.py` — roster endpoint
- `ff-bot-web/app/dashboard/roster/page.tsx` — roster UI

**B1 — Backend: Enrich roster data**

Find the route in `leagues.py` that serves the roster (likely `GET /leagues/{league_id}/roster` or similar). Extend it to return additional fields per player:

1. **dynasty_value:** First query `information_schema.columns` to check if `player_profiles` has a `dynasty_value` column. If yes, join it. If no column exists, compute a rough composite score inline:
   ```
   dynasty_value = (
     (draft_round_score: 100 if rd 1, 80 if rd 2, 60 if rd 3, 40 if rd 4+, 50 if undrafted) +
     (age_score: 100 if age <= 22, 85 if 23–24, 70 if 25–26, 50 if 27–28, 30 if 29–30, 10 if 31+) +
     (production_score: avg fantasy_points from weekly_stats 2022–2024, normalized 0–100)
   ) / 3
   ```
   Store computed values in `player_profiles` (upsert). Include `dynasty_value` in roster response.

2. **injury_status:** Join `injuries` table on `player_name` (or `sleeper_id` if available). Return `injury_status: "out" | "doubtful" | "questionable" | null` and `injury_description: str | null`.

The enriched player object should look like:
```json
{
  "player_id": "...",
  "player_name": "Justin Jefferson",
  "position": "WR",
  "slot_type": "starter",
  "dynasty_value": 94,
  "injury_status": null,
  "injury_description": null,
  "age": 26
}
```

**B2 — Frontend: Roster page overhaul**

Rewrite `ff-bot-web/app/dashboard/roster/page.tsx` with these changes:

1. **Remove position-based tabs.** Replace with 4 slot sections rendered in order:
   - **Starters** (slot_type === 'starter')
   - **Bench** (slot_type === 'bench')
   - **Taxi Squad** (slot_type === 'taxi')
   - **Injured Reserve** (slot_type === 'ir')

   Each section has a header with count (e.g., "Starters (15)"). Sections with 0 players are hidden.

2. **Search bar:** Text input at top of page. Client-side filter across all sections by player name (case-insensitive). Show "No results for [query]" if nothing matches.

3. **Sort toggle:** Three buttons above the player list (or dropdown): "Dynasty Value", "Age", "Projected Pts". Default: Dynasty Value descending. Sorting applies within each slot section independently.

4. **Per-player row** (left to right):
   - Position badge (colored pill: QB=purple, RB=green, WR=blue, TE=orange, K/DEF=gray)
   - Player name (clickable → opens existing PlayerProfileDrawer)
   - Boom/bust tier badge (small pill — rendered as empty/unknown for now; Track E will populate it)
   - Dynasty value (right-aligned, gray text — "DV: 87" format)
   - Injury badge (only if injury_status is non-null): red pill for "OUT", orange for "DOUBTFUL", yellow for "QUESTIONABLE"

5. Keep the existing off-season detection banner and "Roster not synced" empty state — don't remove these.

---

### TRACK C — Waiver Wire Drop Pairing

**Files to modify:**
- `ff-bot/engine/waiver.py`
- `ff-bot/api/routes/waiver.py`
- `ff-bot-web/app/dashboard/waiver/page.tsx`

**C1 — Backend: Add drop candidates**

In `ff-bot/engine/waiver.py`, after the main pickup scoring loop, add a `get_drop_candidates(supabase, league_id, user_id, pickup_position, top_n=3)` helper:

```python
async def get_drop_candidates(supabase, league_id, user_id, pickup_position, top_n=3):
    """
    Returns the top_n weakest rostered players at pickup_position.
    Weakest = lowest dynasty_value (from player_profiles),
              falling back to lowest projected_points if dynasty_value is null.
    Only considers bench players (slot_type='bench') — never suggest dropping starters, taxi, or IR.
    """
```

For each pickup in the results array, call this helper and attach:
```python
pickup["drop_candidates"] = [
    {"player_name": "...", "dynasty_value": 42, "reason": "Lowest dynasty value WR on bench"},
    ...
]
```

The `reason` field should be a 1-line string explaining why (e.g., "Age 31, minimal opportunity in 2025", "3rd-string RB, limited upside"). Generate reasons using a lightweight Claude call (batch all at once — not one call per player).

In `ff-bot/api/routes/waiver.py`, make sure the enriched `drop_candidates` field passes through in the response.

**C2 — Frontend: Show drop candidates**

In `ff-bot-web/app/dashboard/waiver/page.tsx`, under each pickup card add a "Consider dropping:" subsection:

```
[ Chris Godwin — WR — DV: 61 ← PRIMARY PICKUP CARD ]
  Consider dropping:
  • Darnell Mooney — DV: 38 — "Limited target share, aging profile"
  • Tre Tucker — DV: 29 — "3rd WR on depth chart, no consistent role"
```

Style: muted/gray text, smaller font. If `drop_candidates` is empty or missing, hide the section silently.

---

### TRACK D — Home Dashboard New Cards

**Files to modify:**
- `ff-bot/api/routes/home.py`
- `ff-bot-web/app/dashboard/page.tsx`

**D1 — Dynasty Offseason Calendar**

In `ff-bot/api/routes/home.py`, in the `_build_offseason_dashboard()` function, add an `offseason_calendar` key to the response dict:

```python
from datetime import date

def _build_offseason_calendar():
    today = date.today()
    phases = [
        {
            "phase": "Free Agency",
            "dates": "March 12 – April 23, 2026",
            "start": date(2026, 3, 12),
            "end": date(2026, 4, 23),
            "tips": [
                "Monitor FA signings for opportunity shifts",
                "RBs who land on new teams often see dynasty value spikes",
                "WR signings change target share projections"
            ]
        },
        {
            "phase": "NFL Draft",
            "dates": "April 24–26, 2026",
            "start": date(2026, 4, 24),
            "end": date(2026, 4, 26),
            "tips": [
                "Rookie WRs/TEs drafted in rounds 1–2 are immediate targets",
                "Top RBs in dynasty: target round 1–2 picks",
                "Watch for QBs in SF/2QB leagues"
            ]
        },
        {
            "phase": "Dynasty Rookie Draft",
            "dates": "May – June 2026",
            "start": date(2026, 5, 1),
            "end": date(2026, 6, 30),
            "tips": [
                "Use your 2026 picks for positional needs",
                "Stash TEs early — they take time to develop",
                "Sell surplus rookie picks if contending"
            ]
        },
        {
            "phase": "Training Camp",
            "dates": "Late July 2026",
            "start": date(2026, 7, 20),
            "end": date(2026, 8, 25),
            "tips": [
                "Watch depth chart battles for RB committees",
                "Monitor injury reports — sell hurt players fast",
                "Lock in your roster before Week 1"
            ]
        },
        {
            "phase": "Season Kickoff",
            "dates": "September 2026",
            "start": date(2026, 9, 1),
            "end": date(2026, 12, 31),
            "tips": [
                "Set your lineup every week",
                "Work the waiver wire aggressively early",
                "Start making trade moves by Week 4"
            ]
        }
    ]
    for p in phases:
        if today < p["start"]:
            p["status"] = "upcoming"
        elif p["start"] <= today <= p["end"]:
            p["status"] = "active"
        else:
            p["status"] = "complete"
        # Remove date objects before serializing
        del p["start"]
        del p["end"]
    return phases
```

Add `"offseason_calendar": _build_offseason_calendar()` to the offseason dashboard response.

**D1 Frontend:** In `ff-bot-web/app/dashboard/page.tsx`, add an `OffseasonCalendarCard` component to the off-season layout:
- Show 5 phases as a vertical timeline
- Active phase: highlighted (accent color border, bold), with tips shown below
- Upcoming phases: muted with dates
- Complete phases: strikethrough or grayed out with a checkmark

**D2 — Sell-High / Buy-Low Targets**

In `ff-bot/api/routes/home.py`, in `_build_offseason_dashboard()`:

1. After fetching roster data, build a short roster summary (player_name, position, age, dynasty_value) for Claude.

2. Add a Claude call: `get_sell_buy_targets(roster_summary, scoring_format)` to `ff-bot/engine/claude.py`:

```python
async def get_sell_buy_targets(roster_summary: list[dict], scoring_format: str) -> dict:
    """
    Ask Claude to identify:
    - sell_high: 3-5 players who are currently overvalued (age cliff, injury history, declining role, regressing stats)
    - buy_low: 3-5 players who are undervalued (age 21-24, high opportunity, rough recent stats but strong upside)
    Returns: {"sell_high": [...], "buy_low": [...]}
    Each item: {"player_name", "position", "reason"}
    """
```

System prompt for this call:
```
You are a dynasty fantasy football analyst. Given a roster of players with their ages and dynasty values, identify sell-high and buy-low trade targets.
Sell-high: players whose current dynasty value is likely near peak — aging veterans (27+), injury-prone players recovering, players on scheme fits that may change.
Buy-low: players aged 24 or under with strong opportunity metrics but whose dynasty value is suppressed — perhaps due to recent injury, down year, or new surroundings.
Scoring format: {scoring_format}. Adjust your analysis accordingly (PPR boosts WR/TE value, SF boosts QB value).
Return JSON only:
{
  "sell_high": [{"player_name": "...", "position": "...", "reason": "..."}],
  "buy_low": [{"player_name": "...", "position": "...", "reason": "..."}]
}
```

3. Add `"sell_high_targets"` and `"buy_low_targets"` to the offseason dashboard response.

**D2 Frontend:** In `ff-bot-web/app/dashboard/page.tsx`, add a `SellBuyCard` component to the off-season layout:
- Two columns: "Sell High" (red accent) and "Buy Low" (green accent)
- Each column shows 3-5 player rows: position badge, name, 1-line reason
- Card title: "Trade Targets"

---

### TRACK E — Dynasty Intelligence Features

#### E1 — Boom/Bust Engine

**Create:** `ff-bot/engine/boom_bust.py`

This engine tiers every player on a dynasty roster into one of 6 categories using existing Supabase data.

**Tier logic:**

```python
TIERS = ["Elite", "Safe Floor", "High Ceiling", "Boom-Bust", "Aging", "Bust Risk"]

def compute_boom_bust_tier(player: dict) -> str:
    """
    Inputs (all from Supabase):
    - age: int (from players.birth_date)
    - dynasty_value: float 0-100 (from player_profiles or computed)
    - snap_consistency: float 0-1 (std deviation of snap_pct from opportunity_metrics — lower = more consistent)
    - target_share_avg: float (avg from opportunity_metrics)
    - production_variance: float (std deviation of fantasy_points from weekly_stats)
    - role_change_flag_count: int (count of downward flags from role_change_flags)

    Rules (in priority order):
    - Elite: dynasty_value >= 85 AND age <= 26 AND snap_consistency <= 0.15
    - Safe Floor: dynasty_value >= 65 AND production_variance <= 6 AND age <= 28
    - Aging: age >= 30 OR (age >= 28 AND dynasty_value <= 50)
    - Bust Risk: role_change_flag_count >= 3 OR (age >= 29 AND dynasty_value <= 40)
    - High Ceiling: age <= 24 AND target_share_avg >= 0.18
    - Boom-Bust: production_variance >= 10 AND target_share_avg >= 0.15
    - Default: "Safe Floor"
    """
```

**New endpoint:** Add `POST /players/boom-bust` to `ff-bot/api/routes/players.py`:
```
Request: {"league_id": "...", "user_id": "..."}
Response: {
  "tiers": {
    "Elite": [{"player_name": "...", "position": "...", "age": 25, "dynasty_value": 91}],
    "Safe Floor": [...],
    "High Ceiling": [...],
    "Boom-Bust": [...],
    "Aging": [...],
    "Bust Risk": [...]
  }
}
```

Register the route in `ff-bot/api/main.py` if `players.py` isn't already registered.

**Roster page integration:** The roster endpoint (Track B) should also call `boom_bust.compute_boom_bust_tier()` per player and include `"boom_bust_tier": "Elite"` in the player object. The frontend already has placeholder boom/bust badge rendering in place (Track B). The tier→color mapping:
- Elite: gold/yellow
- Safe Floor: green
- High Ceiling: blue
- Boom-Bust: orange
- Aging: purple
- Bust Risk: red

#### E2 — Historical Draft Grades (League-Settings-Aware)

**Files to modify:**
- `ff-bot/engine/draft_grade.py` (179 lines — read it first before modifying)
- `ff-bot/api/routes/draft.py` (370 lines — add a new endpoint)
- `ff-bot-web/app/dashboard/draft/page.tsx` — add draft history section

**Backend:**

Add `GET /draft/grades/{league_id}` to `ff-bot/api/routes/draft.py`:

1. Fetch all drafts for the league from Sleeper:
   - `GET https://api.sleeper.app/v1/league/{league_id}/drafts`
   - For each draft: `GET https://api.sleeper.app/v1/draft/{draft_id}/picks`

2. Pull league settings from `user_leagues` (scoring_format, roster_positions).

3. For each drafted player, compute their actual 2024 production from `weekly_stats` (season totals: `SUM(fantasy_points)` for 2024, `SUM(receptions)`, `SUM(rushing_yards)`, etc.).

4. **League-settings-aware fantasy points formula:**
   ```python
   def compute_league_fantasy_points(stats: dict, settings: dict) -> float:
       scoring_format = settings.get("scoring_format", "ppr")
       roster_positions = settings.get("roster_positions", [])

       # Base stats
       pts = (
           stats.get("passing_yards", 0) * 0.04 +
           stats.get("passing_tds", 0) * 4 +
           stats.get("rushing_yards", 0) * 0.1 +
           stats.get("rushing_tds", 0) * 6 +
           stats.get("receiving_yards", 0) * 0.1 +
           stats.get("receiving_tds", 0) * 6
       )
       # PPR adjustment
       rec_pts = {"ppr": 1.0, "half_ppr": 0.5, "standard": 0.0}.get(scoring_format, 1.0)
       pts += stats.get("receptions", 0) * rec_pts

       # Superflex/2QB: QB points get a 1.5x multiplier for ranking purposes
       is_sf = "SUPER_FLEX" in roster_positions or "QB" in roster_positions[:3]
       if is_sf and stats.get("position") == "QB":
           pts *= 1.5

       # TEP (TE Premium): TE receptions worth extra 0.5 each
       if "REC_TE" in roster_positions and stats.get("position") == "TE":
           pts += stats.get("receptions", 0) * 0.5

       return round(pts, 1)
   ```

5. Grade each pick:
   ```python
   def grade_pick(pick_number: int, actual_points: float, position: str, year: int) -> str:
       """
       Compare actual fantasy points to position-adjusted expectation by draft round.
       Round 1 (picks 1-12): expect >= 200 pts for A; >= 150 for B; >= 100 for C; else D/F
       Round 2 (picks 13-24): expect >= 150 for A; >= 100 for B; >= 70 for C; else D/F
       Round 3+ (picks 25+): expect >= 100 for A; >= 70 for B; >= 40 for C; else D/F
       For rookies in year of draft: adjust thresholds down 40% (rookie wall)
       """
   ```

6. Response:
   ```json
   {
     "drafts": [
       {
         "draft_id": "...",
         "season": 2023,
         "type": "rookie",
         "overall_grade": "B+",
         "best_pick": {"pick": 4, "player_name": "...", "grade": "A+", "actual_points": 287},
         "worst_pick": {"pick": 9, "player_name": "...", "grade": "F", "actual_points": 3},
         "picks": [
           {
             "pick_number": 1,
             "round": 1,
             "player_name": "...",
             "position": "WR",
             "actual_fantasy_points": 234,
             "grade": "A",
             "league_adjusted_points": 234
           }
         ]
       }
     ]
   }
   ```

**Frontend:**

In `ff-bot-web/app/dashboard/draft/page.tsx`, add a "Draft Grade History" section below the existing draft content:
- Tab or accordion per draft (by year + type: "2023 Rookie Draft", "2024 Rookie Draft")
- Overall draft grade shown prominently (large letter grade)
- Pick-by-pick table: Pick #, Player, Position, Points, Grade
- Grade badges: A=green, B=blue, C=yellow, D=orange, F=red
- Callout cards: "Best Pick" and "Worst Pick" highlighted at top

#### E3 — League-Settings-Aware Rankings

**Create or modify:** `ff-bot/engine/rankings.py` (create if it doesn't exist)

Add a shared utility function that any engine can call:

```python
def get_league_settings_weights(scoring_format: str, roster_positions: list[str]) -> dict:
    """
    Returns a weights dict used to adjust player values for the specific league format.

    Example output:
    {
        "rec_bonus": 1.0,       # PPR bonus per reception (0=std, 0.5=half, 1.0=ppr)
        "qb_multiplier": 1.5,   # 1.5 for SF/2QB, 1.0 for standard
        "te_premium": 0.5,      # Extra rec pts for TE in TEP leagues, else 0
        "is_dynasty": True,     # Affects age weighting
    }
    """
    is_sf = "SUPER_FLEX" in roster_positions
    is_tep = "REC_TE" in roster_positions

    return {
        "rec_bonus": {"ppr": 1.0, "half_ppr": 0.5, "standard": 0.0}.get(scoring_format, 1.0),
        "qb_multiplier": 1.5 if is_sf else 1.0,
        "te_premium": 0.5 if is_tep else 0.0,
        "is_dynasty": True,
    }
```

**Apply to:**
1. `ff-bot/engine/waiver.py` — import `get_league_settings_weights()`, apply to pickup opportunity scores. Pass `scoring_format` from `user_leagues` (already fetched in the waiver route).
2. `ff-bot/engine/claude.py` — include league format context in the system prompts for `get_waiver_reasoning()`, `get_trade_reasoning()`, and any new functions added this session. Example addition to system prompt: `"League format: {scoring_format}. Adjust your analysis accordingly."`
3. `ff-bot/api/routes/waiver.py` — pass `league_settings` dict to `WaiverEngine` constructor or `analyze()` call.

---

## Final Checklist — Verify Before Committing

Run these checks manually or via `curl`:

- [ ] `GET /home/dashboard?league_id=1328784373125771264` returns `offseason_calendar`, `sell_high_targets`, `buy_low_targets`
- [ ] Roster page at `/dashboard/roster` shows 4 sections (Starters/Bench/Taxi/IR), search bar works, sort toggle works, dynasty value + injury badges visible per player
- [ ] `POST /waiver/analyze` response includes `drop_candidates` array on each pickup item
- [ ] Waiver page shows "Consider dropping:" section under each pickup
- [ ] `GET /draft/grades/{league_id}` returns valid pick-by-pick grades for Chimp Dynasty (league_id: `1328784373125771264`)
- [ ] Draft page shows draft grade history section with letter grades
- [ ] `POST /players/boom-bust` returns tiered roster object with all 6 tier keys
- [ ] Roster page boom/bust badges display (colored pills per player)
- [ ] `ff-bot/engine/email.py` has `send_injury_alerts_to_all_users()` function defined
- [ ] `.github/workflows/daily-injury-alert.yml` and `lineup-reminder.yml` exist
- [ ] `SETUP_EMAILS.md` written with Railway + GitHub instructions
- [ ] `python -m pytest ff-bot/tests/` passes (or no test failures introduced)
- [ ] `cd ff-bot-web && npm run build` passes (copy to /tmp/mini-games-build first if path has spaces issues)

## Commit instructions

After each track, commit with specific files only:
```bash
# Example — Track B commit:
git add ff-bot/api/routes/leagues.py ff-bot-web/app/dashboard/roster/page.tsx
git commit -m "Track B: roster page overhaul — slot grouping, badges, search, sort"
```

Never use `git add .` or `git add -A`.

Push after all tracks are complete:
```bash
~/.local/bin/gh auth status  # verify auth
git push origin main         # triggers Vercel + Railway auto-deploy
```
