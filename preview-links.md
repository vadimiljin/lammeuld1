# Dev-theme preview URLs

## Buyer-side cookie note (please read before clicking)

These preview URLs include the dev-theme query parameter, but **that parameter alone is not enough** - Shopify's storefront also requires an admin session cookie to surface the dev theme. Without the cookie, these URLs will render the **live theme** (theme 186499793236), not the dev theme.

Since you (Jonathan and Tina) are Shopify admin collaborators on lammeuld.myshopify.com, your browser already holds the admin session cookie. Open Shopify admin (https://admin.shopify.com/store/lammeuld) in one tab to confirm you're logged in, then click the URLs below in the same browser. They'll surface the dev theme correctly.

If you see the live theme instead of the changes described in HANDOVER.md, your admin session has expired - re-log into Shopify admin and try again.

## 5 Layer A preview URLs (one per template type)

1. **Product page (PDP):** [https://lammeuld.dk/products/wall-lamp-celia-white?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588](https://lammeuld.dk/products/wall-lamp-celia-white?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588)
   - Verifies: 4-branch canonical override (Branch 1 product); Product JSON-LD schema; BreadcrumbList JSON-LD on PDP.

2. **Collection page:** [https://lammeuld.dk/collections/sideborde?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588](https://lammeuld.dk/collections/sideborde?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588)
   - Verifies: 4-branch canonical override (Branch 4 default); BreadcrumbList JSON-LD on collection page.

3. **Paginated collection:** [https://lammeuld.dk/collections/alle-produkter?page=2&_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588](https://lammeuld.dk/collections/alle-produkter?page=2&_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588)
   - Verifies: 4-branch canonical override (Branch 3 paginated → page 1; pagination deviation per Tina's spec; see HANDOVER.md §4.2).

4. **Blog post:** [https://lammeuld.dk/blogs/lammeuld-dk-moebel-blog/historien-bag-montessori-mobler?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588](https://lammeuld.dk/blogs/lammeuld-dk-moebel-blog/historien-bag-montessori-mobler?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588)
   - Verifies: Article JSON-LD schema; BreadcrumbList JSON-LD on article page (extension added 2026-05-08).

5. **Blog index:** [https://lammeuld.dk/blogs/lammeuld-dk-moebel-blog?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588](https://lammeuld.dk/blogs/lammeuld-dk-moebel-blog?_ab=0&_fd=0&_sc=1&preview_theme_id=196009525588)
   - Verifies: 4-branch canonical override (Branch 4 default; blog index unchanged).

## How to verify (View Source quick-check)

1. Click a URL above (with admin session cookie active).
2. Right-click → View Page Source (or `Ctrl+U`).
3. Search (`Ctrl+F`) for `<link rel="canonical"` - the canonical URL should match the destination per the 4-branch logic in HANDOVER.md §4.1.
4. Search for `<script type="application/ld+json">` - for PDPs you should see Product + BreadcrumbList; for blog posts Article + BreadcrumbList; for collections BreadcrumbList; etc.

More preview URLs available on request.
