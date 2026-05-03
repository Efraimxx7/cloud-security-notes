# 🐧 Linux for Cloud Security — Essential Commands

> A practical reference of Linux commands used in Cloud Security workflows —  
> covering auditing, hardening, network analysis, permissions, incident response, and monitoring.

---

## 📋 Table of Contents

- [Investigation & Auditing](#-investigation--auditing)
- [Network Analysis](#-network-analysis)
- [Users, Groups & Permissions](#-users-groups--permissions)
- [Processes & Services](#-processes--services)
- [File System & Integrity](#-file-system--integrity)
- [SSH & Cryptography](#-ssh--cryptography)
- [Firewall & Access Control](#-firewall--access-control)
- [Logs & Monitoring](#-logs--monitoring)
- [Package Management & Hardening](#-package-management--hardening)
- [Incident Response](#-incident-response)
- [Quick Reference — Security Best Practices](#-quick-reference--security-best-practices)
- [Recommended Tools](#-recommended-tools)

---

## 🔍 Investigation & Auditing

> Used to understand what is running on a system, who has access, and what has changed.

```bash
# List all system users
cat /etc/passwd

# Find users with UID 0 (root-level access)
grep ':0:' /etc/passwd

# View command history of the current user
history
cat ~/.bash_history

# View history of another user (requires root)
cat /home/username/.bash_history

# See who is currently logged in
who
w

# See recent login sessions
last

# See failed login attempts
lastb

# View authentication logs (Debian/Ubuntu)
cat /var/log/auth.log

# View authentication logs via systemd
journalctl -u ssh

# View all sudo usage
grep 'sudo' /var/log/auth.log

# Check scheduled tasks (possible persistence mechanism)
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
```

---

## 🌐 Network Analysis

> Identify open ports, active connections, and suspicious traffic.

```bash
# List all open ports and listening services
ss -tulnp
netstat -tulnp

# Show all active network connections
ss -anp
netstat -anp

# Display network interfaces and IP addresses
ip a
ifconfig

# Show routing table
ip route
route -n

# Test connectivity
ping -c 4 8.8.8.8
traceroute google.com

# DNS lookup
nslookup domain.com
dig domain.com

# Scan open ports on a host
nmap -sV 192.168.1.1

# Scan for common vulnerabilities (safe scripts)
nmap --script vuln 192.168.1.1

# Capture network traffic on an interface
tcpdump -i eth0
tcpdump -i eth0 port 443
tcpdump -i eth0 -w capture.pcap

# Check which process is using a specific port
lsof -i :22
lsof -i :80
lsof -i :443

# Show ARP table (map IPs to MAC addresses)
arp -a
```

---

## 👤 Users, Groups & Permissions

> Misconfigured users and permissions are among the most common attack vectors.

```bash
# Show current user and UID
whoami
id

# List groups of current user
groups

# List all groups on the system
cat /etc/group

# Add a user
useradd -m username

# Delete a user and their home directory
userdel -r username

# Lock a user account (disable login)
passwd -l username

# Unlock a user account
passwd -u username

# Change user password
passwd username

# Add user to a group (e.g., sudo)
usermod -aG sudo username

# View sudo privileges for current user
sudo -l

# View the sudoers file
cat /etc/sudoers
visudo

# Check file permissions
ls -la /path/to/file

# Change file permissions
chmod 600 private_key.pem      # owner read/write only
chmod 644 public_file.txt      # owner rw, others read
chmod 700 /secure/directory    # owner full, others none

# Change file owner
chown user:group file.txt

# Find files with SUID bit set (privilege escalation risk)
find / -perm -4000 -type f 2>/dev/null

# Find files with SGID bit set
find / -perm -2000 -type f 2>/dev/null

# Find world-writable files (anyone can modify — security risk)
find / -perm -o+w -type f 2>/dev/null

# Find files owned by no user (orphaned files — risk indicator)
find / -nouser -type f 2>/dev/null
```

---

## ⚙️ Processes & Services

> Identify suspicious processes and manage running services.

```bash
# View all running processes
ps aux

# Interactive process monitor
top
htop

# Find a process by name
ps aux | grep nginx
pgrep -a sshd

# Kill a process by PID
kill -9 <PID>

# Kill a process by name
pkill processname

# View processes listening on ports
ss -tulnp
lsof -i

# List all active services
systemctl list-units --type=service --state=running

# Check status of a service
systemctl status sshd

# Start / stop / restart a service
systemctl start nginx
systemctl stop nginx
systemctl restart nginx

# Enable / disable a service at boot
systemctl enable nginx
systemctl disable nginx

# Reload service configuration without restart
systemctl reload nginx

# View services that start at boot
systemctl list-unit-files --state=enabled
```

---

## 📁 File System & Integrity

> Detect unauthorized changes, malicious files, and data exposure.

```bash
# Generate SHA256 hash of a file (verify integrity)
sha256sum file.txt

# Generate MD5 hash
md5sum file.txt

# Compare hash against a known value
echo "expectedhash  file.txt" | sha256sum --check

# Compare two files
diff file1.txt file2.txt

# Find files modified in the last 24 hours
find / -mtime -1 -type f 2>/dev/null

# Find files modified in the last 10 minutes
find / -mmin -10 -type f 2>/dev/null

# Find files larger than 100MB (suspicious on servers)
find / -size +100M -type f 2>/dev/null

# Search for a specific string inside files (e.g., hardcoded credentials)
grep -r "password" /var/www/
grep -r "AWS_SECRET" /home/
grep -rn "api_key" /opt/

# Find hidden files (start with .)
find / -name ".*" -type f 2>/dev/null

# Check disk usage per directory
du -sh /*
df -h

# List open files by a specific process (useful during incidents)
lsof -p <PID>

# View file metadata
stat file.txt
```

---

## 🔐 SSH & Cryptography

> Secure remote access is fundamental in Cloud Security.

```bash
# Generate an ED25519 SSH key pair (recommended — stronger than RSA)
ssh-keygen -t ed25519 -C "your@email.com"

# Generate RSA key pair (4096 bits minimum if required)
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Copy public key to a remote server
ssh-copy-id user@host

# Connect using a specific private key
ssh -i ~/.ssh/my_key user@host

# Connect on a non-default port
ssh -p 2222 user@host

# Enable SSH agent and add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/my_key

# View loaded keys in the agent
ssh-add -l

# Secure copy a file to/from remote
scp file.txt user@host:/remote/path/
scp user@host:/remote/file.txt ./local/

# View SSH server configuration
cat /etc/ssh/sshd_config

# Key hardening settings to look for/set in sshd_config:
#   PermitRootLogin no
#   PasswordAuthentication no
#   PubkeyAuthentication yes
#   MaxAuthTries 3
#   AllowUsers youruser

# Validate sshd config without restarting
sshd -t

# Restart SSH service after config change
systemctl restart sshd

# Encrypt a file using GPG
gpg -c sensitive_file.txt

# Decrypt a GPG-encrypted file
gpg sensitive_file.txt.gpg

# Generate GPG key pair
gpg --full-generate-key
```

---

## 🛡️ Firewall & Access Control

```bash
# --- UFW (Uncomplicated Firewall — Ubuntu/Debian) ---

# Check firewall status
ufw status verbose

# Enable firewall
ufw enable

# Disable firewall
ufw disable

# Allow SSH (always do this before enabling UFW)
ufw allow ssh
ufw allow 22/tcp

# Allow specific port
ufw allow 443/tcp

# Deny a port
ufw deny 23/tcp

# Allow from a specific IP
ufw allow from 192.168.1.100

# Delete a rule
ufw delete allow 80/tcp

# --- iptables ---

# List all rules
iptables -L -v -n

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Block an IP address
iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Save rules (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4
```

---

## 📊 Logs & Monitoring

> Logs are the primary source of truth during investigations and audits.

```bash
# Follow system logs in real time
journalctl -f

# View logs for a specific service
journalctl -u nginx
journalctl -u sshd -n 100

# View logs since a specific time
journalctl --since "2025-04-01 00:00:00"
journalctl --since "1 hour ago"

# View system log (syslog)
cat /var/log/syslog
tail -f /var/log/syslog

# View authentication log
cat /var/log/auth.log
tail -f /var/log/auth.log

# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log

# Find successful SSH logins
grep "Accepted" /var/log/auth.log

# Count failed login attempts by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# View kernel messages (hardware, driver issues)
dmesg | tail -50

# Monitor CPU, memory, disk in real time
top
htop
iostat
vmstat 1

# Check disk usage
df -h

# Check memory usage
free -h

# Check system uptime and load average
uptime
```

---

## 📦 Package Management & Hardening

```bash
# --- Debian / Ubuntu ---

# Update package list
apt update

# Upgrade all packages (includes security patches)
apt upgrade -y

# Apply only security updates
apt-get upgrade --only-upgrade

# List upgradable packages
apt list --upgradable

# Search for a package
apt search packagename

# Install a package
apt install packagename

# Remove a package completely
apt remove --purge packagename

# List installed packages
dpkg -l

# Check if a specific package is installed
dpkg -l | grep nginx

# --- System Hardening Basics ---

# Disable unused services
systemctl disable bluetooth
systemctl disable cups
systemctl disable avahi-daemon

# Check for world-readable sensitive files
ls -la /etc/shadow
ls -la /etc/passwd

# Verify /etc/shadow permissions (should be 640 or 000)
stat /etc/shadow

# Set immutable flag on critical files (prevents modification)
chattr +i /etc/passwd
chattr +i /etc/shadow

# Remove immutable flag
chattr -i /etc/passwd
```

---

## 🚑 Incident Response

> When a security incident is suspected — act methodically, preserve evidence first.

```bash
# 1. PRESERVE — document the current state before any changes
date > incident_$(date +%Y%m%d).txt
uptime >> incident_$(date +%Y%m%d).txt
who >> incident_$(date +%Y%m%d).txt
ps aux >> incident_$(date +%Y%m%d).txt
ss -tulnp >> incident_$(date +%Y%m%d).txt
last >> incident_$(date +%Y%m%d).txt

# 2. IDENTIFY — find suspicious activity
# Recent files modified
find / -mtime -1 -type f 2>/dev/null

# Processes with unusual names or paths
ps aux | grep -v "root\|daemon\|www-data"

# Connections to unknown external IPs
ss -anp | grep ESTABLISHED

# Unusual cron jobs
crontab -l
cat /etc/crontab

# Check for backdoor users
grep ':0:' /etc/passwd
cat /etc/passwd | awk -F: '$3 == 0'

# 3. CONTAIN — isolate the threat
# Block a specific IP immediately
iptables -A INPUT -s <suspicious-IP> -j DROP
iptables -A OUTPUT -d <suspicious-IP> -j DROP

# Lock a compromised user account
passwd -l username

# Kill a suspicious process
kill -9 <PID>

# 4. INVESTIGATE — trace the attack
# Search logs for the attacker's IP
grep "<suspicious-IP>" /var/log/auth.log
grep "<suspicious-IP>" /var/log/syslog

# Check recently created files
find / -newer /tmp/reference_file -type f 2>/dev/null

# Look for files with suspicious extensions
find / -name "*.sh" -newer /etc/passwd 2>/dev/null

# 5. RECOVER — clean and harden
# Remove unauthorized SSH keys
cat ~/.ssh/authorized_keys
# Review and remove any unknown keys

# Rotate credentials
passwd username

# Review and patch the exploited service
apt update && apt upgrade -y
```

---

## 📌 Quick Reference — Security Best Practices

| Practice | Command / Action |
|---|---|
| Disable root SSH login | Set `PermitRootLogin no` in `/etc/ssh/sshd_config` |
| Use SSH keys, not passwords | Set `PasswordAuthentication no` in sshd_config |
| Find SUID binaries | `find / -perm -4000 -type f 2>/dev/null` |
| Check open ports | `ss -tulnp` |
| Monitor failed logins | `grep "Failed password" /var/log/auth.log` |
| Hash verification | `sha256sum file.txt` |
| Lock unused accounts | `passwd -l username` |
| Keep system updated | `apt update && apt upgrade -y` |
| Use ED25519 keys | `ssh-keygen -t ed25519` |
| Audit cron jobs | `crontab -l && cat /etc/crontab` |

---

## 🛠️ Recommended Tools

| Tool | Purpose |
|---|---|
| [Lynis](https://cisofy.com/lynis/) | System security auditing and hardening |
| [Fail2Ban](https://www.fail2ban.org/) | Block IPs after repeated failed logins |
| [OSSEC](https://www.ossec.net/) | Host-based intrusion detection (HIDS) |
| [ClamAV](https://www.clamav.net/) | Open-source antivirus for Linux |
| [auditd](https://linux.die.net/man/8/auditd) | Linux Audit Daemon — track system calls |
| [rkhunter](https://rkhunter.sourceforge.net/) | Rootkit detection |
| [chkrootkit](https://www.chkrootkit.org/) | Check for known rootkits |
| [Wireshark](https://www.wireshark.org/) | Network packet analysis (GUI) |

---

<div align="center">

*Maintained as a personal reference for Cloud Security practices.*  
*Commands tested on Debian/Ubuntu-based Linux distributions.*

</div>
