# Ansible Patching & Configuration Management

## Layout
- `patch_servers.yml`      - pre-checks -> patch -> reboot -> post-checks -> report
- `config_management.yml`  - baseline config (packages, NTP, SSH hardening, sysctl, banner, permissions)
- `group_vars/`            - settings (`all.yml`) and per-group critical services
- `inventory/hosts.ini`    - edit with your servers

## Setup
    ansible-galaxy collection install -r requirements.yml
    ansible all -m ping

## Patching
    ansible-playbook patch_servers.yml --check --diff            # dry run
    ansible-playbook patch_servers.yml --tags precheck           # checks only
    ansible-playbook patch_servers.yml -e target_hosts=webservers
    ansible-playbook patch_servers.yml -e allow_reboot=false
    ansible-playbook patch_servers.yml -e patch_mode=all -e batch_size=1

## What the checks do
Pre:  connectivity, free disk, critical service status, baseline (kernel, listening ports), /etc backup.
Post: host back online, previously-active services active again (with retries), ports listening,
      optional HTTP health check, JSON report in /var/log/patching/.

## Config management
    ansible-playbook config_management.yml --check --diff        # detect drift
    ansible-playbook config_management.yml                       # enforce

## Notes
- Test on a non-production group first. Keep `max_fail_percentage` and `serial` to stop a bad patch early.
- Store secrets with ansible-vault, never in plain vars.
