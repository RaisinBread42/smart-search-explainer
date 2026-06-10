# Prompt — Add scalable semantic search (embedding API + vector database)

> Paste everything below the line into Claude Code (or any capable coding agent),
> inside the repo you want to add search to. It will interview you first, then
> integrate production-grade semantic search into your **existing** project using a
> hosted embedding model and a vector database.
>
> Use this when the in-browser approach won't fit — large catalogs (~50k+ items),
> a shared index across users/devices, heavy metadata filtering, or server-side search.

---

I want to add **semantic search** to my project — matching by **meaning**, not just keywords
(so "cheap couch" finds a "budget-friendly sofa") — built to **scale**. Use a **hosted
embedding model** (e.g. OpenAI, Cohere, Voyage) plus a **vector database** (e.g. pgvector,
Pinecone, Qdrant, Weaviate). API keys live **only on the server** — never in the browser.

## STOP — interview me before writing any code
This must plug into my **existing** codebase and infrastructure, so do not assume anything
and do not scaffold a new app unless I ask for one. First, **inspect the repo** (framework,
backend/serverless setup, where data lives, any current search UI, existing DB) to answer
what you can yourself. Then **ask me the questions below that you could not confidently
answer from the code, and WAIT for my replies.** Do not generate the implementation until
these are settled. Propose a sensible default for each so I can just say "yes," and ask
follow-ups if my answers are vague.

**1. Data — what am I searching?**
   - Where do the records live (DB table, CMS, API, files)? What's the shape of one record?
   - Which fields get embedded (searched) vs. which are just returned for display/filtering?
   - How many records now, and what's the realistic ceiling? (drives DB + cost choices)
   - How often do records change — and how should the index stay in sync (on write / webhook
     / cron / manual reindex)?

**2. Search UI — is there one already?**
   - Existing search box / input to wire into? Where (file/component)?
   - Where should results render — an existing list/component, or somewhere new?
   - Replace my current search, augment it, or run alongside? Do I need **metadata filters**
     (category, price, in-stock) combined with the semantic query?

**3. Stack & infrastructure (this needs a server for the keys)**
   - Framework + version, TypeScript or not?
   - Do I already have a backend or serverless layer I should use (Next.js route handlers,
     Express/Nest, Cloud/Edge Functions, Supabase)? If not, where can one live, and where do
     I host (Vercel/Netlify/AWS/etc.)?
   - Where do secrets go (`.env`, platform secret store)? Confirm keys must stay server-side.

**4. Providers — pick or let me recommend**
   - **Embedding model:** do I have an account/preference (OpenAI `text-embedding-3-small` is
     a strong, cheap default)? Note: the **same** model must embed both documents and queries.
   - **Vector store:** already on Postgres? → **pgvector** is the low-friction default. No DB
     yet / want managed? → **Pinecone** or **Qdrant**. Any data-residency, region, or budget
     constraints?

**5. Behavior & expectations**
   - Top-K to return (default 10)? Need **hybrid** (keyword + vector) ranking? Acceptable
     query latency and rough budget? Auth/rate-limiting needs on the search endpoint?

Once I answer, restate the plan in 3–4 lines (provider, store, where the endpoint lives, how
indexing stays in sync) and proceed.

## How to build it (apply to whatever the answers are)
Build two clearly separated pieces, plus a thin client:

1. **Ingestion / indexing** (server-side, repeatable & idempotent)
   - Batch the records, embed their searchable text via the embedding API (respect rate
     limits / batch sizes), and **upsert** vectors keyed by record id into the vector store,
     storing the display fields + filterable metadata alongside each vector.
   - Make it re-runnable for a full backfill, and wire incremental updates per my sync answer
     (on-write hook, cron, or a `reindex` script). Record the model name/dimension used.

2. **Query endpoint** (server-side; keys never reach the browser)
   - Accept `{ query, filters?, topK? }`, embed the query with the **same** model, run
     nearest-neighbour search in the vector store (with metadata filters applied), and return
     the ranked records (id, display fields, score).
   - Add basic input validation and (if I asked) auth/rate-limiting.

3. **Client integration**
   - Wire my existing search input to call the endpoint (debounced) and render results into my
     existing results component — don't duplicate UI. Show a loading state; handle empty/error.
   - If hybrid is wanted, fuse keyword + vector scores (e.g. weighted, or reciprocal-rank
     fusion) rather than discarding the keyword search that already works.

## Fit my codebase and infra, don't fight it
- Match my framework, file layout, naming, styling, and TypeScript usage. Reuse my existing
  search/results components and, if I have one, my existing DB (prefer pgvector on it before
  adding a new managed service).
- Put **all** provider calls behind one small adapter (embed + upsert + query) so the model
  or vector store can be swapped without touching the rest. Keep secrets in env, never in
  client code or the repo. Add migrations if using pgvector.
- Don't break my build. Keep new dependencies minimal and call out each one, plus any new
  env vars, accounts, or infra I'll need to provision.

## Only scaffold a standalone example IF I ask for it
If I say I have no existing project / just want a runnable sample, then (and only then)
produce a minimal end-to-end example: an ingestion script, one query endpoint, and a tiny
client, with `.env.example` and clear setup steps — but still keep keys server-side.

## Success criteria
- Integrated into my existing search/results surface (or a new one only if I asked), with
  metadata filtering if I needed it.
- A repeatable ingestion path and a working server-side query endpoint; **no keys in the
  browser**.
- Semantic clearly beats literal matching on my data, and the design scales to my record
  ceiling. Provider/store choices are isolated behind one swappable adapter.
