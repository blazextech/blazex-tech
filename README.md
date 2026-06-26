# BlazeX Tech — Project Repository

> Full-stack digital agency website + automation stack
> blazextech.com | Multan, Pakistan

## Quick Start for Claude Code

```bash
# Open this folder in Claude Code
# Claude will auto-read CLAUDE.md on startup

# First thing to do:
cat CLAUDE.md   # read project context
cat SKILL.md    # read available skills/patterns
cat docs/ROADMAP.md  # see what's next
```

## Folder Structure

```
BlazeX/
├── CLAUDE.md                    ← Claude Code memory (READ FIRST)
├── SKILL.md                     ← Reusable patterns & code snippets
├── README.md                    ← This file
│
├── .claude/
│   ├── mcp.json                 ← MCP server config (GitHub, Sheets, etc.)
│   └── settings.json            ← Claude Code permissions + behavior
│
├── docs/
│   ├── ROADMAP.md               ← Project phases + task tracker
│   └── CONNECTORS.md            ← All API integrations + setup guides
│
├── src/
│   ├── theme/
│   │   └── style.css            ← BlazeX child theme CSS
│   ├── plugins/                 ← Custom WP plugin code (future)
│   └── automation/
│       └── lead-to-whatsapp.json  ← n8n workflow: lead → WhatsApp
│
└── assets/
    ├── images/                  ← Hero images, screenshots
    ├── logos/                   ← BlazeX logo variants (PNG, SVG)
    └── fonts/                   ← Inter font files (if self-hosting)
```

## Key Links

| Resource | URL |
|----------|-----|
| Website | https://blazextech.com |
| WordPress Admin | https://blazextech.com/wp-admin |
| n8n Dashboard | http://localhost:5678 (or n8n.cloud) |
| ComfyUI | http://localhost:8188 |
| Meta Ads Manager | https://adsmanager.facebook.com |
| Google Search Console | https://search.google.com/search-console |

## MCP Servers Setup

```bash
# Install MCP servers (run once)
npm install -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-github
npm install -g @modelcontextprotocol/server-brave-search

# Add your API keys to .claude/mcp.json
# Then restart Claude Code
```

## Active Client Projects

| Client | Service | Status |
|--------|---------|--------|
| Ete Clothing | WooCommerce + Meta Ads + ComfyUI | 🟢 Active |

## Tech Stack

- **WordPress** + Blocksy theme + Elementor
- **WooCommerce** for packages/payments
- **n8n** for automation
- **ComfyUI** for AI image generation
- **Meta Ads Manager** for paid campaigns
- **Google Sheets** as lightweight CRM
