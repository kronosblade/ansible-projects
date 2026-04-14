# 📦 ansible-playbooks

A collection of Ansible playbooks I'm writing while learning infrastructure automation as a Linux sysadmin.

Each directory is a standalone playbook targeting a specific service or configuration task. I'm using this repository to build real automation for services I actually run and manage.

## 📂 Playbooks

| Directory | Description |
|---|---|
| `ansible-mumble-debian-server` | Deploy and configure a Mumble on Debian server |
| `ansible-forgjeo` | Deploy and configure a Forgjeo on Debian server |
| `ansible-nginx-rtmp` | Set up Nginx with the RTMP module for live streaming |

## 🛠️ Stack

- **Control node**: Linux workstation
- **Managed nodes**: Debian / AlmaLinux / Alpine Linux
- **Ansible version**: community.general collection

## 🚧 Work in progress

I'm actively learning Ansible and expanding this collection. Playbooks may be rough around the edges improvements and restructuring happen as my knowledge grows.

## 📄 License

See [LICENSE](./LICENSE).
