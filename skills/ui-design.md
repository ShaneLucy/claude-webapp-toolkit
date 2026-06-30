# UI Design, Accessibility & Polish Principles

Reference this document at the start of every design, UI implementation, or polish task.

---

## Design Fundamentals

### Visual Hierarchy

Hierarchy tells users what to look at first, second, and third. Establish it through:
- **Size**: larger elements draw the eye first.
- **Weight**: bold text signals importance.
- **Colour**: high-contrast elements appear more prominent.
- **Spacing**: generous whitespace around an element elevates its importance.
- **Position**: users read top-left to bottom-right (in LTR contexts) — place critical elements accordingly.

Never use size, weight, colour, and animation simultaneously to emphasise something — pick one or two signals. When everything shouts, nothing is heard.

### Spacing

- Use an **8-point grid** as the base unit: all spacing values should be multiples of 4 or 8 (4, 8, 16, 24, 32, 48, 64...).
- Prefer spacing tokens (`--space-sm`, `--space-md`) over hardcoded values — consistency is what makes spacing feel intentional.
- **Whitespace is not wasted space.** Cramped layouts signal amateur work; generous padding signals quality.
- Related elements get less space between them than unrelated elements (law of proximity).

### Typography

- **Measure (line length)**: 60–80 characters for body text. Shorter is fine for narrow components; longer hurts readability.
- **Line height**: 1.5 for body text, 1.2–1.3 for headings. Tighter for display sizes, looser for small text.
- **Type scale**: use a modular scale (e.g., Major Third: 1.25×) with named tokens. Don't invent arbitrary sizes.
- **Variable fonts**: prefer variable fonts — one file, infinite flexibility, better compression.
- `text-wrap: balance` on headings prevents orphaned words on the last line.
- Avoid justified text — it creates uneven word spacing that hurts readability.

### Colour

- Always use **semantic colour tokens** (`--color-text-primary`, `--color-surface-elevated`), never raw hex/rgb values. Tokens make dark mode and theming maintainable.
- **Contrast ratios** (WCAG 2.2):
  - Normal text (< 18px or < 14px bold): minimum 4.5:1
  - Large text (≥ 18px or ≥ 14px bold): minimum 3:1
  - UI components and icons: minimum 3:1
- Never convey information through colour alone — always pair colour with a shape, icon, or text label.
- Test all colour combinations in `forced-colors` mode (Windows High Contrast) — CSS custom properties are not inherited in forced-colors; use `currentColor` and `ButtonText`/`LinkText` system colours as fallbacks.

---

## Component Design

### States

Every interactive element must have a designed state for each of:

| State | What it communicates |
|---|---|
| Default | The element exists and is actionable |
| Hover | The element will respond to a click |
| Focus | Keyboard users know where they are |
| Active (pressed) | The action is happening |
| Disabled | The element exists but cannot be used now |
| Loading | An action has been triggered and is in progress |
| Empty | The container has no content (not an error) |
| Error | Something went wrong — show what and how to recover |

An unstyled or missing state is a design bug, not a missing enhancement.

### Interactive Element Sizing

- Minimum touch/click target: **44×44px** (WCAG 2.2 SC 2.5.8).
- Prefer larger targets for primary actions — 48×48px or larger.
- If a visual element is smaller than 44px, extend the clickable area with padding or pseudo-elements without changing the visual.

---

## Accessibility (WCAG 2.2 AA Minimum)

### Keyboard Navigation

- Every interactive element must be reachable and operable via keyboard alone.
- Tab order must follow the visual reading order — never reorder visually with CSS in a way that breaks keyboard flow.
- Provide a **skip link** (`<a href="#main-content" class="skip-link">Skip to main content</a>`) as the first focusable element on every page.
- Modal dialogs must trap focus inside while open and restore focus to the trigger element on close.
- Dropdown menus must close on `Escape`.

### Focus Indicators

- Use `:focus-visible` — not `:focus` alone (`:focus` fires on mouse click too, which clutters the UI).
- **Never `outline: none` without a replacement.** Removing focus indicators is a WCAG 2.2 SC 2.4.11 failure.
- Focus rings must meet 3:1 contrast against adjacent colours.
- Ensure focus rings are not clipped by `overflow: hidden` on parent elements — use `overflow: clip` with `clip-path` as an alternative, or adjust the outline with `outline-offset`.

### Semantic HTML

- Use the right element for the job — `<button>` for actions, `<a href>` for navigation. Never swap these.
- Every page has exactly one `<h1>`. Heading hierarchy must be logical and unbroken (no skipping from `<h2>` to `<h4>`).
- Use `<nav>` for navigation, `<main>` for primary content, `<header>`, `<footer>`, `<aside>`, `<article>`, `<section>` with intent.
- Use `<ul>/<ol>` for lists — never fake lists with divs.
- `<img>` elements always have `alt`. Decorative images use `alt=""`.

### ARIA

- Prefer native HTML semantics over ARIA. `role="button"` on a `<div>` is always wrong when you can use `<button>`.
- ARIA roles, states, and properties must be accurate — incorrect ARIA is worse than no ARIA.
- Use `aria-live="polite"` for dynamic content that updates without a page navigation (status messages, search results counts).
- Use `aria-busy="true"` on containers while their content is loading.
- `aria-label` and `aria-labelledby` provide accessible names for elements without visible labels.
- `aria-expanded`, `aria-controls`, `aria-haspopup` for disclosure patterns (accordions, dropdowns).
- `aria-current="page"` for the active navigation link.

### Content

- Language set on `<html lang="en">` (or appropriate language code). Mark inline language changes with `lang` attribute.
- No content flashes more than 3 times per second (seizure prevention).
- Error messages are associated with their input via `aria-describedby` and announced to screen readers.
- Success/status messages use `role="status"` or an `aria-live` region.

---

## Motion & Animation

- **All animations and transitions must respect `prefers-reduced-motion`.**

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

- Animate `transform` and `opacity` — they are GPU-composited and don't trigger layout or paint.
- Never animate `width`, `height`, `top`, `left`, `margin`, or `padding` — these cause layout recalculation.
- **Timing guidelines**:
  - Instant feedback (hover, press): 100–150ms
  - UI transitions (panel slide, modal open): 200–300ms
  - Complex choreography: up to 400ms
  - Nothing in the UI should take > 500ms to animate
- **Easing**: `ease-out` for elements entering the screen (fast start, slow finish feels responsive); `ease-in` for elements leaving.
- Motion must be purposeful — it should guide attention or communicate state, not decorate.

---

## Responsive Design

- **Mobile-first**: design the constrained (mobile) layout first, then enhance for larger viewports. `min-width` media queries build up, not down.
- Prefer **intrinsic layouts** (CSS Grid `auto-fill`, `minmax`, `clamp()`) over breakpoint-heavy approaches — they reduce the number of explicit breakpoints needed.
- Use `clamp()` for fluid typography: `font-size: clamp(1rem, 0.5rem + 2.5vw, 1.5rem)`.
- Use `clamp()` or `min()` for fluid spacing and container widths too.
- Test at real device sizes, not just arbitrary breakpoints — 320px (small phone), 375px, 428px, 768px (tablet), 1280px, 1440px, 1920px.

---

## Cross-Browser Compatibility

- Check CSS feature support on Baseline (web.dev/baseline) before using new features.
- **Progressive enhancement**: core content and functionality must work without the latest CSS features. Enhancements layer on top.
- Test in: Chrome (latest), Firefox (latest), Safari (latest, on macOS and iOS), Samsung Internet.
- Test in **Windows High Contrast / forced-colors mode** — colour custom properties are ignored; ensure semantic colour usage.
- Test with keyboard-only navigation in every browser.
- Use `@supports` for CSS feature detection when providing fallbacks.

## Dark Mode

- Every colour token must have a dark-mode value defined under `@media (prefers-color-scheme: dark)` or a `[data-theme="dark"]` selector.
- Don't just invert — dark mode needs its own deliberate contrast ratios. An inversion of a light palette rarely meets contrast requirements.
- Reduce the brightness of images in dark mode slightly: `filter: brightness(0.85)` on `<img>` in dark contexts.
- Test contrast ratios in both light and dark modes independently.

---

## Polish Checklist

Before marking a component or page as complete:

- [ ] All interactive states designed and implemented (hover, focus, active, disabled, loading, empty, error)
- [ ] Focus rings visible, 3:1 contrast, not clipped
- [ ] Hover transitions feel instant (≤ 150ms)
- [ ] `prefers-reduced-motion` suppresses all animations
- [ ] Heading hierarchy is logical and unbroken
- [ ] No orphaned words in headings (`text-wrap: balance`)
- [ ] Consistent border-radius across components
- [ ] Icons baseline-aligned with accompanying text
- [ ] Touch targets ≥ 44×44px
- [ ] Dark mode tested — contrast meets 4.5:1 for text
- [ ] Forced-colors mode tested — no information lost
- [ ] Keyboard navigation tested — logical tab order, no traps
- [ ] Screen reader tested (or at minimum: semantic HTML + ARIA verified correct)
- [ ] Empty states are designed (not blank white boxes)
- [ ] Error states show what went wrong and how to fix it
