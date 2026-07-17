# Flask Reference — REST APIs, Sessions & Auth

Docs: https://flask.palletsprojects.com/en/latest/quickstart/#sessions

## Building a JSON API

```python
from flask import Blueprint, jsonify, request

api_bp = Blueprint("api", __name__, url_prefix="/api")

@api_bp.get("/notes")                       # shorthand for @api_bp.route(..., methods=["GET"])
def get_notes():
    notes = [{"id": 1, "body": "first note"}]
    return jsonify(notes)

@api_bp.post("/notes")
def create_note():
    data = request.get_json()
    if not data or "body" not in data:
        return jsonify({"error": "body is required"}), 400
    # ... save to DB ...
    return jsonify({"id": 1, "body": data["body"]}), 201

@api_bp.get("/notes/<int:note_id>")
def get_note(note_id):
    note = None  # ... look up in DB ...
    if note is None:
        return jsonify({"error": "not found"}), 404
    return jsonify(note)
```
`.get()`/`.post()` shortcuts on `Blueprint`/`app` (Flask 2.0+) are the
current preferred style over `methods=["GET"]` for single-method routes —
more concise and equally explicit.

## Consistent Error Handling

```python
from flask import jsonify
from werkzeug.exceptions import HTTPException

@app.errorhandler(404)
def not_found(e):
    return jsonify({"error": "Resource not found"}), 404

@app.errorhandler(500)
def server_error(e):
    return jsonify({"error": "Internal server error"}), 500

@app.errorhandler(HTTPException)
def handle_http_exception(e):
    # catches any HTTPException not covered above (403, 405, etc.)
    return jsonify({"error": e.description}), e.code
```
Centralizing error handlers like this ensures your API always returns
consistent JSON error shapes, instead of Flask's default HTML error pages
leaking into what's supposed to be a JSON API.

## Request Validation

Manual validation works for small APIs (as above), but for anything larger,
use a schema/validation library rather than hand-rolling checks everywhere:

```bash
pip install marshmallow
```
```python
from marshmallow import Schema, fields, ValidationError

class NoteSchema(Schema):
    body = fields.Str(required=True)

@api_bp.post("/notes")
def create_note():
    try:
        data = NoteSchema().load(request.get_json())
    except ValidationError as err:
        return jsonify(err.messages), 400
    # data is now validated
    return jsonify(data), 201
```
Docs: https://marshmallow.readthedocs.io/
For projects already using Pydantic elsewhere (e.g. via `pydantic-settings`,
see `python-os-env-vars.md`), `pydantic` models work as request validators
too and are a common modern alternative to Marshmallow.

## Sessions (Cookie-Based)

```python
from flask import session

app.config["SECRET_KEY"] = "change-this-in-production"   # required — sessions are signed with this

@app.route("/login", methods=["POST"])
def login():
    session["user_id"] = 1        # stored in a signed cookie on the client
    return "Logged in"

@app.route("/profile")
def profile():
    user_id = session.get("user_id")
    if user_id is None:
        return "Not logged in", 401
    return f"Profile for user {user_id}"

@app.route("/logout")
def logout():
    session.pop("user_id", None)
    return "Logged out"
```
Flask sessions are stored client-side in a cryptographically signed cookie —
the data is readable (not encrypted) but not forgeable without `SECRET_KEY`.
Don't store sensitive data (passwords, tokens) directly in the session.

## Authentication with Flask-Login

```bash
pip install flask-login
```
```python
from flask_login import LoginManager, UserMixin, login_user, login_required, current_user, logout_user

login_manager = LoginManager()

class User(db.Model, UserMixin):     # UserMixin provides is_authenticated, get_id(), etc.
    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str]

@login_manager.user_loader
def load_user(user_id):
    return db.session.get(User, int(user_id))

@app.route("/login", methods=["POST"])
def login():
    user = db.session.execute(
        select(User).where(User.username == request.form["username"])
    ).scalar_one_or_none()
    if user:
        login_user(user)
        return redirect(url_for("main.home"))
    return "Invalid credentials", 401

@app.route("/dashboard")
@login_required                       # blocks access unless logged in
def dashboard():
    return f"Welcome, {current_user.username}"

@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(url_for("main.home"))
```
Docs: https://flask-login.readthedocs.io/
Flask-Login handles session management for you (via cookies under the hood)
— use it over hand-rolling `session["user_id"]` checks once you need real
login/logout flows, remember-me, or route protection decorators.

## Password Hashing

Never store plain-text passwords.
```python
from werkzeug.security import generate_password_hash, check_password_hash

hashed = generate_password_hash("plaintext_password")     # store this in the DB
check_password_hash(hashed, "plaintext_password")           # True/False on login attempt
```
`werkzeug.security` ships with Flask, so no extra install needed for basic
password hashing.

## Token-Based Auth (APIs Without Sessions)

For APIs consumed by non-browser clients (mobile apps, other services),
JWT (JSON Web Tokens) is the common alternative to cookie-based sessions —
each request carries a signed token in an `Authorization: Bearer <token>`
header instead of relying on cookies.

```bash
pip install flask-jwt-extended
```
```python
from flask_jwt_extended import JWTManager, create_access_token, jwt_required, get_jwt_identity

app.config["JWT_SECRET_KEY"] = "change-this-in-production"
jwt = JWTManager(app)

@app.post("/login")
def login():
    # ... verify credentials ...
    token = create_access_token(identity=user.id)
    return jsonify(access_token=token)

@app.get("/protected")
@jwt_required()
def protected():
    user_id = get_jwt_identity()
    return jsonify(user_id=user_id)
```
Docs: https://flask-jwt-extended.readthedocs.io/