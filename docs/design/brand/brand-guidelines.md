# IDIOMAR — Brand Identity Guidelines

**Project:** CRM para Tutores de Idiomas Independientes (**idiom.ar**)  
**Authors:** Axel Ramos & Dmytro Cherednichenko  
**Document Type:** Visual Identity System & Design Specification  

Visual Assets:
* **Interactive HTML Board:** [branding-sheet.html](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/brand/branding-sheet.html)
* **Vector Graphic (SVG):** [branding-sheet.svg](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/brand/branding-sheet.svg)
* **Source Logo (SVG):** [logo.svg](file:///Users/dima/Documents/Programming/idiomar-crm/docs/design/brand/logo.svg)

---

## 1. Brand Concept & Visual Metaphor

**IDIOMAR** combines linguistic mastery with operational autonomy for independent language educators. The visual brand draws inspiration from modern European EdTech design (geometric simplicity, open typography, vibrant accents paired with grounding neutrals).

### The Compass Star Emblem
The core brand icon consists of four directional diamond facets arranged in a 360° star:
* **The Top Diamond (Valencia Flame `#FF7A30`):** Represents pedagogical enthusiasm, language acquisition progress, and the mentor’s guiding spark.
* **The Lateral & Base Diamonds (Obsidian Black `#000000`):** Represent structure, financial tracking, scheduled discipline, and operational stability.

---

## 2. Color Palette & Specifications

The IDIOMAR color system establishes an energetic yet calming visual hierarchy suitable for long hours of tutoring, scheduling, and vocabulary review.

| Swatch | Color Name | HEX | RGB | CMYK | Pantone | Semantic Role in UI |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ![#FF7A30](https://placehold.co/18x18/FF7A30/FF7A30.png) | **Valencia Flame** | `#FF7A30` | `255, 122, 48` | `0, 52, 81, 0` | `1585 C` | Primary CTA buttons, active state highlights, top logo facet, urgent alerts |
| ![#465C88](https://placehold.co/18x18/465C88/465C88.png) | **Oxford Slate** | `#465C88` | `70, 92, 136` | `49, 32, 0, 47` | `534 C` | Navigation headers, calendar time slots, secondary badges, CEFR level tags |
| ![#E9E3DF](https://placehold.co/18x18/E9E3DF/E9E3DF.png) | **Parchment Stone** | `#E9E3DF` | `233, 227, 223` | `0, 3, 4, 9` | `Warm Gray 1 C` | Global application canvas background, soft dividers, table borders |
| ![#000000](https://placehold.co/18x18/000000/000000.png) | **Obsidian Black** | `#000000` | `0, 0, 0` | `0, 0, 0, 100` | `Black C` | High-contrast typography, brand wordmark, icon base facets |
| ![#FFFFFF](https://placehold.co/18x18/FFFFFF/FFFFFF.png) | **Pure Canvas White**| `#FFFFFF` | `255, 255, 255`| `0, 0, 0, 0` | `Opaque` | Card surfaces, modals, flashcards, input fields |
| ![#7084A8](https://placehold.co/18x18/7084A8/7084A8.png) | **Slate Tint** | `#7084A8` | `112, 132, 168`| `33, 21, 0, 34` | `7544 C` | Subtle iconography, hover states, muted calendar gridlines |

### Tailwind CSS / Design Tokens Definition
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          flame: '#FF7A30',
          slate: '#465C88',
          'slate-tint': '#7084A8',
          stone: '#E9E3DF',
          obsidian: '#000000',
        }
      }
    }
  }
}
```

---

## 3. Typography System

### Primary Typeface: **Plus Jakarta Sans**
* **Designer / Foundry:** Tokotype (Available via Google Fonts)
* **Classification:** Modern Geometric Neo-Grotesque
* **Licensing:** Open Font License (OFL) — 100% free for commercial and academic use.

#### Why Plus Jakarta Sans?
1. **Stylistic Match to Reference:** Mirrors the warm geometric curves, open counters, and high energy of *Platform* (used by Preply) without proprietary commercial font license costs.
2. **Pedagogical & UI Legibility:** Clean tall x-height ensures student flashcards, foreign vocabulary accents (e.g. `á, é, ö, ñ, ç`), and CEFR rubrics remain instantly readable.
3. **Tabular Metric Stability:** Form arrays, calendar time columns, and financial payment ledgers stay aligned across rows.

### Font Weights & Hierarchy

```css
@import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap');

body {
  font-family: 'Plus Jakarta Sans', system-ui, -apple-system, sans-serif;
}
```

| Level | Size | Weight | Line Height | Tracking | Recommended Usage |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Display / Hero** | `48px` | `800` (ExtraBold) | `1.1` | `-0.03em` | Main dashboard welcome, marketing banners |
| **Heading 1** | `32px` | `700` (Bold) | `1.2` | `-0.02em` | View titles (e.g., "Glossary Deck", "Student Roster") |
| **Heading 2** | `22px` | `600` (SemiBold) | `1.3` | `-0.01em` | Modal titles, card section headings |
| **Heading 3** | `16px` | `600` (SemiBold) | `1.4` | `0` | Table column headers, calendar days |
| **Body (Default)** | `15px` | `400` (Regular) | `1.5` | `0` | Lesson notes, homework feedback, general content |
| **Body (Medium)** | `15px` | `500` (Medium) | `1.5` | `0` | Form labels, button text, interactive pills |
| **Caption / Meta** | `12px` | `500` (Medium) | `1.4` | `+0.01em` | Timestamps, payment reference codes, helper text |

---

## 4. Logo Construction & Clear Space

```
                 [Top Facet: #FF7A30]
                        /\
                       /  \
     [Left: #000000]  /    \  [Right: #000000]
           <========={  ><  }=========>
                      \    /
                       \  /
                        \/
                [Bottom: #000000]
```

* **Clear Space Rule:** Minimum margin around the logo equals the height of the letter **"i"** in the wordmark.
* **Minimum Size:**
  * Digital favicon / avatar: Minimum `24x24 px`.
  * Printed documentation: Minimum `12 mm` width.
* **Background Rule:** Place primarily on `#FFFFFF` (cards) or `#E9E3DF` (canvas). In dark backgrounds, invert the black facets to `#FFFFFF` while retaining `#FF7A30` for the top flame.
