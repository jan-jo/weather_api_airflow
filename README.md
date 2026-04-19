# ⛅ Weather ETL Pipeline — OpenWeatherMap + Airflow + AWS S3

An end-to-end ETL pipeline that fetches live weather data from the **OpenWeatherMap API**, orchestrates the workflow using **Apache Airflow** on an **AWS EC2** instance, and stores the processed output as a timestamped **CSV file in an AWS S3 bucket**.

---

##  Architecture

```
OpenWeatherMap API
        │
        ▼
  HttpSensor (is_weather_api_ready)
        │  checks API is live before proceeding
        ▼
  SimpleHttpOperator (extract_weather_data)
        │  fetches raw JSON weather data via GET request
        ▼
  PythonOperator (transform_load_weather_data)
        │  cleans & transforms data, converts Kelvin → Fahrenheit
        ▼
  AWS S3 Bucket
        └─ s3://weatherapiairflowyoutubebucket-yml/current_weather_data_portland_<timestamp>.csv
```

---

## Pipeline Steps (DAG Tasks)

| Task ID | Operator | Description |
|---|---|---|
| `is_weather_api_ready` | `HttpSensor` | Polls the OpenWeatherMap endpoint and waits until it's reachable before proceeding |
| `extract_weather_data` | `SimpleHttpOperator` | Makes a `GET` request to fetch current weather JSON for Portland |
| `transform_load_weather_data` | `PythonOperator` | Parses JSON, converts temperatures Kelvin → Fahrenheit, builds a DataFrame, writes CSV to S3 |

**Schedule:** `@daily` — runs once per day, no backfill (`catchup=False`)

---

## Output CSV Schema

Each pipeline run produces one CSV row:

| Column | Description |
|---|---|
| `City` | City name from API response |
| `Description` | Weather condition (e.g., *light rain*) |
| `Temperature (F)` | Current temperature in Fahrenheit |
| `Feels Like (F)` | Feels-like temperature in Fahrenheit |
| `Minimun Temp (F)` | Minimum temperature in Fahrenheit |
| `Maximum Temp (F)` | Maximum temperature in Fahrenheit |
| `Pressure` | Atmospheric pressure in hPa |
| `Humidty` | Humidity percentage |
| `Wind Speed` | Wind speed in m/s |
| `Time of Record` | Local timestamp of the weather reading |
| `Sunrise (Local Time)` | Local sunrise time |
| `Sunset (Local Time)` | Local sunset time |

**S3 output file naming:**
```
s3://weatherapiairflowyoutubebucket-yml/current_weather_data_portland_DDMMYYYYHHMMSS.csv
```

---

##  Prerequisites

- Python 3.8+
- Apache Airflow 2.x
- AWS EC2 instance (Ubuntu 20.04+, t2.medium or higher recommended)
- AWS S3 bucket
- [OpenWeatherMap API key](https://openweathermap.org/api) 

---



## License

MIT License
