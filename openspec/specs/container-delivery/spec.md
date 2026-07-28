# container-delivery Specification

## Purpose
Define how NITA component and Junos MCP images are validated, published, tagged,
and accompanied by supply-chain evidence across supported CPU architectures.

## Requirements
### Requirement: Native multi-platform validation
Every component image workflow SHALL build and smoke-test `linux/amd64` on an `ubuntu-24.04` runner and `linux/arm64` on an `ubuntu-24.04-arm` runner for every branch push and pull request.

#### Scenario: Pull request validates both platforms
- **WHEN** a pull request changes a component repository
- **THEN** blocking amd64 and arm64 jobs build the image and pass its component-specific smoke tests

#### Scenario: Fork branch validates without publication
- **WHEN** a branch is pushed in a fork
- **THEN** both platform builds and smoke tests run without authenticating to a registry or publishing an image

### Requirement: Component smoke coverage
Image validation SHALL verify the executable and packaged runtime assets needed by each component.

#### Scenario: Ansible image is validated
- **WHEN** the Ansible image smoke test runs
- **THEN** `ansible --version` succeeds and the packaged network dependencies can be imported

#### Scenario: Jenkins image is validated
- **WHEN** the Jenkins image smoke test runs
- **THEN** Java, required Python dependencies, installed plugins, and the target-architecture `kubectl` executable are present and runnable

#### Scenario: Robot image is validated
- **WHEN** the Robot image smoke test runs
- **THEN** Robot Framework reports its version and required Python modules can be imported

#### Scenario: Webapp image is validated
- **WHEN** the Webapp image smoke test runs
- **THEN** Django and MySQL modules can be imported and the compiled frontend assets are present

### Requirement: Trusted digest publication
Component workflows SHALL authenticate and publish only for events in their Juniper source repository on `main` or Git-tag pushes, SHALL push one digest per validated platform, and SHALL assemble those digests into a single multi-platform manifest.

#### Scenario: Main publication
- **WHEN** a component workflow succeeds for a push to upstream `main`
- **THEN** the manifest is available as `latest` and `sha-<short-commit>` and contains `linux/amd64` and `linux/arm64`

#### Scenario: Intentional release publication
- **WHEN** a component workflow succeeds for an upstream Git-tag push
- **THEN** the manifest is available as the exact Git tag and `sha-<short-commit>` and contains both required platforms

#### Scenario: Untrusted event cannot publish
- **WHEN** the workflow runs for a pull request, a non-main branch, or a fork repository
- **THEN** no registry login, digest push, or manifest creation occurs

### Requirement: Immutable workflow metadata
Component workflows SHALL NOT read, modify, or commit `VERSION.txt` and SHALL derive published image identity from the trusted Git event.

#### Scenario: Successful CI leaves version metadata unchanged
- **WHEN** any component image workflow completes
- **THEN** `VERSION.txt` is unchanged and no CI-authored source commit is created

### Requirement: Supply-chain evidence
Every published component image SHALL include an OCI source label, a BuildKit SBOM, and provenance attestation, and every validation build SHALL produce a downloadable non-blocking Trivy report for HIGH and CRITICAL vulnerabilities.

#### Scenario: Published image can be traced
- **WHEN** an operator inspects a published platform image or manifest
- **THEN** its source repository label, SBOM, and provenance evidence are available

#### Scenario: Existing vulnerability is reported
- **WHEN** Trivy detects a HIGH or CRITICAL finding
- **THEN** the workflow uploads the report and does not fail solely because of that finding in this change series

### Requirement: Target-aware build inputs
Container builds SHALL select architecture-dependent artifacts from BuildKit target metadata and SHALL NOT mutate application dependency files based on the build host architecture.

#### Scenario: Jenkins downloads kubectl for the target
- **WHEN** the Jenkins image is built for either supported platform
- **THEN** the downloaded `kubectl` binary matches BuildKit's `TARGETARCH`

#### Scenario: Webapp builds without source mutation
- **WHEN** the Webapp image is built on ARM
- **THEN** its dependency source files are not rewritten as an architecture workaround

### Requirement: Junos MCP multi-platform publication
The NITA-owned Junos MCP workflow SHALL build amd64 and arm64 images and, on trusted upstream rebuilds, publish `latest` plus an immutable tag containing the upstream Junos source SHA and workflow run ID.

#### Scenario: Scheduled Junos rebuild succeeds
- **WHEN** the upstream NITA scheduled workflow resolves a Junos source commit and both native builds pass
- **THEN** one manifest containing both platforms is published under `latest` and `source-<short-sha>-run-<run-id>`

#### Scenario: Fork Junos workflow runs
- **WHEN** the Junos workflow is dispatched or pushed in a fork
- **THEN** it may validate builds but cannot authenticate or publish
