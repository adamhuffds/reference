# Flask Reference — Static Files & CSS

Docs: https://flask.palletsprojects.com/en/latest/quickstart/#static-files

## Default Behavior

```
my_app/
├── app.py
└── static/
    ├── style.css
    ├── script.js
    └── images/
        └── logo.png
```
Anything in `static/` is served automatically at the URL path `/static/...`
— no route needs to be written for it.

```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
<script src="{{ url_for('static', filename='script.js') }}"></script>
<img src="{{ url_for('static', filename='images/logo.png') }}">
```
Always use `url_for('static', filename=...)` rather than hardcoding
`/static/style.css` — same reasoning as route `url_for()` calls: it stays
correct if the static folder location or app structure changes.

## Customizing the Static Folder

```python
app = Flask(__name__, static_folder="assets", static_url_path="/assets")
```
- `static_folder` — change which folder on disk is served (default: `static/`)
- `static_url_path` — change the URL prefix it's served under (default: `/static`)

Useful when integrating with a frontend build tool that outputs to a
different folder name (e.g. `dist/`, `public/`).

## Serving Files Outside `static/`

```python
from flask import send_from_directory

@app.route("/downloads/<path:filename>")
def download_file(filename):
    return send_from_directory("downloads", filename, as_attachment=True)
```
- `as_attachment=True` — prompts a browser download instead of displaying
  the file inline (e.g. for a PDF report or CSV export)
- `<path:filename>` converter allows slashes in the matched segment, needed
  if files are nested in subfolders

Docs: https://flask.palletsprojects.com/en/latest/api/#flask.send_from_directory

## Cache-Busting

Browsers aggressively cache static assets by filename. Two common approaches
to force a refresh after deploying a CSS/JS change:

**1. Query string versioning (simple, manual):**
```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}?v=2">
```

**2. Flask-Assets / build-tool hashing (automatic, scales better):**
For projects with many static files, a bundler (Webpack, Vite) that appends
a content hash to filenames (`style.a1b2c3.css`) is the more current
approach than manually bumping `?v=` — worth adopting once a project's
frontend grows beyond a couple of hand-written CSS/JS files.

## Organizing Larger Projects

```
static/
├── css/
│   ├── base.css
│   └── dashboard.css
├── js/
│   ├── main.js
│   └── charts.js
└── images/
```
Subdividing by type keeps `static/` navigable as a project grows — mirrors
the same instinct behind splitting these reference docs into topic-specific
files rather than one giant one.

## CDN vs. Local Static Files

For common libraries (Bootstrap, jQuery, Chart.js), you can either:
- **Self-host** in `static/` — more reliable if the CDN goes down, works
  offline, no third-party tracking, but you own updating versions manually
- **Load from a CDN** — one `<script src="https://cdn...">` tag, faster
  first-load for returning visitors likely to already have it cached from
  another site, but adds an external dependency

For anything internal/private (e.g. a lab tool that isn't public-facing),
self-hosting is generally the safer default.