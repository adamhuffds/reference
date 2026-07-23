# JSON & jsonify: A Practical Crash Course

## Part 1: What is JSON?

**JSON (JavaScript Object Notation)** is a lightweight data interchange format that's human-readable and language-independent. Think of it as a universal language for sending data between systems.

### Basic Structure

```python
# Python dictionary
person = {
    "name": "Adam",
    "age": 30,
    "skills": ["Python", "SQL", "Docker"],
    "active": True,
    "projects": {
        "weather_app": "in_progress",
        "portfolio": "planning"
    }
}

# This looks like JSON, but it's actually a Python dict
# JSON is the TEXT representation of this data
```

### JSON Data Types

JSON supports only these types:
- **String**: `"hello"` (must use double quotes)
- **Number**: `42`, `3.14` (no distinction between int/float)
- **Boolean**: `true`, `false` (lowercase!)
- **Null**: `null` (not `None`)
- **Array**: `[1, 2, 3]`
- **Object**: `{"key": "value"}`

**Key differences from Python:**

```python
# Python dict
python_data = {
    'name': 'Adam',        # Single quotes OK
    'value': None,         # Python's None
    'active': True         # Capitalized
}

# Valid JSON (as string)
json_string = '''
{
    "name": "Adam",        
    "value": null,         
    "active": true         
}
'''
```

## Part 2: Python's `json` Module

Python's standard library provides the `json` module for converting between Python objects and JSON strings.

### Serialization: Python → JSON String

```python
import json

# Python object
data = {
    "user": "adam",
    "scores": [95, 87, 92],
    "verified": True
}

# Convert to JSON string
json_string = json.dumps(data)
# Result: '{"user": "adam", "scores": [95, 87, 92], "verified": true}'

# Pretty-printed version
json_pretty = json.dumps(data, indent=2)
print(json_pretty)
# {
#   "user": "adam",
#   "scores": [95, 87, 92],
#   "verified": true
# }

# Write to file
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)  # dump() writes directly to file
```

**Key functions:**
- `json.dumps()`: **dump string** — returns JSON as a string
- `json.dump()`: **dump** — writes JSON directly to a file object

### Deserialization: JSON String → Python

```python
import json

# JSON string
json_string = '{"name": "Adam", "skills": ["Python", "SQL"]}'

# Convert to Python dict
data = json.loads(json_string)
# Result: {'name': 'Adam', 'skills': ['Python', 'SQL']}

# Read from file
with open('data.json', 'r') as f:
    data = json.load(f)  # load() reads from file object
```

**Key functions:**
- `json.loads()`: **load string** — parses JSON string to Python
- `json.load()`: **load** — reads and parses JSON from file

### Common Serialization Options

```python
import json
from datetime import datetime

data = {
    "timestamp": datetime.now(),  # This will cause an error!
    "value": 42.123456789
}

# Handle non-serializable objects
def json_serializer(obj):
    """Custom serializer for objects json doesn't handle"""
    if isinstance(obj, datetime):
        return obj.isoformat()  # Convert datetime to ISO string
    raise TypeError(f"Type {type(obj)} not serializable")

json_string = json.dumps(data, default=json_serializer)

# Other useful parameters:
json_string = json.dumps(
    data,
    indent=2,           # Pretty print with 2-space indentation
    sort_keys=True,     # Sort dictionary keys alphabetically
    ensure_ascii=False  # Allow unicode characters instead of \uXXXX escapes
)
```

## Part 3: Flask's `jsonify()`

`jsonify()` is Flask's wrapper around `json.dumps()` that creates proper HTTP responses.

### Basic Usage

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/user')
def get_user():
    # Option 1: Pass dict directly
    return jsonify({
        "name": "Adam",
        "role": "Data Engineer"
    })

    # Option 2: Pass keyword arguments
    return jsonify(
        name="Adam",
        role="Data Engineer"
    )

    # Both create the same JSON response with proper headers
```

### What `jsonify()` Actually Does

```python
from flask import jsonify, Response
import json

# These are roughly equivalent:

# Using jsonify (RECOMMENDED)
@app.route('/api/data')
def with_jsonify():
    data = {"key": "value"}
    return jsonify(data)

# Manual approach (DON'T DO THIS)
@app.route('/api/data')
def manual():
    data = {"key": "value"}
    json_string = json.dumps(data)
    return Response(
        json_string,
        mimetype='application/json',  # jsonify sets this automatically
        status=200                     # jsonify handles this too
    )
```

**What `jsonify()` adds:**
1. Sets `Content-Type: application/json` header
2. Returns a Flask `Response` object
3. Handles status codes: `return jsonify(data), 201`
4. Handles arrays properly (Flask-specific quirk, see below)

### Returning Status Codes

```python
from flask import jsonify

@app.route('/api/resource', methods=['POST'])
def create_resource():
    # Success with 201 Created
    return jsonify({"id": 123, "status": "created"}), 201

@app.route('/api/error')
def error_example():
    # Error with 404 Not Found
    return jsonify({"error": "Resource not found"}), 404

@app.route('/api/validation')
def validation_error():
    # Bad request with 400
    return jsonify({
        "error": "Validation failed",
        "details": ["Email is required", "Age must be positive"]
    }), 400
```

### Array Responses

```python
from flask import jsonify

@app.route('/api/users')
def get_users():
    users = [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ]

    # jsonify handles lists directly
    return jsonify(users)
    # Returns: [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]

    # Or wrap in a dict (often better for API design, easier to extend later)
    return jsonify({"users": users, "count": len(users)})
    # Returns: {"users": [...], "count": 2}
```

## Part 4: Practical Examples

### Example 1: SQLAlchemy Model → JSON

```python
from flask import Flask, jsonify
from sqlalchemy import Column, Integer, String, Float, DateTime
from sqlalchemy.orm import DeclarativeBase

# Modern SQLAlchemy 2.0 pattern (preferred over legacy declarative_base())
class Base(DeclarativeBase):
    pass

class WeatherReading(Base):
    __tablename__ = 'weather_readings'

    id = Column(Integer, primary_key=True)
    city = Column(String(100))
    temperature = Column(Float)
    humidity = Column(Float)
    timestamp = Column(DateTime)

    def to_dict(self):
        """Convert model instance to a plain dict jsonify can serialize"""
        return {
            "id": self.id,
            "city": self.city,
            "temperature": self.temperature,
            "humidity": self.humidity,
            # datetime objects aren't JSON-serializable on their own,
            # so convert to an ISO 8601 string first
            "timestamp": self.timestamp.isoformat() if self.timestamp else None
        }

@app.route('/api/weather/<city>')
def get_weather(city):
    # Query database
    reading = session.query(WeatherReading).filter_by(city=city).first()

    if not reading:
        return jsonify({"error": "City not found"}), 404

    # Convert to dict, then to JSON response
    return jsonify(reading.to_dict())
```

### Example 2: Handling External API Response

```python
import requests
from datetime import datetime
from flask import jsonify

@app.route('/api/fetch-weather/<city>')
def fetch_external_weather(city):
    """Fetch from external API and return formatted JSON"""

    # Make request to external API
    api_url = f"https://api.weather.com/data?city={city}"
    response = requests.get(api_url)

    # Parse JSON response from external API
    external_data = response.json()  # This uses json.loads() internally

    # Transform and return
    formatted_data = {
        "city": external_data.get("name"),
        "temp_celsius": external_data.get("main", {}).get("temp"),
        "conditions": external_data.get("weather", [{}])[0].get("description"),
        "fetched_at": datetime.now().isoformat()
    }

    return jsonify(formatted_data)
```

### Example 3: Saving JSON to PostgreSQL JSONB

```python
from flask import request, jsonify
from sqlalchemy import Float
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.dialects.postgresql import JSONB

class WeatherData(Base):
    __tablename__ = 'weather_data'

    id: Mapped[int] = mapped_column(primary_key=True)
    # JSONB (binary JSON) allows querying into the JSON structure
    # directly in PostgreSQL, unlike plain JSON/text columns
    raw_data: Mapped[dict] = mapped_column(JSONB)

@app.route('/api/store-weather', methods=['POST'])
def store_weather():
    # request.get_json() parses the incoming request body
    # (must have Content-Type: application/json) into a Python dict
    incoming_data = request.get_json()

    # Store entire JSON structure in a single JSONB column
    new_record = WeatherData(raw_data=incoming_data)
    session.add(new_record)
    session.commit()

    return jsonify({
        "status": "success",
        "id": new_record.id
    }), 201

@app.route('/api/query-weather')
def query_weather():
    # Query into the JSONB field: extract "temperature" as text,
    # cast it to Float, and filter on it
    results = session.query(WeatherData).filter(
        WeatherData.raw_data['temperature'].astext.cast(Float) > 20
    ).all()

    return jsonify([r.raw_data for r in results])
```

## Part 5: Common Gotchas

### 1. Datetime Serialization

```python
from datetime import datetime
import json

# This FAILS
data = {"time": datetime.now()}
json.dumps(data)  # TypeError: datetime is not JSON serializable

# Solution 1: Convert to string first
data = {"time": datetime.now().isoformat()}
json.dumps(data)  # Works!

# Solution 2: Custom serializer function passed via `default`
def json_serial(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")

json.dumps(data, default=json_serial)
```

### 2. Single vs Double Quotes

```python
import json

# Invalid JSON (a Python dict literal is fine, but this is a STRING of it)
invalid = "{'name': 'Adam'}"
json.loads(invalid)  # JSONDecodeError

# Valid JSON uses double quotes for keys and string values
valid = '{"name": "Adam"}'
json.loads(valid)  # Works!
```

### 3. Trailing Commas

```python
# Python: OK
python_data = {
    "name": "Adam",
    "age": 30,  # Trailing comma is fine in Python syntax
}

# JSON: NOT OK — the JSON spec does not allow trailing commas
json_string = '''
{
    "name": "Adam",
    "age": 30,
}
'''
json.loads(json_string)  # JSONDecodeError
```

### 4. None vs null

```python
import json

# Python
data = {"value": None}

# Converts to JSON null
json_string = json.dumps(data)  # '{"value": null}'

# Back to Python None
parsed = json.loads(json_string)  # {'value': None}
```

## Quick Reference

```python
# Standard library
import json

# Serialize (Python → JSON string)
json.dumps(obj)          # Returns string
json.dump(obj, file)     # Writes to file

# Deserialize (JSON string → Python)
json.loads(string)       # From string
json.load(file)          # From file

# Flask
from flask import jsonify, request

# Create JSON response
jsonify(data)            # Dict/list to JSON HTTP response
jsonify(data), 201       # With status code

# Parse incoming JSON
request.get_json()       # Parse request body
```

## Documentation Links

- Python `json` module: https://docs.python.org/3/library/json.html
- Flask `jsonify`: https://flask.palletsprojects.com/en/3.0.x/api/#flask.json.jsonify
- Flask `request.get_json`: https://flask.palletsprojects.com/en/3.0.x/api/#flask.Request.get_json
- JSON specification: https://www.json.org/json-en.html
- SQLAlchemy 2.0 `DeclarativeBase`: https://docs.sqlalchemy.org/en/20/orm/declarative_styles.html
- PostgreSQL `JSONB` type (SQLAlchemy dialect): https://docs.sqlalchemy.org/en/20/dialects/postgresql.html#sqlalchemy.dialects.postgresql.JSONB