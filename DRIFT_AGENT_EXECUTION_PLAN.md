# DRIFT — Agent Execution Plan

## Brand Constants
```
Name: DRIFT
Tagline: "Wear the Moment."
Colors:
  --color-base: #F5F0EB
  --color-dark: #1A1A18
  --color-accent: #C4622D
  --color-muted: #9A9189
Fonts:
  Display: Playfair Display (400, 700, 400 italic)
  Body: DM Sans (400, 500)
Rules:
  border-radius: 0 on all buttons and cards (no rounding, ever)
  All transforms use translate3d(), will-change: transform on animated elements
  All images: Next.js <Image> component, .webp format
```

---

## PHASE 1 — Scaffold + Layout + Nav + Lenis

```
Scaffold a Next.js 15 project, TypeScript, Tailwind CSS v4, App Router.
Project name: drift-store

Install:
- gsap (ScrollTrigger, Draggable, InertiaPlugin)
- @studio-freight/lenis
- framer-motion
- @next/font

Create folder structure:
/app
/components/sections
/components/ui
/lib
/public/images/hero
/public/images/products
/public/images/editorial
/public/images/ugc
/styles/globals.css

In globals.css:
:root {
  --color-base: #F5F0EB;
  --color-dark: #1A1A18;
  --color-accent: #C4622D;
  --color-muted: #9A9189;
}
body { background: #F5F0EB; }

---

/components/SmoothScroll.tsx
Client component. Initializes Lenis:
  lerp: 0.08
  duration: 1.2
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t))
Syncs to GSAP ticker:
  gsap.ticker.add((time) => lenis.raf(time * 1000))
  gsap.ticker.lagSmoothing(0)
Wraps {children}.

---

/components/Nav.tsx
Fixed, full-width, z-index: 100.

Left: Logo "DRIFT" in Playfair Display, letter-spacing: 0.25em, font-size: 1.1rem, color #1A1A18.
Followed by links: Collections · Lookbook · About — DM Sans 13px, #1A1A18, ml-10.

Right: Search icon · Wishlist icon · Cart icon (static counter showing "0") · "Join" button.
Join button: bg #C4622D, color white, DM Sans 13px, px-5 py-2, border-radius: 0.

Scroll behavior: scrollY > 80 → background rgba(245,240,235,0.96), backdrop-filter: blur(12px).
CSS transition 0.4s ease on background.

Mobile (<768px):
Hamburger icon (right side). Opens full-screen overlay:
  bg: #1A1A18, position: fixed, inset: 0, z-index: 200.
  Links in Playfair Display 3rem white, stacked, centered.
  Framer Motion stagger entrance: each link y: 30→0, opacity: 0→1, stagger: 0.08s, duration: 0.6s.
  Close (✕) button: top-right, color white, 44×44px tap target.

---

/app/layout.tsx
Import SmoothScroll and Nav.
Load fonts via next/font/google:
  Playfair Display: weights [400, 700], style ['normal', 'italic'], display: 'swap'
  DM Sans: weights [400, 500], display: 'swap'
Apply as CSS variables on <html>: --font-display, --font-body.
SmoothScroll wraps all children.
```

---

## PHASE 2 — Hero: Mirror Portal

### 2A — Generate Images
```
Generate and save to /public/images/hero/:

mirror-editorial.webp — 1200×1800px
Young woman, oversized off-white linen jacket, wide-leg trousers. Sun-drenched rooftop terrace,
terracotta walls, plants. Looking slightly off-camera, casual. Warm afternoon light.
Color grade: warm, slightly desaturated.

hero-bg-texture.webp — 1920×1080px
Subtle linen fabric texture, off-white, nearly invisible. Slightly noisy surface. No people.
```

### 2B — Build Component (Opus)
```
Build /components/sections/HeroPortal.tsx

Structure:
  Outer container: position relative, height: 300vh.
  Sticky wrapper: position sticky, top: 0, height: 100vh, overflow: hidden.
  Mirror frame div: position absolute, top: 50%, left: 50%, transform: translate(-50%,-50%).
    Width: 28vw, min-width: 260px. Height: 70vh.
    Border: 2px solid #C4A882.
    Box-shadow: inset 0 0 0 8px #F5F0EB, inset 0 0 0 10px #C4A882.
    Overflow: hidden.
  Next.js <Image> inside frame: fill, object-fit: cover, src: /images/hero/mirror-editorial.webp, priority.
  Frame border overlay div (separate, same absolute position as frame, pointer-events none):
    Used only for the opacity fade animation.

GSAP ScrollTrigger — portal zoom:
  trigger: outer container (300vh div)
  start: "top top"
  end: "bottom bottom"
  scrub: 1.8
  Animate the STICKY WRAPPER: scale 1 → 4.5, transformOrigin: "50% 50%"
  Animate the FRAME BORDER OVERLAY: opacity 1 → 0, completed by progress 0.4

GSAP ScrollTrigger — text fadeout:
  Same trigger element.
  Animate the HERO TEXT GROUP: opacity 1 → 0, completed by progress 0.35.

Hero text group (inside sticky wrapper, outside mirror frame):
  Position: absolute, bottom: 12%, left: 50%, transform: translateX(-50%), text-align: center.
  Line 1: "SS 2025" — DM Sans 11px, letter-spacing: 0.2em, #9A9189, text-transform: uppercase.
  Line 2: "Wear the Moment." — Playfair Display italic, clamp(2.5rem, 5vw, 4rem), #1A1A18.
  Line 3: "Explore Collection ↓" — DM Sans 13px, #C4622D, mt-4.
  Chevron ↓: CSS animation bouncing 6px up/down, 1.5s ease-in-out infinite.
    Removes animation after first scroll event (one-time window scroll listener, passive).

Flanking labels (desktop only, md:block hidden):
  Left of mirror: "New Season" — DM Sans italic 11px, #9A9189, letter-spacing: 0.15em,
    position absolute, transform: rotate(-90deg), left: calc(50% - 18vw), top: 50%.
  Right of mirror: "SS 2025" — same style, rotate(90deg), right: calc(50% - 18vw).

Breathing animation on mirror frame (before scroll):
  @keyframes mirrorBreathe: 0%,100% scale(1) → 50% scale(1.007). Duration 5s infinite ease-in-out.
  Applied to mirror frame div. Removed (class toggle) once scrollY > 10.

Image filter inside mirror: filter: sepia(12%) brightness(103%) saturate(95%)

Grain overlay on sticky wrapper (::after pseudo, or a div):
  Position absolute, inset 0, pointer-events none, z-index: 10.
  Background: inline SVG noise at opacity 0.035.

Background of sticky wrapper: url(/images/hero/hero-bg-texture.webp) center/cover.

gsap.context() for all animations. Kill all instances on useEffect cleanup.
TypeScript throughout.
```

---

## PHASE 3 — Marquee + Kinetic Text

```
Build /components/ui/Marquee.tsx

Props: { variant: 'dark' | 'light' }
Dark: bg #1A1A18, text #F5F0EB.
Light: bg #F5F0EB, text #1A1A18.
Height: 52px. Overflow: hidden.
Content (duplicated for seamless loop):
  "NEW SEASON  ·  WEAR THE MOMENT  ·  THE EASY LAYER  ·  DRIFT SS2025  ·  "
Font: DM Sans 12px, letter-spacing: 0.2em, text-transform: uppercase.
Animation: @keyframes marquee { from: translateX(0) to: translateX(-50%) }
  duration: 22s, linear, infinite.
On hover: animation-play-state: paused.

---

Build /components/sections/KineticText.tsx

Props: { headline: string, subtext: string, bg: 'light' | 'dark', align?: 'left' | 'center' }

Word-split animation:
  Split headline string by spaces. Each word wrapped:
  <span style={{ overflow:'hidden', display:'inline-block' }}>
    <motion.span>word</motion.span>
  </span>
  Each inner span: initial { y: '110%', opacity: 0 }
  On useInView (once: true, margin: "-100px"):
    animate { y: '0%', opacity: 1 }
    transition: duration 0.75, ease: [0.16, 1, 0.3, 1]
    delay: index * 0.055

Subtext: fades in opacity 0→1, delay: (wordCount * 0.055) + 0.2s, duration 0.6s.

bg 'light': background #F5F0EB, headline #1A1A18, subtext #9A9189.
bg 'dark': background #1A1A18, headline #F5F0EB, subtext rgba(245,240,235,0.5).
Headline: Playfair Display, clamp(3.5rem, 7vw, 6.5rem).
Subtext: DM Sans 1rem, max-width: 520px.
Section padding: py-36 px-8. Mobile: py-24 px-6.
align 'center': text-align center, subtext margin auto.

---

In /app/page.tsx, after HeroPortal:
  <Marquee variant="dark" />
  <KineticText headline="Made for the ones who move." subtext="Casual pieces that work as hard as you do — or as little." bg="light" align="center" />
  <KineticText headline="Where everyday meets intentional." subtext="Every fabric chosen. Every cut considered. Nothing wasted." bg="dark" align="left" />
  <KineticText headline="The season, before it arrives." subtext="Drop access for people who know." bg="light" align="center" />
```

---

## PHASE 4 — Horizontal Drag Collection Rail

### 4A — Generate Images
```
Generate and save to /public/images/products/:
All images: 640×960px, portrait 2:3. Warm editorial tone, consistent across all.

01.webp — Oversized light beige linen jacket, flat lay on warm stone surface.
02.webp — Chunky cream knit sweater on a person on a sunlit balcony.
03.webp — Olive cargo trousers with a white tee, urban street background.
04.webp — Deep burgundy silk slip dress, person in warm lamp-lit interior.
05.webp — White fitted crew-neck tee, ultra clean product shot, soft shadow.
06.webp — Dark charcoal waxed canvas jacket, rooftop setting.
07.webp — Sage green wide-leg lounge set (top + trousers), cozy interior.
```

### 4B — Build Component (Opus)
```
Create /lib/collections.ts:
export const collections = [
  { id: 1, name: "The Easy Layer", category: "Outerwear", price: "From €89", image: "/images/products/01.webp" },
  { id: 2, name: "Sunday Morning", category: "Knitwear", price: "From €65", image: "/images/products/02.webp" },
  { id: 3, name: "Urban Utility", category: "Bottoms", price: "From €79", image: "/images/products/03.webp" },
  { id: 4, name: "After Dark", category: "Eveningwear", price: "From €95", image: "/images/products/04.webp" },
  { id: 5, name: "The Base Edit", category: "Essentials", price: "From €45", image: "/images/products/05.webp" },
  { id: 6, name: "Terrain", category: "Outerwear", price: "From €110", image: "/images/products/06.webp" },
  { id: 7, name: "Slow Sundays", category: "Loungewear", price: "From €55", image: "/images/products/07.webp" },
]

---

Build /components/sections/CollectionRail.tsx

Outer section: overflow hidden, bg #F5F0EB, pt-20 pb-16.
Section header inside: "Hidden Gem Pieces" — Playfair Display clamp(2.5rem,5vw,4rem), #1A1A18.
  Below header: a 40px wide, 2px tall, bg #C4622D decorative line. mt-3 mb-10.

Drag hint: "← Drag to explore →" — DM Sans 11px, #9A9189, displayed above the rail.
  Stored in state (visible: true). On first GSAP dragstart event: set visible false.
  Framer Motion opacity transition 0.4s.

Rail wrapper: ref for bounds, overflow hidden.
Rail track: display flex, gap: 20px, width: fit-content, will-change: transform, pl-[10vw] pr-[10vw].

GSAP Draggable on rail track:
  type: "x"
  edgeResistance: 0.85
  bounds: rail wrapper ref
  inertia: true
  snap: (value) => Math.round(value / 340) * 340
  cursor: "grab"
  activeCursor: "grabbing"
  on dragStart: add class 'is-dragging' to rail track → img { pointer-events: none }
  on dragEnd: remove class

Card (320px wide, 480px tall, flex-shrink: 0, overflow hidden, position relative):
  Next.js <Image>: fill, object-fit cover.
  Hover: image scale(1.04), transition 0.7s cubic-bezier(0.25, 1, 0.5, 1).
  Gradient overlay: position absolute, bottom 0, width 100%, height 55%.
    background: linear-gradient(to top, rgba(26,26,24,0.88), transparent).
  Text block (position absolute, bottom 0, left 0, p-5):
    Category: DM Sans 10px, letter-spacing: 0.15em, text-transform: uppercase, #C4622D.
    Name: Playfair Display italic 1.4rem, white, mt-1.
    Price: DM Sans 13px, rgba(255,255,255,0.7), mt-1.

Mobile (<768px):
  Disable Draggable. Rail track: overflow-x auto, scroll-snap-type: x mandatory,
  -webkit-overflow-scrolling: touch. Each card: 80vw, scroll-snap-align: start.
  Drag hint replaced with "Swipe to explore →".

Import collections from /lib/collections.ts. Map to cards.
gsap.context() + cleanup on unmount.
```

---

## PHASE 5A — Product Grid

```
Build /components/ui/ProductCard.tsx
Props: { name: string, category: string, price: string, image: string, index: number }

Image container: aspect-ratio 3/4, overflow hidden, position relative.
Next.js <Image>: fill, object-fit cover.
Hover (on parent card): image scale(1.06), transition 0.8s cubic-bezier(0.25,1,0.5,1).

Wishlist heart: position absolute, top: 12px, right: 12px.
  Default: opacity 0. Card hover: opacity 1. Transition 0.25s.
  onClick: toggle fill state (useState). Purely visual.

Quick Add button: position absolute, bottom: 12px, left: 50%, transform translateX(-50%).
  DM Sans 12px, bg #1A1A18, color white, px-4 py-1.5, border-radius: 100px, white-space: nowrap.
  Default: y: 10px, opacity 0. Card hover: y: 0, opacity 1. Framer Motion, transition 0.25s.
  onClick: set 'added' state true → button text becomes "Added ✓".
    Reset after 1500ms. Purely visual.

Below image (mt-3):
  Category: DM Sans 10px, letter-spacing: 0.15em, uppercase, #9A9189.
  Name: DM Sans 15px, font-weight 500, #1A1A18, mt-1.
  Price: DM Sans 14px, #C4622D, mt-0.5.

Scroll animation (Framer Motion):
  initial: { opacity: 0, y: 50 }
  whileInView: { opacity: 1, y: 0 }
  viewport: { once: true, margin: "-80px" }
  transition: { duration: 0.7, ease: [0.25, 1, 0.5, 1], delay: (index % 3) * 0.1 }

---

Build /components/sections/ProductGrid.tsx
Import collections from /lib/collections.ts.
Section: bg #F5F0EB, py-24 px-8. Max-width: 1400px, margin auto.
Header: "The Edit" — Playfair Display clamp(2.5rem,5vw,4rem), #1A1A18, mb-12.
  Same 40px terracotta underline decoration below.
Grid: display grid, grid-template-columns: repeat(3, 1fr) desktop,
  repeat(2, 1fr) tablet (768px), 1fr mobile. gap: 20px.
Map collections to <ProductCard> with index prop.
```

---

## PHASE 5B — Layered Parallax Story

### 5B-i — Generate Images
```
Generate and save to /public/images/editorial/:

parallax-bg.webp — 1920×1080px
Slightly blurred city street scene, evening golden hour. Warm window lights in buildings.
Blurred feel (this is the background layer, depth-of-field effect). No people visible.

parallax-model.png — 800×1200px, transparent background (cutout)
Full-body person wearing a dark charcoal jacket and relaxed dark trousers.
Standing naturally, not posed. Clean cutout, transparent background.

parallax-jacket.png — 600×800px, transparent background (cutout)
Studio product shot of a dark charcoal jacket, no model. Transparent background.
Slight clockwise rotation (2–3 degrees). No background at all.
```

### 5B-ii — Build Component (Opus)
```
Build /components/sections/ParallaxStory.tsx

Section: height 180vh, bg #1A1A18, position relative.

Inside: a position sticky, top 0, height 100vh, overflow hidden wrapper.
All layers are position absolute inside the sticky wrapper.

LAYER 1 — bg (z-index 1):
  src: /images/editorial/parallax-bg.webp
  Width: 110%, height: 110%, left: -5%, top: -5%.
  filter: blur(3px) brightness(0.55).
  GSAP ScrollTrigger: trigger=section, start="top top", end="bottom bottom", scrub: true.
  Y animation: 0 → -120px.

LAYER 2 — model (z-index 2):
  src: /images/editorial/parallax-model.png
  Width: 40vw, max-width: 560px. Position: right: 12vw, bottom: 0.
  GSAP same trigger. Y animation: 60px → -220px.

LAYER 3 — jacket (z-index 3):
  src: /images/editorial/parallax-jacket.png
  Width: 26vw, max-width: 360px. Position: left: 14vw, top: 8vh.
  filter: drop-shadow(0 30px 80px rgba(0,0,0,0.5)).
  GSAP same trigger. Y animation: 0 → -420px. Rotation: 0deg → -4deg (same scrub).

TEXT BLOCK (z-index 4, position absolute, left: 8vw, top: 32vh):
  "The Easy Layer." — Playfair Display italic, clamp(3rem, 6vw, 5.5rem), white.
  "One piece. Every occasion." — DM Sans 1rem, rgba(255,255,255,0.6), mt-3.
  "Shop Outerwear →" — DM Sans 13px, #C4622D, mt-6, cursor pointer.
  
  Initial state: x: -70px, opacity: 0.
  Separate ScrollTrigger (not scrubbed):
    trigger: section, start: "30% center".
    GSAP to: x: 0, opacity: 1. Duration: 0.9s, ease: "power3.out". Once only.

Mobile (<768px):
  Disable all GSAP animations. Section height: auto.
  Show parallax-bg.webp as fullscreen static image (object-fit cover, height: 100vh).
  Text block: static, centered, position relative.

gsap.context() + cleanup.
TypeScript.
```

---

## PHASE 6A — Size Grid

```
Build /components/ui/SizeGrid.tsx

Sizes (rows): ['XS','S','M','L','XL','XXL']
Colorways (columns): [
  { name: 'Bone', hex: '#F0EBE3' },
  { name: 'Slate', hex: '#8A9BA8' },
  { name: 'Terracotta', hex: '#C4622D' },
  { name: 'Black', hex: '#1A1A18' },
  { name: 'Olive', hex: '#6B7C5C' },
]

Availability data (hardcoded for "The Easy Layer"):
{
  XS:  ['in_stock','in_stock','out_of_stock','in_stock','low_stock'],
  S:   ['in_stock','in_stock','in_stock','low_stock','in_stock'],
  M:   ['low_stock','in_stock','in_stock','in_stock','in_stock'],
  L:   ['out_of_stock','low_stock','in_stock','in_stock','out_of_stock'],
  XL:  ['out_of_stock','out_of_stock','low_stock','in_stock','in_stock'],
  XXL: ['out_of_stock','out_of_stock','out_of_stock','low_stock','in_stock'],
}

Cell styles:
  'in_stock': bg #D4EDD4, color #1A1A18, cursor pointer.
  'low_stock': bg #FFF3CD, color #1A1A18, cursor pointer.
    Tooltip on hover: "Only 2 left" — absolute popover above cell, DM Sans 11px, bg #1A1A18, color white, px-2 py-1.
  'out_of_stock': bg #EDEBE8, color #9A9189, opacity 0.5, cursor not-allowed.
  'selected': bg #C4622D, color white.
    Framer Motion on click: scale spring animation (scale 1→1.15→1, stiffness: 400).

State: selectedCell { size, colorway } | null (useState).
Clicking in_stock or low_stock: sets selectedCell → triggers toast.

Toast: fixed, bottom-6, left-50%, -translate-x-1/2, z-index 300.
  bg #1A1A18, color white, DM Sans 13px, px-6 py-3, border-radius: 100px.
  Content: "Added to wishlist — {size} / {colorway} ✓"
  Framer Motion AnimatePresence: y: 20→0, opacity 0→1 on enter. Reverse on exit.
  Auto-dismiss after 2000ms.

Column headers: colorway name + 10px circle (bg: colorway hex). DM Sans 11px.
Row headers: size label, DM Sans bold 13px.
Legend row below grid: ● In Stock ● Low Stock ● Sold Out — DM Sans 11px, #9A9189.

Section: bg #F5F0EB, py-24 px-8. Centered, max-width 900px.
Header: "Find Your Fit" — Playfair Display clamp(2rem,4vw,3rem), #1A1A18, mb-10.
  Same 40px terracotta underline.

Mobile: grid container overflow-x auto. Size label column: position sticky, left 0, bg #F5F0EB.
```

---

## PHASE 6B — UGC Wall + Sticky Newsletter

### 6B-i — Generate Images
```
Generate and save to /public/images/ugc/ as 01.webp–09.webp.
Mix of 600×600px and 600×750px (vary for masonry).
Aesthetic: candid, phone-camera feel, real people, warm natural light, casual clothes.

01.webp 600×600 — Person in oversized cream jacket on a city street, candid shot.
02.webp 600×750 — Close-up: terracotta linen trousers, sitting on stone steps.
03.webp 600×600 — Two people laughing at an outdoor café, both in neutral tones.
04.webp 600×750 — Mirror selfie, olive green utility jacket.
05.webp 600×600 — Flat lay: neatly arranged neutral wardrobe pieces on a bed.
06.webp 600×750 — Close-up of cream knitwear texture, natural window light.
07.webp 600×600 — Person walking away from camera, wide-leg trousers, cobblestones.
08.webp 600×750 — Rooftop golden hour silhouette, wide-leg trousers, backlit.
09.webp 600×600 — Hands holding a coffee cup, cuffed linen shirt sleeve visible.
```

### 6B-ii — Build Components
```
Build /components/ui/NewsletterCard.tsx

Position: sticky, top: 100px. Max-width: 420px.
bg #1A1A18. Padding: 48px 40px. border-radius: 0.

Content:
  Label: "DRIFT DROPS" — DM Sans 10px, letter-spacing: 0.2em, uppercase, #C4622D.
  Headline: "Join the Drop." — Playfair Display italic 2.8rem, #F5F0EB, mt-3.
  Subtext: "New pieces. First access. No noise." — DM Sans 1rem, rgba(245,240,235,0.55), mt-4.

  Email input (mt-8):
    border: none. border-bottom: 1px solid rgba(245,240,235,0.25).
    bg transparent. color #F5F0EB. DM Sans 15px. py-3. width 100%.
    placeholder "your@email.com" in same rgba color.
    On focus: border-bottom-color #C4622D, transition 0.3s.

  Submit button (mt-4, full width):
    bg #C4622D. color white. DM Sans 14px. py-4. border-radius: 0.
    Hover: bg #A8511F, transition 0.2s.
    onClick: set 'submitted' state true.

  Framer Motion AnimatePresence between two states:
    Default state: input + button.
    Submitted state: "✓ You're on the list." — Playfair Display italic 1.5rem, #F5F0EB, text-center.
    Fade transition 0.4s between states.

  Fine print (mt-6): "No spam. Drop notifications only." — DM Sans 11px, rgba(245,240,235,0.3).

Decorative: Playfair Display "D", font-size: 18rem, opacity: 0.03, position absolute,
  bottom: -2rem, right: -1rem, pointer-events: none, overflow: hidden, line-height: 1,
  color #F5F0EB, user-select: none.

---

Build /components/sections/UGCWall.tsx

Section: bg #F5F0EB, py-24.

Section header (px-8, mb-12):
  "Worn by Real People." — Playfair Display clamp(2.5rem,5vw,4rem), #1A1A18.
  "Tag @drift.wear to be featured." — DM Sans 1rem, #9A9189, mt-3.

Two-column layout (px-8, gap: 40px):
  Left column: flex-1 (60%)
  Right column: w-[38%]

LEFT — UGC photo grid:
  CSS grid, 3 columns, gap: 12px.
  Items 0,2,4,6,8 (odd-indexed by visual position): grid-row: span 2 (taller).
  Heights: span-2 rows = 320px each row, span-1 = 240px.

  Each photo: overflow hidden, position relative, cursor pointer.
  Next.js <Image>: fill, object-fit cover.

  Hover overlay:
    Position absolute, inset 0. bg rgba(26,26,24,0.55). opacity 0→1, transition 0.35s.
  Hover text (position absolute, bottom: 16px, left: 16px):
    opacity 0→1 with overlay. Pointer-events none.
    Handle: DM Sans 12px, white — use these per image:
      01→@lea.moves, 02→@thijs.w, 03→@caro_and_sol, 04→@mats.daily
      05→@drift.wear, 06→@nina.thread, 07→@oluwaseun_, 08→@drift.wear, 09→@amara.style
    Product name below: Playfair Display italic 14px, white, mt-1.
      01→The Easy Layer, 02→Urban Utility, 03→Sunday Morning, 04→Terrain
      05→The Base Edit, 06→Sunday Morning, 07→Slow Sundays, 08→After Dark, 09→The Easy Layer

RIGHT — sticky card:
  Import and render <NewsletterCard />.
  position sticky, top: 100px (handled inside NewsletterCard).

Mobile (<768px):
  Switch to stacked layout (flex-col). Full width on both columns. NewsletterCard not sticky.
  UGC grid: 2 columns.
```

---

## PHASE 7 — Mobile Responsiveness Pass

```
Audit and fix all files in /components/sections/ and /components/ui/ for mobile (<768px).

HeroPortal.tsx:
  If window.innerWidth < 768: skip GSAP ScrollTrigger initialization entirely.
  Show editorial image fullscreen (object-fit cover, height: 100vh).
  Mirror frame: 75vw × 60vh, border only (no GSAP scale).
  Hero text: visible, centered, static.

CollectionRail.tsx:
  Already handled in Phase 4. Verify: drag hint shows "Swipe →", cards are 80vw, snap works.

ParallaxStory.tsx:
  Already handled in Phase 5B. Verify: static image shows, text is readable, no GSAP on mobile.

ProductGrid.tsx:
  1 column on mobile. Quick Add button: always visible (opacity 1, y: 0) on mobile — no hover needed.

SizeGrid.tsx:
  Already handled in Phase 6A. Verify: horizontal scroll works, size column is sticky.

UGCWall.tsx:
  Already handled in Phase 6B. Verify: stacked layout, 2-column ugc grid, card not sticky.

KineticText.tsx:
  On mobile: replace word-split animation with single-line fade.
  If window.innerWidth < 768: animate the entire headline as one element, no word splitting.
  initial: { opacity: 0, y: 30 }. whileInView: { opacity: 1, y: 0 }. duration: 0.6s.

Nav.tsx:
  Verify hamburger tap target ≥ 44×44px.
  Verify overlay closes on any link tap.

All sections:
  Minimum body text: 15px.
  Minimum horizontal padding: 20px.
  No content clips or overflows at 375px viewport width.
```

---

## PHASE 8 — Micro-Details

```
Build /components/ui/CustomCursor.tsx
  Desktop only (hidden if pointer: coarse — touch device detection via matchMedia).
  A div: 12px × 12px, border: 1.5px solid #C4622D, border-radius 50%.
  Position: fixed. z-index: 9999. pointer-events: none. mix-blend-mode: multiply.
  Follows mouse via requestAnimationFrame with lerp 0.15 on x/y.
  On hover of [data-cursor-hover] elements (add this attribute to all <a> and <button>):
    Transition to 40px × 40px, background rgba(196,98,45,0.12), border-color transparent.
  Transition: width, height, background 0.2s ease.
  Import in /app/layout.tsx.

---

Scroll progress bar (in /app/layout.tsx or a dedicated client component):
  A div: position fixed, top 0, left 0, height: 2px, bg #C4622D, z-index: 9999.
  Width updated via requestAnimationFrame:
    progress = window.scrollY / (document.body.scrollHeight - window.innerHeight)
    element.style.width = (progress * 100) + '%'
  transform: translateZ(0), will-change: width.

---

Add a second Marquee instance between ParallaxStory and ProductGrid in /app/page.tsx:
  <Marquee variant="light" />

---

Page load animation — create /components/ui/PageLoader.tsx:
  Full-screen overlay: position fixed, inset 0, bg #F5F0EB, z-index: 1000.
  Centered: "DRIFT" — Playfair Display 3rem, letter-spacing: 0.25em, #1A1A18.
  Framer Motion sequence:
    1. Logo fades in: opacity 0→1, duration 0.5s.
    2. After 300ms: overlay fades out: opacity 1→0, duration 0.5s.
    3. On animation complete: set loaded state true, unmount overlay.
  Wrap in AnimatePresence. Import in /app/layout.tsx.
  While loading: body overflow hidden.

---

Cart animation in ProductCard.tsx:
  On Quick Add click, create a temporary div (8px circle, bg #C4622D, position fixed,
    starting at the card's getBoundingClientRect() center).
  GSAP tween to the nav cart icon's getBoundingClientRect() position.
    duration: 0.6s, ease: "power2.in". Path is a simple arc (set intermediate y via motionPath
    or manual keyframes: y goes up then curves down to cart position).
  On tween complete: remove the circle. Increment cart badge counter (global state or Context).
  Cart badge: Framer Motion scale spring (1→1.4→1, stiffness: 400) on increment.
  Cart badge counter: useState in Nav, or a simple Zustand-free global via Context.
```

---

## Page Assembly — /app/page.tsx

```
Final component order in /app/page.tsx:

<PageLoader />
<HeroPortal />
<Marquee variant="dark" />
<KineticText headline="Made for the ones who move." subtext="Casual pieces that work as hard as you do — or as little." bg="light" align="center" />
<KineticText headline="Where everyday meets intentional." subtext="Every fabric chosen. Every cut considered. Nothing wasted." bg="dark" align="left" />
<CollectionRail />
<KineticText headline="The season, before it arrives." subtext="Drop access for people who know." bg="light" align="center" />
<ParallaxStory />
<Marquee variant="light" />
<ProductGrid />
<SizeGrid />
<UGCWall />
<Footer />
```
