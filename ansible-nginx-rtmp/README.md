# ansible-nginx-rtmp

I create a Playbook Ansible to configure RTMP Nginx server on Alpine Linux.

## Requirements

- Ansible >= 2.14
- Collection `community.general`: `ansible-galaxy collection install community.general`
- Alpine Linux on server target
- SSH Key configured on target server

## Setup
```bash
cp hosts.ini.example hosts.ini
cp group_vars/all.yml.example group_vars/all.yml
# Edit hosts.ini and group_vars/all.yml 
```

## Exe
```bash
ansible-playbook -i hosts.ini playbook.yml
```

## Stream

**Push:**
