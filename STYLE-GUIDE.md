# Style Guide - The "Gold Standard" Design System

> Visual and component standards for the LocalLane ecosystem.
> All nodes MUST follow these patterns for consistency.

---

## Status: ✅ COMPLETE

Last updated: 2026-01-28
Sources: Blueprint Master Doc, existing style doc, live app screenshots

---

## Design Philosophy

From the Blueprint:

> "Local Lane is a premium, trusted community platform. We avoid generic 'template' colors."

**Core Principles:**
- **Uniformity builds trust** — consistent look across all screens
- **No functional color-coding** — No red hearts, green checkmarks, etc.
- **Icons are Gold or White only**
- **Dark theme only** — No light backgrounds (except Admin Panel if intentional)
- **Premium, not generic** — Avoid template aesthetics

---

## Color Palette

### Background Colors

| Name | Tailwind Class | Hex | Usage |
|------|----------------|-----|-------|
| **Main Background** | `bg-slate-950` | `#020617` | Primary page background |
| **Surface/Cards** | `bg-slate-900` | `#0f172a` | Cards, headers, dropdowns |
| **Elevated Surface** | `bg-slate-800` | `#1e293b` | Borders, nested elements |
| **Hover Surface** | `bg-slate-800/50` | — | Hover states on dark elements |

### Accent Color (Amber/Gold)

| Name | Tailwind Class | Hex | Usage |
|------|----------------|-----|-------|
| **Primary Gold** | `bg-amber-500` | `#f59e0b` | Primary buttons, active states, brand |
| **Gold Hover** | `bg-amber-400` | `#fbbf24` | Button hover states |
| **Gold Active** | `bg-amber-600` | `#d97706` | Button active/pressed states |
| **Gold Text** | `text-amber-500` | `#f59e0b` | Accent text, links, icons |
| **Gold Border** | `border-amber-500` | `#f59e0b` | Selected element borders |

### Text Colors

| Name | Tailwind Class | Usage |
|------|----------------|-------|
| **Primary Text** | `text-slate-100` or `text-white` | Main headings, primary content |
| **Secondary Text** | `text-slate-300` | Navigation links, secondary content |
| **Muted Text** | `text-slate-400` | Descriptions, captions, placeholders |
| **Subtle Text** | `text-slate-500` | Very subtle labels |
| **On Gold** | `text-black` | Text on amber/gold backgrounds |

### Border Colors

| Name | Tailwind Class | Usage |
|------|----------------|-------|
| **Default Border** | `border-slate-800` | Standard dividers |
| **Subtle Border** | `border-slate-700` | Lighter dividers |
| **Hover Border** | `border-slate-600` | Border hover states |
| **White Border** | `border-white/10` | Very subtle on dark |

### Status Colors (Use Sparingly)

| Name | Tailwind Class | Usage |
|------|----------------|-------|
| **Success** | `text-emerald-500` / `bg-emerald-500` | Confirmations, sync status |
| **Warning** | `text-amber-400` | Cautions (stays in gold family) |
| **Error** | `text-red-500` / `bg-red-500` | Errors, destructive actions |
| **Info** | `text-blue-400` | Informational (rare) |

---

## Typography

### Font Stack

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### Font Sizes & Weights

| Element | Classes | Example |
|---------|---------|---------|
| **Page Title** | `text-3xl font-bold text-slate-100` | "Welcome to Your Dashboard" |
| **Section Heading** | `text-2xl font-bold text-slate-100` | "Events" |
| **Card Title** | `text-xl font-bold text-slate-100` | "Build Your Local Lane" |
| **Subheading** | `text-lg font-semibold text-slate-100` | "Punch Pass Earnings" |
| **Body Text** | `text-base text-slate-300` | Regular content |
| **Small Text** | `text-sm text-slate-400` | Descriptions, captions |
| **Tiny Text** | `text-xs text-slate-500` | Badges, labels |

---

## Component Patterns

### Primary Button (Gold)

```jsx
<button className="bg-amber-500 hover:bg-amber-400 active:bg-amber-600 text-black font-bold py-3 px-6 rounded-lg transition-colors">
  Primary Action
</button>
```

**States:**
- Default: `bg-amber-500 text-black`
- Hover: `bg-amber-400`
- Active/Pressed: `bg-amber-600`
- Disabled: `bg-amber-500/50 cursor-not-allowed`

### Secondary Button (Outline)

```jsx
<button className="bg-transparent border border-slate-600 text-slate-300 px-4 py-2 rounded-lg hover:border-amber-500 hover:text-amber-500 hover:bg-slate-800/50 transition-all">
  Secondary Action
</button>
```

### Ghost Button (Navigation/Links)

```jsx
<button className="text-slate-300 hover:text-amber-500 transition-colors">
  Ghost Action
</button>
```

### Card

```jsx
<div className="bg-slate-900 border border-slate-800 rounded-xl p-6">
  <h3 className="text-xl font-bold text-slate-100">Card Title</h3>
  <p className="text-slate-400 mt-2 text-sm">Card description text.</p>
</div>
```

**Card Variants:**
- Default: `bg-slate-900 border-slate-800`
- Selected/Active: `bg-slate-900 border-amber-500` (gold border)
- Hover: Add `hover:border-slate-700` or `hover:shadow-lg`

### Input Field

```jsx
<input 
  type="text"
  className="w-full bg-slate-800 border border-slate-700 text-white px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent placeholder:text-slate-500 transition-all"
  placeholder="Enter text..."
/>
```

### Badge

```jsx
// Gold badge (Punch Pass, Network, Tier)
<span className="bg-amber-500 text-black px-2 py-1 text-xs font-semibold rounded-full">
  3 Punches
</span>

// Neutral badge (Category)
<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs font-medium rounded-full">
  sports & active
</span>

// Success badge (Sync status)
<span className="bg-emerald-500/20 text-emerald-500 px-2 py-1 text-xs font-medium rounded-full">
  ✓ Synced to Local Lane
</span>
```

### Toggle/Switch

```jsx
// Use shadcn/ui Switch component with these colors:
// Off: bg-slate-700
// On: bg-amber-500
```

### Dropdown/Select

```jsx
<select className="bg-slate-800 border border-slate-700 text-slate-300 px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500">
  <option>Option 1</option>
  <option>Option 2</option>
</select>
```

---

## Icon Guidelines

### Rules

1. **Colors:** Icons are **Gold (`text-amber-500`)** or **White (`text-white`/`text-slate-400`)** ONLY
2. **No functional color-coding** — Don't use red trash icon, green checkmark icon, etc.
3. **Consistency:** Use Lucide React throughout
4. **Sizes:** `h-4 w-4` (small), `h-5 w-5` (default), `h-6 w-6` (large)

### Examples

```jsx
import { Calendar, MapPin, Star, Ticket, Settings, Check } from 'lucide-react';

// Gold icons for emphasis/active states
<Calendar className="h-5 w-5 text-amber-500" />

// White/gray icons for default states
<Settings className="h-5 w-5 text-slate-400" />

// Icons in buttons inherit text color
<button className="bg-amber-500 text-black">
  <Check className="h-4 w-4" /> {/* Inherits black */}
</button>
```

---

## Layout Patterns

### Page Container

```jsx
<div className="min-h-screen bg-slate-950">
  <header className="bg-slate-900 border-b border-slate-800">
    {/* Header content */}
  </header>
  
  <main className="max-w-7xl mx-auto px-4 py-8">
    {/* Page content */}
  </main>
</div>
```

### Navigation Header

```jsx
<nav className="bg-slate-900 border-b border-slate-800 px-4 py-3">
  <div className="flex items-center justify-between max-w-7xl mx-auto">
    {/* Logo */}
    <div className="flex items-center gap-2">
      <Logo className="h-8 w-8 text-amber-500" />
      <span className="text-xl font-bold text-white">Local Lane</span>
    </div>
    
    {/* Nav Links */}
    <div className="flex items-center gap-6">
      <a className="text-slate-300 hover:text-amber-500 transition-colors">My Lane</a>
      <a className="text-slate-300 hover:text-amber-500 transition-colors">Browse Events</a>
      <a className="text-slate-300 hover:text-amber-500 transition-colors">Browse Directory</a>
    </div>
    
    {/* Actions */}
    <div className="flex items-center gap-4">
      {/* User menu */}
    </div>
  </div>
</nav>
```

### Card Grid

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* Cards */}
</div>
```

### Stats Row (Dashboard)

```jsx
<div className="grid grid-cols-2 md:grid-cols-4 gap-4">
  <div className="bg-slate-900 border border-slate-800 rounded-xl p-4">
    <div className="flex items-center justify-between">
      <span className="text-slate-400 text-sm">Total Events</span>
      <Calendar className="h-5 w-5 text-amber-500" />
    </div>
    <p className="text-3xl font-bold text-white mt-2">12</p>
  </div>
  {/* More stat cards */}
</div>
```

---

## Animation & Transitions

### Standard Transitions

```jsx
// Color transitions (buttons, links)
className="transition-colors duration-200"

// All transitions (cards, complex elements)
className="transition-all duration-200"

// Hover shadow
className="hover:shadow-lg transition-shadow duration-200"
```

### Accordion Animation (shadcn/ui)

Already configured in tailwind.config.js:
```js
animation: {
  "accordion-down": "accordion-down 0.2s ease-out",
  "accordion-up": "accordion-up 0.2s ease-out",
}
```

---

## Form Patterns

### Form Layout

```jsx
<form className="space-y-6">
  <div>
    <label className="block text-sm font-medium text-slate-300 mb-2">
      Label Text
    </label>
    <input 
      className="w-full bg-slate-800 border border-slate-700 text-white px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500"
    />
  </div>
  
  <div>
    <label className="block text-sm font-medium text-slate-300 mb-2">
      Another Field
    </label>
    <textarea 
      className="w-full bg-slate-800 border border-slate-700 text-white px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500 min-h-[100px]"
    />
  </div>
  
  <button type="submit" className="w-full bg-amber-500 hover:bg-amber-400 text-black font-bold py-3 rounded-lg transition-colors">
    Submit
  </button>
</form>
```

### Error States

```jsx
// Input with error
<input 
  className="w-full bg-slate-800 border border-red-500 text-white px-4 py-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-red-500"
/>
<p className="text-red-500 text-sm mt-1">Error message here</p>
```

---

## Modal/Dialog

```jsx
// Use shadcn/ui Dialog with these overrides:
// Overlay: bg-black/80
// Content: bg-slate-900 border border-slate-800
// Close button: text-slate-400 hover:text-white
```

---

## Tables (Admin)

```jsx
<table className="w-full">
  <thead>
    <tr className="border-b border-slate-700">
      <th className="text-left py-3 px-4 text-slate-400 font-medium text-sm">Column</th>
    </tr>
  </thead>
  <tbody>
    <tr className="border-b border-slate-800 hover:bg-slate-800/50">
      <td className="py-3 px-4 text-slate-300">Cell content</td>
    </tr>
  </tbody>
</table>
```

---

## Tier Badges

| Tier | Style |
|------|-------|
| **Standard (Free)** | `bg-slate-700 text-slate-300` |
| **Storefront (Paid)** | `bg-amber-500 text-black` |
| **Partner** | `bg-amber-500 text-black` + icon |

```jsx
// Standard tier
<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs rounded-full">
  Standard
</span>

// Storefront tier
<span className="bg-amber-500 text-black px-2 py-1 text-xs font-semibold rounded-full">
  ⚡ Storefront
</span>

// Partner tier
<span className="bg-amber-500 text-black px-2 py-1 text-xs font-semibold rounded-full">
  ★ Partner
</span>
```

---

## Network Badges

All networks use the same gold styling. Differentiate by text only:

```jsx
<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs rounded-full">
  Recess
</span>

<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs rounded-full">
  Homeschool
</span>
```

---

## Quick Filter Chips

```jsx
// Active filter
<button className="bg-amber-500 text-black px-3 py-1 rounded-full text-sm font-medium">
  All
</button>

// Inactive filter
<button className="bg-slate-800 text-slate-300 px-3 py-1 rounded-full text-sm hover:bg-slate-700 transition-colors">
  This Weekend
</button>
```

---

## Do's and Don'ts

### ✅ Do

- Use gold (`amber-500`) for primary actions and brand emphasis
- Use dark backgrounds consistently (`slate-950`, `slate-900`)
- Keep icons monochromatic (gold or white/gray)
- Use subtle borders (`border-slate-800`, `border-white/10`)
- Maintain high contrast for readability
- Use `transition-colors` or `transition-all` for smooth interactions

### ❌ Don't

- Use bright, generic colors (blue buttons, pink accents)
- Use colorful icons (red hearts, green checkmarks)
- Use light/white backgrounds (dark theme only)
- Mix multiple accent colors in the same view
- Use gradients (keep it flat)
- Use harsh borders without transparency

---

## CSS Variables (for shadcn/ui)

Add to your `index.css` or `globals.css`:

```css
:root {
  /* Backgrounds */
  --background: 222 47% 4%;         /* slate-950 */
  --foreground: 210 40% 98%;        /* white-ish */
  
  /* Cards & Surfaces */
  --card: 222 47% 11%;              /* slate-900 */
  --card-foreground: 210 40% 98%;
  
  /* Primary (Gold) */
  --primary: 38 92% 50%;            /* amber-500 */
  --primary-foreground: 222 47% 11%;
  
  /* Secondary */
  --secondary: 217 33% 17%;         /* slate-800 */
  --secondary-foreground: 210 40% 98%;
  
  /* Muted */
  --muted: 217 33% 17%;
  --muted-foreground: 215 20% 65%;
  
  /* Accent */
  --accent: 38 92% 50%;
  --accent-foreground: 222 47% 11%;
  
  /* Destructive */
  --destructive: 0 84% 60%;
  --destructive-foreground: 210 40% 98%;
  
  /* Borders & Inputs */
  --border: 217 33% 17%;
  --input: 217 33% 17%;
  --ring: 38 92% 50%;
  
  /* Radius */
  --radius: 0.5rem;
}
```

---

## Quick Reference

```
BACKGROUNDS:
  Page:        bg-slate-950   #020617
  Surface:     bg-slate-900   #0f172a
  Elevated:    bg-slate-800   #1e293b

ACCENT:
  Gold:        bg-amber-500   #f59e0b
  Gold Hover:  bg-amber-400   #fbbf24
  Gold Active: bg-amber-600   #d97706

TEXT:
  Primary:     text-slate-100 / text-white
  Secondary:   text-slate-300
  Muted:       text-slate-400
  On Gold:     text-black

BORDERS:
  Default:     border-slate-800
  Subtle:      border-white/10
  Selected:    border-amber-500
```

---

*This style guide reflects the "Gold Standard" design system from the Local Lane Blueprint. All nodes must follow these patterns for visual consistency and brand trust.*
