# Dynasty Rookie Draft — Plan, Changes & Runbook (Session 41)
Date: June 24, 2026
Scope: Make the dynasty **rookie draft** feature work end to end — pre-draft board,
live on-the-clock assistant, pick capital, and post-draft grade.

---

## The problem (why it didn't "just work")
The draft engine was built for **redraft** drafts. "Best available" came from FantasyPros
redraft ADP / Sleeper `search_rank` — so in a rookie draft it would surface veterans, not
this year's rookie class. It defaulted to 15 rounds, and the production-grade endpoint was
hard-coded to season 2024. The rookie *board* (`/players/rookies`) existed but ranked only by
NFL draft capital and wasn't connected to the live recommender.

## The fix (what this session changed)
A dynasty **rookie board** is now a first-class data source, and the whole draft path is
rookie-aware. New code is committed locally in both repos (not yet pushed — see runbook).

### Backend — `ff-bot`
| File | Change |
|---|---|
| `db/migrations/007_rookie_rankings.sql` | **New.** `rookie_rankings` table (season, player, rank, tier, source) + RLS. |
| `ingestion/rookie_rankings.py` | **New.** Builds the dynasty rookie board. Tries FantasyPros dynasty rookie rankings; **falls back to NFL draft capital** (real data in `players`) so the board is never empty. Writes `rookie_rankings`. |
| `api/routes/players.py` | `/players/rookies` now merges `rookie_rankings` (rank + tier), sorts by dynasty rank, returns `rookie_rank` + `ranking_source`. Falls back to NFL-capital tier per player when unranked. |
| `api/routes/draft.py` | `/draft/adp` gains `mode=rookie&season=` (serves the rookie board). `/draft/recommend` auto-detects rookie drafts (Sleeper `season_type`), uses the rookie board for "best available", reads real round/team counts, returns `is_rookie`/`total_rounds`. `/draft/grades` no longer hard-codes 2024 — it grades each draft against its own rookie-year production. |
| `engine/draft_recommend.py` | `get_draft_recommendation(..., is_rookie=)` adds dynasty-rookie framing (long-term upside, age, landing spot) instead of win-now redraft logic. |

### Frontend — `ff-bot-web`
| File | Change |
|---|---|
| `lib/api.ts` | `RookiePlayer.rookie_rank`; `getRookies` default season 2026 + `ranking_source`; `getDraftAdp(mode, season)`; `getDraftRecommendation` accepts `draft_mode`/`season`; `DraftRecommendation` gains `is_rookie`/`total_rounds`. |
| `app/dashboard/rookie-draft/page.tsx` | Cards show the dynasty rank (`#N`) alongside the tier. |
| `app/dashboard/draft/page.tsx` | Live assistant detects rookie drafts, sends `draft_mode:"rookie"` + season, defaults to 4 rounds, and shows a "Rookie board" badge on the recommendation. |

### Verified this session
- `npx tsc --noEmit` in `ff-bot-web` → **0 errors**.
- `python -m py_compile` on all changed backend files → **OK**.
- `ingestion/rookie_rankings.py` imports and helper logic run correctly.
- **Not verifiable here:** live Supabase + Sleeper calls and the FantasyPros scrape — the
  sandbox blocks that egress. Those are validated in the runbook below.

---

## RUNBOOK — get it live (steps I can't run from here)

> Order matters. 1 → 2 → 3 are data prerequisites; the board is empty until they run.

### Step 1 — Apply the migration (Supabase SQL Editor)
Paste and run `ff-bot/db/migrations/007_rookie_rankings.sql`. Confirm:
```sql
select * from information_schema.tables where table_name = 'rookie_rankings';
```

### Step 2 — Refresh the 2026 rookie class into `players`
The board only includes players with `years_exp = 0`. Make sure Sleeper's annual rollover is
reflected:
```bash
cd ff-bot
python3 -m scripts.populate_player_bios
# sanity check afterward (SQL editor):
#   select count(*) from players where years_exp = 0 and position in ('QB','RB','WR','TE');
```
Expect a few dozen 2026 rookies. If it returns ~0, Sleeper hasn't rolled over yet, or bios
haven't synced — re-run closer to your draft.

### Step 3 — Build the dynasty rookie board
```bash
cd ff-bot
python3 -m ingestion.rookie_rankings --season 2026 --dry-run   # preview top 10 + source
python3 -m ingestion.rookie_rankings --season 2026             # write rookie_rankings
```
- `source=fantasypros_dynasty_rookie` → the scrape worked (best case).
- `source=nfl_draft_capital` → scrape was blocked/CSR; board still populated from real NFL
  draft capital. **This is expected and fine** — re-run later to pick up the scrape if it recovers.

### Step 4 — Deploy code
```bash
# Backend → Railway (auto-deploys on push to main)
cd ff-bot
git add db/migrations/007_rookie_rankings.sql ingestion/rookie_rankings.py \
        api/routes/players.py api/routes/draft.py engine/draft_recommend.py
git commit -m "S41: dynasty rookie-aware draft (board + live assistant + grade fix)"
git push origin main

# Frontend → Vercel (auto-deploys on push to main)
cd ../ff-bot-web
git add lib/api.ts app/dashboard/rookie-draft/page.tsx app/dashboard/draft/page.tsx
git commit -m "S41: rookie-aware draft UI (rank on board, rookie mode in live assistant)"
git push origin main
```
After Railway is up: `curl -s https://web-production-dddf8.up.railway.app/health/ping` → `{"ok":true}`.

### Step 5 — Verify end to end
1. **Pre-draft board** — open `/dashboard/rookie-draft`. You should see the 2026 class with
   `#rank` badges and tiers, sorted by dynasty rank.
2. **Live assistant** — `/dashboard/draft` → pick the upcoming rookie draft → "Get
   recommendation". It should show a **rookie** with a "Rookie board" badge and dynasty-focused
   reasoning (not a veteran). Backend self-check (auth required):
   `GET /draft/adp?mode=rookie&season=2026` returns the board.
3. **Pick capital** — `/dashboard/dynasty-capital` should list your owned/owed picks across all
   4 rounds. (Backend already returns these from Sleeper `traded_picks`; if a round is missing,
   it's a Sleeper data issue, not the UI — see follow-ups.)
4. **Post-draft grade** (after the draft) — `/dashboard/draft` → "Grade" on the completed
   rookie draft. Production grades now use the correct season.

---

## Acceptance checklist
- [ ] `rookie_rankings` table exists (Step 1).
- [ ] `players` has the 2026 rookies at `years_exp=0` (Step 2).
- [ ] `rookie_rankings` populated for 2026 (Step 3) — note the `source`.
- [ ] Both repos pushed; Railway `/health/ping` 200; Vercel deployed (Step 4).
- [ ] Rookie board shows ranked 2026 class; live assistant recommends a rookie with the
      "Rookie board" badge (Step 5).

---

## Follow-ups / optional polish (not blocking your draft)
- **Automate the board.** Add `python3 -m ingestion.rookie_rankings --season 2026` to
  `.github/workflows/daily_ingestion.yml` (offseason) so rankings stay fresh without manual runs.
- **Scrape reliability.** If `source` stays `nfl_draft_capital`, the FantasyPros page is likely
  CSR. Options: target their JSON endpoint, or import a KTC/rookie CSV into `rookie_rankings`
  directly (same table, `source='manual'`). NFL-capital ordering is a solid fallback regardless.
- **Pick capital rounds.** If dynasty-capital shows fewer than 4 rounds, confirm
  `user_leagues.sleeper_roster_id` is set for Chimp Dynasty (same gap that affects Schedule —
  see `ff-bot/AUDIT_S41.md` H7) and that Sleeper `traded_picks` returns all rounds.
- **Rookie grade thresholds.** `_grade_pick_by_production` already softens thresholds 40% for
  rookie drafts; revisit once 2026 rookies have real production.
- The broader scoring-pipeline issues (matchup/vegas/weather) in `AUDIT_S41.md` are independent
  of the draft path and don't block this feature.
