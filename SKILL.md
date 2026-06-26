# BlazeX Tech — Skills & Patterns

## Skill: WordPress Theme Customization
**When to use:** Any task touching Blocksy theme, Elementor pages, or WP Customizer settings.

### Blocksy Global Colors (paste directly into Customizer)
```
Accent / Primary:  #E8570E
Accent Hover:      #F4A024
Header BG:         #1C2333
Footer BG:         #1C2333
Footer Text:       #C8CDD6
Body BG:           #F5F6F8
Heading Color:     #1C2333
Body Text:         #2E3A50
```

### Elementor Section Pattern
Every page section follows this rhythm:
1. Light section (bg: #F5F6F8)
2. Dark section (bg: #1C2333)
3. Repeat

### Standard CTA Button CSS
```css
/* Primary button */
.blazex-btn-primary {
  background: #E8570E;
  color: #ffffff;
  padding: 13px 28px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 700;
  border: none;
  font-family: 'Inter', sans-serif;
  letter-spacing: 0.02em;
  transition: background 0.2s;
}
.blazex-btn-primary:hover {
  background: #F4A024;
}

/* Secondary / outline button */
.blazex-btn-outline {
  background: transparent;
  color: #E8570E;
  padding: 12px 26px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 600;
  border: 2px solid #E8570E;
}
```

---

## Skill: WooCommerce Package Setup
**When to use:** Creating/editing the 3 service packages on blazextech.com/packages

### Package Tiers
```
Launch (Starter)
  Price: $299 one-time
  Delivery: 7 days
  Includes: 5-page WordPress site, mobile responsive,
            contact form, basic SEO

Growth (Popular)
  Price: $699 one-time + optional monthly retainer
  Delivery: 14 days
  Includes: WooCommerce/Shopify store, Meta ads setup,
            social media kit, 1 n8n automation workflow

Custom (Enterprise)
  Price: Contact for quote
  Delivery: Custom timeline
  Includes: Full ecommerce system, AI automation agents,
            ongoing ad management, monthly retainer, dedicated support
```

### WooCommerce Product Setup Steps
1. WooCommerce → Products → Add New
2. Product type: Simple product
3. Price: as above (leave Custom at $0, use "Contact for price" plugin)
4. Description: paste package features as bullet list
5. Add to "Packages" category
6. Link to /contact page for Custom tier

---

## Skill: n8n Automation Workflows
**When to use:** Building any automation for BlazeX or client projects.

### Workflow: Lead Capture → WhatsApp Alert
```
Trigger: WPForms form submission
→ Node: Extract fields (name, email, service, budget)
→ Node: Format WhatsApp message
→ Node: Send via WhatsApp Business API or Twilio
→ Node: Save to Google Sheets (CRM)
```

### Workflow: New WooCommerce Order → Alert
```
Trigger: WooCommerce webhook (order.created)
→ Node: Extract order details
→ Node: Send WhatsApp notification
→ Node: Add to Google Sheets log
→ Node: Send thank-you email to client
```

### Workflow: Social Media Scheduler
```
Trigger: Google Sheets row added (content calendar)
→ Node: Read post content + image URL
→ Node: Post to Instagram via Meta Graph API
→ Node: Post to Facebook Page
→ Node: Log result back to Sheets
```

### n8n Environment
- Self-hosted n8n (or n8n.cloud free tier)
- Credentials needed: WhatsApp Business API, Meta Graph API,
  Google Sheets OAuth, WooCommerce REST API key

---

## Skill: ComfyUI AI Image Generation
**When to use:** Generating product photos, Meta ad creatives, model photos for Ete.

### Prompts for Ete Clothing Brand
```
Model photo:
"professional fashion model wearing [GARMENT DESCRIPTION],
studio photography, clean white background, soft lighting,
editorial style, high resolution, photorealistic"

Meta Ad Creative:
"minimalist fashion advertisement, [GARMENT] on model,
bold typography space on left, warm lighting,
Instagram square format, luxury brand aesthetic"

Product flat lay:
"overhead flat lay of [GARMENT], minimal styling,
white marble background, fashion editorial, clean"
```

### Output Settings
- Format: PNG, 1024×1024 (square for Instagram)
- For ads: 1080×1080 or 1080×1920 (stories/reels)
- Always export with transparent bg where possible

---

## Skill: Meta Ads Setup
**When to use:** Creating/managing campaigns for BlazeX clients.

### Campaign Structure (Ete Clothing)
```
Campaign: Ete — [Month] Sales
  Ad Set 1: Pakistan — Women 18-35 — Fashion interests
    Ad 1A: AI model photo + "Shop Now"
    Ad 1B: Reel/video creative
  Ad Set 2: Retargeting — Website visitors 30 days
    Ad 2A: Carousel of products
```

### Copy Formula
```
Hook:    [Bold claim or question — 1 line]
Body:    [Problem → Solution — 2-3 lines]
CTA:     [Single clear action]

Example:
"Your wardrobe upgrade is here. 🔥
Ete's new collection just dropped — limited pieces,
made for women who move fast.
→ Shop before it sells out"
```

---

## Skill: SEO (Rank Math)
**When to use:** Setting up or optimizing any page on blazextech.com.

### Page-by-page Target Keywords
```
Home:      "web development agency Pakistan", "digital marketing Multan"
Services:  "WordPress developer Pakistan", "Shopify store setup Pakistan"
Packages:  "web design packages Pakistan", "affordable web development"
Portfolio: "WooCommerce store Multan", "ecommerce case study"
About:     "BlazeX Tech", "freelance web developer Multan"
Contact:   "hire web developer Pakistan", "digital agency contact"
```

### Rank Math Setup Checklist
- [ ] Set focus keyword per page
- [ ] Write meta title (55–60 chars)
- [ ] Write meta description (150–160 chars)
- [ ] Enable XML sitemap
- [ ] Submit sitemap to Google Search Console
- [ ] Add schema: LocalBusiness for blazextech.com

---

## Skill: Git Workflow for This Project
**When to use:** Saving/versioning any code or config changes.

```bash
# Daily workflow
git add .
git commit -m "feat: [what you built]"
git push origin main

# Branch for new features
git checkout -b feature/homepage-elementor
# ... make changes ...
git checkout main
git merge feature/homepage-elementor
```

### Commit Message Format
```
feat:  new feature or page
fix:   bug fix
style: color/font/design change
auto:  n8n workflow update
docs:  documentation update
```
