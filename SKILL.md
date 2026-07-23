---
name: seo-geo-aeo-audit
description: Audit a website for search engine visibility (SEO) and AI assistant citation (GEO/AEO) — findability, crawlability, structured data, content quality, and whether AI assistants like ChatGPT, Claude, Perplexity, and Google AI Mode would actually cite the page. Use this skill whenever the user mentions an SEO audit, site audit, technical SEO, GEO, AEO, generative engine optimization, answer engine optimization, LLM SEO, AI search visibility, schema markup review, crawlability, "why isn't my site showing up in ChatGPT", "why don't AI assistants cite us", competitor SEO teardown, or asks for any kind of website visibility, indexing, or search performance analysis — even if they don't use the word "audit".
---

# SEO / GEO / AEO Audit

A structured audit for websites that need to work in **search engines** and in **AI assistants**.

Traditional audits stop at "can Google rank this?" A page can be technically flawless, rank on page one, and still never appear as a source in ChatGPT, Claude, Perplexity, or Google AI Mode — because those systems retrieve *chunks*, not pages, and they resolve against indexes that aren't Google's. This skill covers both.

## The five questions

Each gates the next. A page that can't be fetched can't be read; a page that can't be read can't be cited.

1. **Findable** — can a crawler discover the URL?
2. **Readable** — can it get the content once it arrives?
3. **Understandable** — does the page declare what it's about?
4. **Rankable** — is there a reason to prefer it over the alternatives?
5. **Citable** — would an assistant lift a sentence from it and name the source?

## How to run this

### Step 1 — Establish scope and data access

Ask the user, if not already clear:

- Which site (root domain), and is it theirs or a competitor's?
- What platform — static build, WordPress, other CMS, or unknown?
- What data access they can provide

Then read **`references/DATA-SOURCES.md`** and produce the access checklist. Record what's available at the top of the output.

**Do not guess at missing data.** Every check without a data source gets marked `SKIPPED — no data source`. A skipped check is an honest result; an estimated one is a lie with a number attached.

If the user has no access to the site (competitor research, pre-sales), go to **`references/COMPETITOR-TEARDOWN.md`** instead and follow that workflow.

### Step 2 — Load the right references

Always read **`references/AUDIT.md`** in full. It's the core spec and everything else is additive.

Then read the one platform file that matches the build:

- **`references/STATIC.md`** — static site generators, JAMstack, hand-built HTML, SPAs
- **`references/WORDPRESS.md`** — WordPress and other database-driven CMSs

If the platform is unknown, determine it empirically first — fetch the homepage, check response headers, generator meta tags, and asset paths — then load the matching file.

### Step 3 — Work the phases in order

Follow §1 through §6 of `AUDIT.md` sequentially. **Do not skip ahead.**

The ordering matters for a specific reason: an edge firewall blocking AI retrieval crawlers (§2.2) makes every finding in §6 meaningless. Verifying crawler access before analysing citation-worthiness saves the user from a remediation plan aimed at the wrong problem.

Two checks are the most common sources of a false-clean audit, so verify them explicitly even when everything looks green:

- **§2.2 edge / firewall blocking.** robots.txt is a request; the CDN's bot mitigation is enforcement, and it wins. Fetch with each AI crawler user-agent and record status codes.
- **§3.1 raw HTML versus rendered DOM.** Content that only exists after JavaScript runs is invisible to most AI retrieval crawlers.

### Step 4 — Produce the output

Two artefacts, per §8 of `AUDIT.md`:

- **`audit.json`** — machine-readable, per URL, every check with `pass` / `fail` / `warn` / `skipped`, raw values, timestamp. This is what gets diffed between runs.
- **`audit.md`** — the human report: verdict paragraph, five dimension scores, fix list, evidence appendix, and what was skipped.

## Rules that hold throughout

**Report the structural score and the citation score separately. Never blend them.** They're independent. A site can score 95 structurally and get cited zero times; a site can score 60 and get cited constantly because it was the only thing on the topic two years ago. That divergence is usually the most interesting finding in the audit.

**Rank the fix list by (impact × confidence) ÷ effort, not by severity.** A fifteen-minute firewall rule change that unblocks a retrieval crawler beats a forty-hour content rewrite.

**Every finding traces to a URL and a check.** If a claim can't be evidenced, it doesn't go in the report.

**State uncertainty rather than hiding it.** Assistant citation testing is non-deterministic and personalised — report citation rates as fractions across multiple runs, never as a binary. LLM-graded content quality is a model's opinion; report the reasoning alongside the grade so it can be argued with.

**Re-verify volatile facts at audit time.** Crawler user-agent strings, rendering capabilities, and which assistant resolves against which search index all change without notice. Don't trust cached assumptions, including any in these files.

## Reference files

| File | Read it when |
|---|---|
| `references/AUDIT.md` | Always — the core spec, §1 through §9 |
| `references/DATA-SOURCES.md` | At the start, to establish what's available |
| `references/STATIC.md` | The site is a static build, JAMstack, or SPA |
| `references/WORDPRESS.md` | The site is WordPress or a similar CMS |
| `references/COMPETITOR-TEARDOWN.md` | Auditing a site you don't own or have access to |
