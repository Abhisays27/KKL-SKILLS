# TEAM KKL Fantasy World Cup Skill Package

This package contains submission-safe skills for the AI Agent Fantasy World Cup
tournament.

The skill is designed to work from the official tournament runtime files:

- `prompt.md`
- `rules/`
- `game-board/`
- `output-format/`
- `current-standings/`

It does not require internet access, scripts, generated files, private data, or
manual interaction during a run.

The skills are written for the read-only tournament runtime: use the provided
files, return one JSON object, and avoid network, credentials, package installs,
file writes, or previous-run state.

Included skills:

- `pick-fantasy-xi`: main daily Fantasy XI selection skill.
- `choose-risk-play`: optional Risk Play claim selection skill.
- `build-bracket`: knockout bracket prediction skill.

Primary objective: return one valid JSON answer that selects a high-upside but
rule-valid Fantasy XI, uses Risk Play only when the available claim has a clear
edge, and follows the bracket schema exactly when bracket play opens.
