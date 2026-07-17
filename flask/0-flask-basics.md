# Flask Reference — Basics

Docs: https://flask.palletsprojects.com/

## Install & Minimal App

```bash
pip install flask
```

```python
# app.py
from flask import Flask

app = Flask(__name__)
# __name__ tells Flask where to look for templates/static files relative to
# this module — standard convention, almost always just pass __name__

@app.route("/")
def home():
    return "Hello, Flask!"

if __name__ == "__main__":
    app.run(debug=True)
```
- `debug=True` enables the interactive debugger and auto-reload on code
  changes — essential for development, **must be off in production** (see
  `flask-deployment.md`)

## Running the App

```bash
python3 app.py                              # runs via app.run() in the script itself

# OR the Flask CLI (modern preferred approach for dev):
flask --app app run --debug
```
`flask --app app run` — the current recommended way to launch during
development, since it doesn't require an `if __name__ == "__main__":` block
and works cleanly with the app factory pattern (see
`flask-blueprints-app-factory.md`).

## Routing

```python
@app.route("/about")
def about():
    return "About page"

@app.route("/user/<username>")           # variable route segment
def show_user(username):
    return f"User: {username}"

@app.route("/post/<int:post_id>")         # type converter — only matches integers
def show_post(post_id):
    return f"Post #{post_id}"

@app.route("/submit", methods=["GET", "POST"])   # restrict/allow HTTP methods
def submit():
    if request.method == "POST":
        return "Handled POST"
    return "Show form"
```
Route converters: `<string:x>` (default), `<int:x>`, `<float:x>`, `<path:x>`
(allows slashes), `<uuid:x>`. Docs: https://flask.palletsprojects.com/en/latest/quickstart/#variable-rules

## The Request Object

```python
from flask import request

@app.route("/search")
def search():
    query = request.args.get("q")           # query string: /search?q=protein
    return f"Searching for: {query}"

@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username")   # form data (from an HTML <form>)
    return f"Logging in {username}"

@app.route("/api/data", methods=["POST"])
def receive_data():
    data = request.get_json()                  # parsed JSON body
    return data
```
Docs: https://flask.palletsprojects.com/en/latest/api/#flask.Request

## Responses

```python
from flask import jsonify, redirect, url_for, abort

@app.route("/api/status")
def status():
    return jsonify({"status": "ok"})        # sets Content-Type: application/json automatically

@app.route("/old-page")
def old_page():
    return redirect(url_for("home"))          # redirect to another route by function name, not hardcoded URL

@app.route("/protected")
def protected():
    abort(403)                                  # short-circuit with an HTTP error status

@app.route("/custom-status")
def custom_status():
    return "Not found", 404                       # return a (body, status_code) tuple directly
```
`url_for("home")` builds the URL from the view function's name, not a
hardcoded string — the current best-practice approach, since it stays
correct even if you later change the route path in `@app.route`.

## Static Files & Config Basics

```python
app.config["SECRET_KEY"] = "change-this-in-production"   # required for sessions, CSRF protection, etc.
app.config["DEBUG"] = True
```
Static files (CSS, JS, images) go in a `static/` folder by convention and
are served automatically at `/static/<filename>`. Templates go in a
`templates/` folder — see `flask-templates-jinja.md`.

## Minimal Project Structure

```
my_app/
├── app.py
├── static/
│   └── style.css
└── templates/
    └── index.html
```