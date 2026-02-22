# Digital Card System — Planning & Research

## Current State

Two cards have been built so far, both as standalone HTML files with embedded styles, images, and animations:

| Card | File | Event | Design |
|------|------|-------|--------|
| **Ayana Date #1** | `john_ayana_first_date.html` | 3-stop date (Chartreuse, DIA Film, Dragonfly) | Multi-color stops with numbered badges, green/gray/red themes |
| **Chelsea Night** | `chelsea-night.html` | The Common Grill + Purple Rose Theatre | Purple/gold dual-section card with falling emoji animation |

### Shared Design DNA

Both cards already share organic conventions worth formalizing:

- **Typography**: Playfair Display (headings), Lora (body), DM Sans (accents)
- **Container**: `max-width: 520px`, centered, with card sections inside
- **Texture**: SVG noise overlay at 3% opacity via `body::after`
- **Animation**: `fadeUp` entrance animation with staggered delays
- **Responsiveness**: Single breakpoint at `560px`
- **Palette approach**: Warm, editorial color schemes per event theme
- **Images**: Embedded base64 AVIF for portability

---

## Upcoming Cards

### 1. Ayana — Date #2
- Second date card for Ayana
- Likely follows the multi-stop structure from the first date card
- Details TBD (venues, theme)

### 2. Alleghenies Summer Cabin Trip
- Group trip to the Allegheny Mountains (summer 2026)
- Renting a cabin/lodge/house with friends
- Card serves as an invitation/info-share for the group
- Likely needs: location details, dates, logistics, maybe a map reference
- Different tone — group event vs. intimate date

### 3. Future Cards (Ongoing Practice)
- Goal: make card creation a regular practice for events, dates, invitations
- Need a system that makes spinning up new cards fast and consistent

---

## Task 1: Card Size Standards & Reusable Framework

### Goal

Extract shared code into a reusable foundation and adopt a standard print-friendly card size so these cards work well on screens *and* could be printed or saved as PDFs.

### Recommended Card Size: 5" x 7" (A7)

The **5x7** format is the strongest choice for a dual-purpose digital/printable card:

| Property | Value |
|----------|-------|
| Dimensions | 5" x 7" (127mm x 178mm) |
| Pixels at 300 DPI (print) | 1500 x 2100 px |
| Pixels at 96 DPI (screen) | 480 x 672 px |
| Aspect ratio | 5:7 (~0.714) |
| Envelope | A7 (5.25" x 7.25") — widely available |

**Why 5x7:**
- Most universally recognized card size (greeting cards, invitations, photo prints)
- Works beautifully on portrait phone screens (close to mobile aspect ratios)
- Envelopes sold everywhere if you ever want to mail one
- Every print service supports it (Shutterfly, Vistaprint, Minted, etc.)
- CSS `@page { size: 5in 7in; }` is well-supported across browsers

**Alternative sizes to keep in mind:**

| Size | Dimensions | Pixels @300 DPI | Good For |
|------|-----------|-----------------|----------|
| Postcard (4x6) | 4" x 6" | 1200 x 1800 | Smaller/simpler cards |
| A2 Greeting | 4.25" x 5.5" | 1275 x 1650 | Thank-you notes |
| Square | 5" x 5" | 1500 x 1500 | Modern/social media feel |
| Gift card (CR80) | 3.375" x 2.125" | 1013 x 638 | Too small for these |

### What to Extract

**Phase 1 — Shared CSS base (`card-base.css` or embedded foundation)**
- Typography system (font imports, size scale, weight mappings)
- Card container structure (max-width, centering, padding)
- Noise texture overlay
- FadeUp animation + stagger utility classes
- Print media queries with `@page { size: 5in 7in; }`
- Responsive breakpoint
- Hover transitions

**Phase 2 — Card template**
- An HTML skeleton/template that new cards can be copied from
- Slots for: header, sections/stops, dividers, footer
- Color theme variables via CSS custom properties (easy to swap palettes)
- Optional: numbered badge system, button/link styles

**Phase 3 — Per-card customization layer**
- Color palette (CSS custom properties)
- Images (embedded or linked)
- Event-specific animations (like Chelsea's falling emoji)
- Content

### CSS Print Support

```css
@media print {
  @page {
    size: 5in 7in;
    margin: 0;
  }

  body {
    background: none;
    margin: 0;
  }

  .card-container {
    box-shadow: none;
    max-width: 100%;
    width: 100%;
    height: 100%;
  }

  .noise-overlay,
  .falling,
  .no-print {
    display: none !important;
  }

  * {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
}
```

The `@page { size }` property reached Baseline status in December 2024 and is supported in Chrome 15+, Edge 79+, Firefox 116+, and Safari 17+.

---

## Task 2: Better iMessage Sharing

### The Problem

Currently, HTML files are sent directly over iMessage. iPhones render them in a file browser UI that:
- Shows a bulky chrome/toolbar that can't be hidden
- Feels like viewing a document, not receiving a card
- Email doesn't reliably render the HTML either

### Approaches Evaluated

#### A. Host as Web Pages + Rich Link Previews *(Recommended — Start Here)*

**How it works:** Deploy cards as web pages (GitHub Pages, Vercel, Netlify). Add Open Graph meta tags. Share the URL over iMessage.

```html
<meta property="og:title" content="Saturday Night in Chelsea" />
<meta property="og:image" content="https://cards.example.com/chelsea/preview.png" />
<meta property="og:description" content="Dinner & Theatre — Feb 28" />
```

**What the recipient sees:**
1. A **rich link preview card** in the iMessage bubble (image + title + description)
2. Tap → opens full animated HTML card in Safari (full screen, no file browser)

**Effort:** Very low — just deploy existing HTML files and add meta tags
**Apple Developer Account:** Not needed
**Animations preserved:** Yes, on tap (Safari). The bubble preview is a static image.
**Recipient needs app:** No

**Enhancement — video previews:**
```html
<meta property="og:video" content="https://cards.example.com/chelsea/preview.mp4" />
<meta property="og:video:type" content="video/mp4" />
```
iMessage supports autoplaying, looping MP4 video in the link preview bubble. You could pre-render the card animation as a short MP4 for a motion preview right in the conversation.

#### B. Pre-Rendered MP4 Video *(Best In-Bubble Experience)*

**How it works:** Use Puppeteer/Playwright to screen-record the HTML animation as an MP4. Send the video directly in iMessage or use it as the `og:video` for link previews.

**What the recipient sees:** The animation plays *inline in the conversation* — no tap needed.

**Tools:** Puppeteer `page.screencast()`, Playwright video recording, Remotion (React-based), html2canvas + ccapture.js

**Effort:** Low-medium (one-time pipeline setup)
**Animations preserved:** Yes — the video *is* the animation
**Limitation:** Not interactive (it's a video, not live HTML). Lossy compression.

#### C. iMessage App Extension *(Not Recommended Yet)*

**How it works:** Build a native iOS extension using Apple's Messages framework. The extension UI can include a WKWebView that renders HTML cards.

**Catch:** The message bubble itself uses Apple's fixed `MSMessageTemplateLayout` — you get an image, a caption, and a subcaption. No custom HTML in the bubble. Full HTML only shows when the recipient *taps to expand* the extension. **Both sender and recipient need the extension installed.**

**Effort:** High (native iOS dev, Xcode, Swift)
**Apple Developer Account:** $99/year
**Not recommended** unless the card practice scales to something much larger.

#### D. App Clips *(Future Upgrade Path)*

**How it works:** Build a lightweight App Clip (< 50MB) that wraps a WKWebView. When a configured URL is shared in iMessage, iOS shows an App Clip card. Recipient taps → instantly opens the clip without installing anything.

**Effort:** High (full iOS app + App Clip target, App Store Connect config, App Store review)
**Apple Developer Account:** $99/year
**Best for:** If you eventually want a branded, app-like card experience without requiring installs

#### E. PWA *(Not Viable for iMessage)*

PWAs cannot integrate with iMessage in any meaningful way. They can use the Web Share API to *send* URLs but cannot register as share targets or appear in the Messages app. Not a viable path.

#### F. iOS Shortcuts *(Minimal Benefit)*

A Shortcut could open a hosted card URL in Safari or compose an iMessage with the link. But this is essentially just a URL launcher — no richer than sharing the link yourself.

### Recommended Strategy

**Start with Approach A** — it's nearly zero effort and dramatically improves the experience:

1. **Host cards** on GitHub Pages, Vercel, or Netlify (free tiers available)
2. **Add OG meta tags** to each card for rich link previews
3. **Generate a static preview image** (screenshot of the card) for `og:image`
4. **Share the URL** over iMessage instead of the raw HTML file

**Then optionally add Approach B** for cards where the animation is the star:

5. **Pre-render animation as MP4** using Puppeteer/Playwright
6. **Use as `og:video`** for autoplay motion preview in the iMessage bubble
7. **Or send the MP4 directly** for maximum visual impact

**Defer C/D** (native iOS work) unless the practice grows significantly.

### Comparison Matrix

| Approach | Effort | In-Bubble Animation | Full Animation on Tap | Needs App Install | Needs Dev Account |
|----------|--------|---------------------|----------------------|-------------------|-------------------|
| **A. Hosted + OG tags** | Very Low | Static image (or MP4 loop) | Yes (Safari) | No | No |
| **B. Pre-rendered MP4** | Low-Med | Yes (inline video) | N/A | No | No |
| **C. iMessage Extension** | High | No (static template) | Yes (in extension) | Yes (both sides) | $99/yr |
| **D. App Clips** | High | N/A | Yes (in clip) | No | $99/yr |

---

## Proposed Roadmap

### Near-Term (Next 2-3 Cards)

- [ ] **Extract shared CSS** into a base stylesheet or template
- [ ] **Define CSS custom properties** for theming (colors, fonts)
- [ ] **Add `@page` print support** targeting 5x7
- [ ] **Create a card template** (HTML skeleton to copy for new cards)
- [ ] **Build Ayana Date #2 card** using the new template
- [ ] **Set up web hosting** (GitHub Pages or Vercel)
- [ ] **Add OG meta tags** to all cards
- [ ] **Generate preview images** for each card

### Medium-Term (Alleghenies Trip + Polish)

- [ ] **Build Alleghenies cabin trip invitation card**
- [ ] **Set up MP4 pre-rendering pipeline** (Puppeteer or Playwright)
- [ ] **Add `og:video` support** for animated previews in iMessage
- [ ] **Test rich link sharing** across iMessage, SMS, and email

### Long-Term (If Practice Scales)

- [ ] Consider a simple **card gallery/index page** for all cards
- [ ] Evaluate **App Clips** if native experience becomes important
- [ ] Explore **custom domain** (e.g., `cards.yourdomain.com`)
- [ ] Consider a **card generator** tool/script for faster creation

---

## Reference: Key Dimensions

| Format | Inches | Pixels @300 DPI | Aspect Ratio |
|--------|--------|-----------------|--------------|
| **5x7 Card (A7)** | 5 x 7 | 1500 x 2100 | 5:7 |
| **4x6 Postcard** | 4 x 6 | 1200 x 1800 | 2:3 |
| **A2 Greeting** | 4.25 x 5.5 | 1275 x 1650 | ~1:1.29 |
| **Square** | 5 x 5 | 1500 x 1500 | 1:1 |
| **OG Preview Image** | — | 1200 x 630 | ~1.91:1 |
| **Current card max-width** | — | 520px CSS | Scrollable |
