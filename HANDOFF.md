# HANDOFF — AETHER-01 Landing Page + New Animated Telemetry Bento Section

_Written for a brand-new session. Read this first, then `E:\Users\alexc\Desktop\Aether\readme.md` (the page brief) and `E:\Users\alexc\Desktop\Aether\index.html`._

---

## 1. What this project is

A single-file, production-ready landing page for the fictional "AETHER-01" expedition RV.
- Deliverable: `E:\Users\alexc\Desktop\Aether\index.html` (540 lines) — no build step, all CDN.
- Stack: Tailwind CDN (`https://cdn.tailwindcss.com`) + inline `tailwind.config` (colors `ink #0C0E12`, `amber #F59E0B`, `mint #10B981`; fonts display=Space Grotesk, mono=JetBrains Mono); GSAP 3.12.5 + ScrollTrigger; Google Fonts; Lucide UMD (`lucide.createIcons()` is called on load).
- Design language: dark UI (#0C0E12), glassmorphism `.glass` cards, thin `border-white/10`, mono microcopy labels with wide letter-spacing.

## 2. Baseline status (ALL DONE AND VERIFIED — do not redo)

Current page structure:
- **Hero** (`#hero`, h-screen): Unsplash night shot (`photo-1783522271591-e29b05ff38ce`, NOTE: this URL replaced an earlier 404 one — do not revert), van CAD wireframe (`#drawGroup .draw` = 36 paths) draws on, kinetic title chars, custom cursor, parallax fade-out scrub.
- **Section 2** (`#systems`): left column = typography + 4 glass spec rows **using Lucide icons** (solar-panel, battery-charging, thermometer, gauge); right = `.mesh-card` interactive SVG schematic (hover → 3 `.flow` paths light up + stream, `.mesh-node` spring scale).
- **Section 3** (`#reserve`): 10 `.topo` contour lines scrub-drawn on scroll, magnetic CTA (`#cta` inside `#reserve`).

Verified live in-browser: hero intro, parallax, reveals, mesh hover flow, topo scrub (offset 1→0 synced to scroll), magnetic CTA pulls (measured values matched the pull formula), header solid-state toggle, 4/4 ScrollTriggers healthy, GSAP 3.12.5, **0 console errors** (1 benign Tailwind CDN warning only).

One real bug was fixed and verified: the magnetic CTA `mousemove` listener was on the tiny `#ctaZone` wrapper; it now listens on the whole **`#reserve`** section so the 180px attraction radius actually fires.

Testing rig (IMPORTANT): `file://` is blocked in Playwright. Must run a local server first:
`python -m http.server 8765 --bind 127.0.0.1` in `E:\Users\alexc\Desktop\Aether`
(Python 3.12 path: `C:\Users\alexc\AppData\Local\Programs\Python\Python312\python.exe`.)
Then open `http://127.0.0.1:8765/index.html`. The python server was STOPPED — restart it before testing.

---

## 3. ACTIVE TASK — new animated telemetry bento section (NOT YET IMPLEMENTED)

User request (exact wording, abridged): **"add as a section Below the hero, create a 4-column Bento grid displaying the vehicle's telemetry (Solar Yield, Atmospheric Water, Wind Entropy, Core Battery). The Challenge: Do not use static icons. You must write raw inline SVGs for these 4 icons and animate them continuously to mimic Lottie animations using CSS @keyframes or infinite GSAP timelines."**

So: brand-new `<section>` inserted BETWEEN the hero `</section>` (current line 201) and the `<!-- ============ SECTION 2 — THE ENERGY MESH ============ -->` comment (current line 203). Do NOT delete/modify existing sections. Do NOT use Lucide icons inside it — hand-written inline SVGs only, animated forever.

### The 4 required icons (from the user's earlier detail message — this is the authoritative spec)

1. **Solar Yield — Solar Array icon**: sun core surrounded by a gear/ray ring that **rotates continuously** (`transform: rotate(360deg)` loop). Amber tones.
2. **Atmospheric Water — water droplet**: droplet outline + SVG `<clipPath>` (drop-shaped) containing a **wave that translates horizontally AND bobs vertically**, simulating liquid filling/level inside the droplet. Mint/cyan tones.
3. **Wind Entropy — turbine**: **3-blade wind turbine spinning continuously** with *variable speed blur* (e.g. keyframe easing that accelerates/decelerates + faint blade trails and/or slight `blur()` to fake motion blur).
4. **Core Battery — battery**: battery outline/case + **pulsing internal fill height** (grow/shrink) + a **lightning bolt glowing on a continuous opacity loop**.

### Suggested structure (already scouted in the file — free to refine)

```html
<!-- ============ SECTION 1.5 — LIVE TELEMETRY BENTO ============ -->
<section id="telemetry" class="relative py-28 md:py-40">
  <div class="mx-auto max-w-7xl px-6 md:px-10">
    <!-- optional section header, mono kicker style like: // 00 — LIVE TELEMETRY -->
    <div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
      <div class="glass ..."> icon svg + big value + mono label </div>
      x4
    </div>
  </div>
</section>
```

Suggested per-card content pattern (match existing microcopy voice, mono labels in `text-white/40`):
- Solar Yield → e.g. big `2.4 kWp`, label `SOLAR YIELD`, sub `ROOF PHOTOVOLTAIC ARRAY`
- Atmospheric Water → e.g. `6 L / DAY`, label `ATMOSPHERIC WATER`
- Wind Entropy → e.g. `3.1 M/S`, label `WIND ENTROPY`
- Core Battery → e.g. `48 V · 300 AH`, label `CORE BATTERY`
(values are illustrative; pick on-brand values consistent with the systems section copy.)

### Implementation steps to do (in order)

1. **CSS keyframes**: add `@keyframes` to the existing `<style>` block in `<head>` (lines ~37-86), e.g. `spin360`, wave slide (`translateX` loop that is seamless over one wave period), vertical `bob`, turbine `spin` with variable easing, fill `pulse` (use `transform: scaleY` with `transform-origin: bottom` on a full-height rect for the battery fill — more robust than animating the height attr), bolt `glow` opacity loop. SVG transform gotcha: set `transform-box: fill-box; transform-origin: center;` on rotating groups (pattern already used by `.mesh-node`).
2. **HTML**: insert the new section (see suggested structure above). Give icons/parts classes for the JS/CSS hooks (e.g. `.tm-icon`, per-icon classes). Keep hero-adjacent spacing sensible (`py-28 md:py-40` matches `#systems`).
3. **JS**: optional scroll reveal for the new section. Existing reveals target `#systems .reveal` — either add `#telemetry .reveal` classes and a matching `gsap.from(...)` ScrollTrigger, or reuse the pattern. (Do not break the 4 existing ScrollTriggers.)
4. **Verify** with Playwright against the local server:
   - 4 inline SVGs present, each with an infinite running animation (`getAnimations().some(a => a.playState === 'running')` or computed `animation-name` non-none).
   - New section visible on scroll; existing interactions still green; console 0 errors.
   - If the sandbox can't visually confirm, rely on computed styles / animation state as before.

### Open question to confirm with the user (do NOT silently decide)

The user once said "Replace the icon placeholders in the Systems/Telemetry grid with 4 custom inline SVGs" — but then clarified to ADD a new bento section below the hero. So the authoritative current ask = new section. The existing `#systems` left-column Lucide spec rows (solar-panel, battery-charging, thermometer, gauge) are still Lucide. **Ask whether to also swap those 4 for the new animated SVGs, or leave them as-is** — default assumption unless told otherwise: leave existing rows untouched.

---

## 4. Key file locations for reference

- Head/CSS/config: `index.html` lines 1–86.
- Hero close → Section 2 open (insertion point): lines 201–204.
- Systems left spec rows (Lucide icons): lines ~219–248.
- Systems right mesh schematic (`meshSvg`): lines ~252–320.
- JS: intro timeline (~line 437+), Energy Mesh hover (~line 458+), CTA magnetic (~line 505, zone = `#reserve`), header state (~line 534).
- Page brief: `E:\Users\alexc\Desktop\Aether\readme.md`.
