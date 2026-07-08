# AI News Monitor

A monitoring agent for AI/ML releases, research, and discourse — an 08:00 morning digest
for someone building agentic systems.

## The end state

By Wednesday afternoon this repository contains an agent that:

- watches live AI feeds — Hacker News, GitHub, Hugging Face, arXiv, Reddit, and a set of
  lab blogs/newsletters (see `feeds/`)
- filters the firehose and ranks what remains: what shipped, why it matters, who it's for —
  agent-development news first, the rest of AI after
- publishes a morning digest to `dashboard.html` at 08:00 Singapore time (5–8 items)
- runs on a schedule, unattended, and suppresses anything it has already shown you

How it does any of that is not specified anywhere in this repository. That is the course.

## The three days

1. **Plan** — interrogate the feeds, write the PRD, cut it into vertical slices
2. **Autonomy** — build the first slice, write a skill, wire up the 08:30 routine, launch the overnight loop
3. **Trust** — review code you didn't write, harden the pipeline, demo

## Artefacts expected by the end

`prd.html` · `system-view.html` · `implementation-notes.md` · `dashboard.html` · `goal.md` · at least one skill

## Day 1 setup

1. Sign in to Claude Code with your Team seat
2. Create your own repository from this template, then clone it
3. Run `/install-github-app` so @claude reviews your pull requests from Day 2
4. Install OpenCode and sign in with your Go key

Fill in `CLAUDE.md` before your first prompt.
