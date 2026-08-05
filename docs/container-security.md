# Container security

NITA is intended for an isolated, single-operator lab that connects to trusted
Juniper Cloud Labs. It is not a hardened internet-facing or multi-tenant
service. Keep the NITA host and Kubernetes node on a trusted network, restrict
access with an external firewall, and do not expose ports 443, 8443, or 8090 to
untrusted networks.

This deployment boundary reduces the reachability of some vulnerabilities; it
does not remove the need to update dependencies, minimize images, or review
scanner findings. NITA's image workflows therefore enforce the following
baseline for the Ansible, Jenkins, Robot, Webapp, and Junos MCP images.

## Vulnerability baseline

Every amd64 and arm64 image build produces a downloadable Trivy JSON report for
HIGH and CRITICAL findings. The report includes suppressed findings so accepted
risks remain visible. A separate scan fails the build for any CRITICAL finding
that is not covered by an active exception. HIGH findings are reported but are
not blocking in this first enforcement series.

Fork and pull-request workflows perform the same builds, smoke tests, and scans
without logging in to a registry or publishing packages. On trusted Juniper
main, tag, or scheduled builds, both platform images must pass their smoke tests
and CRITICAL gates before a multi-platform manifest can be assembled.

The owned-image baseline does not include the standard `nginx` and `mariadb`
runtime images. Their inventory, version/digest policy, update ownership, and
scanning are tracked separately; they must not be described as covered by the
owned-image gate.

## Current accepted risks

The August 5, 2026 ARM validation images have the following post-remediation
baseline. Counts are scanner records, so one CVE can appear against multiple
binary packages. Raw JSON reports remain the source of record.

| Image | HIGH | CRITICAL | Accepted CRITICAL CVEs |
| --- | ---: | ---: | --- |
| Ansible | 5 | 0 | None |
| Jenkins | 66 | 18 | CVE-2026-13221, CVE-2026-42496, CVE-2026-57433, CVE-2026-8376, CVE-2026-60002, CVE-2026-6653 |
| Robot | 6 | 0 | None |
| Webapp | 29 | 8 | CVE-2026-13221, CVE-2026-42496, CVE-2026-57433, CVE-2026-8376, CVE-2026-44172, CVE-2026-49261 |
| Junos MCP | 18 | 0 | None |

All current exceptions expire on November 3, 2026. The authoritative package
scope, reachability analysis, ownership, tracking link, and expiration live in
the component's `.trivyignore.yaml`; this summary does not grant an exception.
Jenkins retains stable Debian Perl, OpenSSH client, and libxml2 findings needed
by its lab automation toolchain. Webapp retains the base image's `perl-base`
and the MariaDB client library used by its Python database driver. The full
MariaDB command-line client and its larger Perl dependency tree are not shipped.

## Exception policy

Removing an unused package or applying a supported update is preferred to an
exception. An exception is permitted only after maintainers have reviewed the
finding in the context of NITA's trusted-lab deployment.

Each repository stores exceptions in `.trivyignore.yaml`. Every entry must:

- identify the CVE and constrain it to the affected package URL or installed
  path;
- explain applicability and reachability rather than relying only on the lack
  of an upstream fix;
- name the responsible maintainer or team and link a tracking issue;
- expire no more than 90 days after approval; and
- account for architecture-specific behavior when amd64 and arm64 differ.

For example:

```yaml
vulnerabilities:
  - id: CVE-YYYY-NNNNN
    purls:
      - pkg:deb/debian/example
    statement: >-
      Not reachable in the NITA runtime because ...; owner: NITA maintainers;
      tracking: https://github.com/Juniper/nita/issues/75
    expired_at: 2026-11-03
```

Do not use a blanket ignore for all unfixed findings. When an exception expires,
Trivy stops suppressing it and the CRITICAL gate fails until maintainers update
the package or explicitly review the risk again.

## Outside this baseline

The default deployment currently includes credentials intended for a local lab,
Jenkins execution with elevated access, Kubernetes RBAC capable of creating
ephemeral workloads, host-mounted storage, and host-accessible service ports.
Those properties require separate hardening before using NITA on a public,
hostile, shared, or multi-tenant network. Public availability of the source code
or container images must not be interpreted as a claim that a running default
deployment is safe for public exposure.
