# WorldCup Picker — Project Notes

## What this is
A two-page web app for a friend group to randomly pick World Cup 2026 teams and track their progress. Hosted on GitHub Pages. No build step — pure HTML/CSS/JS.

**Live URLs:**
- Picker: `https://sarunwi.github.io/worldcup-picker/`
- Scoreboard: `https://sarunwi.github.io/worldcup-picker/scoreboard.html`

## Files
- `index.html` — Spinning wheel team picker
- `scoreboard.html` — Live scoreboard tracking each player's teams

## Firebase
- Project: `worldcup-picker` (Firestore, free tier)
- Picker state: `/rooms/{roomCode}` — synced in real-time across devices
- Scoreboard data: `/scoreboard/main` — single document, manually seeded

## Players & their teams (draft: 2026-06-19)
| Player | Pot 1 | Pot 2 | Pot 3 | Pot 4 |
|--------|-------|-------|-------|-------|
| ก้อง | Belgium | Morocco | Qatar | DR Congo |
| แว่น | Argentina | Japan | Scotland | New Zealand |
| มิว | France | Uruguay | Tunisia | Cabo Verde |
| นุ | Spain | Croatia | South Africa | Jordan |
| ตือ | Germany | Australia | Algeria | Ghana |
| พิม | England | Austria | Paraguay | Curaçao |
| บลู | Netherlands | Switzerland | Norway | Haiti |
| อาม | Portugal | Senegal | Uzbekistan | Iraq |
| เรีย | Brazil | Ecuador | Egypt | Turkey |

## Scoring system
At the end of the tournament: whoever picked the **1st and 2nd place teams** wins.
No points system during the tournament — the scoreboard just tracks who makes the Round of 32.

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
