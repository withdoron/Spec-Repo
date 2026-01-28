# Style Guide

> Visual standards for the LocalLane ecosystem.  
> All nodes must follow these patterns for consistency.

---

## Status: ✅ EXTRACTED FROM CODE

Last reviewed: 2026-01-27  
Source: `events-node` and `community-node` tailwind.config.js (identical)

---

## Design System: shadcn/ui

Both nodes use **shadcn/ui** — a component library built on Radix UI primitives with Tailwind CSS. This is excellent because:

- Components are consistent and accessible
- Easy to customize via CSS variables
- Well-documented: https://ui.shadcn.com

### Key Dependencies

```json
{
  "tailwindcss": "^3.x",
  "tailwindcss-animate": "^1.x",
  "@radix-ui/*": "various",
  "@tanstack/react-query": "^5.x"
}
```

---

## Color System

Colors are defined using **CSS variables** with HSL values. This allows for easy theming and dark mode support.

### How It Works

In `tailwind.config.js`, colors reference CSS variables:
```javascript
primary: {
  DEFAULT: 'hsl(var(--primary))',
  foreground: 'hsl(var(--primary-foreground))'
}
```

The actual HSL values are defined in your CSS (likely `index.css` or `globals.css`):
```css
:root {
  --primary: 222 47% 11%;
  --primary-foreground: 210 40% 98%;
  /* etc. */
}
```

### Semantic Color Tokens

| Token | CSS Variable | Usage |
|-------|--------------|-------|
| `background` | `--background` | Page backgrounds |
| `foreground` | `--foreground` | Default text color |
| `card` | `--card` | Card backgrounds |
| `card-foreground` | `--card-foreground` | Text on cards |
| `popover` | `--popover` | Dropdown/popover backgrounds |
| `popover-foreground` | `--popover-foreground` | Text in popovers |
| `primary` | `--primary` | Primary buttons, links |
| `primary-foreground` | `--primary-foreground` | Text on primary elements |
| `secondary` | `--secondary` | Secondary buttons |
| `secondary-foreground` | `--secondary-foreground` | Text on secondary elements |
| `muted` | `--muted` | Muted backgrounds |
| `muted-foreground` | `--muted-foreground` | Muted/secondary text |
| `accent` | `--accent` | Accent highlights |
| `accent-foreground` | `--accent-foreground` | Text on accent elements |
| `destructive` | `--destructive` | Delete/danger actions |
| `destructive-foreground` | `--destructive-foreground` | Text on destructive buttons |
| `border` | `--border` | Borders, dividers |
| `input` | `--input` | Input field borders |
| `ring` | `--ring` | Focus rings |

### Chart Colors (for data visualization)

| Token | Variable |
|-------|----------|
| `chart-1` | `--chart-1` |
| `chart-2` | `--chart-2` |
| `chart-3` | `--chart-3` |
| `chart-4` | `--chart-4` |
| `chart-5` | `--chart-5` |

### Sidebar Colors (for navigation)

| Token | Variable |
|-------|----------|
| `sidebar` | `--sidebar-background` |
| `sidebar-foreground` | `--sidebar-foreground` |
| `sidebar-primary` | `--sidebar-primary` |
| `sidebar-accent` | `--sidebar-accent` |
| `sidebar-border` | `--sidebar-border` |

---

## To Get Actual Color Values

The CSS variables are defined in your CSS file. To document the actual colors:

1. Open `src/index.css` (or wherever your CSS variables are defined)
2. Look for the `:root` block
3. Document the HSL values here

**Action item:** Paste the contents of your `index.css` `:root` block here so we can complete the color documentation.

---

## Trust Tier Colors (To Be Defined)

We need to add custom colors for the tier system. Recommendation:

| Tier | Suggested Color | CSS Variable |
|------|-----------------|--------------|
| Tier 1 (Free) | Slate/Gray | `--tier-free` |
| Tier 2 (Pro) | Blue | `--tier-pro` |
| Tier 3 (Partner) | Gold/Amber | `--tier-partner` |

Add to your CSS:
```css
:root {
  --tier-free: 215 16% 47%;      /* slate-500 */
  --tier-pro: 217 91% 60%;       /* blue-500 */
  --tier-partner: 45 93% 47%;    /* amber-500 */
}
```

Add to `tailwind.config.js`:
```javascript
colors: {
  // ... existing colors
  tier: {
    free: 'hsl(var(--tier-free))',
    pro: 'hsl(var(--tier-pro))',
    partner: 'hsl(var(--tier-partner))',
  }
}
```

---

## Typography

### Font Family

```css
/* Primary font */
font-family: ______;

/* Monospace (for codes, IDs) */
font-family: ______;
```

### Font Sizes

| Name | Size | Line Height | Usage |
|------|------|-------------|-------|
| xs | 0.75rem (12px) | 1rem | Captions, fine print |
| sm | 0.875rem (14px) | 1.25rem | Secondary text |
| base | 1rem (16px) | 1.5rem | Body text |
| lg | 1.125rem (18px) | 1.75rem | Large body |
| xl | 1.25rem (20px) | 1.75rem | Subheadings |
| 2xl | 1.5rem (24px) | 2rem | Section headings |
| 3xl | 1.875rem (30px) | 2.25rem | Page titles |
| 4xl | 2.25rem (36px) | 2.5rem | Hero titles |

### Font Weights

| Name | Weight | Usage |
|------|--------|-------|
| normal | 400 | Body text |
| medium | 500 | Emphasis |
| semibold | 600 | Subheadings |
| bold | 700 | Headings, buttons |

---

## Spacing

Use Tailwind's default spacing scale:

| Class | Value | Usage |
|-------|-------|-------|
| `p-1`, `m-1` | 0.25rem (4px) | Tight spacing |
| `p-2`, `m-2` | 0.5rem (8px) | Compact elements |
| `p-4`, `m-4` | 1rem (16px) | Standard spacing |
| `p-6`, `m-6` | 1.5rem (24px) | Section padding |
| `p-8`, `m-8` | 2rem (32px) | Large sections |

---

## Components

### Buttons

#### Primary Button

```jsx
<button className="bg-primary text-white px-4 py-2 rounded-lg font-medium hover:bg-primary-dark transition-colors">
  Primary Action
</button>
```

#### Secondary Button

```jsx
<button className="bg-secondary text-white px-4 py-2 rounded-lg font-medium hover:bg-secondary-dark transition-colors">
  Secondary Action
</button>
```

#### Outline Button

```jsx
<button className="border border-primary text-primary px-4 py-2 rounded-lg font-medium hover:bg-primary hover:text-white transition-colors">
  Outline Action
</button>
```

#### Disabled Button

```jsx
<button className="bg-gray-300 text-gray-500 px-4 py-2 rounded-lg font-medium cursor-not-allowed" disabled>
  Disabled
</button>
```

### Cards

```jsx
<div className="bg-surface rounded-xl shadow-sm border border-border p-6">
  <h3 className="text-lg font-semibold text-text-primary">Card Title</h3>
  <p className="text-text-secondary mt-2">Card content goes here.</p>
</div>
```

### Input Fields

```jsx
<input 
  type="text"
  className="w-full px-4 py-2 border border-border rounded-lg focus:outline-none focus:ring-2 focus:ring-primary focus:border-transparent"
  placeholder="Enter text..."
/>
```

### Badges

#### Tier Badges

```jsx
// Tier 1
<span className="px-2 py-1 text-xs font-medium bg-tier1 text-white rounded-full">
  Free
</span>

// Tier 2
<span className="px-2 py-1 text-xs font-medium bg-tier2 text-white rounded-full">
  Pro
</span>

// Tier 3
<span className="px-2 py-1 text-xs font-medium bg-tier3 text-white rounded-full flex items-center gap-1">
  <TrustIcon size={12} /> Partner
</span>
```

#### Network Badges

```jsx
<span className="px-2 py-1 text-xs font-medium bg-blue-100 text-blue-800 rounded-full">
  Recess
</span>
```

### Trust Stars

```jsx
<div className="flex items-center gap-1">
  {[1, 2, 3, 4, 5].map((star) => (
    <StarIcon 
      key={star}
      className={star <= rating ? "text-yellow-400 fill-current" : "text-gray-300"}
      size={16}
    />
  ))}
  <span className="text-sm text-text-secondary ml-1">({reviewCount})</span>
</div>
```

---

## Layout Patterns

### Page Container

```jsx
<div className="min-h-screen bg-background">
  <header className="bg-surface border-b border-border">
    {/* Header content */}
  </header>
  
  <main className="max-w-7xl mx-auto px-4 py-8">
    {/* Page content */}
  </main>
  
  <footer className="bg-surface border-t border-border mt-auto">
    {/* Footer content */}
  </footer>
</div>
```

### Card Grid

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {items.map((item) => (
    <Card key={item.id} item={item} />
  ))}
</div>
```

### Form Layout

```jsx
<form className="space-y-6">
  <div>
    <label className="block text-sm font-medium text-text-primary mb-1">
      Field Label
    </label>
    <input type="text" className="..." />
  </div>
  
  <div>
    <label className="block text-sm font-medium text-text-primary mb-1">
      Another Field
    </label>
    <input type="text" className="..." />
  </div>
  
  <button type="submit" className="...">
    Submit
  </button>
</form>
```

---

## Icons

### Recommended Icon Library

Use **Lucide React** for consistent icons across all nodes.

```bash
npm install lucide-react
```

```jsx
import { Calendar, MapPin, Users, Star, Check, X } from 'lucide-react';
```

### Common Icons

| Concept | Icon | Usage |
|---------|------|-------|
| Events | `Calendar` | Event listings |
| Location | `MapPin` | Addresses |
| People | `Users` | Attendees, members |
| Rating | `Star` | Trust scores |
| Success | `Check` | Confirmations |
| Error | `X` | Errors, close |
| Punch Pass | `Ticket` | Pass actions |
| Business | `Store` | Business listings |

---

## Responsive Breakpoints

Use Tailwind's default breakpoints:

| Breakpoint | Min Width | Usage |
|------------|-----------|-------|
| `sm` | 640px | Large phones |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |

### Mobile-First Pattern

```jsx
<div className="
  grid 
  grid-cols-1      /* Mobile: 1 column */
  md:grid-cols-2   /* Tablet: 2 columns */
  lg:grid-cols-3   /* Desktop: 3 columns */
  gap-4
">
```

---

## Animation

### Transitions

```css
/* Standard transition for interactive elements */
transition-colors duration-200 ease-in-out

/* For layout changes */
transition-all duration-300 ease-in-out
```

### Hover States

All interactive elements should have visible hover states:

```jsx
// Good
<button className="bg-primary hover:bg-primary-dark transition-colors">

// Bad (no feedback)
<button className="bg-primary">
```

---

## Accessibility

### Color Contrast

- Text on backgrounds must meet WCAG AA (4.5:1 for normal text)
- Use color + another indicator (icon, underline) for important states

### Focus States

All interactive elements need visible focus states:

```jsx
<button className="... focus:outline-none focus:ring-2 focus:ring-primary focus:ring-offset-2">
```

### Screen Reader Support

```jsx
// Decorative icons
<StarIcon aria-hidden="true" />

// Informational icons
<StarIcon aria-label="4 out of 5 stars" />

// Hidden text for context
<span className="sr-only">Open menu</span>
```

---

## Dark Mode (Future)

Currently not implemented. When we add it:

- Use CSS variables for colors
- Respect system preference
- Provide manual toggle

---

## To Do

- [ ] Extract actual colors from existing tailwind.config.js
- [ ] Document any existing component patterns
- [ ] Add screenshots of current UI as reference
- [ ] Define any custom Tailwind extensions

---

*This document is maintained in the Spec-Repo. Update when visual standards change.*
