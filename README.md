# Human Intelligence — UI

A single-file landing page for *Nexa*, built as a pixel-faithful implementation of a
design prototype. The entire experience — markup, styles, artwork and motion — lives in
`index.html` with no build step and no dependencies.

## Running it

Open `index.html` in a browser, or serve the folder:

```bash
python -m http.server 8000
```

Then visit <http://127.0.0.1:8000/index.html>.

## What's in the page

- **Fixed full-viewport scene.** A `fit()` routine measures the viewport and drives
  `--scene-scale`, `--nav-scale` and `--copy-scale` so the composition holds its
  proportions across sizes rather than reflowing.
- **Glass navigation** — logo, link pill and a Request Access control, collapsing to a
  toggle menu on narrow and portrait viewports.
- **Glass arc and rim** — the frosted foreground surface, built entirely from layered
  gradients, `clip-path` and `backdrop-filter`.
- **Hero copy** — headline, supporting paragraph and two calls to action.
- The background portrait is embedded directly in the document, so the page is fully
  self-contained and renders offline.

## Entrance animation

One choreographed sequence runs on first load and then stops for good. It uses the Web
Animations API — a single coordinated timeline, no animation library.

| Time | Element | Behaviour |
|------|---------|-----------|
| 0.00s | Portrait, visor bloom | Static — the scene is the stage |
| 0.05s | Glass arc + lit rim | The surface sweeps its clip edge up into place |
| 0.26–0.39s | Logo → nav pill → Request Access | Descend from their top anchor |
| 0.42 / 0.54s | Headline lines | Focus-pull: blur resolves to sharp, small rise |
| 0.80s | Paragraph | Same focus-pull at supporting scale |
| 0.96 / 1.03s | Explore Nexa → Discover More | Rise from their baseline |
| ~1.69s | — | Timeline cancels itself; the page is static |

Three motion behaviours only — surface reveal, focus settle, quiet lift — with
directions taken from each element's layout anchor. The blur is motivated by the subject:
the scene is a visor, so type resolving from soft to sharp reads as an optic focusing.

Notes on the implementation:

- The glass arc is revealed by animating its own `clip-path`, never its opacity. An
  element carrying `backdrop-filter` composites as a group below opacity 1, which makes
  it flash its unfiltered backdrop — a dark smear across the lower half. Moving only the
  clip edge keeps the filter fully applied, and leaves the portrait behind it perfectly
  still. The rim rides the same offset via `--arc-lift` so the lit edge stays welded to
  the arc, and both clip values are read back from the cascade rather than duplicated in
  JS, so the authored CSS stays the single source of truth at every breakpoint.
- Animations target the standalone `translate` and `filter` properties, never
  `transform`, so the authored transforms on `.hero-copy`, `.navigation` and
  `.hero-accent` — including the JS-driven scale variables — are left untouched.
- Every closing keyframe equals the element's authored CSS value, so cancelling the
  timeline restores the original design exactly. Verified against an un-animated render:
  5 differing pixels out of 1.14M, all sub-perceptual antialiasing.
- Nothing animates afterwards. No scroll triggers, hover motion, parallax or loops.
- `prefers-reduced-motion: reduce` skips the sequence entirely and renders the final
  state immediately. A failsafe timeout guarantees content is never left hidden.

## Repository layout

```
index.html    the entire site
dev/          local reference renders, comparison screenshots and measurement
              passes from the build process (git-ignored)
```
