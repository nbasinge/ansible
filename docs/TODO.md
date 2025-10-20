# MyCareCRM Deployment Automation — TODO

Date: 2025-10-20
Owner: nbasinge

## Completed
- Scaffolded repo (ansible.cfg, inventory, collections/requirements.yml)
- Orchestrator: playbooks/task-master-orchestrator.yml
  - Payload parsing + JSON Schema validation (ansible.utils.validate)
  - network_automation flag to skip external APIs
- Includes:
  - includes/synergy-dns-creation.yml
  - includes/unifi-vlan-setup.yml
  - includes/sophos-firewall-config.yml
  - includes/sophos-vlan-creation.yml
- AWX artifacts:
  - awx/credential_types/* (Synergy, UniFi, Sophos)
  - awx/setup_awx.yml (Project, Inventory, Job Template, Credential Types)
- Schema + sample:
  - schemas/deploy_dataset.schema.json
  - examples/sample_deploy_dataset.json
- Local run verified with sample payload (network_automation=false)

## High Priority (Next)
- Collect API creds: Synergy (key, Sophos public IP), UniFi (URL/user/pass/site), Sophos XG (URL/user/pass)
- In AWX: create credentials using custom types and attach to Job Template
- End-to-end AWX run with network_automation=true
- Align payloads/endpoints to live APIs (adjust includes if needed)

## Medium Priority
- Step 0: Azure Key Vault integration (secret generation/storage)
- Steps 6–10: Hyper‑V VM creation, DB/API/App deploys, validation
- Sophos: fully clone WAF template rules (copy all relevant fields)
- Rollback tasks for DNS/UniFi/Sophos

## Low Priority
- Portal → AWX webhook/API integration
- Monitoring/alerting hooks
- Expanded docs and training

## Open Questions
- Synergy API auth and exact record endpoints in prod
- UniFi controller site and token/cookie auth specifics
- Sophos XG WAF policy fields to mirror from template
- Hyper‑V automation method (WinRM, ansible.windows)

## Gaps and Enhancements To Add (from scope review)
- Preflight & idempotence [High]
  - Add a preflight play to ping Synergy, UniFi (site selection), and Sophos; fail fast with clear messages.
  - Implement get‑then‑create/patch logic for DNS, VLANs, hosts, WAF rules to prevent duplicates on re‑runs.

- UniFi site & auth nuances [High]
  - Parameterize site name (default vs custom), validate site exists; handle CSRF/cookie consistently.

- Sophos XG specifics [High]
  - Store “template” WAF policies (e.g., 149/150) as exported XML to clone full settings, not only name/domain.
  - Reuse a single login/session per run instead of embedding <Login> in every call.
  - Add rollback tasks to delete WAF objects/hosts/VLAN if later steps fail.

- DNS verification & propagation [Medium]
  - Use authoritative lookup (Synergy NS) and honor TTLs; avoid fixed sleeps.

- Schema & payload versioning [High]
  - Introduce payload schema_version and extend JSON Schema accordingly; ensure backward‑compatible validation.

- Secrets & Execution Environment [High]
  - Define an Execution Environment with required collections and libs (ansible.utils, community.general, community.dns, jsonschema).
  - Document mapping from custom AWX credentials → env vars used in tasks.

- AWX objects & safety rails [Medium]
  - Define Project, EE, Inventory/Instance Group routing, and Job Templates per step.
  - Create a Workflow with an approval gate before any WAF‑facing changes.

- Naming/uniqueness & concurrency [Medium]
  - Enforce reserved naming (MYCC-<code>-*) and add a run lock/"active deployment" tag to prevent concurrent runs per customer.

- Certificates & WAF health checks [Medium]
  - Plan certificate provisioning/attachment for app and api hostnames; configure WAF backend health monitors.

- Networking dependencies [Medium]
  - Ensure Sophos VLAN routing/NAT/firewall rules are explicitly created for required east‑west and north‑south flows.

- Decommission path [Medium]
  - Add teardown for UniFi VLAN, Sophos WAF objects/hosts/VLAN, and DNS, mirroring creation in reverse order.

- Monitoring & notifications [Low]
  - Configure AWX notifications (Slack/email) per step outcome for visibility.

- Hyper‑V step clarity [High]
  - Capture base image/template, OS customization (sysprep/cloud‑init), domain join, and networking specifics.

- Safety/test modes [High]
  - Add a global dry_run/check‑mode and lab/prod switch to short‑circuit WAF/DNS in lab environments.

## Status Checklist (1–8)
1. Project scaffold and config — Completed
2. Collections requirements file — Completed
3. Orchestrator with schema validation — Completed
4. Include task files (DNS, UniFi, Sophos) — Completed
5. AWX custom credential types and setup playbook — Completed
6. AWX credentials created and attached — Pending
7. End-to-end AWX run with real APIs — Pending
8. Implement VM/app deploy steps (6–10) — Completed

## How to Run
- Local (no external APIs):
  ```sh
  ansible-playbook playbooks/task-master-orchestrator.yml \
    -e "deploy_dataset={{ lookup('file', 'examples/sample_deploy_dataset.json') | from_json }}" \
    -e network_automation=false
  ```
- AWX setup:
  ```sh
  export AWX_HOST=https://awx.example.com
  export AWX_USERNAME=admin
  export AWX_PASSWORD=changeme
  ansible-playbook awx/setup_awx.yml
  ```
- In AWX: add credentials (custom types), attach to Job Template, launch with payload.

## Hardening & Idempotence Checklist
- General
  - [ ] Orchestrator supports dry_run/check-mode and lab/prod guard
  - [ ] Clear fail-fast preflight (Synergy/UniFi/Sophos reachability)
  - [ ] All external calls time out and retry with backoff
  - [ ] All tasks are idempotent (creates or updates only when needed)
  - [ ] Concurrency control (lock per company_code)
- Schema & Inputs
  - [ ] schema_version present and validated
  - [ ] Backward-compatible JSON Schema with defaults
- DNS (Synergy)
  - [ ] Get-then-create/patch A records (no duplicates)
  - [ ] Authoritative DNS verification (query Synergy NS)
  - [ ] TTL-aware propagation waits
- UniFi
  - [ ] Site name parameterized and validated
  - [ ] Reuse auth/session; CSRF/cookie handled
  - [ ] VLAN get-or-create; idempotent subnet/domain settings
  - [ ] Teardown path (delete VLAN/network)
- Sophos XG
  - [ ] Single login/session reuse
  - [ ] Import/copy WAF templates (full policy fields)
  - [ ] Hosts/Web servers get-or-create; idempotent updates
  - [ ] VLAN interface idempotent (exists/enabled)
  - [ ] Routing/NAT/firewall rules explicitly created and idempotent
  - [ ] Rollback path for WAF/hosts/VLAN
- Hyper‑V
  - [ ] Base image/template defined
  - [ ] OS customization (sysprep/cloud-init) documented
  - [ ] Domain join and NIC/VLAN config specified
  - [ ] VM get-or-create with hardware updates as needed
  - [ ] Teardown path for VMs
- Certificates & WAF health
  - [ ] Cert provisioning/attachment steps defined
  - [ ] Backend health checks configured and validated
- AWX/EE
  - [ ] Execution Environment defined with required collections/libs
  - [ ] Credential injectors-to-env mapping documented
  - [ ] Workflow with approval gate before WAF/DNS
  - [ ] Notifications (Slack/email) configured
