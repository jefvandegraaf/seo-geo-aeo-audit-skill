# STATIC.md

Additive to `AUDIT.md`. Covers static site generators, hand-built HTML, JAMstack, and client-rendered apps.

Static builds win the technical half of the audit almost by default — fast, clean markup, no plugin sprawl. That creates a specific failure mode: **a green Lighthouse score generates false confidence, and the real problems are all in rendering, routing, and freshness.** The checks below are where static sites actually lose.

---

## S1. Rendering mode — establish this first

Everything else depends on which of these you're auditing:

| Mode | Content in raw HTML? | Risk |
|---|---|---|
| Static generation (SSG) | Yes | Low — freshness and routing only |
| Server-side rendering (SSR) | Yes | Low, but check cache and TTFB behaviour |
| Pre-rendering / hydration | Yes, then rehydrated | Watch for hydration mismatch removing content |
| Client-side rendering (CSR / SPA) | **No** | Severe — see S2 |
| Islands / partial hydration | Partially | Audit per component |

Determine it empirically, not from the framework name. Fetch with JavaScript disabled and look at what's actually there.

---

## S2. Client-side rendering (§3.1 amplified)

If content only exists after JavaScript runs, most AI retrieval crawlers never see it. Search engines render at scale; retrieval crawlers largely don't.

**Check specifically:**

- **Client-side routing** — do non-homepage URLs return real HTML on a direct fetch, or an empty shell? Test deep URLs directly, never by clicking through from the homepage.
- **`404` handling** — SPA fallbacks commonly return `200` with the app shell for *any* path, including typos and dead legacy URLs. That's an infinite soft-404 surface, and it makes §2.4 legacy-URL testing return false passes. Verify a deliberately invalid URL returns a real `404` status code.
- **JS-built navigation** — no link graph at all for non-rendering crawlers
- **Content loaded on interaction** — "load more", tabs that fetch on click, modals, client-side search. In-DOM-but-collapsed is fine; fetch-on-click is invisible.
- **Client-side markdown or MDX rendering** — the whole article is JS-only
- **Hydration mismatch** — content in the server HTML that React/Vue/Svelte replaces or drops on hydration. Diff raw versus rendered in *both* directions.

**If the site is CSR:** pre-rendering or SSG for crawler-facing routes is the fix, and it's the highest-priority item in the entire audit. Nothing in §6 matters until it's done.

---

## S3. URL hygiene

Static hosts are stricter and less forgiving than application servers:

- **Trailing slash consistency.** `/page` and `/page/` both returning `200` is duplicate content across your entire site. Pick one, `301` the other, and make sure the sitemap, canonicals, and internal links all use the chosen form. This is the most common static-site duplication bug.
- **Case sensitivity.** Most static hosts are case-sensitive; some are not. `/About` and `/about` both resolving is the same duplication problem.
- **`index.html` exposure.** `/page/index.html` should `301` to `/page/`, not serve alongside it.
- **Query parameters.** Static hosts ignore them but still serve `200`, so `?utm_source=x` creates infinite duplicate URLs. Canonicals must strip parameters.
- **Deploy preview domains.** Branch and PR previews on most platforms are publicly accessible and indexable by default. A full duplicate of your site on a preview subdomain is a genuine, common, and completely silent problem. Enforce `noindex` headers or `Disallow: /` in robots.txt on every non-production domain, and verify it — don't assume the platform does it.

---

## S4. Redirects and headers

- Configured redirects and headers live in source control (`_redirects`, `_headers`, `netlify.toml`, `vercel.json`, `staticwebapp.config.json`, edge worker scripts). **Diff the config against live responses** — they drift, especially after platform migrations.
- Check redirect rule ordering. Most platforms use first-match-wins; a broad wildcard placed early silently swallows every specific rule beneath it.
- Watch for splat/wildcard rules that redirect everything to the homepage. That's a mass soft-404 pattern, and it's the default "fix" people reach for after a migration.
- Verify security and caching headers actually ship — configured is not the same as served.

---

## S5. Sitemap and `lastmod`

- Sitemap should be **generated at build time from the routes that actually shipped**, not maintained by hand. Hand-maintained sitemaps drift within two deploys.
- Exclude drafts, `noindex` pages, redirect targets, and preview routes from generation. Verify — don't trust the plugin.
- **`lastmod` from build time is worthless.** Every page claims to be freshly modified on every deploy, and engines learn to ignore the field across your whole domain. Derive it from git commit history per file, or from the CMS's modification timestamp.
- Same rule applies to `dateModified` in structured data.

---

## S6. Freshness

Static sites decay quietly. There's no dashboard nagging you about a post from 2023, no plugin update forcing you into the editor.

- Run `git log` per content file and rank by staleness
- Cross-reference against search impressions — stale pages that still draw traffic are the highest-ROI updates on the site (§5.1)
- **Headless CMS rebuild lag:** if content lives in a CMS but the site rebuilds on webhook, verify the webhook fires and the build succeeds. A silently failing build means published content that never ships. Check the gap between CMS modification time and live deploy time.
- Scheduled/future-dated posts need a scheduled rebuild — otherwise they publish in the CMS and never appear on the site.

---

## S7. What static builds routinely drop

Rebuilds lose things nobody notices until traffic does:

- **RSS/Atom feeds.** Usually not generated by default. If the old site had one, either regenerate it at the same path or `301` it — aggregators, readers, and newsletter tools still consume feeds.
- **On-site search.** Client-side search produces no crawlable result pages. That's usually fine (search result pages shouldn't be indexed anyway) but verify it isn't generating indexable parameter URLs.
- **Pagination.** Often dropped or converted to infinite scroll. Infinite scroll without real paginated URLs means archive content beyond page one is undiscoverable.
- **Author pages.** Frequently omitted in rebuilds. They're your `Person` schema anchor and your E-E-A-T signal (§4.2, §5.4) — the cheapest expertise win available.
- **Comments.** Removed with the CMS. Usually no loss, but they were unique on-page content and their absence changes thin-page calculations.
- **Media URLs.** Asset paths change on rebuild. Images and PDFs get hotlinked and cited externally; breaking them destroys existing references (§2.4).

---

## S8. Structured data

Static builds have no SEO plugin auto-generating schema, so it's usually either absent or hardcoded and wrong.

- Template it from front matter or CMS fields — never hand-write per page
- `datePublished` and `dateModified` must be dynamic. Hardcoded dates go stale immediately and contradict your sitemap.
- `Person` schema for authors, `BreadcrumbList` sitewide, `Organization` with a populated `sameAs` array — all commonly missing entirely (§4.2)
- Validate on every build. A broken template breaks schema across the entire site at once, which is the failure mode static builds are uniquely exposed to.

---

## S9. Edge configuration

Static hosting almost always sits behind a CDN with bot mitigation enabled.

- Run §2.2 against your production domain specifically, and re-run it after every platform or CDN configuration change
- Managed bot rules on several popular platforms block AI retrieval crawlers by default
- Rate limiting tuned for abuse prevention can throttle a legitimate crawler into uselessness
- If you're using edge middleware or workers for routing, verify they don't behave differently for non-browser user-agents — geo-redirects and A/B logic sometimes serve crawlers a redirect loop

---

## Static-specific priority order

1. Rendering mode and CSR content visibility (S1, S2)
2. Edge crawler blocking (S9, §2.2)
3. SPA soft-404s and trailing-slash duplication (S2, S3)
4. Preview domain indexation (S3)
5. Real `lastmod` and `dateModified` (S5)
6. Structured data templating (S8)
7. Freshness workflow (S6)
8. Whatever the rebuild dropped (S7)
