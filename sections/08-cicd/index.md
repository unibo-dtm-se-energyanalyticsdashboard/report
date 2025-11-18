---
title: CI/CD
has_children: false
nav_order: 9
---

# CI/CD

The project uses an automated CI/CD pipeline based on **GitHub Actions** to guarantee code quality, reproducibility, and reliable deployment.

## **What is Automated?**

* Installation of dependencies using **Poetry**.
* Syntax validation and static type checking via `compileall` and **mypy**.
* Execution of all unit tests with coverage reporting.
* Multi-platform testing on Linux, macOS, and Windows.
* Testing across multiple Python versions (3.10–3.13).
* Automatic release generation (semantic versioning) and optional publishing to PyPI.

## **Why Automation?**

* Ensures consistent and reproducible builds independent of developer machines.
* Detects errors early and prevents merging broken code.
* Validates compatibility across different operating systems and Python versions.
* Improves security by preventing exposure of API keys or database credentials.
* Reduces manual work and eliminates human error in deployment and versioning.

## **How Does It Work?**

Workflows are defined in:

```
.github/workflows/check.yml
.github/workflows/deploy.yml
```

Each workflow includes:

* Code checkout
* Python & Poetry setup
* Virtual environment caching
* Dependency installation
* Static checks
* Unit tests & coverage
* Upload of coverage artifacts
* (Optional) semantic-release for automated tagging and release creation

## **Secrets and Environment Variables**

CI loads sensitive values using **GitHub Secrets**, never stored in the repository:

* `ENTSOE_API_KEY` – ENTSO-E API access
* `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_NAME` – database connection
* `PYPI_TOKEN` (optional) – for publishing releases
* `GITHUB_TOKEN` – auto-generated for release automation

A `.env.example` documents required variables for local development, ensuring a consistent configuration between local runs and CI.
