---
name: pick-fantasy-xi
description: Choose a valid daily Fantasy XI and optional Risk Play from the official tournament game board, using only provided player, match, team, standings, rules, and output-format files.
---

# Pick Fantasy XI

Use this skill for each daily AI Agent Fantasy World Cup run.

## Hard Rules

Return one plain JSON object only. Do not wrap it in Markdown.

Always use the exact `team_id` and `matchday_id` from the prompt or game board.
Always choose player IDs from `game-board/players.json`; never invent IDs.
Always choose match, team, and player IDs for Risk Play from the same official
game-board files.

For normal daily play, always return exactly these five top-level keys:
`team_id`, `matchday_id`, `fantasy_xi`, `risk_play`, and `strategy`.

Fantasy XI must contain exactly 11 eligible players:

- exactly 1 goalkeeper
- 3 to 5 defenders
- 3 to 5 midfielders
- 1 to 3 forwards

There is no budget, no captain, no bench, and no substitutions. Ignore any
outside fantasy-platform assumptions that conflict with the tournament rules.

Runtime constraints:

- Use only the provided runtime files: `rules/`, `game-board/`,
  `output-format/`, `current-standings/`, `team/README.md`, and
  `team/skills/`.
- Do not use internet research, scripts, package installs, credentials,
  cookies, private data, or files outside the tournament workspace.
- Do not create, edit, delete, or move files during the run.
- Do not depend on memory, cache, previous runs, or previous Risk Plays.
- Keep the process direct enough to finish within the 5-minute runtime limit.

## Input Reading Order

1. Read `prompt.md` for the current task, team ID, matchday ID, current score,
   and any Risk Play context.
2. Read `output-format/` and follow the schema exactly.
3. Read `rules/` for the current Fantasy XI, Risk Play, and bracket rules.
4. Read `game-board/matchday.json`, `matches.json`, `players.json`,
   `teams.json`, `claim-catalog.json`, and standings files.
5. Use `team/skills/pick-fantasy-xi/references/strategy.md` for tie-breakers.

## Player Selection Method

Start by removing any player who is not eligible for the current game board.
Prefer likely starters over uncertain players because non-playing selections
score 0.

Rank players by expected scoring path:

- starts and likely plays 60+ minutes
- goal involvement for forwards and attacking midfielders
- clean-sheet path for goalkeepers and defenders
- assist/set-piece path for creators
- lower card and own-goal risk
- stronger team and easier opponent
- compact current or last World Cup metrics in `players.json`

Use this practical score when the board has enough fields. Skip fields that are
absent rather than inventing them:

- confirmed starter or starter likelihood 80%+: +8
- likely 60+ minutes: +6
- strong goal path for FWD/MID: +5
- strong assist or set-piece path: +3
- strong DEF/GK clean-sheet path: +4
- GK has realistic 3+ save path: +2
- yellow-card risk: -2
- red-card or own-goal risk: -5
- unavailable, suspended, ineligible, or high minute risk: -10 and avoid

When player fields are sparse, fall back to team strength, opponent weakness,
position role, and any compact current or last World Cup metrics in the board.

Formation preference:

- Use 3-4-3 or 4-3-3 when three forwards and attacking midfielders have clearly
  better scores than the fourth/fifth defender options.
- Use 4-4-2 when attacking and defensive scores are close.
- Use 5-3-2 or 5-4-1 only when four or five defenders have strong clean-sheet
  paths and the extra forward/midfield options are weak or rotation risks.

Diversify only after expected value is close. Do not force diversity if it
causes a weaker XI, but avoid overloading fragile or uncertain teams.

## Formation Repair Rule

Build and validate the XI from official `position` values in
`game-board/players.json`; never infer position from player name, real-world role,
club usage, or strategy text.

Before final output, create a position count for the selected IDs:
`GK`, `DEF`, `MID`, and `FWD`. If the count is invalid, repair the XI before
returning JSON. Never submit an invalid formation.

Repair in this order:

1. If goalkeeper count is not exactly 1, keep the highest-ranked GK or add the
   highest-ranked available GK, replacing the lowest-ranked player from an
   overfilled position.
2. If defenders are fewer than 3, add the highest-ranked available DEF and
   remove the lowest-ranked player from an overfilled position.
3. If midfielders are fewer than 3, add the highest-ranked available MID and
   remove the lowest-ranked player from an overfilled position.
4. If forwards are fewer than 1, add the highest-ranked available FWD and remove
   the lowest-ranked player from an overfilled position.
5. If defenders are more than 5, remove the lowest-ranked extra DEF and replace
   with the best available player from an underfilled or valid position.
6. If midfielders are more than 5, remove the lowest-ranked extra MID and
   replace with the best available player from an underfilled or valid position.
7. If forwards are more than 3, remove the lowest-ranked extra FWD and replace
   with the best available MID first, then DEF if midfield is already strong.

After each repair, recount positions from `players.json`. Repeat until the XI is
exactly 11 unique eligible IDs with exactly 1 GK, 3-5 DEF, 3-5 MID, and 1-3 FWD.
If no valid repair is possible, choose the safest standard formation available
from the board in this order: 3-4-3, 4-4-2, 4-3-3, 5-3-2.

## Risk Play Method

Risk Play is optional. Choose `null` when confidence is not clear.
If Risk Play is skipped, missing, invalid, or `null`, no Risk Play is used for
that run. Never assume a previous Risk Play is reused.

If standings show team points at 0 or below, skip Risk Play because the stake is
0.

Compute the team's point gap from `game-board/standings-before.json`. If a new
team has no prior standing, treat the starting balance as 50 points. Missing
rank is not by itself a reason to skip Risk Play; evaluate Green claims normally
from the board evidence. Use stake percentages only for internal decision
quality; never output stake fields.

Use standings posture before choosing risk:

- Rank 1 to 3 or less than 10 points behind leader: choose `null` unless a
  Green claim is extremely strong.
- Rank 4 to 10 or 10 to 20 points behind leader: Green claims are acceptable
  with clear evidence; Yellow claims require unusually strong evidence.
- Rank 11+ or 21+ points behind leader: Yellow claims are acceptable with
  strong evidence; Red claims require exceptional evidence and a real need for
  variance.
- Team score at 0 or below: choose `null` because the computed stake is 0.

Use a claim only when:

- the required IDs are valid and belong together
- the match context supports the claim
- the upside is worth the stake category

Default risk posture:

- Green claims: use only when estimated confidence is at least 65%.
- Yellow claims: use only when estimated confidence is at least 80%.
- Red claims: use only when estimated confidence is at least 90%.

Prefer simple high-confidence claims such as `match_2plus_goals` or
`goal_before_halftime` over fragile exact-score or comeback claims. If multiple
claims pass their threshold, prefer Green over Yellow over Red unless standings
strongly require more variance.

## Validate Before Output

Before returning JSON, check:

- `fantasy_xi` has exactly 11 unique player IDs.
- every player ID exists in `game-board/players.json`.
- every selected player is eligible for today's board.
- formation is exactly 1 GK, 3-5 DEF, 3-5 MID, and 1-3 FWD after
  applying the Formation Repair Rule.
- `team_id` matches the prompt or official run identity.
- `matchday_id` matches `game-board/matchday.json` or the prompt.
- `risk_play` is either `null` or a valid claim from `claim-catalog.json`.
- all five daily top-level keys are present: `team_id`, `matchday_id`,
  `fantasy_xi`, `risk_play`, and `strategy`.
- any `team_id` or `player_id` inside `risk_play` belongs to the submitted
  `match_id`.
- no `bet_points`, `stake`, or `stake_percent` fields are included.
- no extra top-level keys are included unless the day's output schema requires
  them.
- `strategy` is a single-line string without raw newlines or unescaped quotes.
- final output starts with `{` and ends with `}`.

Common mistakes to avoid:

- do not use player names instead of player IDs
- do not pick players outside today's eligible list
- do not combine a match ID with a team or player from another match
- do not submit more or fewer than 11 players
- do not submit 4 or more forwards; replace the lowest-ranked extra forward with
  the best available midfielder or defender before output
- do not wrap the answer in Markdown fences
- do not rely on partial validity: invalid identity rejects scoring, invalid
  Fantasy XI scores 0, and invalid Risk Play is ignored

## Output

For normal daily play, return one raw JSON object that follows
`output-format/daily-submission.schema.json`. Use
`output-format/examples/daily-submission.json` only as a shape reference, never
as a source of IDs or values. Do not output Markdown code fences, introductory
text, comments, or trailing explanation.

The daily object should use the five-key contract from this skill unless the
active schema says otherwise: `team_id`, `matchday_id`, `fantasy_xi`,
`risk_play`, and `strategy`. If `risk_play` is active, use the exact claim object
required by `output-format/` and `claim-catalog.json`, with no stake fields.

If the prompt or output schema for the day requests a bracket answer, follow
that schema exactly. Otherwise, use the daily answer contract above.
