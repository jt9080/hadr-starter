# Hacker News

The front page and new-story firehose for technical launches, Show HN posts, and
discussion. Free, no auth, no approval. The Algolia-backed search API is the practical
entry point (the official Firebase API exists but is one-item-per-request).

## Endpoint

Verified 8 Jul 2026:

    https://hn.algolia.com/api/v1/search_by_date?tags=story&numericFilters=points>150&query=AI

`search_by_date` sorts newest-first (use for the daily window); `search` sorts by
relevance/points. `numericFilters` accepts `points>N`, `num_comments>N`,
`created_at_i>UNIX`. `tags=front_page` restricts to stories that reached the front page.

## Example response (truncated)

```json
{
  "hits": [
    {
      "objectID": "42150001",
      "title": "GLM 5.2 and the coming AI margin collapse",
      "url": "https://example.com/glm-5-2",
      "points": 666,
      "num_comments": 412,
      "author": "swyx",
      "created_at_i": 1783312800,
      "_tags": ["story", "front_page"]
    }
  ],
  "nbHits": 5387,
  "hitsPerPage": 20
}
```

## Open questions

1. Points and comment count both signal "hot", but they diverge — a 600-point launch
   vs a 400-point flame war with 900 comments. Which is your inclusion signal, and does
   a high comment-to-point ratio mean *controversy* you want to down-rank?
2. `search_by_date` returns everything above the threshold, including week-old stories
   that only just cleared 150 points. Do you window on `created_at_i` (posted in the last
   24h) or on when it *became* hot? These give different digests.
3. A single launch produces several HN submissions (the repo, the blog post, the HN
   discussion) with different `objectID`s and URLs. What ties them to the same story —
   and to the same repo surfaced by the GitHub feed?
