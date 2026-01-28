# Component Library

> Detailed component specifications for the LocalLane ecosystem.
> Reference this when building any UI component.

---

## Status: ✅ COMPLETE

Last updated: 2026-01-28

---

## Button Components

### PrimaryButton

**Usage:** Main CTAs, form submissions, important actions

```jsx
// Props
interface PrimaryButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  size?: 'sm' | 'md' | 'lg';
}

// Classes by size
const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
};

// Base classes
className={cn(
  "bg-amber-500 hover:bg-amber-400 active:bg-amber-600",
  "text-black font-bold rounded-lg",
  "transition-colors duration-200",
  "disabled:bg-amber-500/50 disabled:cursor-not-allowed",
  fullWidth && "w-full",
  sizes[size]
)}
```

### SecondaryButton (Outline)

**Usage:** Secondary actions, cancel buttons

```jsx
className={cn(
  "bg-transparent border border-slate-600",
  "text-slate-300",
  "hover:border-amber-500 hover:text-amber-500 hover:bg-slate-800/50",
  "rounded-lg transition-all duration-200",
  // ... sizes
)}
```

### GhostButton

**Usage:** Navigation, tertiary actions

```jsx
className={cn(
  "text-slate-300 hover:text-amber-500",
  "transition-colors duration-200",
  // ... sizes
)}
```

### IconButton

**Usage:** Actions with icon only (edit, delete, etc.)

```jsx
className={cn(
  "p-2 rounded-lg",
  "text-slate-400 hover:text-amber-500 hover:bg-slate-800",
  "transition-all duration-200"
)}
```

---

## Card Components

### BaseCard

**Usage:** Standard container for content

```jsx
interface CardProps {
  children: React.ReactNode;
  className?: string;
  onClick?: () => void;
  selected?: boolean;
}

className={cn(
  "bg-slate-900 border rounded-xl p-6",
  selected ? "border-amber-500" : "border-slate-800",
  onClick && "cursor-pointer hover:border-slate-700",
  "transition-all duration-200"
)}
```

### EventCard

**Usage:** Event listings

```jsx
interface EventCardProps {
  event: {
    id: string;
    title: string;
    imageUrl?: string;
    date: Date;
    location: string;
    punchCost?: number;
    price?: number;
  };
  onClick: () => void;
}

// Structure:
<div className="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden hover:border-slate-700 transition-all cursor-pointer">
  {/* Image with badge */}
  <div className="relative">
    <img src={imageUrl} className="w-full h-48 object-cover" />
    {punchCost && (
      <span className="absolute top-3 right-3 bg-amber-500 text-black px-2 py-1 text-xs font-semibold rounded-full">
        {punchCost} Punches
      </span>
    )}
  </div>
  
  {/* Content */}
  <div className="p-4">
    <h3 className="text-lg font-bold text-white">{title}</h3>
    <div className="flex items-center gap-2 text-slate-400 text-sm mt-2">
      <Calendar className="h-4 w-4 text-amber-500" />
      <span>{formatDate(date)}</span>
    </div>
    <div className="flex items-center gap-2 text-slate-400 text-sm mt-1">
      <MapPin className="h-4 w-4 text-amber-500" />
      <span>{location}</span>
    </div>
  </div>
</div>
```

### StatCard

**Usage:** Dashboard statistics

```jsx
interface StatCardProps {
  label: string;
  value: string | number;
  icon: React.ReactNode;
  trend?: { value: number; positive: boolean };
}

<div className="bg-slate-900 border border-slate-800 rounded-xl p-4">
  <div className="flex items-center justify-between">
    <span className="text-slate-400 text-sm">{label}</span>
    <div className="p-2 bg-slate-800 rounded-lg">
      {icon} {/* Should be text-amber-500 */}
    </div>
  </div>
  <p className="text-3xl font-bold text-white mt-2">{value}</p>
  {trend && (
    <p className={cn("text-sm mt-1", trend.positive ? "text-emerald-500" : "text-red-500")}>
      {trend.positive ? "↑" : "↓"} {trend.value}%
    </p>
  )}
</div>
```

---

## Form Components

### TextInput

```jsx
interface TextInputProps {
  label?: string;
  placeholder?: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  disabled?: boolean;
  type?: 'text' | 'email' | 'password' | 'number';
}

<div>
  {label && (
    <label className="block text-sm font-medium text-slate-300 mb-2">
      {label}
    </label>
  )}
  <input
    type={type}
    value={value}
    onChange={(e) => onChange(e.target.value)}
    disabled={disabled}
    className={cn(
      "w-full bg-slate-800 text-white px-4 py-2 rounded-lg",
      "border focus:outline-none focus:ring-2 focus:ring-amber-500 focus:border-transparent",
      "placeholder:text-slate-500 transition-all duration-200",
      error ? "border-red-500" : "border-slate-700",
      disabled && "opacity-50 cursor-not-allowed"
    )}
    placeholder={placeholder}
  />
  {error && (
    <p className="text-red-500 text-sm mt-1">{error}</p>
  )}
</div>
```

### TextArea

```jsx
// Same as TextInput but with:
<textarea
  className={cn(
    "w-full bg-slate-800 text-white px-4 py-2 rounded-lg min-h-[100px] resize-y",
    // ... same classes
  )}
/>
```

### Select

```jsx
interface SelectProps {
  label?: string;
  options: { value: string; label: string }[];
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

<div>
  {label && (
    <label className="block text-sm font-medium text-slate-300 mb-2">
      {label}
    </label>
  )}
  <select
    value={value}
    onChange={(e) => onChange(e.target.value)}
    className={cn(
      "w-full bg-slate-800 text-slate-300 px-4 py-2 rounded-lg",
      "border border-slate-700",
      "focus:outline-none focus:ring-2 focus:ring-amber-500",
      "transition-all duration-200"
    )}
  >
    {placeholder && <option value="">{placeholder}</option>}
    {options.map((opt) => (
      <option key={opt.value} value={opt.value}>{opt.label}</option>
    ))}
  </select>
</div>
```

### Checkbox

```jsx
<label className="flex items-center gap-3 cursor-pointer">
  <input
    type="checkbox"
    checked={checked}
    onChange={(e) => onChange(e.target.checked)}
    className="sr-only"
  />
  <div className={cn(
    "w-5 h-5 rounded border-2 flex items-center justify-center transition-colors",
    checked 
      ? "bg-amber-500 border-amber-500" 
      : "bg-transparent border-slate-600 hover:border-slate-500"
  )}>
    {checked && <Check className="h-3 w-3 text-black" />}
  </div>
  <span className="text-slate-300">{label}</span>
</label>
```

### Toggle/Switch

Use shadcn/ui Switch with custom colors:
```jsx
// In your Switch component, override:
// Unchecked: bg-slate-700
// Checked: bg-amber-500
```

---

## Badge Components

### PunchBadge

```jsx
<span className="bg-amber-500 text-black px-2 py-1 text-xs font-semibold rounded-full">
  {count} Punches
</span>
```

### TierBadge

```jsx
interface TierBadgeProps {
  tier: 'standard' | 'storefront' | 'partner';
}

const tierStyles = {
  standard: "bg-slate-700 text-slate-300",
  storefront: "bg-amber-500 text-black",
  partner: "bg-amber-500 text-black",
};

const tierIcons = {
  standard: null,
  storefront: "⚡",
  partner: "★",
};

<span className={cn("px-2 py-1 text-xs font-semibold rounded-full", tierStyles[tier])}>
  {tierIcons[tier]} {tier.charAt(0).toUpperCase() + tier.slice(1)}
</span>
```

### NetworkBadge

```jsx
<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs rounded-full">
  {networkName}
</span>
```

### StatusBadge

```jsx
interface StatusBadgeProps {
  status: 'active' | 'pending' | 'synced' | 'error';
}

const statusStyles = {
  active: "bg-emerald-500/20 text-emerald-500",
  pending: "bg-amber-500/20 text-amber-400",
  synced: "bg-emerald-500/20 text-emerald-500",
  error: "bg-red-500/20 text-red-500",
};

<span className={cn("px-2 py-1 text-xs font-medium rounded-full", statusStyles[status])}>
  {status === 'synced' && "✓ "}
  {status.charAt(0).toUpperCase() + status.slice(1)}
</span>
```

### CategoryBadge

```jsx
<span className="bg-slate-700 text-slate-300 px-2 py-1 text-xs rounded-full">
  {categoryName}
</span>
```

---

## Filter Components

### FilterChip

```jsx
interface FilterChipProps {
  label: string;
  active: boolean;
  onClick: () => void;
}

<button
  onClick={onClick}
  className={cn(
    "px-3 py-1 rounded-full text-sm font-medium transition-colors",
    active
      ? "bg-amber-500 text-black"
      : "bg-slate-800 text-slate-300 hover:bg-slate-700"
  )}
>
  {label}
</button>
```

### FilterChipGroup

```jsx
<div className="flex flex-wrap gap-2">
  <FilterChip label="All" active={filter === 'all'} onClick={() => setFilter('all')} />
  <FilterChip label="This Weekend" active={filter === 'weekend'} onClick={() => setFilter('weekend')} />
  <FilterChip label="Today" active={filter === 'today'} onClick={() => setFilter('today')} />
</div>
```

---

## Navigation Components

### NavLink

```jsx
interface NavLinkProps {
  href: string;
  children: React.ReactNode;
  active?: boolean;
}

<a
  href={href}
  className={cn(
    "transition-colors duration-200",
    active
      ? "text-amber-500"
      : "text-slate-300 hover:text-amber-500"
  )}
>
  {children}
</a>
```

### TabNav

```jsx
<div className="flex gap-1 bg-slate-800 p-1 rounded-lg">
  {tabs.map((tab) => (
    <button
      key={tab.value}
      onClick={() => setActiveTab(tab.value)}
      className={cn(
        "px-4 py-2 rounded-md text-sm font-medium transition-colors",
        activeTab === tab.value
          ? "bg-slate-900 text-white"
          : "text-slate-400 hover:text-white"
      )}
    >
      {tab.label}
    </button>
  ))}
</div>
```

---

## Modal/Dialog Components

Use shadcn/ui Dialog with these overrides:

```jsx
// DialogOverlay
className="fixed inset-0 bg-black/80"

// DialogContent
className={cn(
  "fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2",
  "bg-slate-900 border border-slate-800 rounded-xl",
  "p-6 shadow-xl max-w-lg w-full max-h-[90vh] overflow-y-auto"
)}

// DialogTitle
className="text-xl font-bold text-white"

// DialogDescription
className="text-slate-400 mt-2"

// DialogClose
className="absolute top-4 right-4 text-slate-400 hover:text-white"
```

---

## Table Components

### DataTable

```jsx
<div className="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden">
  <table className="w-full">
    <thead>
      <tr className="border-b border-slate-700">
        <th className="text-left py-3 px-4 text-slate-400 font-medium text-sm">
          Column Header
        </th>
      </tr>
    </thead>
    <tbody>
      {rows.map((row, i) => (
        <tr 
          key={i}
          className="border-b border-slate-800 hover:bg-slate-800/50 transition-colors"
        >
          <td className="py-3 px-4 text-slate-300">
            {row.cell}
          </td>
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

---

## Empty States

```jsx
<div className="flex flex-col items-center justify-center py-12 text-center">
  <div className="p-4 bg-slate-800 rounded-full mb-4">
    <Calendar className="h-8 w-8 text-slate-500" />
  </div>
  <h3 className="text-lg font-semibold text-white mb-2">No events yet</h3>
  <p className="text-slate-400 mb-4 max-w-sm">
    Create your first event to start connecting with your community.
  </p>
  <PrimaryButton>Create Event</PrimaryButton>
</div>
```

---

## Loading States

### Spinner

```jsx
<div className="animate-spin rounded-full h-8 w-8 border-2 border-slate-700 border-t-amber-500" />
```

### Skeleton

```jsx
<div className="animate-pulse">
  <div className="bg-slate-800 rounded-lg h-4 w-3/4 mb-2" />
  <div className="bg-slate-800 rounded-lg h-4 w-1/2" />
</div>
```

### Full Page Loading

```jsx
<div className="min-h-screen bg-slate-950 flex items-center justify-center">
  <div className="text-center">
    <div className="animate-spin rounded-full h-12 w-12 border-2 border-slate-700 border-t-amber-500 mx-auto mb-4" />
    <p className="text-slate-400">Loading...</p>
  </div>
</div>
```

---

## Toast/Notification

Use shadcn/ui Toast with:

```jsx
// Success
className="bg-slate-900 border border-emerald-500 text-white"

// Error  
className="bg-slate-900 border border-red-500 text-white"

// Info
className="bg-slate-900 border border-slate-700 text-white"
```

---

## Step Wizard

```jsx
interface StepWizardProps {
  steps: { label: string; value: string }[];
  currentStep: number;
}

<div className="flex items-center justify-between">
  {steps.map((step, index) => (
    <React.Fragment key={step.value}>
      {/* Step circle */}
      <div className="flex flex-col items-center">
        <div className={cn(
          "w-10 h-10 rounded-full flex items-center justify-center font-bold",
          index < currentStep
            ? "bg-amber-500 text-black"
            : index === currentStep
            ? "bg-amber-500 text-black"
            : "bg-slate-800 text-slate-500"
        )}>
          {index + 1}
        </div>
        <span className={cn(
          "text-sm mt-2",
          index <= currentStep ? "text-white" : "text-slate-500"
        )}>
          {step.label}
        </span>
      </div>
      
      {/* Connector line */}
      {index < steps.length - 1 && (
        <div className={cn(
          "flex-1 h-0.5 mx-4",
          index < currentStep ? "bg-amber-500" : "bg-slate-700"
        )} />
      )}
    </React.Fragment>
  ))}
</div>
```

---

## Utility Classes Reference

```jsx
// Spacing
p-4, p-6, px-4, py-2, gap-4, gap-6, space-y-4, space-y-6

// Sizing
w-full, max-w-lg, max-w-xl, max-w-7xl

// Flexbox
flex, items-center, justify-between, justify-center, gap-2, gap-4

// Grid
grid, grid-cols-1, md:grid-cols-2, lg:grid-cols-3, gap-4, gap-6

// Border radius
rounded-lg (8px), rounded-xl (12px), rounded-full (9999px)

// Transitions
transition-colors, transition-all, duration-200

// Responsive
sm:, md:, lg:, xl:
```

---

*Use these component patterns for consistency across the LocalLane ecosystem.*
