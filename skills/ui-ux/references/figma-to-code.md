# Figma-to-Code Pipeline

## Overview

Convert Figma designs to production code using the connected Figma MCP. The pipeline: Extract design context → Map to components → Generate code → Validate against design.

## Using Figma MCP

### Get Design Context
```
// Use the Figma MCP tools to extract design information
get_design_context — Fetches component structure, layout, colors, typography from a Figma file
get_screenshot — Captures visual reference of a frame or component
get_metadata — Gets file-level metadata (pages, components, styles)
get_variable_defs — Extracts design tokens (colors, spacing, typography variables)
get_code_connect_map — Maps Figma components to code components
```

### Workflow

**Step 1: Extract Design Tokens**
Pull variables from Figma and map to CSS/Tailwind:

```typescript
// From Figma variables → tailwind.config.ts
const figmaTokens = {
  // Colors from Figma's color styles
  primary: { 50: "#eff6ff", 500: "#3b82f6", 900: "#1e3a5f" },
  neutral: { 50: "#fafafa", 100: "#f5f5f5", 900: "#171717" },

  // Spacing from Figma's spacing variables
  spacing: { xs: "4px", sm: "8px", md: "16px", lg: "24px", xl: "32px" },

  // Typography from Figma's text styles
  fontSize: {
    xs: ["12px", { lineHeight: "16px" }],
    sm: ["14px", { lineHeight: "20px" }],
    base: ["16px", { lineHeight: "24px" }],
    lg: ["18px", { lineHeight: "28px" }],
    xl: ["20px", { lineHeight: "28px" }],
    "2xl": ["24px", { lineHeight: "32px" }],
  },

  // Border radius
  borderRadius: { sm: "4px", md: "8px", lg: "12px", full: "9999px" },
};

// Export as Tailwind config
export default {
  theme: {
    extend: {
      colors: figmaTokens.primary,
      spacing: figmaTokens.spacing,
      fontSize: figmaTokens.fontSize,
      borderRadius: figmaTokens.borderRadius,
    },
  },
};
```

**Step 2: Map Figma Components to Code**

| Figma Component | Code Component | Library |
|----------------|---------------|---------|
| Button (variants: primary, secondary, ghost) | `<Button variant="primary">` | shadcn/ui or custom |
| Input (states: default, focus, error) | `<Input error={msg}>` | shadcn/ui or custom |
| Card (with header, body, footer) | `<Card><CardHeader>...` | shadcn/ui |
| Avatar (sizes: sm, md, lg) | `<Avatar size="md">` | Custom |
| Badge (colors: success, warning, error) | `<Badge variant="success">` | shadcn/ui |
| Modal/Dialog | `<Dialog>` | Radix + shadcn/ui |
| Dropdown Menu | `<DropdownMenu>` | Radix + shadcn/ui |
| Tabs | `<Tabs>` | Radix + shadcn/ui |

**Step 3: Layout Translation**

```
Figma Auto Layout → CSS/Tailwind Mapping:

Direction: Horizontal → flex-row
Direction: Vertical → flex-col
Gap: 16 → gap-4
Padding: 24 → p-6
Alignment: Center → items-center justify-center
Fill Container → w-full / flex-1
Hug Contents → w-fit
Fixed → w-[320px]

Figma Constraints → CSS Positioning:
Left & Right → left-0 right-0 (or w-full)
Center → mx-auto
Scale → w-full max-w-[value]
```

**Step 4: Responsive Breakpoints**

Map Figma frames to responsive breakpoints:
```
Figma Frame "Mobile" (375px) → default (mobile-first)
Figma Frame "Tablet" (768px) → md:
Figma Frame "Desktop" (1280px) → lg:
Figma Frame "Wide" (1440px) → xl:
```

```typescript
// Example: Card component translated from Figma
function FeatureCard({ icon, title, description }: Props) {
  return (
    <div className={cn(
      // Mobile (from Figma mobile frame)
      "flex flex-col gap-3 p-4 rounded-lg border bg-white",
      // Tablet (from Figma tablet frame)
      "md:flex-row md:gap-4 md:p-6",
      // Desktop (from Figma desktop frame)
      "lg:p-8"
    )}>
      <div className="w-10 h-10 md:w-12 md:h-12 rounded-md bg-primary-50 flex items-center justify-center">
        {icon}
      </div>
      <div className="flex flex-col gap-1">
        <h3 className="text-base md:text-lg font-semibold">{title}</h3>
        <p className="text-sm text-neutral-600">{description}</p>
      </div>
    </div>
  );
}
```

## Code Connect Setup

Map Figma components to your codebase so the Figma MCP knows how to translate:

```typescript
// figma.config.ts (or use add_code_connect_map via MCP)
const codeConnectMap = {
  "Button": {
    import: "import { Button } from '@/components/ui/button'",
    component: "Button",
    props: {
      "variant=Primary": 'variant="default"',
      "variant=Secondary": 'variant="secondary"',
      "variant=Ghost": 'variant="ghost"',
      "size=Small": 'size="sm"',
      "size=Medium": 'size="default"',
      "size=Large": 'size="lg"',
      "disabled=true": "disabled",
    },
  },
  "Input": {
    import: "import { Input } from '@/components/ui/input'",
    component: "Input",
    props: {
      "state=Error": 'className="border-red-500"',
    },
  },
  "Card": {
    import: "import { Card, CardHeader, CardContent, CardFooter } from '@/components/ui/card'",
    component: "Card",
  },
};
```

## Design System Rules

Use `create_design_system_rules` to enforce consistency:

```
Rules:
- All colors must come from the design token palette
- Spacing must use 4px/8px grid values
- Typography must use defined text styles only
- Border radius must match component-level tokens
- Icons must be from the approved icon set (Lucide)
- Shadows must use defined elevation levels
```

## Quality Checks

After generating code from Figma, verify:

1. **Pixel accuracy**: Screenshot the implementation and compare with Figma
2. **Responsive behavior**: Check all breakpoints match Figma frames
3. **Interaction states**: Hover, focus, active, disabled all match
4. **Dark mode**: If Figma has dark mode frames, verify the mapping
5. **Accessibility**: Figma doesn't enforce a11y — add roles, labels, focus states
6. **Animation**: Figma prototypes show transitions — implement with CSS/Framer Motion

## Common Translation Gotchas

| Figma | Reality |
|-------|---------|
| Absolute positioned elements | Often need `relative` parent + responsive handling |
| Fixed pixel widths | Usually need `max-width` + responsive sizes |
| Custom fonts | Need @font-face or next/font setup |
| Shadows with specific blur/spread | Map to Tailwind shadow utilities or custom CSS |
| Gradients | Tailwind `bg-gradient-to-r` or custom CSS |
| Blur/backdrop effects | `backdrop-blur-sm` (check browser support) |
| Complex SVG illustrations | Export as optimized SVG, don't recreate in code |
| Figma component variants | Map to React component props with variant pattern |
