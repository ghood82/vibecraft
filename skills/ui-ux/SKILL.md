---
name: ui-ux
description: "UI/UX design, components, visual hierarchy, accessibility. Use when asked about design, UI, UX, component, make it look good, spacing, typography, color, responsive, WCAG, Figma. Do NOT use for performance optimization (use performance) or API design (use api-design)."
---

# UI/UX Pro Max

Design and build interfaces that look professional, feel responsive, and work for everyone.

## Core Design Principles

### Visual Hierarchy
Size (bigger = important), Color (contrast = attention), Space (whitespace = importance), Position (top-left first), Weight (bold draws eye).

### Spacing System (8px Grid)
- `space-1` (4px) — Within elements
- `space-2` (8px) — Related elements
- `space-4` (16px) — Standard gap
- `space-6` (24px) — Between sections
- `space-8` (32px) — Section padding

### Typography Scale
```
text-xs: 12px  | text-lg: 18px  | text-3xl: 30px
text-sm: 14px  | text-xl: 20px  | text-4xl: 36px
text-base: 16px | text-2xl: 24px | text-5xl: 48px
```

Max 2 font families. Default: Inter for both.

### Color Strategy
- One primary color (buttons, links, accents)
- One neutral scale (text, borders, backgrounds)
- Semantic colors (green/success, red/error, yellow/warning, blue/info)

## Essential Components

Create these first for any UI:
1. Button (primary, secondary, ghost, destructive)
2. Input (text, email, password with labels, errors)
3. Card (container with padding)
4. Badge (status indicators)
5. Dialog/Modal (overlay with focus trap)
6. Toast (notifications)
7. Table (sortable with pagination)
8. Dropdown (selects and actions)

Prefer shadcn/ui — unstyled, accessible, Tailwind-native.

## Page Patterns

### Landing Page
Hero → Social Proof → Features → How It Works → Testimonials → Pricing → Footer

### Dashboard
```
┌──────┬──────────────┐
│ Side │  Top Bar     │
│ bar  ├──────────────┤
│ Nav  │  Main Content│
└──────┴──────────────┘
```

### Forms
- Label above input
- Error messages below field
- Group related fields
- Primary action on right

## Avoid "AI Slop"

- Generic purple/blue gradients
- Excessive border-radius
- Text centered everywhere
- Stock photos
- Overly complex animations
- Low-contrast text on gradients

## Accessibility (WCAG 2.1 AA)

- **Contrast**: 4.5:1 for body text
- **Navigation**: Everything clickable must be focusable
- **Semantics**: Use `button`, `nav`, `main`, not `div`
- **Labels**: Every form input needs a label
- **Alt text**: Meaningful images need alt text
- **Focus**: Visible focus ring on interactions

## Responsive Breakpoints

```
sm: 640px  | md: 768px | lg: 1024px | xl: 1280px | 2xl: 1536px
```

Design mobile-first.

Read `references/design-principles.md` for design theory.
Read `references/web-design-guidelines.md` for page patterns.
Read `references/accessibility.md` for WCAG compliance.
Read `references/figma-to-code.md` for Figma to code.
