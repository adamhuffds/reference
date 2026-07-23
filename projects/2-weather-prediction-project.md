# Weather Prediction Project

A simple ETL pipeline that collects weather data from the OpenWeatherMap API, stores it in PostgreSQL, and enables exploratory data analysis and regression modeling in Jupyter notebooks.

## Architecture

- **PostgreSQL** — stores weather observations
- **Flask API** — fetches data from OpenWeatherMap and saves it to PostgreSQL
- **Jupyter** — for EDA and building regression models (predicting "feels like" temperature)
- **Docker Compose** — runs all three services together

## Project Structure

```
weather-prediction/
├── docker-compose.yml   # Defines postgres, flask, and jupyter containers
├── Dockerfile           # Shared image build for flask and jupyter
├── requirements.txt     # Python dependencies
├── app.py               # Flask app (all routes in one file)
├── .env                 # Your API key and settings (not committed to git)
├── .env.example         # Template for .env
├── .gitignore
├── notebooks/           # Jupyter notebooks for analysis
└── README.md
```

## Setup

### 1. Prerequisites

- Docker and Docker Compose installed
- Free OpenWeatherMap API key: https://openweathermap.org/api
  - New keys take ~2 hours to activate

### 2. Configure environment variables

Copy `.env.example` to `.env` and fill in your API key:

```bash
cp .env.example .env
```

Then edit `.env`:

```bash
OPENWEATHER_API_KEY=your-actual-api-key-here
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/weather_db
DEFAULT_LAT=51.5074
DEFAULT_LON=-0.1278
DEFAULT_CITY=London
```

### 3. Start the containers

```bash
docker-compose up --build
```

This starts:
| Service   | Port  | Purpose                        |
|-----------|-------|---------------------------------|
| postgres  | 5432  | Database                       |
| flask     | 5000  | API for collecting weather data |
| jupyter   | 8888  | Notebooks for analysis          |

## Using the Flask API

**Health check**
```bash
curl http://localhost:5000/health
```

**Collect weather data (default location from `.env`)**
```bash
curl -X POST http://localhost:5000/collect
```

**Collect weather data for a specific location**
```bash
curl -X POST http://localhost:5000/collect \
  -H "Content-Type: application/json" \
  -d '{"lat": 40.7128, "lon": -74.0060}'
```
> Note: `Content-Type: application/json` is required, or Flask will return a `415 Unsupported Media Type` error. The endpoint also uses `request.get_json(silent=True)` so a request with no body at all still works fine.

**View collected data**
```bash
curl http://localhost:5000/data
curl http://localhost:5000/data?limit=50
curl http://localhost:5000/data?city=London
```

**View stats**
```bash
curl http://localhost:5000/stats
```

## Building a Dataset

Run `/collect` repeatedly (ideally for multiple cities and over time) to build up enough rows for meaningful EDA and regression. A simple way to collect several cities at once is a small shell script:

```bash
#!/bin/bash
# collect_data.sh

curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d '{"lat": 51.5074, "lon": -0.1278}'   # London
curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d '{"lat": 40.7128, "lon": -74.0060}'  # New York
curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d '{"lat": 35.6762, "lon": 139.6503}'  # Tokyo
curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d '{"lat": -33.8688, "lon": 151.2093}' # Sydney
curl -X POST http://localhost:5000/collect -H "Content-Type: application/json" -d '{"lat": 48.8566, "lon": 2.3522}'    # Paris
```

```bash
chmod +x collect_data.sh
./collect_data.sh
```

## Using Jupyter

1. Open http://localhost:8888 in your browser (runs with no token/password for local dev)
2. Create a notebook inside `notebooks/`
3. Connect to the database:

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine('postgresql://postgres:postgres@postgres:5432/weather_db')
df = pd.read_sql('SELECT * FROM weather_data', engine)

df.head()
df.info()
df.describe()
```

## Planned Analysis

- **EDA**: distributions, correlations between temperature/humidity/pressure/wind, weather condition breakdowns
- **Target variable**: `feels_like` temperature
- **Features**: `temperature`, `humidity`, `pressure`, `wind_speed`
- **Modeling**: linear regression → Ridge/Lasso → tree-based models, using scikit-learn pipelines
- **Evaluation**: RMSE, R², residual plots

## Stopping the Containers

```bash
# Stop containers, keep data
docker-compose down

# Stop containers and delete database data
docker-compose down -v
```

## Notes on Docker Choices

- Both `flask` and `jupyter` build from the same `Dockerfile` to keep things simple.
- `--allow-root` is used for Jupyter because containers run as root by default; this is fine for local development but wouldn't be used in a production setup.
- `--no-browser` just prevents Jupyter from trying to open a browser inside the container — you still access it from a browser on your own machine at `http://localhost:8888`.