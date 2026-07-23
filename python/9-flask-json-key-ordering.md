# Flask JSON Key Ordering

**Docs:** https://flask.palletsprojects.com/en/3.0.x/api/#flask.Flask.json

---

## The Problem

`jsonify()` may serialize dict keys in an unexpected order depending on Flask version.

---

## Fix by Flask Version

### Flask 3.0+ (current)

Flask 3.0 removed `JSON_SORT_KEYS` and its custom JSON provider. Set `sort_keys` directly on the app's JSON provider:

```python
from flask import Flask, jsonify

app = Flask(__name__)

# Disables alphabetical key sorting in Flask 3.0+
app.json.sort_keys = False
```

### Flask 2.x

```python
app.config["JSON_SORT_KEYS"] = False
```

### Flask < 2.2

```python
app.config["JSON_SORT_KEYS"] = False  # same key, supported in older config system
```

---

## Guaranteeing Order with OrderedDict

For explicit, deterministic key order regardless of Flask version, use `OrderedDict` with a list of tuples:

```python
from flask import jsonify
from collections import OrderedDict

@app.route('/')
def home():
    # OrderedDict([('key', value), ...]) locks in insertion order explicitly.
    # Nested dicts must also be OrderedDict if their order matters.
    return jsonify(OrderedDict([
        ('message', 'Weather Data Collection API'),
        ('endpoints', OrderedDict([
            ('/', 'This help message'),
            ('/health', 'GET - Check if API is running'),
            ('/collect', 'POST - Collect weather data'),
            ('/data', 'GET - View collected data'),
            ('/stats', 'GET - View statistics')
        ])),
        ('example_usage', OrderedDict([
            ('collect_default', 'curl -X POST http://localhost:5000/collect'),
            ('collect_custom', 'curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d \'{"lat": 40.7128, "lon": -74.0060}\''),
            ('view_data', 'curl http://localhost:5000/data'),
            ('view_stats', 'curl http://localhost:5000/stats')
        ]))
    ]))
```

> **Note:** Plain `{}` dict literals preserve insertion order in Python 3.7+, but `OrderedDict` makes the intent explicit and is safer across serialization contexts.

---

## Key Distinction

| Approach | When to Use |
|---|---|
| `app.json.sort_keys = False` | Flask 3.0+ global fix — apply once at app init |
| `app.config["JSON_SORT_KEYS"] = False` | Flask 2.x global fix |
| `OrderedDict([...])` | When you need guaranteed explicit order per-route |
| `json.dumps(data, sort_keys=True)` | When you *want* alphabetical order (e.g. diffs, tests) |

---

## Related Docs

- [Flask JSON Provider (3.0)](https://flask.palletsprojects.com/en/3.0.x/api/#flask.Flask.json)
- [collections.OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict)
- [json.dumps](https://docs.python.org/3/library/json.html#json.dumps)