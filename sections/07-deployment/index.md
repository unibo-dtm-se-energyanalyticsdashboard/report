---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

## **User Installation**

### **Does the user need to install something?**

Yes. The user must install:

1. **Python (≥ 3.10)**
2. **Poetry** for dependency and environment management
3. Optional: **Git**, to clone the repository
4. Optional: **Docker**, if the user prefers containerized execution

### **Installation Steps**

#### **1. Install Python**

The software requires Python 3.10 or newer.
The official installer can be downloaded from:
[https://www.python.org/downloads/](https://www.python.org/downloads/)

Verify installation:

```bash
python --version
```

#### **2. Install Poetry**

Poetry manages dependencies and creates an isolated virtual environment.

```bash
pip install poetry
```

Verify:

```bash
poetry --version
```

#### **3. Install the Project Dependencies**

From the project root directory:

```bash
poetry install
```

This creates the virtual environment and installs all required libraries (pandas, SQLAlchemy, entsoe-py, etc.).

#### **4. Configure Environment Variables**

The user must create a `.env` file in the root directory:

```
ENTSOE_API_KEY=your_api_key_here

DB_USER=postgres
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=energy_analytics
```

A template `.env.example` is provided.

#### **5. Initialize the Database Schema**

The project requires a PostgreSQL database.

If PostgreSQL is already installed:

```bash
psql -U postgres -f sql/01_schema.sql
```

To run with Docker:

```bash
docker run --name energy-db -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 -d postgres
```

Then apply schema:

```bash
docker exec -i energy-db psql -U postgres < sql/01_schema.sql
```

#### **6. Run the Ingestion Pipeline**

To fetch ENTSO-E data:

```bash
poetry run python scripts/ingest.py --countries FR DE
```

#### **7. Run the Dashboard**

```bash
poetry run python -m edas.dashboard.app
```

---

## **Server-Side Installation**

### **Does the software need to be installed on a server?**

Yes, if the system is intended to run scheduled ingestion jobs or expose the dashboard to multiple users.

### **Server Requirements**

A server (Linux recommended) must have:

* Python (≥ 3.10)
* Poetry
* PostgreSQL server
* Cron or systemd for scheduled ingestion tasks
* Git (recommended)
* Optional: Docker or Kubernetes for containerized deployment

### **1. Install System Dependencies**

```bash
sudo apt update
sudo apt install python3 python3-pip postgresql git
pip install poetry
```

### **2. Clone the Repository**

```bash
git clone <repository-url>
cd artifact   # or your project folder
```

### **3. Install Software**

```bash
poetry install
```

### **4. Configure Server Environment Variables**

Set them permanently:

```bash
export ENTSOE_API_KEY="xxxx"
export DB_USER="postgres"
export DB_PASSWORD="xxxx"
export DB_HOST="localhost"
export DB_PORT="5432"
export DB_NAME="energy_analytics"
```

Or store them safely in:

```
/etc/environment
```

### **5. Configure PostgreSQL**

If using Docker:

```bash
docker run --name energy-db -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 -d postgres
```

Apply schema:

```bash
docker exec -i energy-db psql -U postgres < sql/01_schema.sql
```

### **6. Configure Scheduled Pipeline Execution**

Example cron job (runs ingestion every night at 03:00):

```bash
0 3 * * * cd /home/user/artifact && poetry run python scripts/ingest.py >> cron.log 2>&1
```

### **7. Deploy the Dashboard**

Run manually:

```bash
poetry run python -m edas.dashboard.app
```

Or use a systemd service for automatic startup.


## **Server-Side External Dependencies**

### **Database**

The system requires a **PostgreSQL** instance with the schema defined in `sql/01_schema.sql`.

### **External API**

The ingestion pipeline depends on the **ENTSO-E Transparency Platform API**.
A valid API key must be provided via environment variables.

