# Prompt — Add in-browser semantic search to my project

> Paste everything below the line into Claude Code (or any capable coding agent),
> inside the repo you want to add search to. It will interview you first, then
> integrate semantic search into your **existing** project — no backend, no API
> keys, no database.

---

I want to add **semantic search** to my project — search that matches by **meaning**, not
just keywords (so "cheap couch" finds a "budget-friendly sofa"). Use a **real pretrained
sentence-embedding model running entirely in the browser** (Transformers.js with
`Xenova/all-MiniLM-L6-v2`): no API keys, no server, no vector database.

## STOP — interview me before writing any code
This must plug into my **existing** codebase, so do not assume anything and do not scaffold
a new page unless I ask for one. First, **inspect the repo** (framework, build tooling,
where data lives, any current search UI) to answer what you can on your own. Then **ask me
the questions below that you could not confidently answer from the code, and WAIT for my
replies.** Do not generate the implementation until these are settled. If my answers are
vague, ask follow-ups. Group the questions and propose a sensible default for each so I can
just say "yes."

**1. Data — what am I searching?**
   - Where do the records live (hardcoded array, local JSON/MDX, a REST/GraphQL API, a DB)?
   - What's the shape of one record, and which fields should be searched vs. just displayed?
   - Roughly how many records? Does the set change at runtime, or is it fixed at build time?

**2. Search UI — is there one already?**
   - Is there an existing search box / input I should wire into? Where (file/component)?
   - Where should results render — an existing results list/component, or somewhere new?
   - Should this **replace** my current search, **augment** it, or run **alongside** it?

**3. Stack & integration constraints**
   - Framework and version (React/Next/Vue/Svelte/Astro/vanilla/etc.), and TypeScript or not?
   - Is there a bundler (Vite/webpack/Next)? Can I add an npm dependency, or must it be a
     CDN/`<script>` include?
   - Server-rendered (Next/Astro/etc.)? The model must run client-side only — I need to know
     so I guard it correctly (`window`, dynamic import, on-mount).

**4. Runtime expectations**
   - OK for the browser to download the model once (~25MB, then cached)? Any hard offline or
     payload limits? Mobile / low-end devices a concern?

Once I answer, restate the plan in 2–3 lines and proceed.

## How to build it (apply to whatever the answers are)
- **Model:** `Xenova/all-MiniLM-L6-v2` (384-dim, ~25MB quantized) via **Transformers.js**.
  If I have a bundler → `npm i @huggingface/transformers` and import it. If not → load it
  from a CDN as an ES module. The model downloads from the Hugging Face CDN on first use and
  is cached by the browser thereafter.
- **Embeddings:** mean pooling + L2 normalization, so cosine similarity is a dot product.
- **Indexing strategy — pick based on my data answer:**
  - *Fixed / build-time data:* **pre-compute** the document embeddings once (a small script
    or build step) and ship them as a `.json` so users don't re-embed on every load. The
    model is still loaded in the browser to embed the **query** at search time.
  - *Dynamic / runtime data:* embed the records in the browser on load (or as they arrive),
    with a progress indicator; keep vectors in memory.
- **Search:** embed the query in-browser → cosine vs every document vector → sort → return
  top N (default 8). Keep it debounced and off the main interaction path so typing stays smooth.
- **Lazy-load the model** (don't block first paint); show loading/progress state and disable
  the search affordance until embeddings are ready. Use the pipeline's `progress_callback`.

## Fit my codebase, don't fight it
- Match my existing framework, file structure, naming, styling, and TypeScript usage. Reuse
  my current search input and results components if they exist — wire into them, don't
  duplicate them.
- Don't break my build or introduce console errors. Keep new dependencies minimal and call
  out anything you add. Isolate the embedding logic into one well-commented module/composable/
  hook so it's easy to find and swap the model later.
- If hybrid makes sense (I already have keyword search), offer to **fuse** keyword + semantic
  scores rather than replacing what works.

## Only build a standalone demo page IF I ask for it
If I say I have no existing project / just want a sample, then (and only then) produce a
single self-contained `index.html` (Transformers.js + Tailwind via CDN) with a sample
catalog, a search box, and ranked result cards with similarity scores.

## Success criteria
- Integrated into my existing search/results surface (or a new one only if I asked).
- After the one-time model download, queries return in well under a second; the UI never
  blocks without feedback.
- Semantic clearly beats literal matching on my data (synonyms / intent match).
- No API keys, no backend, no database. The embed → cosine → rank flow is clear and isolated.
