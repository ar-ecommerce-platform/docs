# 1. Vendor the Gradle quality config per repo

**Status:** Accepted

## Context

Every service's `build.gradle` did `apply from: "https://raw.githubusercontent.com/.../.github/develop/gradle/build-quality.gradle"`
and that script in turn fetched a Checkstyle config from a second raw URL. That means:

- the build depends on GitHub being reachable, on a moving branch, at configure time — and again
  inside `docker build`;
- it breaks silently when a file is moved in a different repo;
- it also pulled in OWASP Dependency-Check (needs the NVD database + an API key), SonarCloud
  (needs `SONAR_TOKEN`) and Snyk (needs `SNYK_TOKEN`) — none of which belong in a local build.

## Decision

Each repo carries its own `gradle/quality.gradle` + `config/checkstyle/*` and does
`apply from: "$rootDir/gradle/quality.gradle"`. The kept gates are offline-capable: Spotless
(google-java-format), Checkstyle (with a cyclomatic-complexity limit of 10), and a JaCoCo report.
OWASP / SonarCloud / Snyk move to CI, where their secrets live as GitHub Environment secrets.

## Consequences

- Builds are reproducible and hermetic — `./gradlew build` and `docker build` work with no
  network to github.com.
- The config is duplicated across ~12 repos. Accepted for now; the intended end state is a
  published convention plugin (or a composite build) in `build-conventions`.
- Security/quality scanning still happens, just per-PR in CI rather than on every local build.
