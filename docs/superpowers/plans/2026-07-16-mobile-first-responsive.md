# Mobile-First Responsive Fix Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the 4 Pump&Flow site pages (`index.html`, `clases.html`, `curso.html`, `personales.html`) render correctly on phones, using a mobile-first CSS architecture (base styles target small screens, `min-width` media queries enhance for desktop).

**Architecture:** Nav, buttons, footer, `.wrap`, and a responsive typography override move out of each page's duplicated inline `<style>` block into the shared `pump-flow-design/diseño-marca-pump-flow/styles.css` (already `<link>`-ed by all 4 pages). Each page's own page-specific layout (hero, grids, cards) gets its own `@media (max-width: 679px)` block appended to its inline `<style>`. Single breakpoint: **679px/680px**. No JS framework — the mobile nav menu uses the checkbox+label CSS toggle pattern (no script).

**Tech Stack:** Static HTML + CSS only (no build step, no JS framework). Design tokens live in `pump-flow-design/diseño-marca-pump-flow/tokens/*.css` and are consumed via CSS custom properties (`var(--...)`).

## Global Constraints

- Breakpoint: `@media (max-width: 679px)` for mobile overrides (mobile-first: unscoped rules are the mobile/base styles where noted, desktop rules are unaffected — see Task 1).
- No new dependencies, no JS framework, no build step.
- Do not modify anything under `pump-flow-design/` other than `diseño-marca-pump-flow/styles.css`.
- Preserve all existing desktop visuals pixel-for-pixel where no mobile-specific override applies (verified at ~1280px viewport in Task 5).
- All interactive elements (nav links, buttons) keep a minimum 44-48px touch target on mobile.
- Local dev server command (already whitelisted): `python3 -m http.server 8791`, served from the repo root `/Users/caroaguilar/CODE/pump-flow`.

---

### Task 1: Shared responsive chrome (styles.css) + index.html

**Files:**
- Modify: `pump-flow-design/diseño-marca-pump-flow/styles.css`
- Modify: `index.html`

**Interfaces:**
- Produces (consumed by Tasks 2-4): shared CSS classes `.wrap`, `.btn`, `.btn-primary`, `.btn-outline`, `.btn-white`, `nav`, `nav .links`, `.nav-toggle`, `.nav-burger`, `footer`, `footer .inner` — all now defined once in `styles.css`. Every other page must delete its own copies of these rules and reuse the shared nav markup pattern shown below.
- Produces: shared nav markup pattern (5 direct children of `<nav>`, in this exact order): logo `<a>`, `<input class="nav-toggle">`, `<label class="nav-burger">`, `.links` `<div>`, CTA `<a class="btn btn-primary">`.

- [ ] **Step 1: Replace `styles.css` with the shared chrome + responsive rules**

Current full file content is just 4 `@import` lines. Replace the entire file with:

```css
@import "tokens/fonts.css";
@import "tokens/colors.css";
@import "tokens/typography.css";
@import "tokens/spacing.css";

/* Shared page chrome: body, links, wrap, buttons, nav, footer */
body { margin: 0; background: var(--surface-page); color: var(--text-body); font: var(--text-body); }
a { color: var(--pf-morado); text-decoration: none; }
a:hover { color: var(--pf-lavanda-hover); }

.wrap { max-width: 1080px; margin: 0 auto; padding: 0 32px; }

.btn { font: var(--text-button); border: none; border-radius: var(--radius-pill); cursor: pointer; display: inline-flex; align-items: center; gap: 8px; padding: 14px 30px; min-height: 48px; transition: background var(--transition-fast), transform var(--transition-fast); text-decoration: none; }
.btn:active { transform: scale(.98); }
.btn-primary { background: var(--pf-lavanda); color: #fff; }
.btn-primary:hover { background: var(--pf-lavanda-hover); color: #fff; }
.btn-outline { background: transparent; color: var(--pf-morado); box-shadow: inset 0 0 0 2px var(--pf-lavanda); }
.btn-outline:hover { background: var(--pf-lila-fondo); }
.btn-white { background: #fff; color: var(--pf-morado); }
.btn-white:hover { background: var(--pf-lila); }

nav { display: flex; align-items: center; justify-content: space-between; padding: 18px 0; }
nav .links { display: flex; gap: 28px; font: var(--text-body-strong); font-size: 15px; }
.nav-toggle { display: none; }
.nav-burger { display: none; }

footer { background: var(--pf-morado); border-top: 1px solid rgba(255,255,255,.15); padding: 32px 0; color: var(--pf-lila); }
footer .inner { display: flex; justify-content: space-between; align-items: center; font: var(--text-small); }

/* Mobile (<680px): smaller display type, tighter spacing, hamburger nav */
@media (max-width: 679px) {
  :root {
    --text-display-xl: 700 34px/1.15 var(--font-display);
    --text-display: 700 28px/1.2 var(--font-display);
    --text-h1: 700 24px/1.2 var(--font-display);
  }

  .wrap { padding: 0 20px; }

  nav { flex-wrap: wrap; row-gap: 12px; }

  .nav-burger {
    order: 0;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 44px;
    height: 44px;
    font-size: 22px;
    color: var(--pf-morado);
    cursor: pointer;
    border-radius: var(--radius-pill);
  }
  .nav-burger:hover { background: var(--pf-lila-fondo); }

  nav .btn-primary { order: 1; margin-left: auto; }

  nav .links {
    order: 2;
    flex-basis: 100%;
    flex-direction: column;
    gap: 0;
    max-height: 0;
    overflow: hidden;
    transition: max-height var(--transition-base);
  }
  .nav-toggle:checked ~ .links { max-height: 260px; }
  nav .links a { padding: 14px 4px; border-top: 1px solid var(--border-soft); font-size: 16px; }

  footer .inner { flex-direction: column; gap: 6px; text-align: center; }
}
```

Why this ordering works without a wrapper `<div>`: on desktop, the visible flex children of `<nav>` in DOM order are logo → `.links` → CTA button (the checkbox and burger label are `display:none`), which reproduces today's layout exactly. On mobile, `order` promotes the CTA button to sit right after the burger icon on row 1 (pushed to the far right via `margin-left:auto`), while `.links` (given `flex-basis:100%`) wraps to row 2 and stays collapsed (`max-height:0`) until the hidden checkbox is checked.

- [ ] **Step 2: Update `index.html`'s nav markup**

Find:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank" style="text-decoration:none">Reserva tu clase ✨</a>
  </nav>
</header>
```

Replace with:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <input type="checkbox" id="nav-toggle" class="nav-toggle">
    <label for="nav-toggle" class="nav-burger" aria-label="Abrir menú">☰</label>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

- [ ] **Step 3: Remove the now-shared rules from `index.html`'s inline `<style>`**

Find (the block right after the `<style>` opening tag):
```html
<style>
  body{margin:0;background:var(--surface-page);color:var(--text-body);font:var(--text-body)}
  a{color:var(--pf-morado);text-decoration:none}
  a:hover{color:var(--pf-lavanda-hover)}
  .wrap{max-width:1080px;margin:0 auto;padding:0 32px}
  .btn{font:var(--text-button);border:none;border-radius:var(--radius-pill);cursor:pointer;display:inline-flex;align-items:center;gap:8px;padding:14px 30px;min-height:48px;transition:background var(--transition-fast),transform var(--transition-fast)}
  .btn:active{transform:scale(.98)}
  .btn-primary{background:var(--pf-lavanda);color:#fff}
  .btn-primary:hover{background:var(--pf-lavanda-hover)}
  .btn-outline{background:transparent;color:var(--pf-morado);box-shadow:inset 0 0 0 2px var(--pf-lavanda)}
  .btn-outline:hover{background:var(--pf-lila-fondo)}
  .btn-white{background:#fff;color:var(--pf-morado)}
  .btn-white:hover{background:var(--pf-lila)}
  nav{display:flex;align-items:center;justify-content:space-between;padding:18px 0}
  nav .links{display:flex;gap:28px;font:var(--text-body-strong);font-size:15px}
  .hero{display:grid;grid-template-columns:1.1fr 1fr;gap:48px;align-items:center;padding-top:56px;padding-bottom:72px}
```

Replace with:
```html
<style>
  .hero{display:grid;grid-template-columns:1.1fr 1fr;gap:48px;align-items:center;padding-top:56px;padding-bottom:72px}
```

- [ ] **Step 4: Add a class to the hero button row and stack it on mobile**

Find:
```html
    <div style="display:flex;gap:12px">
      <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20de%20introducci%C3%B3n%20%F0%9F%A4%8D" target="_blank" style="text-decoration:none">Agenda tu primera clase</a>
      <a class="btn btn-outline" href="curso.html" style="text-decoration:none">Conoce los cursos</a>
    </div>
```

Replace with:
```html
    <div class="cta-row">
      <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20de%20introducci%C3%B3n%20%F0%9F%A4%8D" target="_blank">Agenda tu primera clase</a>
      <a class="btn btn-outline" href="curso.html">Conoce los cursos</a>
    </div>
```

- [ ] **Step 5: Remove the shared footer rules from `index.html`'s inline `<style>` and append the page-specific mobile block**

Find:
```html
  footer{background:var(--pf-morado);border-top:1px solid rgba(255,255,255,.15);padding:32px 0;color:var(--pf-lila)}
  footer .inner{display:flex;justify-content:space-between;align-items:center;font:var(--text-small)}
</style></head><body>
```

Replace with:
```html
  .cta-row{display:flex;gap:12px}
  @media (max-width: 679px) {
    .hero{grid-template-columns:1fr;gap:28px;padding-top:32px;padding-bottom:48px;text-align:center}
    .hero>div:first-child{order:2}
    .hero .foto{order:1;min-height:240px}
    .hero p{margin-left:auto;margin-right:auto}
    .cta-row{flex-direction:column}
    .bloque-lavanda{padding:48px 0}
    .bub:nth-child(even),.bub:nth-child(odd){margin-left:0;margin-right:0}
    .clases{padding-top:48px;padding-bottom:48px}
    .grid3{grid-template-columns:1fr}
    .reserva{padding:48px 0}
    .reserva .inner{grid-template-columns:1fr;gap:28px;text-align:center}
    .form{min-height:220px}
  }
</style></head><body>
```

- [ ] **Step 6: Serve the site and verify `index.html`**

Run: `cd /Users/caroaguilar/CODE/pump-flow && python3 -m http.server 8791`

Open `http://localhost:8791/index.html` in a browser and check with devtools responsive mode:
- At 375px width: hero photo is above the headline/text, the two hero buttons stack full-width, the "estas clases son para ti" bubbles are centered without the left/right zig-zag offset, the 3 class cards stack in one column, the reserve section stacks with the map below the text, the nav shows only logo + ☰ + "Reserva tu clase" button on one row, and tapping ☰ reveals the 3 links below with no horizontal scrollbar anywhere on the page.
- At 1280px width: page looks identical to how it did before this task (nav links visible inline, 3-column class grid, side-by-side hero).

Expected: both checks pass. If not, fix the CSS before proceeding.

- [ ] **Step 7: Commit**

```bash
git add pump-flow-design/diseño-marca-pump-flow/styles.css index.html
git commit -m "Make nav/buttons/footer responsive and mobile-first index.html"
```

---

### Task 2: clases.html responsive update

**Files:**
- Modify: `clases.html`

**Interfaces:**
- Consumes: shared classes/markup pattern produced by Task 1 (`.wrap`, `.btn*`, `nav`, `.nav-toggle`, `.nav-burger`, `footer`).

- [ ] **Step 1: Update the nav markup**

Find:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

Replace with:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <input type="checkbox" id="nav-toggle" class="nav-toggle">
    <label for="nav-toggle" class="nav-burger" aria-label="Abrir menú">☰</label>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

- [ ] **Step 2: Remove the now-shared rules from the inline `<style>`**

Find:
```html
<style>
  body{margin:0;background:var(--surface-page);color:var(--text-body);font:var(--text-body)}
  a{color:var(--pf-morado);text-decoration:none}
  a:hover{color:var(--pf-lavanda-hover)}
  .wrap{max-width:1080px;margin:0 auto;padding:0 32px}
  .btn{font:var(--text-button);border:none;border-radius:var(--radius-pill);cursor:pointer;display:inline-flex;align-items:center;gap:8px;padding:14px 30px;min-height:48px;transition:background var(--transition-fast),transform var(--transition-fast);text-decoration:none}
  .btn:active{transform:scale(.98)}
  .btn-primary{background:var(--pf-lavanda);color:#fff}
  .btn-primary:hover{background:var(--pf-lavanda-hover);color:#fff}
  nav{display:flex;align-items:center;justify-content:space-between;padding:18px 0}
  nav .links{display:flex;gap:28px;font:var(--text-body-strong);font-size:15px}
  .hero{text-align:center;padding:56px 0 40px}
```

Replace with:
```html
<style>
  .hero{text-align:center;padding:56px 0 40px}
```

- [ ] **Step 3: Remove the shared footer rules and append the page-specific mobile block**

Find:
```html
  footer{background:var(--pf-morado);border-top:1px solid rgba(255,255,255,.15);padding:32px 0;color:var(--pf-lila)}
  footer .inner{display:flex;justify-content:space-between;align-items:center;font:var(--text-small)}
</style></head><body>
```

Replace with:
```html
  @media (max-width: 679px) {
    .hero{padding:40px 0 28px}
    .horarios{grid-template-columns:1fr}
    .intro{padding:40px 0}
    .intro .card{grid-template-columns:1fr;gap:24px;padding:28px;text-align:center}
    .precios{padding:40px 0}
    .pgrid{grid-template-columns:1fr}
  }
</style></head><body>
```

- [ ] **Step 4: Serve the site and verify `clases.html`**

Run: `cd /Users/caroaguilar/CODE/pump-flow && python3 -m http.server 8791` (skip if already running from Task 1)

Open `http://localhost:8791/clases.html` and check with devtools responsive mode:
- At 375px width: nav matches index.html's mobile nav (logo + ☰ + reserve button, menu opens/closes); the two schedule cards stack in one column; the intro card (text + checklist) stacks with reduced padding; the two price cards stack in one column; no horizontal scrollbar.
- At 1280px width: identical to how it looked before this task.

Expected: both checks pass.

- [ ] **Step 5: Commit**

```bash
git add clases.html
git commit -m "Make clases.html mobile-first"
```

---

### Task 3: curso.html responsive update

**Files:**
- Modify: `curso.html`

**Interfaces:**
- Consumes: shared classes/markup pattern produced by Task 1.

- [ ] **Step 1: Update the nav markup**

Find:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

Replace with:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <input type="checkbox" id="nav-toggle" class="nav-toggle">
    <label for="nav-toggle" class="nav-burger" aria-label="Abrir menú">☰</label>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

- [ ] **Step 2: Remove the now-shared rules from the inline `<style>`**

Find:
```html
<style>
  body{margin:0;background:var(--surface-page);color:var(--text-body);font:var(--text-body)}
  a{color:var(--pf-morado);text-decoration:none}
  a:hover{color:var(--pf-lavanda-hover)}
  .wrap{max-width:1080px;margin:0 auto;padding:0 32px}
  .btn{font:var(--text-button);border:none;border-radius:var(--radius-pill);cursor:pointer;display:inline-flex;align-items:center;gap:8px;padding:14px 30px;min-height:48px;transition:background var(--transition-fast),transform var(--transition-fast);text-decoration:none}
  .btn:active{transform:scale(.98)}
  .btn-primary{background:var(--pf-lavanda);color:#fff}
  .btn-primary:hover{background:var(--pf-lavanda-hover);color:#fff}
  nav{display:flex;align-items:center;justify-content:space-between;padding:18px 0}
  nav .links{display:flex;gap:28px;font:var(--text-body-strong);font-size:15px}
  .hero{display:grid;grid-template-columns:1.1fr 1fr;gap:48px;align-items:center;padding-top:56px;padding-bottom:64px}
```

Replace with:
```html
<style>
  .hero{display:grid;grid-template-columns:1.1fr 1fr;gap:48px;align-items:center;padding-top:56px;padding-bottom:64px}
```

- [ ] **Step 3: Remove the shared footer rules and append the page-specific mobile block**

Find:
```html
  footer{background:var(--pf-morado);border-top:1px solid rgba(255,255,255,.15);padding:32px 0;color:var(--pf-lila)}
  footer .inner{display:flex;justify-content:space-between;align-items:center;font:var(--text-small)}
</style></head><body>
```

Replace with:
```html
  @media (max-width: 679px) {
    .hero{grid-template-columns:1fr;gap:24px;padding-top:32px;padding-bottom:40px;text-align:center}
    .contenido{padding:40px 0}
    .modulos{grid-template-columns:1fr}
    .cta{padding:40px 0}
  }
</style></head><body>
```

- [ ] **Step 4: Serve the site and verify `curso.html`**

Run: `cd /Users/caroaguilar/CODE/pump-flow && python3 -m http.server 8791` (skip if already running)

Open `http://localhost:8791/curso.html` and check with devtools responsive mode:
- At 375px width: nav matches the mobile pattern from Task 1; hero text and the 2 stacked photos are in one column, centered; the 3 module cards stack in one column; no horizontal scrollbar.
- At 1280px width: identical to how it looked before this task.

Expected: both checks pass.

- [ ] **Step 5: Commit**

```bash
git add curso.html
git commit -m "Make curso.html mobile-first"
```

---

### Task 4: personales.html responsive update

**Files:**
- Modify: `personales.html`

**Interfaces:**
- Consumes: shared classes/markup pattern produced by Task 1.

- [ ] **Step 1: Update the nav markup**

Find:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

Replace with:
```html
<header class="wrap">
  <nav>
    <a href="index.html"><img src="pump-flow-design/diseño-marca-pump-flow/assets/logo-morado.png" alt="Pump&Flow" style="height:64px"></a>
    <input type="checkbox" id="nav-toggle" class="nav-toggle">
    <label for="nav-toggle" class="nav-burger" aria-label="Abrir menú">☰</label>
    <div class="links"><a href="clases.html">Clases</a><a href="curso.html">Curso en línea</a><a href="personales.html">Clases personales</a></div>
    <a class="btn btn-primary" href="https://wa.me/524431331146?text=Hola%20Caro%2C%20quiero%20agendar%20mi%20clase%20%F0%9F%A4%8D" target="_blank">Reserva tu clase ✨</a>
  </nav>
</header>
```

- [ ] **Step 2: Remove the now-shared rules from the inline `<style>`**

Find:
```html
<style>
  body{margin:0;background:var(--surface-page);color:var(--text-body);font:var(--text-body)}
  a{color:var(--pf-morado);text-decoration:none}
  a:hover{color:var(--pf-lavanda-hover)}
  .wrap{max-width:1080px;margin:0 auto;padding:0 32px}
  .btn{font:var(--text-button);border:none;border-radius:var(--radius-pill);cursor:pointer;display:inline-flex;align-items:center;gap:8px;padding:14px 30px;min-height:48px;transition:background var(--transition-fast),transform var(--transition-fast);text-decoration:none}
  .btn:active{transform:scale(.98)}
  .btn-primary{background:var(--pf-lavanda);color:#fff}
  .btn-primary:hover{background:var(--pf-lavanda-hover);color:#fff}
  nav{display:flex;align-items:center;justify-content:space-between;padding:18px 0}
  nav .links{display:flex;gap:28px;font:var(--text-body-strong);font-size:15px}
  .hero{text-align:center;padding:56px 0 40px}
```

Replace with:
```html
<style>
  .hero{text-align:center;padding:56px 0 40px}
```

- [ ] **Step 3: Remove the shared footer rules and append the page-specific mobile block**

Find:
```html
  footer{background:var(--pf-morado);border-top:1px solid rgba(255,255,255,.15);padding:32px 0;color:var(--pf-lila)}
  footer .inner{display:flex;justify-content:space-between;align-items:center;font:var(--text-small)}
</style></head><body>
```

Replace with:
```html
  @media (max-width: 679px) {
    .hero{padding:40px 0 28px}
    .card{padding:28px}
    .cta{padding:32px 0 48px}
  }
</style></head><body>
```

- [ ] **Step 4: Serve the site and verify `personales.html`**

Run: `cd /Users/caroaguilar/CODE/pump-flow && python3 -m http.server 8791` (skip if already running)

Open `http://localhost:8791/personales.html` and check with devtools responsive mode:
- At 375px width: nav matches the mobile pattern from Task 1; the info card has tighter padding and fits the screen with no horizontal scrollbar; the WhatsApp CTA button is fully visible and tappable.
- At 1280px width: identical to how it looked before this task.

Expected: both checks pass.

- [ ] **Step 5: Commit**

```bash
git add personales.html
git commit -m "Make personales.html mobile-first"
```

---

### Task 5: Cross-page verification

**Files:**
- None expected (verification only). If any issue is found, fix it in the relevant file from Tasks 1-4 before committing here.

**Interfaces:**
- Consumes: the completed state of all 4 pages + `styles.css` from Tasks 1-4.

- [ ] **Step 1: Serve the site**

Run: `cd /Users/caroaguilar/CODE/pump-flow && python3 -m http.server 8791` (skip if already running)

- [ ] **Step 2: Walk all 4 pages at 375px (iPhone SE width)**

For each of `index.html`, `clases.html`, `curso.html`, `personales.html`, open in a browser with devtools responsive mode set to 375×667:
- No element causes horizontal scrolling (check by scrolling the page fully right — it should not move).
- Tap the ☰ icon: the menu opens showing the 3 links, tap again (or tap ☰ again): it closes.
- Every link in the opened menu navigates to the right page.
- The "Reserva tu clase" / WhatsApp buttons are visible without opening the menu and are comfortably tappable (not visually cramped or clipped).

- [ ] **Step 3: Walk all 4 pages at 1280px (desktop width)**

For each page, confirm the layout looks the same as it did on the `main` branch before this work (3-column grids where applicable, inline nav links, side-by-side hero) — i.e., no visual regression for desktop users.

- [ ] **Step 4: Fix any issue found**

If Step 2 or Step 3 surfaced a problem, fix it in the offending file (`styles.css` or the specific page's inline `<style>`/markup) and re-run Steps 2-3 for the affected page(s) until they pass.

- [ ] **Step 5: Commit (only if Step 4 required changes)**

```bash
git add -A
git commit -m "Fix mobile verification issues"
```

If Step 4 required no changes, skip this commit — Task 5 is verification-only.
