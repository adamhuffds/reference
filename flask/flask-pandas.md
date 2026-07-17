# Flask Reference — Using pandas in Flask

pandas docs: https://pandas.pydata.org/docs/

Common use cases: accepting a CSV/Excel upload and analyzing it, rendering
tabular data as an HTML table, and returning DataFrame contents as JSON from
an API endpoint. This ties together `flask-forms-wtf.md` (file upload) and
`flask-rest-api-auth.md` (JSON responses).

## Reading an Uploaded File into a DataFrame

```python
import pandas as pd
from flask import request

@app.route("/analyze", methods=["POST"])
def analyze():
    f = request.files["file"]
    df = pd.read_csv(f)          # pandas can read directly from the file object, no need to save to disk first
    return f"Loaded {len(df)} rows, columns: {list(df.columns)}"
```
For Excel files: `pd.read_excel(f)` (requires `openpyxl` installed:
`pip install openpyxl`).

Reading directly from the upload's file object (rather than saving to disk
first) is fine for one-off analysis, but if you need the raw file
afterward (audit trail, reprocessing), save it first with
`secure_filename()` as shown in `flask-forms-wtf.md`, then read from disk.

## Rendering a DataFrame as an HTML Table

```python
from flask import render_template

@app.route("/table")
def table():
    df = pd.read_csv("data.csv")
    table_html = df.to_html(classes="table table-striped", index=False)
    return render_template("table.html", table=table_html)
```
`template.html`:
```html
{{ table|safe }}
```
- `classes=` lets you attach CSS framework classes (e.g. Bootstrap) directly
  to the generated `<table>` tag
- `index=False` omits the DataFrame's row index column from the output
- `|safe` is required here since `to_html()` returns actual HTML — Jinja
  would otherwise escape the `<table>` tags into visible text (see the
  auto-escaping note in `flask-templates-jinja.md`). Only mark trusted,
  server-generated HTML like this as `|safe` — never do this with raw user
  input.

## Returning DataFrame Data as JSON

```python
from flask import jsonify

@app.route("/api/data")
def api_data():
    df = pd.read_csv("data.csv")
    return jsonify(df.to_dict(orient="records"))
```
`orient="records"` produces `[{"col1": val, "col2": val}, ...]` — a list of
row-dicts, the shape most JSON APIs and frontend charting libraries expect.
Other useful orientations: `"columns"` (dict of column -> list of values),
`"split"` (compact, separates columns/index/data).

### The NaN Gotcha

```python
df = df.where(pd.notnull(df), None)      # convert NaN -> None before jsonify
return jsonify(df.to_dict(orient="records"))
```
`NaN` (pandas' missing-value marker) is **not valid JSON** — Python's
`json` module serializes it as the literal `NaN`, which many JSON parsers
(including strict ones in JavaScript) will choke on. Converting `NaN` to
`None` first (which becomes JSON `null`) avoids silently shipping malformed
JSON to API consumers.

## Common Data Cleaning Before Returning/Rendering

```python
df = df.dropna(subset=["required_column"])       # drop rows missing a required field
df = df.fillna("")                                  # or fill missing values instead of dropping
df.columns = df.columns.str.strip().str.lower()       # normalize messy column headers
df["date"] = pd.to_datetime(df["date"], errors="coerce")   # parse dates, invalid ones become NaT
```

## Performance Note for Larger Files

Loading an entire large CSV into memory on every request is expensive. For
files beyond what comfortably fits in memory, or endpoints hit frequently:
- Cache the loaded DataFrame (e.g. with `flask-caching`) instead of
  re-reading the file on every request
- Consider chunked reading: `pd.read_csv(f, chunksize=10000)` and process in
  batches
- For genuinely large datasets, a real database query (see
  `flask-database-sqlalchemy.md`) is usually a better fit than repeatedly
  parsing a flat file

## Example: Full Upload → Analyze → Display Flow

```python
@app.route("/upload-analyze", methods=["GET", "POST"])
def upload_analyze():
    if request.method == "POST":
        f = request.files["file"]
        df = pd.read_csv(f)

        summary = {
            "rows": len(df),
            "columns": list(df.columns),
            "missing_values": df.isnull().sum().to_dict(),
        }
        table_html = df.head(20).to_html(classes="table", index=False)

        return render_template("results.html", summary=summary, table=table_html)

    return render_template("upload.html")
```