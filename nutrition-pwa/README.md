# Field Log — personal nutrition tracker (PWA)

Photo or text → AI nutrition estimate (calories, macros, 10 micros) → stored on-device. Installable on your phone's home screen.

## Files

- `index.html` — the entire app (vanilla JS, no build step, no dependencies)
- `manifest.json` — makes it installable
- `sw.js` — service worker, caches the app shell so it opens offline (analysis still needs internet)
- `icon-192.png`, `icon-512.png` — app icons

## Deploy (5 minutes, GitHub Pages)

```bash
# new repo, e.g. nutrition-log
git init && git add . && git commit -m "field log"
git remote add origin git@github.com:Rohan473/nutrition-log.git
git push -u origin main
```

Repo → Settings → Pages → Source: `main` branch, root. App lives at
`https://rohan473.github.io/nutrition-log/`.

HTTPS is required for camera access and the service worker — GitHub Pages gives you that for free. Opening `index.html` as a local file will NOT work for camera/PWA features.

## Install on phone

Open the URL in Chrome (Android) → menu → **Add to Home screen** → it runs full-screen like a native app.

## API key (required for AI analysis)

1. console.anthropic.com → API Keys → create one
2. **Set a monthly spend limit** (₹200–400 is plenty) — the key lives in your phone's localStorage, so cap the blast radius
3. Open the app → ⚙ Settings → paste key → Save

Cost: Haiku 4.5 (default) ≈ ₹0.3–0.6 per meal analysis. ~3 meals/day ≈ ₹40–60/month. Sonnet 4.6 is 3–4× that with somewhat better portion reasoning.

## Data

- Everything stays in your phone's localStorage under `nt:` keys — nothing leaves the device except the photo/description sent to the Anthropic API during analysis (photos are not stored)
- Trends tab → **Export all data (JSON)** for backup or analysis in pandas
- Clearing browser data for the site wipes the log — export periodically

## Honest limits

- AI calorie estimates run ±30–50% off, micros worse. Every estimate is editable before saving — typing "2 rotis, dal, 1 bowl rice" beats any photo
- localStorage is per-browser, per-device. No sync. Export/import is your backup strategy
- If you later want sync or a proper DB, the storage layer is isolated in the `store` object in index.html — swap it for a FastAPI + SQLite backend in an afternoon
