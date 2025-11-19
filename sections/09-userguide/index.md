---
title: User guide
has_children: false
nav_order: 10
---

# User Guide

## 1. Running the Data Ingestion Pipeline

The ingestion pipeline downloads electricity data from ENTSO-E and stores it in the PostgreSQL database.

### Available Modes

* **Last 10 Days (default)**
  Downloads recent hourly data for the selected countries.
* **Full 2025**
  Downloads the entire 2025 dataset (slow, large dataset).

### Basic Command

```bash
poetry run python artifact/scripts/ingest.py --countries FR DE
```

### Full-year mode

```bash
poetry run python artifact/scripts/ingest.py --mode full_2025 --countries FR DE
```

### Skip cross-border flows

```bash
poetry run python artifact/scripts/ingest.py --no-flows
```

After execution, the pipeline prints progress logs (consumption, production, flows) and writes them to `logs/app.log`.


## 2. Running the Dashboard

The dashboard provides the graphical interface for exploring the stored data.

### Start the dashboard

```bash
poetry run python -m src.edas.dashboard.app
```

### Access the interface

Open the browser and navigate to:

```
http://127.0.0.1:8050
```

### Dashboard Features

* **Country Selector** — choose one or multiple countries (FR, DE, …).
* **Date Range Picker** — select custom time windows.
* **KPIs Section (always visible)**

  * Total Consumption
  * Total Production
  * Net Balance
  * Average Daily / Weekly / Monthly Consumption
* **Tabs**

  * **Overview** — Time-series of consumption vs production
  * **Production Mix** — Area plot of generation by type
  * **Cross-Border Flows** — Imports/exports between neighboring countries
  * **Hourly Heatmap** — Average consumption by day of week × hour
  * **Tables** — Daily summaries and flow tables

Screens update automatically when filters change.


## 3. Configuration Required by the User

Users must ensure their `.env` file contains valid settings:

```env
DB_USER=postgres
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=energy_analytics
ENTSOE_API_KEY=your_api_key
```

### Where to place the `.env` file

At the project root (same level as `pyproject.toml`).

### Important

* The ENTSO-E API key must be valid.
* PostgreSQL must be running before ingestion or dashboard execution.


## 4. Log Files

The system writes logs to:

```
logs/app.log
```

The log file rotates automatically to avoid excessive size and provides:

* INFO logs during ingestion
* WARNING logs for API retries
* ERROR logs if ingestion fails


## 5. Typical User Workflow

1. Ensure PostgreSQL is running.
2. Ensure `.env` contains valid credentials + API key.
3. Run ingestion once:

   ```bash
   poetry run python artifact/scripts/ingest.py --countries FR DE
   ```
4. Start the dashboard:

   ```bash
   poetry run python -m src.edas.dashboard.app
   ```
5. Explore the charts and KPIs in the browser.


## 6. Error Handling and Troubleshooting

### Missing API key

You will see:

```
Missing required environment variable: ENTSOE_API_KEY
```

Fix by adding the key to `.env`.

### Database connection error

Ensure:

* PostgreSQL is running
* User/password in `.env` match your local DB

### No data shown on dashboard

Make sure ingestion ran successfully before launching the dashboard.
