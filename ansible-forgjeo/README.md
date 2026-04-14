# 🏗️ ansible-forgjeo — Deploy Forgejo on Debian

Ansible playbook that deploys a [Forgejo](https://forgejo.org/) instance on a remote Debian server.

## 📁 Project Structure

```
ansible-forgjeo/
├── ansible.cfg              # Ansible configuration
├── hosts.ini                # Inventory file
├── playbooks/
│   └── deploy_forgejo.yml   # Main playbook
├── roles/
│   ├── update/
│   │   ├── tasks/main.yml   # Apt package updates
│   │   ├── handlers/main.yml
│   │   ├── vars/main.yml
│   │   └── meta/main.yml
│   ├── security/
│   │   ├── tasks/main.yml   # fail2ban & unattended-upgrades
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   └── jail.local.j2
│   │   ├── vars/main.yml
│   │   └── meta/main.yml
│   └── forgejo/
│       ├── tasks/main.yml   # Forgejo binary, user, dirs & service
│       ├── handlers/main.yml
│       ├── vars/main.yml
│       └── meta/main.yml
└── README.md
```

## ✅ Prerequisites

- Ansible >= 2.9
- SSH access to the target Debian server with root privileges
- Python installed on the target host (required by Ansible)

## ⚙️ Setup

1. Copy the example inventory and fill in your server details:

```bash
cp hosts.ini.example hosts.ini
```

2. Edit `hosts.ini` replacing the IP address, SSH user, port, and key path to match your configuration.

## 🚀 Usage

Run the playbook:

```bash
ansible-playbook -i hosts.ini playbooks/deploy_forgejo.yml
```

Dry run (no changes applied):

```bash
ansible-playbook -i hosts.ini playbooks/deploy_forgejo.yml --check
```

## 📖 What the Playbook Does

### 🔄 Role: update

Updates the system before deployment:

1. **Update apt cache**: downloads the latest package indexes from all configured repositories.
2. **Upgrade all packages (dist-upgrade)**: upgrades every installed package to the latest available version, handling new or removed dependencies.
3. **Remove unneeded packages**: purges orphaned dependencies including their configuration files.
4. **Clean apt cache**: removes obsolete `.deb` files from `/var/cache/apt/archives/` to free disk space.
5. **Check if reboot is required**: checks `/var/run/reboot-required` and notifies the operator if the server needs a reboot.

### 🛡️ Role: security

Server hardening via fail2ban and unattended-upgrades.

**🚫 fail2ban** — brute-force protection:

1. **Install fail2ban**: installs the package via apt.
2. **Deploy jail.local configuration**: copies the `jail.local.j2` template to `/etc/fail2ban/jail.local`. Default configuration bans a host for 1 hour after 5 failed attempts within 10 minutes. SSH jail is enabled.
3. **Enable and start fail2ban**: starts the daemon and enables it at boot.
4. **Verify status**: runs `fail2ban-client status` and displays the output to confirm the service is active and jails are loaded.

**📦 unattended-upgrades** — automatic security updates:

1. **Install unattended-upgrades**: installs the package via apt.
2. **Enable configuration**: runs `dpkg-reconfigure` to activate automatic updates.
3. **Enable and start the daemon**: starts the service and enables it at boot.
4. **Verify configuration**: checks that `APT::Periodic::Unattended-Upgrade` is active.

### 🍵 Role: forgejo

Installs the Forgejo binary, its dependencies, and prepares the working directories and service:

1. **Install git and git-lfs**: installs the packages required by Forgejo via apt.
2. **Version comparison**: checks if the binary already exists and extracts the installed version. Displays a comparison between the installed version and the desired version (defined in `vars/main.yml`).
3. **Interactive confirmation**: prompts the operator for permission before proceeding with the download. The message varies depending on whether it is a fresh install, an upgrade, or a re-download of the same version.
4. **Download Forgejo binary**: downloads the desired version from Codeberg to `/usr/local/bin/forgejo` with `chmod 755`. The download only proceeds if the operator confirms.
5. **Verify installation**: runs `forgejo --version` and displays the output.
6. **Create `git` system group and user**: system user with `/bin/bash` shell and `/home/git` home directory, no password.
7. **Create data directory `/var/lib/forgejo`**: ownership `git:git`, permissions `750`.
8. **Create configuration directory `/etc/forgejo`**: ownership `root:git`, permissions `770`.
9. **Download systemd service unit**: downloads the official unit file from Codeberg to `/etc/systemd/system/forgejo.service` and reloads the systemd daemon.
10. **Enable and start Forgejo**: enables the service at boot and starts it.
