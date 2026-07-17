# Flask Reference — Blueprints & App Factory Pattern

Docs: https://flask.palletsprojects.com/en/latest/patterns/appfactories/ | https://flask.palletsprojects.com/en/latest/blueprints/

## Why This Pattern

A single `app.py` with everything in it (like `flask-basics.md`) works fine
for small scripts, but doesn't scale to a real project — routes, models, and
config all end up tangled together, and it's hard to run tests against a
fresh app instance. The **app factory** pattern fixes this by wrapping app
creation in a function, and **blueprints** let you split routes across
multiple files by feature area.

## Recommended Project Structure

```
my_app/
├── app/
│   ├── __init__.py          <- app factory lives here
│   ├── config.py
│   ├── models.py
│   ├── extensions.py         <- db, migrate, login_manager instances
│   ├── auth/
│   │   ├── __init__.py
│   │   └── routes.py
│   └── main/
│       ├── __init__.py
│       └── routes.py
├── migrations/
├── tests/
├── .env
└── run.py
```

## Config Classes

`app/config.py`:
```python
import os

class Config:
    SECRET_KEY = os.getenv("SECRET_KEY", "dev-key-change-me")
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL")
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    DEBUG = False

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = "sqlite:///:memory:"
```
Separate config classes per environment (dev/prod/test) is the standard
pattern — see `python-os-env-vars.md` for loading secrets via `.env`.

## Extension Instances (avoid circular imports)

`app/extensions.py`:
```python
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager

db = SQLAlchemy()
migrate = Migrate()
login_manager = LoginManager()
```
Creating extension instances *without* an app attached (no `db = SQLAlchemy(app)`
yet), then binding them to an app later inside the factory with
`.init_app(app)`, avoids circular imports between `models.py` and the routes
that use `db`.

## The App Factory

`app/__init__.py`:
```python
from flask import Flask
from app.config import DevelopmentConfig
from app.extensions import db, migrate, login_manager

def create_app(config_class=DevelopmentConfig):
    app = Flask(__name__)
    app.config.from_object(config_class)

    # Bind extensions to this specific app instance
    db.init_app(app)
    migrate.init_app(app, db)
    login_manager.init_app(app)

    # Register blueprints
    from app.auth.routes import auth_bp
    from app.main.routes import main_bp
    app.register_blueprint(auth_bp)
    app.register_blueprint(main_bp)

    return app
```
`create_app()` returning a fresh, fully-configured app is what makes testing
easy — tests can call `create_app(TestingConfig)` to get an isolated
instance instead of importing one shared global `app`.

## Blueprints

`app/main/routes.py`:
```python
from flask import Blueprint, render_template

main_bp = Blueprint("main", __name__)

@main_bp.route("/")
def home():
    return render_template("index.html")
```

`app/auth/routes.py`:
```python
from flask import Blueprint, request

auth_bp = Blueprint("auth", __name__, url_prefix="/auth")

@auth_bp.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        # handle login logic
        pass
    return "Login page"
```
- `url_prefix="/auth"` — every route in this blueprint is automatically
  prefixed, so `/login` here actually becomes `/auth/login`
- Inside templates/redirects, blueprint routes are referenced as
  `blueprint_name.function_name`, e.g. `url_for("auth.login")`

## Entry Point

`run.py`:
```python
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(debug=True)
```

Run with:
```bash
flask --app run run --debug
```
Or set `FLASK_APP=run.py` in your `.env`/shell so you can just run `flask run`.

## Models File

`app/models.py`:
```python
from app.extensions import db
from sqlalchemy.orm import Mapped, mapped_column

class User(db.Model):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(unique=True)
```
See `flask-database-sqlalchemy.md` for full model/query patterns.