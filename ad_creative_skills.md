# Ad Creative Skills — Technical Architecture & Design Reference

## Overview

This document defines the exact HTML/CSS/JS architecture, design system, typography standards, layout patterns, and client-specific preferences for generating **high-conversion ad creatives** as self-contained `.html` files. Each file contains N slides (minimum 5), all at a consistent platform-appropriate aspect ratio, individually downloadable as PNG.

Platform targets: **Meta Ads** (primary) and **Google Display** (secondary).

---

## 1. Platform Format Reference

| Platform | Format | Dimensions | Aspect Ratio |
|---|---|---|---|
| Meta | Feed Square | 1080 × 1080 px | 1:1 |
| Meta | Feed Portrait | 1080 × 1350 px | 4:5 |
| Meta | Stories / Reels | 1080 × 1920 px | 9:16 |
| Meta | Landscape / Link | 1200 × 628 px | 1.91:1 |
| Google | Medium Rectangle | 300 × 250 px | 6:5 |
| Google | Half Page | 300 × 600 px | 1:2 |
| Google | Discovery / PMax | 1200 × 628 px | 1.91:1 |

**Default if not specified:** Meta Feed Square (1080 × 1080 px).

> All slides in a single HTML file MUST share the same dimensions. Multi-format = multiple HTML files.

---

## 2. Canonical File Structure

```
outputs/<slug>/
├── ad_creatives.html       ← self-contained; all CSS + JS inline
└── images/
    ├── logo.png            ← client logo (copied from /images/<Client>/)
    ├── photo_01.jpg        ← client images (copied from /images/<Client>/)
    └── ...
```

The HTML references images via relative paths: `./images/<filename>`. This means the HTML must be served from its own folder (local HTTP server) — never opened directly with `file://`.

---

## 3. Core HTML Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>[Client Name] — Ad Creatives [Batch ID]</title>
  <!-- ALWAYS use Typekit — NEVER Google Fonts -->
  <link rel="stylesheet" href="https://use.typekit.net/lld6qxp.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    /* ORDER: 1.tokens → 2.reset → 3.page-chrome → 4.ad-wrapper → 5.overlays → 6.logo → 7.typography → 8.per-ad layouts → 9.nav → 10.download */
  </style>
</head>
<body>
  <div class="page-header">
    <h1>[Client] — Ad Creatives [Batch]</h1>
    <p>N Meta Feed Square (1:1) Ads · Navigate with arrows · Download each as PNG</p>
  </div>

  <div class="ad-viewer">
    <div class="ad-counter" id="adCounter">Ad 1 / N</div>
    <div class="ad-wrapper">
      <div class="ad-track" id="adTrack">
        <!-- .ad-slide elements here -->
      </div>
    </div>
    <div class="nav-row">
      <button class="nav-btn" id="btnPrev">←</button>
      <div class="ad-dots" id="adDots"></div>
      <button class="nav-btn" id="btnNext">→</button>
    </div>
    <div class="download-row">
      <button class="download-btn" id="downloadBtn">⬇ Download This Ad as PNG</button>
      <span class="download-label" id="downloadLabel">ad_creative_01.png</span>
    </div>
  </div>

  <script>/* see § 6 */</script>
</body>
</html>
```

---

## 4. CSS Architecture

### 4.1 Brand Tokens — Imperious Beauty (Default)

```css
:root {
  /* Brand palette — Imperious Beauty By Dom */
  --brand-black:   #0A0A0A;
  --brand-gold:    #C9A84C;
  --brand-gold-lt: #E2C97E;
  --brand-white:   #FFFFFF;
  --brand-offwhite:#F5F0EB;
  --brand-gray:    rgba(255,255,255,0.55);

  /* Typography */
  --font-display:  "poppins", sans-serif;
  --font-body:     "liberation-sans", sans-serif;
  --font-serif:    "eskorte-latin", serif;

  /* Ad dimensions — 1:1 square default */
  --ad-size: 1080px;
}
```

> Override `:root` values at the top of every file's `<style>` block. These are not optional defaults — they are the confirmed client tokens.

### 4.2 Page Chrome

```css
*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }

html, body {
  background: #111;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 56px 20px 80px;
  font-family: var(--font-body);
}

.page-header { text-align: center; margin-bottom: 32px; }
.page-header h1 {
  font-family: var(--font-display);
  font-weight: 700;
  font-size: 13px;
  letter-spacing: 0.28em;
  text-transform: uppercase;
  color: var(--brand-gold);
}
.page-header p {
  font-size: 11px;
  color: rgba(255,255,255,0.3);
  letter-spacing: 0.12em;
  margin-top: 6px;
}
```

### 4.3 Ad Wrapper, Track & Slide Base

```css
.ad-viewer { position:relative; display:flex; flex-direction:column; align-items:center; gap:24px; }
.ad-counter { font-family:var(--font-body); font-size:11px; letter-spacing:0.18em; text-transform:uppercase; color:rgba(255,255,255,0.35); align-self:flex-end; }

.ad-wrapper {
  position: relative;
  width: min(var(--ad-size), 92vw);
  aspect-ratio: 1 / 1;
  overflow: hidden;
  border-radius: 2px;
  box-shadow: 0 40px 120px rgba(0,0,0,0.9), 0 0 0 1px rgba(255,255,255,0.06);
}

.ad-track {
  display: flex;
  width: 100%; height: 100%;
  transition: transform 0.5s cubic-bezier(0.77,0,0.175,1);
  will-change: transform;
}

.ad-slide {
  min-width: 100%; height: 100%;
  position: relative; overflow: hidden;
  display: flex; flex-direction: column;
}

.ad-bg {
  position: absolute; inset: 0;
  background-size: cover;
  background-position: center top;
}
```

### 4.4 Overlay Presets

These are the four confirmed overlay types. Pick the right one for each layout:

```css
/* Dark bottom-to-top — keeps upper photo clear, darkens text zone */
.ov-dark-bottom {
  position:absolute; inset:0;
  background: linear-gradient(180deg, rgba(0,0,0,0.08) 0%, rgba(0,0,0,0.22) 40%, rgba(0,0,0,0.82) 100%);
}

/* Split-top — darkens top for text, keeps bottom photo bright */
.ov-split-top {
  position:absolute; inset:0;
  background: linear-gradient(180deg, rgba(0,0,0,0.78) 0%, rgba(0,0,0,0.55) 42%, rgba(0,0,0,0.08) 100%);
}

/* Vignette — darkens edges, shows center clearly */
.ov-vignette {
  position:absolute; inset:0;
  background: radial-gradient(ellipse at center, rgba(0,0,0,0) 30%, rgba(0,0,0,0.68) 100%);
}

/* Black band — solid black rectangle, typically top 38–42% of slide */
.ov-black-band {
  position:absolute; top:0; left:0; right:0;
  height: 42%;
  background: var(--brand-black);
}
```

---

## 5. Typography System — LOCKED VALUES

These sizes were confirmed and approved by the client after review. **Do not reduce any of these values.**

### 5.1 Global Typography Classes

```css
/* ── EYEBROW — small uppercase tracking label ── */
.eyebrow {
  font-family: var(--font-body);
  font-weight: 700;
  font-size: clamp(12px, 1.6vw, 18px);       /* min: 12px | max: 18px */
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--brand-gold);
}

/* ── HEADLINE — primary impact text ── */
.headline {
  font-family: var(--font-display);             /* poppins bold */
  font-weight: 700;
  font-size: clamp(32px, 6.5vw, 82px);         /* base range */
  line-height: 1.0;
  text-transform: uppercase;
  color: var(--brand-white);
}
.headline em { color: var(--brand-gold); font-style: normal; }  /* gold emphasis */

/* ── SUBTEXT / BODY — supporting copy ── */
.subtext {
  font-family: var(--font-body);
  font-weight: 400;
  font-size: clamp(16px, 2.2vw, 26px);         /* min: 16px | max: 26px — NEVER SMALLER */
  color: var(--brand-gray);
  line-height: 1.5;
}

/* ── CTA PILL — solid gold button ── */
.cta-pill {
  display: inline-block;
  background: var(--brand-gold);
  color: var(--brand-black);
  font-family: var(--font-display);
  font-weight: 700;
  font-size: clamp(13px, 1.8vw, 20px);
  letter-spacing: 0.14em;
  text-transform: uppercase;
  padding: clamp(10px,1.6vw,18px) clamp(22px,3.5vw,44px);
  border-radius: 2px;
}

/* ── CTA GHOST — outlined gold button ── */
.cta-ghost {
  display: inline-block;
  border: 1.5px solid var(--brand-gold);
  color: var(--brand-gold);
  font-family: var(--font-display);
  font-weight: 700;
  font-size: clamp(13px, 1.8vw, 20px);
  letter-spacing: 0.14em;
  text-transform: uppercase;
  padding: clamp(10px,1.6vw,18px) clamp(22px,3.5vw,44px);
  border-radius: 2px;
}

/* ── GOLD DIVIDER ── */
.gold-divider { width: 44px; height: 2px; background: var(--brand-gold); border-radius:1px; }

/* ── URL TAG — domain/location in gold ── */
.url-tag {
  font-family: var(--font-body);
  font-weight: 400;
  font-size: clamp(11px, 1.5vw, 16px);
  letter-spacing: 0.18em;
  color: var(--brand-gold);
  text-transform: lowercase;
  opacity: 0.85;
}
```

### 5.2 Headline Size Variants Per Layout

| Ad Type | Headline Font Size |
|---|---|
| Black-band-top hero | `clamp(36px, 7.5vw, 96px)` |
| Full-bleed bottom text | `clamp(28px, 5.8vw, 72px)` |
| Full-bleed centered minimal | `clamp(38px, 8vw, 100px)` |
| Grid collage bottom band | `clamp(20px, 3.8vw, 48px)` |
| Process/artisan layout | `clamp(34px, 7vw, 88px)` |
| Portrait minimal | `clamp(32px, 6.5vw, 82px)` |

---

## 6. Logo Handling

### 6.1 Real Logo (Preferred)

```css
.ad-logo { position: absolute; z-index: 10; }
.ad-logo.top-left  { top:5%;  left:5%; }
.ad-logo.top-right { top:5%;  right:5%; }
.ad-logo.bot-left  { bottom:5%; left:5%; }
.ad-logo.bot-right { bottom:5%; right:5%; }

.brand-mark {
  width: clamp(140px, 20vw, 240px);    /* APPROVED SIZE — do not reduce */
  height: auto;
  display: block;
  filter: brightness(0) invert(1) drop-shadow(0 2px 8px rgba(0,0,0,0.65));
}
```

```html
<div class="ad-logo top-right">
  <img src="./images/logo.png" class="brand-mark" alt="[Brand Name]">
</div>
```

> The logo is dark-on-transparent PNG. `brightness(0) invert(1)` converts it to white for dark backgrounds. The drop-shadow ensures it reads on any image.

### 6.2 SVG Wordmark Fallback (Only if no logo file)

```html
<svg class="brand-mark" viewBox="0 0 320 60" xmlns="http://www.w3.org/2000/svg">
  <text x="0" y="38" font-family="poppins,sans-serif" font-weight="700"
        font-size="28" letter-spacing="6" fill="white" opacity="0.92">IMPERIOUS</text>
  <text x="4" y="58" font-family="poppins,sans-serif" font-weight="300"
        font-size="18" letter-spacing="10" fill="#C9A84C" opacity="0.9">beauty</text>
</svg>
```

---

## 7. Layout Mode Library

Every ad slide must use one of these seven layout modes. Choose based on image content, copy, and variety across the batch.

### MODE A — Black Band Top + Full-Bleed Image Bottom
**Best for:** Hero ads, impactful single-phrase headlines, "Wake Up Like This" energy.
**Image type:** Confident client portrait, strong after result showing face/expression.

```html
<div class="ad-slide ad-XX">
  <div class="ov-black-band"></div>   <!-- solid black top 40% -->
  <div class="ad-bg" style="background-image:url('./images/photo.jpg');"></div>  <!-- positioned below band -->
  <div class="ov-dark-bottom" style="top:38%; height:62%;"></div>  <!-- darken bottom of photo -->

  <!-- Text in black band -->
  <div class="top-content">
    <span class="eyebrow">METAIRIE, LA · PERMANENT MAKEUP</span>
    <div class="headline">WAKE UP<br>LIKE <em>THIS.</em></div>
    <div class="gold-divider" style="margin:4px auto;"></div>
    <p class="subtext" style="max-width:78%;">Custom permanent brows in Metairie, LA.</p>
  </div>

  <div class="ad-logo bot-right">
    <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
  </div>
  <div class="url-tag" style="position:absolute;bottom:4%;left:50%;transform:translateX(-50%);z-index:11;">ImperiousBeautyByDom.com</div>
</div>
```

```css
.ad-XX .ov-black-band { height: 40%; }
.ad-XX .ad-bg { top: 38%; height: 62%; background-position: center top; }
.ad-XX .top-content {
  position:absolute; top:0; left:0; right:0; height:40%;
  display:flex; flex-direction:column; align-items:center; justify-content:center;
  gap: clamp(6px,1.2vw,14px); z-index:5; padding: 0 8%; text-align:center;
}
.ad-XX .top-content .headline { font-size: clamp(36px, 7.5vw, 96px); }
.ad-XX .ad-logo.bot-right { bottom: 3%; right: 4%; }
```

---

### MODE B — Full-Bleed + Dark Gradient Bottom Text Zone
**Best for:** "No Pencil. No Powder." precision messaging. Close-up result photos where the upper portion is the visual focus.
**Image type:** Sharp brow close-up, clean macro result shot.

```html
<div class="ad-slide ad-XX">
  <div class="ad-bg" style="background-image:url('./images/photo.jpg'); background-position: center 30%;"></div>
  <div class="ov-dark-bottom"></div>

  <div class="ad-logo top-right">
    <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
  </div>

  <div class="bottom-content">
    <span class="eyebrow">PERMANENT MAKEUP · BY DOM</span>
    <div class="headline">NO PENCIL.<br>NO POWDER.<br><em>JUST PERFECT BROWS.</em></div>
    <div class="gold-divider"></div>
    <p class="subtext">Permanent makeup designed for your face shape.</p>
    <div style="margin-top:8px;"><span class="cta-pill">NEW ORLEANS AREA · BOOK NOW</span></div>
  </div>
</div>
```

```css
.ad-XX .bottom-content {
  position:absolute; bottom:0; left:0; right:0;
  padding: clamp(16px,3vw,40px) clamp(20px,5%,54px) clamp(20px,4vw,44px);
  z-index:5; display:flex; flex-direction:column; gap: clamp(6px,1vw,12px);
}
.ad-XX .bottom-content .headline { font-size: clamp(28px, 5.8vw, 72px); line-height:0.95; }
```

---

### MODE C — 2×2 Photo Grid Collage + Bottom Band
**Best for:** Social proof. "Real Clients. Real Results." Multiple transformations shown simultaneously.
**Image type:** Use 4 DIFFERENT client photos — mix of close-ups, portraits, and result shots.

```html
<div class="ad-slide ad-XX">
  <div class="photo-grid">
    <div class="photo-cell" style="background-image:url('./images/photo_a.jpg'); background-size:cover; background-position:center 30%;"></div>
    <div class="photo-cell" style="background-image:url('./images/photo_b.jpg'); background-size:cover; background-position:center 20%;"></div>
    <div class="photo-cell" style="background-image:url('./images/photo_c.jpg'); background-size:cover; background-position:center 30%;"></div>
    <div class="photo-cell" style="background-image:url('./images/photo_d.jpg'); background-size:cover; background-position:center 20%;"></div>
  </div>
  <div class="bottom-band">
    <div style="display:flex; flex-direction:column; gap:4px;">
      <div class="headline">REAL CLIENTS.<br><em>REAL RESULTS.</em></div>
      <p class="subtext" style="font-size:clamp(13px,1.8vw,22px);">Permanent brows custom-shaped for every face.</p>
    </div>
    <div class="ad-logo" style="position:relative;">
      <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
    </div>
  </div>
</div>
```

```css
.ad-XX .photo-grid {
  position:absolute; inset:0; display:grid;
  grid-template-columns: 1fr 1fr; grid-template-rows: 1fr 1fr; gap: 3px;
}
.ad-XX .photo-cell { overflow:hidden; background:#111; background-size:cover; background-position:center; }
.ad-XX .bottom-band {
  position:absolute; bottom:0; left:0; right:0;
  background: var(--brand-black);
  padding: clamp(14px,2.5vw,30px) clamp(20px,5%,54px);
  z-index:5; display:flex; align-items:center; justify-content:space-between; gap:12px;
}
.ad-XX .bottom-band .headline { font-size: clamp(20px, 3.8vw, 48px); text-align:left; }
```

---

### MODE D — Full-Bleed + Split-Top Gradient (Text at Top)
**Best for:** Process/artisan shots. "This Is The Work." Trust-building. Shows expertise.
**Image type:** Procedure in progress — hands, tools, close-up of the artisan at work.

```html
<div class="ad-slide ad-XX">
  <div class="ad-bg" style="background-image:url('./images/photo.jpg'); background-position: center 40%;"></div>
  <div class="ov-split-top"></div>

  <div class="top-left-block">
    <span class="eyebrow">THE ARTISAN PROCESS</span>
    <div class="headline"><em>THIS IS</em><br>THE WORK.</div>
    <div class="gold-divider"></div>
    <p class="subtext" style="max-width:72%;">Hand-crafted permanent brows,<br>stroke by stroke.</p>
    <span class="cta-ghost" style="margin-top:4px; align-self:flex-start;">IMPERIOUS BEAUTY BY DOM</span>
  </div>

  <div class="ad-logo bot-right">
    <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
  </div>
</div>
```

```css
.ad-XX .top-left-block {
  position:absolute; top:0; left:0; right:0;
  padding: clamp(20px,4vw,50px) clamp(20px,5%,54px);
  z-index:5; display:flex; flex-direction:column; gap: clamp(8px,1.2vw,16px);
}
.ad-XX .top-left-block .headline { font-size: clamp(34px, 7vw, 88px); }
```

---

### MODE E — Full-Bleed Minimal Center/Bottom Overlay
**Best for:** Lifestyle results, powder brows, emotional transformation. "Soft. Defined. Yours."
**Image type:** Beautiful clean result photo — the face IS the hero; text is secondary.

```html
<div class="ad-slide ad-XX">
  <div class="ad-bg" style="background-image:url('./images/photo.jpg'); background-position: center 25%;"></div>
  <div class="ov-dark-bottom"></div>
  <div class="ov-vignette"></div>

  <!-- Optional price/offer badge top-right -->
  <div class="price-pill">POWDER BROWS<br>STARTING AT $400</div>

  <div class="center-overlay">
    <div class="headline">SOFT.<br><em>DEFINED.</em> YOURS.</div>
    <div class="gold-divider" style="margin:0 auto;"></div>
    <p class="subtext">Ombré powder brows that look naturally you.</p>
    <span class="cta-pill">METAIRIE, LA · BOOK NOW</span>
  </div>

  <div class="ad-logo bot-left">
    <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
  </div>
</div>
```

```css
.ad-XX .center-overlay {
  position:absolute; bottom:6%; left:0; right:0; z-index:5;
  display:flex; flex-direction:column; align-items:center;
  gap: clamp(6px,1vw,12px); text-align:center; padding: 0 10%;
}
.ad-XX .center-overlay .headline { font-size: clamp(30px, 6.5vw, 82px); }
.ad-XX .price-pill {
  position:absolute; top:5%; right:5%; z-index:10;
  background: var(--brand-gold); color: var(--brand-black);
  font-family: var(--font-body); font-weight:700;
  font-size: clamp(9px,1.2vw,13px); letter-spacing:0.14em;
  text-transform:uppercase; padding: 6px 14px; border-radius:2px;
  text-align:center; line-height:1.5;
}
```

---

### MODE F — Full-Bleed Bottom Stack (Centered)
**Best for:** Local identity ads, lip blush, services that need an emotional close. "Your Lips. Perfected."
**Image type:** Multi-face results, lifestyle confidence, lip/eye close-up result.

```html
<div class="ad-slide ad-XX">
  <div class="ad-bg" style="background-image:url('./images/photo.jpg'); background-position: center 30%;"></div>
  <div class="ov-dark-bottom"></div>
  <div class="ov-vignette"></div>

  <div class="ad-logo top-right">
    <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
  </div>

  <div class="center-content">
    <span class="eyebrow">LIP BLUSH · NEW ORLEANS METRO</span>
    <div class="headline">YOUR LIPS.<br><em>PERFECTED.</em></div>
    <div class="gold-divider" style="margin:0 auto;"></div>
    <p class="subtext">Permanent lip blush for fullness and color every day.</p>
    <span class="cta-pill">NEW ORLEANS METRO · IMPERIOUS BEAUTY</span>
  </div>
</div>
```

```css
.ad-XX .center-content {
  position:absolute; inset:0; z-index:5;
  display:flex; flex-direction:column; align-items:center; justify-content:flex-end;
  padding: 0 8% clamp(24px,5vw,60px);
  gap: clamp(6px,1vw,12px); text-align:center;
}
.ad-XX .center-content .headline { font-size: clamp(38px, 8vw, 100px); }
```

---

### MODE G — Full-Bleed + Black Footer Strip + Top Text
**Best for:** Before/after hero ad, community identity ("New Orleans Women Deserve Better Brows").
**Image type:** Before/after lifestyle split photo where you want to show the full result context.

```html
<div class="ad-slide ad-XX">
  <div class="ad-bg" style="background-image:url('./images/photo.jpg');"></div>
  <div class="ov-split-top"></div>
  <div class="ov-dark-bottom" style="opacity:0.6;"></div>

  <div class="headline-block">
    <span class="eyebrow">PERMANENT ARTISTRY · METAIRIE, LA</span>
    <div class="headline" style="text-align:center;"><em>NEW ORLEANS WOMEN</em><br>DESERVE BETTER BROWS.</div>
    <div class="gold-divider" style="margin:0 auto;"></div>
  </div>

  <div class="footer-strip">
    <div style="display:flex; flex-direction:column; gap:2px;">
      <p class="subtext" style="font-size:clamp(13px,1.8vw,22px);">Permanent makeup. Premium artistry. Metairie, LA.</p>
      <span class="url-tag">ImperiousBeautyByDom.com</span>
    </div>
    <div class="ad-logo" style="position:relative;">
      <img src="./images/logo.png" class="brand-mark" alt="[Brand]">
    </div>
  </div>
</div>
```

```css
.ad-XX .headline-block {
  position:absolute; top:5%; left:0; right:0; z-index:5;
  display:flex; flex-direction:column; align-items:center;
  gap: clamp(6px,1vw,12px); text-align:center; padding: 0 8%;
}
.ad-XX .headline-block .headline { font-size: clamp(28px, 5.8vw, 72px); }
.ad-XX .footer-strip {
  position:absolute; bottom:0; left:0; right:0;
  background: var(--brand-black);
  padding: clamp(10px,2vw,22px) clamp(20px,5%,54px);
  z-index:5; display:flex; align-items:center; justify-content:space-between;
}
```

---

## 8. Navigation Shell CSS & Nav Row

```css
.nav-row { display:flex; align-items:center; justify-content:center; gap:20px; width:100%; }

.nav-btn {
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.14);
  color: white; width:42px; height:42px; border-radius:50%;
  display:flex; align-items:center; justify-content:center;
  cursor:pointer; transition: background 0.2s, border-color 0.2s; flex-shrink:0;
}
.nav-btn:hover { background: var(--brand-gold); border-color: var(--brand-gold); }
.nav-btn svg { width:18px; height:18px; }

.ad-dots { display:flex; gap:8px; align-items:center; }
.dot {
  width:7px; height:7px; border-radius:50%;
  background: rgba(255,255,255,0.2); cursor:pointer;
  transition: background 0.3s, transform 0.3s;
}
.dot.active { background: var(--brand-gold); transform: scale(1.5); }

.download-row { display:flex; align-items:center; gap:12px; }
.download-btn {
  background: var(--brand-gold); border:none; color:var(--brand-black);
  font-family: var(--font-display); font-size:11px; font-weight:700;
  letter-spacing:0.14em; text-transform:uppercase;
  padding: 12px 28px; border-radius:3px; cursor:pointer;
  display:flex; align-items:center; gap:8px;
  transition: opacity 0.2s, transform 0.2s;
}
.download-btn:hover { opacity:0.88; transform:translateY(-1px); }
.download-label { font-size:11px; color:rgba(255,255,255,0.25); letter-spacing:0.1em; }
```

---

## 9. JavaScript — Navigation, Touch, Keyboard & PNG Export

```javascript
(function () {
  const track       = document.getElementById('adTrack');
  const dotsWrap    = document.getElementById('adDots');
  const counterEl   = document.getElementById('adCounter');
  const downloadBtn = document.getElementById('downloadBtn');
  const downloadLbl = document.getElementById('downloadLabel');
  const btnPrev     = document.getElementById('btnPrev');
  const btnNext     = document.getElementById('btnNext');
  const slides      = track.querySelectorAll('.ad-slide');
  const total       = slides.length;
  let   current     = 0;
  let   exporting   = false;

  /* Build dots */
  slides.forEach((_, i) => {
    const d = document.createElement('div');
    d.className = 'dot' + (i === 0 ? ' active' : '');
    d.addEventListener('click', () => goTo(i));
    dotsWrap.appendChild(d);
  });

  function getDots() { return dotsWrap.querySelectorAll('.dot'); }

  function update() {
    const num = (current + 1).toString().padStart(2, '0');
    counterEl.textContent = `Ad ${current + 1} / ${total}`;
    downloadLbl.textContent = `ad_creative_${num}.png`;
    getDots().forEach((d, i) => d.classList.toggle('active', i === current));
  }

  function goTo(index) {
    current = (index + total) % total;
    track.style.transform = `translateX(-${current * 100}%)`;
    update();
  }

  function next() { goTo(current + 1); }
  function prev() { goTo(current - 1); }

  btnNext.addEventListener('click', next);
  btnPrev.addEventListener('click', prev);

  /* Touch / swipe */
  let touchX = 0;
  track.addEventListener('touchstart', e => { touchX = e.touches[0].clientX; }, { passive: true });
  track.addEventListener('touchend', e => {
    const d = touchX - e.changedTouches[0].clientX;
    if (Math.abs(d) > 40) d > 0 ? next() : prev();
  });

  /* Keyboard */
  document.addEventListener('keydown', e => {
    if (e.key === 'ArrowRight') next();
    if (e.key === 'ArrowLeft')  prev();
  });

  /* Mouse drag */
  let mouseX = 0, dragging = false;
  track.addEventListener('mousedown', e => { mouseX = e.clientX; dragging = true; });
  track.addEventListener('mouseup', e => {
    if (!dragging) return; dragging = false;
    const d = mouseX - e.clientX;
    if (Math.abs(d) > 40) d > 0 ? next() : prev();
  });
  track.addEventListener('mouseleave', () => { dragging = false; });

  /* PNG download with html2canvas */
  const svgIcon = `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>`;
  downloadBtn.innerHTML = svgIcon + ' Download This Ad as PNG';

  downloadBtn.addEventListener('click', () => {
    if (exporting) return;
    exporting = true;
    downloadBtn.textContent = 'Exporting…';

    const slide = slides[current];
    const num   = (current + 1).toString().padStart(2, '0');

    html2canvas(slide, {
      scale:           2,
      useCORS:         true,
      allowTaint:      true,
      backgroundColor: '#0A0A0A',
      logging:         false
    }).then(canvas => {
      const link    = document.createElement('a');
      link.download = `ad_creative_${num}.png`;
      link.href     = canvas.toDataURL('image/png');
      link.click();
    }).catch(err => {
      alert('Export failed: ' + err.message);
    }).finally(() => {
      exporting = false;
      downloadBtn.innerHTML = svgIcon + ' Download This Ad as PNG';
    });
  });

  update();
})();
```

---

## 10. Imperious Beauty — Client Design Preferences (Confirmed by Client)

These preferences were established and approved during Batch 01 (April 2026) and must be carried forward in all future runs.

### 10.1 Typography Preferences

| Element | Confirmed Value | Notes |
|---|---|---|
| Display font | `poppins` bold 700 | Headlines, CTA buttons, eyebrows |
| Body font | `liberation-sans` regular 400 | Subtext, labels, URL tags |
| Headline case | UPPERCASE always | Never sentence case for headlines |
| Gold emphasis | `<em>` tags → `color: var(--brand-gold)` | Used on 1–2 key words per headline |
| Subtext size | `clamp(16px, 2.2vw, 26px)` | **Client confirmed — was too small before** |
| Eyebrow size | `clamp(12px, 1.6vw, 18px)` | **Client confirmed — was too small before** |

### 10.2 Logo Preferences

| Element | Confirmed Value |
|---|---|
| Logo file | `WhatsApp Image 2026-04-02 at 12.32.06 PM.png` → copy as `logo.png` |
| Logo treatment | `filter: brightness(0) invert(1)` → white on dark backgrounds |
| Logo size | `clamp(140px, 20vw, 240px)` — **client confirmed; was too small at 90–160px** |
| Drop shadow | `drop-shadow(0 2px 8px rgba(0,0,0,0.65))` |

### 10.3 Color & Palette Preferences

| Token | Value | Usage |
|---|---|---|
| `--brand-black` | `#0A0A0A` | Solid bands, backgrounds, CTA text |
| `--brand-gold` | `#C9A84C` | Eyebrows, dividers, CTA pill backgrounds, gold `<em>` in headlines |
| `--brand-gold-lt` | `#E2C97E` | Secondary headline words, lighter gold accents |
| `--brand-white` | `#FFFFFF` | Headline text, logo, primary text on dark |
| `--brand-gray` | `rgba(255,255,255,0.55)` | Subtext / supporting body copy |

### 10.4 Layout Diversity Rules

Across any single batch of 10 ads, aim for this distribution:

| Layout Mode | Count | Reason |
|---|---|---|
| Mode A (Black-band-top) | 2 max | Overused if more — alternate with full-bleed |
| Mode B (Full-bleed dark bottom) | 2–3 | Versatile, works with most client photos |
| Mode C (Grid collage) | 1 | Social proof — 1 per batch is enough |
| Mode D (Split-top artisan) | 1 | Process/trust — 1 per batch |
| Mode E (Minimal centered) | 1–2 | Premium feel — lifestyle photos |
| Mode F (Full-bleed bottom stack) | 1 | Emotional/feature-specific ads |
| Mode G (Footer strip) | 1 | Local identity / before-after |

> **Rule:** Never put the same layout mode back-to-back on consecutive slides. Always alternate.

### 10.5 Image-to-Copy Pairing Logic

| Image Content | Best Copy Theme | Layout Mode |
|---|---|---|
| Before/after split photo | "BEFORE. AFTER. FOREVER." | G or A |
| Clean close-up brow result | "WAKE UP LIKE THIS." / "SO NATURAL." | A or B |
| Process/procedure hands shot | "THIS IS THE WORK." | D |
| Confident lifestyle portrait | Local identity copy | G |
| Multiple client photos | "REAL CLIENTS. REAL RESULTS." | C |
| Powder/ombré brow result | "SOFT. DEFINED. YOURS." | E |
| Lip result close-up | "YOUR LIPS. PERFECTED." | F |
| Portrait after (polished) | "YOUR BROWS. CUSTOM DESIGNED." | B or E |

### 10.6 Copy Tone Rules

- Headlines are **punchy, 2–5 words per line**, UPPERCASE, minimal punctuation (periods only)
- Gold `<em>` emphasis goes on the most emotional or differentiating word (e.g., *This.*, *Real.*, *Perfected.*)
- Subtext is **1–2 short sentences max** — supporting the headline, not repeating it
- CTAs should include **location** ("Metairie, LA" or "New Orleans Metro") to target local audience
- Eyebrows set category + location: `PERMANENT MAKEUP · METAIRIE, LA`

### 10.7 What NOT to Do

- ❌ Do NOT use generic beauty industry colors (millennial pink, lavender, nude beige)
- ❌ Do NOT place text over the client's face, eyes, brows, or lips
- ❌ Do NOT use small logos — the client confirmed the logo was too small in Batch 01
- ❌ Do NOT use Google Fonts — always use the Typekit kit
- ❌ Do NOT repeat the same layout mode more than twice in a 10-ad batch
- ❌ Do NOT use sentence case headlines — always UPPERCASE
- ❌ Do NOT put the same 2 layouts back-to-back on consecutive slides

---

## 11. Batch Evolution Framework

Every run must be BETTER than the previous one. Use this framework:

### 11.1 What to review from previous batches
1. Which layouts were used (count per mode)
2. Which image-to-copy pairings were made
3. Where the logo was placed per slide
4. Typography scale used

### 11.2 What to improve each run
1. **Layout variety:** Reduce overused modes, introduce underused modes
2. **Image variety:** If new images are added by client, prioritize them
3. **Copy freshness:** Avoid verbatim repeat headlines; rephrase or rotate angles
4. **Overlay sophistication:** Try new gradient combinations while maintaining legibility
5. **Hierarchy escalation:** Make headlines feel more commanding with each batch

### 11.3 What to preserve each run
1. Typography scale (confirmed sizes)
2. Brand color tokens
3. Typekit font stack
4. Logo size and CSS filter
5. CTA style (gold pill or ghost outlined)

---

## 12. Platform Copy Length Rules & Generation

> **IMPORTANT:** Always use the **Ad Copywriter Skill** (`ad_copywriter_skill.md`) to generate the platform-specific copy (Meta primary text, Google RSA, etc.) that accompanies these creatives. The copywriter skill contains the exact frameworks, psychological triggers, and output formats required.

| Platform | Element | Max Characters |
|---|---|---|
| Meta Feed | Headline (on image) | 40 chars |
| Meta Feed | Body text | 125 chars |
| Meta Feed | CTA button label | 25 chars |
| Google Display | Short headline | 30 chars |
| Google Display | Long headline | 90 chars |
| Google Display | Description | 90 chars |

---

## 13. Default Brand Tokens — Fallback (Non-Client-Specific)

```css
:root {
  --brand-primary:    #0F0F10;
  --brand-accent:     #C9A84C;    /* gold as default accent */
  --brand-surface:    #1A1A1D;
  --brand-text:       #FFFFFF;
  --brand-subtext:    rgba(255,255,255,0.55);
  --brand-cta-bg:     #C9A84C;
  --brand-cta-text:   #0A0A0A;
  --font-display:     "poppins", sans-serif;
  --font-body:        "liberation-sans", sans-serif;
  --ad-size:          1080px;
}
```
