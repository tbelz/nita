## 1. Repository and Workflow Setup

- [x] 1.1 Create and synchronize the four `tbelz` component forks, configure upstream remotes and repository-local attribution, and create `codex/multiarch-container-delivery` branches.
- [x] 1.2 Establish the shared native-runner workflow structure with untrusted validation, upstream-only digest publication, manifest assembly, BuildKit evidence, and non-blocking Trivy artifacts.

## 2. Component Images

- [ ] 2.1 Implement and smoke-test the amd64/arm64 workflow for `nita-ansible`.
- [ ] 2.2 Make the Jenkins build consume BuildKit `TARGETARCH`, remove host guessing, and implement its amd64/arm64 workflow and smoke tests.
- [ ] 2.3 Implement and smoke-test the amd64/arm64 workflow for `nita-robot`.
- [ ] 2.4 Remove the Webapp ARM dependency mutation, correct OCI source metadata, and implement its amd64/arm64 workflow and smoke tests.

## 3. NITA Integration

- [x] 3.1 Add GHCR defaults and complete image-reference overrides to the installer, renderer, manifests, and Jenkins ephemeral workload configuration.
- [x] 3.2 Convert the Junos MCP image workflow to native amd64/arm64 validation and trusted source-SHA/run-ID manifest publication.
- [x] 3.3 Make fork x86 CI use canonical Juniper packages and add an upstream ARM Kind integration job with equivalent stack and worker smoke coverage.
- [x] 3.4 Add regression tests for defaults, verbatim tag/digest substitution, and Jenkins image propagation.
- [x] 3.5 Update installation and container documentation for GHCR, tag policy, overrides, multi-platform support, and the experimental ARM host boundary.

## 4. Local Validation

- [x] 4.1 Run OpenSpec validation, unit/regression tests, shell and Python checks, YAML/Kubernetes schema validation, and workflow linting.
- [x] 4.2 Build and smoke-test all four component images and Junos MCP natively on the local ARM Docker host.
- [x] 4.3 Exercise per-platform digest publication and manifest assembly against a temporary local registry and verify both required platforms.
- [ ] 4.4 Deploy locally built images through the override interface to a new isolated Kind cluster, run rollout, database, HTTP, Jenkins RBAC, worker, and API integration checks, then remove only temporary test resources.

## 5. Fork Validation and Delivery

- [ ] 5.1 Enable Actions in the five `tbelz` forks and verify component branch runs validate both architectures without creating personal GHCR packages.
- [ ] 5.2 Verify the NITA fork's x86 CI deploys canonical public Juniper images and record the upstream-only dependency for ARM CI.
- [ ] 5.3 Commit and push the five focused branches without creating Juniper pull requests.
- [ ] 5.4 Prepare ready-to-paste titles, bodies, issue references, validation evidence, merge ordering, and maintainer handoff notes for all five pull requests.
- [ ] 5.5 Sync the delta specifications into the main OpenSpec specifications and archive the completed `modernize-container-delivery` change.
