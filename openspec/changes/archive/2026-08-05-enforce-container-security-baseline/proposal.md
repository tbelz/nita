## Why

NITA's image workflows currently report HIGH and CRITICAL vulnerabilities without preventing an image containing an unreviewed CRITICAL finding from becoming publishable. The component images also contain avoidable findings from frozen base images, build-only packages, mutable downloads, and unlocked dependencies, so the multi-architecture delivery work needs an enforceable baseline before ARM integration is enabled.

## What Changes

- Establish a trusted-lab container security policy for every NITA-published image.
- Keep complete HIGH/CRITICAL scan reports while blocking any CRITICAL finding that is neither remediated nor explicitly accepted.
- Define package-scoped, owner-attributed, expiring vulnerability exceptions and keep suppressed findings visible.
- Minimize and update the Jenkins, Robot, Webapp, and Junos MCP images; apply the same gate to the already-clean Ansible image.
- Prevent manifest assembly until both architecture-specific images pass smoke and vulnerability gates.
- Document that NITA is intended for isolated, single-operator lab deployments and is not hardened for public or multi-tenant exposure.
- Track deployment hardening and third-party nginx/MariaDB image policy separately.

## Capabilities

### New Capabilities
- `container-security`: Defines vulnerability reporting, CRITICAL gating, exception governance, and the trusted-lab deployment boundary.

### Modified Capabilities
- `container-delivery`: Makes the vulnerability gate blocking before digest publication and multi-platform manifest assembly while retaining complete reports.

## Impact

This change affects the container workflows and Dockerfiles in NITA, nita-ansible, nita-jenkins, nita-robot, nita-webapp, and junos-mcp-server. It changes CI acceptance criteria but does not change application APIs or environment variables. Existing unaccepted CRITICAL findings become release blockers; documented time-limited exceptions remain visible in reports.
