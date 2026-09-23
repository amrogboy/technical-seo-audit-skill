# Technical SEO Audit Skill

A structured technical SEO audit skill for **Claude** and **ChatGPT**. Point it at a URL (or a full site) and it runs through 13 checklist modules — crawling, indexing, rendering, Core Web Vitals, schema, meta tags, canonicalization, redirects, mobile, link health, images/video, web fundamentals, and a dedicated ecommerce module — then returns a prioritized, PRD-style findings report ready to hand to engineering or a technical SEO analyst.

## What's in this repo

- `SKILL.md` — the Claude skill. Drop the containing folder into `.claude/skills/` (Claude Code) or save it as a custom skill wherever your Claude setup loads skills from.

## What it covers

1. Crawling & Indexing (robots.txt, sitemap.xml, crawl budget, orphan pages, GSC coverage)
2. LLM Crawling & AI Visibility (llms.txt, GPTBot/ClaudeBot/PerplexityBot rules)
3. Rendering — SSR vs CSR / JS SEO
4. Core Web Vitals & Performance (LCP, INP, CLS)
5. Essential Meta Tags & Head Section (title, description, canonical, hreflang, OG/Twitter)
6. Structured Data / Schema Markup (all major Schema.org types)
7. URLs & Canonicalization
8. Redirects (chains, loops, status codes)
9. Link Health (broken links, internal linking depth, anchor text)
10. Mobile Responsiveness & Page Layout
11. Images & Video SEO
12. Web Fundamentals & Best Practices
13. Ecommerce Module (PDP schema, faceted nav, pagination, variants, OOS handling)

Each finding is scored P0 (critical) to P3 (low) and delivered in a copy-paste-ready findings table: issue, evidence, impact, fix, and suggested owner.

## Using it with Claude

Save/install the skill, then trigger it naturally, e.g.:

> Audit https://rupeezy.in/margin-trading-facility — fintech site, Next.js frontend
> > Audit https://skilldirectory.dev — ecommerce site, Next.js frontend


## Using it with ChatGPT

Claude skills (`SKILL.md`) aren't natively supported in ChatGPT. To get the same audit there:

1. Create a Custom GPT (or a Project) in ChatGPT.
2. Paste the checklist and process from `SKILL.md` into its instructions field (or ask Claude/ChatGPT to convert the modules into a system prompt).
3. Enable browsing if you want it to fetch live pages directly; otherwise paste page HTML, a Screaming Frog/Ahrefs export, or GSC/PSI data into the chat instead.

## Notes

- Works best with live page fetching for the rendering (SSR/CSR) and head-tag checks, since those require comparing raw vs. rendered HTML.
- Core Web Vitals findings from a single fetch are lab data — pair with GSC/CrUX/PageSpeed Insights field data for accuracy on real user experience.
- The ecommerce module only runs when the target site sells products.
