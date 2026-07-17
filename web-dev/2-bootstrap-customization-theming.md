# Bootstrap Reference — Customization & Theming

Docs: https://getbootstrap.com/docs/5.3/customize/overview/

## Two Ways to Customize Bootstrap

1. **Override compiled CSS** — write your own CSS rules after Bootstrap's
   stylesheet that override specific properties. Quick, but fights
   Bootstrap's specificity and doesn't let you change core values like the
   color palette or breakpoints globally.
2. **Recompile from Sass source with custom variables** — the current
   best-practice approach for any real customization, since it changes
   values at the source rather than overriding output after the fact.

## Sass-Based Customization (Recommended)

Requires the npm install method (see `bootstrap-setup-grid.md`), plus a Sass
compiler:
```bash
npm install bootstrap sass --save-dev
```

`custom.scss`:
```scss
// Override Bootstrap's default variables BEFORE importing Bootstrap
$primary: #6f42c1;
$border-radius: 0.5rem;
$font-family-sans-serif: "Inter", sans-serif;

// Then import Bootstrap itself
@import "bootstrap/scss/bootstrap";
```
Order matters — Sass variables must be set *before* the `@import`, since
Bootstrap's own Sass files check whether a variable is already defined
(`!default` flag) before applying its own value.

Compile:
```bash
npx sass custom.scss custom.css
```
Then link `custom.css` in your HTML instead of the CDN Bootstrap CSS.

### Common Variables Worth Overriding
```scss
$primary: #0d6efd;         // primary color used across buttons, links, badges, etc.
$secondary: #6c757d;
$success: #198754;
$danger: #dc3545;
$warning: #ffc107;
$info: #0dcaf0;

$font-family-base: "Inter", sans-serif;
$font-size-base: 1rem;

$border-radius: 0.375rem;    // corner rounding used across buttons, cards, inputs, etc.
$spacer: 1rem;                  // base unit all spacing utilities (m-1, p-2, etc.) scale from

$grid-gutter-width: 1.5rem;       // default gutter width in the grid system
```
Full variable list: https://github.com/twbs/bootstrap/blob/main/scss/_variables.scss

## Native Dark Mode (Bootstrap 5.3+)

As of Bootstrap 5.3, color modes are built in — no custom CSS required for
a basic dark theme:
```html
<html data-bs-theme="dark">
```
```html
<div data-bs-theme="light">
  <!-- force light mode within an otherwise-dark page -->
</div>
```
This is more current than the pre-5.3 pattern of manually applying
`bg-dark text-light` classes (see `flask-bootstrap-basics.md` for why that
older pattern leaves component internals mismatched). Toggle it at runtime
with a small JS snippet that sets the attribute based on user preference or
a `prefers-color-scheme` media query check.

### Customizing Dark Mode Colors
```scss
// Sass maps let you override colors specifically within dark mode
@import "bootstrap/scss/bootstrap";

[data-bs-theme="dark"] {
  --bs-body-bg: #1a1a1a;
  --bs-primary: #8b5cf6;
}
```
Bootstrap 5.3's color system is built on CSS custom properties
(`--bs-*`), which is what makes runtime theme-switching possible without a
separate compiled stylesheet per theme.
Docs: https://getbootstrap.com/docs/5.3/customize/color-modes/

## Only Importing What You Need (Reduce File Size)

Instead of `@import "bootstrap/scss/bootstrap"` (everything), import
individual pieces:
```scss
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/mixins";

@import "bootstrap/scss/root";
@import "bootstrap/scss/reboot";
@import "bootstrap/scss/grid";
@import "bootstrap/scss/buttons";
@import "bootstrap/scss/forms";
// skip components you don't use, e.g. carousel, toasts, etc.
```
Worth doing for a production app where every KB of shipped CSS matters —
not necessary for prototyping or internal tools.
Full partial list: https://github.com/twbs/bootstrap/blob/main/scss/bootstrap.scss

## Utility API (Generating Custom Utility Classes)

Bootstrap 5's Sass-based Utility API lets you add your own spacing/color/etc.
utility classes following the same naming pattern as built-in ones:
```scss
$utilities: map-merge(
  $utilities,
  (
    "cursor": (
      property: cursor,
      values: pointer grab,
    ),
  )
);
```
This generates `.cursor-pointer` and `.cursor-grab` classes automatically,
consistent with how `.d-flex`/`.mt-3`/etc. are generated internally.
Docs: https://getbootstrap.com/docs/5.3/utilities/api/

## When Sass Customization Isn't Worth It

For a small internal tool, a lab dashboard, or a prototype, the CDN
+ `data-bs-theme` + a handful of inline style overrides is usually the
pragmatic choice — Sass compilation adds a build step (npm, a bundler like
Vite/Webpack) that's overhead not worth paying unless the project's visual
identity genuinely needs to diverge from Bootstrap's defaults, or you're
optimizing shipped file size for a public-facing production app.