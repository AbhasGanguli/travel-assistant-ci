# Travel Assistant CI Controller

This public repository hosts a low-cost validation workflow for the application source kept in the private `travel-assistant` repository. It does not contain a mirror of the application code.

## Checks

The workflow checks out the exact private commit pinned by `requested-source.txt` onto a temporary GitHub-hosted runner and performs:

1. Kotlin debug compilation.
2. The JVM unit-test suite.
3. The checked-in Room schema drift guard.

APK assembly and artifact uploads are intentionally excluded. Architecture acceptance and device/APK testing remain separate gates.

## Request a validation run

Replace the single 40-character commit SHA in `requested-source.txt` with the private branch commit to validate, then commit the change to `main`. That file change starts the workflow. The workflow reports the source SHA so the result can be tied to the exact code revision.

## Access and privacy

- The `PRIVATE_REPO_READ_TOKEN` Actions secret should be a fine-grained token scoped only to the private application repository with **Contents: Read-only** permission.
- The private source is checked out to the ephemeral runner. Checkout credentials are not persisted for later build steps.
- No application source archive, APK, or raw test-report artifact is uploaded.
- Because this controller repository is public, workflow logs and test identifiers are public. Raw Gradle output, full assertion messages, and stack traces are suppressed. Do not add production credentials or application secrets to this repository.
- A temporary, non-production Google Services configuration is generated only on the runner to satisfy Gradle resource generation for JVM tests. It is not committed or uploaded.

The workflow is a validation mechanism, not proof of full product readiness. A green result means these automated compile/test/schema checks passed on the recorded source commit only.
