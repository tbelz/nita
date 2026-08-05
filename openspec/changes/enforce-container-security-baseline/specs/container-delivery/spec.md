## MODIFIED Requirements

### Requirement: Trusted digest publication
Component workflows SHALL authenticate and publish only for events in their Juniper source repository on `main` or Git-tag pushes, SHALL push one digest per validated platform, SHALL smoke-test and security-gate each exact pushed digest, and SHALL assemble only those verified digests into a single multi-platform manifest.

#### Scenario: Main publication
- **WHEN** a component workflow succeeds for a push to upstream `main`
- **THEN** the manifest is available as `latest` and `sha-<short-commit>` and contains `linux/amd64` and `linux/arm64`

#### Scenario: Intentional release publication
- **WHEN** a component workflow succeeds for an upstream Git-tag push
- **THEN** the manifest is available as the exact Git tag and `sha-<short-commit>` and contains both required platforms

#### Scenario: Published digest verification
- **WHEN** a trusted platform build pushes a canonical digest
- **THEN** that exact digest passes the component smoke test, complete HIGH/CRITICAL reporting, and blocking CRITICAL gate before it is eligible for manifest assembly

#### Scenario: Release tag is not Docker-compatible
- **WHEN** an upstream Git tag cannot be represented unchanged as a Docker tag
- **THEN** the workflow reports the incompatible tag and stops before constructing a release manifest

#### Scenario: Untrusted event cannot publish
- **WHEN** the workflow runs for a pull request, a non-main branch, or a fork repository
- **THEN** no registry login, digest push, or manifest creation occurs

### Requirement: Supply-chain evidence
Every published component image SHALL include an OCI source label, a BuildKit SBOM, and provenance attestation, and every validation build SHALL produce a downloadable Trivy report for HIGH and CRITICAL vulnerabilities while enforcing the container security baseline.

#### Scenario: Published image can be traced
- **WHEN** an operator inspects a published platform image or manifest
- **THEN** its source repository label, SBOM, and provenance evidence are available

#### Scenario: Existing vulnerability is reported
- **WHEN** Trivy detects a HIGH or CRITICAL finding
- **THEN** the workflow uploads the complete report, keeps accepted findings visible, and fails only when a CRITICAL finding lacks an active package-scoped exception

#### Scenario: Manifest input fails security gate
- **WHEN** either platform image fails its blocking CRITICAL scan
- **THEN** the workflow does not publish a multi-platform manifest from that build
