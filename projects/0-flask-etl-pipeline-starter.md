# Flask ETL Pipeline Starter — Project Summary

A summary of everything covered: what was built, what each file does, and what's next.

## 📦 What You Have

A Dockerized Flask + SQLAlchemy + PostgreSQL starter project, ready to be extended into an ETL pipeline.

| File | Purpose |
|---|---|
| `docker-compose.yml` | Defines and links two containers: Flask app (`web`) and PostgreSQL (`db`) |
| `Dockerfile` | Recipe for building the Flask container (Python 3.11 + dependencies) |
| `requirements.txt` | Python packages: Flask, Flask-SQLAlchemy, psycopg2-binary, python-dotenv, requests |
| `app.py` | Main Flask app — config, database model, routes, and startup logic |
| `templates/index.html` | Homepage UI that displays DB status and data via JS `fetch()` calls |
| `README.md` | Quick-start instructions |

Commented versions of each file (with line-by-line explanations) were also created:
`docker-compose-commented.yml`, `Dockerfile-commented`, `app-commented.py`, `requirements-commented.txt`, `html-explanation.md`, and `system-guide.md` (the big-picture architecture doc).

## 🧠 Core Concepts (Deep Dive)

### 1. Docker & Docker Compose

**The problem it solves**: "works on my machine" — different OSes, Python versions, missing system libraries.

**How it works**: A `Dockerfile` builds an *image* (a frozen snapshot of an environment). A *container* is a running instance of that image — isolated from your actual computer.

`docker-compose.yml` manages **multiple** containers together:
- Each service (`web`, `db`) gets its own container.
- Docker Compose creates a private network so containers can reach each other **by service name**. That's why `DATABASE_URL` uses `db` as the hostname, not `localhost` — from inside the `web` container, `localhost` would mean "this container," not the database container.
- `depends_on` controls startup order (though not full readiness — the DB container starts, but Postgres might take a moment to accept connections).
- `volumes` do two different jobs here:
  - `.:/app` — a **bind mount** linking your local folder to the container, so code edits show up live without rebuilding.
  - `postgres_data:/var/lib/postgresql/data` — a **named volume** Docker manages internally, so your database survives container restarts/removals.

**Image vs. container**, in short: image = class, container = instance.

### 2. The Dockerfile Build Process

Each instruction (`FROM`, `RUN`, `COPY`) creates a cached **layer**. Docker only re-runs a layer (and everything after it) if something in that layer changed. That's why `requirements.txt` is copied and `pip install` run *before* copying the rest of the code — your Python dependencies rarely change, but your code changes constantly. This ordering means most rebuilds skip the slow `pip install` step entirely.

### 3. SQLAlchemy (the ORM)

ORM = Object-Relational Mapper. It translates between Python objects and SQL rows so you rarely write raw SQL.

```python
# ORM way
entry = DataEntry(title="Hello", content="World")
db.session.add(entry)
db.session.commit()

# Equivalent raw SQL
# INSERT INTO data_entries (title, content) VALUES ('Hello', 'World');
```

Key pieces:
- **Model** (`class DataEntry(db.Model)`) = table definition. Each attribute (`id`, `title`, `content`) = a column.
- **`db.session`** = a staging area for changes. Nothing hits the database until you call `.commit()`. This lets you batch multiple changes into one transaction (all succeed, or all roll back together).
- **`.query`** = how you read: `DataEntry.query.all()`, `.filter_by(title="x")`, `.get(id)`.
- **Why bother**: prevents SQL injection (parameters are escaped automatically), makes the code portable across database engines, and lets you think in Python objects instead of strings of SQL.

### 4. Flask Routing & Requests

`@app.route('/api/data')` is a **decorator** — it registers the function below it to run whenever that URL is requested. Flask matches the incoming URL to the right function and calls it.

- Routes returning HTML use `render_template()` (looks in the `templates/` folder automatically).
- Routes returning JSON use `jsonify()` — this also sets the correct `Content-Type: application/json` header, which matters for the JS `fetch()` calls to parse it correctly.
- The `app.app_context()` block matters because Flask-SQLAlchemy needs to know *which app* it's attached to before it can touch the database — this becomes more relevant once you have multiple modules importing `db`.

### 5. Frontend ↔ Backend Communication

The HTML page is a client that talks to Flask purely through HTTP requests — the same pattern used by full frontend frameworks (React, Vue), just simplified with vanilla JS:

```javascript
const response = await fetch('/api/data');  // sends GET request
const data = await response.json();          // parses JSON body
```

`async/await` just makes asynchronous code (things that take time, like network requests) read top-to-bottom instead of nesting callbacks. `setInterval(...)` re-runs this every 10 seconds so the page reflects new data without a manual refresh — this is the same *polling* pattern your future ETL pipeline will effectively feed data into.

### 6. Where an ETL Pipeline Fits In

Nothing in this codebase pulls from an external API yet — `app.py` only reads/writes to Postgres. The pipeline you're planning would be a **separate process** (script or scheduled job) that:
1. **Extracts**: calls the public API with `requests.get(...)`
2. **Transforms**: cleans/reshapes the JSON response into your model's fields
3. **Loads**: creates `DataEntry` objects and commits them via SQLAlchemy — the exact same `db.session.add()/commit()` pattern already in `app.py`

That's the piece we'll build next once you pick an API.

## 🔄 How a Request Flows

1. Browser hits `http://localhost:5000/` → Flask's `index()` renders `index.html`.
2. Page JS calls `/api/health` and `/api/data`.
3. Flask queries PostgreSQL via SQLAlchemy → converts rows to dicts → returns JSON.
4. JS updates the DOM with the results.

## 🚀 Running It

```bash
docker-compose up --build
```
Visit `http://localhost:5000`. Stop with `docker-compose down` (add `-v` to also wipe DB data).

## 🛠️ Next Steps (Not Yet Built)

1. **`etl.py`** — script to pull data from your public API (`requests.get(...)`)
2. **Data model tailored to your API's fields** (replace/extend `DataEntry`)
3. **Scheduling** — e.g., APScheduler or a cron job to run the ETL periodically
4. **Error handling & logging** for API failures
5. **Data validation** before inserting into PostgreSQL

## ❓ Still Open

- Which public API will the ETL pipeline pull from? (Needed to build `etl.py` and tailor the data model.)