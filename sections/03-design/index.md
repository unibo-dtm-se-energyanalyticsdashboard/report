---
title: Design
has_children: false
nav_order: 4
---

# Design

### Architecture

#### Architectural Style

The architectural style chosen for this project is the **Hexagonal Architecture** (also known as **Ports & Adapters**), as discussed in the course slides (`[Unit-03.1]`).

**Justification (Why):**
The primary goal of this project is to create a system that is decoupled from its external infrastructure (like the specific ENTSO-E API or the PostgreSQL database). A Hexagonal Architecture was chosen because it achieves this decoupling, which directly satisfies our key non-functional requirements:

1.  **Testability (NFR6):** It allows us to test the core logic (`pipeline.py`) in isolation by mocking the database (`upsert.py`) and the API (`entsoe_client.py`), as demonstrated in `tests/test_pipeline_smoke.py`.
2.  **Maintainability (NFR5):** If the external API changes, we only need to update the `entsoe_client.py` adapter, without touching the core `pipeline.py` logic.

#### High-Level Architecture Overview

The system is designed with a clear separation between the "Core Application" and the "Adapters" that interact with the outside world.

* **Core Application (The "Hexagon"):** This is the business logic, implemented in `src/edas/pipeline.py`. It acts as an **Application Service** that orchestrates the entire data flow.
* **Ports (The "Interfaces"):** These are the contracts. In our Python implementation, they are *implicitly* defined by the function signatures that the core pipeline expects:
    * An "Energy Client Port" (expecting `fetch_consumption`, `fetch_production`).
    * An "Energy Repository Port" (expecting `upsert_energy_consumption`, `upsert_cross_border_flow`).
* **Adapters (The "Implementation"):** These are the concrete tools that implement the ports.
    * **Primary Adapters (Drivers):** `src/edas/cli.py` (CLI) and `src/edas/dashboard/app.py` (Web UI). They *drive* the application.
    * **Secondary Adapters (Driven):** `src/edas/ingestion/entsoe_client.py` (the ENTSO-E API adapter) and `src/edas/ingestion/upsert.py` (the PostgreSQL database adapter). They are *driven by* the application.

---

### Infrastructure

The system is composed of the following infrastructural components:

1.  **Clients:**
    * A **CLI Client** (`edas-ingest`) for the Data Engineer.
    * A **Web Browser** for the Energy Analyst.
2.  **Application Servers:**
    * A **Python Server (Dash)** (`edas-dashboard`) which runs `app.py` to serve the interactive web dashboard.
3.  **Databases:**
    * A **PostgreSQL Database** serves as the single source of truth for all time-series data.
4.  **External Services:**
    * The **ENTSO-E API**, which is the external data source.

#### Distribution and Naming

* **Distribution:** In a production environment, the Python Application (Dash) and the PostgreSQL database would run on separate, dedicated servers within the same private network (e.g., in the cloud). The ENTSO-E API is an external service located on the internet.
* **Naming/Discovery:** Components (like the database) are found via environment variables (e.g., `PGHOST`, `PGPORT`) which are loaded by `config.py` and `connection.py` from the `.env` file.

---

### Modelling

#### Domain-Driven Design (DDD) Modelling

* **Bounded Contexts:** The system is split into two distinct Bounded Contexts that only communicate via the database:
    1.  **Ingestion Context:** (Write-only) Responsible for fetching, transforming, and persisting data. Its Ubiquitous Language includes terms like `fetch_production`, `upsert`, `zone_key`, `to_utc_naive`.
    2.  **Analytics Context:** (Read-only) Responsible for querying, aggregating, and visualizing data. Its Language includes `KPIs`, `production_mix`, `net_balance`, `daily_summary`.
* **Domain Concepts (Entities, Repositories, Services):**
    * **Entities:** The core domain entities are implicitly modeled by the `pandas.DataFrame` structures and explicitly defined by the PostgreSQL schema (`01_schema.sql`): `countries`, `energy_consumption`, `energy_production`, `cross_border_flow`.
    * **Value Objects:** Concepts like `country_code`, `time_stamp`, and `source_type` act as Value Objects that define the identity of the data.
    * **Repositories:** The **Repository Pattern** is explicitly implemented in `src/edas/ingestion/upsert.py`. This module encapsulates all database write logic, particularly the `INSERT ON CONFLICT` (Upsert) strategy, hiding it from the pipeline.
    * **Application Services:** The `src/edas/pipeline.py` file acts as the Application Service, orchestrating calls to the adapters (Client and Repository).

#### Object-Oriented Modelling (UML Class Diagram)

<img width="1818" height="1504" alt="Untitled diagram-2025-11-16-102830" src="https://github.com/user-attachments/assets/04a709e5-5f27-40bb-8354-d036798fe55b" />


This diagram shows the main data entities (based on `01_schema.sql`) and their "1-to-N" relationships.

---

### Interaction

* **Component Communication:** The two Bounded Contexts (Ingestion and Analytics) are fully decoupled. They do not communicate directly.
    * The **Ingestion Context** (`pipeline.py`) **writes** to the PostgreSQL database.
    * The **Analytics Context** (`dashboard/queries.py`) **reads** from the PostgreSQL database.
    This asynchronous, database-centric communication pattern ensures high availability; the dashboard can still serve data even if the ingestion pipeline is temporarily down.

* **Interaction Patterns (UML Sequence Diagrams):**
* 
<img width="3693" height="2986" alt="Untitled diagram-2025-11-16-112431" src="https://github.com/user-attachments/assets/853c6eff-1c9c-450d-99ad-8831e0c841dd" />


    **Dashboard (`edas-dashboard`):** This diagram shows the flow when the Energy Analyst loads the dashboard or changes a filter.

  <img width="2631" height="1906" alt="Untitled diagram-2025-11-16-112644" src="https://github.com/user-attachments/assets/ec3318ba-eae3-41df-af29-33f1ed239347" />
   
---

### Behaviour

* **Component Behaviour:**
    * The **Ingestion Pipeline** (`pipeline.py`) is **stateful during execution** (it loads metadata, computes a date range, and fetches data) but **stateless between runs**.
    * The **Dashboard** (`app.py`) is **stateless**. It relies on Dash callbacks, which re-query the database (`queries.py`) every time a user interaction (like changing a filter) occurs.
* **State Update:**
    * The **State** of the system is the data stored in the PostgreSQL database.
    * Only the **Ingestion Context** (specifically `upsert.py`) is allowed to **write or modify** this state.
    * The **Analytics Context** (`queries.py`) is strictly **Read-Only**.

---

### Data-related Aspects

* **Data Stored:** Yes. Time-series data for energy consumption, production by source, and cross-border flows, as well as metadata for countries (`countries` table).
* **Storage Model:** A **Relational (SQL)** model was chosen (PostgreSQL).
    * **Justification:** The data is highly structured, tabular (time-series), and the relationships between tables (e.g., `energy_consumption` -> `countries`) are critical. A relational database is ideal for the complex `JOIN` and `GROUP BY` aggregations required by the `queries.py` analytics module.
* **Query Responsibility:**
    * **Writes (UPSERT):** `src/edas/ingestion/upsert.py`.
    * **Reads (SELECT):** `src/edas/dashboard/queries.py` (for analytics) and `src/edas/pipeline.py` (for metadata).
* **Concurrency:**
    * **Concurrent Writes:** Handled atomically by the `INSERT ON CONFLICT` (Upsert) mechanism in `upsert.py`. This ensures that even if two pipelines run simultaneously, data integrity is maintained (Idempotency, **NFR1**).
    * **Concurrent Reads:** Handled by PostgreSQL's standard MVCC (Multiversion Concurrency Control). Analysts reading the dashboard will not block the ingestion pipeline.
- Are there **infrastructural components** that need to be introduced? Which and **how many** of each?
    - e.g. **clients**, **servers**, **load balancers**, **caches**, **databases**, **message brokers**, **queues**, **workers**, **proxies**, **firewalls**, **CDNs**, etc.
- How do components **distribute** over the network? **Where** are they located?
    - e.g. do servers / brokers / databases / etc. sit on the same machine? on the same network? on the same datacenter? on the same continent?
- How do components **find** each other?
    - How to **name** components?
    - e.g. **DNS**, **service discovery**, **load balancing**, etc.

> UML deployment diagrams are welcome here

## Modelling

### Domain driven design (DDD) modelling

- Which are the bounded contexts of your domain? 
- Which are domain concepts (entities, value objects, aggregates, etc.) for each context?
- Are there repositories, services, or factories for each/any domain concept?
- What are the relavant domain events in each context?

> Context map diagrams are welcome here

### Object-oriented modelling

- What are the main data types (e.g. classes) of the system?
- What are the main attributes and methods of each data type?
- How do data types relate to each other?

> UML class diagrams are welcome here

### In case of a distributed system

- How do the domain concepts map to the architectural or infrastuctural components?
    + i.e. which architectural/component is responsible for which domain concept?
    + are there data types which are required onto multiple components? (e.g. messages being exchanged between components)

- What are the domain concepts or data types which represent the state of the distributed system?
    + e.g. state of a video game on central server, while inputs/representations on clients
    + e.g. where to store messages in an instant-messaging app? for how long?

- Are there domain concepts or data types which represent messages being exchanged between components?
    + e.g. messages between clients and servers, messages between servers, messages between clients

## Interaction

- How do components *communicate*? *When*? *What*?

- Which **interaction patterns** do they enact?

> UML sequence diagrams are welcome here

## Behaviour

- How does **each** component *behave* individually (e.g., in *response* to *events* or messages)?
    + Some components may be *stateful*, others *stateless*

- Which components are in charge of updating the **state** of the system? *When*? *How*?

> UML state diagrams or activity diagrams are welcome here

## Data-related aspects (in case persistent storage is needed)

- Is there any data that needs to be stored?
    - *What* data? *Where*? *Why*?

- How should **persistent data** be **stored**? Why?
    - e.g., relations, documents, key-value, graph, etc.

- Which components perform queries on the database?
    - *When*? *Which* queries? *Why*?
    - Concurrent read? Concurrent write? Why?

- Is there any data that needs to be shared between components?
    - *Why*? *What* data?

