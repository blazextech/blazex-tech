# BlazeX Tech — Deployment Log

Record of changes pushed directly to the live blazextech.com site (outside the normal Customizer UI), for traceability since the WordPress database itself isn't version controlled.

---

## 2026-06-26 — Typography, Additional CSS sync, WhatsApp button

**Typography:** Set `family` to `Inter` for `rootTypography`, `h1Typography`, `h2Typography`, `h3Typography`, `h4Typography` in the `theme_mods_blocksy` option (Customizer → Typography → Base Font / H1–H4). Headings were previously inheriting via `"family": "Default"`; now set explicitly. Verified live in `wp-content/uploads/blocksy/css/global.css` (`:root`, `h1`–`h4` all show `--theme-font-family: Inter, Sans-Serif`).

**Cache:** Cleared `blocksy_dynamic_styles_descriptor` transient, re-triggered Blocksy's `blocksy:dynamic-css:refresh-caches` action, flushed the WP object cache, and purged LiteSpeed Cache (`wp litespeed-purge all`).

**Custom CSS:** No Blocksy child theme exists on the site (only the parent `blocksy` theme is active), and dropping `style.css` into `wp-content/themes/blocksy/assets/css/` would (a) do nothing on its own — nothing enqueues it — and (b) get wiped on the next Blocksy update. Instead, [src/theme/style.css](../src/theme/style.css) (minus its child-theme header comment) was appended to Blocksy's native Additional CSS field, which WordPress core stores as a `custom_css` post (ID 5) via `wp_update_custom_css_post()`. This survives theme updates and is exactly what the file's own top comment recommended ("Apply in: WordPress Customizer → Additional CSS").

**WhatsApp sticky button:** Added via the Easy Code Manager plugin (a rebrand of FluentSnippets, already active on the site) rather than editing theme `functions.php` directly — same update-safety reasoning as above. Snippet type `php_content`, hook `wp_footer`, status `published`. Source kept in [src/theme/whatsapp-footer-snippet.php](../src/theme/whatsapp-footer-snippet.php) for version control; the live copy lives in `wp-content/fluent-snippet-storage/3-blazex-whatsapp-sticky-button.php` (not committed — file-based plugin storage, not part of the theme). Links to `https://wa.me/923473407764`.

**Verified live:** `:root`/`h1`-`h4` Inter fonts, Additional CSS rules, and the rendered `.blazex-whatsapp-btn` link all confirmed present in the served HTML/CSS after cache purge.

---

## 2026-06-27 — Site logo and header CTA button

**Logo:** Of the candidate files in `assets/logos/`, only [logo.png](../assets/logos/logo.png) (800×315 RGBA) is an actual flat logo — wordmark + flame icon, transparent background, no watermark. The other JPGs (`logo.jpg`, `square.jpg`, `grok_image_83mjvm.jpg`, `grok_image_ejkql9-1.jpg`) are AI-rendered "on a wall" mockups with a baked-in dark background and a visible Grok watermark; `final file.png` is the flame icon only, with no wordmark, sitting in a mostly-empty canvas. Imported `logo.png` into the media library (attachment ID 1197) and set it via `wp theme mod set custom_logo 1197`, replacing the old placeholder SVG (attachment 681). Verified live: header `<img>` now points at `wp-content/uploads/2026/06/blazex-logo.png`.

**Header CTA button:** Blocksy stores this under `theme_mods_blocksy['header_placements'].sections[].items[id=button].values`. Changed `header_button_text` from "Get in Touch" to "Get a Quote", and set `header_button_link` (previously unset, defaulting to `#`) to `https://blazextech.com/contact/`. Verified live: button renders with the new label and `aria-label`, linking to the Contact page.

**Cache:** Same refresh routine as above (Blocksy dynamic-CSS transient + action, WP object cache flush, LiteSpeed purge).

**Incident — settings reverted:** Shortly after, the header logo and button reverted to "Codespot"/"Get in Touch". Root cause: an open WP Customizer browser tab (loaded before these edits) was published, and Blocksy's header builder saves its entire `header_placements` value as one atomic blob — overwriting our nested `header_placements.logo.custom_logo` and `header_placements.button.values` back to their pre-edit state. The top-level `custom_logo` theme_mod survived because it's a separate key written via a different path. Re-applied both values (and synced the nested logo reference to attachment 1197 this time) once the tab was closed without saving. Lesson: avoid leaving the Customizer open while scripting `theme_mods_blocksy` changes via WP-CLI.

---

## 2026-06-27 — Border Animation snippet color

**Change:** The "Border Animation" CSS snippet (FluentSnippets/Easy Code Manager, `type: css`, `run_at: wp_head`, `load_as_file: yes`) had `--border-animation-color: #52fe7d73` (a leftover green from a previous theme), updated to `--border-animation-color: #E8570E73` (Blaze Orange, matching the brand palette). Source kept in [src/theme/border-animation.css](../src/theme/border-animation.css) for version control.

**Note:** Because `load_as_file: yes`, the plugin serves a *compiled* copy from `wp-content/fluent-snippet-storage/cached/1-border-animation.css` via a directly enqueued URL — editing only the source `.php` file's docblock-wrapped code would not have changed what's actually served. Both the source and the cached file were updated in lockstep.

**Cache:** Same refresh routine (Blocksy dynamic-CSS transient + action, WP object cache flush, LiteSpeed purge). Verified live via direct fetch of the cached CSS URL.

**Follow-up — stale cache-busting version:** The color still showed as green in the browser after the above. Cause: the source/cached files were edited directly with `sed`, bypassing FluentSnippets' own index rebuild — the enqueued `<link>` URL's `?ver=` parameter comes from `strtotime($snippet['updated_at'])` in the plugin's cached index (`wp-content/fluent-snippet-storage/index.php`), not from re-reading the snippet file on each request. Since that index still held the old `updated_at`, the URL was byte-identical to before the edit (`?ver=1764843894`), so any browser that had already loaded it kept serving its own cached copy indefinitely — the new file content on disk was correct, but nothing forced a re-fetch. Fixed by calling `Helper::cacheSnippetIndex('', true)` to rebuild the index from the snippet files' docblocks, which bumped the version to `?ver=1782543432`. Lesson: editing FluentSnippets-managed files directly on disk needs a forced index rebuild afterward, or use the plugin's update path instead of raw file edits.

---

## 2026-06-27 — Sitewide green overlay removal

After the border-animation fix, the whole layout still had a greenish cast. Found two separate, unrelated sources of leftover neon-green from the original Codespot demo import:

**1. `.ct-boxshadow` utility class:** Pre-existing Blocksy Additional CSS (post ID 5, not something this project added) had `box-shadow: 0 -1px 0 0 #52ff7d29, 0 0 0 1px #ffffff1f;`, applied to 22 elements site-wide via the `ct-boxshadow` class. Replaced `#52ff7d29` with `#E8570E29` (brand orange, same alpha) via `wp_update_custom_css_post()`.

**2. Elementor container background gradients:** The real source of the dominant "overlay" look — 17–18 containers per page (Home, About, Services, Contact) had dark green gradient backgrounds (`background_color: #0A2B126B` / `#0530083D` / `#051F063D`, paired with `background_color_b: #0C0C0C` / `#021E045C`) baked directly into each page's `_elementor_data` postmeta. Editing the generated CSS files in `wp-content/uploads/elementor/css/` directly would not have stuck — Elementor regenerates them from postmeta. Instead, patched the JSON in `_elementor_data` for posts 1182, 1184, 1186, 1191, replacing every green hex with the brand's Steel Dark (`#1C2333`) at the *same alpha value* (e.g. `#0A2B126B` → `#1C23336B`), preserving the existing gradient depth/opacity while removing the green hue. Then cleared Elementor's CSS cache (`\Elementor\Plugin::$instance->files_manager->clear_cache()`) to force regeneration from the updated data.

**Cache:** Same full routine (Elementor cache clear, Blocksy dynamic-CSS transient + action, WP object cache flush, LiteSpeed purge). Verified by diffing hex colors in the regenerated `post-*.css` files — zero remaining matches for the old green values across all four pages.

**Follow-up — Elementor editor clobbered the homepage again:** Same class of bug as the Customizer incident, different editor. The homepage Elementor editor was open in a browser tab with the pre-fix (green) page state loaded; saves from that tab push the *entire* `_elementor_data` blob back, so each save reverted whichever containers weren't manually touched. Result: 14 of 17 containers reverted to `#0A2B126B`, one pair reverted to `#0530083D`/`#021E045C`, and one manually-recolored container ended up at `#1C233369` — a shade close to but not matching the brand value, which is why the mixed result "didn't look good." The other three pages (About/Services/Contact) were untouched and stayed correct. Re-applied the same replacement map to post 1182 only (including normalizing `#1C233369` → `#1C23336B`), then cleared Elementor's cache again. Lesson: don't have the Elementor editor open for a page while its data is being patched via WP-CLI — same rule as the Customizer, different tool.

---

## 2026-06-27 — Recolored green geometric shape images

Even after the overlay-color fixes, three actual *image* assets from the Codespot demo were still green — these can't be fixed with a hex find-and-replace since they're raster photos/renders, not flat colors. All three were also still hotlinked directly to `startersites.io` (the demo site), never localized.

- **`bg-test-12-scaled-1.jpg`** and **`bg-test-section-2-2-scaled-1.jpg`**: large geometric chevron/wave background images, used as a full-section `background_image` on Home, About, Services, and Contact (one or both per page).
- **`home-hero-image-2.webp`**: the literal 3D metallic icosahedron shape in the homepage hero — this is the item already flagged in this file's "Codespot Template Customization" section ("recolor green geometric shape → orange").

**Approach:** A plain HSV hue-rotation left visible green JPEG-compression artifacts along the chevron edges (numerically unstable at near-black pixel values), so the two background images were instead reconstructed as a clean duotone: take the green channel as a pure intensity/brightness map and recombine it with the brand orange (`#E8570E`) RGB ratios, discarding the original (noisy) hue entirely. The 3D shape image used a different method — a selective mask (`green or cyan channel notably above red`) recolored only the green/cyan metal pixels by brightness, leaving the white background and specular highlights untouched.

**Applied:** Uploaded all three recolored images to the media library (attachment IDs 1216, 1217, 1218) and replaced the exact `"url":...,"id":...` pairs in `_elementor_data` for the affected pages (1182, 1184, 1186, 1191), so the site now serves its own local copies instead of hotlinking the demo site. Source copies kept in [assets/images/](../assets/images/) for version control. Cleared Elementor's CSS cache plus the usual Blocksy/object/LiteSpeed cache routine, then verified the new filenames in the regenerated `post-*.css` files on all four pages.

**Follow-up — five more green shape images flagged by user:** `home-features-image-2.webp`, `home-about-section-image-4-1.webp`, `home-about-section-image-5.webp` (homepage), and `services-page-image-4.webp`, `services-page-image-6.webp` (Services page) — the same family of translucent green 3D-rendered shapes (torus, rounded cube, swirl, overlapping spheres). These were already serving from `blazextech.com` rather than the demo site (something — likely a Hostinger/LiteSpeed image-proxy layer — was already rewriting the `startersites.io` URLs at render time), but the underlying `_elementor_data` still pointed at the original demo URLs/IDs.

Recolored using the same selective channel-mask technique as the hero shape (`g > r+5 OR b > r+5` → recolor by brightness toward `#E8570E`, else leave untouched). One image (`services-page-image-4`) initially kept a couple of visible cyan patches near the edges with a `+15` threshold — these were transitional/anti-aliased pixels where the green cast was too subtle to clear the margin. Lowering the threshold to `+5` caught them cleanly with no visible side effects on the white background or highlights.

Uploaded all five (attachment IDs 1219–1223), swapped the exact `url`/`id` pairs in `_elementor_data` for posts 1182 and 1186, cleared the same cache chain, and verified each new filename appears in the regenerated CSS/HTML. Source copies added to [assets/images/](../assets/images/).
