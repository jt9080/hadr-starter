# Blogs & Newsletters

Two jobs. **Lab blogs** are the authoritative source for official releases (the
announcement, not the rumour). **Newsletters** pre-digest the firehose — they've already
read HN, X, arXiv, and GitHub for you, so they're a cheap sanity check that the digest
didn't miss something big.

## Endpoints

Verified 8 Jul 2026 (follow redirects — several 301/307 before serving):

| Source | Feed | Notes |
|---|---|---|
| OpenAI | `https://openai.com/blog/rss.xml` | ✅ official announcements |
| Hugging Face | `https://huggingface.co/blog/feed.xml` | ✅ ecosystem & tooling |
| Simon Willison | `https://simonwillison.net/atom/everything/` | ✅ high-signal practitioner curation |
| Latent Space | `https://www.latent.space/feed` | ✅ agent-focused newsletter |
| smol.ai (AINews) | `https://news.smol.ai/rss.xml` | ✅ daily digest of everything |
| Product Hunt AI | `https://www.producthunt.com/feed?category=artificial-intelligence` | ✅ new AI tools/products |

**Anthropic has no native RSS** (`/rss.xml` and `/news/rss.xml` both 404). Getting its
announcements means scraping the `/news` page or using a third-party bridge — treat it as
a known gap, not a feed.

## Example response (RSS item, truncated)

```xml
<item>
  <title>Introducing our new agent SDK</title>
  <link>https://openai.com/blog/agent-sdk</link>
  <pubDate>Tue, 07 Jul 2026 17:00:00 GMT</pubDate>
  <description>Today we're releasing ...</description>
</item>
```

## Open questions

1. Newsletters overlap heavily with each other and with HN/Reddit — smol.ai's daily
   digest *is* a summary of the same launches. Are they an intake source you rank, or a
   cross-check you diff against (did we miss anything they covered?)? Those are different
   roles in the pipeline.
2. Lab blogs mix product launches, research, hiring posts, and policy. Only some are
   "hot releases". What filters a blog item in — a keyword set, or the LLM judge?
3. Feed formats vary (RSS vs Atom, `pubDate` vs `updated`, full-text vs summary-only) and
   Anthropic has none at all. How much per-source parsing is acceptable before a single
   adapter interface stops paying off — and what does the digest say the morning Anthropic
   ships something the other feeds are slow to echo?
