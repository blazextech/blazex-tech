# BlazeX Tech — Connectors & Integrations

## Status Legend
- ✅ Ready to use
- 🔧 Needs setup
- 📋 Planned

---

## WordPress / WooCommerce
**Status:** 🔧 Needs setup after site install

### REST API (for n8n automation)
```
Base URL:     https://blazextech.com/wp-json/wp/v2/
WC Base URL:  https://blazextech.com/wp-json/wc/v3/

Setup:
1. WooCommerce → Settings → Advanced → REST API
2. Add Key → Read/Write permissions
3. Save Consumer Key + Consumer Secret
4. Add to n8n as WooCommerce credentials
```

### WP CLI (for Claude Code terminal use)
```bash
# Install WP-CLI on your hosting server
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

# Useful commands
wp plugin list
wp theme status
wp post list
wp option get siteurl
```

---

## Meta / Facebook
**Status:** 🔧 Needs setup

### Meta Graph API (for social posting + ad management)
```
1. Go to: developers.facebook.com
2. Create App → Business type
3. Add products: Instagram Graph API + Marketing API
4. Generate Page Access Token (long-lived)
5. Permissions needed:
   - pages_manage_posts
   - instagram_basic
   - instagram_content_publish
   - ads_management
   - ads_read

Store in n8n credentials as: HTTP Header Auth
Header: Authorization: Bearer YOUR_PAGE_ACCESS_TOKEN
```

### Meta Ads Manager API
```
Ad Account ID:  act_XXXXXXXXXX  (find in Ads Manager URL)
App ID:         from developers.facebook.com
App Secret:     from developers.facebook.com
Access Token:   long-lived page token
```

---

## WhatsApp Business API
**Status:** 🔧 Needs setup (for client notifications + CTA)

### Option A: WhatsApp Cloud API (free via Meta)
```
1. developers.facebook.com → Add product: WhatsApp
2. Get Phone Number ID + Access Token
3. Use in n8n: HTTP Request node
   POST https://graph.facebook.com/v17.0/{PHONE_NUMBER_ID}/messages
   Headers: Authorization: Bearer {TOKEN}
   Body: {
     "messaging_product": "whatsapp",
     "to": "923XXXXXXXXX",
     "type": "text",
     "text": { "body": "New lead: {{name}}" }
   }
```

### Option B: Twilio (easier, small cost)
```
Sign up: twilio.com
Get: Account SID + Auth Token + WhatsApp number
Use Twilio node in n8n directly
Cost: ~$0.005 per message
```

---

## Google Services
**Status:** 🔧 Needs setup

### Google Sheets (CRM + content calendar)
```
1. console.cloud.google.com → Create project "BlazeX"
2. Enable: Google Sheets API + Google Drive API
3. Create Service Account → Download JSON key
4. Share your Sheets with service account email
5. Add JSON key to n8n Google Sheets credentials

Sheets to create:
- BlazeX CRM (leads, clients, status)
- Content Calendar (Ete social posts)
- Order Log (WooCommerce orders)
- Monthly Report
```

### Google Search Console
```
1. search.google.com/search-console
2. Add property: blazextech.com
3. Verify via HTML file or DNS TXT record
4. Submit sitemap: blazextech.com/sitemap_index.xml
   (Rank Math generates this automatically)
```

### Google Analytics 4
```
1. analytics.google.com → Create property
2. Get Measurement ID: G-XXXXXXXXXX
3. Add to WordPress via Rank Math or site header
```

---

## n8n
**Status:** 🔧 Needs setup

### Self-hosted (recommended — your own server)
```bash
# Install via Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Access at: http://localhost:5678
# For production: set up on VPS with domain n8n.blazextech.com
```

### n8n.cloud (easiest start)
```
URL: n8n.io/cloud
Free trial: 14 days
Paid: ~$20/month (worth it for reliability)
```

### Credentials to add in n8n
- [ ] WooCommerce (key + secret)
- [ ] WhatsApp Cloud API (token)
- [ ] Google Sheets (service account)
- [ ] Meta Graph API (token)
- [ ] Gmail / SMTP (for email sequences)

---

## ComfyUI
**Status:** ✅ Already running

### API for n8n integration
```
Base URL: http://localhost:8188 (or your ComfyUI server URL)

n8n integration:
- Use HTTP Request node
- POST /prompt with your workflow JSON
- Poll GET /history/{prompt_id} for results
- Download image from /view?filename=...

This lets you trigger AI image generation
from n8n workflows automatically.
```

---

## GitHub
**Status:** 🔧 Needs setup

### Repo structure
```
github.com/YOUR_USERNAME/blazex-tech-website
├── /wp-content/themes/blazex-child/   ← custom CSS/PHP
├── /wp-content/plugins/blazex-custom/ ← custom functionality
├── /n8n-workflows/                    ← exported .json files
├── /comfyui-workflows/                ← ComfyUI workflow JSONs
└── /docs/                             ← project docs
```

### Setup
```bash
cd /path/to/BlazeX
git init
git remote add origin git@github.com:YOUR_USERNAME/blazex-tech-website.git
git add .
git commit -m "feat: initial BlazeX Tech project setup"
git push -u origin main
```

---

## Hosting / Server
**Status:** 🔧 Confirm details

### Recommended for blazextech.com
```
Current: Domain purchased (blazextech.com)
Hosting needed: Shared or VPS

Recommended hosts (Pakistan-friendly, fast):
- Cloudways (DigitalOcean) — $14/mo — best performance
- SiteGround — $6.99/mo — easy WordPress
- Namecheap EasyWP — $3.88/mo — cheapest

For n8n self-host: Contabo VPS — ~$5/mo (Germany/USA)
```

---

## Quick Setup Priority Order
```
1. ✅ Domain: blazextech.com (done)
2. 🔧 Hosting: Connect domain to host, install WordPress
3. 🔧 WordPress: Install Blocksy + import Codespot
4. 🔧 WooCommerce: Set up 3 packages
5. 🔧 Google Search Console: Verify site
6. 🔧 n8n: Set up on cloud or VPS
7. 🔧 WhatsApp API: Connect to n8n
8. 🔧 Meta API: Connect for Ete social posting
9. 🔧 GitHub: Version control everything
10. 🔧 ComfyUI → n8n: Automate image generation
```
