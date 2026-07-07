---
name: verify
description: How to verify changes to this static HTML site (GK Groupe inc) end-to-end in a real browser.
---

# Verifying the GK Groupe site

This project is a single static file (`index.html`, no build step, no dev server,
no git repo). To verify changes, drive it in a real headless browser rather than
just reading the HTML/JS.

## Setup (one-time per machine)

Playwright is not a project dependency (no `package.json` at repo root). Install it
in a scratch directory instead of polluting this repo:

```bash
mkdir -p /path/to/scratch/pw && cd /path/to/scratch/pw
npm init -y && npm install playwright@latest
npx playwright install chromium
```

This downloads ~300MB (Chromium + headless shell) and takes a few minutes the first time.

## Driving it

Load the file directly via `file://` — no server needed:

```js
const { chromium } = require('/path/to/scratch/pw/node_modules/playwright');
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('file:///c:/Users/karim/Desktop/GK GROUPE inc/GK - site/index.html');
```

Useful checks for this site specifically:
- Partner logos: `page.locator('.partner-logo img')` — count and check `naturalWidth > 0` per image (broken `src` loads a 0x0 image, doesn't throw).
- Pack section: `#packs .pack-card` count, `.pack-card.featured` for the highlighted tier.
- Chatbot: click `#chatbot-btn`, wait for `#chatbot-bubble.open`, then click choice buttons in `#chat-choices` (they're rewritten via `innerHTML` on every step, so re-query after each click).
- Mobile nav: `page.setViewportSize({width:375,height:800})`, click `.burger`, check `#mobileNavOverlay.open`.
- Modal prefill: click a `.pack-btn`, check `#modal-soumission.open` and read `select[name=service]`/`textarea[name=message]` values.
- Collect `page.on('console', ...)` and `page.on('pageerror', ...)` — this file has inline `onclick=` handlers, a typo breaks silently otherwise.

## Gotchas

- The file name has a space (`logo ford.png`) — fine for local `file://` `<img src>`, just don't URL-encode it in the HTML.
- No server means relative image paths resolve relative to the HTML file's own directory — screenshot problems here usually mean a missing/renamed file, not a path bug.
