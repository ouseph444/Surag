# Surag & Mary Joy — Wedding Invitation

A digital wedding invitation, built as a single self-contained HTML file.

The page opens on a clapperboard, then scroll drives a horizontal camera pan across
five frames — titles, a card, a photograph, a live countdown, and the printed
invitation with rolling end credits. Below the reel sit the practical parts: both
call sheets, a contact-sheet gallery, the drive from the airport, where to stay, and
what to see while you are in Kerala.

**[View the invitation →](#publishing)** *(replace this link with your GitHub Pages URL once it is live)*

---

## The details

|  | Ceremony | Reception |
|---|---|---|
| **Date** | Sunday, 6 September 2026 | Monday, 7 September 2026 |
| **Time** | 11:00 IST *(seated by 10:45)* | 18:00 IST |
| **Venue** | Enchanted Woods | Zenha Arena Convention Centre |
| **Where** | Koovappady, Thottuvakavala, Perumbavoor, Kerala 683544 | A M Road, Nellikuzhi, Kothamangalam, Kerala 686691 |
| **Maps** | [Open](https://www.google.com/maps/search/?api=1&query=Enchanted+Woods+Koovappady+Thottuvakavala+Perumbavoor+Kerala+683544) | [Open](https://www.google.com/maps/place/Zenha+Arena+Convention+Centre+%2F+Zenha+Auditorium/@10.0720267,76.5977949,17z) |

The two venues are about 40 minutes apart.

### Getting there

Fly into **Cochin International (COK)**, Nedumbassery — not Trivandrum, not Calicut.
Both venues lie east of the airport on the same road out of Perumbavoor.

- **COK → Enchanted Woods** — ≈ 25 km, ≈ 45 min
- **COK → Zenha Arena** — ≈ 40 km, ≈ 1 hr

---

## What the page does

- **Scroll-driven pan** — vertical scroll moves the camera sideways across the reel,
  with per-frame rack focus, plate parallax, an anamorphic lens flare, film grain,
  gate weave and print scratches.
- **Live countdown** to 11:00 IST on 6 September 2026, so it reads true from Kochi,
  from Manila, from anywhere.
- **Add to calendar** — generates `.ics` files for the ceremony and the reception
  entirely in the browser. No server involved.
- **Contact-sheet gallery** — thirteen frames with a keyboard-navigable viewer
  (arrow keys, `Esc`, focus trapping) and blur-up image loading.
- **Travel guide** — the drive as four-stop rails, plus hotels and things to do as
  scannable tiles with distance badges.
- **Graceful degradation** — under `prefers-reduced-motion` the reel unrolls into a
  plain vertical page and every animation stops.

## How it is built

One file. No build step, no dependencies, no network requests at runtime.

| | |
|---|---|
| **Stack** | Hand-written HTML, CSS and vanilla JavaScript |
| **Fonts** | Cinzel, Spectral, IBM Plex Mono — embedded as base64 WOFF2 |
| **Images** | Embedded as base64 data URIs (JPEG and WebP) |
| **Size** | ≈ 2.4 MB total |
| **Runtime deps** | None |

Because everything is inlined, the page works offline and can be opened straight
from disk, emailed as an attachment, or dropped onto any static host.

### Notes for anyone editing it

**Images are baked into the HTML.** They are not referenced from disk, so replacing a
`.jpg` in this folder does nothing to the page — the base64 payload inside
`index.html` has to be replaced too. The printed invitation appears **twice**
(`#card` in the reel and `#lightbox` for the full-screen view); both copies need
updating together.

**The file is deliberately pure ASCII.** In the markup, every non-ASCII character is
written as an HTML entity — `&#183;` for `·`, `&#8212;` for `—`. Inside `<script>`
blocks, entities are not decoded, so string literals use JavaScript escapes instead:
`"\u2014"`. This keeps the text safe even if a host serves the file with the wrong
charset. If you add content, please follow the same convention rather than typing the
characters directly.

**`<meta name="viewport">` must stay.** Without it phones lay the page out at ~980px
and scale it down, which makes every line unreadable and stops all mobile CSS from
applying.

## Publishing

Any static host will do. For GitHub Pages:

1. Push `index.html` to the repository root.
2. **Settings → Pages → Source:** deploy from a branch, `main` / `root`.
3. The invitation is live at `https://<username>.github.io/<repository>/`.

## Structure

```
index.html    the entire invitation — markup, styles, scripts and assets
README.md     this file
```

---

<p align="center">
  <sub>Titles &amp; code by <a href="https://github.com/ouseph444"><b>Dr. Ouseph C.J.</b></a></sub>
</p>

<p align="center">
  <sub>Surag &amp; Mary Joy &#183; 6 &amp; 7 September 2026 &#183; Perumbavoor and Nellikuzhi, Kerala</sub>
</p>
