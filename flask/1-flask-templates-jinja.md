# Flask Reference — Templates & Jinja2

Jinja2 docs: https://jinja.palletsprojects.com/

## Rendering a Template

```python
from flask import render_template

@app.route("/")
def home():
    return render_template("index.html", name="Adam", items=["a", "b", "c"])
```
Templates live in a `templates/` folder by Flask convention. Variables
passed as keyword arguments to `render_template()` become available inside
the template.

## Jinja2 Syntax Basics

`templates/index.html`:
```html
<!-- Variable output -->
<p>Hello, {{ name }}!</p>

<!-- Conditionals -->
{% if name %}
  <p>Welcome back, {{ name }}</p>
{% else %}
  <p>Welcome, guest</p>
{% endif %}

<!-- Loops -->
<ul>
{% for item in items %}
  <li>{{ item }}</li>
{% endfor %}
</ul>

<!-- Comments (not sent to the browser) -->
{# This is a Jinja comment #}
```
`{{ }}` outputs a value, `{% %}` runs logic/control flow (no output itself).

## Filters (transform a value inline)

```html
{{ name|upper }}                 <!-- uppercase -->
{{ name|lower }}                   <!-- lowercase -->
{{ items|length }}                    <!-- count -->
{{ price|round(2) }}                    <!-- round a number -->
{{ description|truncate(50) }}            <!-- shorten text -->
{{ user_input|escape }}                     <!-- HTML-escape (Jinja does this automatically by default) -->
{{ value|default("N/A") }}                    <!-- fallback if value is undefined -->
```
Full filter list: https://jinja.palletsprojects.com/en/stable/templates/#builtin-filters

## Template Inheritance (avoid repeating layout HTML)

`templates/base.html`:
```html
<!DOCTYPE html>
<html>
<head><title>{% block title %}My Site{% endblock %}</title></head>
<body>
  <nav>...</nav>
  {% block content %}{% endblock %}
</body>
</html>
```

`templates/index.html`:
```html
{% extends "base.html" %}

{% block title %}Home{% endblock %}

{% block content %}
  <h1>Welcome, {{ name }}</h1>
{% endblock %}
```
`{% block %}` defines a named section a child template can override. This is
the standard way to share layout (nav, footer, `<head>`) across pages without
duplicating HTML — prefer it over copy-pasting the same boilerplate into
every template.

## Including Partial Templates

`templates/_navbar.html`:
```html
<nav>...</nav>
```
`templates/index.html`:
```html
{% include "_navbar.html" %}
```
Useful for small reusable snippets that don't need the full
extends/block inheritance structure (e.g. a navbar, a footer).

## Linking Static Files & Other Routes

```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
<a href="{{ url_for('home') }}">Home</a>
<a href="{{ url_for('show_user', username='adam') }}">Profile</a>
```
Always use `url_for()` in templates rather than hardcoding paths — same
reasoning as in Python code (see `flask-basics.md`).

## Rendering HTML Forms

```html
<form method="POST" action="{{ url_for('submit') }}">
  <input type="text" name="username">
  <button type="submit">Submit</button>
</form>
```
The `name` attribute on each input is the key you'll read on the backend via
`request.form.get("username")`.

## Auto-Escaping (Security)

Jinja2 auto-escapes variables by default, converting `<`, `>`, `&`, etc. into
safe HTML entities — this prevents XSS (cross-site scripting) attacks from
user-submitted content rendered back into a page. Only disable it
deliberately and only for trusted content:
```html
{{ trusted_html_content|safe }}      <!-- marks content as safe, skips escaping -->
```
Use `|safe` sparingly and never on raw user input.