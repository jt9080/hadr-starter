# GitHub

The truest signal for what developers are actually adopting. `github.com/trending` is
the page you want — but it has **no API** (it's server-rendered HTML). The Search API is
the supported proxy: ask for repos created recently, sorted by stars.

## Endpoint

Verified 8 Jul 2026:

    https://api.github.com/search/repositories?q=agent+created:>2026-06-01&sort=stars&order=desc&per_page=20

Rate limits: **60 requests/hour unauthenticated**, **5000/hour** with a token
(`Authorization: Bearer <PAT>`). Set `Accept: application/vnd.github+json`. The `q`
grammar supports `created:>DATE`, `pushed:>DATE`, `topic:llm`, `language:python`, and
free-text terms (`agent`, `mcp`, `subagent`).

## Example response (truncated)

```json
{
  "total_count": 204013,
  "items": [
    {
      "full_name": "omnigent-ai/omnigent",
      "html_url": "https://github.com/omnigent-ai/omnigent",
      "description": "Omnigent is an open-source AI agent framework",
      "stargazers_count": 6653,
      "created_at": "2026-06-14T09:12:00Z",
      "pushed_at": "2026-07-08T02:40:00Z",
      "topics": ["agents", "multi-agent", "llm"],
      "language": "Python"
    }
  ]
}
```

## Open questions

1. Absolute star count rewards old, famous repos; what you want is *velocity* — stars
   gained in the last 24h. The API gives you a snapshot total, not a delta. Where does
   the previous total come from — do you store yesterday's numbers to compute the jump?
2. `created:>DATE` finds new repos, but a two-year-old framework can go viral on a new
   release and never show up. Do you also poll by `pushed:` or by a watch-list of known
   projects, and how do those two intake paths get de-duplicated?
3. Search across every AI term (`agent`, `mcp`, `llm`, `rag`, …) returns overlapping
   result sets and burns the rate limit. What's the minimal query set that covers your
   interests without hitting 60/hr — and does that force using a token?
