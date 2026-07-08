# Reddit

Practitioner chatter — releases, benchmarks, and honest reactions before they reach the
newsletters. Relevant subs: `r/LocalLLaMA`, `r/MachineLearning`, `r/AI_Agents`.

## Endpoint

Two paths, both verified 8 Jul 2026:

- **JSON API** — `https://www.reddit.com/r/LocalLLaMA/top.json?t=day&limit=25`.
  Returns **HTTP 403 without OAuth** from a plain/datacenter client (Reddit now gates the
  JSON API; a registered app + OAuth token is required).
- **RSS/Atom** — `https://www.reddit.com/r/LocalLLaMA/top/.rss?t=day`. No auth needed.
  Rate-limit sensitive: rapid or parallel requests return **HTTP 429**. Space them out
  and send a descriptive User-Agent.

Use RSS unless/until you register a Reddit app. `t=day` = top of the last 24h;
`t=hour`/`t=week` also exist.

## Example response (RSS/Atom, truncated)

```xml
<entry>
  <title>New open multi-agent framework beats AutoGen on tau-bench</title>
  <link href="https://www.reddit.com/r/LocalLLaMA/comments/abc123/..."/>
  <updated>2026-07-08T01:14:00Z</updated>
  <author><name>/u/someone</name></author>
  <content type="html">... upvotes and comments live in the linked page ...</content>
</entry>
```

## Open questions

1. The RSS feed gives you titles and links but **not the upvote/comment counts** —
   which are exactly the signal you'd rank on. Do you accept RSS ordering (already
   sorted by "top") as the signal, or is that the reason to register an OAuth app?
2. Reddit is opinion-heavy: a highly-upvoted "is anyone else disappointed by X" thread
   isn't a release. How do you separate *a thing shipped* from *people talking about a
   thing*? This is where the LLM judge earns its place.
3. The same launch appears on `r/LocalLLaMA` and `r/AI_Agents` and Hacker News within
   hours. Reddit posts carry no GLIDE-style id — what joins a Reddit thread to the same
   story elsewhere beyond fuzzy title/URL matching?
