# Web Optimisation Principles

Reference this document at the start of every performance or optimisation task.

---

## The Prime Directive: Measure First

Never optimise without measurement. Speculation about bottlenecks is almost always wrong. Establish a baseline with real data, then optimise against it, then measure again to confirm the improvement.

- **Lab data** (Lighthouse, WebPageTest): controlled environment, reproducible, good for catching regressions.
- **Field data** (CrUX, RUM): real users on real devices and networks — what actually matters.
- Optimise for P75 and P95, not the average — the average hides the worst user experiences.

## Core Web Vitals

### LCP (Largest Contentful Paint) — target < 2.5s

What hurts it:
- Slow server response time (TTFB > 600ms)
- Render-blocking CSS and JS in `<head>`
- LCP element (hero image, heading) not preloaded or discovered late
- Unoptimised images (wrong format, no width/height, loaded lazily above the fold)

How to fix it:
- Preload the LCP image: `<link rel="preload" as="image" href="...">`
- Use `fetchpriority="high"` on the LCP `<img>`
- Never lazy-load above-the-fold images
- Serve images in WebP or AVIF, sized to the display size
- Reduce TTFB with CDN, caching, and server-side rendering where appropriate

### INP (Interaction to Next Paint) — target < 200ms

What hurts it:
- Long tasks (> 50ms) on the main thread blocking input processing
- Heavy event handlers that do too much synchronous work
- Layout thrash inside event handlers (read then write, not read-write-read-write)
- Excessive re-renders triggered by state changes

How to fix it:
- Break long tasks with `scheduler.yield()` or `setTimeout(fn, 0)`
- Debounce scroll/resize handlers
- Batch DOM reads before DOM writes — never interleave
- Move non-critical work off the critical path with web workers or idle callbacks

### CLS (Cumulative Layout Shift) — target < 0.1

What hurts it:
- Images without explicit `width` and `height` attributes
- Dynamically injected content above existing content (banners, ads)
- Web fonts causing FOUT/FOIT (flash of unstyled/invisible text)
- Animations that affect layout properties (`width`, `height`, `top`, `left`)

How to fix it:
- Always set explicit `width` and `height` on `<img>` elements
- Reserve space for dynamic content (skeleton screens, min-height)
- Use `font-display: optional` or `font-display: swap` with `size-adjust`
- Animate `transform` and `opacity` only — never layout-affecting properties

## Bundle Strategy

- Code-split at route boundaries — users should not download code for routes they haven't visited.
- Use dynamic `import()` for heavy components, libraries, or features behind interactions.
- Audit third-party dependencies: check their size on bundlephobia.com before adding. A 50kB utility library for a 3-line function is not a trade-off.
- Enable tree shaking: use named exports, avoid side-effectful imports, set `"sideEffects": false` in package.json where appropriate.
- Analyse bundle composition with source-map-explorer or rollup-plugin-visualizer before shipping.

## Image Pipeline

- **Format**: WebP for photos and complex images; AVIF where browser support allows (check baseline); SVG for icons and illustrations.
- **Sizing**: generate multiple sizes; use `srcset` and `sizes` so the browser downloads only what it needs.
- **Lazy loading**: `loading="lazy"` for all images below the fold — never for above-the-fold images.
- **Dimensions**: always set `width` and `height` to prevent CLS.
- **Quality**: 75–85% quality is visually indistinguishable from 100% at a fraction of the file size.
- **EXIF**: strip metadata before serving — reduces file size and protects privacy.

## Caching

- **Static assets with content hashes** (`app.abc123.js`): serve with `Cache-Control: public, max-age=31536000, immutable`. Never expires until the hash changes.
- **HTML**: `Cache-Control: no-cache` (revalidate on every request). HTML is cheap; stale HTML is costly.
- **API responses**: `stale-while-revalidate` for data that can tolerate brief staleness; `no-store` for sensitive or user-specific data.
- **CDN**: static assets should be served from a CDN edge node, not origin. Round-trip to origin defeats caching.

## Critical Path

- Eliminate render-blocking resources: CSS in `<head>` is blocking by nature; JS in `<head>` must be `defer` or `async` unless critical.
- Inline critical CSS (above-the-fold styles) for the first paint; load the rest asynchronously.
- Preconnect to required origins early: `<link rel="preconnect" href="https://fonts.gstatic.com">`.
- Preload critical fonts, the LCP image, and any JS that must execute before first interaction.
- Avoid `@import` in CSS — it creates serial waterfall fetches; use `<link>` tags instead.

## Runtime Performance

- **Virtualise long lists**: render only what is visible. Rendering 10,000 DOM nodes for a list with 10 visible items is never acceptable.
- **Debounce and throttle**: scroll, resize, and input handlers that trigger layout or network work must be debounced.
- **Web Workers**: move heavy computation (parsing, encoding, image processing) off the main thread.
- **Avoid layout thrash**: batch DOM reads before writes. Reading `offsetHeight` after writing `style.height` causes forced synchronous layout.
- **`will-change`**: use sparingly and only on elements that actually animate — it allocates GPU memory.
- **Passive event listeners**: `addEventListener('scroll', fn, { passive: true })` for scroll handlers that don't call `preventDefault`.

## Performance Budgets

Set budgets before you start, not after you ship:

| Metric | Budget |
|---|---|
| LCP | < 2.5s (P75, mobile) |
| INP | < 200ms (P75) |
| CLS | < 0.1 (P75) |
| Total JS (compressed) | < 150kB per route |
| Total images per page | < 500kB |
| TTFB | < 600ms |

Treat a budget breach as a failing test, not a suggestion.
