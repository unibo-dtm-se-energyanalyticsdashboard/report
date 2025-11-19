---
title: Concept
has_children: false
nav_order: 2
---

# Concept

## Product Type

The project developed is a **Data Processing Toolkit and Analytics Application**. It is not a single, monolithic program but rather a system composed of two distinct components designed for two different user roles:

* **A Command-Line Interface (CLI) Tool:** (`edas-ingest`) A Python script responsible for the complete data ingestion (**ETL**) pipeline. This component acts as the "**backend**" of the system.
* **A Web Application:** (`edas-dashboard`) An interactive dashboard built with Dash and Plotly that serves as the "**frontend**" for data visualization and analytics.

---

## User Roles and Interaction

There are two primary user roles for this system, each interacting with a different component.

### The Data Engineer (Admin Role)

* **Where are they?** The Data Engineer operates from a **server environment** or their local development machine.
* **How do they interact?** They interact with the system via a **terminal (CLI)** using the Poetry scripts defined in `pyproject.toml` (e.g., `poetry run edas-ingest`).
* **When do they interact?** Interaction is **periodic and automated**.
    * **Initial Load:** There is an initial, manual run to populate the database with a large historical dataset (e.g., running the pipeline in `full_2025` mode).
    * **Daily Job:** After the initial load, the plan is for an automated scheduler (like a cron job) to run the `edas-ingest` script daily in its default mode (`last_10_days`) to fetch incremental updates.

### The Energy Analyst (Business User Role)

* **Where are they?** The Analyst accesses the system from their **corporate network**.
* **How do they interact?** They interact with the system using a **standard web browser** (e.g., Chrome, Firefox) on their desktop or laptop computer.
* **When do they interact?** Interaction is **on-demand**, whenever they need to analyze the energy data, check **KPIs**, or identify trends. They interact visually with the Dash interface by selecting countries from a dropdown and picking dates from a `DatePickerRange`.

---

## Data Storage

The system’s primary purpose is to **store the domain data** it processes, not user-specific data (like user accounts or profiles).

* **Which data?** The system stores **time-series data** for:
    * Energy Consumption (**Total Load**)
    * Energy Production (broken down by source type, e.g., **'Nuclear', 'Solar'**)
    * Cross-Border Flows (**net flow** between countries)
* **Where?** All this data is stored in a central **PostgreSQL database**. The structure (**schema**) for this data is defined in the `sql/01_schema.sql` file.
