---
name: choose-risk-play
description: Choose or skip the optional Risk Play from the official claim catalog, using standings posture and only the current tournament game-board files.
---

# Choose Risk Play

Use this skill when the prompt asks for a Risk Play, Jeopardy play, claim
selection, or daily answer that includes a `risk_play` field.

## Hard Rules

Use only claims from `game-board/claim-catalog.json`.
Never invent claim IDs, match IDs, team IDs, player IDs, or stake values.
Never include `stake`, `stake_percent`, `bet_points`, or any manual wager size.
The tournament calculates stake automatically.

If the daily output schema asks for a full answer, place the selected claim
object or `null` inside the `risk_play` key. Do not output a separate object
unless `output-format/` explicitly asks for only Risk Play.

If Risk Play is skipped, missing, invalid, or `null`, no Risk Play is used for
that run. Never assume a previous Risk Play is reused.

Runtime constraints:

- Use only the provided runtime files: `rules/`, `game-board/`,
  `output-format/`, `current-standings/`, `team/README.md`, and
  `team/skills/`.
- Do not depend on internet research. If public web research is available, use
  it only under the Optional Public Web Research rules below. Never use scripts,
  package installs, credentials, cookies, private data, or files outside the
  tournament workspace.
- Do not create, edit, delete, or move files during the run.
- Do not depend on memory, cache, previous runs, or previous Risk Plays.
- Keep the process direct enough to finish within the 5-minute runtime limit.

## Input Reading Order

1. Read `prompt.md` for team identity, matchday, score, and objective.
2. Read `output-format/` so the Risk Play object shape is exact.
3. Read `rules/` for current stake and validity rules.
4. Read `game-board/claim-catalog.json`, `matches.json`, `players.json`,
   `teams.json`, and `standings-before.json`.
5. If current standings are provided, use them to decide whether to protect or
   chase.

## Optional Public Web Research

Use web research only if the prompt says network is open and the run still has
ample time. The Risk Play decision must still work without web access.

Strict limits:

- Use at most 1 quick search for the preferred Risk Play fixture, or 2 searches
  if the first result is unclear. Stop if search is slow or low quality.
- Search only public, unauthenticated sources for current lineup, injury,
  suspension, weather, match-tempo, odds, or card-risk signals.
- Never use web research to invent claim IDs, match IDs, team IDs, player IDs,
  or required fields. `claim-catalog.json`, `matches.json`, and `players.json`
  remain the source of truth.
- Use web findings only to adjust confidence: high-tempo or weak-defense signals
  help goal claims; bad weather or low-goal signals help cautious Green claims;
  confirmed attacking starters help player/team scoring claims; strong card-risk
  signals help card claims.
- Do not force Risk Play because web search was available. If confidence remains
  unclear, return `risk_play: null`.
- Do not add `web_search_details`, source links, or extra fields. If useful,
  summarize the research impact briefly inside the existing `strategy` string.

## Decision Method

Default to `null` for Yellow or Red claims unless the board provides clear
evidence. For Green claims, first evaluate all broad-event Green options in
`claim-catalog.json` before deciding to skip.

When team points are positive and the team is outside the top tier or materially
behind the leader, actively look for the best Green claim on multi-match slates
before returning `null`. Prefer broad-event Green claims with multiple paths,
such as `match_2plus_goals`, `goal_before_halftime`, or `match_2plus_cards`.
Do not skip a broad Green claim only because individual player lineups in that
match are unconfirmed; evaluate team strength, match tempo, scoring environment,
and defensive vulnerability. Skip Risk Play only when all available Green claims
are unsupported, unusually volatile, or tied to clearly low-scoring defensive
match contexts. This does not lower the Yellow or Red thresholds.

Compute the team's point gap from `game-board/standings-before.json`. If a new
team has no prior standing, treat the starting balance as 50 points. Missing
rank is not by itself a reason to skip Risk Play; evaluate Green claims normally
from the board evidence. Use stake percentages only for internal decision
quality; never output stake fields.

Risk posture by standings:

- Rank 1 to 3 or less than 10 points behind leader: choose `null` unless a
  Green claim is extremely strong.
- Rank 4 to 10 or 10 to 20 points behind leader: Green claims are acceptable
  with clear evidence; Yellow claims require unusually strong evidence.
- Rank 11+ or 21+ points behind leader: actively seek the best supported
  Green claim first; Yellow claims are acceptable with strong evidence; Red
  claims require exceptional evidence and a real need for variance.
- Team score at 0 or below: choose `null` because the computed stake is 0.

Minimum confidence thresholds:

- Green: at least 65% confidence.
- Yellow: at least 80% confidence.
- Red: at least 90% confidence.

Stake impact for internal math:

- Green risks 15% of pre-matchday points, rounded half up.
- Yellow risks 25% of pre-matchday points, rounded half up.
- Red risks 35% of pre-matchday points, rounded half up.

Prefer claims with broad event paths over fragile exact outcomes. In close
choices, prefer Green over Yellow over Red because invalid or failed Risk Plays
can erase otherwise good Fantasy XI points.

## Claim Preference

Generally prefer:

- `match_2plus_goals`
- `goal_before_halftime`
- `match_2plus_cards`
- `both_teams_score`
- `match_over_2_5_goals`

Use player-specific claims only when the selected player is very likely to
start, has a strong scoring role, and is attached to the correct match ID.

Use exact-score, comeback, 2+ goal, extra-time, penalties, or red-card claims
only when the prompt or board creates exceptional evidence. Otherwise skip.

Scoring edge cases:

- Group-stage matches end after regulation plus stoppage time; they do not go
  to extra time or penalties.
- Knockout final-score and player-event claims include regulation plus extra
  time unless current rules say otherwise.
- Penalty shootout goals count only for explicit penalty-shootout claims.
- Total-card claims count yellow cards, second-yellow red cards, and red cards.
- Yellow-card claims count only yellow-card events.
- Red-card claims count red cards and second-yellow red cards.

## Validate Before Output

Before delivering the final JSON, run an internal verification loop. If any
step fails, throw out the candidate answer and rewrite:

- Strip away conversational text, introductions, markdown fences, and trailing
  explanations.
- Ensure the final output starts with `{` and ends with `}`.
- `risk_play` is either `null` or exactly follows `output-format/`.
- claim ID exists in `claim-catalog.json`.
- match ID exists in `matches.json`.
- any team ID belongs to the selected match.
- any player ID belongs to a team in the selected match.
- no `stake`, `stake_percent`, or `bet_points` fields are present.
- no extra fields are present unless required by the schema.
- if a `strategy` key is present, it is a single-line string with no raw
  newlines or unescaped double quotes.

If any check fails or confidence is unclear, return `risk_play: null` in the
daily answer.

## Output Format Reference

For a normal daily answer, return one raw JSON object that follows
`output-format/daily-submission.schema.json`. Use
`output-format/examples/daily-submission.json` only as a shape reference, never
as a source of IDs or values. Do not wrap the answer in markdown code fences,
quotes, introductory text, or trailing explanation.

If `output-format/` asks for a Risk Play-only response, follow that schema
instead, but keep the same raw-JSON discipline.
