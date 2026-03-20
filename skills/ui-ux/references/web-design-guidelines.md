# Web Design Guidelines

Page-specific design patterns and component recommendations.

## Landing Pages

### Hero Section
- **Headline**: 5-10 words, clear value proposition. Not clever, clear.
- **Subheadline**: 1-2 sentences expanding on the headline
- **CTA**: One primary button. "Get Started", "Try Free", "Start Building"
- **Visual**: Screenshot, illustration, or demo. Not a stock photo.
- **Social proof**: Small logos or "Trusted by 10,000+ developers"

### Feature Sections
- 3 features per row on desktop, stack on mobile
- Icon + heading + 2-3 sentence description
- Use feature cards or alternating left/right layout for deeper features

### Pricing
- 3 tiers maximum (Good, Better, Best)
- Highlight the recommended tier
- List features with checkmarks
- Annual vs monthly toggle with savings shown

### CTAs
- Repeat your primary CTA 3-4 times throughout the page
- Use action verbs: "Start building", "Get your API key", "Deploy now"
- Don't say "Submit" or "Click here"

## Dashboards

### Layout
- **Fixed sidebar** (240px): Navigation, always visible
- **Top bar**: Search, user menu, notifications, breadcrumbs
- **Main area**: Content with consistent padding (24-32px)

### Data Display
- **Metric cards**: 3-4 KPIs at the top with trend indicators
- **Charts**: Use appropriate chart type (line for trends, bar for comparison, pie for composition)
- **Tables**: Sortable headers, alternating row backgrounds, actions in last column

### Navigation
- Group nav items by category
- Use icons + text labels (never icons alone)
- Indicate active state clearly
- Collapsible sidebar for more content space

## Forms

### Layout
- One column for most forms (two columns only for short related fields like city/state)
- Labels above inputs, aligned left
- Required indicator: red asterisk or "(required)" text
- Optional fields marked "(optional)" — not the other way around

### Validation
- Inline validation on blur (not on every keystroke)
- Error messages below the field, in red, with an icon
- Success state: Green check when field is valid
- Don't clear the field on error — let users fix their input

### Design
```tsx
<div className="space-y-2">
  <label htmlFor="email" className="text-sm font-medium text-foreground">
    Email address
  </label>
  <input
    id="email"
    type="email"
    className="w-full rounded-md border px-3 py-2 text-sm focus:ring-2 focus:ring-primary"
    placeholder="you@example.com"
  />
  {error && (
    <p className="text-sm text-red-500 flex items-center gap-1">
      <AlertCircle className="h-4 w-4" /> {error}
    </p>
  )}
</div>
```

## Navigation Patterns

### Top Navigation
Best for: Marketing sites, simple apps with < 7 nav items
```
┌─────────────────────────────────────┐
│ Logo    Home  Features  Pricing  CTA│
└─────────────────────────────────────┘
```

### Sidebar Navigation
Best for: Dashboards, admin panels, apps with many sections
```
┌──────┬──────────────────┐
│ Logo │                  │
│──────│                  │
│ Home │   Main Content   │
│ Users│                  │
│ Data │                  │
│──────│                  │
│ Help │                  │
└──────┴──────────────────┘
```

### Command Palette
Best for: Power users, apps with many actions
- Trigger with `Cmd+K` or `Ctrl+K`
- Fuzzy search across pages, actions, and settings
- Use cmdk library (https://cmdk.paco.me/)

## Loading States

### Skeleton Screens (Preferred)
Show the shape of content before it loads. Better than spinners.
```tsx
function SkeletonCard() {
  return (
    <div className="animate-pulse space-y-3">
      <div className="h-4 bg-muted rounded w-3/4" />
      <div className="h-3 bg-muted rounded w-1/2" />
      <div className="h-3 bg-muted rounded w-5/6" />
    </div>
  );
}
```

### Progress Indicators
- Spinner: for actions taking 1-5 seconds
- Progress bar: for actions with known duration
- Skeleton: for page/component loads

## Empty States
Every list/table needs an empty state:
```tsx
function EmptyState() {
  return (
    <div className="text-center py-12">
      <InboxIcon className="mx-auto h-12 w-12 text-muted-foreground" />
      <h3 className="mt-2 text-sm font-semibold">No projects yet</h3>
      <p className="mt-1 text-sm text-muted-foreground">
        Get started by creating your first project.
      </p>
      <Button className="mt-4">Create Project</Button>
    </div>
  );
}
```

## Component Libraries

### Recommended: shadcn/ui
- Not a dependency — copies components into your project
- Full Tailwind CSS styling
- Built on Radix UI (accessible primitives)
- Highly customizable
- `npx shadcn@latest add button dialog table`

### Alternatives
- **Radix UI**: Unstyled accessible primitives (shadcn is built on this)
- **Headless UI**: Tailwind Labs' accessible components
- **MUI**: Full-featured, opinionated (good for enterprise)
- **Mantine**: Feature-rich with good defaults
