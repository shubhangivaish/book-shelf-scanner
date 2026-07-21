# Shelf Scan

📚 Point a camera at a bookshelf. Get every book identified, catalogued, and enriched — automatically.

<img width="903" height="409" alt="image" src="https://github.com/user-attachments/assets/2ebc97d5-f20a-4b1a-b50a-e0e384df82ae" />

<img width="907" height="407" alt="image" src="https://github.com/user-attachments/assets/80bc06fa-d945-40ce-9958-54739205f810" />

<img width="889" height="394" alt="image" src="https://github.com/user-attachments/assets/959ba153-2dd4-476b-9047-fbb57d457a7f" />

## What it does

Shelf Scan reads a photo of a bookshelf and returns a structured catalogue of every book it can identify:

- **Title, author, and genre** — read directly from the spine using Claude's vision
- **Confidence flagging** — each book is marked high / medium / low confidence based on how legible the spine was, with an inline correction flow for anything that needs a human check
- **Catalogue enrichment** — publisher, ISBN, page count, edition count, and a summary, pulled from the free, keyless Open Library API
- **Non-bookshelf detection** — tells you clearly (and a little playfully) if you've photographed a chair, a clothes rack, or anything that isn't a bookshelf, instead of failing silently
- **In-browser cropping** — frame the shelf before scanning, which measurably improves read accuracy
- **Camera or gallery capture** — works from a live photo or an existing one

## How it works

Shelf Scan is a single self-contained HTML/CSS/JS file — no build step, no framework, no server of its own. It runs as a Claude "AI-powered artifact": the page calls Claude's vision API directly from the browser, authenticated through the viewer's own Claude account, so no API key ever lives in the code.

```
Photo → crop → Claude vision (classify scene + read spines) → Open Library lookup (enrich) → render
```

The vision call is handled through Claude's own artifact infrastructure. The Open Library enrichment calls are plain public API requests and currently only succeed when this file is hosted **outside** Claude's sandboxed artifact environment — see Limitations below.

## Tech stack

- Vanilla HTML / CSS / JS
- [Cropper.js](https://github.com/fengyuanchen/cropperjs) for in-browser image cropping
- Claude (Sonnet) vision, via the Anthropic Messages API
- [Open Library](https://openlibrary.org) (an Internet Archive project) for publisher, ISBN, page count, edition count, and summaries — free and keyless, no Wikipedia or Amazon/Goodreads dependency

## Running this project

The simplest way: open it inside [Claude](https://claude.ai) as an artifact, logged into your account. Because the vision call authenticates through your Claude session, opening the raw HTML file on its own (downloaded, via `file://`, or hosted with no backend) lets you take and crop a photo, but the scan step won't be able to reach Claude's API.

To run it fully standalone, outside Claude:
1. Get an [Anthropic API key](https://console.anthropic.com) (separate from a claude.ai account — pay-as-you-go, new accounts get a small free trial credit).
2. Add a small backend or serverless proxy (e.g. Cloudflare Workers) to hold that key securely and relay the vision request. **Never ship an API key in client-side code** — anyone could read it from the page source.
3. Point the `fetch` call in `index.html` at your proxy instead of directly at `api.anthropic.com`.

Once hosted with a real backend, the Open Library enrichment — already fully built — should start working too, since the page will no longer be inside Claude's restricted artifact sandbox.

## Known limitations

- **No live ratings.** Neither Amazon nor Goodreads offer a free public API.
- **Open Library enrichment is currently dormant inside Claude's artifact sandbox.** The code is complete and tested; it just can't reach `openlibrary.org` from inside Claude's environment, since only Claude's own API is reachable from there.
- **Very large shelves** (25+ books in one photo) may hit the response length limit on a single scan — splitting into two closer photos reads more reliably.
- **Spine legibility drives confidence, by design.** Angled, tightly packed, or poorly lit shelves will produce more "needs a check" results — that's the confidence system doing its job, not a bug.

## Roadmap

- [ ] Backend proxy for standalone hosting (unlocks Open Library enrichment and removes the Claude-login requirement)
- [ ] Google Books API as a fallback source for ratings
- [ ] Export the catalogue as CSV / JSON

---
Copyright

© 2026 Shubhangi Vaish. All rights reserved.
This project — including its code, design, and written content — may not be copied, reused, or redistributed without permission.


Built with Claude, including the vision recognition it runs on.
