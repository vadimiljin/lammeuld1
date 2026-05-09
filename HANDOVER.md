# HANDOVER - lammeuld.dk technical SEO v1.0

**Date:** 2026-05-09
**Author:** Vadim Iljin (hello@vadimiljin.com)
**Contract:** $1,980, 26h, accepted 2026-05-06
**Dev theme:** 196009525588 (work happens here)
**Live theme:** 186499793236 (untouched throughout the milestone)

What's in this repo:

- `HANDOVER.md` - this file. Liquid changes, diffs, decision rationale, rollback.
- `preview-links.md` - 5 cookied dev preview URLs, one per template type.
- `CRAWL-RESULTS.md` - the 500-URL crawl validation report.
- `qa/phase-3/`, `qa/phase-4/`, `qa/phase-5/` - Rich Results Test screenshots.
- `GROWTH-AUDIT.md` - prioritized backlog for future engagements. Not part of v1.0.

The theme/ folder was sent separately as a zip from dev theme `196009525588`. Pull from Shopify admin if you want a fresher snapshot.

---

## What ships

### 1. Canonical override (`layout/theme.liquid`)

A single Liquid block at the top of `<head>`, four branches, then the `<link rel="canonical">` tag reads from it:

```diff
- <link rel="canonical" href="{{ canonical_url }}">
+ {%- liquid
+   if template == 'product'
+     assign computed_canonical = shop.url | append: product.url
+   elsif template == 'collection' and collection.handle == 'all'
+     assign computed_canonical = shop.url | append: '/collections/alle-produkter'
+   elsif template == 'collection' and current_page > 1 and current_tags == blank
+     assign computed_canonical = shop.url | append: collection.url
+   else
+     assign computed_canonical = canonical_url
+   endif
+ %}
+ <link rel="canonical" href="{{ computed_canonical }}">
```

And the `social-meta-tags` render call now passes `computed_canonical` through:

```diff
- {%- render 'social-meta-tags' -%}
+ {%- render 'social-meta-tags', computed_canonical: computed_canonical -%}
```

**For future devs.** Branch 1 emits the bare PDP URL regardless of how the customer got there (`/collections/sideborde/products/wall-lamp-celia-white` and `/products/wall-lamp-celia-white` both canonical to the bare form). Branch 2 points the English-named "all" collection at the Danish-named "alle-produkter" route - canonical, not 301. Branch 3 collapses pagination to page 1; the `current_tags == blank` guard keeps tag-page paginated URLs out (they're a separate problem). Branch 4 lets every other template fall through to Shopify's default. The `current_page` variable is the global one, not `paginate.current_page` (the latter is section-scope and not visible at head-scope).

### 2. og:url mirror (`snippets/social-meta-tags.liquid`)

```diff
- assign og_url = canonical_url
+ assign og_url = computed_canonical | default: canonical_url
```

`og:url` now mirrors `<link rel="canonical">` byte-for-byte across all four branches. The `default:` fallback keeps `og:url` valid if a future edit ever drops the parameter pass at the call site - it falls back to Shopify default rather than emitting empty. This is the resolution for "self-canonical or conflicting signals" in the spec.

### 3. Product schema (`snippets/schema-product.liquid` + `snippets/product-template.liquid`)

Replaces the legacy emission previously inlined in `product-template-variables.liquid` (lines 16-57; this is the source of the 5,707 product snippets your GSC counted pre-edit). New file:

```liquid
{%- liquid
  assign current_variant = product.selected_or_first_available_variant
  assign sku_value = current_variant.sku
  if sku_value == blank
    assign sku_value = product.variants.first.sku
  endif
  assign price_valid_until = 'now' | date: '%s' | plus: 31536000 | date: '%Y-%m-%d'
  assign barcode = current_variant.barcode
  assign gtin_key = ''
  if barcode != blank
    assign barcode_size = barcode | size
    case barcode_size
      when 8;  assign gtin_key = 'gtin8'
      when 12; assign gtin_key = 'gtin12'
      when 13; assign gtin_key = 'gtin13'
      when 14; assign gtin_key = 'gtin14'
      else;    assign gtin_key = 'mpn'
    endcase
  endif
-%}

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": {{ product.title | json }},
  "description": {{ product.description | strip_html | strip_newlines | escape | truncate: 5000 | json }},
  "url": {{ shop.url | append: product.url | json }},
  "sku": {{ sku_value | json }},
  ...
  "offers": {
    "@type": "Offer",
    "price": {{ current_variant.price | divided_by: 100.00 | json }},
    "priceCurrency": {{ cart.currency.iso_code | json }},
    "availability": "https://schema.org/{% if product.available %}InStock{% else %}OutOfStock{% endif %}",
    "priceValidUntil": {{ price_valid_until | json }},
    "url": {{ shop.url | append: product.url | json }},
    "itemCondition": "https://schema.org/NewCondition"
  }
}
</script>
```

Wired into `product-template.liquid` behind a PDP-only guard:

```diff
- {%- render 'product-template-variables', product: product, current_variant: current_variant -%}
+ {%- if template == 'product' -%}
+   {%- render 'schema-product', product: product -%}
+   {%- render 'schema-breadcrumb' -%}
+ {%- endif -%}
```

**For future devs.** The `template == 'product'` guard is load-bearing. `product-template.liquid` is also pulled in by Featured Product on the homepage and the Quick Shop AJAX modal on collection pages. Without the guard, the homepage emits a Product JSON-LD claiming the homepage is a product, and Quick Shop emits a duplicate Product block in the collection DOM. SKU falls back from `current_variant.sku` to `product.variants.first.sku` so unsynced variants don't emit `null`. Barcode length routes to the correct GTIN slot or falls through to `mpn`. `priceCurrency` reads from `cart.currency.iso_code` so Markets multi-currency works without a Liquid edit. `priceValidUntil` is now+1y. `aggregateRating` is intentionally omitted - Trustpilot loads via JS, the values aren't available at render time, and emitting fake values is a manual-action risk.

### 4. Article schema (`snippets/schema-article.liquid` + `sections/article-template.liquid`)

Replaces the inline 43-line block at `article-template.liquid:196-238` (which had HTTP `@context`, no headline truncate, dateModified that could go future, no author fallback, etc.).

```diff
- <script type="application/ld+json">
- {
-   "@context": "http://schema.org",
-   ...
-   "datePublished": {{ article.published_at | date: '%Y-%m-%dT%H:%M:%SZ' | json }},
-   "dateModified": {{ article.updated_at | date: '%Y-%m-%dT%H:%M:%SZ' | json }},
-   "author": { "@type": "Person", "name": {{ article.author | json }} },
-   ...
- }
- </script>
+ {%- render 'schema-article', article: article -%}
+ {%- render 'schema-breadcrumb' -%}
```

New file (excerpt):

```liquid
{%- liquid
  assign date_published = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
  if article.updated_at != blank
    assign date_modified = article.updated_at | date: '%Y-%m-%dT%H:%M:%S%z'
  else
    assign date_modified = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
  endif
-%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": {{ article.title | escape | truncate: 110 | json }},
  ...
  "datePublished": {{ date_published | json }},
  "dateModified": {{ date_modified | json }},
  ...
  {%- if article.author != blank -%}
    "author": { "@type": "Person", "name": {{ article.author | json }} },
  {%- else -%}
    "author": { "@type": "Organization", "name": {{ shop.name | json }} },
  {%- endif -%}
  ...
}
</script>
```

**For future devs.** `%z` is the ISO 8601 timezone offset (`+0200` for Copenhagen DST), required by Google to interpret article freshness in the right zone. `dateModified` falls back to `published_at` when `updated_at` is blank, so it can never be empty or earlier than `datePublished`. Headline is truncated at 110 chars (Google soft-limit). `description` is omitted when `article.excerpt` is blank rather than emitted as `""`. Author falls back to Organization when no Person is set. Publisher logo uses `shop.brand.logo.image` if set, with a hardcoded URL fallback that matches the site's existing logo asset.

### 5. BreadcrumbList schema (`snippets/schema-breadcrumb.liquid` + 3 host wires)

New 4-branch snippet:

```liquid
{%- liquid
  assign emit_breadcrumb = false
  assign hjem_url = shop.url | append: '/'
  assign breadcrumb_collection = nil
  assign position_count = 2

  if template == 'product'
    assign emit_breadcrumb = true
    if collection
      assign breadcrumb_collection = collection
      assign position_count = 3
    elsif product.collections.first
      assign breadcrumb_collection = product.collections.first
      assign position_count = 3
    endif
  elsif template == 'collection'
    if collection.handle
      assign emit_breadcrumb = true
      assign position_count = 2
    endif
  elsif template == 'article'
    assign emit_breadcrumb = true
    assign position_count = 3
  endif
%}
{%- if emit_breadcrumb -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Hjem", "item": {{ hjem_url | json }} }
    {%- if template == 'product' -%}
      {%- if breadcrumb_collection -%}
        ,{ "@type": "ListItem", "position": 2,
           "name": {{ breadcrumb_collection.title | strip_html | escape | json }},
           "item": {{ shop.url | append: routes.collections_url | append: '/' | append: breadcrumb_collection.handle | json }} }
      {%- endif -%}
      ,{ "@type": "ListItem", "position": {{ position_count }},
         "name": {{ product.title | strip_html | escape | json }},
         "item": {{ shop.url | append: product.url | json }} }
    {%- elsif template == 'collection' -%}
      ,{ "@type": "ListItem", "position": 2,
         "name": {{ collection.title | strip_html | escape | json }},
         "item": {{ shop.url | append: routes.collections_url | append: '/' | append: collection.handle | json }} }
    {%- elsif template == 'article' -%}
      ,{ "@type": "ListItem", "position": 2,
         "name": {{ blog.title | strip_html | escape | json }},
         "item": {{ shop.url | append: blog.url | json }} },
      { "@type": "ListItem", "position": 3,
         "name": {{ article.title | strip_html | escape | json }},
         "item": {{ shop.url | append: article.url | json }} }
    {%- endif -%}
  ]
}
</script>
{%- endif -%}
```

Wired in three places:

- `snippets/product-template.liquid` - inside the same PDP-only guard as the Product render (see §3 above).
- `sections/main-collection.liquid` - one render call after the legacy CollectionPage block (which is preserved unchanged):
  ```diff
  + {%- render 'schema-breadcrumb' -%}
  ```
- `sections/article-template.liquid` - one render call after the Article render. This was added 2026-05-08; your spec said "PDPs and collection pages", article-page breadcrumbs were a free addition.

**For future devs.** Position 2 collection URL is built as `shop.url + routes.collections_url + '/' + handle`, not via the `collection.url` filter. On tag URLs (`/collections/sideborde/black`), `collection.url` resolves to the tag URL itself, which would create a self-referential breadcrumb. The explicit form pins to the bare collection URL. PDP terminal uses `shop.url + product.url` bare for the same reason as Branch 1 of the canonical block. Emission is settings-independent (not gated on the merchant breadcrumb-visibility toggle) - search engines benefit even when the visible HTML breadcrumb is hidden. The legacy CollectionPage block at `main-collection.liquid:91-109` is preserved unchanged (different schema type, pre-existing, out of v1.0 scope).

---

## How to verify

Open Shopify admin in one tab so the admin session cookie is set, then click the URLs in `preview-links.md` from the same browser. Without the cookie, `?preview_theme_id=` returns the live theme.

View source on each preview URL:

| URL | Expected canonical | Expected JSON-LD |
|---|---|---|
| PDP `/products/wall-lamp-celia-white` | bare `/products/wall-lamp-celia-white` | Product + BreadcrumbList |
| Collection `/collections/sideborde` | `/collections/sideborde` | BreadcrumbList (plus the legacy CollectionPage block) |
| Paginated `/collections/alle-produkter?page=2` | `/collections/alle-produkter` (no `?page=`) | BreadcrumbList |
| Blog post | Shopify default | Article + BreadcrumbList |
| Blog index | Shopify default | (none) |

`og:url` should byte-equal `<link rel="canonical">` on every page.

Rich Results Test screenshots are in `qa/phase-3/` (5 PDPs, Product), `qa/phase-4/` (5 articles, Article), `qa/phase-5/` (PDP + collection + article BreadcrumbList). All clean.

The 500-URL pre-publish crawl is in `CRAWL-RESULTS.md`. 495/500 strict-PASS, 5 WARN on tag-page surfaces (out of v1.0 scope, classified explicitly), zero failures, zero crawler errors, zero leakage to live.

Pre-change baseline captured before any edit: 20 URLs (5 product, 5 collection, 5 paginated, 5 blog) plus a GSC indexation snapshot (17,854 indexed / 71,424 not indexed; 5,707 product snippets observed pre-replace, all rolled into the new emission; 6 FAQ + 2 review snippets observed and left alone).

---

## Decision rationale

### Canonical destinations chosen

| Branch | Destination | Why |
|---|---|---|
| Product | `shop.url + product.url` (bare) | Collapses every collection-scoped variant of the same PDP to one canonical. |
| `/collections/all` | `/collections/alle-produkter` | The English-named route canonicals to the Danish-named one. Canonical, not 301, per "no redirects unless absolutely necessary". |
| Paginated collection | bare collection URL (page 1) | Pagination consolidation per spec. See pagination deviation note below. |
| Default | `canonical_url` | Shopify default for every other template (homepage, blog, search, page, etc.). |

The `current_tags == blank` guard on Branch 3 keeps tag-page paginated URLs (`/collections/tv-borde/sort?page=2`) from being canonicaled into the parent collection. Tag-page handling is a separate problem and stays out of v1.0; canonical-to-parent-collection on tags would risk ranking on tag-targeted keywords (top tag URLs accumulate ~265 clicks per 90 days in your GSC). Proper remediation is `<meta name="robots" content="noindex,follow">` gated on `current_tags != blank` - separate engagement.

`og:url` mirroring is wired through `computed_canonical | default: canonical_url`. The fallback is defensive: if a future edit drops the parameter pass at the call site, `og:url` falls back to Shopify default rather than emitting blank, which would break Twitter Card / Facebook share previews.

### Pagination deviation - flagged with date and source

You asked for `?page=2+` to canonical to page 1. Google's current public guidance says the opposite.

**Source:** [Pagination and incremental page loading](https://developers.google.com/search/docs/specialty/ecommerce/pagination-and-incremental-page-loading), updated 2025-12-10. Verbatim: *"Don't use the first page of a paginated sequence as the canonical page. Instead, give each page its own canonical URL."*

**Timeline:**

- **2026-05-05 1:40 PM** - Tina's spec (relayed by Jonathan) included "Ensure pagination (?page=2+) canonicalizes to page 1".
- **2026-05-05 1:57 PM** - my reply flagged the deviation in writing before the contract was signed: *"Pagination ?page=2+ canonical to page 1, per your spec. Flagging in writing once more: this deviates from Google's current published pagination guidance (Dec 2025 update). Implementing as you specified, documenting the deviation and rationale in the handover so the next person who audits the site has the trail."*
- **2026-05-06** - Upwork milestone offer accepted with the flag explicit.

Implemented as specified. If a future audit (yours, mine, or someone else's) wonders why `?page=2+` doesn't self-canonical, this paragraph is the answer. Don't "fix" it by switching to self-referencing canonicals - that's a contract change, not a bug.

If you want to switch to self-referencing pagination canonicals later, drop Branch 3 from the `if/elsif` chain in `theme.liquid` and Branch 4 takes over. One-line edit.

---

## Files changed (8)

| File | Change |
|---|---|
| `layout/theme.liquid` | 4-branch canonical block + parameter pass to `social-meta-tags`. |
| `snippets/social-meta-tags.liquid` | `og_url` reads `computed_canonical \| default: canonical_url`. |
| `snippets/schema-product.liquid` | NEW. Product JSON-LD. |
| `snippets/product-template.liquid` | Replaced legacy `product-template-variables` render with PDP-guarded `schema-product` + `schema-breadcrumb`. |
| `snippets/schema-article.liquid` | NEW. Article JSON-LD. |
| `sections/article-template.liquid` | Removed inline 43-line Article block; added `schema-article` + `schema-breadcrumb` renders. |
| `snippets/schema-breadcrumb.liquid` | NEW. BreadcrumbList, 4-branch (product / collection / article / no-op). |
| `sections/main-collection.liquid` | Added `schema-breadcrumb` render after the legacy CollectionPage block. |

Out-of-scope surfaces are byte-equal to pre-edit: `checkout/`, `customers/`, `cart.liquid`, `assets/`, `locales/`, `config/settings_schema.json`, `orders/`. None touched.

---

## Rollback

`git revert <SHA>` is the safe undo (creates a new commit). Never `git commit --amend`.

| Reverts | Command (reverse order matters) | Effect |
|---|---|---|
| All canonical work | `git revert 843e553 9c07029 e120a4b fd58602` | Back to single-line Shopify default canonical; `og:url` reads `canonical_url` directly. |
| Product schema | `git revert 810e818 45225c4` | Restores the pre-existing `product-template-variables.liquid` emission. |
| Article schema | `git revert 19d56d0` | Restores the inline 43-line Article block in `article-template.liquid`. |
| BreadcrumbList | `git revert 617987f` | Removes the snippet and all 3 host wires (PDP + collection + article). |

To drop only the article-page BreadcrumbList wire (the 2026-05-08 addition) and keep PDP and collection: edit the wire line out of `article-template.liquid` by hand instead of reverting `617987f`.

After any rollback, push to dev and re-run the preview-URL view-source check. Live theme `186499793236` is never the push target.

---

## Publish + 30-day monitoring

Publish is your call ("Publish window your call"). Path: Shopify admin -> Themes -> select dev theme `196009525588` -> Publish. That flips dev to live; the previous live theme drops to unpublished.

Once live, the contracted 30-day window opens:

- **Day 0** - re-run the 500-URL crawl against live URLs (no `?preview_theme_id=`). Same script, same criteria. Expected: 100% strict-PASS plus the same 5 tag-page WARNs.
- **Weekly (4x)** - GSC checks on "Duplicate, Google chose different canonical" and "Indexed, not submitted in sitemap" reports. Expected: duplicate-canonical signals trend down as the bare-PDP canonical propagates.
- **Day 30** - final report appended to this file: clicks/impressions delta, indexation status, RRT re-validation on a few exemplars, verdict.

---

## Contact

`hello@vadimiljin.com` or Upwork DM. For the prioritized backlog of follow-up work (tag-page handling, additional schema, AI-search optimization), see `GROWTH-AUDIT.md`.
