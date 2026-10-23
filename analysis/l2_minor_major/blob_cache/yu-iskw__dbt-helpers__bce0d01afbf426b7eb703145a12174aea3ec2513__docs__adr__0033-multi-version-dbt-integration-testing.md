# 33. Multi-Version dbt & Fusion Integration Testing

Date: 2026-02-17

## Status

Accepted

## Context

We need to ensure that `dbt-helpers` works correctly across multiple versions of dbt, including dbt Core (1.10, 1.11) and the new dbt Fusion engine. Currently, integration tests for warehouses (like DuckDB) run against a single version defined in the test environment or a hardcoded Dockerfile.

Using a single version is insufficient because:

- Different dbt versions might introduce breaking changes in YAML schemas or catalog behavior.
- dbt Fusion introduces significant performance benefits but also architectural changes (e.g., `.env` file support, different installation methods).
- We want to verify compatibility before users encounter issues.
- The current `unittest.TestCase` setup makes it difficult to parameterize tests across different environmental configurations (like dbt versions and flavors).

## Decision

We will implement a unified multi-version integration testing framework using `pytest` parameterization and Docker build arguments that supports both dbt Core and dbt Fusion.

1. **Parameterization Matrix**: Use `@pytest.mark.parametrize` to run integration tests against a matrix of dbt flavors and versions (e.g., `("core", "1.10")`, `("core", "1.11")`, `("fusion", "latest")`).
2. **Conditional Docker Installation**: Enhance the integration `Dockerfile` to accept `DBT_FLAVOR` and `DBT_VERSION` arguments.
   - For `core`, it will use `pip install dbt-duckdb`.
   - For `fusion`, it will use the official CDN shell script installer (`curl ... | sh`).
3. **Testcontainers Build Arguments**: Update the test runner (`conftest.py`) to pass these build arguments to the `DockerContainer` during image construction.
4. **Fusion Guardrails**: Acknowledge that some adapters (like DuckDB) may not yet be supported in dbt Fusion. Implement logic to skip or mark as expected failures these specific combinations until support is available.
5. **Fixture-based Setup**: Move away from `unittest.setUpClass` in favor of session-scoped `pytest` fixtures that handle the database generation for each flavor/version combination in the matrix.
6. **Orchestration with Nox**: Implement a `noxfile.py` to allow easy execution of specific matrix entries from the command line.

## Consequences

### Positive

- **Comprehensive Coverage**: Verifies compatibility across multiple dbt versions and flavors in a single test run.
- **Future-Proofing**: Provides a structure to easily add new dbt versions or flavors as they are released.
- **Improved Test Infrastructure**: `pytest` fixtures and parameterization provide cleaner resource management.
- **Reproducibility**: Docker-based execution ensures consistency.

### Negative

- **Increased Test Duration**: Running tests against a matrix increases total execution time.
- **Complexity**: Requires managing conditional installation logic in Docker and parameterized fixtures in Python.
- **Adapter Limitations**: We must explicitly handle cases where dbt Fusion does not yet support certain adapters.
