# LinkFlow — Hero Section

A standalone Vite + React + TypeScript implementation of the LinkFlow landing-page
hero, featuring a seamless boomerang video background.

## Stack

- Vite + React 18 + TypeScript
- Tailwind CSS 3.4
- lucide-react for icons
- CSS transitions only (no animation library)

## Getting started

```bash
npm install
npm run dev      # start the dev server
npm run build    # type-check + production build
npm run preview  # preview the production build
```

## Structure

- `src/App.tsx` — the full hero section (nav, mobile drawer, hero copy, CTA blocks).
- `src/BoomerangVideoBg.tsx` — captures video frames into a canvas, then plays them
  forward/backward in a seamless boomerang loop at 30fps.

The web font links and the root font stack live in `index.html` and `src/index.css`.
