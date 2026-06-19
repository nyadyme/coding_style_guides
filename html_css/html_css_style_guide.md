# HTML5 & CSS Coding Style Guidelines

A comprehensive guide rooted in the WHATWG HTML Living Standard, W3C CSS
specifications, Google HTML/CSS Style Guide, MDN Web Docs, and established
software engineering literature — notably *CSS: The Definitive Guide*
(Meyer & Weyl, 2017) and *Clean Code* (Martin, 2008).

---

## Table of Contents

1. [Philosophy](#1-philosophy)
2. [HTML Layout & Formatting](#2-html-layout--formatting)
3. [HTML Best Practices](#3-html-best-practices)
4. [CSS Layout & Formatting](#4-css-layout--formatting)
5. [CSS Naming Conventions](#5-css-naming-conventions)
6. [CSS Selectors & Specificity](#6-css-selectors--specificity)
7. [CSS Architecture](#7-css-architecture)
8. [Responsive Design](#8-responsive-design)
9. [Accessibility](#9-accessibility)
10. [Performance](#10-performance)
11. [Documentation & Comments](#11-documentation--comments)
12. [CSS Custom Properties](#12-css-custom-properties)
13. [Modern CSS Features](#13-modern-css-features)
14. [Security & XSS Prevention](#14-security--xss-prevention)
15. [Defensive Programming & Input Validation](#15-defensive-programming--input-validation)
16. [Project Structure](#16-project-structure)
17. [Tooling](#17-tooling)
18. [Build Tools](#18-build-tools)
19. [SBOM Creation](#19-sbom-creation)
20. [References](#20-references)

---

## 1. Philosophy

### 1.1 Core Values

HTML and CSS are the foundation of the web. Prioritise **semantics**,
**accessibility**, **progressive enhancement**, and **maintainability**.

### 1.2 Guiding Principles

| Principle | Source | Summary |
|---|---|---|
| **Semantic HTML** | WHATWG spec, MDN | Use elements for their meaning, not their appearance. |
| **Progressive enhancement** | Web standards community | Build from a solid HTML foundation; layer CSS and JS on top. |
| **Separation of concerns** | Web architecture | Structure (HTML), presentation (CSS), behaviour (JS) are separate. |
| **Accessibility first** | WCAG 2.1 AA | The web is for everyone. Accessibility is not optional. |
| **Mobile-first** | Responsive design best practices | Design for the smallest screen first, enhance upward. |
| **Performance budget** | Web performance community | Every byte matters. Minimise, compress, lazy-load. |

---

## 2. HTML Layout & Formatting

### 2.1 Indentation

- **2 spaces** per indentation level. Never tabs.

### 2.2 Line Length

- **100 characters** recommended maximum. Break long lines after attributes.

### 2.3 Document Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header>
    <nav aria-label="Main navigation">
      <!-- navigation -->
    </nav>
  </header>

  <main>
    <article>
      <h1>Article Title</h1>
      <p>Content here.</p>
    </article>
  </main>

  <footer>
    <p>&copy; 2025 Company Name</p>
  </footer>

  <script src="app.js"></script>
</body>
</html>
```

### 2.4 Attribute Formatting

Single line for few attributes:

```html
<input type="email" id="email" name="email" required>
```

Multi-line for many attributes:

```html
<input
  type="email"
  id="user-email"
  name="user-email"
  class="form-input"
  placeholder="you@example.com"
  aria-describedby="email-help"
  required
>
```

### 2.5 Attribute Order

Use a consistent order:

1. `id`
2. `class`
3. `data-*`
4. `type`, `name`, `value` (form elements)
5. `src`, `href`, `for` (references)
6. `alt`, `title` (descriptions)
7. `aria-*`, `role`
8. Boolean attributes (`required`, `disabled`, `hidden`)

### 2.6 General HTML Formatting Rules

- Use lowercase for all element names and attributes.
- Omit the closing slash on void elements: `<br>` not `<br/>`.
- Always quote attribute values with double quotes.
- Omit `type` on `<link>` and `<script>` (HTML5 defaults are correct).
- No trailing whitespace. No trailing blank lines.

---

## 3. HTML Best Practices

### 3.1 Semantic Elements

| Instead of | Use |
|---|---|
| `<div class="header">` | `<header>` |
| `<div class="nav">` | `<nav>` |
| `<div class="main">` | `<main>` |
| `<div class="footer">` | `<footer>` |
| `<div class="article">` | `<article>` |
| `<div class="sidebar">` | `<aside>` |
| `<div class="section">` | `<section>` |
| `<span class="button">` | `<button>` |
| `<div onclick="...">` | `<button>` or `<a>` |

### 3.2 Headings

- One `<h1>` per page.
- Heading levels must not skip: `<h1>` → `<h2>` → `<h3>`, never
  `<h1>` → `<h3>`.
- Use headings for structure, not for styling.

### 3.3 Forms

```html
<form action="/login" method="post">
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required>
  </div>

  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" id="password" name="password"
      minlength="8" required>
  </div>

  <button type="submit">Log In</button>
</form>
```

- Every `<input>` must have a `<label>` with matching `for`/`id`.
- Use appropriate `type` attributes: `email`, `tel`, `url`, `number`, `date`.
- Use `required`, `minlength`, `maxlength`, `pattern` for validation.
- Use `<fieldset>` and `<legend>` for related groups.
- Use `<button type="submit">` not `<input type="submit">`.

### 3.4 Images

```html
<img
  src="photo.jpg"
  alt="A sunset over the mountains"
  width="800"
  height="600"
  loading="lazy"
>
```

- Always provide `alt` text. Use `alt=""` for decorative images.
- Set `width` and `height` to prevent layout shift.
- Use `loading="lazy"` for below-the-fold images.
- Use `<picture>` with `<source>` for responsive images.

### 3.5 Links

- Use `<a>` for navigation. Use `<button>` for actions.
- Links opening in a new tab: `target="_blank" rel="noopener"`.
- Descriptive link text. Never "click here".

### 3.6 Tables

- Use `<table>` only for tabular data, never for layout.
- Use `<thead>`, `<tbody>`, `<tfoot>`, `<th scope="col">`, `<caption>`.

---

## 4. CSS Layout & Formatting

### 4.1 Indentation

- **2 spaces** per indentation level. Never tabs.

### 4.2 Line Length

- **80 characters** recommended maximum for declarations.

### 4.3 General Formatting

```css
/* One declaration per line, space after colon, semicolon on every declaration */
.card {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  border: 1px solid var(--color-border);
  border-radius: 0.5rem;
  background-color: var(--color-surface);
}
```

- Opening brace on same line as selector, preceded by a space.
- One declaration per line.
- Space after the colon.
- Semicolon after every declaration (including the last).
- Closing brace on its own line.
- One blank line between rule sets.
- No blank line after opening brace or before closing brace.

### 4.4 Declaration Order

Group declarations in a consistent order:

1. **Layout** — `display`, `position`, `top`/`right`/`bottom`/`left`,
   `float`, `flex-*`, `grid-*`, `z-index`
2. **Box model** — `width`, `height`, `margin`, `padding`, `border`,
   `overflow`
3. **Typography** — `font-*`, `line-height`, `text-*`, `color`
4. **Visual** — `background-*`, `box-shadow`, `opacity`, `transform`
5. **Animation** — `transition`, `animation`
6. **Misc** — `cursor`, `pointer-events`, `user-select`

### 4.5 Shorthand Properties

Use shorthand when setting all values:

```css
/* Yes */
margin: 1rem 2rem;
padding: 0.5rem 1rem;          /* vertical, horizontal */

/* Only use longhand when setting a single side */
margin-bottom: 2rem;
```

### 4.6 Values and Units

- Use `rem` for font sizes and spacing (relative to root).
- Use `em` for component-relative spacing.
- Use `px` for borders and fine-grained control.
- Use `%` or viewport units (`vw`, `vh`, `dvh`) for responsive sizes.
- Omit units on zero values: `margin: 0;` not `margin: 0px;`.
- Omit the leading zero: `opacity: .5;` not `opacity: 0.5;`.
- Use lowercase hex colours: `#1a2b3c`. Use shorthand when possible:
  `#abc` for `#aabbcc`.

---

## 5. CSS Naming Conventions

### 5.1 BEM (Block Element Modifier)

```css
/* Block */
.card { }

/* Element (part of block) */
.card__title { }
.card__body { }
.card__footer { }

/* Modifier (variation of block or element) */
.card--featured { }
.card__title--large { }
```

- Block: standalone component name in `kebab-case`.
- Element: `__` double underscore separator.
- Modifier: `--` double hyphen separator.
- Never nest BEM selectors deeper than one element level.

### 5.2 Utility Classes

```css
.text-center { text-align: center; }
.mt-4 { margin-top: 1rem; }
.hidden { display: none; }
.sr-only { /* screen-reader only */ }
```

- Use sparingly. Prefer component classes for complex styling.
- If using a utility framework (Tailwind), follow its conventions
  consistently.

### 5.3 State Classes

Prefix state classes with `is-` or `has-`:

```css
.nav-item.is-active { }
.form-input.is-invalid { }
.dropdown.is-open { }
.card.has-image { }
```

### 5.4 JavaScript Hook Classes

Prefix JS-only selectors with `js-`:

```html
<button class="btn js-toggle-menu">Menu</button>
```

Never style `js-` classes. They exist only for JavaScript targeting.

---

## 6. CSS Selectors & Specificity

### 6.1 Specificity Rules

- Keep specificity low and flat. Prefer class selectors (`.card`) over
  ID selectors (`#card`) and element selectors (`div`).
- Never use `!important` except for utility overrides.
- Never use inline styles except via JavaScript for dynamic values.
- Maximum **3 levels** of selector nesting (in preprocessors).

### 6.2 Selector Guidelines

```css
/* Good — low specificity, clear intent */
.nav-item { }
.nav-item.is-active { }

/* Avoid — high specificity, fragile */
#main-nav > ul > li > a.active { }
div.container div.row div.col { }
```

- Prefer single class selectors.
- Avoid tag-qualifying class selectors: `.card` not `div.card`.
- Avoid descendant selectors when child selectors work: `.nav > .item`
  over `.nav .item`.
- Never select by element alone in component CSS.

---

## 7. CSS Architecture

### 7.1 Component-Based CSS

Organise CSS around components, not pages:

```
styles/
    base/
        _reset.css
        _typography.css
        _variables.css
    components/
        _button.css
        _card.css
        _navbar.css
    layouts/
        _grid.css
        _header.css
        _footer.css
    utilities/
        _spacing.css
        _visibility.css
    main.css
```

### 7.2 Layering with @layer (CSS Cascade Layers)

```css
@layer reset, base, components, utilities;

@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

@layer base {
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.5;
  }
}

@layer components {
  .card { /* ... */ }
}

@layer utilities {
  .hidden { display: none; }
}
```

### 7.3 Reset / Normalize

- Use a modern CSS reset or `normalize.css`.
- Apply `box-sizing: border-box` globally.

---

## 8. Responsive Design

### 8.1 Mobile-First

Write base styles for mobile, then add breakpoints for larger screens:

```css
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@media (min-width: 48rem) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 64rem) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### 8.2 Breakpoints

Use consistent, content-driven breakpoints in `rem` or `em`:

| Name | Value | Typical use |
|---|---|---|
| Small | `30rem` (480px) | Large phones |
| Medium | `48rem` (768px) | Tablets |
| Large | `64rem` (1024px) | Laptops |
| Extra large | `80rem` (1280px) | Desktops |

### 8.3 Container Queries (Modern CSS)

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 30rem) {
  .card {
    flex-direction: row;
  }
}
```

### 8.4 Responsive Images

```html
<picture>
  <source media="(min-width: 64rem)" srcset="hero-large.webp">
  <source media="(min-width: 48rem)" srcset="hero-medium.webp">
  <img src="hero-small.webp" alt="Hero image">
</picture>
```

### 8.5 Responsive Typography

```css
:root {
  font-size: clamp(1rem, 0.9rem + 0.5vw, 1.25rem);
}
```

---

## 9. Accessibility

### 9.1 WCAG 2.1 AA Compliance

- **Perceivable**: alt text, captions, sufficient contrast.
- **Operable**: keyboard navigable, no keyboard traps, skip links.
- **Understandable**: consistent navigation, clear labels, error messages.
- **Robust**: valid HTML, ARIA where needed.

### 9.2 ARIA

- First rule of ARIA: don't use ARIA if a native HTML element works.
- Use `aria-label`, `aria-describedby`, `aria-labelledby` for context.
- Use `role` only when no semantic HTML element exists.
- Use `aria-live` regions for dynamic content updates.
- Use `aria-expanded`, `aria-hidden`, `aria-current` for state.

### 9.3 Keyboard Navigation

- All interactive elements must be focusable and operable via keyboard.
- Visible focus indicators — never `outline: none` without a replacement.
- Logical tab order matching visual order.
- Skip-to-content link for long navigation.

### 9.4 Colour and Contrast

- Minimum 4.5:1 contrast ratio for normal text (WCAG 1.4.3 Contrast Minimum).
- Minimum 3:1 contrast ratio for large text (>=24px, or >=18.66px bold).
- Minimum 3:1 contrast for UI components and graphical objects
  (WCAG 1.4.11 Non-text Contrast — a separate criterion from large text).
- Never convey information by colour alone (WCAG 1.4.1 Use of Color).
- Test with colour blindness simulators.

### 9.5 Screen Reader Only Content

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
  border: 0;
}
```

---

## 10. Performance

### 10.1 CSS Performance

- Minimise CSS file size. Remove unused CSS.
- Avoid deeply nested selectors (3+ levels).
- Avoid universal selectors (`*`) in compound selectors.
- Use `will-change` sparingly and only when needed.
- Prefer `transform` and `opacity` for animations (GPU-composited).

### 10.2 HTML Performance

- Load CSS in `<head>`. Load JavaScript before `</body>` or use `defer`.
- Use `loading="lazy"` on images and iframes below the fold.
- Set explicit `width` and `height` on images to prevent CLS.
- Preload critical assets: `<link rel="preload" href="..." as="...">`.
- Use `fetchpriority="high"` on LCP images.

### 10.3 Font Performance

```css
@font-face {
  font-family: "CustomFont";
  src: url("font.woff2") format("woff2");
  font-display: swap;
}
```

- Use `woff2` format. Subset fonts to required characters.
- Use `font-display: swap` to prevent FOIT.
- Preload critical fonts.

### 10.4 Critical CSS

- Inline critical above-the-fold CSS in `<style>`.
- Load remaining CSS asynchronously.

---

## 11. Documentation & Comments

### 11.1 CSS Comments

```css
/* ==========================================================================
   Component: Card
   ========================================================================== */

/**
 * Card component for displaying content in a contained box.
 *
 * Variants:
 * - .card--featured: highlighted card with accent border
 * - .card--compact: reduced padding for dense layouts
 */
.card {
  /* ... */
}

/* Offset the negative margin from the grid gap */
.card__body {
  margin-top: -0.5rem;
}
```

- Use section headers for major component groups.
- Document non-obvious decisions (workarounds, magic numbers, browser hacks).
- Never leave commented-out CSS in production.

### 11.2 HTML Comments

- Use comments to mark major sections: `<!-- Header -->`, `<!-- Main Content -->`.
- Keep comments brief. HTML comments are sent to the client.

---

## 12. CSS Custom Properties

### 12.1 Design Tokens

```css
:root {
  /* Colours */
  --color-primary: #2563eb;
  --color-primary-dark: #1d4ed8;
  --color-text: #1f2937;
  --color-text-muted: #6b7280;
  --color-background: #ffffff;
  --color-surface: #f9fafb;
  --color-border: #e5e7eb;

  /* Typography */
  --font-family-base: system-ui, -apple-system, sans-serif;
  --font-family-mono: ui-monospace, monospace;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.25rem;
  --font-size-xl: 1.5rem;

  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  /* Borders */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, .05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, .1);
}
```

### 12.2 Dark Mode

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-text: #f9fafb;
    --color-background: #111827;
    --color-surface: #1f2937;
    --color-border: #374151;
  }
}
```

### 12.3 Component-Scoped Properties

```css
.button {
  --button-padding: var(--spacing-sm) var(--spacing-md);
  --button-radius: var(--radius-md);

  padding: var(--button-padding);
  border-radius: var(--button-radius);
}

.button--large {
  --button-padding: var(--spacing-md) var(--spacing-lg);
}
```

---

## 13. Modern CSS Features

### 13.1 Flexbox

```css
.flex-row {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.flex-col {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}
```

### 13.2 Grid

```css
.page-layout {
  display: grid;
  grid-template-columns: 16rem 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  min-height: 100dvh;
}
```

### 13.3 Logical Properties

```css
/* Prefer logical over physical for internationalisation */
.card {
  margin-block-end: 1rem;     /* margin-bottom */
  padding-inline: 1rem;        /* padding-left + padding-right */
  border-inline-start: 3px solid var(--color-primary);  /* border-left in LTR */
}
```

### 13.4 Nesting (CSS Nesting)

```css
.card {
  padding: 1rem;

  & .card__title {
    font-size: var(--font-size-lg);
  }

  &:hover {
    box-shadow: var(--shadow-md);
  }

  &.card--featured {
    border-color: var(--color-primary);
  }
}
```

- Limit nesting to 2–3 levels.
- Use `&` explicitly for clarity.

### 13.5 Subgrid

```css
/* Parent defines explicit tracks the child can adopt */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, auto);
  gap: 1rem;
}

/* Child spans 3 rows of the parent and adopts those row tracks via
   `grid-template-rows: subgrid`, so descendants align to the outer grid. */
.grid-item {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid;
}
```

### 13.6 Color Functions

```css
:root {
  --color-primary: oklch(55% .25 260);
  --color-primary-light: oklch(from var(--color-primary) calc(l + .15) c h);
}
```

---

## 14. Security & XSS Prevention

Cross-site scripting (XSS) is the HTML/CSS equivalent of SQL injection —
the most common and dangerous web vulnerability. Prevent it by treating
all user-supplied content as untrusted.

### 14.1 Never Insert Untrusted Content Without Escaping

```html
<!-- Bad — raw user content injected into HTML -->
<div id="output"></div>
<script>
  document.getElementById("output").innerHTML = userInput; // XSS
</script>

<!-- Good — use textContent for plain text -->
<div id="output"></div>
<script>
  document.getElementById("output").textContent = userInput; // safe
</script>
```

- Never use `innerHTML` or `document.write()` with user-supplied content.
- Use `textContent` for inserting plain text into the DOM.
- Use framework auto-escaping (React, Angular, Vue handle this by
  default). Do not bypass it (`dangerouslySetInnerHTML`, `[innerHTML]`,
  `v-html`) unless the content is sanitized.

### 14.2 Content Security Policy (CSP)

Use CSP headers to restrict the sources from which scripts, styles, and
other resources can be loaded:

```html
<meta http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'">
```

- Prefer HTTP response headers over `<meta>` tags for CSP.
- Avoid `'unsafe-inline'` and `'unsafe-eval'` for script sources.
- Use nonce-based or hash-based CSP for inline scripts when necessary.

### 14.3 Sanitize User-Generated HTML

When user-generated HTML must be rendered (rich text editors, markdown),
use a whitelist-based sanitizer:

- Use **DOMPurify** for client-side sanitization.
- Sanitize on the server side as well — client-side sanitization is a
  defence-in-depth measure, not the sole protection.
- Whitelist allowed tags and attributes. Strip everything else.

### 14.4 Safe Link Practices

- Use `rel="noopener"` on all links that open new tabs
  (`target="_blank"`).
- Validate URLs in `href` and `src` attributes — never allow
  `javascript:` or `data:` protocols from user input.

```html
<!-- Safe — noopener prevents the new page from accessing window.opener;
     noreferrer also strips the Referer header. Modern browsers imply
     noopener for target="_blank" by default, but include it explicitly. -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  External Link
</a>
```

---

## 15. Defensive Programming & Input Validation

### 15.1 Client-Side and Server-Side Validation

Validate all form input on the client side **and** the server side.
Client-side validation improves user experience but is not a security
boundary — it can always be bypassed.

### 15.2 HTML5 Validation Attributes

Use built-in HTML5 validation attributes for immediate user feedback:

```html
<form action="/register" method="post" novalidate>
  <div class="form-group">
    <label for="username">Username</label>
    <input
      type="text"
      id="username"
      name="username"
      required
      minlength="3"
      maxlength="30"
      pattern="[a-zA-Z0-9_]+"
      aria-describedby="username-error"
    >
    <!-- aria-live on the always-present container so screen readers
         announce text inserted into it. role="alert" on a hidden
         element is unreliable across screen readers. -->
    <p id="username-error" class="error-message" aria-live="polite"></p>
  </div>

  <div class="form-group">
    <label for="email">Email</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      aria-describedby="email-error"
    >
    <p id="email-error" class="error-message" aria-live="polite"></p>
  </div>

  <div class="form-group">
    <label for="age">Age</label>
    <input
      type="number"
      id="age"
      name="age"
      min="13"
      max="120"
      required
      aria-describedby="age-error"
    >
    <p id="age-error" class="error-message" aria-live="polite"></p>
  </div>

  <button type="submit">Register</button>
</form>
```

```css
.form-group input:invalid {
  border-color: var(--color-error, #dc2626);
}

.error-message {
  color: var(--color-error, #dc2626);
  font-size: var(--font-size-sm, .875rem);
  margin-top: var(--spacing-xs, .25rem);
}
```

- Use appropriate `<input type>`: `email`, `url`, `tel`, `number`, `date`
  for built-in validation.
- Use `required`, `minlength`, `maxlength`, `min`, `max`, `pattern` for
  constraints.
- Use `<datalist>` for constrained free-text input.

### 15.3 Accessible Error States

- Use `aria-invalid="true"` on fields that fail validation.
- Use `aria-describedby` to associate error messages with their fields.
- Display clear, specific error messages near the invalid field.
- For dynamic error messages, prefer `aria-live="polite"` on an
  always-present container into which text is inserted. `role="alert"`
  (implicitly `aria-live="assertive"`) interrupts the user and is
  unreliable on elements that are initially hidden — toggling `hidden`
  off does not consistently trigger announcement.

### 15.4 Resource Integrity and Sandboxing

- Use `Subresource Integrity` (`integrity` attribute) on third-party
  scripts and stylesheets to detect tampering.

```html
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxAh6VgnRk+dNNKh..."
  crossorigin="anonymous"
></script>
```

- Use the `sandbox` attribute on `<iframe>` elements to restrict their
  capabilities.

```html
<iframe src="widget.html" sandbox="allow-scripts allow-same-origin"></iframe>
```

- Validate URLs in `href` and `src` — never allow `javascript:` protocol.
- Escape all user content rendered in HTML. Use framework auto-escaping.

---

## 16. Project Structure

### 16.1 Static Sites

```
project/
    index.html
    pages/
        about.html
        contact.html
    styles/
        main.css
        base/
        components/
        layouts/
        utilities/
    scripts/
        app.js
    images/
    fonts/
```

### 16.2 CSS File Organisation

Entry point (`main.css`) imports all partials:

```css
@import "base/reset.css";
@import "base/variables.css";
@import "base/typography.css";

@import "layouts/grid.css";
@import "layouts/header.css";
@import "layouts/footer.css";

@import "components/button.css";
@import "components/card.css";
@import "components/form.css";

@import "utilities/spacing.css";
@import "utilities/visibility.css";
```

### 16.3 Framework Projects

Follow the framework's conventions (React, Vue, Svelte, Angular) for
component-scoped styles. Use CSS Modules, scoped styles, or CSS-in-JS
as appropriate for the framework.

---

## 17. Tooling

| Purpose | Tool | Notes |
|---|---|---|
| Linter (HTML) | **HTMLHint** / **html-validate** | Semantic and accessibility checks |
| Linter (CSS) | **Stylelint** | Configurable, plugin ecosystem |
| Formatter | **Prettier** | HTML and CSS formatting |
| Accessibility | **axe-core** / **Lighthouse** / **pa11y** | Automated a11y testing |
| Post-processor | **PostCSS** / **Lightning CSS** | Autoprefixer, nesting, minification |
| Build | **Vite** / **webpack** / **esbuild** | Bundling and optimisation |
| Unused CSS | **PurgeCSS** | Remove unused styles |
| Validator | **W3C Validator** | Standards compliance |
| Browser testing | **BrowserStack** / **Playwright** | Cross-browser verification |

---

## 18. Build Tools

### 18.1 npm scripts (Task Automation)

Define build tasks in package.json:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "stylelint src/**/*.css",
    "format": "prettier --write ."
  }
}
```

### 18.2 Vite (Recommended)

Modern frontend build tool:

```bash
npm create vite@latest my-project -- --template vanilla
npm run dev  # Dev server with HMR
npm run build  # Production build
npm run preview  # Preview production build
```

vite.config.js:
```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  server: { port: 3000 },
  build: { 
    outDir: 'dist',
    minify: 'terser'
  }
})
```

### 18.3 PostCSS (CSS Processing)

Extend CSS with plugins:

```javascript
// postcss.config.js
export default {
  plugins: {
    'postcss-import': {},
    'postcss-nesting': {},
    'autoprefixer': {},
    'cssnano': {}
  }
}
```

### 18.4 Webpack (Complex Projects)

Advanced bundler for complex builds:

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: { path: __dirname + '/dist', filename: 'bundle.js' },
  module: {
    rules: [
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
      { test: /\.(png|jpg)$/, type: 'asset' }
    ]
  }
}
```

### 18.5 Asset Optimization

Modern bundlers handle:
- CSS minification and tree-shaking
- Image optimization (compression, formats)
- Critical CSS extraction
- Lazy-loading chunks

### 18.6 Docker for Static Sites

```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 19. SBOM Creation

### 19.1 What is an SBOM?

Frontend projects have JavaScript/CSS dependencies via npm/Yarn. Document all packages, versions, and frameworks for security and supply chain transparency.

### 19.2 Frontend Dependency Management

See TypeScript/JavaScript guide for npm/Yarn dependency management and `package-lock.json` / `yarn.lock` files.

### 19.3 Third-Party Assets & CDN Dependencies

Document all third-party scripts/styles loaded via CDN:

```html
<!-- List in comments or separate DEPENDENCIES.md -->
<!-- Bootstrap 5.3.0 (CSS) from CDN -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
    integrity="sha384-...">

<!-- Font Awesome 6.4.0 (Icons) -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"
    integrity="sha384-...">
```

### 19.4 SBOM for Frontend

**Using `@cyclonedx/npm`** (covers JavaScript dependencies):

```bash
npm install -g @cyclonedx/npm
cyclonedx-npm --output-file sbom.json
```

**For third-party assets**, create manual inventory:

```json
{
  "components": [
    {
      "name": "Bootstrap",
      "version": "5.3.0",
      "purl": "npm/bootstrap@5.3.0",
      "license": "MIT"
    },
    {
      "name": "Font Awesome",
      "version": "6.4.0",
      "purl": "npm/@fortawesome/fontawesome-free@6.4.0",
      "license": "CC-BY-4.0 or OFL-1.1 or MIT"
    }
  ]
}
```

### 19.5 Vulnerability Scanning

Run `npm audit` for JavaScript dependencies (see TypeScript guide).

For third-party CDN libraries, verify:

- CDN is official/trusted source
- Use Subresource Integrity (`integrity` attribute)
- Check library security advisories

### 19.6 Integration into CI/CD

- Generate SBOM for JavaScript dependencies with `@cyclonedx/npm`
- Document third-party asset versions and licenses
- Run `npm audit` on every PR
- Verify Subresource Integrity hashes on all external resources
- Store SBOM with release
- Monitor for CVEs in third-party libraries (especially UI frameworks)

---

## 20. References

### Official Documentation

| Resource | URL |
|---|---|
| WHATWG HTML Living Standard | https://html.spec.whatwg.org/ |
| W3C CSS Specifications | https://www.w3.org/Style/CSS/ |
| MDN Web Docs (HTML) | https://developer.mozilla.org/en-US/docs/Web/HTML |
| MDN Web Docs (CSS) | https://developer.mozilla.org/en-US/docs/Web/CSS |
| WCAG 2.1 | https://www.w3.org/TR/WCAG21/ |
| Google HTML/CSS Style Guide | https://google.github.io/styleguide/htmlcssguide.html |

### Books

| Book | Authors | Key Takeaways |
|---|---|---|
| *CSS: The Definitive Guide* | Eric Meyer & Estelle Weyl (2017) | Comprehensive CSS reference — selectors, layout, specificity. |
| *Every Layout* | Andy Bell & Heydon Pickering (2024) | Intrinsic, composable CSS layout patterns. |
| *Inclusive Design Patterns* | Heydon Pickering (2016) | Accessible component patterns — forms, navigation, cards. |
| *Refactoring UI* | Adam Wathan & Steve Schoger (2018) | Visual design principles for developers. |
| *Clean Code* | Robert C. Martin (2008) | Small modules, meaningful names, SRP — applies to CSS architecture. |
