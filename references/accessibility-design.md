# Accessibility & Design Thoughtfulness

## Accessibility baseline (aim for WCAG 2.2 AA)

- Semantic HTML first — a real `<button>`, not a `<div onClick>`; a real heading
  hierarchy; a real `<label>` on every form input.
- Everything reachable and operable by keyboard alone. Visible focus states — never
  `outline: none` without a replacement focus style.
- Color contrast meets AA (4.5:1 for body text). Never rely on color alone to convey
  state — an error field also gets an icon or text, not just a red border.
- Images get meaningful `alt` text (or `alt=""` if purely decorative). Icon-only buttons
  get an accessible label.
- Respect `prefers-reduced-motion` for any non-trivial animation.

## Design thoughtfulness (the stuff AI-generated UIs usually skip)

- Every screen that loads data has a loading state, an empty state, and an error state
  designed on purpose — not just the "it worked" state.
- Forms show inline validation errors next to the field, not just a generic banner at the
  top of the page.
- Destructive actions (delete, cancel subscription) get a confirmation step. No one-click
  permanent deletes.
- Responsive by default — test at mobile width, not just the designer's 1440px viewport.
- A consistent spacing and type scale instead of ad hoc pixel values scattered through the
  code. Even a handful of spacing/font-size design tokens pays for itself almost
  immediately.
