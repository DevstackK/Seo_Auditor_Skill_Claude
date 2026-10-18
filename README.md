# SEO Crawler Audit — a Claude Code Skill

A [Claude Code](https://claude.com/claude-code) skill that audits a URL for what search engines and AI crawlers (Googlebot, ChatGPT, Perplexity, AI Overviews) **actually see**, versus what a human sees in a browser.

## Why this exists

ChatGPT was asked to compare two AI products and completely misdescribed one of their pricing models — calling a subscription-first product "credit-based." The reason: that product's homepage was almost entirely client-rendered React, so a non-JS crawler fetch only ever saw a single incomplete JSON-LD snippet. Everything else — the real pricing tiers, the feature list, the actual plans — only existed after JavaScript hydration, invisible to anything that doesn't execute JS.

That's exactly how most of the web actually gets read by machines: Google's first indexing pass, nearly every AI search/answer tool, and most SEO scrapers fetch raw HTML first and either don't run JavaScript at all, or treat JS-rendered content as second-class. If your pricing, features, or key claims only exist after hydration, they're effectively invisible to a large share of what's reading your site.

This skill turns that diagnostic into something repeatable: fetch a page the way a crawler does, compare it to what a browser renders, validate the structured data, and tell you exactly what's missing — with the literal evidence, not a vague "some content may not be visible."

It's grounded in Google's own [AI-optimization guide for Search](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide), not general AEO folklore. That guide both validates the core premise here (pages must be "publicly accessible, crawlable," JS SEO best practices matter) and corrects a common overstatement: Google states plainly that *"structured data isn't required for generative AI search, and there's no special schema.org markup you need to add."* This skill's JSON-LD checks are framed accordingly — they help rich results, entity clarity, and factual accuracy, not "AI search requires this." It also explicitly warns against `llms.txt` files and other "AEO hack" markup Google has stated it doesn't use.

## What it does

### Mode 1 — Crawlability & Structured-Data Audit (mechanical, reliable)

Answers one precise question: **what does a crawler that does NOT execute JavaScript actually receive?**

- **Raw HTML fetch** — pulls the page with a realistic crawler UA, no JS execution
- **JS-rendered DOM capture** — pulls the same page through a real browser (Playwright/Chrome MCP) after hydration
- **Diff** — flags any visible, meaningful content (pricing, features, key claims) that only exists after JS runs, quoting the specific missing text
- **JSON-LD validation** — parses every `<script type="application/ld+json">` block, checks it's valid JSON, checks it matches the real current product state (catches bugs like a logo/URL pointing at a dead domain, or a price claim that matches nothing on the actual page), and checks for the schema.org types that help rich results and entity understanding (`Organization`, `SoftwareApplication`/`Product` with `AggregateOffer`/`OfferCatalog`, `featureList`, `FAQPage`, `Article`, `BreadcrumbList`)
- **Meta tag audit** — `<title>` length/specificity, meta description, canonical URL, robots meta, OpenGraph, Twitter Card, viewport
- **Heading structure** — exactly one `<h1>`, logical nesting
- **`robots.txt` audit** — reachable, correct `Allow`/`Disallow` rules for a production site (or confirms a staging site correctly blocks everything), valid `Sitemap:` directive
- **`sitemap.xml` audit** — reachable, valid XML, lists real pages, spot-checks that listed URLs actually resolve (not 404s)

**Output:** a table — Check | PASS/WARN/FAIL | literal evidence | fix. No vague findings.

### Mode 2 — AEO / Ranking-Signal Estimate (best-effort, explicitly bounded)

This mode is upfront about what it can't do: **no local tool can see Google's live index, backlink graph, domain authority, or the actual ranking algorithm.** It will never produce a fabricated "you'll rank #3" claim. What it *can* reasonably assess from crawlable content:

- **Query-to-content match** — does the page's crawlable text (not the full rendered DOM) directly and specifically answer a target query, near the top of the page?
- **Competitive topical coverage** — pulls current top organic results for a target query via web search and compares topic/entity coverage, as a gap list
- **E-E-A-T-adjacent signals** — author/org identity, dates, citations, supporting structured data
- **Real Core Web Vitals** — calls the actual [Google PageSpeed Insights API](https://developers.google.com/speed/docs/insights/v5/get-started) for genuine LCP/CLS/INP and Lighthouse performance/SEO/accessibility scores — not a proxy heuristic. No OAuth needed, but the keyless quota is effectively zero in practice (confirmed: a bare call 429s with `quota_limit_value: "0"`) — a free API key is required, not optional
- **AEO-specific structure** — does the page use genuine Q&A structure, short quotable lead sentences, comparison tables — the shapes LLM-based answer engines tend to extract cleanly

**Output:** qualitative findings with a confidence level (high/medium/low) per area, always closing with an explicit reminder that this is a signal estimate, not a ranking prediction — a position backed by Google's own guide, which states *"No third-party tool has access to our internal ranking or AI systems."*

### Bottom Line & Remediation (every run)

Every audit closes with two things, not just a findings table:

- **Bottom Line** — 2-4 plain-language sentences on the single most consequential finding and why it matters for that specific site, understandable by a non-technical stakeholder.
- **Remediation** — a developer-facing, leverage-ordered action list: what to change, why (tied to the specific evidence), a concrete code/schema example where non-obvious, and an honest effort estimate (never a guessed one beyond what the audit evidence supports).

Both are also published as a shareable Claude Doc (exportable to .docx/PDF), titled after the audited site, so the output is handoff-ready by default — not something you need to ask for separately.

### Known limitations (stated honestly, on purpose)

- Mode 2 is advisory, not a trackable score — by design, since fabricating a number would be worse than not having one.
- Competitive coverage depends on live web search results, so it varies run to run.
- No backlink/domain-authority data (no free API exists for this on an arbitrary site).
- No multi-page crawl — one URL per run (though you can drive it across a sitemap's URLs yourself).
- No live Google ranking position for any query — intentionally impossible to get from any local tool, and this skill refuses to fake one.
- No real Google Search Console data (indexed-page counts, real impressions/clicks) — that requires the site owner's own OAuth credentials and can't be wired in generically.

There's no official standalone "Google SEO Kit" product this integrates with — "Site Kit by Google" is a WordPress-only dashboard plugin. The individual Google APIs underneath it (PageSpeed Insights, Search Console) are the real integration points, and this skill uses the one that's actually usable keyless: PageSpeed Insights.

## Installation

This is a [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) — a Markdown file with instructions Claude follows when invoked.

### Option 1: User-level install (available in every project)

```bash
mkdir -p ~/.claude/skills/seo-crawler-audit
curl -o ~/.claude/skills/seo-crawler-audit/SKILL.md \
  https://raw.githubusercontent.com/DevstackK/Seo_Auditor_Skill_Claude/main/seo-crawler-audit/SKILL.md
```

Or clone and copy:

```bash
git clone https://github.com/DevstackK/Seo_Auditor_Skill_Claude.git
cp -r Seo_Auditor_Skill_Claude/seo-crawler-audit ~/.claude/skills/
```

### Option 2: Project-level install (only this project)

```bash
mkdir -p .claude/skills
git clone https://github.com/DevstackK/Seo_Auditor_Skill_Claude.git /tmp/seo-audit-skill
cp -r /tmp/seo-audit-skill/seo-crawler-audit .claude/skills/
```

### Verify it's installed

Start (or restart) a Claude Code session in that project and ask it to list available skills, or just invoke it directly — see Usage below.

## Usage

Once installed, it auto-triggers on phrases like:

- "SEO audit [url]"
- "AEO audit"
- "check crawlability of [url]"
- "does Google see this correctly?"
- "will AI search engines get this right?"
- "schema.org check" / "JSON-LD audit" / "structured data audit"

Or invoke it explicitly:

```
/seo-crawler-audit https://example.com/
```

Claude will then:
1. Fetch the raw HTML (no JS) and the JS-rendered DOM, and diff them
2. Validate every JSON-LD block against real site state
3. Check meta tags, headings, `robots.txt`, and `sitemap.xml`
4. Report Mode 1 findings as a PASS/WARN/FAIL table with literal evidence
5. Run Mode 2's AEO signal estimate (query-match, competitive coverage, real Core Web Vitals via PageSpeed Insights, AEO structure) unless you scope the request narrower
6. State Mode 2's limitations explicitly rather than implying more confidence than the data supports
7. Close with a Bottom Line and a prioritized developer Remediation list, and publish both as a shareable doc

## Requirements

- Claude Code with `curl`/Bash access
- A Playwright or Chrome MCP tool connected, for Mode 1's JS-rendered DOM capture (used to diff against the raw fetch)
- `WebSearch` tool access, for Mode 2's competitive topical coverage check
- No API keys required for Mode 1. Mode 2's real Core Web Vitals check needs a free [Google Cloud PageSpeed Insights API key](https://developers.google.com/speed/docs/insights/v5/get-started) — the keyless quota is effectively zero in practice, not just occasionally rate-limited.

## Security

Scanned with [Trivy](https://trivy.dev/) — 0 secrets, 0 vulnerabilities (no scannable code/dependencies exist in this Markdown-only repo). Full scan command and result in [SECURITY.md](./SECURITY.md).

## Credits

Built by [Urfan](https://www.youtube.com/@Thinkgeneng).

## License

MIT
