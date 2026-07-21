# JSON Basics: API Calls & PostgreSQL

Reference notes on JSON structure, Python serialization, API request/response patterns, and PostgreSQL JSON/JSONB usage.

---

## 1. What JSON Is

JSON (JavaScript Object Notation) is a lightweight, text-based, language-agnostic data format. It's the de facto standard for API payloads and is natively supported by PostgreSQL for semi-structured data storage.

**Docs:** https://www.json.org/json-en.html

### Core structures

- **Object** — key-value pairs (maps to a Python `dict`):
  ```json
  { "name": "Alice", "age": 30, "active": true }
  ```
- **Array** — ordered list of values (maps to a Python `list`):
  ```json
  ["apple", "banana", "orange"]
  ```
- Objects and arrays nest freely inside each other.

### Data types

| JSON type | Example        | Python equivalent |
|-----------|----------------|--------------------|
| String    | `"text"`       | `str`              |
| Number    | `42`, `3.14`   | `int`, `float`     |
| Boolean   | `true`/`false` | `True`/`False`     |
| Null      | `null`         | `None`             |
| Object    | `{...}`        | `dict`             |
| Array     | `[...]`        | `list`             |

Note the lowercase `true`/`false`/`null` — this trips people up coming from Python.

---

## 2. JSON in Python: `json` module

**Docs:** https://docs.python.org/3/library/json.html

- `json.dumps(obj)` — **serialize**: Python object → JSON string
- `json.loads(json_str)` — **deserialize**: JSON string → Python object

```python
import json

data = {"name": "Alice", "age": 30}

json_string = json.dumps(data)      # dict -> JSON string
parsed_data = json.loads(json_string)  # JSON string -> dict
```

---

## 3. JSON in API Calls (`requests`)

**Docs:** https://docs.python-requests.org/en/latest/user/quickstart/#more-complicated-post-requests

Passing `json=payload` to `requests` automatically serializes the dict and sets the `Content-Type: application/json` header. Calling `.json()` on the response automatically deserializes it back to a Python object.

```python
import requests

payload = {
    "query": "machine learning",
    "filters": {
        "date_range": "2024",
        "category": "research"
    }
}

response = requests.post(
    "https://api.example.com/search",
    json=payload,  # requests handles json.dumps() internally
    headers={"Content-Type": "application/json"}
)

result = response.json()  # requests handles json.loads() internally
```

### Navigating nested API responses

Access nested objects/arrays the same way you would a nested dict/list in Python:

```python
response_data = {
    "status": "success",
    "data": {
        "users": [
            {"id": 1, "name": "Alice"},
            {"id": 2, "name": "Bob"}
        ]
    }
}

for user in response_data["data"]["users"]:
    print(f"User {user['id']}: {user['name']}")
```

### Common API JSON shapes

**Pagination:**
```json
{ "page": 1, "per_page": 20, "total": 150, "data": [] }
```

**Error response:**
```json
{ "error": { "code": "INVALID_INPUT", "message": "Email format is invalid" } }
```

---

## 4. JSON in PostgreSQL

**Docs:** https://www.postgresql.org/docs/current/datatype-json.html

PostgreSQL has two native JSON types:

| Type    | Storage                  | When to use                          |
|---------|---------------------------|---------------------------------------|
| `JSON`  | Stored as exact text       | Rarely — only if you need to preserve exact formatting/key order |
| `JSONB` | Stored as decomposed binary | **Default choice** — faster to query, supports indexing |

### Creating a table with JSONB

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_data JSONB
);

INSERT INTO events (event_data) VALUES
('{"user_id": 123, "action": "login", "timestamp": "2024-01-15T10:30:00Z"}');
```

### JSON operators

**Docs:** https://www.postgresql.org/docs/current/functions-json.html

| Operator | Purpose                                  |
|----------|-------------------------------------------|
| `->`     | Extract field, **keep as JSON**            |
| `->>`    | Extract field **as text**                  |
| `#>`     | Navigate a nested path, keep as JSON       |
| `#>>`    | Navigate a nested path, return as text     |
| `?`      | Check whether a key exists                 |

```sql
-- Extract a field as text
SELECT event_data->>'action' AS action FROM events;

-- Navigate nested JSON
SELECT event_data->'metadata'->>'browser' FROM events;

-- Filter rows by a JSON field value
SELECT * FROM events WHERE event_data->>'user_id' = '123';

-- Check if a key exists
SELECT * FROM events WHERE event_data ? 'error_code';
```

### Python ↔ PostgreSQL JSON with `psycopg2`

**Docs:** https://www.psycopg.org/docs/

```python
import psycopg2
import json

conn = psycopg2.connect("dbname=mydb user=postgres")
cur = conn.cursor()

event = {
    "user_id": 456,
    "action": "purchase",
    "items": ["book", "pen"],
    "total": 25.50
}

# Insert: serialize dict to JSON string before passing as a parameter
cur.execute(
    "INSERT INTO events (event_data) VALUES (%s)",
    (json.dumps(event),)
)

# Query: deserialize the returned JSON string back into a dict
cur.execute("SELECT event_data FROM events WHERE id = 1")
row = cur.fetchone()
event_data = json.loads(row[0])

conn.commit()
cur.close()
conn.close()
```

> Note: with `psycopg2`'s `register_default_jsonb`, or with SQLAlchemy's `JSONB` column type, this serialize/deserialize step can be handled automatically. Worth exploring once comfortable with the manual pattern above — see SQLAlchemy's JSON type docs: https://docs.sqlalchemy.org/en/20/core/type_basics.html#sqlalchemy.types.JSON

---

## 5. Why JSON Matters Here

- **APIs:** lightweight, human-readable, language-agnostic — every language can parse it, making it the standard for request/response payloads.
- **PostgreSQL (JSONB):** lets you store flexible/semi-structured data (e.g., API responses, event logs, variable metadata) without needing a rigid schema or constant migrations, while still being queryable and indexable.

---

## 6. Key Takeaway

The same mental model — nested key-value structures — applies whether you're looking at a Python `dict`, an API JSON payload, or a PostgreSQL `JSONB` column. The `json` module is the bridge between Python and both API and database representations.