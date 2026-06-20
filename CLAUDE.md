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

## Worst Pick page (`worst.html`)
- **Shame crown**: big featured card for the current worst team (the #1 shame leader)
- **Full team ranking**: all 36 picked teams sorted worst → best
- **Sort order**: losses DESC → goals conceded DESC → goals scored ASC (fewer goals = more shameful)
- **Columns**: GP, L (red if > 0), GA (red if > 0), GF
- **Highlighting**: #1 gets 💀 icon + red border glow; ranks 2–3 get fading red border; teams with 0 losses + 0 GA are faded
- **Language toggle**: shares the same `wc-lang` localStorage key as scoreboard
- **Updating**: `worst.html` copies the same `SEED` data block as `scoreboard.html` — keep both in sync when updating match results

## Updating the scoreboard
When asked to update, Claude should:
1. Web search current match results and standings for all 36 teams above
2. Web fetch FoxSports odds page for updated qualify % (convert American odds to implied probability)
3. Web fetch ESPN fixtures page for updated results and upcoming schedule
4. Update `SEED` data in `scoreboard.html` with new `played`, `next`, and `qualifyPct` values
5. Update `lastUpdated` string
6. Commit and push — GitHub Pages deploys in ~1 min

**Key sources:**
- Results/schedule: `https://www.espn.com/soccer/story/_/id/48939282/2026-fifa-world-cup-fixtures-results-match-schedule-group-stage-knockout-rounds-bracket`
- Qualify odds: `https://www.foxsports.com/stories/soccer/2026-world-cup-odds-teams-favored-advance-knockout-stage`

## Scoreboard data structure (per team)
```js
{
  name: "Belgium", group: "G", pot: 1,
  qualifyPct: 96,          // % chance to make Round of 32 (from FoxSports odds)
  played: [
    { date: "Jun 15", vs: "Egypt", score: "1-1", wdl: "D" }
  ],
  next: [
    { date: "Jun 21", vs: "Iran" },
    { date: "Jun 26", vs: "New Zealand" }
  ]
}
```
`qualifyPct: 100` = confirmed qualified, `qualifyPct: 0` = eliminated. Status badge is auto-derived from this field — no manual status clicks needed.
