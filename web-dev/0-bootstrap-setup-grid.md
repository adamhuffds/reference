# Bootstrap Reference — Setup & Grid System

Official docs: https://getbootstrap.com/docs/5.3/

Bootstrap is a CSS (+ optional JS) framework providing a responsive grid,
pre-styled components, and utility classes — it works with any backend
(Flask, plain HTML, Django, Node, etc.). Flask-specific integration notes
live in `flask-bootstrap-basics.md`; this file covers the framework itself.

## Installation Options

### 1. CDN (fastest to start, no build step)
```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
```
Good for prototyping and small projects. Downsides: no customization
without overriding CSS after the fact, and an external dependency (see
`flask-bootstrap-js-components.md` for the SRI-hash security note).

### 2. npm (needed for customization / Sass)
```bash
npm install bootstrap @popperjs/core
```
Required if you want to customize Bootstrap's Sass variables before
compiling (see `bootstrap-customization-theming.md`) rather than overriding
compiled CSS after the fact.

### 3. Download / self-host compiled files
Download the compiled CSS/JS from https://getbootstrap.com/docs/5.3/getting-started/download/
and serve them from your own `static/` folder — see `flask-static-files.md`
for the self-host vs. CDN trade-offs.

## Required HTML Boilerplate

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- CSS here -->
</head>
<body>
  <!-- content -->
  <!-- JS bundle here, before </body> -->
</body>
</html>
```
The viewport meta tag is not optional — without it, Bootstrap's responsive
breakpoints won't behave correctly on actual mobile devices (they'll render
as a zoomed-out desktop layout instead).

## The Grid System

Bootstrap's grid is based on **flexbox**, organized into a 12-column layout.

```html
<div class="container">
  <div class="row">
    <div class="col">Column 1</div>
    <div class="col">Column 2</div>
    <div class="col">Column 3</div>
  </div>
</div>
```
- `container` — fixed max-width, centered, with padding; use `container-fluid`
  for a full-width container instead
- `row` — a horizontal group of columns; always wrap columns in a `row`
- `col` — columns without an explicit number auto-divide the available
  width equally

### Explicit Column Widths (out of 12)
```html
<div class="row">
  <div class="col-8">Two-thirds width</div>
  <div class="col-4">One-third width</div>
</div>
```

### Responsive Columns (different widths per breakpoint)
```html
<div class="row">
  <div class="col-12 col-md-6 col-lg-4">Responsive column</div>
</div>
```
Reads as: full width on mobile (`col-12`), half width at `md` and up
(`col-md-6`), one-third width at `lg` and up (`col-lg-4`). Breakpoints are
mobile-first — a class applies at that breakpoint *and above*, unless
overridden by a larger breakpoint's class.

### Breakpoints
| Class infix | Min width |
|---|---|
| (none) | 0px |
| `sm` | 576px |
| `md` | 768px |
| `lg` | 992px |
| `xl` | 1200px |
| `xxl` | 1400px |

## Offsets & Ordering

```html
<div class="row">
  <div class="col-4 offset-4">Centered 4-wide column</div>
</div>

<div class="row">
  <div class="col order-2">Shows second</div>
  <div class="col order-1">Shows first</div>
</div>
```
`offset-N` pushes a column right by N columns (out of 12). `order-N`
reorders columns visually without changing their position in the HTML —
useful for reordering content between mobile and desktop layouts.

## Gutters (Spacing Between Columns)

```html
<div class="row g-3">        <!-- gutter spacing on all sides -->
<div class="row gx-4 gy-2">    <!-- separate horizontal (x) and vertical (y) gutter control -->
```
Docs: https://getbootstrap.com/docs/5.3/layout/gutters/

## Flexbox Utilities (Beyond the Grid)

```html
<div class="d-flex justify-content-between align-items-center">
  <span>Left</span>
  <span>Right</span>
</div>
```
- `d-flex` — enables flexbox on this element
- `justify-content-*` — main-axis alignment: `start`, `end`, `center`,
  `between`, `around`
- `align-items-*` — cross-axis alignment: `start`, `end`, `center`,
  `stretch`

## Spacing Utilities

Pattern: `{property}{sides}-{size}`
```
m = margin, p = padding
t/b/s/e = top/bottom/start(left)/end(right), (blank) = all sides, x/y = horizontal/vertical
0-5 = size scale, auto = auto margin
```
```html
<div class="mt-3">        <!-- margin-top -->
<div class="px-4">          <!-- padding-left + padding-right -->
<div class="mb-0">            <!-- remove default bottom margin -->
<div class="mx-auto">           <!-- horizontal auto margin — common centering trick for fixed-width blocks -->
```
Docs: https://getbootstrap.com/docs/5.3/utilities/spacing/

## Display & Visibility Utilities

```html
<div class="d-none d-md-block">    <!-- hidden below md breakpoint, visible md and up -->
<div class="d-md-none">              <!-- visible below md, hidden md and up (e.g. mobile-only nav) -->
```
Common pattern for showing different content/layout on mobile vs desktop
without duplicating markup in JS.

## Text & Color Utilities

```html
<p class="text-center">Centered text</p>
<p class="text-muted">De-emphasized text</p>
<p class="fw-bold">Bold text</p>
<p class="text-truncate" style="max-width: 200px;">Truncates with an ellipsis if too long</p>

<div class="text-bg-primary p-3">Themed background + auto-contrasting text</div>
```
`text-bg-*` (as opposed to just `bg-*`) automatically sets a readable text
color for that background — the current recommended pattern over manually
pairing `bg-primary` with `text-white` and hoping the contrast holds up
across theme changes.