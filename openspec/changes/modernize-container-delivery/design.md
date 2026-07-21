## Context

NITA is delivered from five repositories: this integration repository plus the `nita-ansible`, `nita-jenkins`, `nita-robot`, and `nita-webapp` component repositories. Each component currently has an x86-only image workflow that derives publication tags from `VERSION.txt`; NITA defaults to Docker Hub-style image construction and its fork CI can derive package names that do not exist. Existing public GHCR packages are linked to their Juniper source repositories but contain only `linux/amd64` manifests.

The target is Kubernetes workload portability, not a support claim for installing the full host stack on ARM Linux. GitHub's `ubuntu-24.04-arm` hosted runner remains a public-preview dependency. The change must be testable in personal forks without authenticating to GHCR or creating personal packages, and the four component changes must land before the NITA integration can rely on canonical ARM manifests.

## Goals / Non-Goals

**Goals:**

- Produce and validate native `linux/amd64` and `linux/arm64` component images.
- Publish one attested, multi-platform GHCR manifest per release tag from trusted upstream events only.
- Let installations select every complete image reference independently, including immutable digests.
- Validate the same NITA Kubernetes behavior on x86 and ARM hosts.
- Preserve actionable vulnerability evidence without making known legacy findings a new merge blocker.
- Deliver five independently reviewable branches and PR descriptions.

**Non-Goals:**

- Officially support the full NITA host installer on ARM Linux.
- Include `nita-ansible-ee`, which is not selected by the installer or runtime manifests.
- Automatically edit or commit `VERSION.txt`.
- Implement release branches, `build-templates` cleanup, NOTICES generation, release orchestration, or Teams notifications from issue #29.
- Remediate all existing HIGH/CRITICAL findings in this change series.
- Create pull requests against Juniper repositories.

## Decisions

### Validate and publish in separate trust stages

Each component workflow uses a native runner matrix (`ubuntu-24.04` for amd64 and `ubuntu-24.04-arm` for arm64) to build a locally loaded image and run architecture-specific smoke tests on every branch push and pull request. A separate upstream-only publication matrix runs after validation, authenticates with `GITHUB_TOKEN`, pushes canonical per-platform digests, and uploads digest markers. A final trusted job assembles the manifest from those digests.

This duplicates the BuildKit invocation but makes the security boundary explicit: untrusted events never reach login or publication steps, smoke tests always exercise a runnable local image, and digest assembly is easy to audit. Shared BuildKit cache scopes limit the additional build cost. A single conditional build step was rejected because it tangles local loading, digest output, and credentials in one job.

### Derive public tags from the trusted Git event

Upstream `main` publishes `latest` and `sha-<short-commit>`; an upstream Git-tag push publishes the exact tag and the same immutable SHA tag. `VERSION.txt` remains available to local build scripts and applications but is not an input or output of CI. This separates source-controlled application metadata from container artifact identity and avoids workflows that push commits back to protected branches.

The scheduled Junos MCP rebuild is different because its contents are derived from another repository. It publishes `latest` plus `source-<short-upstream-sha>-run-<workflow-run-id>`, making both the source revision and rebuild attempt discoverable without pretending the NITA commit identifies the image.

### Use complete image references as the deployment interface

`CONTAINER_REGISTRY` defaults to `ghcr.io/juniper` and `GITHUB_ORG` defaults to `Juniper`. Five environment variables default to canonical `:latest` references but are substituted verbatim into manifests: `NITA_WEBAPP_IMAGE`, `NITA_JENKINS_IMAGE`, `NITA_ANSIBLE_IMAGE`, `NITA_ROBOT_IMAGE`, and `JUNOS_MCP_IMAGE`. Because callers supply a complete reference, each component can independently use a tag or digest without ambiguous registry/name/tag composition.

Jenkins receives the selected Ansible and Robot references as environment variables so its generated ephemeral pods use the same deployment choices. Retaining the old name/tag concatenation as a second path was rejected because it would create precedence rules and prevent clean digest use.

### Keep build behavior target-aware and reproducible

The Jenkins Dockerfile consumes BuildKit's `TARGETARCH` when downloading `kubectl`; build scripts no longer guess the host architecture. The Webapp build removes an ARM-only source mutation and corrects its OCI source label. Ansible and Robot stay unchanged unless native ARM builds demonstrate a real incompatibility.

Published images include an OCI source label, BuildKit SBOM, and maximum provenance. Trivy JSON reports for HIGH/CRITICAL findings are uploaded for each platform, but the scan exits successfully during this first series. A later remediation change can establish a reviewed baseline and turn the scan into a gate.

### Stage cross-repository rollout

The four component branches are reviewed and merged first. Once their canonical GHCR manifests contain both platforms, the NITA branch can exercise an upstream-only ARM Kind job alongside existing x86 validation. Personal-fork NITA CI always pulls public Juniper packages; it never constructs `ghcr.io/<fork-owner>` image paths.

## Risks / Trade-offs

- [GitHub ARM hosted runners are public preview] → Keep ARM host installation explicitly experimental, isolate runner selection in the matrix, and retain local Apple Silicon validation evidence.
- [Two native build passes increase workflow time] → Reuse architecture-scoped BuildKit caches and favor the clear trust boundary over minimizing minutes.
- [Canonical ARM images are unavailable until component PRs merge] → Gate upstream ARM integration appropriately and sequence component delivery before the NITA PR is marked ready.
- [Non-blocking scans allow known vulnerabilities] → Retain downloadable reports, document the baseline, and prepare remediation follow-ups before enforcing failures.
- [A tag can be moved after publication] → Also publish immutable source-derived tags and document digest pinning for strict reproducibility.
- [Full image overrides can reference incompatible images] → Document the interface and validate default canonical images; custom images remain an operator responsibility.

## Migration Plan

1. Merge the four component changes and verify the canonical GHCR manifests expose amd64 and arm64.
2. Merge the NITA integration change with GHCR defaults and image overrides; existing deployments can retain custom images by setting the new complete-reference variables.
3. Enable the upstream ARM Kind job after canonical multi-platform images are observable.
4. Retain existing GHCR tags during the transition. Rollback consists of reverting a repository's workflow or NITA defaults; published immutable tags and digests remain available.
5. Track vulnerability remediation and the remaining release automation separately under issue #29 follow-up work.

## Open Questions

No blocking design questions remain. Maintainers may later choose when HIGH/CRITICAL scanning becomes blocking and when GitHub's ARM runner maturity is sufficient to broaden the host-install support statement.
