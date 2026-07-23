# COMPETITOR-TEARDOWN.md

How to run this audit against a site you don't own.

Everything here uses public information only. No credentials, no access grants, no cooperation from the target. It works for three situations:

- **Pre-sales.** Audit a prospect before you ask them for anything. This is usually the deliverable that sells the engagement.
- **Competitive research.** Find out why someone else gets cited for your queries.
- **Benchmarking.** Establish what "good" looks like in your category before you set targets.

The competitive use is the one most people skip and shouldn't. Running §6.1 and §6.2 against whoever *is* getting cited usually explains why in about ten minutes.

---

## 1. Find the actual competitor set

**Don't start from a list.** The competitors you'd name are usually not the sources getting cited.

Run the §6.3 assistant queries first, before any structural work. Collect every cited domain across every run. **That list is your competitor set** — empirically derived, not assumed.

Then classify what you find, because the response to each type is completely different:

| Source type | What it means | Response |
|---|---|---|
| Direct competitor's own site | Straight fight, winnable on content | Full teardown, close the gaps |
| Review aggregator / directory | Assistant trusts the category page over any vendor | Get listed and optimise the listing; you probably can't outrank it |
| Forum or community thread | Assistant is weighting first-hand experience | On-site work won't fix this. You need presence where the discussion happens. |
| Trade press / industry publication | Editorial authority | Pursue coverage, contribute, get quoted |
| Documentation or standards body | Primary source | Cite it yourself; don't try to replace it |
| Video platform | Format preference for this query | Different content type entirely |

**If aggregators and forums dominate your queries, stop the teardown.** No amount of on-site optimisation changes that outcome, and telling a client otherwise is how you lose them in month four. The finding *is* the deliverable.

---

## 2. Audit the cited page, not the homepage

The most common teardown mistake. When an assistant cites `competitor.com/guide/thing`, that specific URL is what earned the citation. The homepage tells you almost nothing.

For each cited URL, run:

- §3.1 — raw versus rendered. Sometimes the answer is boring: they ship server-rendered HTML and you don't.
- §4.1, §4.2 — heading structure, title/H1 alignment, structured data completeness
- §4.3 — entity clarity
- §5.4 — named author, credentials, dates, outbound citations to primary sources
- §6.1 — **chunk self-containment percentage**
- §6.2 — **quotable sentence count, and the top five extractable sentences**

§6.1 and §6.2 are the payoff. Pull the five sentences a model would most likely take from their page, then pull the five from your equivalent page, and put them side by side. That comparison is more persuasive than any score, and it usually makes the fix obvious to a non-technical client in about thirty seconds.

---

## 3. Compare like for like

Map each cited competitor URL to your closest equivalent page. If you don't have one, that's the finding — record it as a content gap and move on.

For each pair, tabulate:

| Dimension | Them | You |
|---|---|---|
| Word count, unique body copy | | |
| Chunk self-containment % | | |
| Quotable sentences | | |
| Original data points (theirs, not cited from elsewhere) | | |
| Outbound citations to primary sources | | |
| Named author with credentials | | |
| Last meaningful update | | |
| Answer-first or preamble-first | | |
| Comparison tables | | |
| Structured data completeness | | |

Most of the time the gap is not length, design, or technical quality. It's **original data and answer-first structure**. Someone published a number nobody else had, in a sentence that survives extraction, and now they own the query.

---

## 4. Freshness and authority context

Two things that explain outcomes the on-page comparison can't:

**Freshness.** Check `dateModified` in structured data, visible dates, and archive snapshots to see whether the page was genuinely updated or just restamped. A page that gets meaningfully revised twice a year beats a better page that hasn't moved since 2023.

**Domain authority.** This needs a third-party backlink index (see `DATA-SOURCES.md` §2) — it's the one input in this entire teardown that has no free substitute. Compare referring domains, link velocity, and which specific pages attract links. If the citation gap is purely an authority gap, on-page fixes have a ceiling, and the honest recommendation is a link and PR strategy rather than a content sprint.

Without a backlink tool, you can partially substitute by checking whether the cited page is referenced in obvious places — Wikipedia, industry associations, documentation — but you're guessing at scale. Say so.

---

## 5. What you can and can't run without access

**Runs in full, public data only:**

- §1.1 sitemap, crawl, and reconciliation — minus the search console column
- §2.1 robots.txt and crawler directives
- §2.2 edge blocking, via spoofed user-agent fetches
- §2.3 sitemap quality
- §2.5 internal link graph
- §2.6 canonical and indexability checks
- §3 all of it — raw versus rendered, headers, mobile, lab performance
- §4 all of it — semantics, structured data, entity clarity
- §5.2, §5.3 — duplication and devalued patterns
- §5.4 — expertise markers
- §6.1, §6.2 — chunk and sentence extractability
- §6.3 — assistant testing, with a query set from keyword research instead of impression data

**Requires access, so it's out:**

- §2.4 legacy URL decay (needs the historical URL set; a web archive gets you a partial substitute)
- §2.6 index coverage buckets
- §5.5 opportunity analysis in full
- §6.4 measured AI referral traffic
- Server log crawl data
- True modification dates from version control

**What you lose is prioritisation.** You'll find every problem but won't know which ones cost money. Say that in the verdict rather than implying the ranking is data-backed. A third-party SEO platform closes maybe half this gap with estimated traffic and keyword data — estimated, not measured, and it should be labelled that way in the report.

---

## 6. Teardown output

Shorter than a full audit. Five parts:

1. **Who actually gets cited** — the empirical competitor set from §1, with source-type classification and the share of runs each appeared in
2. **Why they win** — the side-by-side from §3, one table per query cluster
3. **The extractable-sentence comparison** — their top five versus yours. Lead with this if the audience is non-technical.
4. **Gaps ranked by (impact × confidence) ÷ effort** — same rule as the main audit
5. **What's out of reach** — the aggregators, forums, and authority gaps that on-site work won't close, stated plainly

For pre-sales, stop at three findings. A twelve-page teardown reads as a free audit and closes nothing; three specific, evidenced, uncomfortable findings read as expertise.

---

## 7. Crawl conduct

You're accessing someone else's infrastructure without asking. Behave accordingly:

- Respect `robots.txt`, including on sites you're competing with
- Rate limit aggressively — one request per second or slower, and never in parallel
- Identify your crawler honestly in the user-agent, with a contact URL. The exception is §2.2 edge-blocking checks, which require spoofing crawler user-agents by definition — keep those to a handful of requests.
- Public pages only. Nothing behind a login, a paywall, or a form.
- Don't hammer a small site. Sample 40 pages as the spec says, not the whole domain.

This is ordinary competitive research using public data. Keep it that way.
