---
title: Validation
has_children: false
nav_order: 6
---

# Validation

### Testing Approach

My testing strategy was not a strict "Test-Driven Development (TDD)" approach, as the core features were implemented first. Instead, I followed a **Test-After Development** methodology, which is crucial for validation and ensuring long-term maintainability.

The goal was to create a "safety net" of tests that validate the system at different levels, allowing for safe refactoring and future development.

**Testing Frameworks Used:**
* **`unittest`:** I used Python's built-in `unittest` library as the base framework for writing all my test cases (e.g., `TestPipelineSmoke(unittest.TestCase)`). It is robust and requires no external dependencies for basic tests.
* **`pytest`:** I used `pytest` (defined in `pyproject.toml`) as the primary test runner, executed via the `poe test` task. `pytest` has a superior test discovery mechanism and integrates seamlessly with plugins like `coverage`.
* **`unittest.mock`:** This was essential for my Integration tests to create **Test Doubles (Mocks)**.

---

### Testing (Automated)

My automated testing strategy is divided into three distinct levels, as defined in the `tests/` directory and executed by the `check.yml` CI pipeline.

#### 1. Unit Testing

* **Description:** This level tests individual functions or "units" of logic in complete isolation from the rest of the system (like databases or APIs).
* **Implementation (`tests/test_pipeline_unit.py`):**
    * **Rationale:** The `_compute_range` function in `pipeline.py` contains critical, time-sensitive business logic for defining the ingestion window. A bug here could lead to missing data or incorrect API calls.
    * **Test Cases:**
        * `test_last_10_days_dynamic_now`: Verifies the default (`last_10_days`) mode correctly calculates a dynamic 10-day window relative to the current time. (Covers **US2**, **FR1**).
        * `test_full_2025_range`: Verifies the project-specific (`full_2025`) mode returns the exact, hardcoded start and end timestamps. (Covers **US1**, **FR1**).
        * `test_invalid_mode_raises`: Verifies that the function fails fast by raising a `ValueError` if an unsupported mode is supplied. (Covers **NFR3**).
* **Future Work (Missing Tests):** A more comprehensive unit test suite would also include isolated tests for:
    * `entsoe_client.py` -> `to_utc_naive()` to validate timestamp conversions. (Covers **FR4**)
    * `entsoe_client.py` -> `_flatten_columns()` to validate the DataFrame transformation logic. (Covers **FR2**)
    * `upsert.py` -> Mocking the `raw_conn` and `pg_extras` to verify that the correct SQL and data tuples are generated, without touching a database. (Covers **FR5**)

#### 2. Integration Testing

* **Description:** This level tests the *orchestration* of multiple components working together, as defined in our Hexagonal Architecture (Design section). We test that the "Application Service" (`pipeline.py`) correctly interacts with its "Ports" (the adapters).
* **Implementation (`tests/test_pipeline_smoke.py`):**
    * **Rationale:** This test validates the *entire* `run_pipeline` function (our Application Service) without the cost or instability of real network calls or a live database.
    * **Test Doubles (Mocks):** This test relies heavily on **Test Doubles** (specifically, **Mocks** using `unittest.mock.patch`). We "patch out" (replace) all external infrastructure:
        * `@patch("edas.pipeline.get_engine")`: Replaces the real database engine with a **Fake Engine** (`_FakeEngine`).
        * `@patch("edas.pipeline._load_countries")`: Replaces the database call with a hardcoded dictionary.
        * `@patch("edas.pipeline.fetch_consumption")`: Replaces the ENTSO-E API call with a simple function returning a dummy DataFrame.
        * `@patch("edas.pipeline.upsert_energy_consumption")`: Replaces the database write function.
    * **Acceptance Criteria:** The test passes if `run_pipeline` (in a minimal 'FR' only, no-flows configuration) calls all the mocked functions exactly once (`.assert_called_once()`). This proves the orchestration logic is correct. (Covers **FR1, FR2, FR5**).

#### 3. System Testing (Smoke Tests)

* **Description:** This level tests the system as a whole to ensure it initializes without errors. This is the highest level test and matches our "Definition of Done" for basic functionality.
* **Implementation:**
    * **`tests/test_pipeline_smoke.py`:** This test (described above) also serves as a *System Test* for the entire Ingestion Bounded Context (`edas-ingest`).
    * **`tests/test_dashboard_smoke.py`:**
        * **Rationale:** This test ensures the Dash application (our Analytics Bounded Context) can be imported and its layout can be built without errors. It smoke-tests all imports and component initializations in `app.py` and `queries.py`.
        * **Acceptance Criteria:** The test passes if `from edas.dashboard.app import app` runs successfully and `app.layout` is not `None`. (Covers **FR6, FR7, FR8**).

#### Test Results (Success Rate & Coverage)

* **Success Rate:** 100% (All automated tests pass in the `check.yml` CI pipeline).
* **Test Coverage:**
    *(TODO: Run `poetry run poe coverage-report` and paste the resulting table here. This is a hard requirement from the checklist.)*

    ```
    Name                              Stmts   Miss  Cover   Missing
    ---------------------------------------------------------------
    src/edas/__init__.py                  ...
    src/edas/cli.py                      ...
    ...
    ---------------------------------------------------------------
    TOTAL                               ...         XX%
    ```

---

### Acceptance Tests (Manual)

In addition to automated tests, manual acceptance tests were performed to validate the end-to-end user experience, matching the acceptance criteria in the **Requirements** section.

* **Test Plan (Data Engineer - US1 & US2):**
    1.  **Setup:** Run `poetry run bash scripts/db_init.sh`. (Verify: `DB initialized...` message appears).
    2.  **Test (FR5 - Upsert):** Run `poetry run edas-ingest --mode full_2025`. (Verify: Log output shows `Upserted...` rows).
    3.  **Test (NFR1 - Idempotency):** Run `poetry run edas-ingest --mode full_2025` a *second time*. (Verify: Log output shows `Upserted...` rows, and the database row count does not double).
* **Test Plan (Energy Analyst - US3, US4, US5):**
    1.  **Setup:** Run `poetry run edas-dashboard`. (Verify: Server starts on `http://127.0.0.1:8050`).
    2.  **Test (FR6 - Filters):** Open the URL. Change the Date Range and add 'DE' to the Country filter. (Verify: The charts and KPIs update).
    3.  **Test (FR7 - KPIs):** Check the "Total Consumption" KPI card. (Verify: A numerical value is displayed).
    4.  **Test (FR8 - Visuals):** Click the "Production Mix" tab. (Verify: The stacked area chart loads correctly).
* **Success Rate:** 100% (All manual test steps passed as expected).
