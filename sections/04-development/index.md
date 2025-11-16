---
title: Development
has_children: false
nav_order: 5
---

# Development

### DVCS (Distributed Version Control)

This project was managed entirely using Git and GitHub, adhering to modern DVCS conventions as required by the course slides (`03-dvcs-basics.pdf`, `06-versioning.pdf`) and the evaluation checklist.

* **Branching Convention:** We adopted a simplified **GitFlow** model to manage development:
    * `master` (or `main`): This branch is kept stable and protected. All production releases (e.g., `v0.2.0`) are tagged and merged into this branch.
    * `dev`: This branch serves as the main integration branch for all new features.
    * `feature/*` (e.g., `feature/tests-logging`, `feature/document`): All new work, bug fixes, or documentation changes were developed in these isolated branches before being merged into `dev` via Pull Requests.
* **Commit Convention:** We strictly follow the **Conventional Commits** standard (e.g., `feat:`, `fix:`, `docs:`, `chore(release):`).
    * **Justification:** This standard is essential as it is directly used by our `semantic-release` tool (configured in `release.config.mjs`) to automatically determine the next version number (SemVer) and automatically generate the `CHANGELOG.md`.
* **Pull Requests (PRs) & Code Reviews:** All `feature` branches were merged into `dev` (and `dev` into `master`) via Pull Requests on GitHub.
    * **Merge Strategy:** We adopted the **"Squash and merge"** strategy. This convention is critical: it ensures that a single, meaningful Conventional Commit message enters the main branch, which in turn allows `semantic-release` to function correctly and maintain a clean, readable history.

---

### Implementation Details

#### Network Protocols

1.  **HTTP (Hypertext Transfer Protocol):**
    * **Usage:** This protocol is used for all communication with web-based services.
    * **Client (Ingestion):** The `entsoe_client.py` adapter uses HTTP (via the `entsoe-py` library) to send GET requests to the external ENTSO-E API.
    * **Server (Dashboard):** The `dashboard/app.py` (Dash) server runs on HTTP, allowing analysts to access the UI in their web browser (`http://127.0.0.1:8050`).
    * **Justification:** HTTP is the standard, ubiquitous, and stateless protocol for web APIs and web applications.

2.  **TCP (Transmission Control Protocol):**
    * **Usage:** This is the underlying protocol used for the stateful, persistent database connection.
    * **Client:** The `psycopg2` driver (managed by `SQLAlchemy` in `db/connection.py`) opens a TCP connection to the PostgreSQL server (e.g., on port 5432).
    * **Justification:** TCP guarantees reliable, in-order delivery of SQL queries and results, which is essential for database integrity.

#### Data Representation

* **In-Transit (API):** Data from the ENTSO-E API is received as **JSON** (or XML, which is handled transparently by the `entsoe-py` library).
* **In-Memory (Python):** This raw data is immediately transformed into **`pandas.DataFrame`** objects within the `entsoe_client.py` adapter.
    * **Justification:** DataFrames are the industry standard for data manipulation in Python. They are highly efficient for the cleaning and time-series transformations we need (e.g., `to_utc_naive`) before converting the data for database insertion (`upsert.py`).
* **In-Database (Storage):** Data is stored in a structured **SQL** format (e.g., `VARCHAR`, `TIMESTAMP`, `DOUBLE PRECISION`) as defined in our `sql/01_schema.sql`.

#### Database Querying

* **SQL (Structured Query Language):** We use raw SQL for all database interactions.
    * **Justification:** As established in the Design section, our data is highly structured and relational. Raw SQL (executed via `SQLAlchemy`'s `text()` and `psycopg2`'s `execute_values`) provides the highest performance and is the only appropriate way to execute the complex `JOIN`/`GROUP BY` aggregations (in `queries.py`) and the critical `INSERT ... ON CONFLICT DO UPDATE` (Upsert) operations (in `upsert.py`).

#### Authentication & Authorization

* **Authentication:** Authentication is **Token-Based**.
    * **Usage:** The ENTSO-E API requires an API key for authentication.
    * **Implementation:** The key is securely loaded from `.env` files (via `python-dotenv` in `config.py`) into an environment variable (`ENTSOE_API_KEY`) and passed to the `EntsoePandasClient`. The `.env` file is explicitly listed in `.gitignore` to prevent leaking secrets.
* **Authorization:** This system does not manage its own user accounts or roles (e.g., no login/password for the dashboard). Authorization is handled at the infrastructure level (e.g., access to the CLI, possession of database credentials, or network access to the dashboard URL).

---

### Technological Details

#### Core Technologies

* **Python (3.10+):** The core programming language used for all backend, ingestion, and analytics logic.
* **Poetry:** Used for dependency management, build automation (`pyproject.toml`), and script execution (`poe.tasks`).
* **PostgreSQL:** The relational database used for data storage.
* **Dash & Plotly:** Frameworks used for the interactive web dashboard (`app.py`).

#### Key Libraries (Dependencies)

* **`entsoe-py`:** The critical adapter library for communicating with the ENTSO-E API.
* **`SQLAlchemy`:** Used to create the database engine (`connection.py`), providing a robust connection pool.
* **`psycopg2-binary`:** The low-level driver for PostgreSQL, which enables the high-performance `execute_values` batch function used in `upsert.py`.
* **`pandas`:** Used for all in-memory data transformation and cleaning.
* **`pytest` & `coverage`:** Used for the Validation (Testing) framework.
* **`mypy`:** Used for static type checking (CI/CD Quality Gate).

#### External Technology Dependencies

* **ENTSO-E API:** The external web service that provides the raw energy data.
* **GitHub Actions:** The CI/CD platform used to run `check.yml` (testing) and `deploy.yml` (release).
* **PyPI / TestPyPI:** The software repositories where the final Python package is published.
* **Node.js Tools (`semantic-release`, `renovate`):** These JavaScript-based tools are external dependencies used in our DevOps workflow to automate versioning and dependency updates.
