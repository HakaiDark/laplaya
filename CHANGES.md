# La Playa Blanca — Session Changes

Work done on top of the design handoff, taking the prototype from a reference
bundle to a deployed, mobile-friendly, performance-tuned live site.

Entry point was renamed `La Playa Blanca.dc.html` → **`index.html`** so it serves
at the site root (Netlify/any static host).

---

## 1. Video playback fix (the "frozen videos" bug)
The re-encoded MP4s had their `moov` atom at the **end** of the file (HandBrake's
"Web Optimized" box was unchecked), so the WebGL `VideoTexture`s showed a frozen
first frame over HTTP.
- Relocated `moov` to the front (**faststart**) — lossless remux, no re-encode.
- Made the shader animation clock **framerate-independent** (real elapsed time
  instead of a fixed per-frame step) so wave/ambient motion is smooth on any
  refresh rate.
- Added `?v=N` cache-buster on video URLs so replaced files aren't served stale.

## 2. WhatsApp checkout
The **Checkout** button now opens WhatsApp pre-filled with the order.
- Routes to **+961 71 245 911** (`wa.me/96171245911`).
- **Bilingual** message (Arabic / English).
- Handles **any number of items with quantities**; shows per-line price (price ×
  qty), a Subtotal, and marks BOBA / Ice Cream as priced in-store.
- Code: `_orderText()` + `_waCheckout()` in `index.html`.

## 3. Mobile responsiveness
- **Root cause of right-edge clipping:** `html` was `overflow-x: visible` while
  bleeding fixed/absolute art (giant chapter titles, glows) widened the mobile
  viewport to ~417px. Fixed with `overflow-x: clip` on `html`+`body` (clip, not
  hidden, so vertical scroll, `position:sticky`, and Lenis keep working). Viewport
  is now exactly device-width.
- **Nav:** compact padding + smaller Cart button on phones.
- **Fan menu:** category pills are non-sticky and compacted on mobile (they were
  covering card names when sticky); fan deck shortened to remove a big gap.
- Verified 292 / 320 / 375 / 390 / 430 px — no horizontal overflow; desktop
  unchanged.

## 4. Performance (slow-connection tuning)
- **Hero images PNG → JPEG (q80):** `hero-bg-beach` + `hero-poster-crepe`,
  4.1 MB → 0.7 MB (~83%). Old PNGs removed.
- **Deferred video loading:** crêpe hero eager; waffle + smoothie
  `preload="metadata"` so they stream in on scroll instead of all downloading up
  front.
- **Re-encoded videos to 720p** (~4 MB → ~2 MB each), faststart verified.
- **First-load weight: ~15.5 MB → ~2.8 MB (~82% reduction).** Total assets 16 MB → 6.8 MB.

## 5. Mobile video fit + backdrop (the hero polish)
16:9 videos "cover"-filled a tall phone to a zoomed centre crop (food looked
oversized). Now:
- New `uFit` shader uniform eases **cover → contain** based on viewport aspect
  (0 on desktop/landscape; ramps up as it gets tall/narrow — see `_fitValue()`).
- The empty space is filled with a **blurred, dimmed, teal-tinted version of the
  same video** (5-tap blur) instead of flat teal — reads as an intentional
  cinematic backdrop, not letterbox bars.
- **Feathered** sharp→blur boundary (no visible seam — reads as depth-of-field)
  and a **framing vignette** (blur darkens toward the screen edges).
- Guarded on `uFit` so desktop/landscape skips the extra sampling entirely.

## 6. Housekeeping
- Added `.gitignore` (`.DS_Store`, `node_modules/`, `*.orig`, editor dirs).

---

## Commits (this session, `main`)
```
170f206  Refine mobile hero: feathered depth-of-field + framing vignette
0b36d79  Fill mobile letterbox with a blurred video backdrop instead of flat teal
bcc9162  Mobile video fit: ease 16:9 hero from cover->contain on tall screens
785240f  Re-encode hero videos to 720p (~4MB -> 2MB each), bump cache-buster
8987bfb  Optimize for slow connections: JPEG heroes + deferred video preload
f1c125f  La Playa Blanca — immersive one-page site prototype (faststart videos,
         framerate-independent clock, bilingual WhatsApp checkout, mobile
         hardening, index.html rename)
```

## Still open / TODO
- **Push:** all 6 commits are **local only** — `git push` failed on GitHub auth
  in this environment. Push from your own Terminal (`git push -u origin main`) or
  after `gh auth login`. Not needed for drag-and-drop Netlify deploys.
- **Deploy:** re-upload the folder to Netlify (drag-and-drop) to publish today's
  changes.

## Tunable dials (if you want to revisit)
- Hero fit strength / dish size: cap + slope in `_fitValue()`.
- Blurred-backdrop brightness / tint: the `fill*=...` and teal-tint `mix` in the
  fragment shader's `if(uFit>0.001){ … }` block.
- Video playback speed: was reverted to original (1.0); no `playbackRate` set.
