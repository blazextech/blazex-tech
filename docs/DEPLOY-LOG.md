# BlazeX Tech — Deployment Log

Record of changes pushed directly to the live blazextech.com site (outside the normal Customizer UI), for traceability since the WordPress database itself isn't version controlled.

---

## 2026-06-26 — Typography, Additional CSS sync, WhatsApp button

**Typography:** Set `family` to `Inter` for `rootTypography`, `h1Typography`, `h2Typography`, `h3Typography`, `h4Typography` in the `theme_mods_blocksy` option (Customizer → Typography → Base Font / H1–H4). Headings were previously inheriting via `"family": "Default"`; now set explicitly. Verified live in `wp-content/uploads/blocksy/css/global.css` (`:root`, `h1`–`h4` all show `--theme-font-family: Inter, Sans-Serif`).

**Cache:** Cleared `blocksy_dynamic_styles_descriptor` transient, re-triggered Blocksy's `blocksy:dynamic-css:refresh-caches` action, flushed the WP object cache, and purged LiteSpeed Cache (`wp litespeed-purge all`).

**Custom CSS:** No Blocksy child theme exists on the site (only the parent `blocksy` theme is active), and dropping `style.css` into `wp-content/themes/blocksy/assets/css/` would (a) do nothing on its own — nothing enqueues it — and (b) get wiped on the next Blocksy update. Instead, [src/theme/style.css](../src/theme/style.css) (minus its child-theme header comment) was appended to Blocksy's native Additional CSS field, which WordPress core stores as a `custom_css` post (ID 5) via `wp_update_custom_css_post()`. This survives theme updates and is exactly what the file's own top comment recommended ("Apply in: WordPress Customizer → Additional CSS").

**WhatsApp sticky button:** Added via the Easy Code Manager plugin (a rebrand of FluentSnippets, already active on the site) rather than editing theme `functions.php` directly — same update-safety reasoning as above. Snippet type `php_content`, hook `wp_footer`, status `published`. Source kept in [src/theme/whatsapp-footer-snippet.php](../src/theme/whatsapp-footer-snippet.php) for version control; the live copy lives in `wp-content/fluent-snippet-storage/3-blazex-whatsapp-sticky-button.php` (not committed — file-based plugin storage, not part of the theme). Links to `https://wa.me/923473407764`.

**Verified live:** `:root`/`h1`-`h4` Inter fonts, Additional CSS rules, and the rendered `.blazex-whatsapp-btn` link all confirmed present in the served HTML/CSS after cache purge.
