# Field Log — personal nutrition tracker (PWA)

Photo or text → AI nutrition estimate (calories, macros, 10 micros) → stored on-device in IndexedDB. Installable on your phone's home screen.

## v2 changes

- **Provider switch**: DeepSeek (default, `deepseek-v4-flash` — supports vision, very cheap) or Anthropic. Set in ⚙ Settings.
- **Database**: entries now live in **IndexedDB** (`fieldlog` DB, `days` store) instead of localStorage. Old localStorage entries auto-migrate on first load.
- **Base URL override** in Settings — escape hatch if DeepSeek blocks direct browser calls (see CORS section).

## API key security — read this

Your key is **never in this repo**. The repo being public is fine. The key is typed into Settings at runtime and stored only in your phone's localStorage.

**Never hardcode the key into index.html and push it.** Public repo + hardcoded key = stolen key within hours (bots scan GitHub for `sk-` patterns continuously). Rules:

1. Key goes in the app's Settings screen only
2. Set a monthly spend limit on the key at the provider dashboard
3. If you ever accidentally commit a key, revoke it immediately — deleting the commit does not help, it's already scraped

## CORS caveat (DeepSeek)

Anthropic explicitly supports direct browser calls. DeepSeek's API is not documented to send CORS headers for browser-origin requests — it may work, it may not. If analysis fails with a "Request blocked — likely CORS" error:

Deploy this 10-line Cloudflare Worker (free tier) and put its URL in Settings → Base URL override:

```js
export default {
  async fetch(req) {
    const url = new URL(req.url);
    url.hostname = "api.deepseek.com";
    if (req.method === "OPTIONS")
      return new Response(null, { headers: cors() });
    const res = await fetch(new Request(url, req));
    const out = new Response(res.body, res);
    Object.entries(cors()).forEach(([k, v]) => out.headers.set(k, v));
    return out;
  }
};
const cors = () => ({
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "POST, OPTIONS",
  "Access-Control-Allow-Headers": "content-type, authorization"
});
```

(Tighten `Allow-Origin` to your GitHub Pages domain once it works.)

## Deploy

```bash
git add . && git commit -m "v2: deepseek + indexeddb" && git push
```

GitHub Pages serves it at `https://rohan473.github.io/Nutrition-tracker/`. HTTPS is required for camera + service worker. After pushing, hard-refresh once on the phone (the service worker is network-first, so updates land on next load).

## Install on phone

Chrome (Android) → open the URL → menu → **Add to Home screen**.

## Cost

DeepSeek V4 Flash: $0.14/M input, $0.28/M output — a meal analysis is a fraction of a paisa. Effectively free at personal scale. Anthropic Haiku 4.5 ≈ ₹0.3–0.6/meal if you switch providers.

## Data

- Entries: IndexedDB (`fieldlog` → `days`), one record per date
- Settings + API key: localStorage (`nt:settings`)
- Photos are never stored — only the extracted nutrition numbers
- Trends → **Export all data (JSON)** for backup / pandas. Clearing site data wipes both stores — export regularly
