# k8s-ansible-config

<<<<<<< HEAD
Ansible configuration for the EKS worker nodes (Ubuntu 24.04 managed node group).

Based on a classmate's role layout; adapted for EKS: SSM connection, tag-based
dynamic inventory, no host firewall, containerd/kubelet verified (not rewritten),
and monitoring roles that render config for the in-cluster Prometheus/Alertmanager.

## Roles
| Role | Runs on | What it does |
|---|---|---|
| node-setup | workers | kernel modules/sysctl, SSH hardening drop-in (if sshd exists), unattended security upgrades, checks containerd + kubelet |
| node-exporter | workers | installs Node Exporter as a hardened systemd service (port 9100) |
| prometheus-config | control machine | renders scrape targets from the live inventory, optional Secret in cluster |
| alertmanager-config | control machine | renders PrometheusRule + Alertmanager Slack config (optional apply) |

## Prerequisites (control machine)
```bash
sudo apt-get install -y ansible python3-boto3 python3-botocore
ansible-galaxy collection install -r requirements.yml
# AWS Session Manager plugin: https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-debian-and-ubuntu.html
export ANSIBLE_SSM_BUCKET=<existing S3 bucket>
export AWS_DEFAULT_REGION=ap-south-1
```
Workers need the SSM policy (added in Terraform) and must show `Online` in SSM.

## Run
```bash
ansible-inventory --graph
ansible kubernetes_nodes -m ping
ansible-playbook playbooks/site.yml --syntax-check
ansible-playbook playbooks/site.yml --check --diff
ansible-playbook playbooks/site.yml
```
Output for the monitoring stack goes to `rendered/` (git-ignored, contains the webhook).
Once the monitoring stack exists, add `-e monitoring_apply_to_cluster=true`.

## Secrets
Keep the Slack webhook out of Git, use Ansible Vault:
```bash
ansible-vault encrypt_string --name alertmanager_slack_webhook_url 'https://hooks.slack.com/services/REPLACE'
```
Paste the result into `inventory/group_vars/all.yml` and run with `--ask-vault-pass`.

## Notes
- Turn off the Helm chart's own node-exporter (`nodeExporter.enabled: false`) to avoid a port 9100 clash.
- No UFW: security groups handle network access, a host deny-all rule can break kubelet/pod traffic.

EKS: SSM connection, tag-based dynamic inventory, no host firewall,
containerd/kubelet verification, and monitoring roles that render config for in-cluster Prometheus.
=======
Ansible roles for the EC2 worker nodes:
node-setup, node-exporter, prometheus-config, alertmanager-config.

