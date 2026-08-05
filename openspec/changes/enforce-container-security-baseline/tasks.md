## 1. Repository and policy setup

- [x] 1.1 Synchronize all personal fork bases, preserve unrelated work, and create the six security branches with the required stack bases
- [x] 1.2 Add central trusted-lab security policy documentation and the governed Trivy exception format
- [x] 1.3 Update the NITA container-delivery specification and validate the OpenSpec change

## 2. Component image remediation

- [x] 2.1 Update and minimize the Jenkins Java 21 image, pin direct inputs, and checksum kubectl
- [x] 2.2 Convert Robot to a pinned multi-stage Alpine image and verify its runtime imports
- [x] 2.3 Update Webapp Python and Node dependencies, split its runtime build, and pin the yaml-to-excel source
- [x] 2.4 Minimize Junos MCP, install its locked dependencies, and verify its application/runtime contract
- [x] 2.5 Preserve the clean Ansible runtime without unrelated Dockerfile changes

## 3. Blocking image security gates

- [x] 3.1 Add complete HIGH/CRITICAL reporting and blocking CRITICAL gating to Ansible
- [x] 3.2 Add complete reporting, governed exceptions, and blocking gating to Jenkins
- [x] 3.3 Add complete reporting and blocking gating to Robot
- [x] 3.4 Add complete reporting, governed exceptions, and blocking gating to Webapp
- [x] 3.5 Add complete reporting and blocking gating to Junos MCP's source repository
- [x] 3.6 Gate NITA's Junos MCP publisher and manifest assembly while preserving non-publishing fork behavior

## 4. Validation

- [x] 4.1 Run OpenSpec, workflow, YAML, dependency, unit, and component smoke validation
- [ ] 4.2 Build and scan the owned images on native ARM and validate amd64/arm64 in fork Actions
- [ ] 4.3 Exercise the locally built image set in an isolated temporary Kind cluster and clean up only test artifacts
- [x] 4.4 Confirm accepted findings remain visible and unaccepted or expired CRITICAL findings block delivery

## 5. Delivery and follow-up

- [ ] 5.1 Make existing meaningful multi-architecture drafts ready and retire the obsolete Webapp mirror
- [ ] 5.2 Create fork stacks and ready security pull requests with dependency ordering and validation evidence
- [ ] 5.3 Request and address Codex reviews on the distinctive NITA, Jenkins, Webapp, and Junos MCP fork pull requests
- [ ] 5.4 Create the deployment-hardening and third-party-runtime follow-up issues and update issue 75
- [ ] 5.5 Create upstream component pull requests in prerequisite order and hold the NITA integration pull request until canonical images qualify
