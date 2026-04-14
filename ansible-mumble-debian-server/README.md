# 🎙️ Ansible Mumble Debian Server

Welcome! This repository is a one-stop shop for deploying a secure, production-ready **Mumble (Murmur)** server on Debian. No more manual configuration—just one command and you're ready to talk. 🚀

## 🤔 Why Ansible?
Setting up a server manually is fine once, but doing it twice is a chore. 
* **Simplicity:** We use human-readable YAML files. If you can read a list, you can understand this setup. 📖
* **Consistency:** Whether you're deploying to one VPS or ten, Ansible ensures they are all configured *exactly* the same way. 🤖
* **Speed:** From a fresh "blank" server to a fully hardened Mumble host in under 2 minutes. ⚡

## 🎯 Why we created this
We wanted a way to deploy Mumble that wasn't just "functional" but also **secure**. 
Public servers are constantly poked by bots. This playbook automatically sets up **Fail2Ban** to watch your logs and kick out anyone trying to brute-force their way into your SSH or Mumble ports. 🛡️

## 📦 What’s under the hood?
When you run this playbook, it handles the following:

* **System Prep:** Updates your `apt` cache and installs essentials like `vim`, `curl`, and `git`. 🛠️
* **Mumble Server:** Installs and configures the Murmur (Mumble) daemon. 🔊
* **Security (Fail2Ban):** * Protects **SSH** (Port 22).
    * Protects **Mumble** (Port 64738) for both TCP and UDP traffic.
    * Includes "auto-healing" logic to create log files so services never crash on startup. 🩹

---

## 🚀 Quick Start

### 1. Requirements
Make sure you have **Ansible** installed on your local machine (your "control node").

### 2. Configure your Inventory
Edit your `inventory` file (or `hosts.ini`) to include your server's IP address:
```ini
[mumble_servers]
your_server_ip_here
```

### 3. Run the Playbook
Fire off the automation with this command:
```sh
ansible-playbook playbooks/setup_mumble_debian.yml -i inventory
```

---

## ✨ Features
* ✅ **Hardened:** Automatic IP banning for bad actors.
* ✅ **Lightweight:** Uses minimal resources on your Debian box.
* ✅ **Ready-to-talk:** Just open your Mumble client and connect! 🎧

**Happy Chatting!** 🌀✨