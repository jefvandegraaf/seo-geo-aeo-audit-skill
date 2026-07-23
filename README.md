# SEO / GEO / AEO Audit Skill

An open, platform-agnostic audit specification for websites that need to work in **search engines** and in **AI assistants**. Packaged as a [Claude Code](https://docs.claude.com/en/docs/claude-code) skill, and usable as plain Markdown with any assistant or by a human.

Traditional SEO audits stop at "can Google rank this?" That's no longer the whole question. A page can be technically flawless, rank on page one, and still never appear as a source in ChatGPT, Claude, Perplexity, or Google AI Mode — because those systems retrieve *chunks*, not pages, and they resolve against indexes that aren't Google's.

This spec covers both.

---

## The five questions

Each one gates the next. A page that can't be fetched can't be read; a page that can't be read can't be cited.

1. **Findable** — can a crawler discover the URL?
2. **Readable** — can it get the content once it arrives?
3. **Understandable** — does the page declare what it's about?
4. **Rankable** — is there a reason to prefer it over the alternatives?
5. **Citable** — would an assistant lift a sentence from it and name the source?

Questions 1–4 are conventional SEO, tightened. Question 5 is the part most audits don't have.

---

## Install as a Claude Code skill

**Personal (available in every project):**

```bash
git clone https://github.com/jefvandegraaf/seo-geo-aeo-audit-skill.git ~/.claude/skills/seo-geo-aeo-audit
```

**Project-scoped (committed with a repo, shared with your team):**

```bash
git clone https://github.com/jefvandegraaf/seo-geo-aeo-audit-skill.git .claude/skills/seo-geo-aeo-audit
```

Then just ask:

> Audit https://example.com. It's a static build on Cloudflare Pages and I've got Search Console access.

The skill triggers on audit, SEO, GEO, AEO, crawlability, schema, and AI-citation phrasing without needing to be named.

## Use without Claude Code

Nothing here depends on Claude Code — it's Markdown.

- **Claude Projects / ChatGPT Projects / custom GPTs** — upload `SKILL.md` and the `references/` files as project knowledge
- **Cursor, Windsurf, or any agent** — point it at `SKILL.md` and let it route
- **By hand** — read `references/AUDIT.md` and work the phases in order

---

## Repository structure

```
├── SKILL.md                          entry point — scope, workflow, routing
└── references/
    ├── AUDIT.md                      the core spec, §1–§9 — start here
    ├── DATA-SOURCES.md               which data feeds unlock which checks
    ├── COMPETITOR-TEARDOWN.md        auditing a site you don't own
    ├── STATIC.md                     static generators, JAMstack, SPAs
    └── WORDPRESS.md                  WordPress and CMS-driven sites
```

Read `AUDIT.md` in full, then the one platform file matching your build. Platform files are additive — they add checks and gotchas, they don't replace anything in the core.

**`COMPETITOR-TEARDOWN.md` is the fastest way to get value out of this repo.** It needs no credentials and no cooperation from the target, and running it against whoever currently gets cited for your queries usually explains why in about ten minutes.

---

## What it actually checks

Some of what's in here that isn't in a standard audit:

- **Edge and firewall blocking of AI crawlers.** robots.txt is a request; your CDN's bot mitigation is enforcement. Several popular configurations block retrieval crawlers by default. This is the most common cause of a technically perfect site getting zero citations, and almost nobody checks it.
- **Training crawlers versus retrieval crawlers.** They're different bots. Blocking the first costs nothing; blocking the second makes you uncitable. Most "block the AI scrapers" advice conflates them.
- **Raw HTML versus rendered DOM.** Search engines render JavaScript at scale. AI retrieval crawlers largely don't. Client-rendered content is invisible to them.
- **Chunk-level retrieval simulation.** Retrievers ingest chunks, not pages. The metric is what percentage of your chunks stand alone with the subject named and no orphaned pronouns — a page can read beautifully top to bottom and shatter into unusable fragments.
- **Sentence-level extractability.** Definition sentences, original numbers, answer-first structure, hedge ratio, comparison tables. The audit outputs the five sentences a model would most likely lift from each page.
- **Measured AI referral traffic.** Segmenting analytics by assistant hostnames is the only hard evidence in the whole GEO half.

---

## Design principles

**Structure and citation scores are reported separately, never blended.** A site can score 95 structurally and get cited zero times. That divergence is usually the most interesting finding in the audit.

**Every finding traces to a URL and a check.** No vibes. If a claim can't be evidenced, it doesn't go in the report.

**The fix list ranks by (impact × confidence) ÷ effort, not by severity.** A fifteen-minute firewall rule change beats a forty-hour content rewrite.

**Uncertainty is stated, not hidden.** Assistant citation testing is non-deterministic. LLM-graded content quality is an opinion. Both are useful; both are labelled as what they are.

**No vendor lock-in.** The spec names categories of data source, not products. Bring your own analytics, host, and crawler.

---

## What this is not

- **Not a tool.** It's a specification. Run it with an agent, or build the tooling from it.
- **Not a ranking-factor list.** Nobody outside the search and assistant companies has one.
- **Not a guarantee.** It tells you what's *blocking* citation. Nothing guarantees citation.

---

## Contributing

Findings from real audits are the most valuable contribution — especially:

- Retrieval behaviour that contradicts something in §6 of `AUDIT.md`
- Platform gotchas for builds not yet covered
- Crawler user-agent and rendering-capability changes

Open an issue with evidence: URLs, dates, and raw responses. Claims without evidence won't be merged.

Crawler behaviour and assistant retrieval architectures change constantly. Anything in here stated as current fact has a shelf life — corrections are welcome and expected.

## License

MIT. Use it commercially, fork it, sell audits built on it.
