---
name: technical-seo-audit
description: Run a full technical SEO audit on a website or set of URLs — crawling/indexing, rendering (SSR/CSR/JS SEO), Core Web Vitals, all schema types, meta/head elements, canonicalization, redirects, mobile/layout, link health, images/video, hreflang, LLM crawling, and an ecommerce-specific module. Use when asked to "audit", "technical SEO review", "site health check", or when diagnosing indexing/ranking drops.
---

# Technical SEO Audit

Runs a structured technical SEO audit and returns a PRD-style findings report: prioritized issues, evidence, business impact, and the fix, ready to hand to engineering or a technical SEO analyst.

## Step 1 — Scope the audit

Before auditing, establish:

1. **Target**: single URL, a list of URL types (home, category/PLP, PDP, blog, landing), or full site.
2. **Rendering stack** if known (Next.js/SSR, React CSR, WordPress, Shopify, custom) — ask if unclear, it changes what to check in the JS SEO section.
3. **Ecommerce or not** — if yes, run Module 13 as well.
4. **Access available**: can fetch live pages directly (default), and/or the user can supply GSC, Ahrefs/Screaming Frog exports, or PageSpeed Insights data for more precision on Core Web Vitals and crawl stats (field data > lab data from a single fetch).

Don't block on missing exports — audit what's fetchable live, and note in the report which checks would be stronger with GSC/CWV field data or a full crawl export.

## Step 2 — Fetch and inspect

For each target URL, fetch:
- Rendered HTML (what a browser sees, post-JS)
- Raw/initial HTML (view-source equivalent, pre-JS) — the diff between the two is the core rendering check
- `/robots.txt`
- `/sitemap.xml` (and any sitemap index it references)
- `/llms.txt` if present

Then work through the checklist modules below. Mark each item **Pass / Fail / Warning / Not applicable**, and only carry Fail/Warning items into the final report — don't list passes in the report body, just count them in the summary.

## Checklist modules

### 1. Crawling & Indexing
- `robots.txt` exists, returns 200, doesn't block CSS/JS needed for rendering, doesn't accidentally disallow important paths
- Meta robots / X-Robots-Tag: no unintended `noindex`, `nofollow` on indexable pages
- `sitemap.xml` valid, referenced in robots.txt, only contains 200/indexable/canonical URLs (no 3xx/4xx/noindex/non-canonical URLs in it)
- Sitemap segmented sensibly at scale (pages/posts/products/images/video split, each under 50k URLs / 50MB)
- Crawl budget: check for infinite spaces (calendar pages, faceted nav, session IDs in URLs) that waste crawl
- Orphan pages (in sitemap/CMS but no internal links pointing to them)
- Google Search Console coverage report reviewed if available (Excluded reasons: crawled-not-indexed, discovered-not-indexed, duplicate-without-canonical)

### 2. LLM Crawling & AI Visibility
- `/llms.txt` present and correctly formatted (site summary + key page links) — increasingly relevant for AI answer engines
- Robots rules for AI crawlers reviewed deliberately (GPTBot, ClaudeBot/anthropic-ai, Google-Extended, PerplexityBot, CCBot) — flag if blocked unintentionally, or if allowed without the user having made a conscious call
- Content is extractable as clean text server-side (AI crawlers generally don't execute JS) — ties into Module 4
- Structured data present so AI systems can parse entities/facts directly (ties into Module 6)

### 3. Rendering: SSR vs CSR / JS SEO
- Compare raw HTML vs rendered HTML: is primary content (headings, body copy, product info) present in raw HTML, or only injected by JS?
- If CSR-only: flag risk — confirm via "Fetch as Google" / URL Inspection equivalent that Google's renderer sees the content; note this is a real risk for other crawlers (bots that skip rendering)
- Internal links present as real `<a href>` tags in rendered DOM, not `onclick`/JS-only navigation
- No client-side-only redirects for important routing (window.location driven redirects instead of HTTP redirects)
- Hydration doesn't shift/rewrite critical SEO elements (title, canonical, meta description) after load
- If SSR/SSG: confirm it's actually serving pre-rendered HTML (check raw response, not just framework capability)
- Infinite scroll / pagination has crawlable paginated URLs as a fallback, not JS-only loading

### 4. Core Web Vitals & Performance
- **LCP** (target <2.5s): identify LCP element, check if it's render-blocked, lazy-loaded incorrectly, or served unoptimized
- **INP** (target <200ms): heavy JS execution, long tasks, third-party scripts blocking main thread
- **CLS** (target <0.1): images/embeds without explicit width/height, web fonts causing FOUT/FOIT shift, ads/banners injected without reserved space
- TTFB, render-blocking CSS/JS, unused JS/CSS, image compression/format (WebP/AVIF), caching headers, CDN usage
- Note whether findings are from lab data (this fetch) or field data (CrUX/PSI/GSC) — flag if only lab data available

### 5. Essential Meta Tags & Head Section
- `<title>`: present, unique per page, appropriate length, primary keyword placement
- Meta description: present, unique, compelling, within length
- Canonical tag: present, self-referencing by default, absolute URL, one per page (no conflicting canonicals)
- Viewport meta tag present and correct
- Charset declared
- `hreflang` tags (if multi-locale): correct bidirectional pairing, includes self-reference, correct language-region codes, `x-default` set if applicable
- Meta robots tag correctness (covered in Module 1, verify here at head level)
- Open Graph + Twitter Card tags (og:title, og:description, og:image, twitter:card) — not core rankings but affects CTR/sharing, flag if missing
- Favicon, apple-touch-icon present
- No duplicate/conflicting tags (multiple titles, multiple canonicals) from CMS or plugin misconfig

### 6. Structured Data / Schema Markup
Validate presence, correctness, and required-property completeness (use Schema.org + Google's supported types as reference) for whichever apply:
- Organization / WebSite (with SearchAction if applicable)
- BreadcrumbList
- Article / BlogPosting (for content pages)
- FAQPage / HowTo (only where content genuinely matches — flag misuse)
- Product, Offer, AggregateRating, Review (ecommerce — see Module 13 for depth)
- LocalBusiness (if applicable)
- VideoObject / ImageObject
- WebPage / Person as needed
- No structured data errors/warnings (test via schema validation logic: missing required fields, wrong types, mismatched content e.g. review markup with no visible reviews on page)
- JSON-LD used (preferred format), not deprecated microdata unless legacy constraint

### 7. URLs & Canonicalization
- URL structure: clean, descriptive, lowercase, hyphen-separated, no unnecessary parameters/session IDs
- Consistent trailing slash handling
- Consistent protocol (all HTTPS, no mixed content)
- Consistent host (www vs non-www resolved to one, with the other 301-redirecting)
- Canonical tags point to the correct preferred version and don't create cross-domain or paginated-to-page-1 canonical errors
- Parameter handling: tracking/filter/sort parameters canonicalized to the clean URL, not indexed as duplicates
- No duplicate content served at multiple URLs without canonicalization (http/https, trailing slash, case, www variants)

### 8. Redirects
- No redirect chains (A→B→C) — flag anything more than 1 hop
- No redirect loops
- Correct status codes: 301 for permanent, 302/307 only for genuinely temporary cases
- Old/legacy URLs from migrations still redirect (spot-check known historical paths if available)
- No redirects to error pages or unrelated content (soft 404 risk)
- Internal links updated to point directly to final destination, not through a redirect

### 9. Link Health
- Internal broken links (4xx) and external broken links (dead outbound links)
- Internal linking depth: important pages reachable within 3-4 clicks from home
- Anchor text: descriptive, not generic "click here" for key internal links
- No excessive redirect hops in internal link paths (ties to Module 8)
- Nofollow used correctly (paid/UGC/sponsored links tagged appropriately, not accidentally applied to normal internal links)

### 10. Mobile Responsiveness & Page Layout
- Mobile-friendly rendering (responsive breakpoints, no horizontal scroll, no fixed-width layouts)
- Tap targets sized/spaced adequately (≥24px, enough spacing to avoid mis-taps)
- Font sizes legible without zoom (≥16px body text generally)
- No intrusive interstitials/pop-ups blocking content on mobile entry
- Layout stability on mobile specifically (CLS often worse on mobile — re-check Module 4 at mobile viewport)
- Content parity: mobile page shows same primary content/structured data as desktop (no hidden-behind-tabs content stripped from mobile DOM)

### 11. Images & Video SEO
- Descriptive, unique `alt` text on meaningful images (empty `alt=""` for purely decorative images, not missing entirely)
- Descriptive file names (not `IMG_1234.jpg`)
- Responsive images (`srcset`/`sizes`) served at appropriate size, modern formats (WebP/AVIF) with fallback
- Lazy-loading (`loading="lazy"`) on below-fold images, NOT on LCP image
- Image sitemap or images included in main sitemap for image-heavy sites
- Video: `VideoObject` schema, thumbnail, transcript/captions where relevant, hosted or embedded in a crawlable way, video sitemap if video-heavy

### 12. Web Fundamentals & Best Practices
- HTTPS site-wide, valid certificate, no mixed-content warnings
- Basic security headers present (HSTS, X-Content-Type-Options, etc.) — flag as best-practice, not core SEO
- 404 page: custom, helpful, returns actual 404 status (not 200 "soft 404")
- Consistent, logical heading hierarchy (single H1, logical H2/H3 nesting) per page
- Accessibility basics that double as SEO signals (semantic HTML, landmark regions, label associations) — flag major gaps, defer full audit to an accessibility review
- Page doesn't rely on Flash/deprecated tech
- International/legal basics if relevant (cookie consent not blocking crawlers, GDPR banner not injecting noindex accidentally)

### 13. Ecommerce Module (run only if the site sells products)
- **PDP schema**: `Product` + `Offer` (price, currency, availability, valid `priceValidUntil` if used) + `AggregateRating`/`Review` only when real reviews exist on page
- **Out-of-stock handling**: OOS products return correct availability schema, aren't 404'd/removed causing link rot, have a strategy (keep indexed with alternatives, or noindex if permanently discontinued)
- **Variants**: color/size variant URLs canonicalize correctly (either self-canonical if each is a distinct sellable/indexable unit, or canonical to parent if not)
- **Faceted/filtered navigation**: filter combinations (price range, size, brand) don't create infinite thin/duplicate crawlable URLs — check for parameter-based canonicalization, noindex on deep filter combos, or robots.txt blocking of filter parameters
- **Pagination on category/PLP pages**: paginated pages are indexable and self-canonical (not all canonicaled to page 1, which deindexes deeper products), `rel=next/prev` not required by Google but pagination should still be crawlable
- **Breadcrumbs**: present in UI and marked up with `BreadcrumbList`, matching actual category hierarchy
- **Reviews**: review schema matches visible on-page reviews (no schema without visible content — policy violation risk)
- **Internal linking**: category → subcategory → product hierarchy is crawlable via real links, not JS-only filters
- **Duplicate content risk**: near-identical PDPs (same product, different color, thin unique content) — check for adequate unique content or correct canonicalization
- **Checkout/cart pages**: correctly noindexed (shouldn't compete for rankings or leak into index)
- **Currency/region variants**: if multi-region store, hreflang + currency handled consistently with Module 5

## Step 3 — Prioritize

Score each finding:

| Priority | Definition |
|---|---|
| P0 – Critical | Blocking indexing/crawling or causing active ranking/traffic loss (noindex on live pages, broken canonical loop, sitemap full of errors, CSR hiding all content from non-JS crawlers) |
| P1 – High | Meaningful ranking/CTR/conversion impact but not actively blocking (poor CWV, missing schema on money pages, redirect chains on key paths) |
| P2 – Medium | Best-practice gaps with moderate impact (missing OG tags, alt text gaps, minor mobile UX issues) |
| P3 – Low | Polish items (favicon missing, non-critical heading structure issues) |

## Step 4 — Output the report

Deliver in this PRD-style structure:

```
# Technical SEO Audit — [Site/Section] — [Date]

## Summary
- Pages/URLs audited: N
- Pass / Warning / Fail counts by module
- Top 3 issues by business impact

## Findings

| # | Priority | Module | Issue | Evidence (URL/snippet) | Impact | Recommended Fix | Suggested Owner |
|---|----------|--------|-------|-------------------------|--------|------------------|------------------|
| 1 | P0 | Rendering | ... | ... | ... | ... | Engineering |
| 2 | P1 | Schema | ... | ... | ... | ... | Technical SEO |

## Ecommerce Findings (if applicable)
[same table format]

## What we couldn't verify from a live fetch
- (e.g., "CWV field data — pull from GSC/PSI for real user data")
- (e.g., "Full crawl scope — recommend Screaming Frog export for site-wide duplicate/canonical check")

## Next steps
Numbered, in priority order, each mapped to an owner (SEO Analyst / Technical SEO / Dev / Content).
```

Keep the findings table copy-paste ready — this is the primary deliverable, not prose explanation. Only add prose context where a finding needs it to be actionable.

## Notes on tone

Be direct. State the issue and the fix, skip caveats and hedging unless genuinely uncertain about the diagnosis (e.g., can't confirm without field data — say so once, in the "couldn't verify" section, not per-finding).
