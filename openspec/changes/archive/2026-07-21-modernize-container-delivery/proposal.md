## Why

NITA's component images are currently published as x86-only artifacts with workflow-managed version files, while the installer still defaults to Docker Hub and cannot select complete image references independently. This prevents reproducible ARM Kubernetes deployments and makes fork CI depend on packages that do not exist.

## What Changes

- Build and smoke-test the Ansible, Jenkins, Robot, Webapp, and Junos MCP images natively for `linux/amd64` and `linux/arm64`.
- Publish upstream-only multi-platform GHCR manifests with `latest`, immutable build, and intentional Git-tag tags; never authenticate or publish from pull requests or forks.
- Stop mutating `VERSION.txt` in CI while retaining it as local application/build metadata.
- Attach OCI source metadata, BuildKit SBOMs and provenance, and non-blocking HIGH/CRITICAL Trivy reports.
- Default NITA to public `ghcr.io/juniper` packages and expose complete per-component image overrides that accept tags or digests.
- Pass the selected Ansible and Robot image references into Jenkins-generated ephemeral workloads.
- Validate the stack on x86 and, upstream-only after component publication, on an experimental ARM-hosted Kind runner.
- Document GHCR, multi-platform manifests, tag policy, image overrides, and the boundary between supported ARM Kubernetes operation and experimental ARM host installation.

## Capabilities

### New Capabilities

- `container-delivery`: Defines multi-architecture image validation, security metadata, publication boundaries, and tag/manifest policy for NITA container artifacts.

### Modified Capabilities

- `installation`: Adds registry and complete image-reference overrides while retaining the existing host-architecture support warning.
- `kubernetes`: Requires manifests and generated Jenkins workloads to consume the selected complete image references and adds architecture-parity validation.

## Impact

The change affects GitHub Actions in NITA and four component repositories, the Jenkins and Webapp container build implementations, NITA installer defaults, Kubernetes substitution and manifests, Junos MCP publication, CI tests, and user documentation. Maintainers must merge and publish the four component manifests before enabling the upstream ARM stack job; no new package ownership configuration is expected for the existing repository-linked public GHCR packages.
