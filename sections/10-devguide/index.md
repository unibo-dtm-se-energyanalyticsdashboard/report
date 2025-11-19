---
title: Developer guide
has_children: false
nav_order: 11
---

# Developer Guide

This section explains how a new contributor can join the project, understand its structure, and start contributing effectively.

---

## **1. Team & Communication**

* **Project Repository:** GitHub (inside the project organization).
* **Issue Reporting:**
  Use the GitHub *Issues* section to report bugs, request enhancements, or ask questions.
* **Internal Communication:**
  New contributors should contact the team via GitHub issues or the course communication channels suggested by the instructor.

---

## **2. Development Conventions**

### **Code Style**

* Python code follows **PEP8** and **PEP484 typing**.
* All modules must include:

  * meaningful function/method names
  * docstrings for public functions
  * English comments only
* Logging uses the central logging utility (`edas.logging_config`).

### **Branching & Naming**

* **Main branches:**

  * `master` → stable releases
  * `dev` → integration of feature branches
* **Feature branches naming:**

  ```
  feat/<short-description>
  fix/<issue>
  refactor/<module>
  docs/<section>
  ```
* **Commit messages:** follow **Conventional Commits**
  Examples:

  * `feat: add country selector to dashboard`
  * `fix: correct timezone conversion`
  * `docs: update user guide`

---

## **3. Development Environment Setup**

### **Clone the project**

```bash
git clone https://github.com/<org>/<repo>
cd <repo>
```

### **Install Poetry**

```bash
pip install poetry
```

### **Install dependencies**

```bash
poetry install
```

### **Environment Variables**

Create a `.env` file:

```bash
DB_USER=postgres
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=energy_analytics
ENTSOE_API_KEY=your_api_key
```

### **Database Setup**

```bash
bash artifact/scripts/db_init.sh
```

### **Run Tests**

```bash
poetry run poe test
```

### **Run Static Checks**

```bash
poetry run poe mypy
poetry run poe compile
```

---

## **4. Development Workflow**

### **Creating a New Feature**

1. Start from the `dev` branch:

   ```bash
   git checkout dev
   git pull
   ```
2. Create feature branch:

   ```bash
   git checkout -b feat/<feature-name>
   ```
3. Implement changes.
4. Run tests and static checks before commit:

   ```bash
   poetry run poe test
   poetry run poe mypy
   ```
5. Commit with Conventional Commit rules:

   ```bash
   git commit -m "feat: add new KPI"
   ```
6. Push the branch:

   ```bash
   git push -u origin feat/<feature-name>
   ```
7. Open a **Pull Request** into `dev`:

   * Include a description
   * Mention what was changed
   * Reference issues if required

### **PR Requirements**

* All CI checks must pass.
* Code must follow style conventions.
* At least one reviewer must approve the PR.

---

## **5. IDE & Tooling Instructions**

### **Recommended IDE**

* **Visual Studio Code** or **PyCharm**
* Enable:

  * Python extension
  * Pylance or MyPy support
  * Black or autopep8 for formatting

### **Recommended Tools**

* **Poetry**: dependency, virtualenv, and scripts
* **Poe the Poet**: task runner (`poetry run poe <task>`)
* **GitHub Actions**: CI pipeline
* **Docker (optional)**: for database or dashboard containerization

### **Useful Commands**

Run ingestion:

```bash
poetry run python artifact/scripts/ingest.py --countries FR DE
```

Run dashboard:

```bash
poetry run python -m src.edas.dashboard.app
```

Run coverage:

```bash
poetry run poe coverage
```

---

## **6. How to Extend the System**

### **Add a new data metric**

* Add query logic in `src/edas/dashboard/queries.py`
* Add UI logic in `src/edas/dashboard/app.py`

### **Add a new ingestion source**

* Implement new fetching logic in `edas.ingestion.entsoe_client`
* Add new upsert function in `edas.ingestion.upsert.py`
* Update pipeline logic in `edas.pipeline`

### **Add new dashboards**

* Add a new tab and callback in `src/edas/dashboard/app.py`

---

## **7. Release Workflow (for developers)**

1. Merge all feature branches into `dev`.
2. Ensure CI passes.
3. Merge `dev` → `master`.
4. GitHub Actions triggers **semantic-release**:

   * Bumps version
   * Updates changelog
   * Publishes release
