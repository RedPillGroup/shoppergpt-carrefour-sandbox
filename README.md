# shopper-gpt-carrefour-sandbox

A minimal fake OpenMage 20.13.0 environment for testing the ShopperGPT widget locally. No server required — just open `index.html` in a browser.

## What it does

- Renders a realistic Carrefour Traiteur product listing page
- Fires all 4 DOM events on page load / user actions
- Listens for widget events and reacts (updates cart count, shows toast, logs to console)
- Includes a store selector modal that re-fires `shoppergpt:page_context`
- Shows a live event log at the bottom of the page for debugging

## Setup & usage

### 1. Widget script (GitHub CDN)

`chat.html` loads the bundle from [shoppergpt-carrefour-scriptag](https://github.com/selim-redpill/shoppergpt-carrefour-scriptag) via [jsDelivr](https://cdn.jsdelivr.net/gh/selim-redpill/shoppergpt-carrefour-scriptag@main/dist/agent.js):

```html
<script
  src="https://cdn.jsdelivr.net/gh/selim-redpill/shoppergpt-carrefour-scriptag@main/dist/agent.js"
  defer
></script>
```

Ensure `dist/agent.js` is committed on `main` in that repo (run `npm run build` locally, then push). For local widget development, swap the `src` to `../shopper-gpt-carrefour-scriptag/dist/agent.js` (see comment in `chat.html`).

### 2. Open the sandbox

```bash
# Option A — just open in browser
open index.html

# Option B — simple local server (avoids any CORS quirks)
npx serve .
# or
python3 -m http.server 8080
```

The sandbox loads the widget from the GitHub CDN (same URL pattern as OpenMage ScriptTag integration).

## DOM Events flow

```
Page loads
  └─ setTimeout 800ms
       ├─ fires shoppergpt:session       { session_id: "sess_xxxx" }
       └─ fires shoppergpt:page_context  { store_id: "42", store_name: "Paris République" }

User clicks store selector
  └─ fires shoppergpt:page_context  { store_id, store_name }

Widget adds product to cart
  └─ fires shoppergpt:cart_updated  { success: true, product_id, action: "add" }
       └─ sandbox updates cart badge + page product button + shows toast

Widget dispatches change_shop
  └─ fires shoppergpt:change_shop   { store_id }
       └─ sandbox updates navbar store label
```

## Event log

The dark panel at the bottom of the page shows all events in real time:
- `OUT → widget` = events fired by the sandbox toward the widget
- `IN ← widget` = events received from the widget

## File structure

```
shopper-gpt-carrefour-sandbox/
└── index.html    # Everything in one self-contained file
```

## Relationship to the scriptag repo

```
Desktop/Carrefour/
├── shopper-gpt-carrefour-scriptag/   ← source + build → push dist/ to GitHub
│   └── dist/agent.js
└── shopper-gpt-carrefour-sandbox/
    └── chat.html                     ← script tag → jsDelivr @ main
```
