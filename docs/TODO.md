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
