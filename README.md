# Samba 4 Active Directory Domain Controller Installation Guide

## 📋 Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Post-Installation Configuration](#post-installation-configuration)
- [Management Commands](#management-commands)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Best Practices](#best-practices)
- [References](#references)

## 🔰 Introduction

This guide provides comprehensive instructions for installing Samba 4 as an Active Directory Domain Controller (AD DC) on Ubuntu Linux. Samba is an open-source implementation of the SMB/CIFS networking protocol that provides file and print services, authentication/authorization, and name resolution services. Samba 4 includes full AD DC capabilities compatible with Microsoft Active Directory.

### ✨ Key Features
- Domain Controller functionality
- DNS server with internal backend
- Kerberos authentication
- Group Policy support
- Interoperability with Windows clients/servers

## 📝 Prerequisites

Before starting the installation, ensure you have:

- **Ubuntu 22.04 LTS server**
- **Static IP configuration**
- **Hostname configured** (e.g., soc6)
- **Root privileges**
- **Minimum 4GB RAM** (recommended)
- **Proper time synchronization** (NTP)

## 🚀 Installation Steps

### Step 1: Install Required Packages

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install samba winbind krb5-config krb5-user dnsutils -y
```

### Step 2: Prepare Environment

```bash
# Backup existing configuration
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak

# Remove old data
sudo rm -rf /var/lib/samba/private/*
```

### Step 3: Provision the Domain

```bash
sudo samba-tool domain provision \
  --use-rfc2307 \
  --realm=VGIPL.LOCAL \
  --domain=VGIPL \
  --server-role=dc \
  --dns-backend=SAMBA_INTERNAL \
  --adminpass='YourSecurePassword123!'
```

#### Parameters Explanation:
- `--use-rfc2307`: Add POSIX attributes to AD schema
- `--realm`: Kerberos realm (UPPERCASE)
- `--domain`: NetBIOS domain name
- `--server-role=dc`: Configure as Domain Controller
- `--dns-backend=SAMBA_INTERNAL`: Use built-in DNS server
- `--adminpass`: Domain Administrator password (meet complexity requirements)

### Step 4: Configure Kerberos

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
```

### Step 5: Enable and Start Services

```bash
sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc
sudo systemctl start samba-ad-dc
```

### Step 6: Verify Installation

```bash
# Check service status
sudo systemctl status samba-ad-dc

# Verify domain information
sudo samba-tool domain info 127.0.0.1

# Test DNS resolution (replace with your domain)
host -t A soc6.vgipl.local
```

## ⚙️ Post-Installation Configuration

### 1. Firewall Configuration

Allow required ports:

```bash
sudo ufw allow 53/tcp    # DNS
sudo ufw allow 53/udp
sudo ufw allow 88/tcp    # Kerberos
sudo ufw allow 88/udp
sudo ufw allow 135/tcp   # RPC
sudo ufw allow 137-138/udp # NetBIOS
sudo ufw allow 139/tcp
sudo ufw allow 389/tcp   # LDAP
sudo ufw allow 445/tcp   # SMB
sudo ufw allow 464/tcp   # Kerberos password change
sudo ufw allow 636/tcp   # LDAPS
sudo ufw allow 3268/tcp  # Global Catalog
sudo ufw allow 3269/tcp
```

### 2. DNS Configuration

Update `/etc/resolv.conf`:

```text
nameserver 127.0.0.1
search vgipl.local
```

### 3. Test Authentication

```bash
sudo kinit administrator@VGIPL.LOCAL
sudo klist
```

## 🛠️ Management Commands

### User Management

```bash
# Create user
sudo samba-tool user add username

# Reset password
sudo samba-tool user setpassword username

# List users
sudo samba-tool user list
```

### Group Management

```bash
# Create group
sudo samba-tool group add "Group Name"

# Add members to group
sudo samba-tool group addmembers "Domain Admins" administrator
```

### DNS Management

```bash
# List DNS zones
sudo samba-tool dns zone list 127.0.0.1

# Add DNS record
sudo samba-tool dns add 127.0.0.1 vgipl.local newhost A 192.168.1.10
```

## 🔧 Troubleshooting

### Common Issues

#### DNS Resolution Failures:
- Verify `/etc/resolv.conf` points to DC
- Check zone creation: `sudo samba-tool dns zonelist 127.0.0.1`

#### Service Start Failures:
- Check logs: `journalctl -u samba-ad-dc -f`
- Verify configuration: `sudo testparm`

#### Time Synchronization Issues:
- Ensure NTP is working: `timedatectl status`
- Max allowed time skew: 5 minutes

### Log Locations
- **Samba logs**: `/var/log/samba/`
- **Detailed debug logs**: `journalctl -u samba-ad-dc`

## ⚠️ Limitations

- Supports up to 100,000 objects per domain
- Limited Group Policy Object (GPO) support compared to Windows
- No support for Active Directory Federation Services (ADFS)
- Requires manual configuration for complex DNS setups
- Certificate Services require additional configuration

## 📚 Best Practices

- **Use strong passwords** (14+ characters, complexity)
- **Regular backups**:
  ```bash
  sudo samba-tool domain backup offline --target=backup_directory
  ```
- **Maintain time synchronization**
- **Use dedicated DNS servers** for large deployments
- **Implement regular security updates**

## 📖 References

- [Official Samba Documentation](https://www.samba.org/samba/docs/)
- [Samba AD DC Deployment Guide](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
- [RFC2307 Schema Extension](https://tools.ietf.org/html/rfc2307)

---

## 📄 License

This documentation is provided under the MIT License. Feel free to use, modify, and distribute as needed.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request or open an Issue for any improvements or corrections.

## 📞 Support

For issues and questions:
1. Check the [troubleshooting section](#troubleshooting)
2. Review the [official Samba documentation](https://www.samba.org/samba/docs/)
3. Open an issue in this repository
