## ADDED Requirements

### Requirement: Complete vulnerability evidence
Every NITA-published image workflow SHALL produce a downloadable HIGH/CRITICAL vulnerability report for each supported architecture, and the report SHALL include findings accepted by an exception.

#### Scenario: Accepted finding remains visible
- **WHEN** a scan encounters a CRITICAL finding covered by an active exception
- **THEN** the complete report records the finding as suppressed while the blocking decision succeeds

#### Scenario: High finding is reported
- **WHEN** a scan encounters a HIGH finding without a CRITICAL finding
- **THEN** the workflow uploads the finding and does not fail solely because of its HIGH severity

### Requirement: Unaccepted CRITICAL findings block delivery
Every NITA-published image workflow SHALL fail validation and prevent publication when either supported architecture contains a CRITICAL finding that is not covered by an active package-scoped exception.

#### Scenario: Pull request introduces a CRITICAL finding
- **WHEN** an amd64 or arm64 validation image contains an unaccepted CRITICAL finding
- **THEN** that platform job fails and the pull request cannot satisfy the image validation checks

#### Scenario: Trusted build introduces a CRITICAL finding
- **WHEN** a trusted main, tag, or scheduled build contains an unaccepted CRITICAL finding
- **THEN** no digest from the failed build is eligible for multi-platform manifest assembly

### Requirement: Vulnerability exceptions are governed
Every accepted CRITICAL finding MUST be constrained to the affected package or installed path and MUST document its applicability, rationale, owner, tracking issue, and an expiration date no more than 90 days after approval.

#### Scenario: Exception expires
- **WHEN** the current date is later than an exception's expiration date
- **THEN** the finding is no longer suppressed and the CRITICAL gate fails

#### Scenario: CVE appears in an unlisted package
- **WHEN** an accepted CVE is detected in a package or path outside the exception scope
- **THEN** the new finding remains unaccepted and the CRITICAL gate fails

### Requirement: Trusted-lab deployment boundary
NITA documentation SHALL identify the supported deployment as an isolated, single-operator trusted lab and SHALL warn that the default deployment is not hardened for public, hostile, or multi-tenant exposure.

#### Scenario: Operator reviews exposure guidance
- **WHEN** an operator evaluates exposing NITA services beyond a trusted lab network
- **THEN** the documentation warns against exposing ports 443, 8443, or 8090 and points to the separate deployment-hardening work

### Requirement: Owned-image baseline
The security baseline SHALL cover the Ansible, Jenkins, Robot, Webapp, and NITA-published Junos MCP images while tracking third-party runtime images separately.

#### Scenario: Owned image changes
- **WHEN** any owned image Dockerfile, base image, or dependency set changes
- **THEN** both supported architectures run the complete report and blocking CRITICAL gate

#### Scenario: Third-party runtime finding is discovered
- **WHEN** a finding is reported in the nginx or MariaDB runtime image
- **THEN** it is recorded under the third-party image policy follow-up and is not silently represented as remediated by the owned-image gate
