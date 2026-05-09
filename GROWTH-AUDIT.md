# LAMMEULD-DK Growth Audit - AI-Search Position & Strategic Roadmap

**Generated:** 2026-05-09
**Author:** Vadim Iljin (hello@vadimiljin.com)
**Audience:** Jonathan Kiilerich, Tina

---

## §0 - Read this first

Three buyer queries run live on 2026-05-09 against ChatGPT and Perplexity in incognito. Twelve Danish furniture and lighting competitors recommended: Nordic Nest, AndLight, Lampegiganten, Illums Bolighus, Luxlight, Lampefeber, Lampemesteren, Lysmesteren, Moster D, others. Zero lammeuld citations.

The v1.0 milestone closed the contracted canonical and schema work. What's left is the AI-search positioning layer that decides whether buyers reach lammeuld.dk through ChatGPT, Perplexity, Claude, and Gemini.

I shipped v1.0. I know the theme, the Liquid scope, the GSC property, the supplier-feed dependencies, the Impulse internals. Onboarding tax is zero. Same architecture I shipped at Route4Me to land enterprise-tier ChatGPT placement next to Samsara, Geotab, Locus, FarEye.

---

## §1 - What buyers see today

Three queries, live, 2026-05-09. Web search enabled. Incognito mode. Real Danish buyer phrasing.

**Q1 (Perplexity, DA, "anbefal mig de bedste danske online butikker for skandinaviske væglamper og loftslamper med god kvalitet og hurtig levering"):** Recommended Nordic Nest, Lampegiganten, Lysmesteren, Luxlight, Moster D. 10 sources cited. lammeuld absent.

**Q2 (Perplexity, EN, "best danish online stores for scandinavian furniture and lighting in 2026"):** Recommended AndLight, Illums Bolighus, Nordic Nest. 20 sources cited. lammeuld absent.

**Q3 (ChatGPT with web tools, DA, same as Q1, 49-second web search):** Recommended Lampemesteren, AndLight, Luxlight, Illums Bolighus, Lysmesteren, Lampefeber. ChatGPT pulled Luxlight's delivery SLA verbatim (*"80% af ordrer før kl. 14 sendes samme dag; 95% leveres inden for 4 hverdage"*). lammeuld absent.

**Why competitors win:** Luxlight publishes a structured delivery SLA. Lampegiganten declares catalog scale (*"over 40.000 produkter"*). Nordic Nest publishes brand counts (*"over 250 brands"*). Lampefeber anchors itself geographically (*"showroom i Harlev lidt uden for Aarhus"*). Each exposes specific extractable claims AI engines can quote.

**Why lammeuld doesn't:** Trustpilot rating is JS-loaded, invisible to AI engines. Catalog scale is implicit. Delivery commitment lives in the FAQ, not as a structured SLA. The brand entity is ambiguous; "lammeuld" is also Danish for sheepskin, and no Knowledge Graph anchor disambiguates them.

---

## §2 - The structural diagnosis

1. **Trust capital invisible.** Trustpilot widget is JS; AI engines crawl static HTML. PDP DOM audit 2026-05-09 (live browser, JS rendered, on `/products/beech-bookcase-with-full-side-panels-6-shelves-165x60x25-cm`) confirms the widget loads in the rendered DOM but emits zero structured-data signal: no `AggregateRating` JSON-LD, no `Review` microdata, no rating in any extractable surface. The star rating exists visually for human shoppers and is invisible to every AI engine and Google's Rich Results pipeline.
2. **Brand entity ambiguous.** "lammeuld" = sheepskin. No Knowledge Graph card. `Organization` JSON-LD with `sameAs` to existing identifiers (Trustpilot, social profiles) closes this. No Wikipedia/Wikidata edits promised.
3. **No retrieval surface for AI ingestion.** No `llms.txt`, no per-category retrieval objects. Bolia and Illums already ship richer surfaces.
4. **Brand-entity confusion already in AI training data.** External AI aggregator sites (AskmeOffers, Zoneoffer) classify lammeuld.dk as a *"lambswool products"* / *"wool store"*, confirming the §2.2 disambiguation problem isn't theoretical. The misclassification is in the model corpus already.
5. **Product schema dimensional + material deep-cuts missing.** v1.0 ships the contracted minimum *plus* `sku` *plus* `gtin8`/`gtin12`/`gtin13`/`gtin14`/`mpn` from `barcode` length-routing (snippets/schema-product.liquid:24-39, 49, 63-65). Still missing for AI Overview eligibility on a furniture catalog: `material`, `weight`, dimensional triple `width`/`depth`/`height`, plus `color`, `pattern`, `size`. These map to existing Shopify metafields; theme-side wiring per metafield namespace, no buyer data work needed.
6. **Tag-page indexation tree at multiplicative scale.** Color, material, price-range, and tag-with-parent-name duplicates indexed across `/collections/{cat}/{tag}`, `/collections/all/{tag}`, even `/collections/alle-produkter/{tag}`. Google snippets cannibalize parent meta descriptions. Theme-wide `noindex,follow` gated on `current_tags != blank` is the fix.
7. **Visible HTML breadcrumbs absent.** v1.0 ships JSON-LD; matched JSON-LD + visible HTML is the stronger signal.
8. **Foreign-language slug debt.** Thousands of product handles in Italian/German/French/English from supplier feeds (VidaXL/SoBuy/Wohnling/VASAGLE/Homcom/Aosom). Bulk theme-side remap with sequenced 301s.
9. **`www.lammeuld.dk` separately indexed.** `www -> apex` 301 + HSTS closes the duplicate.
10. **NAP drift.** Google's cache holds 4 distinct phone-hour ranges (9-11, 10-12, 12-14, 16-17) for the same `42 90 54 44` line. Theme-side single source of truth.
11. **Legacy microdata schema in header.** Header logo carries an incomplete `Organization` microdata fragment from Impulse's pre-JSON-LD era, shipping site-wide. I'll strip it and ship one canonical JSON-LD `Organization` block in `theme.liquid` head with the full field set (name, url, logo, address, contactPoint, sameAs to Trustpilot and socials), so every schema signal on the site speaks one dialect.

---

## §3 - The architecture (what gets built)

A retrieval graph under `/llms/`. Each grouping file is one angle into the catalog: category, style, room, sizing, material, use-case, comparison. The same product clusters surface from multiple angles, so AI engines hit them no matter which way the buyer asks.

- **`/llms.txt`** at the root, the procurement-evaluation entry point. Compliance block (EU consumer protection, 14-day fortrydelsesret per dansk købelov, 2-year reklamationsret), brand-entity anchors, directory map.
- **Master `/llms/lammeuld.md`** + matching `Organization` JSON-LD on homepage. Closes the wool-store misclassification (§2.4), anchors brand entity in retrievable text and machine-readable structured data, navigation hub for the rest.
- **Category cluster docs** (`/llms/categories/{handle}.md`): per-Shopify-category retrieval surface for the high-traffic taxonomy: sofaborde, spiseborde, skriveborde, sengeborde, reoler, kommoder, pendler, loftlamper, væglamper, gulvlamper, bordlamper, barstole, spisestole, sofaer, lænestole, opbevaring, badeværelse, stue, spisestue, entre, kontor, bornesenge, udendors. Each pairs the Shopify collection page with extractable claims (count, price band, dominant materials, common rooms, brand mix) and links into related sizing, style, room, and material guides. v1.0's canonicals get AI engines to the collection page; this layer gives them text to quote once they're there.
- **Style guides** (`/llms/styles/{aesthetic}.md`): japandi, skandinavisk, boho, classic-modern, industrial. Maps aesthetic-discovery queries to product clusters.
- **Room guides** (`/llms/rooms/{room}.md`): stue, soveværelse, spisestue, hjemmekontor, hall. Spatial-query intent.
- **Sizing matrices** (`/llms/sizing/{topic}.md`): TV-bord-til-tv, sofabord-til-sofa, spisebord-pladsbehov, lampe-til-rum-størrelse. Number-tables AI engines quote verbatim.
- **Material guides** (`/llms/materials/{type}.md`): eg, fyr, bambus, valnød, messing, sortmat-metal, linnedstof, glas. Durability, sourcing, EU compliance.
- **Care/maintenance** (`/llms/care/{type}.md`): post-purchase by material or product type. Long-tail durability and ownership queries.
- **Comparison docs** (`/llms/compare/{competitor}.md`): vs Lampegiganten, Lampemesteren, AndLight, Nordic Nest. Honest positioning at consideration stage.
- **Use-case clusters** (`/llms/usecases/{handle}.md`): sofabord-under-2000-kr, sengelampe-med-dæmper, tv-bord-til-65-tomme, hyggebelysning-til-stue. Multi-axis intent that doesn't map to one filter.
- **Buying guides** (`/llms/guides/{topic}.md`): vælg-sofabord-størrelse, valg-af-loftslampe, opbevaring-til-lille-hall. Mid-funnel decision content.
- **Trustpilot rating extractability**: buyer-prerequisite (Trustpilot Business subscription on lammeuld's side); separate scoped engagement on top of retainer if you subscribe.

**Example `/llms/sizing/tv-bord-til-tv.md` (excerpt):**

```markdown
# TV-bord størrelse til TV-størrelse - lammeuld.dk

[DOC_TYPE]: sizing-matrix
[INTENT]: practical-fit-decision

## Anbefalet bredde af TV-bord

| TV-størrelse | Min. bredde | Optimal | Produktklynge på lammeuld.dk |
|---|---|---|---|
| 32-43" | 80 cm  | 100-120 cm | [80-120 cm tv-borde](/collections/tv-borde?width=80-120)   |
| 50-55" | 120 cm | 140-160 cm | [120-160 cm tv-borde](/collections/tv-borde?width=120-160) |
| 60-65" | 140 cm | 160-180 cm | [160-180 cm tv-borde](/collections/tv-borde?width=160-180) |
| 70-75" | 160 cm | 180-200 cm | [180-200 cm tv-borde](/collections/tv-borde?width=180-200) |
| 80"+   | 200 cm | 220-240 cm | [200+ cm tv-borde](/collections/tv-borde?width=200-plus)   |

Tommelfingerregel: TV-bordet bør være mindst 20 cm bredere end TV'et på
hver side for visuel balance og kabelplads.

## Højde til sofa

| Sofa-sædehøjde | Anbefalet TV-bord-højde | Produktklynge |
|---|---|---|
| Lav (35-40 cm)          | 40-50 cm | [Lave tv-borde](/collections/tv-borde?height=40-50) |
| Standard (45-50 cm)     | 50-60 cm | [Standard-højde tv-borde](/collections/tv-borde?height=50-60) |
| Sektionssofa (40-45 cm) | 45-55 cm | [Lav-til-standard tv-borde](/collections/tv-borde?height=45-55) |

## Krydshenvisninger

- Stil: [skandinavisk](/llms/styles/skandinavisk.md), [japandi](/llms/styles/japandi.md)
- Rum: [stue](/llms/rooms/stue.md)
- Materialer: [eg](/llms/materials/eg.md), [valnød](/llms/materials/valnoed.md)

[META_TAGS]: tv-bord, fjernsynsbord, sizing, fit, room-decor
```

The Krydshenvisninger block is the network glue. Each grouping file links to related style, room, and material files, so every product cluster has multiple retrieval paths into it. Depth varies by file type: sizing matrices are 1-2 days, style/room/material/buying/category guides are 2-3 days, master doc and competitor comparisons are 4-5 days.

**SEO leg (parallel build, same source files).** Every doc above doubles as a Google-indexed surface, not just an LLM retrieval one. Category docs render into Shopify `collection.description` as deep-content intro blocks (already used on lammeuld for ~half the catalog at shorter depth, e.g. *"Find dine blandt 90+ modeller fra 33 til 180 cm bredde"* on `/collections/bambus`); the long form replaces the supplier-feed thin-content default and competes for Google category SERPs. Buying guides and use-case clusters publish as `/blogs/guides/{handle}` posts, picking up traditional informational queries (*"hvor bredt skal et tv-bord være"*, *"vælg sofabord størrelse"*) that AI engines also cite. Comparison docs publish as `/pages/sammenlign-{competitor}` for branded comparison queries. Sizing matrices embed in both the `/llms/` retrieval surface and the relevant collection-page intros. One source of truth, two delivery channels: AI retrieval under `/llms/`, Google-indexed pages under `/pages/`, `/blogs/`, and inline collection descriptions. The retainer authoring pace (2-4 docs/month) is the same; the SEO leg is incremental wiring on top of the LLM leg, not a separate workstream.

**Schema cleanup, your written approval needed before week 1.** §2.11 confirmed via static + JS-rendered DOM audit that the theme emits exactly *one* legacy microdata `Organization` fragment from the header partial - shipping site-wide on every page, only `url` + `logo`, the older dialect Google has discouraged since 2018. v1.0 dev already ships clean JSON-LD for Product, Brand, Offer, and BreadcrumbList on PDPs (verified on `/products/beech-bookcase-with-full-side-panels-6-shelves-165x60x25-cm`); the legacy microdata is the last remaining inconsistency. The canonical JSON-LD Organization block planned above must replace it in the same push, not ship alongside it - otherwise Google sees two competing Organization records site-wide and entity resolution degrades. Concrete proposal:

1. Strip the four microdata attributes (`itemscope`, `itemtype`, `itemprop="url"`, `itemprop="logo"`) from the header partial in `sections/header.liquid` (or wherever Impulse renders the logo div - one Liquid edit, audit confirmed only one fragment exists site-wide).
2. Author a single canonical JSON-LD `Organization` block in `layout/theme.liquid` head with `name`, `legalName`, `url`, `logo`, `address` (PostalAddress with streetAddress, postalCode, addressLocality, addressCountry), `contactPoint` (telephone + contactType), and `sameAs` array (Trustpilot profile, Facebook, Instagram, LinkedIn, any other identifiers you want crawled).
3. Validate the rendered JSON-LD via Google Rich Results Test (clean validation screenshot in handover).
4. Push to a fresh dev theme duplicate, send preview link, wait for your written sign-off before live publish - same protocol as v1.0.

Estimated effort: 2-3 hours. This is the natural week-1 ship of any retainer tier and a near-zero-risk cleanup, but it edits a global theme file so I want your explicit yes in writing before I touch anything. Reply *approved* to authorize, or *hold* to discuss the JSON-LD field set first.

---

## §4 - The retainer

v1.0 was fixed scope (€1,980, 22-28h). Retainer is ongoing work across AI-search, Shopify engineering, and recurring theme work. Three levels.

### €4,500/month

`/llms.txt` + master `/llms/lammeuld.md` + `Organization` JSON-LD on homepage ship in week 1. Each month after: 2 new grouping files chosen against roadmap (mix of sizing matrix, style guide, room guide, material guide, use-case cluster, or buying guide; cross-references wired to existing files so the retrieval graph compounds), AI Visibility scoring across 12 buyer-intent queries × 4 engines (verbatim responses, competitor placements, month-over-month trajectory), brand-mention monitoring (alerts when lammeuld surfaces or stops surfacing in AI answers), Wikidata entity-claim watch (SPARQL queries against the live entity graph), `llms.txt` content updates, Product JSON-LD deep-cuts wiring on new product additions (material/weight/dimensions/color/pattern/size from existing metafields), canonical drift detector across 5 templates (Python, 50 URLs each), GSC error and coverage monitoring, sitemap and robots.txt health check, Rich Results Test on new PDPs. Each quarter: schema.org / AI Overview spec changes summarized, FAQ schema candidates drafted for high-traffic PDPs (you review, ship if approved). Plus small theme work each month: bug fixes, micro-features (size-guides, Liquid logic, schema field additions, accessibility quick wins, OG-image fixes).

### €6,500/month

Everything in €4,500, plus more authorship and intel. Each month: 2 additional grouping files (4 total per month, mix across all surface types: style, room, sizing, material, care, comparison, use-case, buying guide; comparison docs and master-doc updates included at this tier), competitor retrieval-surface diff against 10 named DK competitors (what they shipped, what counter-moves to make), vision-LLM image alt-text on new catalog additions (Claude/GPT-4V on product images, programmatic), LLM-generated meta-description drafts for collection pages (you review, deploys via metafields), comparative buyer-intent ranking analysis (where each competitor wins, where they lose), product-description deduplication via vector embeddings (find supplier-feed near-duplicates across the catalog, surface clusters for editorial decision). Each quarter: AI Overview eligibility heatmap per category, schema.org entity-graph audit (sameAs link freshness, broken-target removal), internal-link graph analysis (Python, orphans and over-linked nodes), GSC indexation drilldown, automated 404 inventory and repair, schema extension recommendations (FAQ on PDPs, Offer with shipping-speed schema), competitive intelligence report. Plus scoped multi-day Shopify projects each quarter: small features (size-guides, custom upsell logic, metafield UI surfacing, search relevance tuning, email-notification customization), accessibility upgrades, schema extensions.

### €8,500/month

Everything in €6,500, plus you become my largest active retainer with first-priority calendar slots. Shopify engineering scales to multi-week projects: full-surface theme and Liquid work, accessibility (WCAG), metafield architecture, product-variant theme logic, search relevance, Shopify Functions. Larger feature builds: bundle and upsell builders, product configurators, custom search experiences. LLM-driven product-attribute extraction pipeline (parse descriptions, extract material/weight/dimensions/color candidates, queue for your review and theme metafield wiring). Custom retrieval-object generation pipeline (programmatic drafts from product clusters, you review, theme deploys). Editorial AI agent for collection-page intro copy and meta descriptions (LLM drafts, your review, theme deploy). Retrieval-object expansion paced to product introductions and roadmap (no fixed per-month count). Bulk foreign-language slug migration with sequenced 301s. Faceted-nav canonical strategy. Quarterly featured-snippet eligibility audit. Annual schema-evolution roadmap (FAQ/HowTo/Review where catalog supports). Annual competitive moat assessment.

**Out of scope (any tier):** paid ads.

**Engagement.** Upwork hourly/milestone-based or direct contract with monthly invoice. Deliverables in shared private repo. 30-day notice either side. Quarterly review with upgrade or downgrade option. Single point of accountability, no subcontracting.

---

## §5 - Why timing matters

LLM training corpora rebuild monthly. The store with the cleanest retrieval surface today wins placement in the next cycle.

Q1: brand-entity disambiguation begins; long-tail Danish queries start surfacing lammeuld. Q2-3: mid-tail category queries shift. Q3+: peer-tier displacement compounds against Lampegiganten, Lampemesteren, AndLight.

---

## §6 - Contact

`hello@vadimiljin.com`, +372 5551 4747, or via Upwork DM.

---

*§1 evidence captured live 2026-05-09. Retainer terms are firm offers.*
