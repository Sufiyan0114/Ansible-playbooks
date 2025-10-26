# 🔒 Server Hardening with Ansible

> A production-ready Ansible playbook for automating server security hardening on Ubuntu servers.

[![Ansible](https://img.shields.io/badge/Ansible-2.9+-green.svg)](https://www.ansible.com/)
[![License](https://img.shields.io/badge/license-Custom-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Ubuntu-orange.svg)](https://ubuntu.com/)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [What This Playbook Does](#-what-this-playbook-does)
- [Security Considerations](#-security-considerations)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

This Ansible playbook automates the process of hardening Ubuntu servers by implementing industry-standard security best practices. It configures firewall rules, sets up intrusion detection, and hardens SSH access to protect your infrastructure from common attack vectors.

### Why Use This Playbook?

- ✅ **Automated Security**: Apply security configurations consistently across multiple servers
- ✅ **Idempotent**: Safe to run multiple times without side effects
- ✅ **Compliance**: Follows security best practices for server hardening
- ✅ **Time-Saving**: Eliminate manual server hardening tasks
- ✅ **Version Controlled**: Track all security changes in your infrastructure

---

## ✨ Features

### 🛡️ Firewall Configuration
- Configures UFW (Uncomplicated Firewall) with strict rules
- Denies all incoming traffic by default
- Allows only specified ports (SSH, HTTPS)
- Enables outgoing traffic for system functionality

### 🔐 Security Hardening
- **Fail2Ban**: Automated intrusion detection and prevention
- **SSH Hardening**: 
  - Disables root login
  - Disables password authentication
  - Enforces key-based authentication only
- **User Management**: Creates secure sudo user with proper permissions

### 🚀 System Updates
- Ensures all packages are up-to-date
- Implements apt cache optimization

---

## 📦 Prerequisites

Before using this playbook, ensure you have:

### On Your Control Machine:
- **Ansible** 2.9 or higher
  ```bash
  pip install ansible
  # or
  sudo apt-get install ansible
  ```

### On Target Servers:
- Ubuntu 18.04+ or Debian 10+
- Python 3 installed (`/usr/bin/python3`)
- SSH access with sudo privileges
- SSH key-based authentication configured

### Required Permissions:
- SSH access to target servers
- Sudo privileges (configured to use passwordless sudo is recommended)

---

## 🏗️ Project Structure

```
Server-hardening/
│
├── ansible.cfg              # Ansible configuration file
├── inventory.ini            # Server inventory definition
├── vars.yml                 # Configuration variables
├── playbook.yaml            # Main playbook file
├── ssh-private-key          # SSH private key for authentication
│
└── roles/
    ├── firewall/            # Firewall configuration role
    │   └── tasks/
    │       └── main.yaml
    │
    └── security/            # Security hardening role
        ├── tasks/
        │   └── main.yaml
        └── handlers/
            └── main.yaml
```

---

## ⚙️ Configuration

### 1. Inventory Setup

Edit `inventory.ini` to configure your target servers:

```ini
[webserver]
server1 ansible_host=YOUR_SERVER_IP ansible_user=ubuntu

[webserver:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=/path/to/your/private/key
```

### 2. Variable Configuration

Customize `vars.yml` to match your security requirements:

```yaml
# Define allowed ports with their protocols
allowed_ports:
  - { port: "22", proto: "tcp" }    # SSH
  - { port: "443", proto: "tcp" }   # HTTPS
  - { port: "80", proto: "tcp" }    # HTTP (optional)
  - { port: "3306", proto: "tcp" }  # MySQL (optional)

# Firewall default policies
ufw_default_incoming: deny    # Block all incoming by default
ufw_default_outgoing: allow   # Allow all outgoing traffic
```

### 3. Security Settings

In `roles/security/tasks/main.yaml`, you can customize:

- **User creation**: Modify the sudo user name and groups
- **Fail2Ban**: Adjust jail rules in the Fail2Ban configuration
- **SSH settings**: Modify SSH daemon configuration parameters

---

## 🚀 Usage

### 1. Initial Setup

Clone this repository:

```bash
git clone <repository-url>
cd Server-hardening
```

### 2. Test Connectivity

Verify connectivity to your servers:

```bash
ansible all -m ping
```

### 3. Run the Playbook

Execute the playbook:

```bash
# Run on all servers
ansible-playbook playbook.yaml

# Run on specific hosts
ansible-playbook playbook.yaml --limit webserver

# Run with verbose output
ansible-playbook playbook.yaml -v

# Run with check mode (dry-run)
ansible-playbook playbook.yaml --check
```

### 4. Verify Changes

After running the playbook, verify the configuration:

```bash
# SSH into your server
ssh user@your-server-ip

# Check UFW status
sudo ufw status verbose

# Check Fail2Ban status
sudo systemctl status fail2ban

# Verify SSH configuration
sudo sshd -T | grep -E "PermitRootLogin|PasswordAuthentication"
```

---

## 🔍 What This Playbook Does

### Firewall Role (`roles/firewall/`)

1. Updates apt cache
2. Installs UFW firewall package
3. Resets UFW to default settings
4. Configures default policies:
   - Incoming: Deny
   - Outgoing: Allow
5. Opens specified ports (SSH, HTTPS)
6. Enables UFW firewall
7. Displays firewall status

### Security Role (`roles/security/`)

1. **Fail2Ban Setup**:
   - Installs Fail2Ban package
   - Starts and enables the service
   - Protects against brute-force attacks

2. **User Management**:
   - Creates a new sudo user
   - Configures proper group membership
   - Sets up bash shell

3. **SSH Hardening**:
   - Disables root login (`PermitRootLogin no`)
   - Disables password authentication (`PasswordAuthentication no`)
   - Restarts SSH service to apply changes

---

## 🔒 Security Considerations

### ⚠️ Important Warnings

1. **SSH Configuration**: 
   - After disabling root login and password authentication, ensure you have:
     - An existing non-root user with sudo privileges
     - SSH key-based authentication configured
   - Otherwise, you may lock yourself out of the server!

2. **Firewall Rules**:
   - Only ports in `allowed_ports` will be accessible
   - Ensure critical services are included before running

3. **User Creation**:
   - The playbook creates a specific user (default: `sufiyan`)
   - Modify this in `roles/security/tasks/main.yaml` to match your needs

### 🛡️ Security Best Practices

- [ ] Use SSH keys instead of passwords
- [ ] Limit SSH access by IP if possible
- [ ] Regularly update the playbook and systems
- [ ] Monitor Fail2Ban logs (`/var/log/fail2ban.log`)
- [ ] Review UFW rules after deployment
- [ ] Implement regular security audits

---

## 🐛 Troubleshooting

### Common Issues

#### Issue: Connection timeout
```bash
# Solution: Verify network connectivity and SSH configuration
ansible all -m ping --verbose
```

#### Issue: Permission denied
```bash
# Solution: Ensure SSH key has correct permissions
chmod 600 /path/to/your/ssh-key
```

#### Issue: Locked out of server
```bash
# Solution: Use console access or reset SSH configuration
# The server should still allow access via console
```

#### Issue: UFW not enabling
```bash
# Solution: Check for conflicting firewall rules
sudo ufw status
sudo iptables -L -n
```

### Debug Mode

Run playbook with maximum verbosity:

```bash
ansible-playbook playbook.yaml -vvv
```

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/AmazingFeature`)
3. **Commit** your changes (`git commit -m 'Add some AmazingFeature'`)
4. **Push** to the branch (`git push origin feature/AmazingFeature`)
5. **Open** a Pull Request

### Contribution Guidelines

- Follow Ansible best practices
- Add comments to complex tasks
- Test your changes before submitting
- Update documentation as needed
- Follow the existing code style

---

## 📄 License

This project is licensed under the Custom License - see the [LICENSE](LICENSE) file for details.

---

## 📞 Support

For issues, questions, or contributions:

- Open an issue on GitHub
- Review existing documentation
- Check Ansible documentation for reference

---

## Acknowledgments

- Built with [Ansible](https://www.ansible.com/)
- Inspired by Ubuntu security best practices
- Community contributions are welcome

---

<div align="center">

**⚠️ Use at your own risk. Always test in a non-production environment first.**

Made with ❤️ for secure infrastructure

</div>
