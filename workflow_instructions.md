# Workflow Instructions — Ad Creative Generator SOP
> **This is your Standard Operating Procedure.**
> Every time an ad creative batch is requested, execute these steps IN ORDER — zero skips, zero shortcuts.
> Do not ask for clarification mid-run unless a required file is outright missing.

---

## ⚠️ MANDATORY PRE-RUN PROTOCOL — Do This Before Anything Else

Before parsing the brief or opening any content file, you MUST complete two mandatory intelligence-gathering phases. These are not optional. They directly shape every design decision in the current run.

### Phase A — Read All References

1. List all files in `/references/`.
2. Open and **fully read** each file in this order:
   - **`fonts.json`** first — extract Typekit kit ID, import tag, all font families, variant weights, and role assignments.
   - **`ad_creative_skills.md`** second — read in full; internalize all layout modes, typography scale, and client design preferences.
   - **`ad_copywriter_skill.md`** third — read in full; internalize the copywriting frameworks, psychological triggers, and platform-specific formats for Meta and Google.
   - **`design_system.json`** (if present) — extract brand color tokens.
   - **`Beauty_ad_creatives.json`** (if present) — extract competitor layout archetypes, dominant color patterns, and imagery strategies from `cross_creative_patterns`.
   - Any other `*.json` or `*.md` file in `/references/`.
3. After reading all references, compile a mental **Brand Intelligence Summary**:
   - Confirmed font families + their assigned roles
   - Color palette tokens
   - Layout archetypes available
   - Image strategies drawn from competitor analysis

### Phase B — Review All Previous Output Creatives

1. List all subfolders inside `/outputs/<ClientName>/` (brand-specific subfolder).
2. For **each output subfolder**, scan for any `.png` files (these are the user-approved final ad creatives the user manually exported and placed there).
3. Open and **visually review every PNG** found. For each one, note:
   - Layout type used (black-band-top, full-bleed, collage grid, etc.)
   - Copy placement (top, bottom, centered)
   - Logo size and position
   - Color/overlay approach
   - What worked well (what the user approved)
   - What could be improved (based on design principles)
4. After reviewing all previous outputs, compile a **Design History Log**:
   ```
   Batch 01 (imperious-beauty-batch-01):
   - 10 ads reviewed
   - Strong performance: black-band-top layouts with gold headline + white subtext
   - Logo placed bottom-right on image-bottom layouts, top-right on full-bleed layouts
   - Subtext at clamp(16px, 2.2vw, 26px) — approved size
   - Logo at clamp(140px, 20vw, 240px) — approved size (was too small initially)
   - Gold CTA pills performed well
   - Image-to-copy pairing was strategic (before-after → "BEFORE. AFTER. FOREVER.")
   ```
5. Use the Design History Log to:
   - **AVOID repeating** the exact same layouts from previous batches (vary it)
   - **PRESERVE** sizing, font choices, and color palette that were approved
   - **ESCALATE** quality with each new batch — every run should be better than the last

> **Rule:** If `/outputs/` contains previous PNG exports, the current batch MUST demonstrate visible design evolution. Do not simply copy the previous batch.

---

## Step 0 — Parse the Ad Brief

Extract from the user's request:

| Fact | Where to look | Default |
|---|---|---|
| **Campaign name / Client** | User prompt | Required — stop if missing |
| **Target platform** | "Google", "Meta", "both" | Meta |
| **Ad format / dimensions** | "feed square", "stories", "300×250" | Meta Feed Square (1080×1080) |
| **Campaign objective** | Awareness / Consideration / Conversion | Conversion |
| **Number of ad creatives** | Specified in prompt or content file header | Equal to number of content blocks in `/content/` |

> **Image mode is always ON.** Text-only fallback only if `/images/<ClientName>/` is completely empty.

---

## Step 1 — Determine Ad Format & Dimensions

```
Platform: Meta   + Format: Feed Square    →  1080 × 1080 px  (default)
Platform: Meta   + Format: Feed Portrait  →  1080 × 1350 px
Platform: Meta   + Format: Stories/Reels  →  1080 × 1920 px
Platform: Meta   + Format: Landscape      →  1200 × 628 px
Platform: Google + Format: Medium Rect    →  300 × 250 px
Platform: Google + Format: Half Page      →  300 × 600 px
Platform: Google + Format: Discovery/PMax →  1200 × 628 px
```

Set `--ad-w` and `--ad-h` CSS variables accordingly. If "both" platforms: generate two separate HTML files.

---

## Step 2 — Load Brand Style from `fonts.json` & Design References

*(Already read in Phase A — apply now)*

1. Use `typekit.import_tag` from `fonts.json` as the font `<link>` in `<head>`. **Never use Google Fonts.**
2. Apply `css_token_mapping.recommended_defaults` to `:root {}`:
   - `--font-display`: `"poppins", sans-serif`
   - `--font-body`: `"liberation-sans", sans-serif`
3. For Imperious Beauty specifically, also apply confirmed brand tokens:
   ```css
   --brand-black:   #0A0A0A;
   --brand-gold:    #C9A84C;
   --brand-gold-lt: #E2C97E;
   --brand-white:   #FFFFFF;
   --brand-gray:    rgba(255,255,255,0.55);
   ```
4. If a different client's `design_system.json` exists, override tokens from there.

---

## Step 2b — Logo Handling

1. Check for logo PNG/SVG in `/images/<ClientName>/` — look for any file named `logo`, `Logo`, or containing the brand name that is NOT a client photo.
2. **If logo file exists** (e.g., `WhatsApp Image 2026-04-02 at 12.32.06 PM.png` for Imperious Beauty):
   - Copy it into the output folder as `./images/logo.png`
   - Reference it as `<img src="./images/logo.png" class="brand-mark" alt="[Brand Name]">`
   - Apply CSS: `filter: brightness(0) invert(1) drop-shadow(0 2px 8px rgba(0,0,0,0.65));`
   - Size: `width: clamp(140px, 20vw, 240px); height: auto;` — **never smaller than this**
3. **If no logo exists:** Generate an inline SVG wordmark using `poppins` bold white + gold — see `ad_creative_skills.md § 11`.
4. Logo placement rules:
   - Black-band-top layouts → bottom-right corner of the image section
   - Full-bleed layouts → top-right or top-left corner (NEVER over face/brows/lips)
   - Collage/grid layouts → inside the bottom content band alongside headline
   - **The logo must appear on every single slide, consistently placed**

---

## Step 3 — Scan `/images/<ClientName>/` and Build the Image Index

1. List all files in `/images/<ClientName>/`.
2. Filter for `.jpg`, `.jpeg`, `.png`, `.webp` — exclude logo files.
3. **Visually review every image** — this is non-negotiable. You must see what each photo shows before assigning it to an ad. Note for each:
   - Is it a BEFORE or AFTER state?
   - Is it a close-up (brows/lips/eyes) or a portrait?
   - Is it a process/procedure shot (hands, tools)?
   - Is it a lifestyle/confidence shot?
   - Does it already contain a side-by-side split?
   - Does it already have overlaid text? (if so, can it still be used — will your overlay text conflict?)
4. Build the Image Assignment Map — matching each image to the ad copy it best serves:

```
Image Assignment Map (Imperious Beauty example):
  Before/after split photo    → AD with "BEFORE. AFTER. FOREVER."
  Close-up result (brows)     → AD with "WAKE UP LIKE THIS." or "SO NATURAL."
  Process shot (hands/tools)  → AD with "THIS IS THE WORK."
  Confident lifestyle client  → AD with local identity copy ("NEW ORLEANS WOMEN...")
  Multi-client collage        → AD with "REAL CLIENTS. REAL RESULTS."
  Tight brow close-up         → AD with "NO PENCIL. NO POWDER."
```

**If `/images/<ClientName>/` is empty:** STOP. Notify user. Do not proceed.

---

## Step 3b — Fallback: Text-Only Mode (No Images)

Only reached if image folder is empty. Use layout modes 5B, 5D, 5E from `ad_creative_skills.md`. Note in report.

---

## Step 4 — Parse Content File & Generate Platform Copy

1. List files in `/content/`. Select the one matching the campaign name.
2. Each AD block maps 1:1 to one image from the Image Assignment Map.
3. Extract per-slide: Eyebrow · Headline · Body/Subtext · Offer badge · CTA · Visual type note.
4. If slide counts differ — see error handling table.
5. **Ad Copy Generation:** Using the **Ad Copywriter Skill** (`ad_copywriter_skill.md`), generate the accompanying platform text (Meta Primary Text/Headline/Description OR Google RSA headlines/descriptions) for the campaign.
   - Base the copy on the parsed brief, the target audience, and the Image Assignment Map.
   - Follow the precise output formats and psychological triggers defined in the skill document.
   - Save this generated ad copy into a new file in `/outputs/<ClientName>/<batch-slug>/ad_copy.txt` alongside the creatives.

---

## Step 5 — Design Each Slide Individually

This is the most critical step. Do NOT apply a single template to all slides. Each slide must have its own layout decision based on:
- The image's content and composition
- The ad's copy strategy (hint from copywriter notes in the content file)
- The Design History Log (avoid repeating layouts from previous batches)
- The Imperious Beauty design preferences encoded in `ad_creative_skills.md § 10`

**For each slide, explicitly decide:**

| Decision | Options |
|---|---|
| Layout mode | Black-band-top / Full-bleed / Side-split / Grid-collage / Minimal-portrait |
| Text zone | Top / Bottom / Centered / Left-aligned |
| Overlay | Dark-bottom gradient / Split-top gradient / Vignette / Solid black band |
| Logo position | Top-right / Top-left / Bottom-right / Bottom-left / Inside band |
| Accent use | Gold headline only / Gold headline + gold CTA / Gold divider only |
| CTA style | Solid gold pill / Ghost border / Inline URL tag |

---

## Step 6 — Build the Slide Pairing Table

Before writing HTML, document the complete plan:

```
SLIDE PLAN:
──────────────────────────────────────────────────────────────
Slide 01 | Image: 569940701...n.jpg | Layout: Black-band-top + image-bottom
          | Eyebrow: METAIRIE, LA · PERMANENT MAKEUP
          | Headline: WAKE UP LIKE THIS.
          | Subtext: Custom permanent brows in Metairie, LA.
          | Logo: bottom-right | CTA: URL tag bottom-center

Slide 02 | Image: 573695026...n.jpg | Layout: Full-bleed + dark-bottom overlay
          | Eyebrow: PERMANENT MAKEUP · BY DOM
          | Headline: NO PENCIL. NO POWDER. JUST PERFECT BROWS.
          | Subtext: Permanent makeup designed for your face shape.
          | Logo: top-right | CTA: Gold pill

[... repeat for all N slides ...]
──────────────────────────────────────────────────────────────
```

This table is the single source of truth. Generate HTML only after it is complete.

---

## Step 7 — Generate `ad_creatives.html`

### File structure
```
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Client] — Ad Creatives [Batch]</title>
  <link rel="stylesheet" href="https://use.typekit.net/lld6qxp.css">  <!-- FROM fonts.json -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    /* 1. :root brand tokens */
    /* 2. Page chrome (body, .page-header, .ad-viewer) */
    /* 3. Ad wrapper, track, slide base */
    /* 4. Overlay presets (.ov-dark-bottom, .ov-split-top, .ov-vignette, .ov-black-band) */
    /* 5. Logo (.ad-logo, .brand-mark) */
    /* 6. Typography utilities (.eyebrow, .headline, .subtext, .cta-pill, .cta-ghost, .gold-divider, .url-tag) */
    /* 7. Per-ad layout classes (.ad-01 ... .ad-N) */
    /* 8. Nav UI (.nav-btn, .ad-dots, .dot) */
    /* 9. Download UI (.download-btn, .download-label) */
  </style>
</head>
<body>
  <div class="page-header">...</div>
  <div class="ad-viewer">
    <div class="ad-counter" id="adCounter">Ad 1 / N</div>
    <div class="ad-wrapper">
      <div class="ad-track" id="adTrack">
        <!-- N .ad-slide divs here -->
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
  <script>/* navigation + export JS */</script>
</body>
```

### Typography scale — LOCKED VALUES (do not reduce)

| Element | CSS value |
|---|---|
| Logo width | `clamp(140px, 20vw, 240px)` — minimum |
| Eyebrow | `clamp(12px, 1.6vw, 18px)` |
| Headline | `clamp(32px, 6.5vw, 82px)` base; up to `clamp(38px, 8vw, 100px)` for hero ads |
| Subtext / body | `clamp(16px, 2.2vw, 26px)` — **never below 16px** |
| CTA pill | `clamp(13px, 1.8vw, 20px)` with `padding: clamp(10px,1.6vw,18px) clamp(22px,3.5vw,44px)` |
| URL tag | `clamp(11px, 1.5vw, 16px)` |

### Validation checklist before saving:

- [ ] Every slide has a `background-image` from `./images/`
- [ ] Every slide has: eyebrow + headline + CTA (subtext optional but strongly preferred)
- [ ] All image paths are relative (`./images/...`) — never absolute
- [ ] Gradient overlay applied — no raw text directly on unprocessed photo
- [ ] Logo appears on every slide at correct size and position
- [ ] Logo uses `filter: brightness(0) invert(1)` on dark backgrounds
- [ ] No element overflows its container
- [ ] JS IDs: `adTrack`, `adDots`, `adCounter`, `btnPrev`, `btnNext`, `downloadBtn`, `downloadLabel`
- [ ] Typekit `<link>` tag in `<head>` (NOT Google Fonts)
- [ ] html2canvas CDN `<script>` in `<head>`
- [ ] Touch swipe, keyboard arrows, mouse drag all wired in JS
- [ ] Download generates correct filename: `ad_creative_01.png` ... `ad_creative_0N.png`

---

## Step 8 — Create Output Folder, Copy Assets & Save

1. Slug: `<batch-NN>` in lowercase-hyphens (e.g., `batch-01`, `batch-02`)
2. Brand folder: `/outputs/<ClientName>/` (use exact brand name as folder, e.g., `CLG_Canteen`, `Imperious_Beauty`)
3. Create: `/outputs/<ClientName>/<batch-slug>/`
4. Copy images: `Copy-Item "<workspace>/images/<ClientName>" "<workspace>/outputs/<ClientName>/<slug>/images" -Recurse -Force`
5. Copy logo separately if applicable: `Copy-Item "<workspace>/images/<ClientName>/logo-file.png" "<workspace>/outputs/<ClientName>/<slug>/images/logo.png" -Force`
6. Save HTML: `/outputs/<ClientName>/<slug>/ad_creatives.html`
7. Save Ad Copy: `/outputs/<ClientName>/<slug>/ad_copy.txt`

---

## Step 9 — Start Local HTTP Server & Provide Preview Link

1. Start Python HTTP server from output folder:
   ```powershell
   python -m http.server 8765 --directory "c:\...\outputs\<ClientName>\<slug>"
   ```
2. Run with `WaitMsBeforeAsync: 2000` (background).
3. Confirm running with `command_status`.
4. Provide: **`http://localhost:8765/ad_creatives.html`**

---

## Step 10 — Completion Report

```
✅ Pre-Run Intelligence:
   - References read: fonts.json, ad_creative_skills.md, ad_copywriter_skill.md, [design_system.json], [Beauty_ad_creatives.json]
   - Previous batches reviewed: [N batches, M total PNG exports]
   - Design evolution applied: [describe what changed vs previous batch]

✅ Campaign: <name>
✅ Platform: Meta Feed Square (1080×1080)
✅ Slides generated: N
✅ Output: /outputs/<ClientName>/<slug>/ad_creatives.html
✅ Copy Output: /outputs/<ClientName>/<slug>/ad_copy.txt
✅ Server: http://localhost:8765/ad_creatives.html

Layout breakdown:
  Slide 01 — Black-band-top + image bottom
  Slide 02 — Full-bleed + dark gradient overlay
  [...]

⚠️ Notes (if any):
  - [e.g., "Ad 03 reused same image as Ad 07 — recommend adding more source images next run"]
  - [e.g., "Previous batch used black-band-top for 4/10 ads — reduced to 2/10 in this batch for variety"]
```

---

## Error Handling

| Situation | Action |
|---|---|
| `/images/<ClientName>/` is empty | STOP — notify user that client image folder is required |
| `/references/` is empty | Use default brand tokens from `ad_creative_skills.md § 9` |
| `/outputs/` is empty | Skip Phase B; note "First run — no design history available" |
| Content file missing | STOP — provide content file format template |
| Image count < content blocks | Cycle images; note in report |
| Content blocks < image count | Generate minimal slides (logo + CTA) for unmatched images |
| Slide count below 5 | Duplicate with copy variants to reach minimum 5 |
| PNG export fails | Save HTML anyway; report failure separately |
| Previous batch PNGs not viewable | Skip that file; note it could not be reviewed |

---

## Quick Reference Checklist

```
PRE-RUN:
[ ] All files in /references/ read in full
[ ] fonts.json — Typekit tag, font roles extracted
[ ] ad_creative_skills.md — layout modes, preferences internalized
[ ] ad_copywriter_skill.md — copywriting frameworks internalized
[ ] /outputs/ scanned — all previous PNG exports reviewed
[ ] Design History Log compiled
[ ] Design evolution plan defined (what will be different this batch)

THIS RUN:
[ ] Brief parsed (campaign, platform, format, objective, slide count)
[ ] Ad dimensions set
[ ] /images/<ClientName>/ scanned — all images visually reviewed
[ ] Logo file identified and copied to output /images/logo.png
[ ] Output folder created at /outputs/<ClientName>/<batch-slug>/
[ ] Image Assignment Map built (image ↔ copy strategy)
[ ] Content file parsed from /content/
[ ] Platform Ad Copy generated using Ad Copywriter skill and saved to /outputs/
[ ] Slide Pairing Table complete (all N slides planned)
[ ] ad_creatives.html generated and validated
[ ] Output folder created and assets copied
[ ] Local HTTP server running
[ ] Preview link provided to user
[ ] Completion report delivered
```
