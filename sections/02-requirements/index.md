---
title: Requirements
has_children: false
nav_order: 3
---

# Requirements

### User Stories

To define the system's scope, we identified two main personas:

* **US1 (Data Engineer):** As a Data Engineer, I want to run an ingestion pipeline for both a full historical year (`full_2025` mode) and for daily incremental updates (`last_10_days` mode), so that the database is populated correctly and kept up-to-date.
* **US2 (Energy Analyst):** As an Energy Analyst, I want to view KPIs, visual charts (Time-Series, Mix), and data tables on an interactive web dashboard, so that I can analyze energy patterns.

### Requirements Analysis

Based on the user stories, we defined the following requirements.

#### Functional Requirements (FR)

* **FR1 (Ingestion):** The system must fetch Consumption, Production (by source), and Cross-Border Flow data from the ENTSO-E API.
    * **Acceptance Criteria:** Data is successfully fetched by `entsoe_client.py`.
* **FR2 (Transformation):** All timestamps must be normalized to a standard UTC-naive format.
    * **Acceptance Criteria:** The `to_utc_naive` function correctly handles conversions, as validated by `tests/test_pipeline_unit.py`.
* **FR3 (Storage):** The system must store ingested data in PostgreSQL using an "UPSERT" strategy (Update or Insert) to prevent duplicates.
    * **Acceptance Criteria:** Running the pipeline twice for the same time range does not create duplicate rows, as handled by `upsert.py` using `ON CONFLICT DO UPDATE`.
* **FR4 (Dashboard Filters):** The web dashboard must allow filtering by Country and Date Range.
    * **Acceptance Criteria:** Changing filters in `app.py` triggers the update callbacks.
* **FR5 (Dashboard Visuals):** The dashboard must display KPIs, a Consumption vs. Production chart, and a Production Mix chart.
    * **Acceptance Criteria:** The `queries.kpis` and `queries.consumption_vs_production` functions provide data, and the `render_tab` callback displays the correct Plotly figures.

#### Non-Functional Requirements (NFR)

* **NFR1 (Idempotency):** The ingestion pipeline must be safely re-runnable. (Met by **FR3**).
* **NFR2 (Performance):** Ingestion must use efficient batch database operations. (Met by `psycopg2.extras.execute_values` in `upsert.py`).
* **NFR3 (Reliability):** The pipeline must handle API errors gracefully (e.g., missing flow data). (Met by `try...except` in `fetch_flow`).
* **NFR4 (Configurability):** Secrets (API keys, DB passwords) must not be hardcoded. (Met by `config.py` using `.env` files).
* **NFR5 (Testability):** The system must be testable, including automated test coverage reports. (Met by `pytest`, `coverage`, and `test_pipeline_smoke.py`).

#### Implementation Requirements (IR)

* **IR1: Core Technology:** Must be Python (>=3.10) using **Poetry** (`pyproject.toml`).
* **IR2: Database:** Must use **PostgreSQL** (`01_schema.sql`).
* **IR3: Domain Tooling:** Must use **`entsoe-py`** (for API) and **`Dash`** (for Dashboard).
* **IR4: DevOps:** Must use **GitHub Actions** (`check.yml`, `deploy.yml`).
* **Justification:** These technologies were chosen by the project brief to ensure standardization and focus on modern data engineering practices.

---

### Glossary (Ubiquitous Language)

* **ENTSO-E:** The European Network of Transmission System Operators for Electricity (the data source).
* **Zone Key:** The ENTSO-E identifier for a country (e.g., `10YFR-RTE------C`).
* **Upsert:** The "Update or Insert" operation (`INSERT ... ON CONFLICT`), critical for **NFR1**.
* **Adapter:** A component connecting our application to external infrastructure (e.g., `entsoe_client.py` is the API Adapter; `upsert.py` is the DB Adapter).
* **Application Service:** The component that orchestrates adapters (e.g., `pipeline.py`).

---

### Use Case Diagram (UML)

This diagram shows the main actors and their interactions with the EDAS system.


<img width="1113" height="872" alt="Untitled diagram-2025-11-16-122513" src="https://github.com/user-attachments/assets/e6b4f283-c0f9-43c9-869c-a3742e43e0d1" />
