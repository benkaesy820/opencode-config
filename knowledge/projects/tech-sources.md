# tech-sources — curated feeds for staying current

Verified 2026-11-02 with the honest UA (`research-bridge/0.1 (local single-user research client)`). All returned HTTP 200 and parsed as RSS/Atom unless noted.

Use `research_feed { url, limit }` for discovery. Feed items are leads, never evidence: `research_read` the item URL and record a verbatim quote before it appears in a report. Keep `limit` low (3–5) for a daily scan; the summaries are capped at 300 chars.

| Source | Feed | Why | Caveat |
| --- | --- | --- | --- |
| LWN.net | `https://lwn.net/headlines/rss` | Deep kernel/Linux reporting, original | Some articles subscriber-only after a delay |
| This Week in Rust | `https://this-week-in-rust.org/rss.xml` | Best weekly Rust ecosystem digest | Weekly cadence |
| Rust Blog | `https://blog.rust-lang.org/feed.xml` | Authoritative release/roadmap posts | Low volume |
| Rust releases | `https://github.com/rust-lang/rust/releases.atom` | Exact release notes | Notification stream |
| Tokio releases | `https://github.com/tokio-rs/tokio/releases.atom` | Async runtime releases | Notification stream |
| Simon Willison | `https://simonwillison.net/atom/everything/` | Practitioner LLM/agent experiments, primary work | Personal blog, mixed topics |
| Google Project Zero | `https://googleprojectzero.blogspot.com/feeds/posts/default` | Primary vulnerability research | Low frequency |
| Krebs on Security | `https://krebsonsecurity.com/feed/` | Investigative security reporting | Low volume |
| SANS ISC | `https://isc.sans.edu/rssfeed.xml` | Daily operational security diaries | Short tactical posts |
| Cloudflare Blog | `https://blog.cloudflare.com/rss/` | Engineering postmortems, protocol depth | Some marketing |
| Phoronix | `https://www.phoronix.com/rss.php` | Fast Linux kernel/hardware coverage | High volume, occasional rumor |
| Hacker News | `https://news.ycombinator.com/rss` | Community-ranked primary links | High noise; verify at source |
| Lobsters | `https://lobste.rs/rss` | Technical link quality, low noise | Smaller volume |
| Ars Technica | `https://feeds.arstechnica.com/arstechnica/index` | Established tech newsroom | Broad, some consumer noise |
| GitHub Blog | `https://github.blog/feed/` | Official platform engineering/product updates | Some marketing |
| GitHub Changelog | `https://github.blog/changelog/feed/` | Platform changes that affect workflows | Product-specific |
| OpenAI News | `https://openai.com/news/rss.xml` | Primary lab announcements | Huge archive; limit by date |
| Hugging Face Blog | `https://huggingface.co/blog/feed.xml` | ML engineering posts | High volume, mixed quality |

## Patterns

- **Any GitHub project:** `https://github.com/<owner>/<repo>/releases.atom` and `.../commits.atom` are zero-key and avoid the 60 requests/hour REST limit.
- **kernel.org releases:** `https://www.kernel.org/releases.json` is JSON, not a feed. Use `research_read` on it; `research_feed` will reject it.
- **Scholaric depth:** `research_lookup` (Crossref) for DOI/title/authors/year/abstract; `research_read` the DOI landing page or PDF before quoting.

## A daily set that stays cheap

LWN, This Week in Rust, Rust Blog, Simon Willison, Phoronix, Hacker News, Krebs, Cloudflare. Eight `research_feed` calls at `limit: 3` are a few hundred lines of titles and summaries; only read the items that matter.

## Limits

`research_feed` supports RSS 2.0 and Atom only — not RSS 1.0/RDF, JSON Feed, enclosures, or polling. No subscriptions, no ranking, no storage. It caches parsed results in-process for 10 minutes.
