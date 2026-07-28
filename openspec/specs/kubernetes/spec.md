# Kubernetes Deployment Specification

## Purpose
NITA runs as a set of Kubernetes pods in a dedicated `nita` namespace. This
spec covers namespace, RBAC, storage, deployments, and services as defined by
the manifests in `k8s/`.
## Requirements
### Requirement: Namespace Isolation
The system SHALL deploy all NITA components into a dedicated `nita` namespace.

#### Scenario: Namespace exists after install
- GIVEN install.sh has completed successfully
- WHEN `kubectl get ns` is run
- THEN a namespace named `nita` is listed with status Active

### Requirement: RBAC Permissions
The system SHALL define a ClusterRole that grants the service account
permission to get, list, watch, create, update, patch and delete pods, and to
get, list and watch jobs and pod logs.

#### Scenario: Jenkins can query pod state
- GIVEN the ClusterRole and ClusterRoleBinding are applied
- WHEN Jenkins queries pod status via the Kubernetes API
- THEN the request is authorised and returns pod state

### Requirement: Persistent Storage
The system SHALL provision PersistentVolumes and PersistentVolumeClaims for
Jenkins home data and the MariaDB data directory.

#### Scenario: Jenkins data persists across pod restart
- GIVEN a Jenkins pod with a PVC for its home directory
- WHEN the Jenkins pod is deleted and recreated
- THEN previously configured jobs and build history remain intact

#### Scenario: Database data persists across pod restart
- GIVEN a MariaDB pod with a PVC for its data directory
- WHEN the MariaDB pod is deleted and recreated
- THEN the Sites database and all network data remain intact

### Requirement: Core Pod Deployments
The system SHALL deploy and keep running four persistent pods: `webapp`,
`jenkins`, `proxy`, and `db`.

#### Scenario: All pods reach Running state
- GIVEN the k8s manifests have been applied with `apply-k8s.sh`
- WHEN `kubectl get pods -n nita` is run after 2 minutes
- THEN webapp, jenkins, proxy and db pods all show STATUS Running

### Requirement: Readiness Probes
The webapp and db deployments SHALL define readinessProbes so that Kubernetes
only routes traffic once each pod is fully initialised.

#### Scenario: Webapp readiness check
- GIVEN the webapp pod is starting up
- WHEN the TCP socket on port 8000 is not yet accepting connections
- THEN the pod is not marked Ready and receives no traffic

#### Scenario: Webapp ready after init
- GIVEN the webapp has finished starting
- WHEN the TCP socket on port 8000 accepts a connection
- THEN the pod is marked Ready and starts receiving traffic

### Requirement: Services and Port Exposure
The system SHALL expose the webapp on port 8000, Jenkins on port 8443, and the
proxy on ports 443 (HTTPS) and 8000 within the cluster.

#### Scenario: Webapp service reachable
- GIVEN the webapp deployment and service are applied
- WHEN an in-cluster request is made to the webapp service on port 8000
- THEN a valid HTTP response is returned

### Requirement: Optional Junos MCP Pod
The system SHALL support optional deployment of a `junos-mcp-server` pod in the
nita namespace listening on port 8090, controlled by a ConfigMap for device
mapping.

#### Scenario: MCP pod deployed
- GIVEN the junos-mcp-deployment.yaml and junos-mcp-service.yaml are applied
- WHEN `kubectl get pods -n nita` is run
- THEN a junos-mcp-server pod shows STATUS Running

#### Scenario: MCP device list updated
- GIVEN the MCP pod is running
- WHEN the junos-mcp-devices-cm ConfigMap is recreated with new device data
  and the pod is restarted
- THEN the pod uses the new device list

### Requirement: Jenkins Port Isolation via NetworkPolicy
The system SHALL deploy a `NetworkPolicy` in the `nita` namespace that restricts
ingress to the Jenkins pod on port 8080 to only pods labelled
`io.kompose.service: webapp` in the same namespace. Ingress on port 8443 SHALL
remain unrestricted.

#### Scenario: Webapp pod can reach Jenkins on port 8080
- GIVEN the ClusterRole and ClusterRoleBinding are applied
- WHEN a request originates from the pod labelled `io.kompose.service: webapp`
  in the `nita` namespace and targets the Jenkins pod on port 8080
- THEN the connection is permitted

#### Scenario: Non-webapp pod cannot reach Jenkins on port 8080
- WHEN a request originates from any pod NOT labelled `io.kompose.service: webapp`
  and targets the Jenkins pod on port 8080
- THEN the connection is dropped by the NetworkPolicy

#### Scenario: Any pod can reach Jenkins on port 8443
- WHEN any pod in any namespace makes a request to the Jenkins pod on port 8443
- THEN the connection is permitted

#### Scenario: NetworkPolicy exists after apply
- GIVEN `apply-k8s.sh` has completed successfully
- WHEN `kubectl get networkpolicy -n nita` is run
- THEN a NetworkPolicy named `jenkins-network-policy` is listed

### Requirement: Declarative Jenkins Service Manifest
The Jenkins Service manifest SHALL contain no cluster-assigned immutable fields
(`clusterIP`, `clusterIPs`, `resourceVersion`, `uid`) so that `kubectl apply`
succeeds idempotently on any cluster.

#### Scenario: Jenkins service applies without error on a clean cluster
- GIVEN no prior NITA install on the cluster
- WHEN `kubectl apply -f k8s/jenkins-service.yaml` is run
- THEN the command exits with code 0 and the service is created

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
Upstream NITA CI SHALL deploy the same rendered manifests on x86 and, only when the `NITA_ARM_CI_ENABLED` repository variable is `true`, experimental ARM Kind hosts after canonical component images are multi-platform, verify Ansible and Robot smoke workloads, and run the existing stack integration checks.

#### Scenario: x86 fork CI uses public packages
- **WHEN** NITA CI runs in a personal fork
- **THEN** the x86 Kind deployment pulls canonical public Juniper images rather than deriving package names from the fork owner

#### Scenario: Upstream ARM stack validation
- **WHEN** CI runs in the Juniper NITA repository with multi-platform canonical packages available and `NITA_ARM_CI_ENABLED=true`
- **THEN** the ARM Kind job verifies rollout, database, HTTP, Jenkins RBAC, Ansible, Robot, and API integration behavior

#### Scenario: Canonical packages are not yet ARM-ready
- **WHEN** `NITA_ARM_CI_ENABLED` is absent or is not `true`
- **THEN** CI schedules only the x86 Kind job, even in the Juniper NITA repository
