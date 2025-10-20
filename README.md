# MyCareCRM AWX Deployment Template

This provides an AWX-ready Ansible workflow to automate MyCareCRM deployments: DNS (Synergy), UniFi VLAN, and Sophos Firewall. VM and app deploy steps are placeholders.

## Included
- Orchestrator: `playbooks/task-master-orchestrator.yml`
- Includes: `includes/synergy-dns-creation.yml`, `includes/unifi-vlan-setup.yml`, `includes/sophos-firewall-config.yml`, `includes/sophos-vlan-creation.yml`
- Schema: `schemas/deploy_dataset.schema.json`
- AWX setup: `awx/setup_awx.yml`
- Credential types: `awx/credential_types/*`
- Sample payload: `examples/sample_deploy_dataset.json`

## Setup
1) Install collections:
```
ansible-galaxy collection install -r collections/requirements.yml
```
2) Configure AWX:
```
export AWX_HOST=https://awx.example.com
export AWX_USERNAME=admin
export AWX_PASSWORD=changeme
ansible-playbook awx/setup_awx.yml
```
3) In AWX, create credentials for Synergy, UniFi, Sophos using the custom types and attach them to the Job Template.

## Run
Launch the job template and paste JSON in the survey, or run locally:
```
ansible-playbook playbooks/task-master-orchestrator.yml \
  -e "deploy_dataset={{ lookup('file', 'examples/sample_deploy_dataset.json') | from_json }}"
```

Env vars (injected by AWX creds): SYNERGY_API_KEY, SOPHOS_PUBLIC_IP, UNIFI_CONTROLLER_URL, UNIFI_USERNAME, UNIFI_PASSWORD, UNIFI_SITE, SOPHOS_API_URL, SOPHOS_USERNAME, SOPHOS_PASSWORD.

## Dry run / local test
- Syntax check only:
```
ansible-playbook --syntax-check playbooks/task-master-orchestrator.yml
```
- Dry run with sample payload, skip external APIs (recommended locally):
```
ansible-playbook playbooks/task-master-orchestrator.yml \
  -e "deploy_dataset={{ lookup('file', 'examples/sample_deploy_dataset.json') | from_json }}" \
  -e network_automation=false \
  --check
```
Notes:
- The `network_automation=false` toggle skips DNS/UniFi/Sophos steps.
- `--check` is optional; with network steps skipped, the play prints plans and validation without changing anything.
- In AWX, set an extra var `network_automation=false` (or add a survey question) for a safe dry run.

## Notes
- APIs may need tweaks to match actual versions.
- Sophos rule cloning is minimal; extend as needed.
- TLS validation is disabled for UniFi/Sophos by default.

## Next steps
- Azure Key Vault integration
- Hyper‑V VM creation and app deploy steps
- Rollback and monitoring