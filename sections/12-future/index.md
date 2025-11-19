---
title: Future work
has_children: false
nav_order: 13
---

# Known issues and future work

* **Limited test coverage:** Only smoke tests and minimal unit tests are implemented. Edge cases, failure modes, and dashboard logic are not fully tested.
* **API rate-limit sensitivity:** The ENTSO-E API occasionally returns connection reset errors when many requests are made in a short time. The retry logic works but is not always sufficient.
* **Timezone inconsistencies:** Although handled carefully, the ENTSO-E data contains inconsistent timestamps in rare cases, which may lead to small misalignments in visualizations.
* **Dashboard performance on large time ranges:** Plotly Dash becomes slower when loading large datasets (e.g., entire year). No caching layer is implemented yet.
* **Static type checking warnings:** Some Dash components trigger mypy warnings due to library-level typing issues.

---

## **Missing Features**

* **Authentication / user management:** The dashboard is public locally; no login or authorization mechanism is implemented.
* **Automated deployment:** Releases are automated, but the application itself is not deployed on a server (e.g., Docker, cloud hosting, or a managed service).
* **Comprehensive error reporting UI:** The dashboard does not show error messages or validation to the end user when data is missing.
* **Front-end polish:** Some visual elements are basic and could be refined (styling, animations, responsiveness).

---

## **Future Work**

* **Add full unit tests & integration tests** for ingestion, database operations, and dashboard callbacks.
* **Implement caching or incremental ingestion** to reduce API calls and improve performance.
* **Introduce Docker support** for easier deployment and reproducible environments.
* **Add automated dashboard deployment** (e.g., GitHub Pages for static parts, Heroku/Fly.io/Render for the app).
* **Improve UX/UI** with enhanced layouts, loading indicators, mobile responsiveness, and better error messages.
* **Add support for more countries** and cross-border flow visualizations using geographic maps.
* **Create REST APIs** for external clients to consume processed data.
* **Add configuration UI** allowing users to modify date ranges, data sources, and metrics directly in the dashboard.

