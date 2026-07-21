# Installation Specification

## Purpose
Automated installation of the full NITA platform onto a supported Linux host,
including all system dependencies, Kubernetes, and pod images.
## Requirements
### Requirement: Supported Operating Systems
The system SHALL support installation on Ubuntu 22.04 LTS and AlmaLinux 9.3 Server.

#### Scenario: Ubuntu installation
- GIVEN a host running Ubuntu 22.04 LTS with sudo access
- WHEN `sudo -E ./install.sh` is executed
- THEN all dependencies are installed and NITA pods reach Running state

#### Scenario: AlmaLinux installation
- GIVEN a host running AlmaLinux 9.3 Server with sudo access
- WHEN `sudo -E ./install.sh` is executed
- THEN all dependencies are installed and NITA pods reach Running state

### Requirement: Interactive Prompts
The install script SHALL prompt the user before each major action (system
dependencies, Kubernetes, NITA pods, optional MCP server) and accept y/n/q answers.

#### Scenario: Accept all defaults
- GIVEN the install script is running interactively
- WHEN the user presses Enter at every prompt
- THEN all components are installed with default values

#### Scenario: Skip a step
- GIVEN the install script is running
- WHEN the user answers n to a prompt
- THEN that installation step is skipped and the next prompt is shown

#### Scenario: Quit the installer
- GIVEN the install script is running
- WHEN the user answers q to any prompt
- THEN the script exits immediately without further changes

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

### Requirement: Minimum Hardware
The system SHALL require at least 8 GB of free memory and 20 GB of free storage.

#### Scenario: Insufficient storage warning
- GIVEN a host with less than 20 GB free
- WHEN install.sh is run without IGNORE_WARNINGS=true
- THEN a warning is displayed and the installer pauses for confirmation

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

### Requirement: Debug Mode
The installer SHALL emit additional diagnostic output when the DEBUG environment
variable is set to true.

#### Scenario: Debug output
- GIVEN `DEBUG=true` is exported in the parent shell
- WHEN install.sh is run
- THEN each installation step prints additional trace output to stdout

### Requirement: ARM host installation boundary
The installer SHALL warn when it is run on an architecture for which full host installation is not officially supported, even though the NITA workload images support ARM Kubernetes operation.

#### Scenario: ARM Linux host starts installation
- **WHEN** `install.sh` detects an ARM Linux host
- **THEN** it displays the existing unsupported-architecture warning unless warnings are explicitly ignored
