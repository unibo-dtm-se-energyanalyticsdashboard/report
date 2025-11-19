---
title: Release
has_children: false
nav_order: 7
---

# Release

## Artefacts and Repositories

The project produces **one primary software artefact**: the Python package named **`edas`** (`unibo-dtm-se-energyanalyticsdashboard`).

This single artefact is built into two standard distribution formats and is configured to be released to the official Python repositories:

* **Artefact Types (Quantity: 1 package, 2 formats):**
    1.  **Python Wheel (`.whl` file):** A binary, pre-built distribution for fast installation by end-users.
    2.  **Source Archive (`.tar.gz` file):** A source distribution (sdist).
* **Release Repositories:**
    1.  **PyPI / TestPyPI:** The final destination for the package, making it globally installable via `pip`. TestPyPI is used for dry-run testing before deployment to the official PyPI.
    2.  **GitHub Releases:** The CI/CD pipeline uploads the compiled artefact files (`.whl` and `.tar.gz`) directly to the GitHub Releases page, creating a versioned history of deliverables.

## Release Process

The release process is **100% fully automated** using the Node.js tool **`semantic-release`** (configured in `release.config.mjs`) and executed by the **`deploy.yml` GitHub Actions workflow**. This automation ensures consistency and adherence to versioning rules.

* **Trigger:** The process is triggered automatically upon a push or merge into the `master` branch.
* **Configuration Steps & Commands:**
    1.  **Semantic Analysis:** `semantic-release` analyzes the commit history on the main branch for **Conventional Commit** messages.
    2.  **Version Bump:** It calculates the next version number (SemVer) and updates the version in `pyproject.toml` (running the command `poetry version $nextVersion`).
    3.  **Artefact Build:** It builds the package formats using `poetry build`.
    4.  **Publish:** It runs the publish command, using secrets:
        ```bash
        poetry publish --build --username __token__ --password ${{ secrets.PYPI_PASSWORD }}
        ```
    5.  **Post-Release:** It creates the Git Tag (`vX.Y.Z`), generates the **`CHANGELOG.md`** file, and commits these changes back to the repository using the authorized GitHub Token.

## Choice of the Versioning Schema

The project uses **Semantic Versioning (SemVer)** (format: `MAJOR.MINOR.PATCH`).

* **Justification:** SemVer provides a clear contract for consumers, communicating the scope and potential impact of changes between versions.
* **Versioning Mechanism:** The `semantic-release` tool dictates when to increment each segment:
    * **PATCH:** Incremented by `fix:` or `docs:` commits (backward-compatible fixes).
    * **MINOR:** Incremented by `feat:` commits (new, backward-compatible features).
    * **MAJOR:** Incremented by any commit containing `BREAKING CHANGE:` in the body or footer (incompatible changes).
* **Artefact Alignment:** All generated artefacts (the Python package files, the GitHub Tag, and the `CHANGELOG.md` entry) share the exact **same version number** to ensure consistency across the release lifecycle.

## Choice of the License

* **License Chosen:** **Apache License 2.0** (full text in the `LICENSE` file and declared in `pyproject.toml`).
* **Justification:** This choice is made because it is a **permissive** (non-copyleft) license that is compatible with the licenses of our main dependencies (`pandas`, `dash`, etc.). Crucially, it allows for widespread use while including an explicit **Patent Grant** and protecting against patent claims, making it suitable for a data analytics tool.
