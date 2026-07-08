# Remaining feeds — Hugging Face, arXiv, Reddit, blogs/newsletters

**Date:** 2026-07-08
**Status:** Agreed
**Source of truth:** `prd.html` (§2 feed catalog, §7 decisions), `CLAUDE.md`,
`feeds/huggingface.md`, `feeds/arxiv.md`, `feeds/reddit.md`,
`feeds/blogs-newsletters.md`.

## Goal

Complete the feed catalog. HN and GitHub are live; add the remaining four
sources behind the existing `fetch(window_start, window_end) -> FetchResult`
seam so the LLM judge (already feed-agnostic) has the full firehose to cluster
and rank. Each feed degrades independently — a flaky source drops out with a
banner, never crashes the run. Stdlib only.

## Decisions carried in

From the brainstorming pass:

- **Signal is one input.** Feeds without a numeric signal carry
  `signal_value=0` and an honest `signal_name` (`recency`, `top`, `editorial`).
  They flow to the judge like any feed; the judge weighs them on content.
  Velocity is 0 for them (meaningless); repeat-suppression still works because
  it keys on `reported_at`, not signal.
- **Reddit via RSS**, no auth. Accept `top/day` ordering as the signal; no OAuth
  app, no second secret.
- **arXiv** scoped to `cat:cs.MA OR cat:cs.AI`, `submittedDate` desc,
  first-submission only (skip `v2+` revisions).
- **Newsletters = intake** (feed the judge). The "did we miss anything?"
  cross-check role is deferred to S4.
- **One PR**, built incrementally (TDD per adapter, fixtures per source).

## Deliberate deferrals

- **Reddit upvote counts / OAuth** → later, only if RSS ordering proves too weak.
- **Newsletter cross-check + retraction corrections** → S4.
- **Anthropic blog** → known gap (no native RSS), not a feed.
- **Raw-arXiv beyond cs.MA/cs.AI** → out; HF `daily_papers` covers community-noticed
  papers with a real upvote signal.

## Architecture

One shared parser + four thin adapters, mirroring the existing `hackernews.py` /
`github.py` pattern.

| Module | Role |
|---|---|
| `feedparse.py` | **New.** Stdlib `xml.etree.ElementTree` reader for both RSS (`<item>`, `pubDate`) and Atom (`<entry>`, `updated`/`published`). Returns normalized `FeedEntry`-like dicts: `title`, `link`, `published` (aware UTC datetime or None), `summary`. Pure function over bytes — no network. |
| `huggingface.py` | **New.** JSON adapter. Two endpoints: `api/models?sort=trendingScore` (signal `likes`) and `api/daily_papers` (signal `upvotes`; paper carries arXiv id → good clustering key). Source id `huggingface`. |
| `arxiv.py` | **New.** Atom via `feedparse`. `export.arxiv.org/api/query?search_query=cat:cs.MA+OR+cat:cs.AI&sortBy=submittedDate&sortOrder=descending`. Signal `recency`=0. https only, descriptive UA. Source id `arxiv`. |
| `reddit.py` | **New.** RSS via `feedparse` across r/LocalLLaMA, r/MachineLearning, r/AI_Agents (`top/.rss?t=day`). Signal `top`=0. Small delay between the 3 same-host calls; descriptive UA. Source id `reddit`. |
| `blogs.py` | **New.** RSS/Atom via `feedparse` across the 6 sources in `feeds/blogs-newsletters.md`. Signal `editorial`=0. Follow redirects. Source id `blogs`. |
| `render.py` | Add source badges (HF, arXiv, Reddit, Blog); for value-0 feeds show the badge + age but omit the meaningless `0`. |
| `run.py` | Fetch all six feeds; cap each feed's post-relevance contribution to bound the judge payload; everything else unchanged. |

Each adapter maps its raw items to `Candidate`s and filters to the window by
`published` (where the source dates items). HF trending models are a "now"
snapshot with no per-item window date, so they are taken as-is (not
window-filtered); everything else is filtered to `[window_start, window_end]`.

## Data flow (`run.py`)

```
window
  → [hackernews, github, huggingface, arxiv, reddit, blogs].fetch()   # each degrades
  → relevance pre-filter (all feeds)
  → cap per feed (PER_FEED_CAP, ~20) to bound the judge payload
  → state.load → ingest (velocity/memory; velocity 0 for value-0 feeds)
  → judge  ── fails ──▶ rank.select (stand-in)  + banner
  → render → state.save + append_run
```

`runs.json` feed-health now records all six sources plus `judge`.

## Signal & Candidate mapping

| Feed | source | signal_name | signal_value | discussion_url | summary |
|---|---|---|---|---|---|
| Hugging Face (models) | huggingface | likes | likes | model page | model id |
| Hugging Face (papers) | huggingface | upvotes | upvotes | — | paper summary |
| arXiv | arxiv | recency | 0 | — | abstract |
| Reddit | reddit | top | 0 | thread url | — |
| blogs | blogs | editorial | 0 | — | item description |

`created_at` = the item's published time (HF models: fetch time, since trending
is a snapshot). Relevance runs over title + url + summary, so editorial/abstract
text gives the allowlist something to match.

## Payload bounding

Six feeds can yield 100+ candidates; the judge payload (and latency/cost) grows
with it. After the relevance filter, keep at most `PER_FEED_CAP` (~20) per feed —
numeric feeds by signal desc, value-0 feeds by recency — before handing the set
to the judge. A one-line tunable constant. Logged in the run summary so a silent
truncation is visible.

## Error handling

- Every adapter returns `FetchResult(status="failed")` on any error (network,
  HTTP, timeout, malformed XML/JSON). One dead feed never fails the run; its
  banner names it, the others still publish. (Existing pattern, now ×6.)
- `feedparse` never raises on malformed XML — a bad document yields `[]`.
- Politeness: descriptive User-Agent on every request; one query per feed per
  run (arXiv/Reddit are 429-sensitive); a short delay between Reddit's 3
  same-host calls. No tight loops.

## Testing (TDD, all offline)

Fixtures per source under `tests/fixtures/`; mock `urlopen`.

- `test_feedparse.py` — parses RSS (`<item>`/`pubDate`) and Atom
  (`<entry>`/`updated`); missing fields tolerated; malformed XML → `[]`.
- `test_huggingface.py` — models JSON → likes signal; daily_papers JSON →
  upvotes signal + arXiv id; malformed skipped; failure → degraded.
- `test_arxiv.py` — Atom entry → Candidate (recency=0); v2+ revision skipped;
  window filter; failure → degraded.
- `test_reddit.py` — RSS entry → Candidate (top=0); per-sub aggregation; failure
  of one sub degrades the whole feed gracefully; window filter.
- `test_blogs.py` — mixed RSS/Atom across sources → Candidates (editorial=0);
  one bad source doesn't sink the rest.
- `test_render.py` — value-0 feeds render the badge without a `0`; new badges.
- `test_run.py` — six feeds wired; per-feed cap applied; one feed failing still
  publishes; `runs.json` records all six.

## Success criteria

- `python run.py` fetches all six feeds; a judged digest can include a Reddit
  thread, an arXiv paper, an HF model, and a lab-blog post — clustered where they
  are the same story.
- Any single feed failing degrades to a banner; the run still publishes.
- The judge payload stays bounded (per-feed cap) so latency stays under the
  timeout.
- All `unittest` tests green on Python 3.9, no network in the suite.
