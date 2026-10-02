# ✈️ Flight Arrivals Data Pipeline: AeroDataBox API to MySQL

An educational data engineering project that finds the airports near Germany's five largest cities, pulls their **flight arrival data** from the [AeroDataBox API](https://rapidapi.com/aedbx-aedbx/api/aerodatabox), cleans it with **pandas**, and stores it in a relational **MySQL** database linked to the city tables.

**Cities covered:** Berlin, Hamburg, Frankfurt, Munich, Cologne

> 🔗 This project is the second part of my city data pipeline. The first part collects city facts and weather forecasts and creates the `city_list`, `fact_list` and `cities_forecast` tables that this notebook builds on.

---

## What this project does

1. **Reads city coordinates from MySQL** (`fact_list`): `city_id`, `latitude`, `longitude`.
2. **Finds nearby airports**: for each city, it calls the AeroDataBox *airport search by location* endpoint (50 km radius, up to 10 airports, only airports with flight info) and stores the IATA codes in `flight_iata`.
3. **Fetches arrivals**: for each airport in `flight_iata`, it requests that day's arrivals (00:00 to 11:59) and extracts the key fields: departure airport, scheduled arrival time and flight number.
4. **Cleans the data**: it removes duplicate rows, sorts by scheduled arrival time and converts timestamps to a proper datetime type.
5. **Loads the result into MySQL** in the `flight_arrivals` table, linked back to the city through `city_id`.

```
MySQL (fact_list: lat/lon)
        │
        ▼
AeroDataBox: airports within 50 km ──► MySQL (flight_iata)
                                              │
                                              ▼
AeroDataBox: arrivals per airport ──► pandas (clean) ──► MySQL (flight_arrivals)
```

The notebook also handles API limits: it retries once after a `429 Too Many Requests` response and pauses between calls so it stays within the free-tier limits.

---

## Repository contents

| File | Description |
|---|---|
| `Collecting_flight_data_cleaned.ipynb` | Jupyter notebook with the full workflow (airport search → arrivals → cleaning → SQL load) |
| `county_city_info_api_dump.sql` | MySQL dump of the `county_city_info_api` database (schema and data for all tables) |

---

## Database schema

Database name: `county_city_info_api`

| Table | Purpose | Key columns |
|---|---|---|
| `city_list` | One row per city | `city_id` (PK, auto-increment), `name` |
| `fact_list` | City facts | `city_id` (FK), `latitude`, `longitude`, `country`, `population` |
| `cities_forecast` | Weather forecast per city | `city_id` (FK), `forecast_time`, `temperature`, `forecast`, `wind_speed`, ... |
| `flight_iata` | Airports found near each city | `city_id` (FK), `iata` |
| `flight_arrivals` | Scheduled flight arrivals | `arrival_id` (PK), `city_id` (FK), `arrival_iata`, `departure_airport_iata`, `scheduled_arrival_time`, `flight_number` |

### Example query

```sql
USE county_city_info_api;

SELECT *
FROM flight_arrivals
WHERE city_id = 3;
```

---

## Getting started

### 1. Prerequisites

- Python 3.9+
- MySQL Server 8.0 (and optionally MySQL Workbench)
- A free [RapidAPI](https://rapidapi.com/) account subscribed to the [AeroDataBox API](https://rapidapi.com/aedbx-aedbx/api/aerodatabox)

### 2. Install dependencies

```bash
pip install pandas requests sqlalchemy pymysql beautifulsoup4 jupyter
```

### 3. Restore the database

```bash
mysql -u root -p < county_city_info_api_dump.sql
```

Or in MySQL Workbench: **Server → Data Import → Import from Self-Contained File**.

The dump already contains the city tables, so the notebook can run straight away.

### 4. Add your credentials

For security, no real keys or passwords are included in this repo. Before running the notebook, replace the placeholders with your own values:

| Placeholder | Replace with |
|---|---|
| `YOUR_RAPIDAPI_KEY` | Your RapidAPI key for AeroDataBox |
| `YOUR_DB_PASSWORD` | Your local MySQL password |

> 💡 **Tip:** a safer approach is to keep secrets in environment variables (e.g. `os.getenv("RAPIDAPI_KEY")`) and never commit them.

### 5. Run the notebook

```bash
jupyter notebook
```

Open `Collecting_flight_data.ipynb` and run the cells from top to bottom.

---

## Tech stack

- **Python**: pandas, requests, SQLAlchemy, PyMySQL
- **MySQL 8.0**
- **API**: AeroDataBox (via RapidAPI)
- **Jupyter Notebook**

---

## Known notes

- The notebook uses the current date (`datetime.now()`) and a fixed morning window (00:00 to 11:59), so each run collects a different snapshot of arrivals.
- To respect free-tier rate limits, the arrivals function waits 15 seconds between airports, so a full run takes a few minutes.
- Re-running the "push to SQL" cells appends rows again (`if_exists="append"`), which creates duplicates in the database. Clear the table first if you want to start fresh.
- Scheduled arrival times are converted to UTC before they are saved.

---

## Disclaimer

This project is for **educational purposes only**. Flight data belongs to AeroDataBox and its data providers. Please follow their terms of use and do not use this data for operational or commercial purposes.

---

## Author

**<Your Name>**
[LinkedIn](https://www.linkedin.com/in/tovhowani-kwinda-dr-rer-nat) · [GitHub](https://github.com/Kwindati)
