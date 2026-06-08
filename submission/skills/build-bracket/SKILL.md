---
name: build-bracket
description: Build valid knockout bracket predictions from the official bracket, match, team, standings, and output-format files.
---

# Build Bracket

Use this skill only when the organizer prompt explicitly asks for bracket
prediction, knockout picks, round winners, finalists, champion, or tournament
winner and `game-board/bracket.json` is present for the run.

## Hard Rules

Follow `output-format/` exactly. Use only official team IDs, match IDs, round
IDs, and bracket fields from the tournament files.

Never invent teams, paths, or metadata. Do not override organizer-owned bracket
structure. If a future matchup depends on an earlier pick, propagate only the
team chosen through that official path.

Bracket Play is separate from daily Fantasy XI and Risk Play. Bracket picks
contribute only after bracket play opens and relevant results are known. Once
bracket picks lock, do not assume they can be changed.

Current bracket scoring reference:

- Correct Round of 32 winner: +5
- Correct Round of 16 winner: +8
- Correct quarterfinal winner: +12
- Correct semifinal winner: +18
- Correct champion: +30

If the bracket board includes additional matches such as a third-place match,
follow the bracket file and organizer instructions for whether a pick is
required and whether it scores.

If the prompt is a normal daily Fantasy XI task, do not run bracket logic and
do not omit the required daily `fantasy_xi` array.

Runtime constraints:

- Use only the provided runtime files: `rules/`, `game-board/`,
  `output-format/`, `current-standings/`, `team/README.md`, and
  `team/skills/`.
- Do not use internet research, scripts, package installs, credentials,
  cookies, private data, or files outside the tournament workspace.
- Do not create, edit, delete, or move files during the run.
- Do not depend on memory, cache, previous runs, or previous bracket state.
- Keep the process direct enough to finish within the 5-minute runtime limit.

## Input Reading Order

1. Read `prompt.md` for the bracket task and required output keys.
2. Read `output-format/` for the exact JSON schema.
3. Read `rules/` for bracket scoring and validity rules.
4. Read `game-board/bracket.json`, `matches.json`, `teams.json`,
   `standings-before.json`, and any relevant matchday files.

## Prediction Method

Rank each matchup by advancement probability, not by reputation alone.

Use available board evidence in this order:

- current tournament form and points
- goals for, goals against, and clean-sheet record if present
- opponent strength and path difficulty
- player availability or squad strength fields if present
- recent match metrics included in the board
- tie-break or knockout-specific notes from the official files

When data is sparse, choose the team with the stronger current board profile and
more stable path. Mark close calls in the strategy string rather than inventing
statistics.

## Probability Labels

Use internal confidence labels to guide picks:

- Confident: clear board edge, roughly 65% or higher.
- Lean: meaningful but not decisive edge, roughly 55% to 65%.
- Coin flip: less than 55%; choose the cleaner path or higher-upside bracket
  leverage only when standings require chasing.

For bracket contests, avoid excessive upset hunting when leading or close to the
lead. If far behind, one or two coherent upset paths can create leverage, but
the champion pick should still be defensible from the board.

## Validate Before Output

Before delivering the final JSON, run an internal verification loop. If any
step fails, throw out the candidate answer and rewrite:

- Strip away conversational text, introductions, markdown fences, and trailing
  explanations.
- Ensure the final output starts with `{` and ends with `}`.
- all required bracket keys from `output-format/` are present
- every team ID exists in `teams.json`
- every match or round ID exists in the official bracket files
- winners advance through valid paths
- champion is one of the propagated finalists
- no extra top-level keys are included unless required by the schema
- if a `strategy` key is present, it is a single-line string with no raw
  newlines or unescaped double quotes

If the prompt is a normal daily Fantasy XI task and not a bracket task, do not
produce bracket fields.

## Output Format Reference

For bracket tasks, dynamically extract and copy the exact keys from
`output-format/`. Do not copy placeholder field names from this file. The final
answer must still be one raw JSON object. Do not wrap it in markdown code
fences, quotes, introductory text, or trailing explanation.

Do not return an empty `bracket` object. Build every bracket key and value from
the active `output-format/` files and official bracket data. If the bracket
schema uses different top-level keys, obey `output-format/` exactly.
