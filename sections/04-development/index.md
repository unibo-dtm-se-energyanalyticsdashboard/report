---
title: Development
has_children: false
nav_order: 5
---

# Development

DVCS (Distributed Version Control)

This project was managed entirely using Git and GitHub, adhering to modern DVCS conventions as required by the checklist and course slides (03-dvcs-basics.pdf, 06-versioning.pdf).

Branching Convention: We adopted a simplified GitFlow model.

master (or main): This branch is kept stable. All releases (e.g., v0.2.0) are tagged from this branch.

dev: This branch serves as the main integration branch for new features.

feature/* (e.g., feature/tests-logging, feature/document): All new work, bug fixes, or documentation changes were developed in isolated feature branches.

Commit Convention: We strictly follow the Conventional Commits standard (e.g., feat:, fix:, docs:, chore(release):).

Justification: This standard is essential as it is directly used by the semantic-release tool (configured in release.config.mjs) to automatically determine the next version number (SemVer) and generate the CHANGELOG.md.

Pull Requests (PRs) & Code Reviews: All feature branches were merged into dev (and dev into master) via Pull Requests on GitHub.

Merge Strategy: We use the "Squash and merge" strategy. This is a critical convention that ensures a single, meaningful Conventional Commit message enters the main branch, which in turn allows semantic-release to function correctly.

Implementation Details

Network Protocols

HTTP (Hypertext Transfer Protocol):

Usage: This protocol is used for all communication with external and internal web services.

Client (Ingestion): The entsoe_client.py adapter uses HTTP (via the entsoe-py library) to send GET requests to the external ENTSO-E API.

Server (Dashboard): The dashboard/app.py (Dash) server runs on HTTP, allowing analysts to access the UI in their web browser (http://127.0.0.1:8050).

Justification: HTTP is the standard, ubiquitous protocol for web APIs and web applications.

TCP (Transmission Control Protocol):

Usage: This is the underlying protocol used for the database connection.

Client: The psycopg2 driver (managed by SQLAlchemy in db/connection.py) opens a persistent TCP connection to the PostgreSQL server (e.g., on port 5432).

Justification: TCP guarantees reliable, in-order delivery of SQL queries and results, which is essential for database integrity.

Data Representation

In-Transit (API): Data from the ENTSO-E API is received as JSON (or XML, which is handled transparently by the entsoe-py library).

In-Memory (Python): This data is immediately transformed into pandas.DataFrame objects within the entsoe_client.py adapter.

Justification: DataFrames are the industry standard for data manipulation in Python. They allow for efficient cleaning, transformation (e.g., to_utc_naive), and handling of time-series data before it is converted for database insertion (upsert.py).

In-Database (Storage): Data is stored in a structured SQL format (e.g., VARCHAR, TIMESTAMP, DOUBLE PRECISION) as defined in sql/01_schema.sql.

Database Querying

SQL (Structured Query Language): We use raw SQL for all database interactions.

Justification: As established in the Design section, our data is highly structured and relational. Raw SQL (executed via SQLAlchemy's text() and psycopg2's execute_values) provides the highest performance and is the only appropriate way to execute the complex JOIN, GROUP BY aggregations (in queries.py) and the critical INSERT ... ON CONFLICT DO UPDATE (Upsert) operations (in upsert.py).

Authentication

API Key (Token-Based):

Usage: The ENTSO-E API requires an API key for authentication.

Implementation: The key is securely loaded from .env files (via python-dotenv in config.py) into an environment variable (ENTSOE_API_KEY) and passed to the EntsoePandasClient.

Security: The .env file is explicitly listed in .gitignore to prevent leaking secrets.

Authorization: This system does not manage user accounts (e.g., login/password for the dashboard). Authorization is handled at the infrastructure level (e.g., access to the CLI, access to the database credentials, or network access to the dashboard URL).

Technological Details

Core Technologies

Python: The core programming language used for all backend, ingestion, and analytics logic.

Poetry: Used for dependency management, build automation (pyproject.toml), and script execution (poe.tasks).

PostgreSQL: The relational database used for data storage.

Dash & Plotly: Frameworks used for the interactive web dashboard (app.py).

Key Libraries (Dependencies)

entsoe-py: (External Dependency) The critical adapter library for communicating with the ENTSO-E API.

SQLAlchemy: Used to create the database engine (connection.py), providing a robust connection pool.

psycopg2-binary: The low-level driver for PostgreSQL, enabling high-performance batch operations (execute_values in upsert.py).

pandas: Used for all in-memory data transformation and cleaning.

pytest & coverage: Used for the Validation (Testing) framework.

mypy: Used for static type checking (CI/CD Quality Gate).

semantic-release (JavaScript): (External Tool) Used for automated versioning and release, configured via release.config.mjs.

Renovate (JavaScript): (External Tool) Used for automated dependency updates, configured via renovate.json.

External Technology Dependencies

ENTSO-E API: The external web service that provides the raw energy data.

GitHub Actions: The CI/CD platform used to run check.yml (testing) and deploy.yml (release).

PyPI / TestPyPI: The software repositories where the final Python package is published.
