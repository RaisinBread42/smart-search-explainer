# Smart Search Explainer

An interactive, single-file presentation that demonstrates the difference between
**keyword search** and **semantic search** on a sample marketplace dataset — fully
client-side, no APIs or build step.

Built by **Obsidian Software**.

## Files

| File | What it is |
|------|------------|
| `index.html` | The presentation. 7 navigable slides (React + Tailwind + Recharts via CDN). Open it in any browser. |
| `report.html` | Interactive critique & roadmap from a 5-persona expert review. |
| `CRITIQUE_AND_GOALS.md` | The same critique in markdown. |

## Run it

Just open the file — no install required:

```bash
open index.html
```

The libraries (React, Tailwind, Recharts) load from CDN, so the first open needs
internet; the search data and logic are entirely local.

## How it works

- **Keyword search:** `(bodyMatches + titleMatches×2) / (queryWords×2) × 100`, plus an exact-phrase bonus.
- **Semantic search:** deterministic, client-side **TF-IDF** vectors compared with **cosine similarity** — a lightweight proxy for true neural embeddings.
