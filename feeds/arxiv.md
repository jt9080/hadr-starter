# arXiv

The primary source for AI research preprints. Free Atom API. Deep and noisy — hundreds
of papers a day — so treat it as the *research* tier, cross-referenced against Hugging
Face daily papers (`feeds/huggingface.md`) for what the community actually noticed.

## Endpoint

Verified 8 Jul 2026 (**https only** — the `http` host 301-redirects and drops the query):

    https://export.arxiv.org/api/query?search_query=cat:cs.MA&sortBy=submittedDate&sortOrder=descending&max_results=20

Categories relevant to agent development: `cs.MA` (multiagent systems), `cs.AI`,
`cs.CL` (NLP/LLMs), `cs.LG` (machine learning). Combine with `+OR+`:
`search_query=cat:cs.MA+OR+cat:cs.AI`.

**Rate limits bite.** Rapid requests return HTTP 429. arXiv asks for a ~3s delay between
calls and a descriptive User-Agent. One windowed query per run is fine; a loop is not.

## Example response (Atom, truncated)

```xml
<entry>
  <id>http://arxiv.org/abs/2607.04567v1</id>
  <title>Emergent Coordination in LLM Multi-Agent Debate</title>
  <published>2026-07-07T18:02:11Z</published>
  <summary>We study ...</summary>
  <author><name>...</name></author>
  <category term="cs.MA"/>
  <link href="http://arxiv.org/abs/2607.04567v1" rel="alternate"/>
</entry>
```

## Open questions

1. arXiv has no "hotness" signal at all — no stars, no upvotes, just submission time.
   A brand-new preprint and a landmark one look identical in the feed. Without Hugging
   Face daily papers or HN to rank them, how do you decide which papers clear the bar?
2. `submittedDate` ordering includes revisions (`v2`, `v3`) resurfacing old work. Do you
   want first-submission-only, or is a significant revision also news?
3. Papers arrive days before any code, model, or blog post. On the morning a paper is
   the only signal, is a research-only item something the digest reports, or does it wait
   until a repo/model confirms it landed?
