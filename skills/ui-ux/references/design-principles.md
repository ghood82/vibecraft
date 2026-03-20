# Design Principles

Deep dive into visual design theory for developers.

## Visual Hierarchy

### The Squint Test
Squint at your design (or blur it). If you can still tell what's most important, your hierarchy works. If everything looks the same, it doesn't.

### Techniques
1. **Scale contrast**: Make the most important thing 2-3x larger than secondary content
2. **Color weight**: Use your primary/brand color sparingly — only on things you want clicked
3. **Whitespace**: More space around an element = more importance
4. **Depth**: Subtle shadows or elevation for primary actions
5. **Typography weight**: Bold for headings, regular for body, light for captions

## Typography

### Type Scale
Use a consistent scale. The "Major Third" scale (1.25 ratio) works well:
```
12px → 14px → 16px → 20px → 25px → 31px → 39px → 49px
```

### Font Pairing Rules
- One sans-serif is almost always enough (Inter, Plus Jakarta Sans, Geist)
- If pairing: serif headings + sans-serif body (or vice versa)
- Never use more than 2 font families
- Match x-height when pairing fonts

### Readability
- Body text: 16px minimum, 1.5-1.75 line height
- Line length: 45-75 characters per line (max-w-prose in Tailwind)
- Paragraph spacing: 1-1.5em between paragraphs
- Letter spacing: Slightly loose for ALL CAPS text (+0.05em)

## Color Theory

### Building a Palette
1. **Start with one color** — your primary brand/action color
2. **Generate a scale** — 50 through 950 (Tailwind convention)
3. **Add neutrals** — A warm or cool gray that pairs with your primary
4. **Add semantics** — Success (green), error (red), warning (amber), info (blue)
5. **Test contrast** — Every text/background combination must pass WCAG AA

### Dark Mode
- Don't just invert colors — rebuild the palette for dark backgrounds
- Use lighter, slightly desaturated versions of your primary color
- Background: Not pure black (#000). Use very dark gray (#0a0a0a to #171717)
- Text: Not pure white (#fff). Use slightly off-white (#fafafa)
- Reduce shadow intensity and increase border visibility
- Semantic colors may need lighter variants

### Accessibility
- **AA Normal text**: 4.5:1 contrast ratio
- **AA Large text**: 3:1 contrast ratio (18px+ or 14px+ bold)
- **AAA Normal text**: 7:1 contrast ratio
- Never use color as the ONLY indicator (add icons, text, or patterns)

## Spacing

### The 8px Grid
All spacing should be multiples of 8px (with 4px for tight spacing):
```
4px  — Icon padding, inline elements
8px  — Between related items
16px — Standard component padding
24px — Between component groups
32px — Section padding
48px — Between page sections
64px — Major page breaks
```

### Spacing Relationships
- Closer = more related (Gestalt proximity)
- Elements in the same group should have less space between them than between groups
- Consistent spacing rhythm creates visual calm

## Layout

### Grid System
- Use CSS Grid for 2D layouts, Flexbox for 1D
- Standard content width: 1280px max with padding
- Sidebar: 240-280px fixed, content fills remaining
- Card grids: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6`

### Responsive Strategy
- **Mobile first**: Design the smallest screen, enhance with breakpoints
- **Content out**: Let content dictate breakpoints, not device sizes
- **Flexible images**: `max-w-full h-auto` on all images
- **Stack on mobile**: Multi-column layouts collapse to single column

## Modern Design Patterns

### Glass Morphism (Use Sparingly)
```css
background: rgba(255, 255, 255, 0.1);
backdrop-filter: blur(10px);
border: 1px solid rgba(255, 255, 255, 0.2);
```

### Subtle Gradients
```css
/* Good: Subtle, not screaming */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* Bad: Rainbow vomit */
background: linear-gradient(90deg, red, orange, yellow, green, blue, purple);
```

### Micro-Interactions
- Button hover: Slight scale (1.02) or color shift
- Input focus: Ring color + subtle shadow
- Card hover: Lift with shadow increase
- Toggle: Smooth spring animation
- Loading: Skeleton screens over spinners
