# Flask + Bootstrap — JS Components

Docs: https://getbootstrap.com/docs/5.3/components/

Since `bootstrap.bundle.js` is loaded in `base.html` (see
`flask-bootstrap-basics.md`), most components work automatically via
`data-bs-*` attributes — **no custom JavaScript required** for modals,
toasts, collapse, or dropdowns. Tooltips and popovers are the one exception
(see below).

## Modals (Confirmations)

```html
<button class="btn btn-danger" data-bs-toggle="modal" data-bs-target="#confirmModal">
  Overwrite Items
</button>

<div class="modal fade" id="confirmModal" tabindex="-1">
  <div class="modal-dialog modal-dialog-centered">
    <div class="modal-content">

      <div class="modal-header">
        <h5 class="modal-title">Confirm Overwrite</h5>
        <button class="btn-close" data-bs-dismiss="modal"></button>
      </div>

      <div class="modal-body">
        Type "OVERWRITE" to confirm:
        <input id="overwriteInput" class="form-control mt-2">
      </div>

      <div class="modal-footer">
        <button class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
        <button class="btn btn-danger" id="confirmOverwriteBtn">Confirm</button>
      </div>

    </div>
  </div>
</div>
```
```html
{% block scripts %}
<script>
document.getElementById("confirmOverwriteBtn").addEventListener("click", () => {
  const val = document.getElementById("overwriteInput").value.trim();
  if (val === "OVERWRITE") {
    document.getElementById("realOverwriteForm").submit();
  } else {
    alert("Type OVERWRITE to continue.");
  }
});
</script>
{% endblock %}
```
Modals are the standard "safe UX" pattern for destructive actions — forcing
a typed confirmation (rather than just an OK/Cancel click) adds meaningful
friction for genuinely irreversible operations like a bulk delete.

## Collapse / Accordion (Hide Advanced Options)

```html
<a class="btn btn-outline-secondary" data-bs-toggle="collapse" href="#advanced">
  Show Advanced Options
</a>

<div class="collapse" id="advanced">
  <div class="card card-body">
    Extra filters, developer controls, etc.
  </div>
</div>
```
Good for keeping a form or dashboard visually simple by default while still
exposing power-user options for those who need them.

## Toasts (Popup Notifications)

```html
<div class="position-fixed bottom-0 end-0 p-3" style="z-index: 11;">
  <div id="statusToast" class="toast text-bg-success">
    <div class="toast-header">
      <strong class="me-auto">Status</strong>
      <button class="btn-close" data-bs-dismiss="toast"></button>
    </div>
    <div class="toast-body">Operation successful.</div>
  </div>
</div>
```
```html
{% block scripts %}
<script>
const toastEl = document.getElementById("statusToast");
const toast = new bootstrap.Toast(toastEl);
toast.show();
</script>
{% endblock %}
```
Unlike modals/collapse, a toast needs one line of JS (`.show()`) to actually
appear — `data-bs-*` attributes alone only wire up *dismissal*, not the
initial display, since toasts are meant to be triggered programmatically
(e.g. after an AJAX call succeeds) rather than by a user click.

## Tooltips & Popovers — Manual Initialization Required

Unlike every component above, tooltips and popovers **do not activate
automatically** just by including `bootstrap.bundle.js` — they must be
explicitly initialized in JS, for performance reasons (Bootstrap doesn't
scan the whole page for every possible tooltip on load).

```html
<button class="btn btn-secondary" data-bs-toggle="tooltip" data-bs-placement="top"
  title="This deletes the record permanently">
  Delete
</button>
```
```html
{% block scripts %}
<script>
// Required initialization — easy to forget, and the tooltip will simply
// never appear without this, with no error to indicate why
const tooltipTriggerList = document.querySelectorAll('[data-bs-toggle="tooltip"]');
[...tooltipTriggerList].forEach(el => new bootstrap.Tooltip(el));
</script>
{% endblock %}
```
This is the single most common Bootstrap gotcha for people coming from
modals/collapse/dropdowns, which "just work" — tooltips silently do nothing
without this snippet, and there's no console error to point you at why.
Docs: https://getbootstrap.com/docs/5.3/components/tooltips/

## Dropdowns

```html
<div class="dropdown">
  <button class="btn btn-secondary dropdown-toggle" data-bs-toggle="dropdown">
    Actions
  </button>
  <ul class="dropdown-menu">
    <li><a class="dropdown-item" href="#">Edit</a></li>
    <li><a class="dropdown-item" href="#">Delete</a></li>
  </ul>
</div>
```
Works automatically, same as modals/collapse — no JS init needed.

## Cards (Content Containers)

```html
<div class="card" style="width: 18rem;">
  <div class="card-body">
    <h5 class="card-title">Card Title</h5>
    <h6 class="card-subtitle mb-2 text-body-secondary">Subtitle</h6>
    <p class="card-text">Some quick example content.</p>
    <a href="#" class="btn btn-primary">Action</a>
  </div>
</div>
```
Cards are the standard container for dashboard widgets, list items, or
grouped content — commonly used with the grid system (`row`/`col`) to build
card grids.

## Badges & Pagination

```html
<span class="badge text-bg-primary">New</span>
<span class="badge text-bg-danger">3 errors</span>
```
```html
<nav>
  <ul class="pagination">
    <li class="page-item"><a class="page-link" href="?page=1">1</a></li>
    <li class="page-item active"><a class="page-link" href="?page=2">2</a></li>
    <li class="page-item"><a class="page-link" href="?page=3">3</a></li>
  </ul>
</nav>
```
Useful for paginating a dataset alongside the `LIMIT`/`OFFSET` SQL pattern
in `sql-queries-basics.md`.

## Icons

Bootstrap's core CSS/JS does **not** include icons — that's a separate
library, Bootstrap Icons:
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
```
```html
<i class="bi bi-trash"></i>
<i class="bi bi-check-circle-fill text-success"></i>
```
Docs / full icon list: https://icons.getbootstrap.com/

## Security Note on CDN Usage

The CDN links throughout this guide (`cdn.jsdelivr.net`) are convenient for
learning/prototyping, but for anything production-facing, consider adding a
Subresource Integrity (SRI) hash — this ensures the browser refuses to
execute the file if the CDN is ever compromised and serves altered content:
```html
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
  integrity="sha384-..." crossorigin="anonymous"></script>
```
Bootstrap's own docs page includes current SRI hashes per version — copy
them directly from there rather than generating your own, since they must
match the exact file. See `flask-static-files.md` for the trade-offs of
self-hosting these files instead of using a CDN at all.

## Summary: What Covers ~90% of Flask + Bootstrap Apps

- `base.html` with Bootstrap CSS + JS, `data-bs-theme="dark"` for dark mode
- Forms: `form-control`, `form-label`, `form-select`, `btn`, `mb-3`
- Flask's `flash()` + `get_flashed_messages()` wired to Bootstrap alerts
- Tables: `table`, `table-hover`, `table-responsive`
- JS components: modals, collapse, toasts, dropdowns (auto) + tooltips
  (manual init required)
- Cards, badges, pagination for dashboard-style layouts