# Style Guide

> Visual standards for the LocalLane ecosystem.  
> All nodes must follow these patterns for consistency.

---

## Status: ✅ COMPLETE

Last updated: 2026-01-27  
Source: Local Lane Blueprint + existing tailwind.config.js

---

## The "Gold Standard" Design Philosophy

From the Blueprint:

> "Local Lane is a premium, trusted community platform. We avoid generic 'template' colors (blue/green/pink)."

**Core Principles:**
- Uniformity builds trust
- No functional color-coding (no red hearts, no green money signs)
- Icons are Gold or White only
- Dark, premium feel

---

## Color Palette

### Primary Brand Colors

| Name | Tailwind Class | Hex | Usage |
|------|----------------|-----|-------|
| **Background** | `bg-slate-900` | `#0f172a` | Page backgrounds, dark surfaces |
| **Local Lane Gold** | `text-amber-500` / `bg-amber-500` | `#f59e0b` | Primary accent, CTAs, brand identity |
| **Gold (hover)** | `bg-amber-600` | `#d97706` | Hover states on gold elements |
| **Text Primary** | `text-white` | `#ffffff` | Main text on dark backgrounds |
| **Text Secondary** | `text-white/70` | `rgba(255,255,255,0.7)` | Muted text, captions |
| **Borders** | `border-white/10` | `rgba(255,255,255,0.1)` | Subtle dividers, card borders |

### Semantic Colors (Use Sparingly)

| Name | Tailwind Class | Usage |
|------|----------------|-------|
| Success | `text-emerald-500` | Confirmations, check marks |
| Warning | `text-amber-400` | Cautions (use gold family) |
| Error | `text-red-500` | Errors, destructive actions |
| Info | `text-blue-400` | Informational (rare) |

### Trust Tier Colors

| Tier | Color | Badge Style |
|------|-------|-------------|
| **Standard (Free)** | Slate/Gray | `bg-slate-700 text-white` |
| **Storefront (Paid)** | Amber/Gold | `bg-amber-500 text-slate-900` |
| **Partner** | Gold with emphasis | `bg-amber-500 text-slate-900` + icon |

### Network Badge Colors

| Network | Style | Notes |
|---------|-------|-------|
| **Recess** | Gold badge | Movement/fitness/play |
| **TCA** | Gold badge | Arts/trades/skills |
| **Homeschool** | Gold badge | Education network |

**Rule:** All network badges use the same gold styling. Differentiate by text/icon, not color.

---

## CSS Variables (shadcn/ui)

Your apps use shadcn/ui with CSS variables. Here's how to align them with the Local Lane brand:

```css
:root {
  /* Backgrounds */
  --background: 222 47% 11%;        /* slate-900 */
  --foreground: 210 40% 98%;        /* white-ish */
  
  /* Cards & Surfaces */
  --card: 217 33% 17%;              /* slate-800 */
  --card-foreground: 210 40% 98%;
  
  /* Primary (Gold) */
  --primary: 38 92% 50%;            /* amber-500 */
  --primary-foreground: 222 47% 11%; /* dark text on gold */
  
  /* Secondary */
  --secondary: 217 33% 17%;         /* slate-800 */
  --secondary-foreground: 210 40% 98%;
  
  /* Muted */
  --muted: 217 33% 17%;
  --muted-foreground: 215 20% 65%;
  
  /* Accent (same as primary for Local Lane) */
  --accent: 38 92% 50%;
  --accent-foreground: 222 47% 11%;
  
  /* Destructive */
  --destructive: 0 84% 60%;         /* red-500 */
  --destructive-foreground: 210 40% 98%;
  
  /* Borders & Inputs */
  --border: 217 33% 17%;
  --input: 217 33% 17%;
  --ring: 38 92% 50%;               /* gold focus ring */
  
  /* Radius */
  --radius: 0.5rem;
}
```

---

## Typography

### Font Stack

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

(Using system fonts for performance. Can add custom font later if branding requires.)

### Font Weights

| Weight | Usage |
|--------|-------|
| 400 (normal) | Body text |
| 500 (medium) | Emphasis, labels |
| 600 (semibold) | Subheadings |
| 700 (bold) | Headings, buttons |

---

## Component Patterns

### Primary Button (Gold)

```jsx
<button className="bg-amber-500 text-slate-900 px-4 py-2 rounded-lg font-semibold hover:bg-amber-600 transition-colors">
  Primary Action
</button>
```

### Secondary Button (Dark with Border)

```jsx
<button className="bg-transparent border border-white/10 text-white px-4 py-2 rounded-lg font-medium hover:bg-white/5 transition-colors">
  Secondary Action
</button>
```

### Card

```jsx
<div className="bg-slate-800 border border-white/10 rounded-xl p-6">
  <h3 className="text-white font-semibold">Card Title</h3>
  <p className="text-white/70 mt-2">Card content goes here.</p>
</div>
```

### Badge (Network/Tier)

```jsx
<span className="bg-amber-500 text-slate-900 px-2 py-1 text-xs font-semibold rounded-full">
  Recess
</span>
```

### Input Field

```jsx
<input 
  type="text"
  className="w-full bg-slate-800 border border-white/10 text-white px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent placeholder:text-white/40"
  placeholder="Enter text..."
/>
```

---

## Icons

### Rules

1. **Color:** Icons are **Gold (`text-amber-500`)** or **White (`text-white`)** only
2. **No functional color-coding** — Don't use red for delete, green for success in icons
3. **Consistency:** Use the same icon library throughout (Lucide React recommended)

### Example

```jsx
import { Calendar, MapPin, Star, Ticket } from 'lucide-react';

// Gold icons for emphasis
<Calendar className="text-amber-500" size={20} />

// White icons for navigation/secondary
<MapPin className="text-white" size={20} />
```

---

## Layout Patterns

### Page Container (Dark Theme)

```jsx
<div className="min-h-screen bg-slate-900">
  <header className="bg-slate-800 border-b border-white/10">
    {/* Header content */}
  </header>
  
  <main className="max-w-7xl mx-auto px-4 py-8">
    {/* Page content */}
  </main>
</div>
```

### Two-Sided Dashboard (from Blueprint)

The Blueprint specifies separating:

1. **The "Living Room"** (Main Dashboard) — For browsing users
   - Personalized feed
   - Saved items
   - Event discovery

2. **The "Back Office"** (Host Wizard) — For business owners
   - Create organization
   - Manage listings
   - Analytics

Don't mix these UX patterns. Keep them distinct.

---

## Do's and Don'ts

### ✅ Do

- Use gold for primary actions and brand emphasis
- Use dark backgrounds consistently
- Keep icons monochromatic (gold or white)
- Use subtle borders (`border-white/10`)
- Maintain high contrast for readability

### ❌ Don't

- Use bright, generic colors (blue buttons, pink accents)
- Use colorful icons (red hearts, green checkmarks)
- Use light/white backgrounds (dark theme only)
- Mix multiple accent colors
- Use gradients (keep it flat)

---

## Responsive Breakpoints

Use Tailwind defaults:

| Breakpoint | Min Width |
|------------|-----------|
| `sm` | 640px |
| `md` | 768px |
| `lg` | 1024px |
| `xl` | 1280px |

Mobile-first approach:
```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
```

---

## Quick Reference

```
Background:     bg-slate-900    #0f172a
Surface:        bg-slate-800    #1e293b
Gold:           bg-amber-500    #f59e0b
Gold Hover:     bg-amber-600    #d97706
Text:           text-white      #ffffff
Text Muted:     text-white/70
Border:         border-white/10
```

---

*This style guide reflects the "Gold Standard" design system from the Local Lane Blueprint. All nodes must follow these patterns.*
