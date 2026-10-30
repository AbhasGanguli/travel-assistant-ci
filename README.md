# Travel Assistant CI Controller

This public repository hosts low-cost validation and internal APK distribution workflows for application source kept in the private `travel-assistant` repository. It does not contain a mirror of the application code.

## Validation and internal APK build

The private source commit is pinned by `requested-source.txt`. The controller workflow:

1. Checks out and verifies the exact private application SHA.
2. Runs the JVM regression suite.
3. Assembles the internal debug APK with a monotonically increasing CI-run version code.
4. Verifies the APK version code and stable signing certificate.
5. Stages the APK in a temporary private release in the application repository.
6. When Workload Identity Federation is configured, a separate Firebase distribution job downloads and SHA-256-verifies the APK, authenticates to Google Cloud, and uploads it to the configured App Distribution tester group.
7. Deletes the private staging release after successful Firebase distribution. If Firebase distribution is not configured or fails, the private release is retained as a fallback.
8. Suppresses raw Firebase CLI output in public workflow logs because successful CLI output includes a temporary APK download URL with a bearer token.
9. Records non-secret build status in `ops/latest-apk-build-status.txt`.

The build job that executes private application source has no GitHub OIDC token permission. Only the separate distribution job receives `id-token: write`; it does not check out or execute application source. The status update is also a separate job and is the only job with a write-capable controller `GITHUB_TOKEN`.

Architecture acceptance and on-device installation testing remain separate gates.

## Request a validation/distribution run

Replace the single 40-character commit SHA in `requested-source.txt` with the private branch commit to validate, then update `ops/BUILD_VALIDATED_APK` on `main`. The marker change starts the build workflow. Alternatively, use `workflow_dispatch` on the `main` branch and supply the exact full SHA in `source_sha`. The workflow reports the source SHA so the result can be tied to the exact code revision.

## Access and privacy

- The `PRIVATE_REPO_READ_TOKEN` Actions secret should be a fine-grained token scoped only to the private application repository with **Contents: Read-only** permission.
- The private source is checked out to the ephemeral runner. Checkout credentials are not persisted for later build steps.
- No application source archive, APK, or raw test-report artifact is uploaded.
- Because this controller repository is public, workflow logs and test identifiers are public. Raw Gradle output, full assertion messages, and stack traces are suppressed. Do not add production credentials or application secrets to this repository.
- A temporary, non-production Google Services configuration is generated only on the runner to satisfy Gradle resource generation for JVM tests. It is not committed or uploaded.

The workflow is a validation mechanism, not proof of full product readiness. A green result means these automated compile/test/schema checks passed on the recorded source commit only.
