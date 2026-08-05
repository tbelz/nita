## Context

The multi-architecture workflows build and smoke-test native amd64 and arm64 images, upload Trivy HIGH/CRITICAL reports, and publish per-platform digests before assembling a manifest. The reports are currently non-blocking. Refreshed scans show no CRITICAL findings in Ansible, but multiple findings in Jenkins, Robot, Webapp, and Junos MCP from frozen base images, unnecessary runtime packages, application dependencies, and mutable build inputs.

NITA is deployed by an individual operator into a trusted lab environment and connects to Juniper Cloud Labs. It is not designed as an internet-facing or multi-tenant service. That boundary lowers the likelihood of several findings but does not justify silently publishing new CRITICAL vulnerabilities.

## Goals / Non-Goals

**Goals:**

- Block publication and manifest assembly for every unaccepted CRITICAL finding on either supported architecture.
- Preserve complete HIGH/CRITICAL reports, including accepted findings.
- Make every exception package-scoped, justified, owned, reviewable, and time-limited.
- Reduce findings by updating supported bases, removing build-only or unused runtime packages, and locking mutable inputs.
- Keep fork and pull-request validation non-publishing.

**Non-Goals:**

- Claim support for public, hostile, or multi-tenant deployment.
- Resolve default credentials, root execution, host mounts, cluster-wide RBAC, or host-port exposure in this change.
- Gate on HIGH findings.
- Introduce Dependabot, Renovate, or another dependency-update policy.
- Gate third-party nginx and MariaDB images under the owned-image workflow.

## Decisions

### Separate complete reporting from the blocking decision

Each native image job runs Trivy 0.69.3 twice after its component smoke test. This is the first verified binary release recommended after Trivy's 2026 release-infrastructure incident; the workflow uses the corresponding recommended `trivy-action` 0.35.0 release and reuses the installed binary for the second scan. Both invocations explicitly select Trivy's vulnerability scanner; secret and misconfiguration scanning are separate policy concerns and are not represented as package-CVE findings. The first scan writes a HIGH/CRITICAL JSON artifact with suppressed findings shown and never fails. The second scans only CRITICAL findings, applies the repository's `.trivyignore.yaml`, and exits non-zero for an unaccepted finding. This preserves evidence while giving publication jobs an ordinary blocking dependency.

An alternative single filtered scan was rejected because accepted findings would disappear from the review artifact. Blanket `--ignore-unfixed` was rejected because fix availability is not a sufficient risk decision.

### Scope and govern exceptions

Exceptions use Trivy's YAML ignore format and constrain each CVE to its package URL or installed path. The statement records applicability, trusted-lab reachability, owner, and tracking issue; `expired_at` is no more than 90 days from approval. An expired exception no longer suppresses the gate. The full report uses `--show-suppressed` so an exception is never invisible.

Exceptions are considered only after removing unused packages and applying supported security updates. Architecture-specific applicability must be stated when a finding differs between amd64 and arm64.

### Prefer runtime minimization over broad risk acceptance

- Jenkins moves to the supported Java 21 LTS line, removes unused `git-lfs` and build-only packages, pins plugins and direct Python dependencies, and checks the architecture-specific `kubectl` download.
- Robot uses a multi-stage Alpine runtime, removes editor/build tooling, and pins direct dependencies. Slim-trixie is the predetermined fallback only if native musl validation fails.
- Webapp keeps a conservative Debian runtime but separates compilation, updates Django and the npm lock, replaces mutable source downloads, and removes build-only packages. Its database wait loop uses the existing Python driver so the runtime keeps only the MariaDB client library rather than the full CLI and Perl dependency tree.
- Junos MCP uses its existing `uv.lock` in a minimized Alpine runtime and removes operating-system networking tools not invoked by the service. It retains the OpenSSH client because documented `ssh_config` `ProxyCommand` connections invoke the local `ssh` binary through the NETCONF dependency stack. Slim-trixie is the fallback only if native musl validation fails.
- Ansible retains its existing Alpine image and receives only the common gate.

### Gate the exact publishable object

Validation builds are scanned locally on branch and pull-request events. Trusted main/tag jobs also scan the exact per-platform image before it is eligible for manifest assembly. The manifest job depends on every platform's smoke test and CRITICAL gate; no partial or failed pair can become `latest` or a release tag.

### Use one cross-repository OpenSpec authority

The NITA change records the contract and delivery ordering for all six repositories. Component pull requests link to this change instead of initializing OpenSpec in repositories that do not currently use it or duplicating the same policy in repositories that do.

## Risks / Trade-offs

- **Alpine can expose musl or native-extension incompatibilities** → Build and smoke-test natively on both architectures; use slim-trixie only under the documented fallback rule.
- **Vendor images can retain CRITICAL findings without an available update** → Minimize packages first, then add package-scoped exceptions with a maximum 90-day lifetime.
- **Severity records can duplicate one CVE across binary packages** → Review distinct CVE/package/path tuples and retain the raw report for auditability.
- **Blocking scans can interrupt scheduled publication when databases update** → Treat this as the intended safe failure; remediate or explicitly review the finding before retrying.
- **The trusted-lab warning can be mistaken for hardening** → State plainly that public and multi-tenant exposure are unsupported and track deployment controls separately.

## Migration Plan

1. Make the existing multi-architecture pull requests ready and synchronize personal fork bases.
2. Implement and validate security branches in personal forks, using same-repository stacks where the multi-architecture branch is still a prerequisite.
3. Request Codex review on the distinctive NITA, Jenkins, Webapp, and Junos MCP fork pull requests and address findings.
4. Merge component multi-architecture changes upstream, then submit component security pull requests.
5. Confirm canonical manifests contain amd64 and arm64 images that satisfy the gate.
6. Submit the NITA integration pull request and enable ARM integration only after the canonical image prerequisite is true.

Rollback consists of reverting a component security pull request. The workflow gate itself must not be bypassed; an urgent vendor finding is handled through a reviewed, expiring exception.

## Open Questions

None. Dependency automation, public-deployment hardening, and third-party runtime image policy require separate maintainer decisions.
