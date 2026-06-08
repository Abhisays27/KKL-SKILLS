# Strategy Notes

## Fantasy XI Priorities

Scoring rewards starts, 60+ minutes, goals, assists, defender/GK clean sheets,
and goalkeeper saves. Penalties are yellow cards, red cards, and own goals.

Use these tie-breakers:

1. Confirmed or likely starter beats uncertain higher-ceiling player.
2. Player with goal/assist path beats defensive-only player unless clean-sheet
   probability is strong.
3. Defender/GK from a strong favorite beats defender/GK from an even match.
4. Set-piece taker beats similar open-play creator.
5. Forwards and attacking midfielders from high-total matches get priority.
6. Avoid players flagged unavailable, suspended, injured, or low-minute risk.

## Position Heuristics

Goalkeeper:

- The lineup must contain exactly 1 goalkeeper. Do not compare goalkeeper value
  against outfield value in a way that removes the mandatory GK slot.
- There is no player budget. Do not settle for a cheaper or lower-quality
  goalkeeper for cost reasons.
- Prioritize the highest clean-sheet probability first.
- Use save upside as a secondary path when the goalkeeper faces shots but is not
  a heavy underdog.

Defenders:

- Prioritize clean sheet plus attacking involvement.
- Fullbacks and set-piece aerial threats beat center-backs with no attacking
  role when clean-sheet odds are similar.
- Use 4 or 5 defenders only when multiple defenders have real clean-sheet paths.

Midfielders:

- Prioritize goals, assists, set pieces, penalty involvement, and 60+ minute
  reliability.
- Reliable 60+ minute attacking midfielders beat volatile wingers if the winger
  is a rotation or early-substitution risk.

Forwards:

- Prioritize goal probability, minutes security, penalty role, and opponent
  defensive weakness.
- Use three forwards when three eligible forwards have strong starter and goal
  paths. Use one or two forwards when the forward pool is weak or uncertain.

## Formation Heuristics

Use an expected-value mindset:

- Attacking formations such as 3-4-3 or 4-3-3 are best when the board has
  several likely starters with goal or assist paths.
- Balanced 4-4-2 is best when player scores are close across positions.
- Defensive 5-3-2 or 5-4-1 is best only when clean-sheet paths are unusually
  strong and extra attackers are low quality or low minutes.
- The 60-minute bonus matters. A safe 60+ minute player often beats a high-name
  substitute or rotation risk.

## Phase-Specific Adjustments

Group stage:

- Prioritize reliable starters and 60+ minute players because baseline points
  are a large share of expected value.
- Group-stage matches end after regulation plus stoppage time. Do not project
  extra-time or penalty-shootout events in group-stage decisions.

Knockout stage:

- Regulation plus extra time can expand scoring windows. If the board strongly
  suggests a match may go to extra time, a high-impact attacking substitute from
  a strong team can become viable.
- Still avoid likely DNP players. A substitute only beats a starter when there
  is clear evidence of minutes, attacking role, and extra-time upside.
- Penalty shootout goals count only for explicit penalty-shootout claims. Do
  not treat shootout goals as normal Fantasy XI or player-event scoring unless
  the active rules or claim schema explicitly says so.

## Risk Play Heuristics

Skip Risk Play if the claim edge is unclear.
If Risk Play is skipped, missing, invalid, or `null`, no previous Risk Play is
reused.

Check `standings-before.json` before deciding. Risk Play is based on team
points before the matchday. If the team is missing from standings, treat it as a
new 50-point team with unknown rank rather than skipping solely for being
unranked. Use lower-risk claims when protecting rank and only increase risk when
standings show the team must climb.

Risk posture:

- rank 1 to 3 or less than 10 points behind leader: usually skip or use only a
  very strong Green claim
- rank 4 to 10 or 10 to 20 points behind leader: consider Green claims; use
  Yellow only with unusually strong evidence
- rank 11+ or 21+ points behind leader: consider Yellow claims; use Red only
  when the board strongly supports it
- team points at 0 or below: always skip because the computed stake is 0

Green claims are the default when using Risk Play. Prefer:

- match has 2+ total goals
- goal before halftime
- match has 2+ total cards, when both teams are physical or the referee profile
  is strict

For card claims, total-card claims count yellow, second-yellow red, and red
cards. Yellow-card claims count only yellow cards. Red-card claims count red and
second-yellow red cards.

If provider data is missing or ambiguous, use the official game board as source
of truth and avoid inventing facts. Organizer scoring decisions are final after
review.

Yellow claims require stronger evidence. Prefer:

- both teams score in balanced high-total matches
- selected team scores first when one team is a strong favorite
- selected player scores only when the player is a likely starter with strong
  goal role

Red claims are rare. Use only for exceptional favorites or clear knockout
contexts. Avoid exact score unless the board strongly supports one low-variance
scoreline.

Claim priority when several claims look valid:

1. `match_2plus_goals`
2. `goal_before_halftime`
3. `team_scores_first` for a heavy favorite
4. `player_scores` for a likely starter with a strong goal role
5. Red claims only when chasing and evidence is exceptional

## Final Answer Checklist

Return one plain JSON object with `team_id`, `matchday_id`, `fantasy_xi`,
`risk_play`, and `strategy`.

Never include stake fields. The tournament computes the stake.

If unsure about any Risk Play field, set `risk_play` to `null` and protect the
Fantasy XI score.
