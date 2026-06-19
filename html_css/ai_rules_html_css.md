# HTML5 & CSS — AI Coding Rules

Apply these rules when generating or reviewing HTML and CSS code.

## HTML Boilerplate

- `<!DOCTYPE html>` and `<html lang="...">`.
- `<meta charset="UTF-8">` and `<meta name="viewport" content="width=device-width, initial-scale=1.0">`.
- CSS in `<head>`, JS before `</body>` or with `defer`.

## HTML Formatting

- 2 spaces indentation. Never tabs.
- 100 characters recommended line limit.
- Lowercase element names and attributes.
- Double quotes for attribute values.
- Omit closing slash on void elements: `<br>` not `<br/>`.
- Omit `type` on `<link>` and `<script>`.
- Multi-line attributes for elements with many attributes (one per line, indented).

## HTML Attribute Order

- `id`, `class`, `data-*`, `type`/`name`/`value`, `src`/`href`/`for`, `alt`/`title`, `aria-*`/`role`, boolean attributes.

## Semantic HTML

- Use semantic elements: `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`.
- `<button>` for actions, `<a>` for navigation. Never `<div onclick>`.
- One `<h1>` per page. No skipped heading levels.
- `<table>` only for tabular data, never layout. Use `<thead>`, `<tbody>`, `<th scope>`, `<caption>`.

## Forms

- Every `<input>` must have a `<label>` with matching `for`/`id`.
- Appropriate `type` attributes: `email`, `tel`, `url`, `number`, `date`.
- Native validation: `required`, `minlength`, `maxlength`, `pattern`.
- `<fieldset>` and `<legend>` for related groups.
- `<button type="submit">` not `<input type="submit">`.

## Images

- Always `alt` text. `alt=""` for decorative images.
- `width` and `height` attributes to prevent layout shift.
- `loading="lazy"` for below-the-fold images.
- `<picture>` with `<source>` for responsive images.

## CSS Formatting

- 2 spaces indentation. Never tabs.
- 80 characters recommended line limit.
- Opening brace on same line, preceded by space.
- One declaration per line. Space after colon.
- Semicolon after every declaration including last.
- Closing brace on its own line.
- One blank line between rule sets.

## CSS Declaration Order

1. Layout: `display`, `position`, `top`/`right`/`bottom`/`left`, `flex-*`, `grid-*`, `z-index`.
2. Box model: `width`, `height`, `margin`, `padding`, `border`, `overflow`.
3. Typography: `font-*`, `line-height`, `text-*`, `color`.
4. Visual: `background-*`, `box-shadow`, `opacity`, `transform`.
5. Animation: `transition`, `animation`.

## CSS Naming (BEM)

- Block: `.card` (standalone component, `kebab-case`).
- Element: `.card__title` (double underscore).
- Modifier: `.card--featured` (double hyphen).
- State: `.is-active`, `.is-open`, `.has-image`.
- JS hooks: `.js-toggle-menu` (never styled).

## Selectors & Specificity

- Keep specificity low and flat. Prefer class selectors.
- No `!important` except utility overrides.
- No inline styles except via JS for dynamic values.
- Max 3 levels of nesting. No deeply nested selectors.
- No tag-qualifying class selectors: `.card` not `div.card`.
- No ID selectors for styling.

## CSS Custom Properties (Design Tokens)

- Define in `:root`: `--color-primary`, `--font-size-base`, `--spacing-md`, `--radius-md`.
- Dark mode via `@media (prefers-color-scheme: dark)` overriding `:root` variables.
- Component-scoped properties for variant customisation.

## Units & Values

- `rem` for font sizes and spacing. `em` for component-relative.
- `px` for borders. `%` or viewport units for responsive sizes.
- Omit units on zero: `margin: 0;`. Omit leading zero: `opacity: .5;`.
- Lowercase hex colours. Shorthand when possible: `#abc`.

## Responsive Design

- Mobile-first: base styles for mobile, `@media (min-width: ...)` for larger.
- Breakpoints in `rem`: 30rem (480px), 48rem (768px), 64rem (1024px), 80rem (1280px).
- Container queries for component-level responsiveness.
- `clamp()` for responsive typography.

## Modern CSS

- Flexbox for one-dimensional layout. Grid for two-dimensional.
- CSS Nesting with `&` — limit to 2-3 levels.
- `@layer` for cascade layer management.
- Logical properties (`margin-block-end`, `padding-inline`) for internationalisation.
- Subgrid for aligned nested grids.

## Accessibility (WCAG 2.1 AA)

- 4.5:1 contrast ratio for normal text. 3:1 for large text.
- All interactive elements keyboard-accessible.
- Visible focus indicators — never `outline: none` without replacement.
- ARIA only when no semantic HTML element works.
- `aria-label`, `aria-describedby`, `aria-expanded`, `aria-hidden` for state.
- `.sr-only` class for screen-reader-only content.
- Never convey information by colour alone.

## Performance

- Minimise CSS. Remove unused rules (PurgeCSS).
- `loading="lazy"` on below-the-fold images and iframes.
- Explicit `width`/`height` on images for CLS prevention.
- `<link rel="preload">` for critical assets. `fetchpriority="high"` on LCP images.
- `font-display: swap`. `woff2` format. Subset fonts.
- `transform`/`opacity` for animations (GPU-composited).

## Security & XSS Prevention

- Never insert untrusted content into HTML without escaping. Use `textContent`, not `innerHTML`, for user content.
- Use framework auto-escaping (React, Angular, Vue). Never bypass it (`dangerouslySetInnerHTML`, `v-html`) without sanitization.
- Never use `document.write()` with user content.
- Content Security Policy (CSP) headers to restrict script sources. Avoid `'unsafe-inline'` and `'unsafe-eval'`.
- Sanitize user-generated HTML with a whitelist-based sanitizer (DOMPurify).
- `rel="noopener noreferrer"` on all `target="_blank"` links (modern browsers imply `noopener`, but include explicitly).
- Validate URLs in `href` and `src` — never allow `javascript:` protocol from user input.

## Defensive Programming & Input Validation

- Validate all form input on the client side AND the server side. Client validation is UX, not security.
- HTML5 validation attributes: `required`, `minlength`, `maxlength`, `min`, `max`, `pattern`, appropriate `<input type>` (`email`, `url`, `tel`, `number`, `date`).
- `<datalist>` for constrained free-text input.
- Accessible error states: `aria-invalid="true"`, `aria-describedby` linking to error messages, `aria-live="polite"` on an always-present error container (prefer over `role="alert"`, which is `aria-live="assertive"` and unreliable on initially-hidden elements).
- Subresource Integrity (`integrity` attribute) on third-party scripts and stylesheets.
- `sandbox` attribute on `<iframe>` elements. Validate URLs — never allow `javascript:` protocol.
- Escape all user content rendered in HTML. Use framework auto-escaping.

## Documentation

- Section comments for major component groups.
- Document non-obvious CSS (workarounds, magic numbers, browser hacks).
- No commented-out CSS in production.

## CSS Architecture

- Component-based: `base/`, `components/`, `layouts/`, `utilities/` directories.
- Entry point `main.css` imports all partials.
- Modern reset with `box-sizing: border-box` globally.

## Tooling

- Stylelint (CSS linter), Prettier (formatter).
- HTMLHint / html-validate (HTML linter).
- axe-core / Lighthouse (accessibility).
- PostCSS / Lightning CSS (post-processing).
- PurgeCSS (unused CSS removal).
- W3C Validator (standards compliance).

## Build Tools

- npm scripts in package.json for dev, build, test tasks.
- Vite (recommended): modern dev server, fast production builds.
- PostCSS for CSS processing (autoprefixer, nesting, minification).
- Webpack for complex/legacy projects.
- Bundlers handle asset optimization, minification, code splitting.
- Docker multi-stage builds for static site hosting.

## SBOM Creation

- Generate SBOM for npm/Yarn JavaScript dependencies with `@cyclonedx/npm`.
- Document third-party CDN assets (versions, licenses) separately.
- Verify Subresource Integrity (`integrity` attribute) on all external resources.
- Run `npm audit` on JavaScript dependencies.
- Store SBOM and audit reports with release.
- Monitor third-party libraries for CVEs.
