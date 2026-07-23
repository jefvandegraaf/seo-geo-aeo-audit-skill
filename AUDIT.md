# AUDIT.md — Core Specification

Platform-agnostic. Read `STATIC.md` or `WORDPRESS.md` alongside this for build-specific checks.

Work the phases in order. Each gates the next.

---

## 1. Inventory & sampling

### 1.1 Build the URL set from four sources, then reconcile

| Source | What it tells you |
|---|---|
| `sitemap.xml` (and nested sitemaps) | What you *claim* exists |
| Internal link crawl from `/` | What's actually reachable |
| Search console indexed-pages export | What the engine *found* |
| Build output / server file listing | What actually shipped |

**The gaps between these four are the findings:**

- In build, not in sitemap → invisible to discovery
- In sitemap, not in build → 404s advertised to crawlers
- In search console, not in sitemap or build → orphaned legacy URLs
- In sitemap, zero internal links → orphan pages, near-zero crawl priority

### 1.2 Filter junk before sampling

Exclude: taxonomy archives with no unique copy, pagination, parameterised duplicates, admin paths, API endpoints, feeds, thank-you and confirmation pages, on-site search results.

### 1.3 Sample ~40 pages, weighted

- Homepage and every top-level product/service page — **always included, never sampled**
- Top 10 by search impressions
- Top 10 by search clicks (a different set than impressions — that difference is itself a finding)
- Top 10 by analytics pageviews
- 5 random long-tail pages as a control
- Any page an AI crawler has hit, per server logs

Under ~60 total pages: skip sampling, audit everything. Record sample size and selection method in the output.

---

## 2. Findable

### 2.1 robots.txt

- Parse and resolve every rule against the sampled URLs. Report which pages are disallowed and whether that's intentional.
- Confirm a `Sitemap:` directive is present and resolves to a 200.
- **Check AI crawler user-agents explicitly:**

| User-agent | Operator | Purpose |
|---|---|---|
| `GPTBot` | OpenAI | Training |
| `OAI-SearchBot` | OpenAI | Search index |
| `ChatGPT-User` | OpenAI | Live user-triggered fetch |
| `ClaudeBot` | Anthropic | Index / training |
| `Claude-User` | Anthropic | Live user-triggered fetch |
| `PerplexityBot` | Perplexity | Index |
| `Perplexity-User` | Perplexity | Live user-triggered fetch |
| `Google-Extended` | Google | Gemini training (does **not** affect Search or AI Overviews) |
| `Applebot-Extended` | Apple | Training |
| `CCBot` | Common Crawl | Corpus used by many models |
| `Bytespider` | ByteDance | Training |
| `meta-externalagent` | Meta | Training |

**The distinction most sites get wrong:** training crawlers and retrieval crawlers are different bots. Blocking the training crawlers costs you nothing in citations. Blocking the retrieval and user-triggered fetchers makes you uncitable. Most "block the AI scrapers" advice conflates them and quietly kills the site's ability to be cited.

Decide the policy deliberately, then verify robots.txt actually implements it.

### 2.2 Edge / firewall blocking — check this before anything else

robots.txt is a request. Your CDN or host's bot protection is enforcement, and it wins.

- Fetch a sample page with each AI crawler user-agent and record the status code. A `403`, a `503`, or a JS challenge means you're blocked regardless of what robots.txt permits.
- Check any managed bot-mitigation rules matching "AI crawlers", "AI scrapers", or "verified bots". Several popular CDN configurations block retrieval crawlers by default, and some hosts enable it without telling you.
- Check rate limiting — an aggressive crawler hitting a per-IP limit gets throttled into uselessness.
- Repeat this test after every CDN or host configuration change.

**This is the single most common cause of a technically flawless site getting zero AI citations.**

### 2.3 Sitemap quality

- Valid XML, under 50,000 URLs and 50MB per file
- No 404s, no redirects, no `noindex` pages, no non-canonical URLs
- Consistent protocol and host with canonical tags (no `www` vs apex mismatch)
- `lastmod` present and **accurate**. If every URL carries the same date, or the date changes on every deploy regardless of content change, the field is worthless and engines learn to ignore it. Derive it from actual content-modification history.

### 2.4 Legacy URL decay

Run this on any site that has been rebuilt, replatformed, or restructured:

- Pull the historical URL set from search console (full available window) and from a web archive
- Test every historical URL for a `301` to a live equivalent
- Report the redirect map and, more importantly, the unmapped remainder
- Flag redirect chains (more than one hop) and loops
- Check that redirects go to the *relevant* page, not a blanket redirect to the homepage — engines treat mass homepage redirects as soft 404s
- Check whether media/asset URLs still resolve. Images and PDFs get cited and hotlinked from elsewhere; breaking them destroys existing references.

### 2.5 Internal link graph

- Build the graph and compute inbound internal links per URL
- Flag orphans (0 inbound) and near-orphans (1–2 inbound, all from a sitewide footer)
- Flag pages more than 3 clicks from the homepage
- Verify money pages have more inbound links than blog posts. On most sites they don't.
- Report anchor text distribution — generic anchors ("read more", "click here") waste the signal entirely

### 2.6 Canonicals and indexability

- Self-referencing canonical on every indexable page, absolute URL, matching the served URL exactly
- Nothing in the sitemap carrying `noindex`
- No conflict between `<meta name="robots">` and the `X-Robots-Tag` response header
- Cross-reference search console's *discovered but not indexed* and *crawled but not indexed* buckets. Those are a direct quality verdict. List every URL in them.
- **Cannibalisation:** find queries where two or more of your URLs both draw impressions. That's you competing with yourself, and it usually means a consolidation, not an optimisation.

---

## 3. Readable

### 3.1 Raw HTML vs rendered DOM

The most important technical check in this document.

Fetch every sampled page twice:

1. No JavaScript execution (plain HTTP fetch)
2. Headless browser, fully rendered

Diff the extracted text and structure. **Content present only in the rendered version is invisible to most AI retrieval crawlers.** Google renders JavaScript at scale; the AI crawlers largely do not, or do it inconsistently and with significant delay.

Report per page:

- Word count raw vs rendered, and the percentage of body text that is JS-injected
- Whether `<h1>`, `<title>`, meta description, canonical, and structured data exist in the raw HTML
- Whether primary navigation and internal links exist in the raw HTML — a JS-built nav means no link graph at all for those crawlers

**Thresholds:** more than 10% of body text JS-only is a fail. Any headings, links, canonicals, or structured data missing from raw HTML is a fail.

Content inside a collapsed accordion is fine *if it's in the DOM*. Content fetched on click is not.

### 3.2 Performance

Use field data (real-user metrics) rather than lab scores alone. A perfect lab score with poor field data is common.

- Largest Contentful Paint, Interaction to Next Paint, Cumulative Layout Shift — 75th percentile, split mobile and desktop
- Time to first byte from at least two geographies
- Render-blocking resources, uncompressed or unsized images, fonts without a swap strategy

Performance is a ranking input and a crawl-budget input. It is **not** a citation input — a slow page that answers the question still gets cited. Weight accordingly and don't let a green Lighthouse score create false confidence about the rest of the audit.

### 3.3 Response and headers

- `200` on every sampled URL, and no soft 404s (a `200` status on an empty or error page)
- `X-Robots-Tag` headers not contradicting on-page directives
- HTTPS everywhere, no mixed content, valid certificate chain
- Correct `Content-Type` and charset
- Compare configured redirects/headers in source control against actual live responses — they drift

### 3.4 Mobile

- Viewport meta present, no horizontal scroll at 360px width
- Tap targets ≥44px, base font ≥16px
- No interstitial, cookie wall, or modal covering primary content on load

---

## 4. Understandable

### 4.1 On-page semantics

- Exactly one `<h1>`; no skipped heading levels
- `<title>` unique sitewide, under ~60 characters, and **semantically matching the H1 and opening paragraph**. Compute similarity between the three — divergence means the page is confused about its own subject.
- Meta description unique, under ~155 characters, written as a claim rather than a keyword list
- Semantic HTML: `<main>`, `<article>`, `<nav>`, `<time datetime="">`. Div soup is renderable but gives a parser nothing to anchor on.
- Descriptive alt text on content images; empty alt on decorative ones
- Descriptive link anchors

### 4.2 Structured data

Validate against schema.org and current rich-result requirements. Score **completeness**, not just validity — most sites emit technically valid schema that's missing everything useful.

| Page type | Core type | Frequently missing |
|---|---|---|
| Sitewide | `Organization` or `LocalBusiness` | `sameAs` array pointing to profiles elsewhere |
| Article / post | `Article` / `BlogPosting` | `author` as a linked `Person` entity, both `datePublished` and `dateModified` |
| Service | `Service` | `provider`, `areaServed`, `serviceType` |
| Product | `Product` | valid `offers` price and availability |
| Q&A block | `FAQPage` | must match visible page text exactly |
| Sitewide | `BreadcrumbList` | absent on the majority of sites |
| Author / about | `Person` | `jobTitle`, `knowsAbout`, `alumniOf`, `sameAs` |

Check for **duplicate or conflicting schema** — sites running more than one SEO tool commonly emit two `Organization` blocks with different values, which is worse than emitting none.

`sameAs` does more work than most people realise: it's how a parser connects "this site" to "this entity" as described everywhere else on the web. It's the cheapest entity-disambiguation win available.

### 4.3 Entity clarity

- Primary entity named in the first 100 words, unabbreviated
- Pronouns resolved. "We do X" is unusable in a retrieved chunk; "[Company] does X" survives extraction intact.
- Location stated in body text where relevant, not only in schema
- Consistent name, address, and phone across pages and against external listings

---

## 5. Rankable

### 5.1 Freshness

- Derive true modification dates from version control or CMS revision history, not build or deploy time
- Flag pages untouched for 18+ months that still hold search impressions — these are decay candidates and usually the highest-ROI updates on the site
- Flag stale in-text references (year mentions, "the current version is…", superseded pricing)
- Check that dates visible in copy match the dates in structured data

### 5.2 Uniqueness and duplication

- **Near-duplicate detection:** shingle or MinHash comparison across all pages. Flag pairs above ~85% similarity. Location and service-variant pages are the usual offenders.
- **Boilerplate ratio:** what percentage of each page is navigation, footer, CTA blocks, and sitewide testimonials versus unique body copy? Below ~40% unique is thin regardless of total word count.
- **External duplication:** spot-check distinctive sentences to detect syndication, scraping, or purchased template copy.

### 5.3 Patterns search engines have publicly devalued

Flag and count:

- **Programmatic/templated pages** — same structure, swapped noun. Detect via high structural similarity paired with low content similarity.
- **Doorway pages** — multiple near-identical pages targeting location or keyword variants, all funnelling to one conversion point
- **Thin pages** — under ~300 words of unique body copy and indexable
- **Listicle overuse** relative to site size
- **Purposeless pages** — no conversion path, no unique information, no inbound links
- **Low site focus** — embed every page, compute the centroid, measure per-page distance. A wide spread means the site has no clear topical identity, which hurts classical ranking *and* retrieval.

### 5.4 Experience and expertise markers

Per page, present or absent:

- Named author with a real bio, stated credentials, and a linked person page
- Publication and modification dates visible to a human, not only in schema
- Outbound citations to primary sources — count them
- First-hand evidence: original screenshots, own data, "we tested / built / measured" language
- Specificity: named tools, named numbers, named constraints, versus generic advice
- Contact information, business registration, verifiable physical presence

### 5.5 Search-console-driven opportunity analysis

Where the audit stops describing and starts producing work:

- **Striking distance** — queries at average position 5–15 with meaningful impressions, mapped to pages, ranked
- **High impressions, low CTR** — title and meta rewriting candidates, sorted by clicks recoverable
- **Query/page mismatch** — query intent that doesn't match the ranking page's actual subject. Usually means a new page, not a rewrite.
- **Lost queries** — last 90 days versus the prior 90 days, flag drops
- **Zero-impression indexed pages** — indexed, crawled, returning nothing. Consolidate, redirect, or delete.

---

## 6. Citable

Everything above is a prerequisite for this section.

### 6.1 Chunk-level retrieval simulation

Retrieval systems don't ingest pages, they ingest chunks. Simulate it.

- Split each page the way a retriever would: by heading section, roughly 200–500 tokens, with overlap
- Score each chunk on whether it **stands alone**:
  - Subject named explicitly — no orphaned "it", "this", "the company"
  - Contains at least one concrete, checkable claim
  - Doesn't depend on a preceding chunk for context
  - Heading accurately describes the contents
- Report the percentage of chunks that are self-contained.

**This is the single most important GEO metric and almost nobody measures it.** A page can read beautifully top-to-bottom and shatter into unusable fragments.

### 6.2 Sentence-level extractability

Within each chunk, count sentences that could be lifted verbatim as a source for a specific claim:

- **Definition sentences** — "X is Y", "X means Y". Assistants use these for the opening line of an answer more than any other construction.
- **Original numbers** — a figure *you* produced, with the method stated. A figure you cited from someone else sends the citation to them, not you.
- **Answer-first openers** — does the section lead with the answer, then explain? Or bury it under 200 words of preamble? Preamble is where citations go to die.
- **Comparison tables** — disproportionately extracted and cited. Count them.
- **Named-entity density near claims** — a claim with no named subject can't be attributed back to you.
- **Hedge load** — "may", "might", "generally", "it depends". Hedged sentences are rarely quoted. Report a hedge ratio per page.

Output a per-page quotable-sentence count and list the top five most extractable sentences, so you can see what a model would actually take.

### 6.3 Live assistant testing

**Query set — 10–15 per site, built from real demand:**

- Top queries by impressions from search console
- Question-form queries (who / what / how / best / versus)
- The buyer's own phrasing — how someone describes the problem *before* they know the vocabulary
- Head-to-head comparison queries
- **Brand-association queries** ("who does X in [location]") — tests whether the model knows you exist at all, which is a separate outcome from whether it cites you
- One or two queries you should dominate — your niche, your original data

**Run across:** ChatGPT (search mode), Claude, Perplexity, Google AI Mode, Copilot, Gemini.

**Capture per run:** full response text, every cited URL and its position, whether your domain appears, whether your brand is mentioned *without* a link (a real and distinct outcome), which competitors appear, and what *type* of source won — first-party site, review aggregator, forum, video, trade press.

**Methodology, stated honestly:** these results are non-deterministic, personalised, and shift week to week.

- Run each query at least 3 times, clean session, logged out, from the target market
- Report citation rate as a fraction ("3 of 5 runs"), never as a binary
- Log raw response text every time so you can diff across audits
- Treat it as directional trend data, not a score

**Per-engine index dependency** — this explains most "cited on one, absent on another" results. Different assistants resolve against different indexes, so being well-indexed in one search engine tells you nothing about the others. Verify indexation in **every** search index your target assistants rely on, not just the dominant one, and use push-based submission protocols where the index supports them. Absence from a secondary index is a common and completely invisible cause of zero citations.

Retrieval architectures change frequently. Re-verify which assistant uses which index at audit time rather than trusting a table written months ago.

### 6.4 Measured AI referral traffic

The only hard evidence in this section. From your analytics, segment referrals by assistant domains — the major chat and answer-engine hostnames, refreshed at audit time since new ones appear regularly.

Report volume, trend, landing pages receiving them, and engagement compared to organic search traffic. AI referrals are typically lower volume and higher intent. **The trend line is the KPI that makes this whole exercise defensible to a client.**

Cross-reference with server logs: crawler hits mean the bot came; referrals mean a human clicked through. High crawl volume with zero referrals means you're being read and not cited — which points you straight back at §6.1 and §6.2.

### 6.5 `llms.txt`

No major assistant has confirmed adoption, and there's no public evidence it affects retrieval. Cost to ship is about an hour. Include it as a low-priority, optional item and label it speculative. Don't sell it as a ranking factor.

---

## 7. Scoring

Five dimensions, 0–100 each. Report sub-scores separately — an average hides everything useful.

| Dimension | Weight | Hard fail condition |
|---|---|---|
| Findable | 20% | Retrieval crawlers blocked at the edge, or sitemap broken |
| Readable | 20% | More than 10% of body text is JS-only |
| Understandable | 20% | No structured data, or sitewide title/H1 divergence |
| Rankable | 20% | More than 30% near-duplicate or thin pages |
| Citable | 20% | Under 40% of chunks self-contained |

**Never blend the structural score and the citation score into one number.** They're independent, and their divergence is usually the most interesting finding in the audit.

---

## 8. Output

**`audit.json`** — machine-readable, per URL, every check with `pass` / `fail` / `warn` / `skipped`, raw values, and a timestamp. This is what you diff between runs to evidence improvement.

**`audit.md`** — the human report:

1. **Verdict** — one paragraph, plain language, no scores
2. **Five dimension scores**, each with its single biggest issue
3. **Fix list, ranked by (impact × confidence) ÷ effort** — not by severity
4. **Evidence appendix** — every claim traced to a specific URL and check
5. **What was skipped, and why**

**Cadence:** structural checks monthly. Assistant testing every two weeks — it's the noisiest and the fastest-moving. Retain every historical `audit.json`; the trend line is the product.

---

## 9. Known limits

State these in every client-facing report:

- Assistant citation testing is non-deterministic and personalised. Results are directional, not measurements.
- LLM-graded content quality — voice, originality, point of view — is a model's opinion. Report the reasoning alongside the grade so it can be argued with.
- No public tool sees inside any assistant's retrieval or ranking. Everything in §6 is black-box observation.
- Structural passes are necessary but not sufficient. This audit tells you what is *blocking* citation. Nothing guarantees it.
- Crawler behaviour, user-agent strings, and rendering capabilities change without notice. Re-verify §2.1 and §6.3 at every audit rather than trusting cached assumptions.
