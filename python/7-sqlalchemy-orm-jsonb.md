# SQLAlchemy ORM with PostgreSQL and JSONB

## What is an ORM?

An **ORM (Object-Relational Mapper)** lets you interact with a database using
your programming language's objects instead of writing raw SQL. It maps:

- Python classes → database tables
- Class attributes → columns
- Class instances → rows

**Benefits:** less boilerplate, database portability, parameterized queries
(protection against SQL injection), easier maintenance via Python classes.

**Drawbacks:** generated SQL isn't always optimal, adds a learning curve, can
obscure what's actually happening at the SQL level.

For PostgreSQL, **SQLAlchemy** is the industry-standard Python ORM and has
first-class support for Postgres-specific features like JSONB, arrays, and
advanced indexing.

Docs: https://docs.sqlalchemy.org/en/20/

---

## What is JSONB?

**JSONB** is PostgreSQL's binary JSON storage format, distinct from plain
`JSON`:

- **Faster to process** — stored in decomposed binary form, not raw text
- **Indexable** — supports GIN indexes for fast key/containment lookups
- **Queryable** — search inside JSON structures with native operators
- **Normalized on write** — whitespace/key order/duplicate keys are removed

Good fit for semi-structured data: user preferences, flexible product
attributes, API payloads, audit/event metadata.

Docs: https://www.postgresql.org/docs/current/datatype-json.html

---

## ⚠️ Modern vs. Legacy SQLAlchemy

SQLAlchemy 2.0 changed the recommended declarative pattern. Older
tutorials/blogs (and my earlier reply in this chat) often still show the
**legacy** style — use the **modern 2.0** style going forward.

| | Legacy (SQLAlchemy 1.x style) | Modern (SQLAlchemy 2.0, current best practice) |
|---|---|---|
| Base class | `declarative_base()` | `class Base(DeclarativeBase)` |
| Column definition | `Column(Integer, primary_key=True)` | `Mapped[int] = mapped_column(primary_key=True)` |
| Typing | No type hints; type lives only in the `Column` call | Uses Python type hints (`Mapped[...]`) — your IDE/type checker understands your model |
| Querying | `session.query(Model).filter(...)` | `select(Model).where(...)` executed via `session.execute()` / `session.scalars()` |
| Session usage | Manual `session = Session()` / `session.close()` | Context manager: `with Session(engine) as session:` |

Docs: https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html#orm-mapping-styles
Docs (what's new in 2.0): https://docs.sqlalchemy.org/en/20/changelog/migration_20.html

The rest of this document uses the **modern 2.0 style**.

---

## Setup

```bash
# Install SQLAlchemy and the PostgreSQL driver.
# psycopg2-binary is the PostgreSQL adapter for Python — it's what actually
# speaks the Postgres wire protocol under the hood; SQLAlchemy sits on top
# of it and generates the SQL.
pip install sqlalchemy psycopg2-binary
```

Docs: https://docs.sqlalchemy.org/en/20/core/engines.html#postgresql

---

## Defining a Model with a JSONB Column

```python
from datetime import datetime

from sqlalchemy import String, DateTime
from sqlalchemy.dialects.postgresql import JSONB
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


# DeclarativeBase is the SQLAlchemy 2.0 way to create your model base class.
# All ORM models inherit from this. This replaces the legacy declarative_base().
# Docs: https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html#declarative-mapping
class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"  # actual PostgreSQL table name

    # Mapped[int] is a type hint telling SQLAlchemy (and your IDE) that this
    # attribute is an int. mapped_column() is where you configure the actual
    # column behavior (primary key, constraints, defaults, etc).
    # Docs: https://docs.sqlalchemy.org/en/20/orm/declarative_tables.html
    id: Mapped[int] = mapped_column(primary_key=True)

    # String(255) sets a VARCHAR(255) limit at the DB level.
    # unique=True adds a UNIQUE constraint.
    email: Mapped[str] = mapped_column(String(255), unique=True)

    # The JSONB column — Postgres-specific type imported from the postgresql
    # dialect module. default=dict means each new row starts as an empty {}
    # rather than NULL. Using the `dict` callable (not `{}` directly) avoids
    # all instances sharing the same mutable default object.
    # Docs: https://docs.sqlalchemy.org/en/20/dialects/postgresql.html#sqlalchemy.dialects.postgresql.JSONB
    preferences: Mapped[dict] = mapped_column(JSONB, default=dict)

    # datetime.utcnow (no parentheses) is passed as a callable — SQLAlchemy
    # calls it at insert time, so every row gets its own timestamp.
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)

    def __repr__(self) -> str:
        return f"<User(id={self.id}, email='{self.email}')>"
```

## Engine and Session

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import Session

# The engine is the connection point to PostgreSQL.
# Format: postgresql+psycopg2://username:password@host:port/database_name
# echo=True logs every generated SQL statement to stdout — very useful while
# learning, turn it off in production.
# Docs: https://docs.sqlalchemy.org/en/20/core/engines.html
engine = create_engine(
    "postgresql+psycopg2://username:password@localhost:5432/mydb",
    echo=True,
)

# Creates all tables defined by Base subclasses (issues CREATE TABLE).
# In a real project you'd use Alembic migrations instead of create_all()
# once your schema needs to evolve over time.
# Docs (create_all): https://docs.sqlalchemy.org/en/20/core/metadata.html#sqlalchemy.schema.MetaData.create_all
# Docs (Alembic): https://alembic.sqlalchemy.org/en/latest/
Base.metadata.create_all(engine)
```

---

## CREATE — Inserting JSONB Data

```python
new_user = User(
    email="alice@example.com",
    preferences={
        "theme": "dark",
        "notifications": {
            "email": True,
            "push": False,
            "frequency": "daily",
        },
        "languages": ["en", "es"],
        "privacy": {
            "profile_visible": True,
            "show_email": False,
        },
    },
)

# Session as a context manager (modern pattern) — automatically closes
# the session when the block exits, even on error.
# Docs: https://docs.sqlalchemy.org/en/20/orm/session_basics.html
with Session(engine) as session:
    session.add(new_user)
    session.commit()
    print(f"Created user: {new_user.id}")
```

---

## READ — Querying JSONB Data

The 2.0 style builds a `select()` statement and executes it via the session,
rather than using the legacy `session.query()`.

Docs: https://docs.sqlalchemy.org/en/20/orm/queryguide/query.html

```python
from sqlalchemy import select

with Session(engine) as session:

    # --- All users ---
    all_users = session.scalars(select(User)).all()

    # --- Filter by a regular column ---
    stmt = select(User).where(User.email == "alice@example.com")
    user = session.scalars(stmt).first()

    # --- JSONB: filter by top-level key ---
    # SQL generated: WHERE preferences ->> 'theme' = 'dark'
    # ->> extracts a JSON field and casts it to text for comparison.
    # .astext is SQLAlchemy's way of requesting the ->> operator.
    # Docs: https://docs.sqlalchemy.org/en/20/dialects/postgresql.html#comparator-methods
    stmt = select(User).where(User.preferences["theme"].astext == "dark")
    dark_theme_users = session.scalars(stmt).all()

    # --- JSONB: filter by nested key ---
    # SQL: WHERE preferences -> 'notifications' ->> 'email' = 'true'
    # Chain [] indexing to walk into nested objects; .astext on the last hop.
    stmt = select(User).where(
        User.preferences["notifications"]["email"].astext == "true"
    )
    email_notification_users = session.scalars(stmt).all()

    # --- JSONB: does key exist? ---
    # SQL: WHERE preferences ? 'theme'
    stmt = select(User).where(User.preferences.has_key("theme"))
    users_with_theme = session.scalars(stmt).all()

    # --- JSONB: contains ALL of these keys? ---
    # SQL: WHERE preferences ?& array['theme', 'notifications']
    stmt = select(User).where(User.preferences.has_all(["theme", "notifications"]))
    users_with_multiple_keys = session.scalars(stmt).all()

    # --- JSONB: contains ANY of these keys? ---
    # SQL: WHERE preferences ?| array['theme', 'dark_mode']
    stmt = select(User).where(User.preferences.has_any(["theme", "dark_mode"]))
    users_with_any_keys = session.scalars(stmt).all()

    # --- JSONB: array containment ---
    # SQL: WHERE preferences -> 'languages' @> '"en"'
    # @> checks whether the left JSON value contains the right JSON value.
    stmt = select(User).where(User.preferences["languages"].contains(["en"]))
    english_speakers = session.scalars(stmt).all()
```

---

## UPDATE — Modifying JSONB Data

There are three common approaches, in order of how "Pythonic" vs. "SQL-native"
they are.

```python
from sqlalchemy import update
from sqlalchemy.orm.attributes import flag_modified

with Session(engine) as session:

    # --- Approach 1: Replace the entire JSONB column ---
    stmt = select(User).where(User.email == "alice@example.com")
    user = session.scalars(stmt).first()
    user.preferences = {
        "theme": "light",
        "notifications": {"email": False, "push": True},
    }
    session.commit()

    # --- Approach 2: Mutate nested keys in Python, then flag as modified ---
    # SQLAlchemy tracks changes by detecting attribute *reassignment*.
    # Mutating a dict in place (user.preferences['theme'] = 'light') doesn't
    # trigger that detection, so you must call flag_modified() to tell the
    # session "this attribute changed, include it in the UPDATE".
    # Docs: https://docs.sqlalchemy.org/en/20/orm/session_state_management.html#flagging-attributes-as-modified
    stmt = select(User).where(User.email == "alice@example.com")
    user = session.scalars(stmt).first()
    user.preferences["theme"] = "light"
    user.preferences["notifications"]["push"] = True
    flag_modified(user, "preferences")
    session.commit()

    # --- Approach 3: Server-side merge using Postgres || operator ---
    # SQL: UPDATE users SET preferences = preferences || '{"theme": "dark"}'::jsonb
    # || merges two JSON objects; keys on the right overwrite keys on the left.
    # This avoids loading the row into Python at all — good for bulk updates.
    # Docs: https://www.postgresql.org/docs/current/functions-json.html
    stmt = (
        update(User)
        .where(User.email == "alice@example.com")
        .values(preferences=User.preferences.concat({"theme": "dark"}))
    )
    session.execute(stmt)
    session.commit()
```

---

## DELETE — Removing JSONB Keys

```python
with Session(engine) as session:

    # --- In Python, then flag_modified ---
    stmt = select(User).where(User.email == "alice@example.com")
    user = session.scalars(stmt).first()
    if "theme" in user.preferences:
        del user.preferences["theme"]
        flag_modified(user, "preferences")
        session.commit()

    # --- Server-side, using Postgres - operator ---
    # SQL: UPDATE users SET preferences = preferences - 'theme'
    # - removes a top-level key from the JSONB value.
    stmt = (
        update(User)
        .where(User.email == "alice@example.com")
        .values(preferences=User.preferences.op("-")("theme"))
    )
    session.execute(stmt)
    session.commit()
```

---

## Indexing JSONB for Performance

```python
from sqlalchemy import Index

# GIN index — general-purpose, supports @>, ?, ?&, ?| operators.
# Best when you query many different keys/paths inside the JSONB.
# Docs: https://www.postgresql.org/docs/current/datatype-json.html#JSON-INDEXING
Index("idx_user_preferences_gin", User.preferences, postgresql_using="gin")

# B-tree index on one specific extracted path — faster than a full GIN index
# if you repeatedly filter on that exact key (e.g. preferences->>'theme').
Index(
    "idx_user_preferences_theme",
    User.preferences["theme"].astext,
    postgresql_using="btree",
)

Base.metadata.create_all(engine)
```

---

## Real-World Example: Product Catalog with Variable Attributes

```python
class Product(Base):
    __tablename__ = "products"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(255))

    # Different product categories have entirely different attribute sets —
    # a fixed relational schema would need a lot of nullable columns or
    # separate tables per category. JSONB lets one column handle all of them.
    #
    # Electronics: {"brand": "Sony", "warranty_months": 24, "voltage": "110V"}
    # Clothing:    {"size": "M", "color": "blue", "material": "cotton"}
    # Books:       {"author": "Smith", "pages": 350, "isbn": "123-456"}
    attributes: Mapped[dict] = mapped_column(JSONB, default=dict)


with Session(engine) as session:
    stmt = select(Product).where(Product.attributes["brand"].astext == "Sony")
    sony_products = session.scalars(stmt).all()

    stmt = select(Product).where(Product.attributes["size"].astext == "M")
    medium_clothes = session.scalars(stmt).all()

    stmt = select(Product).where(Product.attributes.has_key("warranty_months"))
    warranty_products = session.scalars(stmt).all()
```

---

## Key Takeaways

1. **JSONB is flexible** — good for data that doesn't fit a rigid schema.
2. **Use GIN indexes** — otherwise JSONB queries do a full table scan.
3. **`flag_modified()` is required** for in-place (nested) Python mutations to
   be picked up by the session — reassignment (`user.preferences = {...}`) is
   detected automatically, mutation is not.
4. **Balance relational and JSONB** — put frequently-filtered, structurally
   consistent fields in real columns; reserve JSONB for the genuinely
   variable parts.
5. **No foreign keys into JSONB** — you cannot enforce referential integrity
   on values stored inside a JSONB blob.

---

## Reference Links

- SQLAlchemy 2.0 ORM Quickstart: https://docs.sqlalchemy.org/en/20/orm/quickstart.html
- SQLAlchemy Declarative Mapping (2.0 style): https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html#declarative-mapping
- SQLAlchemy Migration Guide (1.x → 2.0): https://docs.sqlalchemy.org/en/20/changelog/migration_20.html
- SQLAlchemy PostgreSQL JSONB type: https://docs.sqlalchemy.org/en/20/dialects/postgresql.html#sqlalchemy.dialects.postgresql.JSONB
- SQLAlchemy Session Basics: https://docs.sqlalchemy.org/en/20/orm/session_basics.html
- SQLAlchemy `flag_modified`: https://docs.sqlalchemy.org/en/20/orm/session_state_management.html#flagging-attributes-as-modified
- PostgreSQL JSON Types: https://www.postgresql.org/docs/current/datatype-json.html
- PostgreSQL JSON Indexing: https://www.postgresql.org/docs/current/datatype-json.html#JSON-INDEXING
- PostgreSQL JSON Functions & Operators: https://www.postgresql.org/docs/current/functions-json.html
- Alembic (SQLAlchemy migrations tool): https://alembic.sqlalchemy.org/en/latest/