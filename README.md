# Reel 02 — a scroll-driven film reel in one HTML file

A self-contained interactive page: vertical scroll is translated into a horizontal
camera pan across a filmstrip, with rack focus, plate parallax, an anamorphic lens
flare, film grain and a live countdown. No build step, no dependencies, no network
requests at runtime.

## At a glance

| | |
|---|---|
| **Stack** | Hand-written HTML, CSS, vanilla JavaScript (ES5 syntax, no transpiler) |
| **Files** | One — `index.html`, 2,730 lines, ≈ 2.4 MB |
| **Dependencies** | None, at build time or runtime |
| **Network requests** | Zero after the document loads |
| **Fonts** | 3 families / 6 faces, base64 WOFF2, inlined |
| **Images** | 31 base64 payloads (JPEG + WebP), inlined |
| **CSS** | 1,332 lines, 20 custom properties, 13 `@keyframes`, 7 width breakpoints |
| **JS** | 2 IIFEs, 549 lines, no globals leaked |

Because everything is inlined, the page runs offline, opens straight from `file://`,
survives being emailed as an attachment, and deploys to any static host by copying a
single file.

---

## Architecture

The document is one linear stream, deliberately kept in load order:

```
<head>            charset, viewport, theme-color, description
<style>           tokens -> layout -> components -> effects -> breakpoints
  #clap           opening slate, removed from the DOM after its animation
  .dolly          the tall scroll container that drives the pan
    .gate         sticky viewport-sized window ("the film gate")
      .track      horizontal flex strip
        .frame    x5 — each exactly 100vw
  .stills         x2 static frames, lifted out of the pan
  .extras         normal vertical page — gallery, guides, footer
  .lb / .lightbox two overlay viewers
  .hud            fixed viewfinder furniture
<script>          the reel
<script>          the extras
```

Two scripts rather than one, because they are independent: the second runs even if the
first throws, and neither touches the other's state.

## The pan

The trick is that nothing scrolls horizontally. A tall container provides scroll
distance, a sticky child stays fixed in the viewport, and scroll progress is mapped
onto a `translate3d` on the strip inside it.

```css
.dolly { height: calc(var(--frames) * 100svh); }   /* the scroll budget   */
.gate  { position: sticky; top: 0; height: 100svh; overflow: hidden; }
.track { display: flex; transform: translate3d(var(--panx, 0px), 0, 0); }
.frame { flex: 0 0 100vw; width: 100vw; }
```

```js
var top  = dolly.offsetTop;
var span = dolly.offsetHeight - vh;
var p    = span > 0 ? (window.scrollY - top) / span : 0;   // 0 .. 1
p = p < 0 ? 0 : p > 1 ? 1 : p;

track.style.setProperty("--panx", (-p * travel).toFixed(1) + "px");
```

`travel` is `(FRAMES - 1) * viewportWidth`, cached on resize rather than recomputed per
frame. Writing to a custom property instead of `style.transform` keeps the transform
declaration in the stylesheet, so the gate can compose its own independent jitter on
the same element.

### Rack focus and culling

Each frame's distance from the gate drives a blur and an opacity, so frames resolve as
they arrive rather than cutting in. Frames further than 1.3 screens away are pulled out
of the render path entirely:

```js
var head = p * (FRAMES - 1);          // which frame is in the gate, fractional
var d    = Math.abs(i - head);

if (d > 1.3) { f.style.visibility = "hidden"; continue; }

f.style.setProperty("--fb", (d * maxBlur).toFixed(2) + "px");   // blur()
f.style.setProperty("--fo", (1 - d * 0.72).toFixed(3));         // opacity
```

Background plates read the same value at a different rate to give parallax
(`--px`, ±7%). `maxBlur` scales with viewport width, clamped to 2.5–10px, so the effect
is proportional rather than absolute.

### Scheduling

The scroll listener is `{ passive: true }` and does no work itself — it sets a flag and
defers to `requestAnimationFrame`, so bursts of scroll events collapse into one layout
read per frame:

```js
window.addEventListener("scroll", function () {
  if (!panQueued) { panQueued = true; requestAnimationFrame(runPan); }
}, { passive: true });
```

Resize is debounced at 160ms and re-measures cached geometry.

## The effects layer

| Effect | Technique |
|---|---|
| **Film grain** | Inline SVG `feTurbulence` as a `data:` background, stepped through 5 positions |
| **Sprocket holes** | Repeating SVG background, offset by `calc(var(--panx) * 0.6)` so the strip travels with the pan at a different rate |
| **Anamorphic flare** | `<canvas>` + `requestAnimationFrame`; a horizontal streak, a warm core and four ghosts down the lens axis. Peaks at frame boundaries, which is what hides the cut |
| **Gate weave** | Sub-pixel jitter on the gate, repainted at ~12fps to mimic film not sitting still |
| **Print scratches** | Two 1px gradient columns, randomly repositioned and faded |
| **Chromatic aberration** | Two `::before`/`::after` copies of the title in red and cyan, `mix-blend-mode: screen`, converging on load |
| **Vignette / scan / scrim** | Fixed gradient layers with blend modes |

The flare loop suspends itself when the reel scrolls out of view rather than burning
frames on an invisible canvas:

```js
if (dolly.getBoundingClientRect().bottom < 0) { setTimeout(drawFlare, 400); return; }
```

## Components

**Countdown** — one `setInterval` at 250ms against a fixed ISO timestamp with an
explicit offset (`+05:30`), so it reads identically in every timezone. Two constants
bracket the range: `WEDDING` is the target, `ASKED` the start, and elapsed progress
between them drives a "footage exposed" bar. Handles the post-target case by switching
copy rather than counting negative.

**Calendar export** — builds a VCALENDAR string in JS, wraps it in a `Blob`, and
triggers a download through `URL.createObjectURL` and a synthetic `<a>` click. Object
URLs are revoked after a second. No server, no library.

**Gallery viewer** — 13 frames, opened from a contact sheet. Arrow keys and `Esc`,
`Tab` cycling trapped inside the dialog, `aria-modal`, scroll locked on `<html>` while
open, and focus returned to the invoking element on close. Images use blur-up: a ~22px
version sits in the element's `background-image` and the full image fades over it once
decoded.

**Scroll reveal** — one `IntersectionObserver` for every animated element on the page.
Items crossing the threshold together are sorted top-to-bottom, left-to-right and given
a staggered `--d` delay so the eye reads them in order instead of everything arriving at
once. Each element is unobserved after firing.

## CSS system

**Tokens.** 20 custom properties on `:root` — a colour grade (cold shadows, tungsten
highlights), three type roles, two easing curves, and three layout scalars.

**Fluid type.** Every size is a `clamp()`. The *minimum* is the important half: it is
what a phone actually gets, so those floors are set from readability rather than from
scaling the desktop value down.

**Breakpoints.** `40rem` (phones), `52rem` and `60rem` (split layouts and the route
map), plus `560/620/780/940px` for individual components, and three
`prefers-reduced-motion` blocks.

**Reduced motion.** Not a decorative concession — the media query structurally unrolls
the reel into a plain vertical document: the dolly collapses to `height: auto`, the
gate stops being sticky, the track stops being a flex row, and every animation, the
clapper, the flare and the scratches are removed.

## Constraints worth knowing before editing

**Images are inlined, not referenced.** Replacing a file next to `index.html` changes
nothing — the base64 payload has to be swapped. One image is duplicated in two places
(the card and its full-screen view) and both copies must change together.

**The file is pure ASCII on purpose.** In markup, non-ASCII is written as entities
(`&#183;`, `&#8212;`). Entities are *not* decoded inside `<script>`, so string literals
use JavaScript escapes instead (`"\u2014"`). This makes the page immune to being served
with the wrong charset — which produces `â€"` mojibake through every dash and degree
sign. Keep to the convention when adding content.

**`<meta name="viewport">` is load-bearing.** Without it, mobile browsers lay the
document out at ~980px and scale the result down: text becomes unreadable *and* no
`max-width` media query ever matches, so the entire mobile stylesheet silently does
nothing.

**Grid tracks want `minmax(0, 1fr)`.** A bare `auto` or `1fr` track takes an
`auto` minimum, so one wide child can size the track past the viewport and drag
its siblings out with it. The frame grids pin their tracks explicitly.

**`svh`, not `vh`.** Mobile toolbars change `vh` mid-scroll, which would resize the gate
underneath a running pan.

## Performance

- One layout read per animation frame, geometry cached between resizes.
- Off-screen frames dropped via `visibility`, not just opacity.
- The canvas loop parks itself when out of view.
- `will-change` declared only on elements that actually transition.
- `font-display: swap` on all six faces.
- Cost is front-loaded: one large document, then zero requests. Good for offline and
  for hosts without compression; bad for a cold connection on mobile data, which is the
  trade this design accepts deliberately.

## Publishing

Any static host. For GitHub Pages:

1. Push `index.html` to the repository root.
2. **Settings → Pages → Source:** deploy from a branch, `main` / `root`.
3. Served at `https://<username>.github.io/<repository>/`.

No CI, no build step, nothing to configure.

## Structure

```
index.html    the entire page — markup, styles, scripts and assets
README.md     this file
```

---

<p align="center">
  <sub>Titles &amp; code by <a href="https://github.com/ouseph444"><b>Dr. Ouseph C.J.</b></a></sub>
</p>
