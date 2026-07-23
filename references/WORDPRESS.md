# WORDPRESS.md

Additive to `AUDIT.md`. Written for WordPress; most of it transfers to any database-driven CMS that auto-generates archives.

WordPress inverts the static-site failure mode. Rendering is rarely the problem — PHP outputs complete HTML, so §3.1 usually passes. The problems are **surface area** (the CMS generates thousands of URLs you never asked for), **plugin conflict** (multiple tools emitting contradictory metadata), and **markup weight** (page builders producing div soup that parses badly).

---

## W1. Auto-generated URL surface

WordPress creates indexable URLs from nothing. Enumerate and decide on each:

| URL type | Default | Usual correct handling |
|---|---|---|
| Category archives | Indexable | Index only if it has unique intro copy and real search demand |
| Tag archives | Indexable | `noindex` unless deliberately curated — this is the biggest source of thin pages on most WP sites |
| Author archives | Indexable | Index if you have real authors with bios; `noindex` on single-author sites |
| Date archives | Indexable | `noindex` always — no search intent exists for them |
| Attachment/media pages | Indexable | Redirect to the parent post or disable entirely |
| Paginated archives | Indexable | Index with self-referencing canonicals, never canonical to page one |
| Search results (`?s=`) | Crawlable | `noindex, nofollow` — otherwise you generate unlimited thin URLs |
| Custom post type archives | Varies | Audit each; many are registered by plugins you forgot about |
| `?replytocom=` comment links | Crawlable | `nofollow` or disable threaded comments |
| Feeds (`/feed/`, per-taxonomy) | Live | Keep the main feed; `noindex` the rest |
| REST API (`/wp-json/`) | Public | Restrict if unused; check what it exposes |

Count the total indexable URLs against the number of pages you actually wrote. A ratio above roughly 3:1 means the CMS is generating most of your index, and that's a §5.3 thin-content finding.

---

## W2. Plugin conflict and duplicate output

The most common WordPress-specific technical fault, and it's invisible in a browser.

- **Two SEO plugins active**, or one active and one deactivated-but-still-outputting via a theme function. Symptoms: two `<title>` tags, two canonicals, two `Organization` schema blocks with different values. **Conflicting schema is worse than no schema.**
- **Theme plus plugin both emitting schema.** Many commercial themes ship their own. View source and count `application/ld+json` blocks — more than one graph is a finding.
- **Duplicate Open Graph tags** from a social plugin and the SEO plugin.
- **Sitemap ambiguity** — core, the SEO plugin, and a caching plugin can each generate one. Confirm which is live, which is in robots.txt, and that the other two return `404` rather than serving a stale second sitemap.
- **Multiple analytics or tag manager instances** double-counting, which corrupts every engagement metric in §5.3.

---

## W3. Page builders and markup quality

Applies to Elementor, Divi, WPBakery, Beaver Builder, and heavy block libraries.

- **Heading structure.** Builders let editors pick heading levels for visual size. Expect multiple `<h1>`s, skipped levels, and `<h2>`s used as decorative subtitles. Audit the heading tree of every builder-made page (§4.1).
- **Div nesting depth.** Builders wrap content in 6–10 nested divs with no semantic elements. It renders fine and parses badly — nothing for an extractor to anchor on. Check for the presence of `<main>`, `<article>`, `<time>`.
- **Text in widget attributes.** Some builder elements store content in shortcode attributes or data attributes rather than text nodes. Extractors miss it. Run the §3.1 diff even though the site is server-rendered — you're looking for text that's in the HTML but not in the *extractable text*.
- **Lazy-loaded builder sections.** Some builders defer below-fold sections via JS. That content is invisible to retrieval crawlers (§3.1).
- **Inline CSS bloat.** Builders inject per-page stylesheets inline, sometimes 100KB+ before any content. Pushes content deep into the document and hurts LCP.
- **Global template content** counted as page content — inflates word count while unique-content ratio stays thin (§5.2).

---

## W4. Freshness and `lastmod` integrity

- WordPress `post_modified` updates on *any* save, including a typo fix, a plugin's bulk operation, or a builder re-save. Bulk actions can restamp hundreds of posts with no content change, which destroys the credibility of your `lastmod` signal exactly as thoroughly as a static build stamping everything with the deploy date (§2.3, §5.1).
- Cross-check `post_modified` against actual revision diffs where possible. Report pages whose date moved without meaningful content change.
- Scheduled posts that failed to publish (`future` status past their date) are a common silent gap — check for them.
- Draft and pending content accidentally set to `publish` with no internal links is an orphan (§2.5).

---

## W5. Caching, CDN, and crawler behaviour

- **Serve cached HTML to bots.** Some security plugins and firewalls challenge or serve degraded pages to non-browser user-agents. Run §2.2 with every AI crawler user-agent against a production URL.
- **Security plugins** (firewall, login protection, anti-bot) are a frequent cause of `403`s to legitimate crawlers. Check their bot rules explicitly, not just the CDN's.
- **Cache variance** — logged-out cached HTML can differ from what an editor sees. Always audit logged out, in a clean session.
- **Cache purge lag** — updated content that still serves the old cached version is a freshness problem no amount of `dateModified` fixes.

---

## W6. Structured data

Usually present via the SEO plugin, usually incomplete.

- Plugin-generated `@graph` typically covers `WebSite`, `WebPage`, `Organization`, and `Article` — and typically ships with an empty `sameAs`, a generic `author` reference that doesn't resolve to a real `Person` entity, and no `BreadcrumbList` unless breadcrumbs are visibly displayed.
- **`Person` schema for authors is the biggest available win** (§4.2, §5.4). Populate the user profile bio, add `jobTitle`, `knowsAbout`, and `sameAs` links. Most SEO plugins expose fields for this and most sites leave them empty.
- `Service` and `LocalBusiness` schema usually require manual addition — plugin defaults rarely cover service pages properly.
- Verify `FAQPage` schema matches visible page text exactly. Plugins that generate FAQ schema from a hidden field violate this.

---

## W7. Programmatic and templated pages

WordPress makes mass page generation easy, which makes §5.3 violations easy.

- Location/service-variant pages generated from a template with a swapped city name are textbook doorway pages
- Directory, listing, and event plugins generate large volumes of near-identical thin pages — audit what they've produced and whether it's indexable
- WooCommerce product variations, faceted filters, and sort parameters generate combinatorial duplicate URLs. Check parameter handling and canonicals.
- Auto-generated author, taxonomy, and archive pages with no unique copy (see W1)

Run the near-duplicate detection in §5.2 across the *full* URL set, not just the sample — templated bloat hides in the long tail by definition.

---

## W8. Migration and restructure history

WordPress sites accumulate archaeology. Beyond §2.4:

- Permalink structure changes leave the entire old URL set orphaned. Check for `/?p=123`, dated permalinks, and old category-prefixed paths.
- Theme changes break internal links hardcoded in widgets, menus, and post content
- Plugin removals leave shortcodes rendering as literal text in old posts — grep published content for orphaned shortcode syntax
- Media library reorganisation breaks image URLs cited elsewhere
- Old staging subdomains left indexable are a full duplicate of the site

---

## W9. Performance

Rarely a citation blocker, but WordPress has predictable causes:

- Render-blocking plugin CSS and JS loading sitewide when needed on one page
- jQuery and its dependents loaded on pages that don't use them
- Unoptimised media library images served at full resolution
- Too many HTTP requests from plugin assets
- Slow TTFB from uncached database queries — usually the real problem, and usually invisible to front-end optimisation plugins

Measure field data, not just lab scores (§3.2).

---

## WordPress-specific priority order

1. Plugin conflict and duplicate metadata (W2)
2. Security plugin / firewall blocking retrieval crawlers (W5, §2.2)
3. Auto-generated URL surface and thin archives (W1)
4. Programmatic and templated page bloat (W7)
5. `Person` and entity schema completeness (W6)
6. Heading structure and semantic markup in builder pages (W3)
7. `post_modified` integrity (W4)
8. Legacy URL archaeology (W8)
