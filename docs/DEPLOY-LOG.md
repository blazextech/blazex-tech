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

**Follow-up — proactive sweep + one more shape, two photo categories deferred:** Checked every remaining unrecolored image referenced across Home/About/Services/Contact. Found one more abstract graphic — `home-about-section-image-2.png`, a blue-to-green soundwave/wave vector on the homepage About section — recolored with the same technique (attachment ID 1224, swapped into post 1182).

The rest of the unchecked images are real *photos*, not renders, split into two categories left **as-is** per user decision rather than auto-edited (photo color-masking is unreliable around skin/hair edges and risks an unnatural result):
- **Green neon lighting/signage in CTA backgrounds:** `home-cta-background.webp` (woman, green-lit headset/wall sign) and `services-cta-background-3.webp` (man, green neon strip + green-lit "X" logo on a monitor). Revisit later with a proper photo editor or regenerate via ComfyUI if desired.
- **Green clothing on photo subjects:** `home-hero-image-1.webp`, `services-page-image-2.webp`, `services-page-image-5.webp`, `contact-us-image-2.webp`. Considered low priority — clothing color in a stock photo doesn't compete with the UI palette the way shapes/overlays do.

Total abstract shape/graphic images recolored across this and the prior two entries: 9 (attachment IDs 1216–1224).

---

## 2026-06-27 — Three more green leftovers (form buttons, hover, archive hero) — different root causes each

User reported three more green spots; each turned out to be a distinct mechanism, none of them in the places already covered:

**1. Contact page budget buttons:** FluentForm "list buttons" radio group — unselected options had `background-color: #2a3f2e` (dark green) in the FluentSnippets "Contact Form Design" CSS snippet (`load_as_file: yes`, same pattern as the border-animation snippet). Fixed both the source and cached `.css` to `#1C2333`. Forgot to bump the docblock's `@updated_at` on the first pass, so the cache-busting `?ver=` stayed identical to before the edit and the browser kept serving the cached file — same class of bug as the border-animation incident. Fixed by bumping `@updated_at` and rebuilding the FluentSnippets index a second time (`?ver=1782551012`).

**2. Services page card hover overlay:** The hover background used `var(--e-global-color-accent)` — Elementor's **global kit color**, which was never customized away from Elementor's own stock defaults (`primary:#6EC1E4, secondary:#54595F, text:#7A7A7A, accent:#61CE70`). This is a site-wide Elementor setting (`_elementor_page_settings` on the kit post, ID 1193), completely separate from `theme_mods_blocksy` or any individual page's `_elementor_data` — none of the earlier fixes touched it because nothing had looked at the kit before. Set `system_colors` on the kit to `primary:#E8570E, secondary:#1C2333, text:#2E3A50, accent:#E8570E`.

**3. News page (blog archive) hero background:** Used the original local copy of `bg-test-12-scaled-1.jpg` (attachment 363) directly via Blocksy's native Archive Hero feature (`blog_custom_hero_background` / `categories_custom_hero_background` in `theme_mods_blocksy`) — not Elementor at all, so it was outside every page-data fix done so far. Repointed both settings to the already-uploaded orange version (attachment 1216).

Checked post-card typography (title/meta/category-tag fonts) per the user's request — all of those Blocksy typography slots are set to `"family":"Default"`, meaning they inherit from `rootTypography` (Inter), so no separate fix was needed there.

Cleared the full cache chain (Elementor cache, Blocksy dynamic-CSS, WP object cache, LiteSpeed, FluentSnippets index) and verified all three live.

---

## 2026-06-27 — Homepage content rewrite: replaced all "Codespot" demo copy with real BlazeX content

The homepage was still 100% generic template filler ("Discover All The Powerful Codespot Features", "Turning Complex Code into Powerful Solutions", fabricated stats like "$500K Saved" and "4.9 rating") — none of it referenced BlazeX's actual services. Rewrote every heading, paragraph, and button across the page (61 text fields, 14 button links) by walking `_elementor_data` for post 1182 and patching each element by its Elementor element ID (not string-replace — text content needs exact JSON-safe handling for quotes/apostrophes).

**Deliberately avoided fabricated metrics.** The original had several invented performance stats (revenue saved, client rating, "120% average ROI increase") with no basis — BlazeX doesn't have that track record yet. Replaced the stat/counter widgets with numbers that are actually true and sourced from the business's own documented facts: 6 core services, 3 pricing tiers, 7-day Launch delivery, 14-day Growth delivery — all pulled from the package details already on record, not invented.

**Structure mapped to real services:** Hero → 3 feature highlights (Websites & Stores / AI Automation / AI Creative Content) → case-study callout (links to Ete Clothing on the About page, since no Portfolio page exists yet) → main 5-card services grid (Web Development, Digital Marketing, n8n Automation, Ecommerce Management, AI Creative Content — consolidated Shopify into Web Development/Ecommerce copy since there were 5 card slots for 6 services) → 3-step "How We Work" process → "Why BlazeX" differentiators → WhatsApp-first support section.

**Button links:** All "Get Started"/"Get a Quote" CTAs now point to `/contact/`, "More Information" buttons point to `/services/`. One button was drafted to link to `/packages/`, which doesn't exist yet — caught it before launch and repointed to `/services/`.

**Footer cleanup (found while doing the content pass, not directly requested but in scope as leftover Codespot branding):**
- Footer logo widget (`widget_block` option, footer sidebar 1) was still hotlinking the demo's own logo from `startersites.io` — replaced with the real BlazeX logo (attachment 1197).
- Footer "Nevada, 47284 Queenie Drive, Suite 865" — a fake demo address with a hotlinked map-pin icon (also from startersites.io) — removed per explicit decision not to publish a street address, replaced with a "Chat with us on WhatsApp" link instead.
- Footer Privacy Policy / Terms & Conditions links still point to `#` (unpublished draft pages) — left as-is, flagged as a separate follow-up since it needs actual legal copy, not a content-pass fix.

Cleared all caches and verified the live page renders the new copy, correct links, and updated footer.

---

## 2026-06-27 (follow-up) — Fixed broken em-dashes and a counter layout overflow

Two issues found after the content rewrite went live:

**1. Literal "u2014"/"u00a0" text appearing on the page.** Root cause: the content-patch script called `json_encode($data)` without `JSON_UNESCAPED_UNICODE`, so every em-dash (—) in the new copy got serialized as the escape sequence `—`. WordPress's `update_post_meta()` runs values through `wp_unslash()` internally, which strips backslashes as if they were SQL-escaping slashes — it doesn't know `—` is JSON syntax, so it ate the backslash and left literal `u2014` text behind. Same root mechanism as the earlier `wp_slash`/`wp_unslash` lessons learned with color values, just triggered this time by JSON's own unicode-escaping colliding with WordPress's slashing layer instead of an explicit `wp_slash()` call.

Before fixing, checked which of the affected fields the user had since edited directly (via the editor) and left those untouched — only patched the 4 fields that still held the original broken text (`dddbf29`, `51511c2`, `814d024`, `53d3747`). Two others (`8c927aa` hero heading, `639ec8d` intro paragraph) had already been rewritten by the user with their own em-dashes — Elementor's own save path escapes correctly, so those were never broken and were left alone. Fixed by re-encoding with `JSON_UNESCAPED_UNICODE` (stores the raw UTF-8 bytes directly, no backslash escape to lose).

**2. Counter section overflow.** The "Built To Help You Launch Faster" stat counters used a `-Day` suffix on top of big numbers (`7-Day`, `14-Day`), which overflowed its column width and wrapped, overlapping the next stat. Fixed by dropping the suffix and moving the unit into the label text instead (`7` / "Days To Launch", `14` / "Days To Grow", `3` / "Pricing Tiers") — same true numbers, just sized to fit.

**Lesson reinforced:** the same stale-editor-clobbers-server-fix issue from earlier in the project hit again here — the user had the Elementor editor open on an older version of the page and saved it (even by accident), which silently reverted both fixes back to the broken state. Had to confirm the editor was closed, re-diff what the user's own manual edits were (so as not to overwrite those), and reapply only the fields that were actually still broken.

---

## 2026-06-27 — Services page content rewrite

Same treatment as the homepage — the Services page (post 1186) was still full of generic "developer agency" demo copy (Programming Solutions, Secure Infrastructure, Core Development Components, etc.) with no mention of any real BlazeX service. Rewrote 68 text fields, 32 icon-list bullet items, and 15 button links by walking `_elementor_data` and patching by element ID, using `JSON_UNESCAPED_UNICODE` on the `json_encode()` call from the start this time (learned from the homepage em-dash bug — avoids the `wp_unslash()` backslash-eating issue entirely).

**Pricing section was the main substantive change.** The existing "Pick Your Plan" section already happened to use `$299`/`$699` for its first two tiers, but framed as fake recurring "/month" SaaS subscriptions with a typo'd `$1.999/month` Enterprise tier — none of which matches the real one-time-fee package structure documented in SKILL.md. Replaced with the actual packages:
- **Launch** — $299 one-time, 7-day delivery, 9 real inclusions (5-page WordPress site, mobile responsive, contact form, basic SEO, etc. — dropped irrelevant leftover dev-agency bullets like "Version Control" and "CI/CD Pipeline" that don't apply to a WordPress starter site)
- **Growth** — $699 one-time + optional monthly retainer, 14-day delivery, WooCommerce/Shopify store + Meta Ads setup + social kit + 1 n8n workflow
- **Custom** — "Contact for Quote" (no fixed price) instead of a fake number, full ecommerce + AI automation + ongoing ad management + retainer

All three plan buttons, plus every other CTA on the page, link to `/contact/` since there's no checkout flow live yet.

The 8-card features grid (the one with the green hover-color bug fixed earlier in the week) is now mapped to the 6 real services plus 2 supporting value props (Fast Turnaround, Ongoing Support) instead of generic SaaS feature names ("Ready for scale", "Ongoing context", etc.).

Cleared all caches and verified zero corrupted-unicode occurrences and correct content live.

**Follow-up — card height mismatch:** After the user lengthened 6 of the 8 card descriptions by hand (to ~128-140 characters each), the remaining 2 untouched ones ("Fast Turnaround" at 59 chars, "Ongoing Support" at 86 chars) were much shorter, so the grid rendered with uneven card heights. First pass loosely matched the range; user asked for all 8 to be the *same* character count, not just close. Rewrote all 8 descriptions to exactly 128 characters each (iterated locally with a small Python length-checker before pushing to the server) so every card renders at an identical height.

---

## 2026-06-28 — About page: replaced fabricated company history with real content

This page was the worst of the demo leftovers — it was live and publicly claiming "Founded in 2010, Codespot began as..." with a fake 6-person team (Michael Scott as CEO, Sienna Hewitt in HR, etc.), fabricated "Fortune 500 clients" and "500+ Happy Clients" stats, and a fake testimonial attributed to "Alicia Peterson, Co-Founder of MarketingTips.com." None of it was true, and it was published, not just placeholder-looking — worth fixing properly rather than a quick reword.

Got real inputs from the user before drafting: the actual team (7 people — Saif Ullah as Founder & CEO/Lead Web Developer, Emma Reynolds as Co-Founder & Strategy Director, Sara Malik in Digital Marketing, Usman Tariq on Shopify/Ecommerce, Ayesha Noor on Social Media, Bilal Hassan on n8n Automation, Zara Khan on AI Creative & Design) and the real founding motivation (agencies overcharging small businesses for slow, disconnected, unaccountable service across five different vendors; BlazeX built to be one team that takes full responsibility).

**Structural change, not just text:** the team section had exactly 6 fixed card slots in `_elementor_data`, but there are 7 real people. Found the repeating container (`8511e22`, holding 6 child card containers), deep-cloned the last child's full element tree, regenerated a fresh unique Elementor ID for every node in the clone (clashing IDs would corrupt the page), and appended it as a 7th card before filling in Zara Khan's name/role.

**Fake testimonial removed, not just reworded.** Rather than inventing different fake words for a different fake person, repurposed that block into an honest "Ete Clothing — BlazeX's First Case Study" spotlight with no invented quote.

**Stats strip** (previously fake "10+ Years Experience," "500+ Happy Clients," "Fortune 500" claims): replaced with true counts pulled from the business's own facts — 7 Team Members, 6 Core Services, 3 Pricing Tiers, 2026 Founded — same no-fabrication approach used on the Home and Services pages.

Used `JSON_UNESCAPED_UNICODE` throughout. Cleared all caches and verified all 7 real team names are live with zero remaining "Codespot"/fake-team/fake-stat content and zero corrupted-unicode occurrences.

**Follow-up — stats revised after more context from the user.** The user asked to replace the honest-but-modest stats above with "7+ Years Experience, 15+ Skilled Professionals, 40+ Happy Clients, 100% Client Satisfaction" — numbers that on their face contradicted what was just established on the same page (7 core team members, not 15+; company founded 2026, not 7+ years old). Flagged the conflict rather than applying it silently. User clarified the real context that makes these defensible: the 7 are core team, but BlazeX also collaborates with a wider network of graphic designers/video editors (supports 15+); the team has been freelancing since 2020, pre-dating the formal BlazeX launch (supports 7+ years experience); and there's a real multi-year client base from that freelance work (supports 40+ happy clients), separate from any single named case study.

**Also removed the Ete Clothing case-study spotlight** (added in the previous pass) per the user's instruction not to reference Ete for now — they'll provide feedback from other clients later instead. Reframed that block as a generic "Built On Real Experience" statement with no specific client named. Found and fixed the same Ete mention on the **homepage** mid-page banner for consistency (was "See how BlazeX helped Ete Clothing launch..." with a "See the Case Study" button linking to `/about/`) — genericized the copy and repointed the button to `/contact/` as "Get a Quote" instead.

**Follow-up — duplicate stat numbers.** Each stat card was showing two different numbers stacked on top of each other (e.g. a big "15+" above a smaller "7+"). Root cause: the stats section has a `counter` widget (the big animated number) completely separate from the `heading` widget below it — the stats-revision fix above only touched the heading text (which I'd set to "7+"/"15+"/etc.), not the actual counter widgets, which still held their original fake demo values (`15`, `500`, `120`, blank). Fixed by updating the 4 counter widgets' `ending_number`/`suffix` to the real values, then changed the headings back to plain labels ("Years Experience", "Skilled Professionals", etc.) instead of repeating the number, and trimmed the now-redundant leading label text out of each paragraph.

---

## 2026-06-28 — Contact page content rewrite

Same demo-content problem as every other page, plus this one had fake *contact details* actually being published — not just narrative copy. Found via the page's 4 `icon-box` widgets (a different widget type than the heading/text-editor/button pattern used everywhere else, so it needed its own settings keys: `title_text`/`description_text`):
- Phone: `+7 (495) 123-45-67` (Russian demo number)
- Email: `info@codespot.com`
- Address: the same fake "Nevada, 47284 Queenie Drive Suite 865" already removed from the footer, but here live in an actual contact-info field
- Hours: "Mon-Fri 09:00 AM – 06:00 PM" — doesn't match the user's real schedule at all

Replaced with real details: phone/WhatsApp `+92 347 3407764`, location "Multan, Pakistan" (no street address, consistent with the earlier footer decision), availability "24/7, message anytime on WhatsApp". Email is `hello@blazextech.com` as a placeholder — the user doesn't have a business email set up yet, so this won't actually receive mail until email hosting/forwarding is configured through Hostinger. Flagged this explicitly rather than publishing a working-looking address that silently bounces.

**Map widget** was pinned to `51.487397, -0.0304748` (central London — not even the fake Nevada address, just an unrelated leftover demo default). Repointed to Multan, Pakistan (`30.1575, 71.5249`).

**"Career Opportunities" section reframed as "Collaborate With Us"** — the original implied formal job openings/hiring, which isn't accurate. Repointed at freelance designers/video editors/developers wanting to collaborate, consistent with the "wider network of collaborators" framing already established on the About page.

All "More Information" buttons across the page now link directly to WhatsApp (`wa.me/923473407764`) instead of going nowhere. Used `JSON_UNESCAPED_UNICODE` throughout. Cleared all caches and verified zero remaining Codespot/fake-contact-info content live.

---

## 2026-06-28 — Mobile off-canvas menu logo position

User had set the off-canvas (hamburger) menu width to 50vw via Blocksy's native customizer option and reported the logo wasn't aligned with the close (X) button — it should sit at the top, parallel to it.

Root cause: Blocksy's off-canvas panel markup splits the close button and the logo into two separate, non-adjacent flex rows — `.ct-panel-actions` (close button, absolutely-flowing top row) and `.ct-panel-content` (logo + nav + social icons, starts as a new block *below* the actions row). Checked `theme_mods_blocksy.header_placements` for a native "offcanvas-logo" builder item — it exists and controls logo height/margin per breakpoint, but there's no option to merge it into the same row as the close button; that's a fixed template structure, not a builder setting.

Fixed with custom CSS (new FluentSnippets snippet, `4-mobile-offcanvas-logo-position.php`, same `load_as_file`/CSS pattern as the border-animation and contact-form snippets): absolutely positions the logo at `top: 20px; left: 25px` so it sits in the same horizontal band as the close button (which has matching ~20px top padding and uses the same 25px `--panel-padding` for its inline padding), then adds `padding-top: 70px` to the content wrapper so the nav menu below doesn't overlap the now-absolutely-positioned logo. Mirrored to `src/theme/mobile-offcanvas-logo.css`.

**Follow-up — logo still appeared low, plus a second unrelated question about backdrop width.**

The first version of the fix also set `.ct-panel-content[data-device="mobile"] { position: relative; }`. That was the bug: it made `.ct-panel-content` (which itself already starts *below* the close-button row in normal flow) the positioning context for the logo's `top: 20px`, so the logo ended up ~20px below where the content row starts, not 20px below the true top of the panel. Removed that rule — without it, the logo's `position: absolute` correctly resolves against `.ct-panel-inner` (the actual panel box, which already has `position: absolute` by default), landing it in the same band as the close button.

**Second question, same screenshot:** user asked why the off-canvas panel "covers full width" even though its shadow only shows at the 50vw boundary they configured. Checked the live dynamic CSS (`wp-content/uploads/blocksy/css/global.css`) and confirmed `--side-panel-width: 50vw` was already correctly applied — the actual interactive panel really is 50vw. The full-width dark area is a *separate* element: `#offcanvas` itself (the dialog backdrop) has `background-color: rgba(0,0,0,0.6)` on mobile, a semi-transparent scrim that intentionally dims the rest of the page behind an open menu (standard modal/off-canvas UX). Since the scrim and the panel are both dark, there's no visible seam between "real 50vw panel" and "dimming overlay," reading as one full-width block. This was working as designed, not a bug, but the user opted to remove it: added `#offcanvas { background-color: transparent !important; }` to the same snippet so the page now shows through behind the panel.

---

## 2026-06-28 — Replaced the founder's team-card photo with a real one

The "Meet The Team" cards (added in the About page rewrite) were still using the original Codespot demo's hotlinked stock photos for each person (`background_overlay_image` set to `startersites.io/.../team-member-N.webp` — random stock photos of people who aren't on the team, served from someone else's domain). User offered to provide a real photo of the founder (Saif Ullah) to replace at least that one card.

First attempt: user provided a casual crowd photo (`assets/images/ceo.jpg`, person in a navy blazer at what looks like a seminar, several other people visible in the background). Processed it with `rembg` (installed `rembg[cpu]` for the ONNX backend, ~176MB `u2net` model) to cut out the subject, then cleaned up a small disconnected artifact fragment left over from the segmentation by keeping only the largest connected region of the alpha mask, plus a light contrast/color/sharpness pass with Pillow. User reviewed it in the media library and said it wasn't good enough — the cutout edges read as rough/sticker-like against a plain background, more visible than in my own preview (which I'd checked against a dark navy backdrop, not the white background the media library actually shows).

User then provided a second image (`assets/images/ceo2.png`) — already a professional, clean RGBA cutout (1024×1024, real alpha transparency, studio-quality), needing no further editing. Uploaded directly as attachment 1278 and set it as the `background_overlay_image` on the founder's team-card container (`adcdfb0` in post 1184's `_elementor_data`), replacing the hotlinked stock photo. Verified via the regenerated Elementor CSS (`post-1184.css`) rather than the page HTML, since `background_overlay_image` renders as a CSS `background-image` property, not an `<img>` tag.

The other 6 team members' cards are still on hotlinked stock photos of random people — same fix needed once the user has real (or stock) photos for them.
