## MODIFIED Requirements

### Requirement: Environment Variable Overrides
The installer SHALL respect environment variables set in the parent shell to override default installation paths, registry configuration, and complete component image references. Complete image overrides SHALL accept independent tags or immutable digests.

#### Scenario: Custom install root
- GIVEN `NITAROOT=/srv` is exported before running install.sh
- WHEN the installer runs
- THEN repositories are cloned under `/srv` instead of `/opt`

#### Scenario: Custom binary directory
- GIVEN `BINDIR=/usr/bin` is exported
- WHEN the installer runs
- THEN `nita-cmd` and CLI scripts are installed to `/usr/bin`

#### Scenario: Default public image references
- **GIVEN** no registry or image variables are exported
- **WHEN** the installer prepares Kubernetes manifests
- **THEN** `CONTAINER_REGISTRY` is `ghcr.io/juniper`, `GITHUB_ORG` is `Juniper`, and every NITA image override resolves to its canonical `:latest` image

#### Scenario: Independent immutable component selection
- **GIVEN** one or more of `NITA_WEBAPP_IMAGE`, `NITA_JENKINS_IMAGE`, `NITA_ANSIBLE_IMAGE`, `NITA_ROBOT_IMAGE`, and `JUNOS_MCP_IMAGE` contain complete digest references
- **WHEN** the installer prepares Kubernetes manifests
- **THEN** each provided reference is preserved verbatim and components without an override retain their canonical defaults

### Requirement: Optional Junos MCP Server
The installer SHALL offer to deploy a Junos MCP server pod on port 8090 as an optional component using the selected `JUNOS_MCP_IMAGE` reference.

#### Scenario: MCP server accepted
- GIVEN the user answers y to the MCP server prompt
- WHEN the installer runs
- THEN the selected Junos MCP image is deployed in the nita namespace

#### Scenario: MCP server declined
- GIVEN the user answers n to the MCP server prompt
- WHEN the installer runs
- THEN the MCP pod is not deployed and the installer continues

## ADDED Requirements

### Requirement: ARM host installation boundary
The installer SHALL warn when it is run on an architecture for which full host installation is not officially supported, even though the NITA workload images support ARM Kubernetes operation.

#### Scenario: ARM Linux host starts installation
- **WHEN** `install.sh` detects an ARM Linux host
- **THEN** it displays the existing unsupported-architecture warning unless warnings are explicitly ignored
