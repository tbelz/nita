## ADDED Requirements

### Requirement: Complete workload image references
NITA manifest rendering SHALL substitute the complete values of `NITA_WEBAPP_IMAGE`, `NITA_JENKINS_IMAGE`, and `JUNOS_MCP_IMAGE` into their corresponding persistent workload manifests without appending a registry, repository, or tag.

#### Scenario: Tagged Webapp override is rendered
- **GIVEN** `NITA_WEBAPP_IMAGE=example.invalid/team/webapp:test` is exported
- **WHEN** `apply-k8s.sh` renders the manifests
- **THEN** the Webapp deployment uses exactly `example.invalid/team/webapp:test`

#### Scenario: Jenkins digest override is rendered
- **GIVEN** `NITA_JENKINS_IMAGE=ghcr.io/juniper/nita-jenkins@sha256:<digest>` is exported
- **WHEN** `apply-k8s.sh` renders the manifests
- **THEN** the Jenkins deployment uses the exact digest reference

#### Scenario: Junos MCP override is rendered
- **GIVEN** `JUNOS_MCP_IMAGE=example.invalid/junos-mcp:validation` is exported
- **WHEN** the optional MCP manifests are rendered
- **THEN** the MCP deployment uses exactly that image reference

### Requirement: Ephemeral workload image propagation
The Jenkins deployment SHALL receive `NITA_ANSIBLE_IMAGE` and `NITA_ROBOT_IMAGE` as complete references, and generated Ansible and Robot workloads SHALL use those selected values verbatim.

#### Scenario: Custom Ansible worker image is selected
- **GIVEN** `NITA_ANSIBLE_IMAGE` contains a custom tag or digest
- **WHEN** Jenkins creates an Ansible workload
- **THEN** the workload uses exactly the selected Ansible reference

#### Scenario: Custom Robot worker image is selected
- **GIVEN** `NITA_ROBOT_IMAGE` contains a custom tag or digest
- **WHEN** Jenkins creates a Robot workload
- **THEN** the workload uses exactly the selected Robot reference

### Requirement: Architecture-parity integration validation
Upstream NITA CI SHALL deploy the same rendered manifests on x86 and experimental ARM Kind hosts after canonical component images are multi-platform, verify Ansible and Robot smoke workloads, and run the existing stack integration checks.

#### Scenario: x86 fork CI uses public packages
- **WHEN** NITA CI runs in a personal fork
- **THEN** the x86 Kind deployment pulls canonical public Juniper images rather than deriving package names from the fork owner

#### Scenario: Upstream ARM stack validation
- **WHEN** CI runs in the Juniper NITA repository with multi-platform canonical packages available
- **THEN** the ARM Kind job verifies rollout, database, HTTP, Jenkins RBAC, Ansible, Robot, and API integration behavior
