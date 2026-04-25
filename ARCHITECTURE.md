# MERCY REVIVAL — Architecture & Build Notes

A scroll-driven visual album experience. Single self-contained HTML file. No build step required — just open `index.html`.

---

## 1. Site Architecture (Acts → Scenes)

| # | Section ID | Narrative Act | Emotion | Key Visual |
|---|---|---|---|---|
| 0 | `#cover` | Entry | Anticipation | Title card, drifting embers, glitch on title |
| 1 | `#act-i` | THE BREAK — Trauma | Disorientation, loss | Burning church silhouette, flickering neon "MERCY", SVG cracks drawing across screen, hooded scream fragment |
| 2 | `#act-ii` | THE ROAD — Wandering | Loneliness | Moonlit highway, telephone poles sweeping past, motel "VACANCY", floating lyric fragments, wanderer/diner photo memory ghosts |
| 3 | `#act-iii` | THE RECKONING | Stillness, confrontation | Single mic-stand silhouette, breathing spotlight, **featured Samply track** |
| 4 | `#act-iv` | THE SPARK — Redemption | Hope rebuilding | Volumetric god-rays, glowing lantern SVG with flickering flame, piano-hand fragment |
| 5 | `#act-v` | THE REVIVAL | Release, renewal | Sunrise over fields, full band photo grid |
| 6 | `#music` | Music Section | — | Spinning vinyl + **full Samply channel embed** |
| 7 | `footer` | Outro | — | Mark + tagline + minimal links |

---

## 2. File Structure

```
Mercy Revival Assets/
├── index.html              ← the entire site
├── ARCHITECTURE.md         ← this doc
├── images/                 ← band photos (referenced by index.html)
│   ├── DSC07101.jpg        Act II wanderer
│   ├── DSC07103.jpg        Act V band
│   ├── DSC07121.jpg        Act III mic profile
│   ├── DSC07123-2.jpg      Act I scream fragment
│   ├── DSC07137.jpg        Act II diner ghost
│   ├── DSC07148.jpg        Act IV piano hands
│   ├── DSC07159.jpg        Act V wide group portrait
│   ├── DSC07189.jpg        Act V B&W studio
│   ├── DSC07221.jpg        Act V band feature
│   ├── DSC07248.jpg        Act V smiling moment
│   └── DSC07257.jpg        Act V B&W five-piece
└── DSC07*.jpg (originals also at root)
```

---

## 3. Visual Direction (per act)

**Color palette (CSS variables in `:root`):**
- `--burnt-orange: #c25a2b`
- `--dusty-amber: #d4a256`
- `--deep-charcoal: #14110f`
- `--washed-denim: #5a7a8c`
- `--ember: #f5904c`
- `--sepia: #c9a679`
- `--bone: #e8dcc4`

**Typography stack:**
- Cinematic serif headlines → **Cormorant Garamond** (italic for emotional lines)
- Distressed Americana display → **Rye** (cover title, revival mark)
- Typewriter UI/labels → **Special Elite**
- Clean sans for body → **Inter**

**Lighting per act:**
- I — single radial fire glow + flicker
- II — moonlight from upper-left, blue/charcoal palette
- III — single breathing spotlight, total darkness elsewhere
- IV — conic god-rays from above, warming amber palette
- V — full sunrise wash, warm bone/amber

---

## 4. Interaction & Animation Breakdown

| System | Technique |
|---|---|
| Smooth scroll | **Lenis** (CDN) with custom easing, hooked into GSAP ticker so ScrollTrigger stays in sync |
| Scroll animations | **GSAP 3.12 + ScrollTrigger** — every reveal, every parallax layer |
| Parallax | All elements with `data-speed="X"` — value < 1 = mid/foreground (slower), > 1 = background. Driven by single ScrollTrigger loop |
| Cursor parallax | Subtle 2D translate on title/lantern/neon based on mouse position |
| Title glitch | CSS `::before`/`::after` color-channel split, triggered every 4–10s randomly |
| Cover embers | 24 ember dots, each independently animated rising on a loop |
| SVG cracks | `pathLength="1"` + `stroke-dashoffset` scrubbed from 1→0 across Act I scroll |
| Church embers | 18 SVG circles randomly emitting upward from the burning church |
| Telephone poles | JS-generated, swept horizontally as you scroll Act II |
| Lyric fragments | Each fades in/out across its own scroll range |
| Lantern flame | SVG path with `<animate>` opacity flicker + radial glow pulse |
| Vinyl record | CSS `@keyframes spin`, `animation-play-state` toggled by ScrollTrigger entering view; spin rate accelerates on hover |
| Click ripple | 8 ember dots burst from cursor on click anywhere except interactive elements |
| Custom cursor | Two-element design: lagging ring + immediate dot. Mix-blend-mode difference. Disabled on mobile |
| Progress rail | Right-edge 1px line; fills with ember as you scroll. Active act highlighted in nav |

---

## 5. Samply Integration

The site has **two** Samply embed slots wired up:

1. **Featured Track** in Act III (`.samply-embed`) — 220px height
2. **Full Channel Player** in Music section (`.samply-embed-frame`) — 600px height

Both use Samply's official iframe format:
```html
<iframe src="https://samply.app/embed/PLAYER_ID?color=c25a2b" ...></iframe>
```

**To wire in your real content:**
1. Open your Samply channel/player
2. Click **Embed** in the upper-right
3. Copy the `src` URL
4. Paste it over `https://samply.app/embed/PLAYER_ID` (Act III) and `https://samply.app/embed/CHANNEL_ID` (music section) in `index.html`
5. The `?color=c25a2b` query param themes the player to the burnt-orange palette — keep it or change the hex

Both `<iframe>` lines have `TODO:` comments in the HTML to make them easy to find.

---

## 6. Performance Plan

**Already implemented:**
- All animations use `transform` and `opacity` (GPU-accelerated)
- `will-change: transform` on parallax layers
- `loading="lazy"` on Samply iframes
- Reduced-motion media query disables grain, vinyl spin, all transitions
- Single HTML file, zero build step, zero JS bundle to ship
- CDN-hosted GSAP, ScrollTrigger, Lenis (cached cross-site)

**Recommended next steps before shipping to production:**
1. Convert hero photos to **WebP** (60–70% smaller). Replace `.jpg` references with `.webp` in `<img>` and `<picture>` fallbacks.
2. Generate **2x** and **1x** versions per photo, use `srcset`
3. Run images through `imagemin` or Squoosh — current photos are 1–2MB each, target 200–400KB
4. Add `<link rel="preload">` for the cover-section background photos
5. Self-host Google Fonts as WOFF2 (saves 1 round trip)

---

## 7. Migration Path → Next.js (when you're ready)

When you outgrow the single file, the natural component split is:

```
app/
├── layout.tsx              ← global grain, cursor, Lenis provider
├── page.tsx                ← assembles all acts in order
└── components/
    ├── Cover.tsx
    ├── ActBreak.tsx
    ├── ActRoad.tsx
    ├── ActReckoning.tsx
    ├── ActSpark.tsx
    ├── ActRevival.tsx
    ├── MusicSection.tsx
    ├── Footer.tsx
    ├── overlays/
    │   ├── Grain.tsx
    │   ├── Cursor.tsx
    │   └── ProgressRail.tsx
    └── lib/
        ├── lenis-provider.tsx
        └── use-parallax.ts   ← hook wrapping the data-speed loop

styles/
└── tokens.css              ← :root variables (palette, fonts)
```

Use `gsap/ScrollTrigger` via `useGSAP` (`@gsap/react`) for safe React lifecycle.

---

## 8. UX Rules (Enforced)

- **No traditional navigation** — hidden minimal nav with act numerals only, mix-blend-difference so it adapts to background
- **Scroll IS the narrative** — no skip-to buttons, no tabs
- **Each section is one scene** — no nested tabs, no carousels
- **Transitions are emotional, not just smooth** — every act has a deliberate tonal shift in palette + light
