# WorldCup Picker — Project Notes

## What this is
A three-page web app for a friend group to randomly pick World Cup 2026 teams and track their progress. Hosted on GitHub Pages. No build step — pure HTML/CSS/JS.

**Live URLs:**
- Picker: `https://sarunwi.github.io/worldcup-picker/`
- Scoreboard: `https://sarunwi.github.io/worldcup-picker/scoreboard.html`
- Worst Pick: `https://sarunwi.github.io/worldcup-picker/worst.html`

## Files
- `index.html` — Spinning wheel team picker
- `scoreboard.html` — Live scoreboard tracking each player's teams
- `worst.html` — Shame ranking: worst team among all picks wins a special award
- `teams.js` — **Single source of truth for all match data** (`teamByName` + `LAST_UPDATED`); loaded by both scoreboard and worst pages

## Firebase
- Project: `worldcup-picker` (Firestore, free tier)
- Picker state: `/rooms/{roomCode}` — synced in real-time across devices
- Scoreboard: **no longer uses Firestore**. Renders directly from the `SEED` constant in `scoreboard.html`. Firebase SDKs were removed from scoreboard.html entirely.

## Players & their teams (draft: 2026-06-19)
| Thai name | English name | Pot 1 | Pot 2 | Pot 3 | Pot 4 |
|-----------|--------------|-------|-------|-------|-------|
| ก้อง | Kong | Belgium | Morocco | Qatar | DR Congo |
| แว่น | OakVaan | Argentina | Japan | Scotland | New Zealand |
| มิว | Mew | France | Uruguay | Tunisia | Cabo Verde |
| นุ | Nuu | Spain | Croatia | South Africa | Jordan |
| ตือ | Oou | Germany | Australia | Algeria | Ghana |
| พิม | Pim | England | Austria | Paraguay | Curaçao |
| บลู | Chablue | Netherlands | Switzerland | Norway | Haiti |
| อาม | Arm | Portugal | Senegal | Uzbekistan | Iraq |
| เรีย | Lea | Brazil | Ecuador | Egypt | Turkey |

## Scoring system
- **Main award**: whoever picked the **1st and 2nd place teams** wins.
- **Worst Pick award**: whoever has the single worst-performing team (most losses, most goals conceded, fewest goals scored) wins a shame prize.
- No points system during the tournament — the scoreboard just tracks who makes the Round of 32.

## Scoreboard UI features
- **Team cards**: two-column layout — flag image (flagcdn.com) + group + status badge on left; match results with W/D/L badges + upcoming fixtures + qualify % bar on right
- **Click popup**: shows full match history, upcoming schedule, and qualify % bar
- **Language toggle**: TH/EN button in header switches all text including player names; preference saved in `localStorage` under key `wc-lang`
- **Status badge**: auto-derived from `qualifyPct` — 100 = qualified, 0 = eliminated, else alive
- **Flag images**: from `https://flagcdn.com/w80/{code}.png`; Scotland = `gb-sct`, England = `gb-eng`
- **Navigation**: header links to Picker, Worst Pick page, and lang toggle
- **Team name filter**: live text search above the board — hides non-matching cards and collapses empty player rows
- **Group filter chips**: A–L letter buttons below the text filter — click to show only teams in that group; combines with text filter (AND logic); click again to deselect; ✕ clears both; hidden when in Group view
- **Non-picked teams**: when filtering by group (in Player view), a "Group X — Other teams" section appears below the board showing cards for teams not picked by any player. All 48 WC2026 teams are in `teamByName` in `teams.js` — 36 picked (referenced from `SEED.players`), 12 non-picked referenced from `SEED.extras`
- **View toggle**: "👤 By Player" / "📊 By Group" buttons above the filter bar switch between views
  - **Player view** (default): one row per player, 4 team cards across
  - **Group view**: standings table per group (A–L), all 4 teams sorted by Pts → GD → GF; dashed line separates automatic qualifiers (top 2); player column highlights owners in gold, non-picked teams show "—"; clicking any row opens the same popup; text filter hides groups with no matches

## Worst Pick page (`worst.html`)
- **Shame crown**: big featured card for the current worst team (the #1 shame leader)
- **Full team ranking**: all 36 picked teams sorted worst → best
- **Sort order**: losses DESC → goals conceded DESC → goals scored ASC (fewer goals = more shameful)
- **Columns**: GP, L (red if > 0), GA (red if > 0), GF
- **Highlighting**: #1 gets 💀 icon + red border glow; ranks 2–3 get fading red border; teams with 0 losses + 0 GA are faded
- **Language toggle**: shares the same `wc-lang` localStorage key as scoreboard
- **Updating**: match data lives in `teams.js` only — `worst.html` and `scoreboard.html` both load it via `<script src="teams.js">`

## Updating the scoreboard
When asked to update, Claude should:
1. Web search current match results and standings for all 36 teams above
2. Web fetch FoxSports odds page for updated qualify % (convert American odds to implied probability)
3. Web fetch ESPN fixtures page for updated results and upcoming schedule
4. Update **`teams.js` only** — edit `qualifyPct`, `played`, `next` for each team in `teamByName`, and update `LAST_UPDATED`
5. Commit and push — GitHub Pages deploys in ~1 min

**Key sources:**
- Results/schedule: `https://www.espn.com/soccer/story/_/id/48939282/2026-fifa-world-cup-fixtures-results-match-schedule-group-stage-knockout-rounds-bracket`
- Qualify odds: `https://www.foxsports.com/stories/soccer/2026-world-cup-odds-teams-favored-advance-knockout-stage`

## Auto-update schedule (group stage)
CronCreate jobs were set up on 2026-06-21 to auto-update after each match day. **These are session-only — they die if the Claude Code session is closed.** If the session closes, recreate them by asking: *"Please recreate the 7 scoreboard auto-update cron jobs for the World Cup 2026 group stage."*

| Fire time (Thailand, UTC+7) | Covers |
|-----------------------------|--------|
| Jun 22, 2:07 AM | Jun 21 TH matches (NED/SWE, GER/CIV, ECU/CUW, TUN/JPN, ESP/KSA) |
| Jun 22, 11:07 AM | Jun 21 ET late matches (BEL/IRN, URU/CPV, NZL/EGY) |
| Jun 23, 1:13 PM | Jun 22 ET matches |
| Jun 24, 12:17 PM | Jun 23 ET matches |
| Jun 25, 11:07 AM | Jun 24 ET matches |
| Jun 26, 12:13 PM | Jun 25 ET matches |
| Jun 27, 1:17 PM | Jun 26 ET matches |
| Jun 28, 12:07 PM | Jun 27 ET matches (group stage final day) |

Timezone note: Thailand = ET + 11 hours. ESPN fixture times are in ET.

## Data structure

### `teams.js` — edit this file to update scores
```js
const LAST_UPDATED = "21 มิ.ย. 2026";

const teamByName = {
  "Belgium": { name:"Belgium", group:"G", pot:1,
    qualifyPct: 96,        // % chance to advance (from FoxSports odds)
    played: [{ date:"Jun 15", vs:"Egypt", score:"1-1", wdl:"D" }],
    next:   [{ date:"Jun 21", vs:"Iran" }, { date:"Jun 26", vs:"New Zealand" }]
  },
  // ... all 48 teams
};
```
- All 48 teams are in `teamByName` (36 picked + 12 non-picked extras)
- Non-picked teams have no `pot` field
- `qualifyPct: 100` = qualified, `qualifyPct: 0` = eliminated
- Status badge is auto-derived — no manual flag needed

### `SEED` in each HTML file — never needs to change
**`scoreboard.html`** has player assignments + extras list:
```js
const SEED = {
  players: [
    { name:"ก้อง", teams:["Belgium","Morocco","Qatar","DR Congo"] },
    // ...
  ],
  extras: ["Mexico","South Korea","Czechia","Canada","Bosnia","USA",
           "Ivory Coast","Sweden","Iran","Saudi Arabia","Colombia","Panama"],
};
```

**`worst.html`** has player assignments only (no extras):
```js
const SEED = {
  players: [
    { name:"ก้อง", teams:["Belgium","Morocco","Qatar","DR Congo"] },
    // ...
  ],
};
```

Both files resolve team names via `teamByName[name]` from `teams.js`.

Non-picked teams: Mexico, South Korea, Czechia (A) · Canada, Bosnia (B) · USA (D) · Ivory Coast (E) · Sweden (F) · Iran (G) · Saudi Arabia (H) · Colombia (K) · Panama (L)
