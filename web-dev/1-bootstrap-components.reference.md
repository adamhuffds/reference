# Bootstrap Reference — Components

Docs: https://getbootstrap.com/docs/5.3/components/

Framework-level component reference, independent of any backend. See
`flask-bootstrap-js-components.md` for modals, toasts, tooltips, dropdowns,
and collapse (those are covered there since they involve Flask-side
patterns like flash messages and confirmation flows).

## Buttons

```html
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-danger">Danger</button>
<button class="btn btn-outline-primary">Outline variant</button>
<button class="btn btn-primary btn-sm">Small</button>
<button class="btn btn-primary btn-lg">Large</button>
<button class="btn btn-primary" disabled>Disabled</button>
```
Color variants (`primary`, `secondary`, `success`, `danger`, `warning`,
`info`, `light`, `dark`) are consistent across buttons, alerts, badges, and
text/background utilities — learning the palette once applies everywhere.

## Navbar

```html
<nav class="navbar navbar-expand-lg bg-body-tertiary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Brand</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navContent">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navContent">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link active" href="#">Home</a></li>
        <li class="nav-item"><a class="nav-link" href="#">About</a></li>
      </ul>
    </div>
  </div>
</nav>
```
`navbar-expand-{breakpoint}` controls at which screen width the nav switches
from a hamburger menu to full inline links. See `bootstrap-setup-grid.md`
for the breakpoint table.

## Alerts

```html
<div class="alert alert-success">Success message</div>
<div class="alert alert-danger">Error message</div>
<div class="alert alert-warning alert-dismissible fade show">
  Dismissible warning
  <button class="btn-close" data-bs-dismiss="alert"></button>
</div>
```
For Flask projects, pair these with `flash()` — see
`flask-bootstrap-basics.md`.

## Cards

```html
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="...">
  <div class="card-body">
    <h5 class="card-title">Title</h5>
    <p class="card-text">Body text.</p>
    <a href="#" class="btn btn-primary">Action</a>
  </div>
</div>
```
Cards are the standard building block for dashboards, product grids, and
list-style layouts — commonly placed inside grid columns (`col-md-4`) to
build a responsive card grid.

## List Groups

```html
<ul class="list-group">
  <li class="list-group-item">Item one</li>
  <li class="list-group-item active">Active item</li>
  <li class="list-group-item disabled">Disabled item</li>
</ul>

<!-- Clickable list group (e.g. a sidebar nav) -->
<div class="list-group">
  <a href="#" class="list-group-item list-group-item-action">Link item</a>
</div>
```

## Badges

```html
<span class="badge text-bg-primary">New</span>
<button class="btn btn-primary">
  Inbox <span class="badge text-bg-light">4</span>
</button>
```

## Breadcrumbs

```html
<nav aria-label="breadcrumb">
  <ol class="breadcrumb">
    <li class="breadcrumb-item"><a href="#">Home</a></li>
    <li class="breadcrumb-item"><a href="#">Library</a></li>
    <li class="breadcrumb-item active" aria-current="page">Data</li>
  </ol>
</nav>
```
`aria-current="page"` on the final item is an accessibility attribute — it
tells screen readers this is the current page, not just styling.

## Pagination

```html
<nav>
  <ul class="pagination">
    <li class="page-item disabled"><a class="page-link" href="#">Previous</a></li>
    <li class="page-item active"><a class="page-link" href="#">1</a></li>
    <li class="page-item"><a class="page-link" href="#">2</a></li>
    <li class="page-item"><a class="page-link" href="#">Next</a></li>
  </ul>
</nav>
```

## Progress Bars

```html
<div class="progress" role="progressbar" aria-valuenow="65" aria-valuemin="0" aria-valuemax="100">
  <div class="progress-bar" style="width: 65%">65%</div>
</div>

<div class="progress">
  <div class="progress-bar progress-bar-striped progress-bar-animated" style="width: 100%"></div>
</div>
```
The `progress-bar-animated` variant is useful for indeterminate/in-progress
states (e.g. a long-running upload or ETL job), where you don't have an
exact percentage to report.

## Spinners

```html
<div class="spinner-border text-primary" role="status">
  <span class="visually-hidden">Loading...</span>
</div>

<div class="spinner-grow text-secondary" role="status">
  <span class="visually-hidden">Loading...</span>
</div>
```
`visually-hidden` text is read by screen readers but not visible on
screen — always include it on a spinner so assistive tech announces that
content is loading.

## Tables

```html
<table class="table table-striped table-hover table-bordered">
  <thead>
    <tr><th>Name</th><th>Score</th></tr>
  </thead>
  <tbody>
    <tr><td>Adam</td><td>95</td></tr>
  </tbody>
</table>
```
See `flask-pandas.md` for generating this directly from a DataFrame with
`df.to_html(classes=...)`.

## Accessibility Notes (Applies Across All Components)

- Icon-only buttons need an accessible label:
  ```html
  <button class="btn-close" aria-label="Close"></button>
  ```
- Color alone shouldn't convey meaning (e.g. red = error) — pair with text
  or an icon for colorblind users.
- `visually-hidden` (not `display: none`) for text that should be
  screen-reader-only but visually absent — `display: none` hides it from
  assistive tech too, defeating the purpose.