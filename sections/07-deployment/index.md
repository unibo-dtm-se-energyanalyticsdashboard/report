---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

### User Installation

This section describes the steps required for an end-user (e.g., an Energy Analyst) to install and run the `edas` package on their local machine.

*   **Software Requirements:**
    *   Yes, the user must install software. They need **Python (version 3.10 or higher)** and the **`pip`** package manager.
    *   They also need access to a **PostgreSQL** database instance (either locally or on a server).

*   **Installation Commands:**
    *   The user installs the application package directly from PyPI (or TestPyPI):
        ```bash
        pip install unibo-dtm-se-energyanalyticsdashboard
        ```

*   **Configuration:**
    *   Yes, the user must configure the application. They must create a `.env` file in their working directory.
    *   This file must contain the credentials for the database and the API key for the external service, as shown in the `.env.example` file:
        ```env
        # .env file
        DB_USER=postgres
        DB_PASSWORD=your_password
        DB_HOST=localhost
        DB_PORT=5432
        DB_NAME=energy_analytics
        ENTSOE_API_KEY=your_entsoe_token
        ```

### Server-Side Installation

This section describes the setup required on a server environment, typically managed by a Data Engineer, to run the automated ingestion pipeline and host the persistent database.

*   **Server Installation:**
    *   Yes, the system requires server-side installation for its core infrastructure (database) and for running the automated ingestion job.

*   **Further Software Required on Server:**
    *   **PostgreSQL Database:**
        *   **What:** A PostgreSQL server instance is required to store all energy time-series data.
        *   **Install:** This must be installed using the standard procedures for the server's operating system (e.g., `apt install postgresql`).
        *   **Configure:** The database must be configured with a user and password matching the environment variables (e.g., `PGUSER`, `PGPASSWORD`) and a database (e.g., `energy_analytics`) must be created.
    *   **Application Environment (Python & Poetry):**
        *   **What:** The server needs Python 3.10+ and Poetry to run the ingestion scripts reliably.
        *   **Install:**
            ```bash
            # 1. Install pip, poetry, and checkout the code
            pip install poetry
            git clone [your-repo-url]
            cd [your-repo-name]
            # 2. Install dependencies using the lock file for consistency
            poetry install
            ```
    *   **Database Schema:**
        *   **What:** The database tables must be created using the DDL file.
        *   **Install:** This is a one-time setup step executed via the provided bash script:
            ```bash
            poetry run bash scripts/db_init.sh
            ```
