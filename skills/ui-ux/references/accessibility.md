# Accessibility Reference

WCAG 2.1 AA compliance guide for developers.

## Semantic HTML Foundation

Use the right element for the job:

| Instead of | Use | Why |
|-----------|-----|-----|
| `<div onClick>` | `<button>` | Keyboard accessible, announces as button |
| `<div>` for nav | `<nav>` | Screen readers announce navigation |
| `<div>` for main | `<main>` | Landmark, skip-to-content target |
| `<span>` for heading | `<h1>`-`<h6>` | Heading hierarchy for screen readers |
| `<div>` for list | `<ul>`, `<ol>` | Announces list with count |
| `<a>` without href | `<button>` | Links navigate, buttons trigger actions |

### Landmark Structure
```html
<body>
  <header><nav>...</nav></header>        <!-- Site header + nav -->
  <main>                                  <!-- One per page -->
    <article>                             <!-- Self-contained content -->
      <section>                           <!-- Themed grouping -->
        <h2>...</h2>
      </section>
    </article>
  </main>
  <aside>...</aside>                      <!-- Sidebar content -->
  <footer>...</footer>                    <!-- Site footer -->
</body>
```

## Keyboard Navigation

### Requirements
- Every interactive element reachable with Tab
- Activation with Enter and/or Space
- Escape closes modals/dropdowns
- Arrow keys navigate within components (tabs, menus, lists)
- Focus never gets trapped (except modals, intentionally)

### Focus Management
```tsx
// Focus the first element when a modal opens
useEffect(() => {
  if (isOpen) {
    firstFocusableRef.current?.focus();
  }
}, [isOpen]);

// Return focus when modal closes
useEffect(() => {
  if (!isOpen && triggerRef.current) {
    triggerRef.current.focus();
  }
}, [isOpen]);
```

### Focus Indicators
```css
/* Never remove focus indicators — restyle them instead */
:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
  border-radius: 2px;
}
```

## ARIA Patterns

### Tabs
```tsx
<div role="tablist" aria-label="Settings">
  <button role="tab" aria-selected="true" aria-controls="panel-1" id="tab-1">
    General
  </button>
  <button role="tab" aria-selected="false" aria-controls="panel-2" id="tab-2">
    Security
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  General settings content
</div>
```

### Modal/Dialog
```tsx
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Deletion</h2>
  <p>Are you sure you want to delete this item?</p>
  <button onClick={onCancel}>Cancel</button>
  <button onClick={onConfirm}>Delete</button>
</div>
```

### Live Regions (Announcements)
```tsx
// For dynamic content updates (toasts, form errors, loading states)
<div aria-live="polite" aria-atomic="true" className="sr-only">
  {statusMessage}
</div>
```

### Toggle Button
```tsx
<button
  aria-pressed={isActive}
  onClick={() => setIsActive(!isActive)}
>
  {isActive ? "Active" : "Inactive"}
</button>
```

## Color Contrast

### Requirements
- **Normal text** (< 18px): 4.5:1 minimum
- **Large text** (>= 18px or >= 14px bold): 3:1 minimum
- **UI components** (borders, icons): 3:1 minimum
- **Focus indicators**: 3:1 against adjacent colors

### Common Failures
- Light gray text on white background
- Placeholder text too low contrast
- Links indistinguishable from body text (use underline too)
- Colored text on colored backgrounds without checking contrast

## Forms

### Required Elements
```tsx
<div>
  <label htmlFor="email">
    Email address <span aria-hidden="true">*</span>
    <span className="sr-only">(required)</span>
  </label>
  <input
    id="email"
    type="email"
    required
    aria-required="true"
    aria-describedby="email-error"
    aria-invalid={!!error}
  />
  {error && (
    <p id="email-error" role="alert" className="text-red-500 text-sm">
      {error}
    </p>
  )}
</div>
```

### Form Error Summary
```tsx
// Show at the top of the form after submission fails
<div role="alert" aria-live="assertive">
  <h2>Please fix the following errors:</h2>
  <ul>
    {errors.map((err) => (
      <li key={err.field}>
        <a href={`#${err.field}`}>{err.message}</a>
      </li>
    ))}
  </ul>
</div>
```

## Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```tsx
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
```

## Screen Reader Only Text

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

Use for: icon-only buttons (`<button><Icon /><span className="sr-only">Close</span></button>`), decorative images that need context, skip links.

## Testing Checklist

- [ ] Tab through the entire page — can you reach and operate everything?
- [ ] Use a screen reader (VoiceOver on Mac, NVDA on Windows)
- [ ] Check all color contrast ratios
- [ ] Test with browser zoom at 200%
- [ ] Verify reduced motion preference is respected
- [ ] Check heading hierarchy (no skipped levels)
- [ ] Test forms with keyboard only
- [ ] Verify error messages are announced
