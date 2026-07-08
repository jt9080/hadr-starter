# Hugging Face

Two feeds worth watching: **trending models** (new weights and releases) and **daily
papers** (a human-curated slice of arXiv, already filtered for what the ML community
cares about). Both are free and need no auth.

## Endpoints

Verified 8 Jul 2026:

    https://huggingface.co/api/models?sort=trendingScore&limit=20
    https://huggingface.co/api/daily_papers?limit=20

Models also sort by `downloads`, `likes`, `createdAt`. Daily papers carries the
underlying arXiv id, so it doubles as a relevance filter over `feeds/arxiv.md`.

## Example response (models, truncated)

```json
[
  { "id": "zai-org/GLM-5.2",        "downloads": 281584, "likes": 3596 },
  { "id": "InternScience/Agents-A1","downloads": 14723,  "likes": 375  },
  { "id": "baidu/Unlimited-OCR",    "downloads": 1084945,"likes": 1834 }
]
```

## Example response (daily papers, truncated)

```json
[
  { "paper": { "id": "2607.01234", "title": "MuseBench: Benchmarking Intent-Level ...",
               "upvotes": 92, "summary": "..." } }
]
```

## Open questions

1. `trendingScore` is opaque — Hugging Face computes it, you don't. A model with 1.7M
   downloads and one with 375 likes both surface. What makes a model release *report-worthy*
   for you: a new base model? a new agent-tuned checkpoint? Raw popularity alone will
   surface OCR and image models you don't care about.
2. Daily papers is curated but broad (audio, video, vision). Your interest is agent-shaped
   (multiagent, tool use, memory, planning, evals). Do you filter daily papers by topic, or
   is the community curation itself enough of a filter?
3. A model release, its paper, its GitHub repo, and its HN thread are the same story across
   four feeds. Hugging Face gives you the model id and (via daily papers) the arXiv id —
   is that enough to join them, or do you still need fuzzy title matching?
