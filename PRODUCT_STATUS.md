# FF Bot — Product Status
**As of:** March 16, 2026 (Session 28 — Competitive UX Redesign Sprints 1–4)
**What it is:** AI-powered fantasy football assistant — works with any Sleeper league. Claude analyzes your roster, waiver wire, trades, and matchup every week.

**Live URLs:**
- Frontend: https://ff-bot-web.vercel.app
- Backend: https://web-production-dddf8.up.railway.app
- API Docs: https://web-production-dddf8.up.railway.app/docs

---

## Overall Progress: ~92% complete (re-assessed March 16, 2026)

### Session 28 — UX Redesign Complete ✅
The frontend now has feature-parity with FantasyPros/Sleeper/ESPN/4for4 visual patterns:
- **Tier dividers** in all ranked lists (FantasyPros pattern)
- **Inline stat chips** on every player row (4for4 pattern)
- **Floor-ceiling range bars** in Lineup (ESPN pattern)
- **Day-of-week action bar** on Home (ESPN/Sleeper)
- **Collapsed `<details>` reasoning** on all AI pages
- **SVG radar chart** in War Room
- **DV Balance Meter** in Trade
- **Pick timeline grid** in Dynasty Capital
- **Composite bar viz** in Power Rankings
- **Sparkline bar chart** in Schedule
- **Context strip** (team + week) in top bar on all pages
- **POS_COLOR centralized** — single source of truth in `lib/constants.ts`

---

## Overall Progress: ~35% usable (re-assessed March 10, 2026)
> ⚠️ Previous 75% estimate was based on file existence, not functional completeness. See `AUDIT_REPORT.md` for full breakdown. See `MASTER_TODO.md` for all open work.

### New features added to roadmap (Session 16)
- Year-round dynasty support (off-season hub, dynasty capital, trade market, calendar)
- Prior year draft analysis + historical grade view
- Boom/bust profiles by position (QB/RB/WR/TE each have different key metrics)
- League settings-aware rankings (PPR vs Half-PPR shifts values, SF shifts QB value)
- In-season live stats pipeline (Sleeper for live scoring, nflverse for post-game analysis)
- Live draft agent improvements (real-time board, dynasty vs redraft mode, auction support)
- Notifications system fix (spam, automation, role change alerts, trade offer detection)
- Login flow full audit and fix
- Drop suggestions engine (AI-driven, dynasty-context-aware)
- Shared agent context object (structured payload for all Claude calls)
- Roster display logic (starter/bench/taxi/IR groupings, dynasty value inline)

---

## Feature Status

### Infrastructure & Auth
| Feature | Status | Notes |
|---|---|---|
| Supabase database (19 tables) | ✅ Done | Players, stats, rosters, schedules, snap counts, projections, injuries, waiver flags — all populated |
| FastAPI backend | ✅ Done | 14 routes, boots clean |
| Next.js 16 frontend | ✅ Done | Dark theme, Tailwind, 18 pages |
| Magic link auth (email login) | ✅ Done | Supabase SSR, no passwords. **VERIFIED WORKING v1.4.0** |
| Multi-league architecture | ✅ Done | One user → many leagues across platforms |
| Invite-only beta access | ✅ Done | Owner sends invite link → /accept-invite?token= → magic link sent. **VERIFIED WORKING v1.4.0** |
| Mobile responsive layout | ✅ Done | Hamburger + slide-in sidebar; responsive padding; touch-friendly |
| Waitlist / self-serve signup | ⬜ Not started | Planned after beta |

---

### Core AI Engines
| Engine | Status | Notes |
|---|---|---|
| **Lineup Optimizer** | ✅ Done | Scores all roster players across 5 pillars (opportunity, projections, vegas, matchup, news), picks optimal starters using your league's actual roster slots, scoring-format-aware |
| **Waiver Wire Analyzer** | ✅ Done | Role change alerts, opportunity metrics, FAAB bid suggestions (knows your remaining budget), Claude ranking + reasoning |
| **Trade Evaluator** | ✅ Done | ROS value comparison, positional scarcity, dynasty-aware (factors in age + picks vs redraft ROS), ACCEPT/DECLINE/NEUTRAL verdict |
| **Trade League Scan (Mode B)** | ✅ Done | Scans all 12 rosters, Claude ranks top 3 partners with targets + offer suggestions |
| **AI Chat** | ✅ Done | 4 context modes: trade negotiation, start/sit debate, waiver advice, season strategy. Conversation history. Floating panel on every page. |
| **Home Briefing** | ✅ Done | Claude writes a 3-4 sentence war room memo every week summarizing your situation |
| **Self-learning weights** | ✅ Done | Weekly job updates pillar weights based on actual vs predicted scores (MAE feedback loop) |
| **Keeper / Dynasty Optimizer** | ✅ Done | POST /keepers/analyze — KEEP/TRADE/CUT recommendations per player; dynasty = value-based; keeper = cost-vs-value |
| **Boom/Bust Model** | ✅ Done | Elite/High Floor/High Ceiling/Boom-or-Bust/Fade/Value tiers computed server-side from projection variance |
| **Scoring engine (JSONB)** | ✅ Done | Rec-value adjustment using target_share + scoring format; full JSONB weights passed to Claude |

---

### Dashboard Pages
| Page | Status | Notes |
|---|---|---|
| **Home** | ✅ Done | Card grid + ActionBar (day-of-week nudge), position depth mini bars with POS_COLOR, matchup, injury report, waiver alerts, Claude briefing |
| **Roster** | ✅ Done | HealthStrip, position-grouped with TierDividers, DV + age chips, SlotBadge, BOOM_BUST_BORDER left border, PlayerProfileDrawer |
| **Lineup Optimizer** | ✅ Done | Floor-ceiling RangeBar, tier-grouped list (5 tiers) with TierDividers, optimal lineup strip, collapsed `<details>` reasoning |
| **Waiver Wire** | ✅ Done | Sort modes (rank/proj/opp/role), PlayerInlineRow with color-coded Opp chip, FAAB banner, collapsed reasoning, inline drop suggestion |
| **Trade Analyzer** | ✅ Done | DV Balance Meter (all-roster fetch), upgraded verdict card (large centered verdict + trajectory line), Mode B league scan |
| **GM War Room** | ✅ Done | SVG radar chart (5-axis: Age/Upside/Pick Capital/Floor/Depth), Gantt timeline, Playoff Push tab, Rebuild vs Contend verdict |
| **League** | ✅ Done | Standings with +/- column, Power Rankings composite bar viz, Rosters tab (all 12 teams) |
| **Schedule** | ✅ Done | Playoff seeding banner, sparkline bar chart header, PF/PA/+/- columns, backend-proxied Sleeper calls |
| **Dynasty Capital** | ✅ Done | Capital Balance bar, Pick Timeline grid (seasons × rounds), Net Capital Score bar, AI analysis |
| **Player Profile** | ✅ Done | Full drawer: bio, injury, boom/bust tier badge, projection, recent stats, opportunity trend |
| **Notifications** | ✅ Done | Bell in top bar with badge; dropdown; injury + role change alerts |
| **Rookie Draft Prep** | ✅ Done | Dynasty off-season — incoming class, dynasty tier (Elite→Flier), position filter, PlayerProfileDrawer |
| **Draft** | ✅ Done | Live board (5s polling), AI Pick Assistant, pick grading vs ADP (A-F), Grade Draft slide-out |
| **Keepers** | ✅ Done | Dynasty: value-based optimizer. Keeper: round cost vs value → KEEP/TRADE/CUT |
| **Settings (Leagues)** | ✅ Done | Connect leagues, auto-imported settings, manual re-sync |
| **AI Agent** | ✅ Done | 11-source real data context, inline markdown, localStorage history, 4 quick chips |

---

### League Integration
| Feature | Status | Notes |
|---|---|---|
| **Sleeper** — connect league | ✅ Done | Username lookup → auto-import all settings (scoring, roster slots, waiver type, playoff config) |
| **Sleeper** — roster sync | ✅ Done | Pulls live roster on connect + manual re-sync button |
| **Sleeper** — standings | ✅ Done | Live W/L/PF/PA pulled on demand |
| **Sleeper** — matchups | ✅ Done | Current week matchup card on Home page |
| **Sleeper** — transactions (news) | ✅ Done | Recent adds/drops/trades with resolved player names |
| **Sleeper** — drafts | ✅ Done | Draft list + live pick board via Sleeper draft API |
| **ESPN** | 🔶 Stub | Phase 2 — route + full OAuth design notes written (SWID+espn_s2). Implementation deferred. |
| **Yahoo** | ⬜ Not started | Phase 3 |

---

### Data Pipeline
| Data Source | Status | Notes |
|---|---|---|
| NFL player roster (2022–2025) | ✅ Done | 4,596 players |
| Weekly stats (2022–2024) | ✅ Done | 16,881 rows. 2025 not yet published by nflverse (expected mid-2026). |
| Schedules (2022–2025) | ✅ Done | 1,139 games |
| Snap counts (2022–2025) | ✅ Done | 106,148 rows |
| Next Gen Stats | ✅ Done | 5,776 rows |
| Injury reports | ✅ Done | 793 live rows via ESPN |
| Weekly projections (2025) | ✅ Done | 30,651 rows seeded |
| Opportunity metrics (2025) | ✅ Done | 32,454 rows seeded |
| Role change flags | ✅ Done | 10 seeded for testing |
| Vegas lines | ✅ Done | Live via The Odds API (no lines off-season — expected) |
| Weather | ✅ Done | Live via OpenWeatherMap (no games off-season — expected) |
| 2025 nflverse data | ⬜ Waiting | nflverse publishes ~mid-2026. Ingestion handles gracefully when available. |

---

### Bugs Found in First Real Use (Session 15)
| Bug | Status |
|---|---|
| Dynasty league detected as redraft | ✅ Fixed — `settings.type` not `league_info.league_type`. Re-sync required. |
| Keepers + Rookie Draft nav shown for all league types | ✅ Fixed — now filtered by `activeLeague.league_type` |
| All pages blank (off-season) | Known — off-season content strategy needed |
| Roster not flowing to pages | Needs league re-sync to populate `league_rosters` |
| Home page empty | Off-season: no current matchup. Needs dynasty off-season view |
| Trade analyzer blank | Needs roster sync first |
| War Room blank | Needs roster sync + league_type = dynasty |

### What's Left (Gap Analysis)
| Priority | Feature | Notes |
|---|---|---|
| P0 | Off-season content for all pages | All pages show blank in off-season. Dynasty pages especially need purpose: rookie prep, trade market, dynasty capital. |
| P0 | Roster sync verification | Confirm `league_rosters` table is populated after connect/sync for this user |
| P1 | Per-page off-season states | Each page needs a meaningful off-season state instead of blank/error |
| P2 | ESPN full integration | OAuth flow + roster/lineup/trade sync. Design already written. |
| Low | Auction / salary cap support | Dollar valuations, live auction tracker, inflation calculator |
| Low | Email notifications | Working — injury alerts via Resend. Add custom domain to fix spam folder. |
| Low | Waitlist / self-serve signup | Invite-only beta is fine for now. |

---

### Deferred Questions — All Resolved (Session 13)
| Question | Decision |
|---|---|
| Draft assistant live sync | Polling every 5–10s (stateless, simpler) |
| Historical draft ADP source | Sleeper ADP primary + FantasyPros scrape supplement |
| ESPN OAuth | Deferred — validate with Sleeper users first |
| Push notifications | Email only (Resend) for injury alerts. Web push deferred. |

---

## Tech Stack
| Layer | Stack |
|---|---|
| Backend | Python 3.9, FastAPI, Supabase (Postgres), supabase-py |
| AI | Claude Sonnet 4.6 (Anthropic) — all reasoning goes through Claude |
| Frontend | Next.js 16, TypeScript, Tailwind CSS |
| Auth | Supabase magic link |
| Data | nflverse (parquet), ESPN injuries, The Odds API, OpenWeatherMap |
| Hosting | Local dev. Vercel (frontend) + Railway/Fly (backend) when ready to deploy. |
