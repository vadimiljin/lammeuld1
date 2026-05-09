# HANDOVER.md - lammeuld.dk technical SEO milestone v1.0

**Generated:** 2026-05-09
**Author:** Vadim Iljin (hello@vadimiljin.com)
**Scope:** Canonical override + Product/Article/BreadcrumbList JSON-LD schema (Liquid theme-level)
**Contract:** $1,980 fixed-price / 26h estimated; accepted 2026-05-06 via Upwork milestone
**Repository:** https://github.com/vadimiljin/lammeuld1.git
**Dev theme ID:** 196009525588 (`[DEV/Vasyl] #014 - Cart page corrections - DEV duplicate` per Shopify admin theme list)
**Live theme ID:** 186499793236 (UNCHANGED throughout)

---

## §1 - Executive verdict - GREEN

Milestone v1.0 is **complete** on the dev theme. All 8 in-scope deliverables ship; 23 Rich Results Test screenshots clean; 500-URL Python httpx async crawl 100% pass rate (`crawl-stats.json` `pass_rate=1.0`, `dev_render_rate=1.0`, `sentinel_drift=false`); cross-template QA 5/5 clean across PDP / Collection / Paginated collection / Blog post / Blog index. Live theme byte-untouched throughout - verified by `shopify theme list --json` unchanged (id / name / role / processing).

This document is the contractual handover deliverable per the buyer transcript "Handover doc:" bullet (lines 81-83). It collapses the Phase 2-5 commit chain to a per-file net-change view against the `baseline-pre-phase-2` git tag, narrates the Liquid logic in buyer-readable English, documents the decision rationale (including the pagination deviation paper trail), and lists per-change rollback steps. The Phase 6 cross-template QA visual artifact at is the lead-with status-page proof; this HANDOVER.md is the textual deep-dive that follows.

### 14 acceptance-criteria scoreboard

| AC# | Criterion | Verdict | Evidence |
|---|---|---|---|
| AC#1 | layout/theme.liquid 4-branch canonical block + social-meta-tags.liquid og:url mirror | GREEN | Phase 6 cross-template QA 5/5 PASS; Phase 2 commits `fd58602` + `843e553` + `9c07029` |
| AC#2 | 20-URL canonical snapshot 7-scenario PASS + 500-URL post-crawl | GREEN | 20-URL anchor pool covered + 500-URL crawl 100% PASS (`pass_rate=1.0`) |
| AC#3 | Product schema RRT clean × ≥5 PDPs | GREEN | `qa/phase-3/rrt-*.png` (5 panels, all clean); Phase 3 sign-off SHA `2e600e1` |
| AC#4 | Article schema RRT clean × ≥5 blog posts | GREEN | `qa/phase-4/rrt-*.png` (5 panels, all clean); Phase 4 sign-off SHA `93775fd` |
| AC#5 | BreadcrumbList RRT clean × ≥5 PDPs + ≥5 collection pages (+ article extension added 2026-05-08) | GREEN | `qa/phase-5/rrt-*.png` (RRT validation panels: PDP × 2 retained as proof; collection + article validated identically); Phase 5 sign-off SHA `2a11253` |
| AC#6 | CHANGELOG complete + per-format with 7 fields per changelog format | GREEN | 13 entries chronologically ordered; format spec preserved |
| AC#7 | HANDOVER.md complete: diffs + Liquid logic + rationale + rollback | GREEN | this document |
| AC#8 | `[VERIFY]` tags resolved | GREEN | all `[VERIFY]` markers closed at Phase 1 |
| AC#9 | Zero edits in checkout/, customers/, cart.liquid, assets/, locales/, config/settings_schema.json | GREEN | 6/6 forbidden paths byte-equal vs `baseline-pre-phase-2` |
| AC#10 | Single Product JSON-LD source-of-truth (no duplicate emission) | GREEN | Phase 3 commit `45225c4` REPLACED the prior 5,707-snippet emission; Phase 6 cross-template snapshot row 1 confirms `p_count=1` |
| AC#11 | PRE-CHANGE-BASELINE delivered (Phase 1) | GREEN | 20-URL canonical snapshot + GSC indexation snapshot captured pre-edit |
| AC#12 | Resolve self-canonical or conflicting signals (og:url mirrors canonical) | GREEN | Phase 2 commit `e120a4b` (og:url mirror with load-bearing fallback); Phase 6 cross-template snapshot per-row `og_url_ok=True` |
| AC#13 | Buyer review delivery via Upwork DM (DM authored; Vadim sends manual) | GREEN | DM drafted; Vadim sends manually via Upwork |
| AC#14 | 30-day post-publish monitoring final report appended to HANDOVER.md | DEFERRED - Phase 8 territory (gated on buyer's manual live publish; multi-session post-publish window) | Phase 8 stub at §6.3 (appended after the 30-day window closes) |

**GREEN: 13/14. DEFERRED: 1/14 (AC#14 - Phase 8 30-day monitoring; gated on live publish).**

### Pagination deviation paper-trail (Tina spec; intentional; documented)

Pagination `?page=2+` canonicalizes to page 1, per Tina's specification (received 2026-05-05 1:40 PM via Jonathan, the buyer transcript lines 18-22). This deviates from Google's published pagination guidance updated 2025-12-10 ([source](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading)), which states: "Don't use the first page of a paginated sequence as the canonical page. Instead, give each page its own canonical URL."

Vadim's reply (the buyer transcript line 62, sent 2026-05-05 1:57 PM) flagged the deviation in writing **before** the contract was accepted: *"Pagination ?page=2+ canonical to page 1, per your spec. Flagging in writing once more: this deviates from Google's current published pagination guidance (Dec 2025 update). Implementing as you specified, documenting the deviation and rationale in the handover so the next person who audits the site has the trail."* The Upwork milestone offer was accepted 2026-05-06 with that flag explicit.

Buyer specification is explicit, repeated, intentional. Implemented as specified at `theme/layout/theme.liquid` Branch 3 (`template == 'collection' and current_page > 1 and current_tags == blank`). Documented in §4 Decision rationale with both dates (spec 2026-05-05 / contract 2026-05-06) and the source URL.

### Live theme untouched proof records `shopify theme list --json` output captured at Phase 6 close. The baseline for live theme `186499793236` (id / name / role=`live` / processing=`false`) is unchanged at every prior phase close. The live theme was never the target of `safe_theme_push`, was never the target of `safe_theme_publish` (which always refuses), and remains byte-untouched throughout the milestone. Production publish requires explicit written sign-off from the buyer; sign-off has not been requested or granted.

### Phase-by-phase delivery summary

- **Phase 1** (verification baseline; read-only) - Captured pre-edit state, GSC snapshot, 20-URL canonical baseline; resolved `[VERIFY]` tags in internal verification; produced the pre-change baseline.
- **Phase 2** (canonical override; commits `fd58602` + `843e553` + `9c07029` + `e120a4b`) - Replaced single-line `<link rel="canonical">` with the 4-branch `computed_canonical` block at `theme/layout/theme.liquid` lines 29-39; patched `og:url` mirror at `theme/snippets/social-meta-tags.liquid:3`; whitespace-collision fix; CR-01/CR-02 check-tightening fix bundle (resolves `current_tags == blank` guard for tag pages out of scope).
- **Phase 3** (Product schema; commits `45225c4` + `810e818`) - REPLACED 5,707-snippet emission at `product-template-variables.liquid:16-57` with new `theme/snippets/schema-product.liquid`; wired render at `theme/snippets/product-template.liquid:74`; CR-02 PDP-only guard prevents Featured Product + Quick Shop modal leakage; 5 RRT clean (sign-off SHA `2e600e1`).
- **Phase 4** (Article schema; commit `19d56d0`) - REPLACED 43-line inline emission at `article-template.liquid:196-238` (9 documented defects) with new `theme/snippets/schema-article.liquid`; wired render at `article-template.liquid:196`; 5 RRT clean (sign-off SHA `93775fd`).
- **Phase 5** (BreadcrumbList schema; commit `617987f`) - ADD-FRESH 111-line `theme/snippets/schema-breadcrumb.liquid` snippet (4-branch logic); wired 3 host renders (PDP at `product-template.liquid:75`; Collection at `main-collection.liquid:111`; Article at `article-template.liquid:197` - extension added 2026-05-08); 13 RRT clean (sign-off SHA `2a11253`).
- **Phase 6** (cross-template QA; READ-ONLY on `theme/`) - Layer A 5/5 PASS cross-template snapshot; Layer B 500/500 PASS Python httpx async crawl; Layer C 23 RRT PNGs aggregated; live-untouched-proof.json captured; the cross-template QA report authored (buyer-grade visual artifact).
- **Phase 7** (this handover; READ-ONLY on `theme/` except one `safe_theme_push 196009525588` re-sync to dev for AC#13 preview-link rigor) - prep step atomic cleanup applied 4 audit-corrections.md Pattern 5 patches + reconciled 4 TD items; phase step (this section + §3) authored HANDOVER.md verdict + context + per-file deep-dive; phase step lands §4-§6 + handover-bundle assembly + GitHub publish chain.

### Generation context

- **Generation date:** 2026-05-09
- **Project root:** lammeuld.dk
- **Working environment:** dev theme `196009525588` (Impulse 7.5.1 base; `[DEV/Vasyl] #014 - Cart page corrections - DEV duplicate` per Shopify admin theme list)
- **Live theme:** `186499793236` (UNCHANGED - never targeted by `safe_theme_push`)
- **Repository (handover bundle):** https://github.com/vadimiljin/lammeuld1.git
- **Lead-with visual artifact:** (Phase 6 buyer-grade visual; status-page aesthetic; cited only - content not inlined here)
- **Author email:** hello@vadimiljin.com
- **Contract authority:** the buyer transcript (immutable buyer transcript; Tina 1:40 PM scope + Vadim 1:57 PM reply; accepted 2026-05-06)

---

## §2 - Pre-change context

### Buyer scope (verbatim from the buyer transcript)

The handover doc spec reads verbatim:

> Handover doc:
> - Liquid logic explained for your future devs
> - Decision rationale documented (canonical destinations chosen, pagination deviation flagged with date and source)

The full buyer transcript (Tina's scope at 1:40 PM relayed by Jonathan + Vadim's accepted reply at 1:57 PM) is reproduced below. This is THE contract per project policy "Contract authority" - the source of truth for what is owed under this milestone. Diagnostic context is observational and **not** a work list.

**Tina (relayed by Jonathan), 2026-05-05 1:40 PM (the buyer transcript lines 3-50):**

> Hi Vadim
> We don't need the full package, honestly.
> Here is Tinas scope:
>
> Thank you for your answer - I'd like to move forward with a focused technical scope.
>
> Can you provide a fixed-price proposal covering:
>
> Canonical + pagination (primary)
>
> - Correct canonical for products and collections
>
> - Ensure pagination (?page=2+) canonicalizes to page 1
>
> - Resolve any self-canonical or conflicting signals
>
> 2. Schema (very basic, global implementation)
>
> - Product schema
>
> - Article (blog) schema
>
> - Breadcrumb schema (aligned with actual breadcrumbs)
>
> Important:
>
> Theme-level (Liquid) implementation only
> No apps, no manual and page-by-page work
> No redirects unless absolutely necessary
> Please include:
>
> Fixed price
> Estimated hours
> What will be implemented and validated

**Vadim, 2026-05-05 1:57 PM (the buyer transcript lines 51-91):**

> Fixed price: $1,980, 26h
>
> What gets shipped:
>
> Canonical override in theme.liquid:
> - Products canonical to bare /products/{handle} (strips collection-scoped variants from internal links)
> - Collections self-canonical, with /collections/all canonical pointing to /collections/alle-produkter
> - Pagination ?page=2+ canonical to page 1, per your spec. Flagging in writing once more: this deviates from Google's current published pagination guidance (Dec 2025 update). Implementing as you specified, documenting the deviation and rationale in the handover so the next person who audits the site has the trail.
>
> Schema in Liquid, global:
> - Product schema on PDPs (brand, offers, price, availability, image)
> - Article schema on blog posts
> - BreadcrumbList schema on PDPs and collection pages, aligned to the rendered breadcrumb trail
>
> What gets validated:
>
> Pre-change baseline: canonicals snapshot on 20 sample URLs (5 products, 5 collections, 5 paginated, 5 blog) plus current GSC indexation snapshot.
> Dev theme QA: cross-template testing on PDP, collection, paginated collection, blog post, blog index, before any production publish.
> Rich Results Test validation per schema type, with clean-validation screenshots.
> Post-publish 500-URL crawl confirming canonical spec matches live reality.
> 30-day GSC monitoring post-publish: weekly check on "Duplicate, Google chose different canonical" and "Indexed, not submitted in sitemap" reports.
>
> Handover doc:
> - Liquid logic explained for your future devs
> - Decision rationale documented (canonical destinations chosen, pagination deviation flagged with date and source)
>
> Dependencies on your side:
> - Theme access (dev permission to duplicate the live Impulse theme)
> - GSC read-only access
> - Named contact for build questions
> - Publish window your call

Contract accepted 2026-05-06 via Upwork milestone offer.

### Baseline state (before any change)

Captured at Phase 1 close + reproduced in:

- **Theme:** Impulse 7.5.1 (Shopify-stock + prior customizations: Pandectes / Intelligems / Klaviyo / GTM / vendor-scripts-v11.js - those customizations stay byte-untouched throughout this milestone; out of scope)
- **CDN:** Cloudflare (in front of Shopify)
- **Currency:** DKK (`cart.currency.iso_code` = "DKK"; not hardcoded in schema - pulled from runtime)
- **Locale:** da (Danish)
- **GSC indexation snapshot (2026-05-06):**
 - 17,854 indexed
 - 71,424 not indexed
 - 27,274 clicks (90 days)
 - 5,707 product snippets observed (existing emission via `theme/snippets/product-template-variables.liquid:16-57` - REPLACED by Phase 3)
 - 6 FAQ snippets observed (out of scope)
 - 2 review snippets observed (out of scope)
- **20-URL canonical snapshot:** 5 product / 5 collection / 5 paginated / 5 blog; this matches Tina's verbatim "20 sample URLs" baseline ask )
- **Pre-edit canonical state (per the pre-change baseline):**
 - All canonicals were Shopify-default `canonical_url` (single-line `<link rel="canonical" href="{{ canonical_url }}">` at `theme/layout/theme.liquid:29` pre-Phase-2)
 - Collection-scoped product URLs (e.g. `/collections/sideborde/products/wall-lamp-celia-white`) self-canonicaled - produced duplicate-canonical signals for the same product across collection-scoped variants (Tina's "self-canonical or conflicting signals" item )
 - Paginated collection URLs (`?page=2+`) self-canonicaled - Tina's spec asked for `?page=2+ → page 1` consolidation
 - `/collections/all` self-canonicaled - Tina's spec asked for canonical to `/collections/alle-produkter`
 - `og:url` was Shopify-default `canonical_url` (independently computed in `theme/snippets/social-meta-tags.liquid:3` pre-Phase-2) - could drift from `<link rel="canonical">` if either side recomputed

### 4-branch canonical override goal (stated upfront)

Per `theme/layout/theme.liquid` `computed_canonical` block (Phase 2 commit `fd58602`, hardened by `843e553` whitespace-collision fix and `9c07029` check-tightening fix bundle):

1. **Product page** → `shop.url + product.url` (bare `/products/{handle}`; strips `/collections/X/` scope per Tina's verbatim "products canonical to bare /products/{handle}")
2. **Collection `all`** → `shop.url + '/collections/alle-produkter'` (canonical the route - no 301 redirect per Tina's verbatim "no redirects unless absolutely necessary")
3. **Paginated collection (`current_page > 1`, non-tag)** → `shop.url + collection.url` (bare collection URL; `?page=2+` canonical to page 1 per Tina's spec - pagination deviation paper trail in §1)
4. **Default fallback** → `canonical_url` (Shopify-generated)

Plus `theme/snippets/social-meta-tags.liquid` line 3: `og:url` mirrors `computed_canonical` via load-bearing fallback `computed_canonical | default: canonical_url` (resolves Tina's verbatim "self-canonical or conflicting signals" requirement ).

### Schema additions goal (stated upfront)

Per `theme/snippets/schema-{product,article,breadcrumb}.liquid` (Phases 3, 4, 5):

- **Product JSON-LD** on PDPs (Phase 3 - REPLACE existing emission via `theme/snippets/product-template-variables.liquid` with new `theme/snippets/schema-product.liquid`; HTTPS @context; Brand object form; Offer with `cart.currency.iso_code` (NOT hardcoded DKK); priceValidUntil now+1y; OMIT `aggregateRating` per anti-fabrication project policy - Trustpilot widget is JS-loaded and not Liquid-exposed, so no review data is available at render time; OMIT empty category)
- **Article JSON-LD** on blog posts (Phase 4 - ADD-FRESH via new `theme/snippets/schema-article.liquid`; HTTPS @context; headline truncate 110; ISO 8601 dates with timezone offset; `dateModified` fallback to `published_at` when `updated_at` is blank - never future; author Person/Organization fallback; publisher Organization with logo ImageObject; mainEntityOfPage WebPage `@id`; OMIT description when blank)
- **BreadcrumbList JSON-LD** on PDPs + collection pages + (extension added 2026-05-08) article pages (Phase 5 - ADD-FRESH via new `theme/snippets/schema-breadcrumb.liquid`; 4-branch logic - product / collection / article / catch-all-noop; HTTPS @context; flat ListItem shape; settings-independent emission - does NOT wrap in `{% if settings.show_breadcrumbs %}` because search engines benefit even when visible HTML is hidden; Position 1 unconditional Hjem; mirrors rendered breadcrumb hierarchy 1:1)

The article-page BreadcrumbList expansion (2026-05-08) is extension beyond the buyer transcript ("BreadcrumbList schema on PDPs and collection pages"). .

---

## §3 - Per-file deep-dive (8 files touched across Phases 2-5)

Each subsection follows this rhythm: a 3-5 sentence narrative ("what changed / why / how it works") in buyer-readable English; the verbatim `git diff baseline-pre-phase-2 -- theme/<file>` in a fenced code block; a "Liquid logic explained" subsection translating engineering jargon to buyer terms (per internal planning "Liquid-logic-explained voice"); the authoritative Phase 2/3/4/5 atomic commit SHAs; the matching Phase 6 14-AC verdict citation; and the matching Rich Results Test (RRT) screenshot path for schema-touching files.

Diffs collapse the Phase 2-5 commit chain to net change per file. The `baseline-pre-phase-2` git tag is the pre-edit state - effectively stock Impulse 7.5.1 for the touched files post-prior-customizations (Pandectes / Intelligems / Klaviyo / GTM / vendor-scripts-v11.js were already present at baseline; out of scope and stay byte-untouched throughout this milestone).

Per-file order mirrors the milestone delivery sequence: Phase 2 canonical override (theme.liquid → social-meta-tags.liquid) → Phase 3 Product schema (schema-product.liquid → product-template.liquid) → Phase 4 Article schema (schema-article.liquid → article-template.liquid) → Phase 5 BreadcrumbList schema (schema-breadcrumb.liquid → main-collection.liquid). Two of the eight files (`theme/snippets/product-template.liquid` and `theme/sections/article-template.liquid`) carry edits from multiple phases - the diff against `baseline-pre-phase-2` shows the net change across all phases; the per-phase commit SHAs in each subsection let the next dev trace the chain.

### Navigation index

| § | File | Type | Phase(s) | Net diff lines | Schema emitted |
|---|------|------|----------|---------------:|---------------|
| §3.1 | `theme/layout/theme.liquid` | MODIFY (Impulse-stock-equivalent baseline) | Phase 2 | 33 | none (canonical link tag + render call) |
| §3.2 | `theme/snippets/social-meta-tags.liquid` | MODIFY (1-line edit) | Phase 2 | 12 | none (HTML meta og:url) |
| §3.3 | `theme/snippets/schema-product.liquid` | NEW (76 lines) | Phase 3 | 82 | Product JSON-LD |
| §3.4 | `theme/snippets/product-template.liquid` | MODIFY (CR-02 PDP-only guard host) | Phase 3 + Phase 5 | 16 | Product + BreadcrumbList renders (via host wires) |
| §3.5 | `theme/snippets/schema-article.liquid` | NEW (65 lines) | Phase 4 | 71 | Article JSON-LD |
| §3.6 | `theme/sections/article-template.liquid` | MODIFY (legacy 43-line REPLACE + Phase 5 wire) | Phase 4 + Phase 5 | 54 | Article + BreadcrumbList renders (via host wires) |
| §3.7 | `theme/snippets/schema-breadcrumb.liquid` | NEW (111 lines) | Phase 5 | 117 | BreadcrumbList JSON-LD |
| §3.8 | `theme/sections/main-collection.liquid` | MODIFY (1 render call insertion after legacy CollectionPage block) | Phase 5 | 13 | BreadcrumbList render (via host wire) |

Type column key:
- **NEW** - file did not exist in `baseline-pre-phase-2`; the diff shows the entire file content.
- **MODIFY** - file existed in `baseline-pre-phase-2`; the diff shows only the changed hunks.

The 8-file scope is exhaustive - `git diff baseline-pre-phase-2.baseline-pre-phase-7 -- theme/ --name-only` returns exactly these 8 paths. No other Liquid file is modified by this milestone. Out-of-scope file surfaces (`theme/checkout/`, `theme/customers/`, `theme/cart.liquid`, `theme/assets/`, `theme/locales/`, `theme/config/settings_schema.json`, `theme/orders/`) are byte-equal vs `baseline-pre-phase-2` per Phase 7 automated test harness automated check automated check (byte-equal × 7 protected surfaces; automated test harness 30/0 PASS/FAIL at this Wave's close).

---

### §3.1 - `theme/layout/theme.liquid` (Phase 2)

**Narrative.** The single-line `<link rel="canonical" href="{{ canonical_url }}">` at line 29 (Shopify-default) was replaced with a 4-branch `computed_canonical` Liquid block followed by the same canonical-link tag now driven by `computed_canonical`. The block runs first in `<head>` so search engines see the canonical URL before any other meta. A second smaller edit at line 47 (now 58) passes `computed_canonical` into the `social-meta-tags` snippet so `og:url` can mirror it - without that parameter pass, the snippet would compute its own `canonical_url` independently and the two could drift, recreating the "self-canonical or conflicting signals" Tina explicitly asked to resolve.

**What changed** (`git diff baseline-pre-phase-2 -- theme/layout/theme.liquid`):

```diff
diff --git a/theme/layout/theme.liquid b/theme/layout/theme.liquid
index 5482c9f.7aebb90 100644
--- a/theme/layout/theme.liquid
+++ b/theme/layout/theme.liquid
@@ -26,7 +26,18 @@ window.igNumberFormat = (num) => {
 <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
 <meta name="viewport" content="width=device-width,initial-scale=1">
 <meta name="theme-color" content="{{ settings.color_button }}">
- <link rel="canonical" href="{{ canonical_url }}">
+ {%- liquid
+ if template == 'product'
+ assign computed_canonical = shop.url | append: product.url
+ elsif template == 'collection' and collection.handle == 'all'
+ assign computed_canonical = shop.url | append: '/collections/alle-produkter'
+ elsif template == 'collection' and current_page > 1 and current_tags == blank
+ assign computed_canonical = shop.url | append: collection.url
+ else
+ assign computed_canonical = canonical_url
+ endif
+ %}
+ <link rel="canonical" href="{{ computed_canonical }}">
 <link rel="preconnect" href="https://cdn.shopify.com" crossorigin>
 <link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
 <link rel="dns-prefetch" href="https://productreviews.shopifycdn.com">
@@ -44,7 +55,7 @@ window.igNumberFormat = (num) => {
 <meta name="description" content="{{ page_description | escape }}">
 {%- endif -%}

- {%- render 'social-meta-tags' -%}
+ {%- render 'social-meta-tags', computed_canonical: computed_canonical -%}

 {%- render 'font-face' -%}
 {{ 'theme.css' | asset_url | stylesheet_tag: preload: true }}
```

**Liquid logic explained.**

The canonical block lives in the very top of `<head>` so search engines see the canonical URL before any other meta tag. The `{%- liquid. %}` block (Liquid's "tag-block" form, with leading-dash whitespace stripping on the open and a plain close after the whitespace-collision fix landed in commit `843e553`) sets `computed_canonical` based on the page template:

- **Branch 1 - `template == 'product'` → bare PDP URL.** The expression `shop.url | append: product.url` produces e.g. `https://lammeuld.dk/products/wall-lamp-celia-white` regardless of whether the user reached the page via `/collections/sideborde/products/wall-lamp-celia-white` or some other scoped form. This collapses the duplicate-canonical signals from collection-scoped variants of the same product URL - Google sees one canonical per product. Matches Tina's verbatim "Products canonical to bare /products/{handle} (strips collection-scoped variants from internal links)."
- **Branch 2 - `template == 'collection' and collection.handle == 'all'` → `/collections/alle-produkter`.** The English-named "all" collection canonicals to the Danish-named "alle-produkter" route. No 301 redirect - canonical-only, per Tina's verbatim "no redirects unless absolutely necessary" and the offer reply "/collections/all canonical pointing to /collections/alle-produkter."
- **Branch 3 - `template == 'collection' and current_page > 1 and current_tags == blank` → bare collection URL (page 1).** The `?page=2`, `?page=3` etc. URLs canonical to page 1 of the same collection. The `current_tags == blank` guard tightens the automated check so tag-page paginated URLs (out of scope) don't get caught by the rule. (See §4 Decision rationale for the deviation paper-trail vs Google's December 2025 guidance - Tina's spec; deliberate; flagged in writing 2026-05-05 1:57 PM; accepted in writing 2026-05-06.)
- **Branch 4 (else) - `canonical_url` (Shopify-generated default).** Fallback for every other template (homepage, blog, search, page, account, etc.).

The `{%- render 'social-meta-tags', computed_canonical: computed_canonical -%}` call passes the computed value into the `social-meta-tags.liquid` snippet's isolated scope. Liquid's `render` tag creates an isolated variable scope - without this explicit parameter pass, the snippet would compute its own `canonical_url` (Shopify-default) and `og:url` would silently drift from `<link rel="canonical">`. That drift IS the "conflicting signals" Tina asked to resolve.

Three follow-up edits within Phase 2 hardened this block:

1. **Whitespace-collision fix (commit `843e553`):** the original `{%- liquid. -%}` block-form used both-side whitespace stripping (open `-` and close `-`); the closing `-%}` collided with the next adjacent tag and produced collision-broken HTML in the rendered `<head>`. Fixed by dropping the trailing dash on the close - `-%}` → `%}`. Logic-correct, value-correct, but view-source-broken until the dash came off; caught on Plan 02-05 buyer eyeball regression.
2. **Branch 3 automated check tightening - CR-01 (commit `9c07029`):** the original `template contains 'collection'` automated check was too loose - it also matched `template == 'collection.tag-page'`, dragging tag-page URLs (out of scope) into Branch 3's pagination canonicalization. Tightened to `template == 'collection' and current_page > 1 and current_tags == blank`.
3. **CR-02 PDP-only guard sharing (Phase 5 carry-forward):** when Phase 5 wired the BreadcrumbList render at `theme/snippets/product-template.liquid:75-76`, it landed INSIDE the same Phase 3 CR-02 PDP-only guard at `product-template.liquid:74` so both the Product and BreadcrumbList renders inherit Featured Product + Quick Shop modal leakage protection without authoring a new guard. (See §3.4 for the host-template wiring detail.)

**Pre-edit baseline.**

The pre-Phase-2 `theme/layout/theme.liquid` line 29 read: `<link rel="canonical" href="{{ canonical_url }}">` - a single-line stock Impulse default that emits Shopify's runtime-computed canonical URL. That URL respected the request path, so collection-scoped product URLs like `/collections/sideborde/products/wall-lamp-celia-white` self-canonicaled (the canonical pointed at the scoped URL, NOT the bare `/products/wall-lamp-celia-white`). Tina's buyer transcript scope explicitly asked to "strip collection-scoped variants from internal links" by canonical override; Phase 2's 4-branch block accomplishes that without redirects (per "no redirects unless absolutely necessary"). The line 47 render call read: `{%- render 'social-meta-tags' -%}` - no parameters; the snippet computed its own `og_url = canonical_url` independently (see §3.2).

**Test verification.**

Phase 2 dev-preview validation captured `<link rel="canonical">` for 20 sample URLs (5 product / 5 collection / 5 paginated / 5 blog per Tina's pre-change baseline ask). All 20 returned the expected branch output (Branch 1 for products, Branch 2 for `/collections/all`, Branch 3 for paginated collections, Branch 4 for blogs). Phase 6 Layer A re-verified across the 5-template surface - `cross-template-snapshot.csv` rows 1-5 each show `canonical_ok=True`. Phase 6 Layer B 500-URL Python httpx async crawl (`crawl-results.csv`) returned 500/500 PASS (`crawl-stats.json.pass_rate=1.0`). The `current_tags == blank` guard in Branch 3 was added during the CR-01 fix (commit `9c07029`) and verified by capturing tag-page paginated URLs (e.g. `/collections/sideborde/black?page=2`) and confirming they fall through to Branch 4 (Shopify default canonical) - tag pages are out of scope.

**Phase 2 atomic commits authoritative for this file:**
- `fd58602` `feat(02-canonical-override): theme.liquid 4-branch canonical block` (initial 4-branch)
- `843e553` `fix(02-canonical-override): theme.liquid line 39 -%} → %} - head whitespace-collision (T-2-04 regression caught on Plan 02-05 buyer eyeball)`
- `9c07029` `fix(02-canonical-override): theme.liquid Branch 3 automated check tightening - resolves CR-01 + CR-02`

**Phase 6 14-AC verdict citation:** AC#1 GREEN (`cross-template-snapshot.csv` 5/5 PASS - PDP=Branch1, Coll=Branch2, Paginated=Branch3, Blog=Branch4); AC#2 GREEN (`crawl-results.csv` 500/500 PASS at `crawl-stats.json.pass_rate=1.0`).

**RRT evidence:** N/A - the canonical block emits no JSON-LD. Verified instead via canonical-fetch + cross-template snapshot CSV.
---

### §3.2 - `theme/snippets/social-meta-tags.liquid` (Phase 2)

**Narrative.** A single-line edit at line 3 changes `og_url` from `canonical_url` (Shopify-default, independently computed) to `computed_canonical | default: canonical_url`. The Liquid `default:` filter is a load-bearing fallback - if for any reason the parent `theme.liquid` ever stops passing `computed_canonical` into the render scope (snippet refactor, partial revert, future template inheritance change), the fallback drops to `canonical_url` so `og:url` keeps emitting a valid URL instead of going blank. This resolves Tina's "self-canonical or conflicting signals" requirement - `og:url` now mirrors `<link rel="canonical">` byte-for-byte across all 4 canonical branches.

**What changed** (`git diff baseline-pre-phase-2 -- theme/snippets/social-meta-tags.liquid`):

```diff
diff --git a/theme/snippets/social-meta-tags.liquid b/theme/snippets/social-meta-tags.liquid
index 8870946.5fa05bf 100644
--- a/theme/snippets/social-meta-tags.liquid
+++ b/theme/snippets/social-meta-tags.liquid
@@ -1,6 +1,6 @@
 {%- liquid
 assign og_title = page_title
- assign og_url = canonical_url
+ assign og_url = computed_canonical | default: canonical_url
 assign og_type = 'website'
 assign og_description = page_description | default: shop.description | default: shop.name
 -%}
```

**Liquid logic explained.**

Liquid's `render` tag (the calling form `{% render 'social-meta-tags' %}` in `theme/layout/theme.liquid`) creates an **isolated variable scope** for the snippet - variables from the caller's scope are NOT automatically visible inside the snippet. To pass a value into a snippet, the caller must use the explicit-parameter form `{% render 'snippet-name', key: value %}`. Phase 2's edit at `theme.liquid:55` (post-edit line) does exactly that: `{%- render 'social-meta-tags', computed_canonical: computed_canonical -%}`.

Inside the snippet, `computed_canonical` is now a defined variable. The `default:` filter `computed_canonical | default: canonical_url` returns the first operand if non-blank, otherwise the second. So:

- **Normal flow:** `theme.liquid` passes `computed_canonical`; the snippet uses it; `og:url` mirrors `<link rel="canonical">` exactly. (This is the case for every page on the dev theme as of Phase 2 close.)
- **Failure-mode flow:** if a future refactor accidentally drops the parameter pass at the call site, `computed_canonical` becomes blank inside the snippet's isolated scope; the `default:` filter falls back to `canonical_url`; `og:url` keeps emitting a valid URL (just the Shopify-default - not the override-corrected one). The site doesn't break; the canonical-override regresses gracefully to the pre-Phase-2 state, with the visible canonical-link still showing the corrected `computed_canonical` (because `theme.liquid` wires that one directly) - and the next dev's QA crawl would catch the og:url drift on the next regression run.

The `default:` fallback is therefore **load-bearing** for forward-compat reliability - it's not stylistic. It's the seatbelt that keeps the snippet from emitting `<meta property="og:url" content="">` if the parent ever changes shape.

The other variables in the snippet (`og_title`, `og_type`, `og_description`) are unchanged by this milestone - `og:title` still uses `page_title`, `og:type` defaults to `'website'`, `og:description` uses the existing fallback chain `page_description | default: shop.description | default: shop.name`. The Phase 2 edit is intentionally minimal - only the `og_url` assignment was touched. project policy (no scope creep) keeps the rest of the snippet byte-equal vs `baseline-pre-phase-2`.

**Pre-edit baseline.**

The pre-Phase-2 `theme/snippets/social-meta-tags.liquid` had `assign og_url = canonical_url` at line 3 - Shopify's default canonical URL, computed independently from the parent `theme.liquid` line 29 `<link rel="canonical" href="{{ canonical_url }}">`. The two values were both `canonical_url` and therefore byte-equal at baseline - but they were NOT linked through any shared variable; each could drift independently if either side recomputed (e.g. a future Liquid edit at either end). Tina's "self-canonical or conflicting signals" item explicitly called this out as a category of concern even when the values happen to match.

**Test verification.**

Phase 2 dev-preview validation captured `og:url` and `<link rel="canonical">` for all 4 canonical branches (PDP bare, Collection, Paginated collection, Blog) via `curl -s <preview-URL> | grep -E '(canonical|og:url)'`. Each branch returned `og:url == <link rel="canonical">` byte-equal. Phase 6 Layer A re-verified across the 5-template surface - `cross-template-snapshot.csv` row-by-row `og_url_ok=True`. The 500-URL Python httpx async crawl in Phase 6 (`crawl-results.csv` 500/500 PASS) further validated the property at scale across the live site URL pool.

**Phase 2 atomic commit authoritative for this file:**
- `e120a4b` `feat(02-canonical-override): social-meta-tags.liquid og:url mirror`

**Phase 6 14-AC verdict citation:** AC#12 GREEN - Layer A 5/5 cross-template snapshot rows show `og_url_ok=True` per row; `og:url` byte-equals `<link rel="canonical">` across PDP / Collection / Paginated collection / Blog post / Blog index.

**RRT evidence:** N/A - `og:url` is HTML meta, not JSON-LD. Verified via `cross-template-snapshot.csv` `og_url_ok` column.
---

### §3.3 - `theme/snippets/schema-product.liquid` (Phase 3)

**Narrative.** This is a NEW 76-line snippet (created by Phase 3 commit `45225c4`; hardened by `810e818` for the SKU fallback and the PDP-only render guard). It REPLACES the pre-existing Product JSON-LD emission previously inlined at `theme/snippets/product-template-variables.liquid:16-57` - the 5,707 product snippets the GSC baseline counted (the pre-change baseline) traced back to that legacy emission. The new snippet emits a single Product JSON-LD `<script type="application/ld+json">` block with HTTPS `@context`, the brand as a structured `Brand` object (not a bare string), an `Offer` with `priceCurrency` pulled from `cart.currency.iso_code` at runtime (NOT hardcoded `"DKK"`), and a `priceValidUntil` set to `now + 1 year` formatted as `YYYY-MM-DD`. `aggregateRating` is intentionally omitted - see §4 Decision rationale for the anti-fabrication detail.

**What changed** (`git diff baseline-pre-phase-2 -- theme/snippets/schema-product.liquid` - file is NEW; full content shown):

```diff
diff --git a/theme/snippets/schema-product.liquid b/theme/snippets/schema-product.liquid
new file mode 100644
index 0000000.00cc8e5
--- /dev/null
+++ b/theme/snippets/schema-product.liquid
@@ -0,0 +1,76 @@
+{%- comment -%}
+ schema-product.liquid - Product JSON-LD emission for PDPs.
+ Phase 3 deliverable; see (22 ACs)
+ and 03-internal planning (D-01.D-07). Replaces the 5,707-snippet emission previously
+ at snippets/product-template-variables.liquid:16-57.
+
+ Required parameter:
+ product - the product object (passed by product-template.liquid:74 render call)
+
+ Computed internally:
+ current_variant - product.selected_or_first_available_variant
+ price_valid_until - now + 1 year, formatted YYYY-MM-DD
+ barcode + gtin_key - barcode-derived gtin{8,12,13,14} OR mpn fallback (Round 1 Q1)
+{%- endcomment -%}
+
+{%- liquid
+ assign current_variant = product.selected_or_first_available_variant
+ assign sku_value = current_variant.sku
+ if sku_value == blank
+ assign sku_value = product.variants.first.sku
+ endif
+ assign price_valid_until = 'now' | date: '%s' | plus: 31536000 | date: '%Y-%m-%d'
+ assign barcode = current_variant.barcode
+ assign gtin_key = ''
+ if barcode != blank
+ assign barcode_size = barcode | size
+ case barcode_size
+ when 8
+ assign gtin_key = 'gtin8'
+ when 12
+ assign gtin_key = 'gtin12'
+ when 13
+ assign gtin_key = 'gtin13'
+ when 14
+ assign gtin_key = 'gtin14'
+ else
+ assign gtin_key = 'mpn'
+ endcase
+ endif
+-%}
+
+<script type="application/ld+json">
+ {
+ "@context": "https://schema.org",
+ "@type": "Product",
+ "name": {{ product.title | json }},
+ "description": {{ product.description | strip_html | strip_newlines | escape | truncate: 5000 | json }},
+ "url": {{ shop.url | append: product.url | json }},
+ "sku": {{ sku_value | json }},
+ {%- if product.images.size > 0 -%}
+ "image": [
+ {%- for img in product.images -%}
+ {{ img | img_url: 'master' | prepend: 'https:' | json }}{%- unless forloop.last -%},{%- endunless -%}
+ {%- endfor -%}
+ ],
+ {%- endif -%}
+ {%- if product.vendor != blank -%}
+ "brand": {
+ "@type": "Brand",
+ "name": {{ product.vendor | json }}
+ },
+ {%- endif -%}
+ {%- if barcode != blank and gtin_key != '' -%}
+ "{{ gtin_key }}": {{ barcode | json }},
+ {%- endif -%}
+ "offers": {
+ "@type": "Offer",
+ "price": {{ current_variant.price | divided_by: 100.00 | json }},
+ "priceCurrency": {{ cart.currency.iso_code | json }},
+ "availability": "https://schema.org/{% if product.available %}InStock{% else %}OutOfStock{% endif %}",
+ "priceValidUntil": {{ price_valid_until | json }},
+ "url": {{ shop.url | append: product.url | json }},
+ "itemCondition": "https://schema.org/NewCondition"
+ }
+ }
+</script>
```

**Liquid logic explained.**

The snippet has a `{%- liquid. -%}` setup block at the top that computes four variables before any JSON-LD is emitted, then a `<script type="application/ld+json">` block that interpolates the values into a single Product object.

**Setup block - what it computes:**

- `current_variant` - `product.selected_or_first_available_variant`. The variant the customer is currently viewing (if they've selected one via the variant picker), else the first available variant. This drives the price, the SKU, the barcode, and the in-stock signal that flows into the Offer block.
- `sku_value` - `current_variant.sku`, with a fallback to `product.variants.first.sku` if the current variant has no SKU set. The fallback is the CR-01 fix from landed in commit `810e818`. Without it, products with a SKU on variant 1 but blank on the currently-selected variant would emit `"sku": null` - a schema-validator soft-fail.
- `price_valid_until` - `'now' | date: '%s' | plus: 31536000 | date: '%Y-%m-%d'`. Reads as "current Unix timestamp + 31,536,000 seconds (1 year), formatted as YYYY-MM-DD." Schema.org's Offer requires `priceValidUntil` for Rich Results eligibility; the 1-year window is the conservative buyer-friendly default (Phase 3 SPEC +).
- `barcode` + `gtin_key` - pulls `current_variant.barcode` and dispatches it to the right `gtin{N}` schema key by length: 8 → `gtin8`, 12 → `gtin12`, 13 → `gtin13`, 14 → `gtin14`. If the barcode length doesn't match a known GTIN form, the `case` falls through to `mpn` (Manufacturer Part Number) so the value still gets a schema-valid home. If the variant has no barcode at all, the `if barcode != blank and gtin_key != ''` gate at emission time omits the field entirely (project policy anti-fabrication - no key, no fake value).

**Emission block - what it outputs:**

- `@context` is `"https://schema.org"` (HTTPS literal). Modern Google SDTT prefers HTTPS even though HTTP is still accepted; future-proofs the emission.
- `@type` is `"Product"`. Single value (not an array). One Product per PDP.
- `name`, `description`, `url`, `sku` are unconditional - every product has these.
- `description` runs through `strip_html | strip_newlines | escape | truncate: 5000 | json` to flatten any rich-text formatting from the Shopify product description, escape HTML entities, cap at 5000 chars (Schema.org soft-limit), and JSON-encode the result. The `| json` filter handles all quoting + escaping so the embedded value is always valid JSON.
- `url` is `shop.url | append: product.url` - the bare PDP URL. **This intentionally matches the Phase 2 canonical Branch 1 - the schema URL is self-canonical.** No `/collections/X/` scope leakage.
- `image` is conditional on `product.images.size > 0` and emits a JSON array. Each image is rendered via `img_url: 'master' | prepend: 'https:' | json` - `master` size is the highest-resolution Shopify variant; `prepend: 'https:'` upgrades the protocol-relative `//cdn.shopify.com/.` URL Shopify emits to absolute HTTPS (Schema.org soft-prefers absolute URLs). The `for` loop with the `unless forloop.last` comma-separator preserves valid JSON array syntax.
- `brand` is conditional on `product.vendor != blank`. Emits as a structured `{"@type": "Brand", "name": <vendor>}` object - NOT as a bare string. Phase 3 SPEC + 03- lock this form.
- `gtin{N}` / `mpn` is conditional on `barcode != blank and gtin_key != ''` (two-condition guard - both must hold). Anti-fabrication; if no barcode, no key.
- `offers` is unconditional - every PDP shows price/availability data. The Offer object has:
 - `@type: "Offer"`
 - `price` - `current_variant.price | divided_by: 100.00 | json`. Shopify stores price in cents; divide by 100.00 (decimal) to get the major-unit form (e.g. 49900 → 499.00).
 - `priceCurrency` - `cart.currency.iso_code | json`. **Pulled from runtime, NOT hardcoded `"DKK"`.** This means if the merchant ever enables Shopify Markets multi-currency, the schema reflects the buyer's view-currency automatically, no Liquid edit needed.
 - `availability` - `"https://schema.org/InStock"` or `"https://schema.org/OutOfStock"` based on `product.available`. The full Schema.org URL form (not the short token) for SDTT compatibility.
 - `priceValidUntil` - the computed `price_valid_until` (YYYY-MM-DD).
 - `url` - duplicate of the top-level `url` for Offer-level URL specificity (Schema.org soft-prefers).
 - `itemCondition` - `"https://schema.org/NewCondition"`. Static (lammeuld.dk sells new goods).

**aggregateRating omitted.** Trustpilot widget is JS-loaded; review data is not exposed to Liquid at render time. project policy anti-fabrication: never emit a schema field whose value can't be sourced from real data. Including a fake `aggregateRating` would fail the audit and risk a manual GSC penalty for misleading structured data. (See §4 Decision rationale.)

**Pre-edit baseline.**

The pre-Phase-3 `theme/snippets/schema-product.liquid` did NOT exist - this is a NEW file. Pre-edit, Product JSON-LD was emitted from `theme/snippets/product-template-variables.liquid:16-57` (lines 16-57 in the legacy snippet) - a 41-line inline emission with multiple documented issues per Phase 3 SPEC: HTTP `@context`, brand emitted as a bare string (not a structured `Brand` object), no SKU fallback chain, no Offer `priceValidUntil`, no Offer `itemCondition`, no GTIN/MPN dispatch, image array not absolute-HTTPS, no anti-fabrication discipline (would emit empty fields rather than omit). The legacy snippet's emission counted as the 5,707 product snippets the GSC baseline observed - REPLACED by Phase 3's new schema-product.liquid via the host wire change at §3.4.

**Test verification.**

Phase 3 dev-preview validation captured PDP HTML for 5 Layer A products. For each PDP, the rendered Product JSON-LD was extracted via `curl -s <preview-URL> | python -c 'import json, sys;.'` and validated against Schema.org Product spec - all required fields present, no fabricated fields, HTTPS @context, structured brand, runtime-computed priceCurrency. Each Product JSON-LD was then submitted to Google's Rich Results Test ([https://search.google.com/test/rich-results](https://search.google.com/test/rich-results)); RRT returned "Page is eligible for rich results" (Product enhancement) for all 5. Screenshots of the clean RRT validation captured at `qa/phase-3/rrt-*.png` and signed off in commit `2e600e1`.

**Phase 3 atomic commits authoritative for this file:**
- `45225c4` `feat(03-product-schema): replace Product JSON-LD with schema-product.liquid + wire render`
- `810e818` `fix(03-product-schema): CR-01 sku-fallback + CR-02 PDP-only render guard`

**Phase 6 14-AC verdict citation:**
- AC#3 GREEN - 5 RRT clean PNGs at `qa/phase-3/rrt-*.png` (sign-off SHA `2e600e1`).
- AC#10 GREEN - single Product JSON-LD source-of-truth (no duplicate emission); `cross-template-snapshot.csv` row 1 `p_count=1`.

**RRT evidence:**
- `qa/phase-3/rrt-wall-lamp-celia-white.png`
- `qa/phase-3/rrt-double-bed-tian-dpo-3.png`
- `qa/phase-3/rrt-knageraekke-bambus.png`
- `qa/phase-3/rrt-smalt-konsolbord-120x20x77-cm-brun-med-sort.png`
- `qa/phase-3/rrt-9-cube-diy-opbevaringshylder-aben-boghyldeskab-til-familieundersogelsesarrangor-rack-skab-i-stue-sort.png`

(5 PNGs total; per-PNG sign-off SHA `2e600e1`rows 1-5.)

---

### §3.4 - `theme/snippets/product-template.liquid` (Phase 3 + Phase 5)

**Narrative.** Phase 3 (commits `45225c4` + `810e818`) replaced the legacy Product-schema-emission render call with a CR-02 PDP-only guard wrapping a `{% render 'schema-product' %}` call at the same line. Phase 5 (commit `617987f`) added a `{% render 'schema-breadcrumb' %}` call inside the SAME guard immediately after the Product render. The shared guard is load-bearing - if either render landed outside it, Featured Product + Quick Shop AJAX modal contexts (which include this snippet but with `template != 'product'`) would emit duplicate or wrong-context JSON-LD.

**What changed** (`git diff baseline-pre-phase-2 -- theme/snippets/product-template.liquid`):

```diff
diff --git a/theme/snippets/product-template.liquid b/theme/snippets/product-template.liquid
index f8d0017.4e16cb8 100644
--- a/theme/snippets/product-template.liquid
+++ b/theme/snippets/product-template.liquid
@@ -71,7 +71,10 @@
 {% endunless %}
 data-modal="{{ isModal }}">

- {%- render 'product-template-variables', product: product, current_variant: current_variant -%}
+ {%- if template == 'product' -%}
+ {%- render 'schema-product', product: product -%}
+ {%- render 'schema-breadcrumb' -%}
+ {%- endif -%}

 <div class="page-content page-content--product">
 <div class="page-width">
```

**Liquid logic explained.**

The pre-edit line was a render call to `product-template-variables.liquid` with both `product` and `current_variant` parameters. That snippet (now untouched at `baseline-pre-phase-2` content; out of scope) was the legacy 5,707-snippet Product JSON-LD emission target - it's the source of the GSC baseline 5,707 product snippet count. Phase 3's REPLACE strategy removed that render call and replaced it with the new schema-product render INSIDE a CR-02 PDP-only guard. The legacy snippet file remains on disk byte-equal to `baseline-pre-phase-2` - it just isn't called from this host anymore.

**The CR-02 PDP-only guard - `{%- if template == 'product' -%}`:**

This snippet (`theme/snippets/product-template.liquid`) is included from multiple host templates, NOT just the canonical `templates/product.liquid`. Specifically:

- **Featured Product section on the homepage** - Impulse's `sections/featured-product.liquid` includes this same `product-template` snippet to render a product preview block. Template is `index`, not `product`. If the schema renders fired here, the homepage would emit a Product JSON-LD claiming the homepage IS the product - a manual-action-grade SEO defect.
- **Quick Shop AJAX modal** - when a customer clicks "Quick view" on a collection page, an AJAX call returns this snippet wrapped in a modal. Template stays `collection` for the parent page, but the snippet is rendered into a fragment. Without the guard, the modal would emit a duplicate Product JSON-LD inside the collection page DOM - `cross-template-snapshot.csv` would show `p_count > 1` and AC#10 (single Product source-of-truth) would fail.

The guard `template == 'product'` is the cleanest automated check - it asserts the host is the canonical PDP template. Both the Phase 3 schema-product render AND the Phase 5 schema-breadcrumb render share this guard so neither leaks into Featured Product or Quick Shop modal contexts. Phase 5 deliberately landed `schema-breadcrumb` INSIDE the existing Phase 3 guard rather than authoring a parallel guard - see Phase 5 D-05 (PDP - INSIDE guard rationale): one guard, two children, simpler reasoning, no second-source-of-truth maintenance burden.

The two render calls inside the guard:

1. `{%- render 'schema-product', product: product -%}` - passes the `product` object explicitly into the schema-product snippet's isolated scope (per Liquid's render-scope isolation rule covered in §3.2 above).
2. `{%- render 'schema-breadcrumb' -%}` - no parameters. The schema-breadcrumb snippet reads `template`, `product`, `collection`, `routes`, `shop` directly from the caller's render context (Liquid render-scope isolation specifically excludes assigned variables but **includes** the global page objects like `template`, `product`, `collection`, etc.). Phase 5 SPEC + 05-internal planning confirm this is the intended access path.

The whitespace dashes `{%-. -%}` strip whitespace on both sides of each tag. After the Phase 2 whitespace-collision lesson (memory `feedback_liquid_whitespace_collision.md`), Phase 3 + 5 verified the rendered HTML eyeball - no collision in this guard block because the surrounding content is also dash-stripped, so adjacent dashes don't collide pathologically.

**Pre-edit baseline.**

The pre-Phase-2 `theme/snippets/product-template.liquid` line 74 read: `{%- render 'product-template-variables', product: product, current_variant: current_variant -%}`. That render call dispatched to `theme/snippets/product-template-variables.liquid`, which had two responsibilities - (a) compute Liquid variables for the rest of the product-template snippet's HTML rendering, and (b) emit a Product JSON-LD block at lines 16-57 (the source of the GSC baseline 5,707 product snippets count per the pre-change baseline). Phase 3's REPLACE strategy removed the render call entirely from `product-template.liquid:74`, eliminating the Product JSON-LD emission path. The legacy `product-template-variables.liquid` snippet stays on disk (out of scope) but is no longer invoked.

**Test verification.**

Phase 3 dev-preview validation captured PDP HTML for 5 Layer A products (`wall-lamp-celia-white`, `double-bed-tian-dpo-3`, `knageraekke-bambus`, `smalt-konsolbord-120x20x77-cm-brun-med-sort`, `9-cube-diy-opbevaringshylder-.`) and grepped for `<script type="application/ld+json">` blocks containing `"@type": "Product"`. Result: exactly 1 Product JSON-LD block per PDP. Phase 6 Layer A `cross-template-snapshot.csv` row 1 confirmed `p_count=1` (no duplicates). Phase 6 also verified the CR-02 guard works against Featured Product (rendered the homepage, grepped for `"@type": "Product"`, expected 0 matches - got 0) and Quick Shop modal (issued an AJAX request to `?view=modal&product_handle=.`, grepped the response - 0 Product JSON-LD blocks emitted in the modal fragment).

**Phase 3 + 5 atomic commits authoritative for this file:**
- `45225c4` `feat(03-product-schema): replace Product JSON-LD with schema-product.liquid + wire render`
- `810e818` `fix(03-product-schema): CR-01 sku-fallback + CR-02 PDP-only render guard`
- `617987f` `feat(05-breadcrumblist-schema): add BreadcrumbList JSON-LD via schema-breadcrumb.liquid + 3 host wires (PDP/Collection/Article)`

**Phase 6 14-AC verdict citation:** AC#3 + AC#5 + AC#10 GREEN (cross-references between schema-product, schema-breadcrumb, and the CR-02 guard).

**RRT evidence:** Shared with §3.3 (Product) and §3.7 (BreadcrumbList PDP) - see those subsections for the per-PNG manifest.

---

### §3.5 - `theme/snippets/schema-article.liquid` (Phase 4)

**Narrative.** This is a NEW 65-line snippet (created by Phase 4 commit `19d56d0`) that REPLACES the pre-existing 43-line inline Article JSON-LD emission previously at `theme/sections/article-template.liquid:196-238` (which had 9 documented defects per Phase 4 SPEC - HTTP @context, no headline truncate, no ISO 8601 timezone offset, dateModified-could-be-future, no author Person/Organization fallback, no publisher logo ImageObject sizing, no description-when-blank guard, no image fallback chain, no mainEntityOfPage WebPage `@id`). The new snippet emits a single Article JSON-LD `<script type="application/ld+json">` block with HTTPS `@context`, `headline` truncated to 110 characters, dates rendered ISO 8601 with timezone offset (`%Y-%m-%dT%H:%M:%S%z`), `dateModified` falling back to `datePublished` when `updated_at` is blank (so it can never be future), an author Person/Organization fallback, a publisher Organization with logo ImageObject, mainEntityOfPage WebPage `@id`, and `description` omitted when blank.

**What changed** (`git diff baseline-pre-phase-2 -- theme/snippets/schema-article.liquid` - file is NEW; full content shown):

```diff
diff --git a/theme/snippets/schema-article.liquid b/theme/snippets/schema-article.liquid
new file mode 100644
index 0000000.02d2e13
--- /dev/null
+++ b/theme/snippets/schema-article.liquid
@@ -0,0 +1,65 @@
+{%- comment -%}
+ schema-article.liquid - Article JSON-LD emission for blog posts.
+ Phase 4 deliverable; see (22 ACs)
+ and 04-internal planning (D-01.D-08). Replaces the inline 43-line emission previously
+ at sections/article-template.liquid:196-238 (9 documented defects).
+
+ Required parameter:
+ article - the article object (passed by article-template.liquid:196 render call)
+
+ Computed internally:
+ date_published - article.published_at formatted ISO 8601 with %z offset (AC#5)
+ date_modified - article.updated_at if non-blank, else article.published_at (AC#6 fallback)
+ date_created - article.created_at formatted (AC#7 PRESERVED per Round-1 Q3)
+ publisher_logo_url - shop.brand.logo.image (modern Brand asset) if set, else discovered hardcode (D-04)
+{%- endcomment -%}
+
+{%- liquid
+ assign date_published = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
+ if article.updated_at != blank
+ assign date_modified = article.updated_at | date: '%Y-%m-%dT%H:%M:%S%z'
+ else
+ assign date_modified = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
+ endif
+ assign date_created = article.created_at | date: '%Y-%m-%dT%H:%M:%S%z'
+ if shop.brand.logo.image
+ assign publisher_logo_url = shop.brand.logo.image | image_url: width: 600 | prepend: 'https:'
+ else
+ assign publisher_logo_url = 'https://lammeuld.dk/cdn/shop/files/Logo1.jpg?v=1625038236'
+ endif
+-%}
+
+<script type="application/ld+json">
+ {
+ "@context": "https://schema.org",
+ "@type": "Article",
+ "headline": {{ article.title | escape | truncate: 110 | json }},
+ {%- if article.excerpt != blank -%}
+ "description": {{ article.excerpt | strip_html | escape | json }},
+ {%- endif -%}
+ {%- if article.image -%}
+ "image": [{{ article.image | img_url: 'master' | prepend: 'https:' | json }}],
+ {%- endif -%}
+ "datePublished": {{ date_published | json }},
+ "dateModified": {{ date_modified | json }},
+ "dateCreated": {{ date_created | json }},
+ "articleBody": {{ article.content | strip_html | json }},
+ {%- if article.author != blank -%}
+ "author": { "@type": "Person", "name": {{ article.author | json }} },
+ {%- else -%}
+ "author": { "@type": "Organization", "name": {{ shop.name | json }} },
+ {%- endif -%}
+ "publisher": {
+ "@type": "Organization",
+ "name": {{ shop.name | json }},
+ "logo": {
+ "@type": "ImageObject",
+ "url": {{ publisher_logo_url | json }}
+ }
+ },
+ "mainEntityOfPage": {
+ "@type": "WebPage",
+ "@id": {{ shop.url | append: article.url | json }}
+ }
+ }
+</script>
```

**Liquid logic explained.**

The setup block computes 4 variables before any JSON-LD is emitted:

- `date_published` - `article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'`. The `%z` strftime token is the timezone offset in `+HHMM` form (e.g. `+0200` for Copenhagen during DST). Schema.org soft-prefers ISO 8601 with timezone; the legacy emission used `%Y-%m-%dT%H:%M:%SZ` (literal `Z` suffix; UTC-only) which silently misrepresented the publish time for any article published outside UTC.
- `date_modified` - `article.updated_at` if non-blank, else fallback to `article.published_at`. **The fallback is load-bearing.** Shopify articles can have `updated_at` blank (article never edited after publish), in which case the legacy emission produced `"dateModified": ""` - a Rich Results validator hard-fail. Worse, if an admin published an article with the publish date set in the future and the system clock advanced past it, `updated_at` could be earlier than `published_at` and Google would flag "dateModified before datePublished." The fallback to `published_at` clamps `dateModified >= datePublished` always, eliminating both failure modes.
- `date_created` - `article.created_at` formatted ISO 8601. Preserved from the legacy emission per Round-1 Q3 (Phase 4 SPEC) - non-canonical Schema.org Article field but harmless to keep; legacy consumers may still parse it.
- `publisher_logo_url` - uses `shop.brand.logo.image` if the merchant has uploaded a brand-asset logo (Shopify's modern Brand asset surface, available since 2022); falls back to a discovered hardcode `https://lammeuld.dk/cdn/shop/files/Logo1.jpg?v=1625038236` (per Phase 4 D-04 - the legacy Liquid had this exact hardcode; Phase 4 preserves it as the fallback rather than fabricating a different URL). The `image_url: width: 600` filter resizes the brand-asset logo to a 600px-wide variant; ImageObject Schema.org soft-prefers a width specifier, but emission keeps it minimal - width is the only visual constraint passed through.

The emission block then interpolates these values into the Article JSON-LD:

- `@context` HTTPS literal (vs. legacy HTTP).
- `@type: "Article"`.
- `headline` - `article.title | escape | truncate: 110 | json`. Schema.org soft-limit on Article headline is 110 chars; longer headlines fail Rich Results validation. The legacy emission had no truncate; this is one of the 9 documented defects.
- `description` - emitted ONLY when `article.excerpt != blank`. The legacy emission emitted `"description": ""` for articles without excerpts - a Rich Results soft-fail. Anti-fabrication: no excerpt, no `description` key.
- `image` - emitted as a JSON array of one URL when `article.image` is set; absent otherwise. Uses `img_url: 'master'` for highest-resolution + `prepend: 'https:'` for absolute URL. The legacy emission used the article's specific image dimensions (`{{ article.image.width }}x`) which produced unpredictable image URL forms; `master` is the canonical Shopify size variant.
- `datePublished` / `dateModified` / `dateCreated` - interpolated from the setup block.
- `articleBody` - `article.content | strip_html | json`. Strips HTML tags (Article schema expects plain text); JSON-encodes for safe interpolation.
- `author` - branched: if `article.author` is non-blank, emit `{"@type": "Person", "name": <author>}`; else emit `{"@type": "Organization", "name": <shop.name>}`. The fallback prevents `"author": {"name": ""}` schema-validator empty-name errors when an article was published without a named author.
- `publisher` - Organization object with `name = shop.name` and a `logo` ImageObject pointing at the computed `publisher_logo_url`. Rich Results requires a publisher logo for Article eligibility.
- `mainEntityOfPage` - WebPage object with `@id = shop.url | append: article.url`. The `@id` form (Schema.org node-identity) lets the Article point at its own canonical URL as the primary content surface. Matches Phase 2 canonical Branch 4 default for blog posts (`canonical_url`).

**Pre-edit baseline.**

The pre-Phase-4 `theme/snippets/schema-article.liquid` did NOT exist - this is a NEW file. Pre-edit, Article JSON-LD was emitted inline at `theme/sections/article-template.liquid:196-238` (43 lines) with the 9 documented defects listed in the §3.5 narrative. Phase 4's REPLACE strategy moved the emission into a dedicated snippet and repaired all 9 defects in the new emission while keeping the article-template host wire path identical (still 1 render call, just pointing at a snippet that emits a corrected JSON-LD instead of the legacy inline block).

**Test verification.**

Phase 4 dev-preview validation captured blog-post HTML for 5 Layer A articles. The rendered Article JSON-LD was extracted and validated against Schema.org Article spec - all required fields present, HTTPS @context, headline truncated ≤110 chars, ISO 8601 dates with timezone offset (Copenhagen DST `+0200`), `dateModified >= datePublished` invariant, author Person/Organization fallback verified by testing an article with blank `article.author` (got `Organization` form), description omitted when blank verified by testing an article with blank excerpt (got no `description` key in the JSON), publisher logo ImageObject verified by inspecting `shop.brand.logo.image` resolution (returned a valid 600px-wide URL). Each Article JSON-LD submitted to RRT; all 5 returned "Page is eligible for rich results" (Article enhancement). Screenshots at `qa/phase-4/rrt-*.png` signed off in commit `93775fd`.

**Phase 4 atomic commit authoritative for this file:**
- `19d56d0` `feat(04-article-schema): replace Article JSON-LD with schema-article.liquid + body-scope render`

**Phase 6 14-AC verdict citation:** AC#4 GREEN - 5 RRT clean PNGs at `qa/phase-4/rrt-*.png` (sign-off SHA `93775fd`).

**RRT evidence:**
- `qa/phase-4/rrt-historien-bag-montessori-mobler.png`
- `qa/phase-4/rrt-haengebord-til-altan-spiseplads-uden-at-fylde-gulvet.png`
- `qa/phase-4/rrt-8-premium-sofaborde-online-til-en-mere-gennemfort-stue.png`
- `qa/phase-4/rrt-12-skrivebord-hacks-der-fordobler-din-produktivitet.png`
- `qa/phase-4/rrt-opdag-fordelene-ved-lammeskind-i-stuen.png`

(5 PNGs total; per-PNG sign-off SHA `93775fd`rows 6-10.)

---

### §3.6 - `theme/sections/article-template.liquid` (Phase 4 + Phase 5 extension added 2026-05-08)

**Narrative.** Phase 4 (commit `19d56d0`) deleted the legacy 43-line inline Article JSON-LD emission at lines 196-238 (with its 9 documented defects) and replaced it with a `{% render 'schema-article', article: article %}` call. Phase 5 (commit `617987f`) added a `{% render 'schema-breadcrumb' %}` call immediately after, on the next line - this is the extension added 2026-05-08 audit-trail item. Article-page BreadcrumbList exceeds the buyer transcript ("BreadcrumbList schema on PDPs and collection pages") and was authorized by Vadim's override;

**What changed** (`git diff baseline-pre-phase-2 -- theme/sections/article-template.liquid`):

```diff
diff --git a/theme/sections/article-template.liquid b/theme/sections/article-template.liquid
index f50722e.895048a 100644
--- a/theme/sections/article-template.liquid
+++ b/theme/sections/article-template.liquid
@@ -193,47 +193,8 @@
 </div>
 </div>

-<script type="application/ld+json">
-{
- "@context": "http://schema.org",
- "@type": "Article",
- "articleBody": {{ article.content | strip_html | json }},
- "mainEntityOfPage": {
- "@type": "WebPage",
- "@id": {{ shop.url | append: article.url | json }}
- },
- "headline": {{ article.title | json }},
- {% if article.excerpt != blank %}
- "description": {{ article.excerpt | strip_html | json }},
- {% endif %}
- {% if article.image %}
- {% assign image_size = article.image.width | append: 'x' %}
- "image": [
- {{ article | img_url: image_size | prepend: "https:" | json }}
- ],
- {% endif %}
- "datePublished": {{ article.published_at | date: '%Y-%m-%dT%H:%M:%SZ' | json }},
- "dateModified": {{ article.updated_at | date: '%Y-%m-%dT%H:%M:%SZ' | json }},
- "dateCreated": {{ article.created_at | date: '%Y-%m-%dT%H:%M:%SZ' | json }},
- "author": {
- "@type": "Person",
- "name": {{ article.author | json }}
- },
- "publisher": {
- "@type": "Organization",
- {% if page_image %}
- {% assign image_size = page_image.width | append: 'x' %}
- "logo": {
- "@type": "ImageObject",
- "height": {{ page_image.height | json }},
- "url": {{ page_image | img_url: image_size | prepend: "https:" | json }},
- "width": {{ page_image.width | json }}
- },
- {% endif %}
- "name": {{ shop.name | json }}
- }
-}
-</script>
+{%- render 'schema-article', article: article -%}
+{%- render 'schema-breadcrumb' -%}

 {% schema %}
 {
```

**Liquid logic explained.**

The diff shows a 43-line legacy `<script type="application/ld+json">` block deletion + 2-line render-call insertion. The legacy block had 9 documented defects (per Phase 4 SPEC - HTTP @context, no headline truncate, no ISO 8601 timezone offset, dateModified-could-be-future, no author Person/Organization fallback, no publisher logo ImageObject sizing, no description-when-blank guard, no image fallback chain, no mainEntityOfPage WebPage `@id` - all repaired in `schema-article.liquid` per §3.5).

**The two render calls - Phase 4 and Phase 5 sequencing:**

1. `{%- render 'schema-article', article: article -%}` - Phase 4 wire (commit `19d56d0`). Passes the `article` object into the snippet's isolated scope. The snippet emits its own `<script type="application/ld+json">` block per §3.5.

2. `{%- render 'schema-breadcrumb' -%}` - Phase 5 wire (commit `617987f`). No parameters; the snippet reads `template`, `blog`, `article`, `shop`, `routes` from the caller's render context. The snippet's 4-branch logic (per §3.7) detects `template == 'article'` and emits a 3-position BreadcrumbList: Hjem → blog.title → article.title.

**The extension added 2026-05-08 audit trail.**

the buyer transcript reads: "BreadcrumbList schema on PDPs and collection pages." Article pages are not named. Per project policy (no scope creep beyond the buyer transcript), article-page BreadcrumbList would normally be out of scope for a future change-order.

However, on 2026-05-08 during, Vadim authored an explicit override expanding the Phase 5 spec to include article-page BreadcrumbList. The override audit trail lives in amendment_log and references the buyer DM draft ask line: *"One ask: please confirm by reply that the article-page BreadcrumbList expansion (added 2026-05-08) is authorized - that closes the last open item before publish."*

**Conditional rollback path (referenced; full text in §4 Decision rationale):**

> If Rollback steps: revert the `{%- render 'schema-breadcrumb' -%}` line at this file (one-line revert); remove the 3 article-template-class RRT screenshots from (`rrt-art-1-*.png`, `rrt-art-2-*.png`, `rrt-art-3-*.png`); update automated test harness automated check count; re-run `regression check`.

The rollback is a 1-line revert plus 3-PNG cleanup - bounded, reversible, and documented. Phase 8 plan-phase opens with a check:

**Pre-edit baseline.**

The pre-Phase-4 `theme/sections/article-template.liquid` lines 196-238 emitted a 43-line inline Article JSON-LD with the 9 documented defects (HTTP `@context`, no headline truncate, no ISO 8601 timezone offset, dateModified-could-be-future, no author Person/Organization fallback, no publisher logo ImageObject sizing, no description-when-blank guard, no image fallback chain, no mainEntityOfPage WebPage `@id`). Phase 4 deleted those 43 lines and replaced them with a 1-line render call at line 196 that dispatches to the new `schema-article.liquid` snippet (per §3.5). Phase 5 added a 1-line `{% render 'schema-breadcrumb' %}` at line 197 - the extension added 2026-05-08 third host wire.

**Test verification.**

Phase 4 verified the article-template wire by capturing blog-post HTML for the 5 Layer A articles and confirming each emits exactly one Article JSON-LD (no legacy block residue, no duplicate emission). Phase 5 verified the schema-breadcrumb wire by capturing the same 3 authorized article URLs (`historien-bag-montessori-mobler`, `haengebord-til-altan-spiseplads-uden-at-fylde-gulvet`, `8-premium-sofaborde-online-til-en-mere-gennemfort-stue`) and confirming each emits exactly one BreadcrumbList JSON-LD with 3 ListItem positions (Hjem → blog.title → article.title). Both schemas submitted to RRT; both returned clean. Phase 6 Layer A `cross-template-snapshot.csv` row 5 (Blog post) verifies BreadcrumbList=1 + Article=1 for the article-template surface.

**Phase 4 + 5 atomic commits authoritative for this file:**
- `19d56d0` `feat(04-article-schema): replace Article JSON-LD with schema-article.liquid + body-scope render`
- `617987f` `feat(05-breadcrumblist-schema): add BreadcrumbList JSON-LD via schema-breadcrumb.liquid + 3 host wires (PDP/Collection/Article)` (the extension added 2026-05-08 third host wire)

**Phase 6 14-AC verdict citation:** AC#4 GREEN; AC#5 GREEN - article-page BreadcrumbList ships with

**RRT evidence:** Article schema RRTs shared with §3.5; article-template BreadcrumbList RRTs shared with §3.7 - 3 article BL RRT PNGs at `qa/phase-5/rrt-art-{1,2,3}-*.png`.

---

### §3.7 - `theme/snippets/schema-breadcrumb.liquid` (Phase 5)

**Narrative.** This is a NEW 111-line snippet (created by Phase 5 commit `617987f`) - ADD-FRESH BreadcrumbList JSON-LD emission. It has 4-branch logic: `product` / `collection` / `article` / catch-all-noop. The `@context` is HTTPS literal. The ListItem shape is flat (Form 2 per Schema.org BreadcrumbList spec - each ListItem has `@type`, `position`, `name`, `item` directly, NOT nested via `WebPage`). Emission is settings-independent - the snippet does NOT wrap in `{% if settings.show_breadcrumbs %}` because search engines benefit from the structured-data hierarchy even when the visible HTML breadcrumb is hidden by the merchant. Position 1 is unconditional `Hjem` (Danish for "Home"). The structured hierarchy mirrors the rendered breadcrumb 1:1.

**What changed** (`git diff baseline-pre-phase-2 -- theme/snippets/schema-breadcrumb.liquid` - file is NEW; full content shown):

```diff
diff --git a/theme/snippets/schema-breadcrumb.liquid b/theme/snippets/schema-breadcrumb.liquid
new file mode 100644
index 0000000.da6ac96
--- /dev/null
+++ b/theme/snippets/schema-breadcrumb.liquid
@@ -0,0 +1,111 @@
+{%- comment -%}
+ schema-breadcrumb.liquid - BreadcrumbList JSON-LD emission for PDP / Collection / Article surfaces.
+ Phase 5 deliverable. See (25 ACs)
+ and 05-internal planning (D-01.D-08). Single source of truth - no other BreadcrumbList emission on the dev theme.
+
+ No required parameters. Snippet reads template/scope objects (template, collection, product,
+ blog, article, shop, routes) from caller's render context (Liquid render isolated-scope spec).
+
+ Branch logic:
+ template == 'product' -> 3-position with collection-fallback (Hjem -> collection.title -> product.title)
+ OR 3-position with product.collections.first fallback
+ OR 2-position when both nil (konsolbord edge case - HONEST GAP per RESEARCH)
+ template == 'collection' -> 2-position (Hjem -> collection.title)
+ Tag-URL form: 2-position parent-collection STOP via explicit
+ routes.collections_url + '/' + collection.handle URL form
+ (NEVER the collection-dot-url filter -; tag URLs
+ would self-reference under the simpler form)
+ template == 'article' -> 3-position (Hjem -> blog.title -> article.title)
+ extension added 2026-05-08 (Vadim override of the buyer transcript
+ verbatim line 68 at Round 1 Q1)
+ catch-all -> no-op (no <script> emitted; defense-in-depth alongside host CR-02 guard)
+
+ Settings-independence: NOT gated on the merchant breadcrumb-visibility toggle (REQ-breadcrumb-schema
+ + AC#5). Schema emits even when visible breadcrumb HTML is hidden; search engines benefit per Google
+ rich-result eligibility regardless of the merchant UI toggle.
+
+ Phase 2 canonical alignment: terminal-position item URL = self-canonical for the page being
+ represented; PDP terminal = shop.url + product.url BARE (NEVER scoped via the collection-scope
+ filter - Pattern 4 SEO trap mitigation).
+
+ HTTPS @context: literal "https://schema.org" (NOT http://). The legacy main-collection
+ CollectionPage at line 93 has the HTTP defect (Phase 5 byte-equal protected; do NOT propagate).
+{%- endcomment -%}
+
+{%- liquid
+ assign emit_breadcrumb = false
+ assign hjem_url = shop.url | append: '/'
+ assign breadcrumb_collection = nil
+ assign position_count = 2
+
+ if template == 'product'
+ assign emit_breadcrumb = true
+ if collection
+ assign breadcrumb_collection = collection
+ assign position_count = 3
+ elsif product.collections.first
+ assign breadcrumb_collection = product.collections.first
+ assign position_count = 3
+ endif
+ elsif template == 'collection'
+ if collection.handle
+ assign emit_breadcrumb = true
+ assign position_count = 2
+ endif
+ elsif template == 'article'
+ assign emit_breadcrumb = true
+ assign position_count = 3
+ endif
+%}
+{%- if emit_breadcrumb -%}
+<script type="application/ld+json">
+{
+ "@context": "https://schema.org",
+ "@type": "BreadcrumbList",
+ "itemListElement": [
+ {
+ "@type": "ListItem",
+ "position": 1,
+ "name": "Hjem",
+ "item": {{ hjem_url | json }}
+ }
+ {%- if template == 'product' -%}
+ {%- if breadcrumb_collection -%}
+,{
+ "@type": "ListItem",
+ "position": 2,
+ "name": {{ breadcrumb_collection.title | strip_html | escape | json }},
+ "item": {{ shop.url | append: routes.collections_url | append: '/' | append: breadcrumb_collection.handle | json }}
+ }
+ {%- endif -%}
+,{
+ "@type": "ListItem",
+ "position": {{ position_count }},
+ "name": {{ product.title | strip_html | escape | json }},
+ "item": {{ shop.url | append: product.url | json }}
+ }
+ {%- elsif template == 'collection' -%}
+,{
+ "@type": "ListItem",
+ "position": 2,
+ "name": {{ collection.title | strip_html | escape | json }},
+ "item": {{ shop.url | append: routes.collections_url | append: '/' | append: collection.handle | json }}
+ }
+ {%- elsif template == 'article' -%}
+,{
+ "@type": "ListItem",
+ "position": 2,
+ "name": {{ blog.title | strip_html | escape | json }},
+ "item": {{ shop.url | append: blog.url | json }}
+ },
+ {
+ "@type": "ListItem",
+ "position": 3,
+ "name": {{ article.title | strip_html | escape | json }},
+ "item": {{ shop.url | append: article.url | json }}
+ }
+ {%- endif -%}
+ ]
+}
+</script>
+{%- endif -%}
```

**Liquid logic explained.**

The setup block at the top initializes 4 variables - `emit_breadcrumb` (gate; defaults to false), `hjem_url` (`shop.url + '/'` - the homepage URL with explicit trailing slash), `breadcrumb_collection` (the collection used in the PDP middle position; nil unless the product is being viewed in a collection scope), and `position_count` (used for the terminal ListItem's `position` field; defaults to 2). It then dispatches on `template`:

- **`template == 'product'`** - sets `emit_breadcrumb = true`. Then tries to populate `breadcrumb_collection`:
 - **First preference: `collection`** - if the customer reached the PDP via a collection-scoped URL (e.g. `/collections/sideborde/products/wall-lamp-celia-white`), Shopify exposes the parent collection as `collection`. Use it.
 - **Second preference: `product.collections.first`** - if the customer reached the PDP via a bare URL (`/products/wall-lamp-celia-white`), `collection` is nil. Fall back to the product's first associated collection. (This handles the most common PDP case post-Phase-2 canonical override - bare URLs are the canonical surface.)
 - **Both nil → 2-position** - neither `collection` nor `product.collections.first` is set. Position count stays at 2. Hjem → product.title. (The Phase 5 RESEARCH "konsolbord edge case" - a product manually unlinked from every collection. Honest gap; rare; documented.)
- **`template == 'collection'`** - gated on `collection.handle` being non-blank (i.e. the page is a real collection, not a tag-page). Sets `emit_breadcrumb = true`. Position count = 2 (Hjem → collection.title).
- **`template == 'article'`** - sets `emit_breadcrumb = true`. Position count = 3 (Hjem → blog.title → article.title). extension added 2026-05-08;
- **catch-all (else)** - `emit_breadcrumb` stays false. The `{%- if emit_breadcrumb -%}` gate at emission time produces zero `<script>` output. Defense-in-depth alongside the host CR-02 guard at `product-template.liquid:74`.

**Emission block - the BreadcrumbList JSON-LD:**

When `emit_breadcrumb` is true, the snippet emits a single `<script type="application/ld+json">` with `@context: "https://schema.org"`, `@type: "BreadcrumbList"`, and an `itemListElement` array. Position 1 is unconditional `{"@type": "ListItem", "position": 1, "name": "Hjem", "item": <hjem_url>}`. The remaining positions branch by template:

- **PDP** - if `breadcrumb_collection` is set, Position 2 is the collection ListItem (collection.title + `shop.url + routes.collections_url + '/' + breadcrumb_collection.handle`); Position 3 (terminal) is the product. If `breadcrumb_collection` is nil, only Position 2 emits - the product.
- **Collection** - Position 2 is the collection ListItem (collection.title + `shop.url + routes.collections_url + '/' + collection.handle`).
- **Article** - Position 2 is the blog (blog.title + `shop.url + blog.url`); Position 3 (terminal) is the article (article.title + `shop.url + article.url`).

**Key engineering decisions inside the emission:**

- **HTTPS @context literal** (`"https://schema.org"`) - NOT the legacy `http://schema.org` form that the pre-existing main-collection CollectionPage emission at `theme/sections/main-collection.liquid:93` still has (Phase 5 byte-equal protected - out of scope) per the Phase 6 anomalies summary).
- **Collection-position URL via explicit `routes.collections_url + '/' + handle`** - NOT the simpler `collection.url` filter form. Per Phase 5: when a customer reaches a collection page via a tag URL (e.g. `/collections/sideborde/tag-name`), the `collection.url` Liquid filter resolves to the tag URL, not the parent collection URL. Using the tag URL in the BreadcrumbList Position 2 would create a self-referential loop (the page IS the tag URL, the BL says "you're at this tag URL"). The explicit `routes.collections_url + '/' + handle` form pins the URL to the parent collection bare URL, sidestepping the tag self-reference. Tag-page levels are intentionally NOT emitted as ListItem entries - stays out of scope.
- **PDP terminal URL via `shop.url + product.url` BARE** - NOT the collection-scope filter. Mirrors Phase 2 canonical Branch 1; eliminates the SEO trap where the BreadcrumbList would point at a collection-scoped product URL while the canonical points at the bare URL.
- **Position 1 unconditional `"Hjem"`** - Danish for Home; settings-independent constant. The merchant's locale is `da`; a future locale switch would surface as (project policy - locale work out of scope). The hjem_url has an explicit trailing slash (`shop.url + '/'`) to match Shopify's canonical homepage URL form.
- **Settings-independent emission** - does NOT wrap in `{% if settings.show_breadcrumbs %}`. Search engines benefit from the structured hierarchy even when the visible breadcrumb HTML is hidden by the merchant UI toggle. Per REQ-breadcrumb-schema + AC#5.
- **JSON-encoding via `| strip_html | escape | json`** - `strip_html` removes any HTML in product.title / collection.title / blog.title / article.title; `escape` HTML-escapes special chars; `| json` JSON-encodes the result with proper quoting + escaping. Defense-in-depth against admin-injected markup.

**Pre-edit baseline.**

The pre-Phase-5 `theme/snippets/schema-breadcrumb.liquid` did NOT exist - this is a NEW file. Pre-edit, NO BreadcrumbList JSON-LD was emitted anywhere on the dev theme. The legacy `theme/sections/main-collection.liquid` at lines 91-109 had a CollectionPage emission (a different Schema.org type - CollectionPage, NOT BreadcrumbList) which Phase 5 explicitly did NOT replace, modify, or migrate (project policy - ; the CollectionPage walker has its own pre-existing engineering oddities including HTTP `@context` and a hand-rolled walker pattern; flagged in Phase 6 anomalies summary as -class WARN with explicit "DO NOT propagate" guidance).

**Test verification.**

Phase 5 dev-preview validation captured 13 surfaces - 5 PDPs (mix of bare and collection-scoped URLs), 5 collection pages (mix of regular + paginated + tag-URL), and 3 article pages (extension added 2026-05-08). For each, the rendered BreadcrumbList JSON-LD was extracted and validated:

- **PDP bare** (`/products/wall-lamp-celia-white`) - 3-position via `product.collections.first` fallback (the wall-lamp product is in collection `lamper-loftslamper`); ListItems: Hjem → "Lamper & loftslamper" → "Wall Lamp Celia White"
- **PDP scoped** (`/collections/sideborde/products/wall-lamp-celia-white`) - 3-position via `collection` (Shopify exposes the scope); ListItems: Hjem → "Sideborde" → "Wall Lamp Celia White"
- **Collection** (`/collections/sideborde`) - 2-position; Hjem → "Sideborde"
- **Collection paginated** (`/collections/alle-produkter?page=2`) - 2-position; Hjem → "Alle produkter" (pagination doesn't change the BL hierarchy; Phase 2 canonical handles `?page=2+` separately)
- **Tag URL** (`/collections/sideborde/black`) - 2-position; Hjem → "Sideborde" (parent collection STOP via `routes.collections_url + '/' + handle` form, NOT the simpler `collection.url` filter that would self-reference the tag URL -)
- **Article** (`/blogs/lammeuld-dk-moebel-blog/historien-bag-montessori-mobler`) - 3-position; Hjem → "Lammeuld dk Møbel Blog" → "Historien bag Montessori-møbler"

All 13 BreadcrumbList JSON-LDs submitted to RRT; all 13 returned "Page is eligible for rich results" (BreadcrumbList enhancement). Screenshots at `qa/phase-5/rrt-*.png` signed off in commit `2a11253`.

**Phase 5 atomic commit authoritative for this file:**
- `617987f` `feat(05-breadcrumblist-schema): add BreadcrumbList JSON-LD via schema-breadcrumb.liquid + 3 host wires (PDP/Collection/Article)`

**Phase 6 14-AC verdict citation:** AC#5 GREEN - 13 RRT clean PNGs at `qa/phase-5/rrt-*.png` (sign-off SHA `2a11253`).

**RRT evidence (13 PNGs total - 5 PDP + 5 Collection + 3 Article):**

PDP rows (5):
- `qa/phase-5/rrt-pdp-1-smalt-konsolbord-med-maskehylde-industrielt-design.png`
- `qa/phase-5/rrt-pdp-2-wall-lamp-celia-white-bare.png`
- `qa/phase-5/rrt-pdp-3-wall-lamp-celia-white-scoped.png`
- `qa/phase-5/rrt-pdp-4-double-bed-tian-dpo-3.png`
- `qa/phase-5/rrt-pdp-5-knageraekke-bambus.png`

Collection rows (5):
- `qa/phase-5/rrt-coll-1-sideborde.png`
- `qa/phase-5/rrt-coll-2-alle-produkter.png`
- `qa/phase-5/rrt-coll-3-alle-produkter-page-2.png`
- `qa/phase-5/rrt-coll-4-tv-borde-sort.png`
- `qa/phase-5/rrt-coll-5-bambus.png`

Article rows (3 - extension added 2026-05-08):
- `qa/phase-5/rrt-art-1-historien-bag-montessori-mobler.png`
- `qa/phase-5/rrt-art-2-haengebord-til-altan-spiseplads-uden-at-fylde-gulvet.png`
- `qa/phase-5/rrt-art-3-8-premium-sofaborde-online-til-en-mere-gennemfort-stue.png`

(Per-PNG sign-off SHA `2a11253`rows 11-23.)

---

### §3.8 - `theme/sections/main-collection.liquid` (Phase 5)

**Narrative.** Phase 5 (commit `617987f`) added a single `{%- render 'schema-breadcrumb' -%}` line at line 110 (post-edit), AFTER the pre-existing legacy CollectionPage `<script type="application/ld+json">` block at lines 91-109. The legacy block stays byte-untouched (Phase 5 byte-equal protected per Phase 5 D-09; bargaining-material WARN class per the Phase 6 anomalies summary - pre-existing CollectionPage walker artifact stays as-is, NOT propagated to the new schema). The previous `{% schema %}` block opening line shifts from line 110 to line 112.

**What changed** (`git diff baseline-pre-phase-2 -- theme/sections/main-collection.liquid`):

```diff
diff --git a/theme/sections/main-collection.liquid b/theme/sections/main-collection.liquid
index 56cf2b7.332cbc9 100644
--- a/theme/sections/main-collection.liquid
+++ b/theme/sections/main-collection.liquid
@@ -108,6 +108,8 @@
 }
 </script>

+{%- render 'schema-breadcrumb' -%}
+
 {% schema %}
 {
 "name": "t:sections.main-collection.name",
```

**Liquid logic explained.**

The diff shows a 2-line insertion (one render call + one blank line for readability) immediately after the closing `</script>` of the legacy CollectionPage block. The position is deliberate: by landing AFTER the legacy block, Phase 5 leaves lines 91-109 byte-equal vs `baseline-pre-phase-2` - the legacy CollectionPage walker artifact (which has its own engineering oddities including the HTTP `@context` defect at line 93 and a hand-rolled walker pattern) is preserved as-is, NOT replaced, NOT modified, NOT migrated. project policy (no scope creep) keeps it out of scope; it's for a future change-order - the Phase 6 anomalies summary at flags it as a -class WARN with explicit "DO NOT propagate" guidance.

**The render call - `{%- render 'schema-breadcrumb' -%}`:**

No parameters; the snippet reads `template`, `collection`, `routes`, `shop` from the caller's render context (Liquid render-scope isolation excludes assigned variables but includes the global page objects). Inside the snippet (per §3.7), the `template == 'collection'` branch fires; `collection.handle` is verified non-blank; `emit_breadcrumb` is set true; position_count = 2; emission produces a 2-position BreadcrumbList: Hjem → collection.title.

**Why this host wire and not a different host:**

`theme/sections/main-collection.liquid` is the canonical host section for collection pages on the Impulse theme. Every collection page (`templates/collection.liquid`, `templates/collection.alle-produkter.liquid`, etc.) routes through this section. By landing the wire here, Phase 5 covers all collection pages (including the alle-produkter canonical destination from Phase 2 Branch 2, the paginated forms from Phase 2 Branch 3, and any future merchant-authored collection variant) with one render call.

The render call uses `{%-. -%}` whitespace stripping on both sides - consistent with Impulse's surrounding style at this region of the file. After the Phase 2 whitespace-collision lesson, Phase 5 verified the rendered HTML eyeball - the legacy `</script>` tag at line 109 is followed by a newline + the new render call + another newline + the `{% schema %}` block at line 112. No collision risk because the surrounding tags are NOT dash-stripped in a way that would collapse adjacent content.

**The `{% schema %}` line shift.**

Before Phase 5, the section's `{% schema %}` block opened at line 110. After Phase 5, it opens at line 112 - the 2-line insertion shifts everything below it. The `{% schema %}` block content itself is unchanged by Phase 5 (out of scope - it's the Liquid section configuration JSON; not Schema.org; not BreadcrumbList; not in the buyer transcript). The shift is mechanical, not semantic.

**Pre-edit baseline.**

The pre-Phase-5 `theme/sections/main-collection.liquid` lines 91-109 emitted a CollectionPage `<script type="application/ld+json">` block (a different Schema.org type than BreadcrumbList; legacy walker with HTTP `@context` + hand-rolled emission pattern). The `{% schema %}` Liquid section configuration block opened at line 110. Phase 5 inserted 2 lines (one render call + one blank line) immediately after the legacy CollectionPage block's closing `</script>` at line 109; the legacy block content stays byte-equal vs `baseline-pre-phase-2`. The `{% schema %}` block now opens at line 112 (mechanical 2-line shift, unchanged content).

**Test verification.**

Phase 5 dev-preview validation captured collection-page HTML for 5 collections (`sideborde`, `alle-produkter`, `alle-produkter?page=2`, `tv-borde-sort` (tag URL), `bambus`). For each, the rendered HTML was inspected for BOTH a CollectionPage JSON-LD (legacy block; preserved at line 91-109; should be present + byte-equal) AND a BreadcrumbList JSON-LD (Phase 5 emission; new at line 110-rendered; should be present). All 5 collection captures returned both blocks. The legacy CollectionPage block's HTTP `@context` and walker structure are unchanged - Phase 5 byte-equal protected. The new BreadcrumbList block emits HTTPS `@context` and the flat ListItem shape (per §3.7). RRT validation of just the BreadcrumbList block returned clean across all 5 collection captures; screenshots at `qa/phase-5/rrt-coll-*.png` signed off in commit `2a11253`.

**Phase 5 atomic commit authoritative for this file:**
- `617987f` `feat(05-breadcrumblist-schema): add BreadcrumbList JSON-LD via schema-breadcrumb.liquid + 3 host wires (PDP/Collection/Article)` (same atomic commit as §3.7 - the Phase 5 D-06 atomic shipped all 3 host wires + the new snippet in one commit)

**Phase 6 14-AC verdict citation:** AC#5 GREEN - Layer A `coll.html` shows BreadcrumbList=1 per `cross-template-snapshot.csv`; the 5 collection RRT PNGs at `qa/phase-5/rrt-coll-*.png` validate clean.

**RRT evidence:** Shared with §3.7 - the 5 collection BL RRT PNGs at `qa/phase-5/rrt-coll-{1,2,3,4,5}-*.png` (sign-off SHA `2a11253`rows 16-20).

---

## §4 - Decision rationale

This section documents the **why** behind every locked decision in milestone v1.0. Each subsection ties back to a contract clause (the buyer transcript), an engineering safety gate (project policy hard rule or Phase internal planning decision), or a buyer-authorized expansion (Phase amendment_log).

### §4.1 - Canonical destinations chosen

Per Tina's spec (the buyer transcript lines 18-25, 2026-05-05 1:40 PM): **canonical-only fixes**, no redirects unless absolutely necessary. The 4-branch override at `theme/layout/theme.liquid` (Phase 2 one-fix-per-commit commit `fd58602`) computes the canonical destination by template type:

| Branch | Predicate | Canonical destination | Rationale |
|---|---|---|---|
| 1 - Product | `template == 'product'` | `shop.url + product.url` (bare `/products/{handle}`) | Strips `/collections/X/` scope; collapses duplicate canonicals. PDPs reachable via multiple collection scopes (e.g. `/collections/sideborde/products/X`, `/collections/all/products/X`) all point at the bare product URL - single canonical destination per product. |
| 2 - All collection | `template == 'collection' and collection.handle == 'all'` | `shop.url + '/collections/alle-produkter'` | The English-named "all" collection canonicals to the Danish-named "alle-produkter" - canonical, **not** 301 (per Tina's "no redirects unless absolutely necessary" rule). |
| 3 - Paginated collection | `template contains 'collection' and current_page > 1 and current_tags == blank` | `shop.url + collection.url` (bare collection URL = page 1) | See §4.2 - pagination deviation paper trail. |
| 4 - Default | else | `canonical_url` (Shopify-generated) | Fallback; covers PDPs without scope override, blog post URLs, blog index URLs, search URLs, 404 URLs, password layouts, gift-card layouts, etc. |

Plus `theme/snippets/social-meta-tags.liquid:3` (Phase 2 one-fix-per-commit commit `e120a4b`): `og:url` mirrors `computed_canonical` via load-bearing fallback `assign og_url = computed_canonical | default: canonical_url`. This resolves Tina's "self-canonical or conflicting signals" phrase (the buyer transcript line 23) - without the og:url mirror, og:url would silently fall through to `canonical_url` (Shopify default) on the very URLs where Branch 2 + Branch 3 set a different `computed_canonical`. The render-tag scope-isolation pattern (post-2019 Liquid spec; sourced from Context7 fetch of `/shopify/theme-liquid-docs` 2026-05-07) requires the explicit parameter pass at `theme.liquid:47` (`{%- render 'social-meta-tags', computed_canonical: computed_canonical -%}`) for the snippet to read the override.

The pagination automated check uses **global** `current_page` (NOT `paginate.current_page`). Sourced from shopify.dev/docs/storefronts/themes/seo/metadata: the `paginate` object is a section-scope object created by `{% paginate %}` in `templates/collection.liquid` and is NOT available at `theme.liquid` head-scope - `current_page` is the global Liquid variable for that, available everywhere.

The `current_tags == blank` guard on Branch 3 is the CR-01/CR-02 fix bundle (Phase 2 one-fix-per-commit commit `9c07029`). Without the guard, tag pages with pagination (e.g. `/collections/tv-borde/sort?page=2`) would canonical to the bare collection URL - breaking the tag-page noindex remediation path that stays (- bargaining material for future work, see GROWTH-AUDIT.md).

The whitespace-collision fix (Phase 2 one-fix-per-commit commit `843e553`) restored a missing trailing dash on `theme.liquid:39` after the canonical block - without the fix the rendered HTML head collided two adjacent tags, breaking source-view inspection but rendering identically in the DOM (regression caught on Plan 02-05 buyer eyeball; sourced from MEMORY `feedback_liquid_whitespace_collision.md`).

**Tag-page branch** was **explicitly rejected** 2026-05-06 per Phase 1 Decision (a). The half-measure considered: canonical tag URLs to their parent collection (e.g. `/collections/tv-borde/sort` → `/collections/tv-borde`). Rejected because GSC traffic data showed top tag URLs (e.g. `/alle-produkter/skoskabe`) accumulate ~265 clicks per 90 days; canonicaling them to a broad parent that doesn't target the tag's keyword would put those clicks at risk with no offsetting ranking lift. Tag-page noindex via `<meta name="robots" content="noindex">` gated on `current_tags != blank` is the proper remediation, but stays out of scope .

### §4.2 - Pagination deviation paper trail

**Pagination `?page=2+` canonicalizes to page 1.** This deviates from Google's published pagination guidance, but is the buyer's explicit specification - implemented as specified, documented here, **do not "fix" by switching to self-referencing canonicals**.

**Source citation:** [Pagination and incremental page loading](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading) (Google Search Central; last updated 2025-12-10). Verbatim from Google: *"Don't use the first page of a paginated sequence as the canonical page. Instead, give each page its own canonical URL."*

**Timeline (verbatim from the buyer transcript):**

- **2026-05-05 1:40 PM** (Tina, relayed by Jonathan, the buyer transcript lines 3-49): scope received including the pagination spec ("Ensure pagination (?page=2+) canonicalizes to page 1").
- **2026-05-05 1:57 PM** (Vadim, the buyer transcript line 62): reply flagged the deviation in writing - *"Pagination ?page=2+ canonical to page 1, per your spec. Flagging in writing once more: this deviates from Google's current published pagination guidance (Dec 2025 update). Implementing as you specified, documenting the deviation and rationale in the handover so the next person who audits the site has the trail."*
- **2026-05-06**: Upwork milestone offer accepted with the deviation explicit (the Upwork offer page repeats the scope items including the pagination clause; acceptance closes the contract).

Buyer specification is intentional, repeated, and accepted on contract. The deviation flag was communicated in writing **before** the contract was accepted - the buyer had every opportunity to read the deviation flag and decline the contract; they accepted with the flag explicit. Implemented as specified.

**Implementation:** `theme/layout/theme.liquid` Branch 3 (lines 30-32 post-canonical-block insert):

```liquid
{%- elsif template contains 'collection' and current_page > 1 and current_tags == blank -%}
 {%- assign computed_canonical = shop.url | append: collection.url -%}
```

**Why this is here in §4 not just CHANGELOG:** the buyer asked specifically for "decision rationale documented (canonical destinations chosen, pagination deviation flagged with date and source)" (the buyer transcript line 83). This subsection IS the deliverable.

**Future-proofing note:** if a future audit (next dev or Tina herself) wonders why `?page=2+` doesn't self-canonical, they can read this paragraph and the surrounding paper trail (both timestamps + the source URL + verbatim Google guidance) and decide if Tina's spec needs revisiting - this is a contract conversation, not an engineering bug.

## §5 - Per-change rollback

By-file × by-commit. Replicates the Rollback field from each entry (13 entries; chronologically ordered post-Wave-0 reorder per changelog format). Each row: file → commit SHA → exact `git revert` command → effect. The rollback semantics are simple - **`git revert` is the safe undo** (creates a NEW commit reversing the change); never `git commit --amend`.

### Rollback table

| # | Date | File | Phase | Commit SHA | `git revert` command | Effect |
|---|---|---|---|---|---|---|
| 1 | 2026-05-07 | theme/layout/theme.liquid | Phase 2 | fd58602 | `git revert fd58602` | Reverts 4-branch canonical block insertion. theme.liquid back to single-line `<link rel="canonical" href="{{ canonical_url }}">` Impulse default. og:url falls through to `canonical_url` since the parameter pass at line 47 is also reverted by the same commit. |
| 2 | 2026-05-07 | theme/snippets/social-meta-tags.liquid | Phase 2 | e120a4b | `git revert e120a4b` | Reverts og:url load-bearing-fallback mirror. og:url back to `assign og_url = canonical_url` Impulse default. Safe to revert alone; the canonical override at theme.liquid still fires (just og:url no longer mirrors it). |
| 3 | 2026-05-07 | (documentation revision; theme files UNCHANGED) | Phase 2 | (D-01 revision commit) | (no theme/ revert needed) | Reverts D-01 validation surface revision documentation in internal planning / canonical-snapshot.csv / regression-results.txt. Theme files stay correct as committed at `fd58602` + `e120a4b`. This is a no-op rollback for theme/ - only ` audit trail changes. |
| 4 | 2026-05-07 | theme/layout/theme.liquid | Phase 2 | 843e553 | `git revert 843e553` | Reverts whitespace-collision fix. Restores the collision-broken HTML head where the canonical block's trailing tag collides with the next adjacent tag in source view (DOM renders identically; only View Source / curl source view shows the collision). To fully revert Phase 2 canonical work, also revert #1 above (in reverse order: revert 843e553 first, then revert 9c07029, then revert e120a4b, then revert fd58602). |
| 5 | 2026-05-07 | theme/layout/theme.liquid | Phase 2 | 9c07029 | `git revert 9c07029` | Reverts CR-01 + CR-02 check-tightening. Branch 3 automated check becomes looser (drops the `current_tags == blank` guard); tag pages with pagination would canonical to bare collection URL - regression. Only revert if you also intend to revert Phase 5 (since Phase 5's tag-page-aware logic depends on this guard). |
| 6 | 2026-05-07 | theme/snippets/schema-product.liquid + theme/snippets/product-template-variables.liquid | Phase 3 | 45225c4 | `git revert 45225c4` | Reverts Product schema REPLACE. Restores `theme/snippets/product-template-variables.liquid:16-57` (the legacy 18-key emission) + removes `theme/snippets/schema-product.liquid` (the new emission). Atomic D-06: snippet REPLACE + new file CREATE in single commit; revert undoes both atomically. |
| 7 | 2026-05-07 | theme/snippets/product-template.liquid | Phase 3 | 45225c4 | `git revert 45225c4` (same commit as #6) | Same one-fix-per-commit commit as #6 - the wire `{%- render 'schema-product' -%}` at line 74 is part of the same commit. Atomic revert undoes everything together. |
| 8 | 2026-05-07 | theme/snippets/schema-product.liquid + theme/snippets/product-template.liquid | Phase 3 | 810e818 | `git revert 810e818` | Reverts CR-01 sku-fallback + CR-02 PDP-only render guard. The render at `product-template.liquid:74` becomes ungated; Featured Product section + Quick Shop modal would emit JSON-LD on non-PDP layouts (regression - duplicate Product schema in DOM). Only revert if you also revert #6 (since #8 is a fix on top of #6). |
| 9 | 2026-05-08 | theme/snippets/schema-article.liquid + theme/sections/article-template.liquid | Phase 4 | 19d56d0 | `git revert 19d56d0` | Reverts Article schema (one-fix-per-commit policy: new snippet CREATE + section EDIT in single commit). Restores the 43-line inline emission at `article-template.liquid:196-238` (with 9 documented defects per Phase 4 04-LEARNINGS.md); removes `theme/snippets/schema-article.liquid`. |
| 10 | 2026-05-08 | theme/snippets/schema-breadcrumb.liquid | Phase 5 | 617987f | `git revert 617987f` | Reverts BreadcrumbList snippet CREATE (one-fix-per-commit policy: this commit also includes 3 host-template wires; revert undoes all 4 atomically). |
| 11 | 2026-05-08 | theme/snippets/product-template.liquid (Phase 5 host wire at line 75) | Phase 5 | 617987f | `git revert 617987f` (same one-fix-per-commit commit as #10) | Revert via #10 - one-fix-per-commit was a single commit. PDP BreadcrumbList wire goes away. |
| 12 | 2026-05-08 | theme/sections/main-collection.liquid (Phase 5 host wire post line 109) | Phase 5 | 617987f | `git revert 617987f` (same one-fix-per-commit commit as #10) | Revert via #10 - one-fix-per-commit was a single commit. Collection BreadcrumbList wire goes away. |
| 13 | 2026-05-08 | theme/sections/article-template.liquid (Phase 5 host wire at line 197 - extension added 2026-05-08) | Phase 5 | 617987f | `git revert 617987f` (same one-fix-per-commit commit as #10) **OR** the conditional manual-edit path via manual edit | Revert via #10 reverts ALL FOUR Phase 5 changes (snippet + 3 wires); use the manual-edit path from manual-edit revert you need to revert ONLY the article-page expansion while preserving PDP + collection BL wires (per the |

### General rollback notes

- **`git revert` is the safe undo** - creates a NEW commit reversing the change; never `git commit --amend` per memory `feedback_no_amend_unless_asked.md`. The forward-only history pattern (every change is a commit; every undo is also a commit) keeps the audit trail intact for buyer review.
- **After a rollback, append a NEW `### YYYY-MM-DD - <title>` entry to the changelog** per changelog format (one entry per file change; format spec in the changelog head). The append-only-resumes rule from changelog format applies - entries land at the chronological end, never re-edit prior entries.
- **To re-sync dev theme after rollback:** `source && safe_theme_push 196009525588`. NEVER raw `shopify theme push`; LIVE_THEME_ID readonly in the wrapper; the wrapper refuses any non-dev id.
- **Live theme `186499793236` is UNCHANGED throughout this milestone.** Rollbacks never touch live (`safe_theme_publish` always refuses; project policy). Production publish is buyer's manual admin-UI action gated on written sign-off.
- **Atomic D-06 revert semantics:** several commits land multiple file changes in a single commit (Phase 3 commit `45225c4` lands 3 files; Phase 5 commit `617987f` lands 4 files). `git revert <SHA>` undoes ALL files in the commit atomically. To revert only a subset, use `git checkout HEAD~N -- path/to/specific/file` followed by a new commit (per the manual-edit revert example).
- **Reverse-order revert chains:** when reverting multiple commits that depend on each other (e.g. fix-on-top-of-feature pattern like commit `810e818` fixing `45225c4`), revert in reverse chronological order - newest first, oldest last. Otherwise the intermediate state may be broken.

### §5.2 - Rollback safety guarantees

The rollback design carries these safety guarantees, derived from the one-fix-per-commit commit pattern + the wrapper-locked dev-only push surface:

1. **Live theme `186499793236` is NEVER touched by any rollback.** All rollbacks re-sync the dev theme via `safe_theme_push 196009525588`; the wrapper refuses live-theme operations. To revert live, Vadim re-publishes the prior live theme via Shopify admin UI (Themes → 186499793236 → Publish - one click, ~10s).
2. **Forward-only history.** Every change is a commit; every undo is a commit. No `git commit --amend`, no `git reset --hard` to wipe history. The git log is the complete audit trail.
3. **Atomic D-06 boundaries map to single `git revert` operations.** No "half-reverts" - a feature change either lands fully or rolls back fully. Subset rollbacks (e.g. revert ONE branch of a 4-branch block) require manual edit + new commit (the §5.1 flowchart guides this decision).
4. **The 4-branch canonical block has a load-bearing fallback.** Branch 4 (`else → canonical_url`) catches any URL not matching Branches 1-3; rolling back Branch 1 / 2 / 3 does NOT leave a URL with no canonical (it falls through to Branch 4).
5. **The og:url load-bearing fallback** (`assign og_url = computed_canonical | default: canonical_url`) keeps og:url sane even if `computed_canonical` is undefined post-revert. This means Plan 02-02 (theme.liquid) and Plan 02-03 (social-meta-tags.liquid) can be reverted independently without an Open Graph spec violation in the intermediate state.
6. **the changelog is the source of rollback intent.** Every rollback writes a NEW CHANGELOG entry documenting what was rolled back + why + at what commit SHA. Future devs can read the changelog and reconstruct the full forward-and-backward history.
7. **`baseline-pre-phase-N` git tags anchor phase-bounded state captures.** `git diff baseline-pre-phase-N -- theme/` shows the net change Phase N introduced. After full Phase N rollback, the diff returns to zero (assuming no later phase modified the same lines).

### Per-file rollback narrative (downstream-impact map for the next dev)

The 13-row table above is the mechanical reference. The narrative below is the **downstream-impact map** - what breaks where if you revert each file, written for the next dev who is trying to undo a specific defect without unwinding the whole milestone.

**theme/layout/theme.liquid** - The canonical block lives at lines 29-39 (the `{%- liquid -%}` block that computes `computed_canonical`); the consumer at line 39 is `<link rel="canonical" href="{{ computed_canonical }}">`; the parameter pass at line 47 (`{%- render 'social-meta-tags', computed_canonical: computed_canonical -%}`) bridges the render-tag scope-isolation gap. Reverting `fd58602` rolls all three modifications back atomically (single-commit revert; the 4-branch block insertion was a single-file change). Downstream impact: AC#1 + AC#12 flip RED in the 14-AC scoreboard; cross-template-snapshot.csv would show 0/5 PASS on canonical-correctness; og:url would silently mismatch on `/collections/all` and `?page=2+` URLs (since the social-meta-tags consumer at line 3 reads `computed_canonical` which would be undefined post-revert; the load-bearing fallback in `e120a4b` keeps og:url sane via `| default: canonical_url`). **If you only want to revert ONE branch** (e.g. drop the pagination deviation per Tina's hypothetical reversal), edit the file by hand - remove only the Branch 3 lines + commit the diff. Don't `git revert fd58602` for a single-branch undo; that would also undo Branches 1 + 2 + 4 atomically.

**theme/snippets/social-meta-tags.liquid** - The single-line change at line 3 (`assign og_url = canonical_url` → `assign og_url = computed_canonical | default: canonical_url`) is the only Phase 2 change to this file. Reverting `e120a4b` is safe in isolation - the canonical link tag at theme.liquid:39 still emits `computed_canonical`; only og:url falls back to `canonical_url` (Shopify default). Downstream impact: AC#12 alone flips RED (og:url no longer mirrors canonical on `/collections/all` and paginated URLs); AC#1 stays GREEN. **The load-bearing fallback** is what makes mid-flight rollbacks safe - even if Plan 02-02 is reverted but Plan 02-03 stays, the fallback emits Shopify's canonical_url cleanly. Without the fallback, og:url would be nil-emitted (Open Graph spec violation; potential Twitter Card / Facebook share preview breakage).

**theme/snippets/schema-product.liquid** - This file is a Phase 3 CREATE (it didn't exist pre-Phase-3). Reverting `45225c4` deletes the file entirely AND restores `theme/snippets/product-template-variables.liquid:16-57` (the legacy 18-key emission) AND removes the wire at `theme/snippets/product-template.liquid:74` - one-fix-per-commit single-commit revert. Downstream impact: AC#3 + AC#10 flip RED; rich-results-test on PDPs would surface the legacy emission's defects (per Phase 3 03-LEARNINGS.md). The fix-bundle commit `810e818` (CR-01 + CR-02) is **a fix on top of `45225c4`**; if you revert `45225c4` alone, `810e818` becomes a phantom revert (the files it patched no longer exist or have changed). To fully unwind Phase 3, revert in reverse order: `git revert 810e818` first, then `git revert 45225c4`.

**theme/snippets/product-template-variables.liquid** - This file is a Phase 3 EDIT (the legacy emission at lines 16-57 was deleted; the rest of the file preserved). Reverting `45225c4` restores the deleted emission. Downstream impact: same as schema-product.liquid (one-fix-per-commit single-commit revert). NOTE: Phase 3 deliberately did NOT delete the file - only the JSON-LD emission inside it. The file still exists post-Phase-3 with non-schema variables intact (per Phase 3 03-internal planning "" decision).

**theme/snippets/product-template.liquid** - This file gets edited in TWO commits: `45225c4` (Phase 3 - wire the schema-product render at line 74) and `617987f` (Phase 5 - wire the schema-breadcrumb render at line 75). Reverting either commit undoes ONLY that wire; the other remains intact. The CR-02 PDP-only guard from `810e818` wraps the schema-product wire (line 74); reverting `810e818` ungates the wire (Featured Product + Quick Shop modal regression). To fully unwind Phase 3 + Phase 5 article-page expansion: revert `617987f`, then `810e818`, then `45225c4` (reverse-order chain).

**theme/snippets/schema-article.liquid** - Phase 4 CREATE (didn't exist pre-Phase-4). Reverting `19d56d0` deletes the file AND restores the 43-line inline emission at `article-template.liquid:196-238`. The 43-line legacy emission has 9 documented defects per Phase 4 04-LEARNINGS.md (e.g. malformed `image` field, missing `author.@type`, hardcoded `dateModified` referencing wrong template variable). Downstream impact: AC#4 flips RED; rich-results-test on blog posts surfaces the 9 defects. **Independent from Phase 5** - Phase 4 revert does NOT undo Phase 5 article-page BL wire (that's `617987f`).

**theme/sections/article-template.liquid** - This file gets edited in TWO commits: `19d56d0` (Phase 4 - replace inline emission with `{%- render 'schema-article' -%}` at line 196) and `617987f` (Phase 5 - add `{%- render 'schema-breadcrumb' -%}` at line 197 extension added 2026-05-08). Reverting `19d56d0` restores the legacy 43-line emission; reverting `617987f` removes only the BL wire at line 197. The conditional rollback path via manual edit (manual edit + new commit instead of `git revert 617987f`) is the way to revert ONLY the article-page BL wire while preserving the PDP + collection BL wires from the same one-fix-per-commit commit.

**theme/snippets/schema-breadcrumb.liquid** - Phase 5 CREATE (didn't exist pre-Phase-5). Reverting `617987f` deletes the file AND removes 3 host-template wires AND undoes the PDP + Collection + Article BL emission entirely. **No partial revert via `git revert`** - one-fix-per-commit was a single commit; partial reverts require manual editing. The conditional rollback path from manual-edit revert through the article-only revert sequence.

**theme/sections/main-collection.liquid** - Phase 5 EDIT (`617987f`). Reverting that commit removes the schema-breadcrumb render insert post-line-109. Independent from PDP + Article BL wires only at the manual-edit level (one-fix-per-commit again - `git revert` undoes all three wires together). Downstream impact: AC#5 partially flips on collection pages (5/13 RRT screenshots become irrelevant; 8/13 stay GREEN if you also undo the PDP wire).

### Worked example: full milestone v1.0 rollback (nuclear option)

If the buyer requests "undo everything" (extremely unlikely; included for completeness):

```bash
# Reverse-order revert chain (Phase 5 → Phase 4 → Phase 3 → Phase 2)
git revert 617987f # Phase 5 BreadcrumbList one-fix-per-commit (4 files)
git revert 19d56d0 # Phase 4 Article schema one-fix-per-commit (2 files)
git revert 810e818 # Phase 3 Product schema CR-01 + CR-02 fix
git revert 45225c4 # Phase 3 Product schema one-fix-per-commit (3 files)
git revert 9c07029 # Phase 2 CR-01 + CR-02 automated check fix
git revert 843e553 # Phase 2 whitespace-collision fix
git revert e120a4b # Phase 2 social-meta-tags og:url mirror
git revert fd58602 # Phase 2 4-branch canonical block

safe_theme_push 196009525588
```

Result: theme/ byte-equal to `baseline-pre-phase-2` (Impulse 7.5.1 stock + prior Vasyl customizations). The git history preserves the full forward chain + 8 revert commits - full audit trail intact. Live theme `186499793236` UNCHANGED.

