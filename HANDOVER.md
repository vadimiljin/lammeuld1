# HANDOVER - lammeuld.dk technical SEO v1.0

<details>
<summary><b>Build context</b> (date, author, themes)</summary>

- **Date:** 2026-05-09
- **Author:** Vadim Iljin (hello@vadimiljin.com)
- **Dev theme:** 196009525588 (work happens here)
- **Live theme:** 186499793236 (untouched throughout the milestone)

</details>

What's in this repo:

- `HANDOVER.md` - this file. Liquid changes, diffs, decision rationale, rollback.
- `preview-links.md` - 5 cookied dev preview URLs, one per template type.
- `CRAWL-RESULTS.md` - the 500-URL crawl validation report.
- `qa/phase-3/`, `qa/phase-4/`, `qa/phase-5/` - Rich Results Test screenshots.
- `GROWTH-AUDIT.md` - prioritized backlog for future engagements. Not part of v1.0.

The theme/ folder was sent separately as a zip from dev theme `196009525588`. Pull from Shopify admin if you want a fresher snapshot.

---

## Verification baseline (before any edit)

Before touching a single Liquid file I duplicated the live Impulse 7.5.1 theme to dev theme `196009525588` as an unpublished copy and worked exclusively against that. The live theme was never the push target.

The pre-change baseline:

- **20-URL canonical snapshot** (5 product / 5 collection / 5 paginated / 5 blog) to anchor before/after diffs.
- **GSC indexation snapshot:** 17,854 indexed / 71,424 not indexed; 5,707 product snippets, 6 FAQ snippets, 2 review snippets currently emitted.
- **Schema source verified theme-shipped.** I grepped the rendered HTML on PDPs, collection pages, and blog posts and traced every JSON-LD block back to theme Liquid - the 5,707 product snippets all came from the legacy emission at `snippets/product-template-variables.liquid:16-57`, and the article block from `sections/article-template.liquid:196-238`. No installed app (JSON-LD for SEO, Schema App, Smart SEO, etc.) emits competing markup. That's what makes theme-Liquid the right implementation surface: no apps are in the way. The 6 FAQ + 2 review snippets are out of v1.0 scope and stay byte-untouched.

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

Branch 1 emits the bare PDP URL regardless of how the customer got there - `/collections/sideborde/products/wall-lamp-celia-white` and `/products/wall-lamp-celia-white` both canonical to the bare form. Branch 2 points the English-named "all" collection at the Danish-named "alle-produkter" route. Canonical, not 301. Branch 3 collapses pagination to page 1; the `current_tags == blank` guard keeps tag-page paginated URLs (`/collections/tv-borde/sort?page=2`) out of Branch 3 - they fall through to Branch 4 and self-canonical via Shopify's default. Non-paginated tag pages (`/collections/sideborde/black`) also self-canonical via Branch 4. That's intentional - see the Decision rationale below for the GSC traffic data behind that call. Branch 4 covers everything else (homepage, blog, search, page templates, account, etc.).

The `current_page` variable used in Branch 3 is the global one, not `paginate.current_page` (which is section-scope and not visible at head-scope). The render-tag parameter pass at line 58 is required by Liquid's render-scope isolation rule (Shopify Liquid spec: `render` creates an isolated scope; the explicit-parameter form `{% render 'name', key: value %}` is required to surface caller variables). Without it, `social-meta-tags` would compute its own `canonical_url` independently and `og:url` would silently drift from `<link rel="canonical">`. One regression caught and fixed during Phase 2: the original `{%- liquid -%}` block had whitespace stripping on both sides; the closing dash collided with the next adjacent tag in source view (rendered fine in the DOM but failed the View Source eyeball check). Fixed by dropping the trailing dash, commit `843e553`.

### 2. og:url mirror (`snippets/social-meta-tags.liquid`)

```diff
- assign og_url = canonical_url
+ assign og_url = computed_canonical | default: canonical_url
```

`og:url` now mirrors `<link rel="canonical">` byte-for-byte across all four branches. The `default:` fallback keeps `og:url` valid if a future edit ever drops the parameter pass at the call site - it falls back to Shopify default rather than emitting empty.

### 3. Product schema (`snippets/schema-product.liquid` + `snippets/product-template.liquid`)

Replaces the legacy emission previously inlined in `product-template-variables.liquid:16-57` (the source of the 5,707 Product snippets indexed at baseline). New file:

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
  {%- if product.images.size > 0 -%}
  "image": [
    {%- for img in product.images -%}
      {{ img | img_url: 'master' | prepend: 'https:' | json }}{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
  ],
  {%- endif -%}
  {%- if product.vendor != blank -%}
  "brand": { "@type": "Brand", "name": {{ product.vendor | json }} },
  {%- endif -%}
  {%- if barcode != blank and gtin_key != '' -%}
  "{{ gtin_key }}": {{ barcode | json }},
  {%- endif -%}
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

The `"{{ gtin_key }}": {{ barcode | json }}` line is what makes barcode/gtin actually land in the JSON output. On a product whose variant has a 13-digit barcode (e.g. EAN), it renders as `"gtin13": "5901234123457"`. On a 12-digit barcode (UPC-A), `"gtin12": ...`. Anything else falls through to `"mpn": ...` so the value still has a schema-valid home. If the variant has no barcode at all, the `barcode != blank and gtin_key != ''` two-condition guard omits the field entirely rather than emitting an empty value.

Wired into `product-template.liquid` behind a PDP-only guard:

```diff
- {%- render 'product-template-variables', product: product, current_variant: current_variant -%}
+ {%- if template == 'product' -%}
+   {%- render 'schema-product', product: product -%}
+   {%- render 'schema-breadcrumb' -%}
+ {%- endif -%}
```

The legacy emission at `product-template-variables.liquid:16-57` had multiple defects worth understanding so they don't get reintroduced: HTTP `@context` (Google soft-prefers HTTPS for structured data), no SKU fallback chain (variants without their own SKU emitted `"sku": null`), no `priceValidUntil` on the Offer (Rich Results ineligibility), no GTIN/MPN dispatch (the barcode field was unused), brand emitted as a bare string instead of a structured `Brand` object, image URLs not coerced to absolute HTTPS, no `itemCondition`. The new snippet fixes all of those.

The `template == 'product'` guard does real work. `product-template.liquid` is also pulled in by Featured Product on the homepage and the Quick Shop AJAX modal on collection pages. Without the guard the homepage emits a Product JSON-LD claiming the homepage IS a product, and Quick Shop emits a duplicate Product block inside the collection DOM. SKU falls back from `current_variant.sku` to `product.variants.first.sku` so partially-configured variants don't emit `null`. Barcode dispatch routes by length (8/12/13/14 to the matching `gtin{N}` key, anything else to `mpn`); the two-condition guard `barcode != blank and gtin_key != ''` omits the field entirely when the variant has no barcode, instead of emitting an empty value. `priceCurrency` reads from `cart.currency.iso_code` at runtime, so Shopify Markets multi-currency just works without a Liquid edit. `priceValidUntil` is `now + 31536000s` (one year) formatted YYYY-MM-DD. The `Offer` is single (one Offer per Product, current variant); per-variant Offers (price ranges, region-specific availability) are a future change if needed. `aggregateRating` is intentionally omitted: Trustpilot loads via JS, the review values aren't available at Liquid render time, and if I emitted fake numbers Google can issue a manual penalty for misleading structured data.

### 4. Article schema (`snippets/schema-article.liquid` + `sections/article-template.liquid`)

Replaces the inline 43-line block at `article-template.liquid:196-238`. The legacy block had nine documented defects: HTTP `@context`, no headline truncate (long titles failed Rich Results validation at the 110-char soft limit), `dateModified` could be future or earlier than `datePublished`, no author Person/Organization fallback (blank `article.author` emitted `"name": ""`), no publisher logo `ImageObject` (Article ineligibility), no `description`-when-blank guard (emitted `""`), unpredictable image URL dimensions tied to per-article width, no `mainEntityOfPage` `@id`, dates with literal `Z` suffix on Copenhagen-time articles (silent timezone misrepresentation).

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

New file:

```liquid
{%- liquid
  assign date_published = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
  if article.updated_at != blank
    assign date_modified = article.updated_at | date: '%Y-%m-%dT%H:%M:%S%z'
  else
    assign date_modified = article.published_at | date: '%Y-%m-%dT%H:%M:%S%z'
  endif
  assign date_created = article.created_at | date: '%Y-%m-%dT%H:%M:%S%z'
  if shop.brand.logo.image
    assign publisher_logo_url = shop.brand.logo.image | image_url: width: 600 | prepend: 'https:'
  else
    assign publisher_logo_url = 'https://lammeuld.dk/cdn/shop/files/Logo1.jpg?v=1625038236'
  endif
-%}

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": {{ article.title | escape | truncate: 110 | json }},
  {%- if article.excerpt != blank -%}
  "description": {{ article.excerpt | strip_html | escape | json }},
  {%- endif -%}
  {%- if article.image -%}
  "image": [{{ article.image | img_url: 'master' | prepend: 'https:' | json }}],
  {%- endif -%}
  "datePublished": {{ date_published | json }},
  "dateModified": {{ date_modified | json }},
  "dateCreated": {{ date_created | json }},
  "articleBody": {{ article.content | strip_html | json }},
  {%- if article.author != blank -%}
  "author": { "@type": "Person", "name": {{ article.author | json }} },
  {%- else -%}
  "author": { "@type": "Organization", "name": {{ shop.name | json }} },
  {%- endif -%}
  "publisher": {
    "@type": "Organization",
    "name": {{ shop.name | json }},
    "logo": {
      "@type": "ImageObject",
      "url": {{ publisher_logo_url | json }}
    }
  },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": {{ shop.url | append: article.url | json }}
  }
}
</script>
```

`%z` is the ISO 8601 timezone offset (`+0200` for Copenhagen DST), required by Google to interpret article freshness in the right zone. The legacy block emitted `Z` (UTC literal) on every article, silently misrepresenting the publish time for any article published outside UTC. `dateModified` falls back to `published_at` when `updated_at` is blank, so it can never be empty (`""` is a Rich Results hard-fail) and never earlier than `datePublished` (the "modified before published" inversion is also a hard-fail). The snippet emits all three article timestamps - `datePublished`, `dateModified`, `dateCreated` - the legacy block emitted them too but with the wrong format. Headline is truncated at 110 chars (Google soft-limit). `description` is omitted when `article.excerpt` is blank rather than emitted as `""`. Author has symmetric Person/Organization branches: if `article.author` is set, emit `{"@type": "Person", "name": <author>}`; else emit `{"@type": "Organization", "name": <shop.name>}`. Publisher logo prefers `shop.brand.logo.image` (Shopify's modern Brand asset surface) resized to 600px wide; falls back to the explicit hardcode `https://lammeuld.dk/cdn/shop/files/Logo1.jpg?v=1625038236` so emission never depends on the merchant having filled in the Brand setting. The fallback URL is the same logo asset already on disk, not a fabricated path.

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
- `sections/article-template.liquid` - one render call after the Article render.

This is the single source of truth for BreadcrumbList - no other BreadcrumbList emission exists anywhere on the catalog. Position 2 collection URL is built as `shop.url + routes.collections_url + '/' + handle`, not via the `collection.url` Liquid filter. On tag URLs like `/collections/sideborde/black`, `collection.url` resolves to the tag URL itself, which would create a self-referential breadcrumb (the page IS the URL the breadcrumb points at). The explicit form pins to the bare collection URL. PDP terminal uses `shop.url + product.url` bare for the same reason as Branch 1 of the canonical block - keeps the schema URL aligned with the canonical, no collection-scope leakage. The PDP branch has a 3-position fallback chain: `collection` (the scope Shopify exposes when the customer reached the PDP via a collection-scoped URL), then `product.collections.first` (when the customer reached via the bare URL), then 2-position when both are nil (a product manually unlinked from every collection - rare). Emission is settings-independent: not gated on the merchant breadcrumb-visibility toggle, because search engines benefit from the structured hierarchy even when the visible HTML breadcrumb is hidden. The catch-all branch emits no `<script>` at all - defense-in-depth alongside the host-side PDP guard.

The legacy `CollectionPage` JSON-LD block at `main-collection.liquid:91-109` is a different schema type (CollectionPage, not BreadcrumbList) and is preserved byte-equal. It has its own pre-existing oddities including HTTP `@context` and a hand-rolled walker pattern; the new schema deliberately does NOT propagate the HTTP form. If you ever rewrite that legacy block, fix the HTTP-to-HTTPS migration there too. For v1.0 it stays untouched (out of scope).

### Schema emission summary

| Page type | New JSON-LD blocks emitted |
|---|---|
| PDP | Product + BreadcrumbList |
| Blog post | Article + BreadcrumbList |
| Collection | BreadcrumbList (plus the legacy CollectionPage block, preserved unchanged) |
| Paginated collection | BreadcrumbList |
| Blog index, homepage, page, search, account, cart, checkout | None (existing markup unchanged) |

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

Phase-by-phase validation gates closed clean before each next phase started. Phase 2 close: re-ran the canonical check on the same 20-URL pool I captured pre-edit. All 20 returned the expected branch output (Branch 1 for products, Branch 2 for `/collections/all`, Branch 3 for paginated, Branch 4 for everything else) and `og:url` byte-equal to `<link rel="canonical">` on every row. Cross-template regression on the five surfaces Tina specified (PDP, collection, paginated collection, blog post, blog index) returned 5/5 PASS on the dev theme. Phases 3, 4, 5 each closed with their own RRT validation panels (`qa/phase-{3,4,5}/`) before the next phase's atomic commit landed.

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

### What I deliberately did not touch

- **Tag-page indexability.** Tag URLs like `/collections/tv-borde/sort` are out of v1.0 scope and self-canonical via Branch 4. Top tag URLs accumulate ~265 clicks per 90 days in your GSC; canonical-to-parent (the half-measure I considered first) would risk those clicks for no offsetting ranking lift. Proper remediation is `<meta name="robots" content="noindex,follow">` gated on `current_tags != blank`, separate engagement.
- **The legacy CollectionPage block** at `main-collection.liquid:91-109`. Different schema type (CollectionPage, not BreadcrumbList), pre-existing, has HTTP `@context` and a hand-rolled walker that the new schema deliberately does not propagate. ~10-minute fix when you want it; not v1.0.
- **The legacy `product-template-variables.liquid` file itself.** I removed its 57-line Product JSON-LD emission (lines 16-57) because that's the source of the duplicate Product snippets. The rest of the file (non-schema Liquid assignments) stays byte-equal and the file stays on disk. Cleaning up the husk is a future-engagement decision.
- **FAQ and review snippets** observed pre-edit (6 + 2 respectively). Already emitted by the existing theme; out of v1.0 scope; unchanged.
- **Foreign-language slugs, dual-URL crawl waste, NAP consistency, image alt-text, head-term page rewrites.** All visible during this milestone; all flagged in `GROWTH-AUDIT.md` as separate engagements.

---

## Files changed (9)

Line counts are `git diff --numstat baseline-pre-phase-2..HEAD -- theme/`. Total: +274 / -102.

| File | Change | +/- |
|---|---|---|
| `layout/theme.liquid` | 4-branch canonical block + parameter pass to `social-meta-tags`. | +13 / -2 |
| `snippets/social-meta-tags.liquid` | `og_url` reads `computed_canonical \| default: canonical_url`. | +1 / -1 |
| `snippets/schema-product.liquid` | NEW. Product JSON-LD. | +76 |
| `snippets/product-template.liquid` | Replaced legacy `product-template-variables` render with PDP-guarded `schema-product` + `schema-breadcrumb`. | +4 / -1 |
| `snippets/product-template-variables.liquid` | Removed legacy 57-line Product JSON-LD emission (lines 16-57). File preserved; non-schema Liquid in the same file is untouched. | -57 |
| `snippets/schema-article.liquid` | NEW. Article JSON-LD. | +65 |
| `sections/article-template.liquid` | Removed inline Article block at lines 196-238; added `schema-article` + `schema-breadcrumb` renders. | +2 / -41 |
| `snippets/schema-breadcrumb.liquid` | NEW. BreadcrumbList, 4-branch (product / collection / article / no-op). | +111 |
| `sections/main-collection.liquid` | Added `schema-breadcrumb` render after the legacy CollectionPage block. | +2 |

Out-of-scope surfaces are byte-equal to pre-edit: `checkout/`, `customers/`, `cart.liquid`, `assets/`, `locales/`, `config/settings_schema.json`, `orders/`. None touched.
