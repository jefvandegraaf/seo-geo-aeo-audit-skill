# DATA-SOURCES.md

Which data feeds improve which checks, and what breaks without them.

Nothing here is required. The audit degrades gracefully — every check that depends on missing data gets marked `SKIPPED — no data source` in the output rather than guessed at. **A skipped check is an honest result. An estimated one is a lie with a number attached.**

To run the audit with no access at all — a prospect, a competitor, a pre-sales teardown — see `COMPETITOR-TEARDOWN.md`.

---

## 1. Search console data
*Any first-party webmaster tool from a search engine you care about.*

**Feeds:** §1.3 sampling, §2.4 legacy URLs, §2.6 index coverage and cannibalisation, §5.1 freshness prioritisation, §5.5 opportunity analysis, §6.3 query set construction.

The highest-value source in the audit. It's the only place you learn what the engine actually *did* with each URL — what it indexed, what it crawled and rejected, and which queries it thinks each page answers. Everything else in this document is inference; this is the engine telling you directly.

Get it for every engine your target assistants resolve against, not just the dominant one. Sites routinely discover they're absent from a secondary index entirely, which silently zeroes out citations in whichever assistants depend on it.

**Minimum useful export:** query/page/impressions/clicks/position over 90 days, plus the full index-coverage report.

**Without it:** §5.5 is impossible, §6.3's query set becomes guesswork, and sampling falls back to analytics plus judgement. You lose roughly a third of the audit's value.

---

## 2. Third-party SEO data platforms
*Ahrefs, Semrush, Moz, Majestic, SE Ranking, Sistrix, or equivalent — via API or scheduled export.*

**Feeds:** §1.3 sampling, §2.5 external link graph, §5.4 authority signals, §5.5 opportunity analysis, §6.3 query and competitor set construction, and the whole of `COMPETITOR-TEARDOWN.md`.

These platforms provide one thing that has **no substitute anywhere**: a backlink index. Search console shows a partial view of your own links and nothing at all about anyone else's. If you want to know whether a citation gap is a content problem or an authority problem, this is the only way to find out — and that distinction changes the entire remediation plan.

What they contribute, roughly in order of value:

- **Backlink index** — referring domains, link velocity, which specific pages attract links, and link gap versus competitors. Unavailable from any first-party source.
- **Competitor keyword and traffic estimates** — builds the §6.3 query set and the competitor set when you have no search console access
- **Keyword volume and difficulty** — query set construction, and demand-sizing content gaps
- **SERP feature tracking** — most platforms now track whether a query triggers an AI answer panel and which domains it pulls from. That's directly relevant to §6.3 and it's tracked at a scale you can't replicate by hand.
- **Historical rank tracking** — substitutes for §5.5 lost-query analysis when search console history is short or missing
- **Site audit crawlers** — overlap §2 through §4 and can replace running your own crawler
- **Unlinked brand mention monitoring** — feeds the brand-association half of §6.3

**Label the estimates as estimates.** Keyword volume, difficulty scores, and competitor traffic figures are modelled, not measured — they're derived from clickstream panels and proprietary weighting, and they differ substantially between vendors. Backlink counts differ too, because each platform runs its own crawler with its own index and its own definition of a link worth keeping. Numbers from two tools are not comparable, and neither is a measurement. Use them for **relative** comparison and trend, never as an absolute figure in a client report.

**Without it:** the audit still runs. What you lose is the ability to separate "our content is worse" from "their domain is stronger", plus most of the competitor teardown's authority section. On a small or new site that gap matters less than people assume — original data and answer-first structure (§6.2) beat authority on long-tail and question-form queries more often than they used to.

---

## 3. Web analytics
*Any analytics platform that reports referrer hostnames. Privacy-focused tools work fine and often better — no consent-banner sampling loss.*

**Feeds:** §1.3 sampling, §5.3 purposeless page detection, §6.4 AI referral measurement.

**The referrer segment is the point.** Filter traffic by assistant hostnames and you have the only hard evidence in the entire GEO half of this audit that a human actually arrived from an AI answer.

Requirements: referrer hostname reporting, per-page views, an engagement metric, and ideally an API or scheduled export.

**Without it:** §6.4 is gone entirely. You can still measure crawler *arrival* from logs, but not human *arrival* from citations. Sampling falls back to search console.

---

## 4. Server / edge logs
*Raw access logs from your origin, CDN, or edge platform.*

**Feeds:** §1.3 sampling, §2.2 edge blocking verification, §6.4 crawl-versus-referral reconciliation.

The only place AI crawler traffic is visible. Search console doesn't show it. Analytics doesn't show it — bots don't execute the tracking script.

Extract, per user-agent: request count, unique URLs fetched, status codes returned, bytes served, and first/last seen. A crawler receiving `403`s at volume is the finding.

**Without it:** you can still *test* edge blocking by fetching with spoofed user-agents (§2.2), which catches most cases. But you lose crawl-frequency trends and can't distinguish "never crawled" from "crawled and not cited" — a distinction that changes the entire remediation plan.

---

## 5. Field performance data
*Real-user monitoring, or the public field-data dataset your search engine publishes.*

**Feeds:** §3.2.

Lab scores are synthetic. Field data is what users experienced. Perfect lab with poor field is common — usually a slow origin, a heavy third-party script, or a device and geography mix the lab test didn't simulate.

**Without it:** fall back to lab testing and label it as such. Lowest-impact gap in this list; performance is a ranking and crawl-budget input, not a citation input.

---

## 6. Source control / CMS revision history
*Git history, or your CMS's revision timestamps.*

**Feeds:** §2.3 accurate `lastmod`, §5.1 freshness.

Deploy timestamps are not modification timestamps. A build system that stamps every file with the build date makes `lastmod` meaningless, and engines learn to ignore the field across your whole domain. Derive real dates from actual content-change history.

**Without it:** freshness analysis becomes manual sampling. Reliability drops sharply on large sites.

---

## 7. Assistant access
*Working accounts on each assistant you're testing.*

**Feeds:** §6.3.

Manual browser runs are more reliable than API access here — the API and the consumer product often use different retrieval paths, and you care about what real users see. Use clean sessions, logged out where possible, and route through the target market's geography.

**Without it:** §6.3 is gone. Every other GEO check still runs — §6.1 and §6.2 are structural analyses of your own content and need nothing external.

---

## Access checklist

Record what you have before starting. Put this table at the top of the output.

| Source | Available | Scope / window | Notes |
|---|---|---|---|
| Search console (primary engine) | ☐ | | |
| Search console (secondary engines) | ☐ | | |
| Third-party SEO platform | ☐ | | which vendor? API or export? |
| Web analytics | ☐ | | referrer reporting confirmed? |
| Server / edge logs | ☐ | | retention window? |
| Field performance data | ☐ | | |
| Source control history | ☐ | | |
| Assistant accounts | ☐ | | which ones? |

---

## Priority if you're only getting one or two

1. **Search console** — nothing else replaces knowing what the engine did
2. **Analytics with referrer data** — the only proof AI citations produce visits
3. **Third-party SEO platform** — the only source of backlink and competitor data
4. **Server logs** — the only source of crawler behaviour

The first two are usually free and usually already installed. The third is the first thing worth paying for.
