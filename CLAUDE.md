# CLAUDE.md

## What this project is

HADR Monitor — an agent that watches live disaster feeds (GDACS, USGS, ReliefWeb),
filters noise, assesses what remains (what happened, where, how bad, who is affected),
and publishes a morning situation report to `dashboard.html` at 08:30 Singapore time
(Asia/Singapore). It runs on a schedule, unattended, and stays quiet when nothing
has changed.

## Data feeds

Full details and example responses live in `feeds/`. Key facts:

- **GDACS** (`feeds/gdacs.md`) — multi-hazard GeoJSON event list:
  `https://www.gdacs.org/gdacsapi/api/events/geteventlist/EVENTS4APP`.
  Events carry colour-coded alert levels (Green/Orange/Red) and an `alertscore`.
- **USGS** (`feeds/usgs.md`) — real-time earthquake GeoJSON, regenerated every minute:
  `https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson`.
  Other rolling windows exist (`all_hour`, `4.5_week`, `significant_month`, …).
- **ReliefWeb** (`feeds/reliefweb.md`) — curated, slower-moving. The v2 API requires a
  **pre-approved appname** (403 without one) and v1 is decommissioned (410).
  The RSS feed needs no approval: `https://reliefweb.int/disasters/rss.xml`.

## Repository layout

- `scripts/` — deterministic checks; anything that must give the same answer twice
  does not belong in a prompt
- `skills/` — one folder per skill: `SKILL.md`, supporting assets, and a note on
  which model each step should use
- `docs/solutions/` — reusable write-ups of solved problems; check here before
  re-debugging something
- `implementation-notes.md` — kept by the agent, reviewed by the human; one entry
  per working block (decisions, open questions, deviations)
- Expected end artefacts: `prd.html` · `system-view.html` · `dashboard.html` ·
  `goal.md` · at least one skill

## Language & tooling

Not yet decided — record the choice here once the first slice is built.

## Test command

Not yet defined — record it here once tests exist.

## Conventions

- Prefer the documented feed endpoints in `feeds/`; they were verified 6 Jul 2026
- Deterministic logic goes in `scripts/`, not in prompts
- The dashboard build must be idempotent: no changes in the feeds means no changes
  to `dashboard.html`

## Deviations policy

Anything built that departs from the PRD or this file is recorded in
`implementation-notes.md` under **Deviations**, with the reason.
An undocumented deviation is a bug.
