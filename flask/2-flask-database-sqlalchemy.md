# Flask Reference — Database (Flask-SQLAlchemy)

Docs: https://flask-sqlalchemy.palletsprojects.com/

## Install & Setup

```bash
pip install flask-sqlalchemy flask-migrate
```

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "postgresql://user:password@localhost:5432/mydb"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False   # disables an unused feature, saves overhead

db = SQLAlchemy(app)
```
Connection string format varies by database:
- PostgreSQL: `postgresql://user:pass@host:port/dbname`
- SQLite (good for quick local dev): `sqlite:///app.db`
- MySQL: `mysql+pymysql://user:pass@host:port/dbname`

## Defining Models (SQLAlchemy 2.0 style)

```python
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy import String, ForeignKey
from datetime import datetime

class User(db.Model):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    username: Mapped[str] = mapped_column(String(80), unique=True, nullable=False)
    email: Mapped[str] = mapped_column(String(120), unique=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

    def __repr__(self):
        return f"User({self.username})"
```
This uses the **modern SQLAlchemy 2.0 `Mapped`/`mapped_column` style**, not
the legacy `db.Column(db.String(80))` pattern still common in older Flask
tutorials — prefer `Mapped` annotations going forward, since they give you
type hints and better IDE support. Docs: https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html#orm-mapping-styles

### Relationships
```python
from sqlalchemy.orm import relationship

class Post(db.Model):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    author: Mapped["User"] = relationship(back_populates="posts")

# Add the reverse side on User:
class User(db.Model):
    ...
    posts: Mapped[list["Post"]] = relationship(back_populates="author")
```
`back_populates` keeps both sides of the relationship in sync — setting
`post.author` automatically updates `user.posts` in memory.

## Creating Tables

```python
with app.app_context():
    db.create_all()
```
`db.create_all()` is fine for quick prototyping, but for anything you'll
evolve over time, use migrations instead (see below) — `create_all()` won't
alter existing tables when you change a model later.

## Querying (SQLAlchemy 2.0 style)

```python
from sqlalchemy import select

# Get by primary key
user = db.session.get(User, 1)

# Select all
users = db.session.execute(select(User)).scalars().all()

# Filter
adults = db.session.execute(
    select(User).where(User.age >= 18)
).scalars().all()

# Filter, single result
user = db.session.execute(
    select(User).where(User.username == "adam")
).scalar_one_or_none()          # None if not found, raises if more than one match

# Order & limit
recent = db.session.execute(
    select(Post).order_by(Post.created_at.desc()).limit(10)
).scalars().all()
```
This `select()`-based querying is the current SQLAlchemy 2.0 pattern —
prefer it over the legacy `User.query.filter_by(...)` style still seen in
many older Flask tutorials, since `Model.query` is a legacy convenience
layer being phased toward `db.session.execute(select(...))`.
Docs: https://docs.sqlalchemy.org/en/20/orm/queryguide/select.html

## Inserting, Updating, Deleting

```python
# Insert
new_user = User(username="adam", email="adam@example.com")
db.session.add(new_user)
db.session.commit()

# Update
user = db.session.get(User, 1)
user.email = "new_email@example.com"
db.session.commit()             # SQLAlchemy tracks the change automatically

# Delete
user = db.session.get(User, 1)
db.session.delete(user)
db.session.commit()
```

## Migrations with Flask-Migrate (Alembic)

Model-only changes (`db.create_all()`) don't alter existing tables. For
real projects, use Flask-Migrate to generate versioned migration scripts.

```bash
flask db init            # one-time setup, creates a migrations/ folder
flask db migrate -m "add users table"    # auto-generate a migration from model changes
flask db upgrade            # apply pending migrations to the database
flask db downgrade            # roll back the last migration
```
Docs: https://flask-migrate.readthedocs.io/

Setup in code:
```python
from flask_migrate import Migrate

migrate = Migrate(app, db)
```