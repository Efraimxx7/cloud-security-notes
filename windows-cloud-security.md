# 🪟 Windows for Cloud Security — Essential Commands

> A practical reference of Windows commands used in Cloud Security workflows —  
> covering auditing, user management, network analysis, firewall, event logs, and incident response.  
> Includes both **CMD** and **PowerShell** approaches.

---

## 📋 Table of Contents

- [CMD — System Information](#-cmd--system-information)
- [CMD — Users & Groups](#-cmd--users--groups)
- [CMD — Network Analysis](#-cmd--network-analysis)
- [CMD — Processes & Services](#-cmd--processes--services)
- [CMD — File System](#-cmd--file-system)
- [PowerShell — Users & Groups](#-powershell--users--groups)
- [PowerShell — Processes & Services](#-powershell--processes--services)
- [PowerShell — Network & Firewall](#-powershell--network--firewall)
- [PowerShell — Event Logs & Auditing](#-powershell--event-logs--auditing)
- [PowerShell — File Integrity & Scheduled Tasks](#-powershell--file-integrity--scheduled-tasks)
- [PowerShell — Execution Policy & Hardening](#-powershell--execution-policy--hardening)
- [Windows Security Event IDs](#-windows-security-event-ids)
- [Incident Response — Windows](#-incident-response--windows)
- [Quick Reference — Security Best Practices](#-quick-reference--security-best-practices)
- [Recommended Tools](#-recommended-tools)

---

## 💻 CMD — System Information

> Always start an investigation by understanding the machine's current state.

```cmd
:: View full system information (OS version, hostname, domain, patches)
systeminfo

:: View installed patches and hotfixes
wmic qfe list

:: View environment variables
set

:: View current logged-in user
whoami

:: View current user privileges
whoami /priv

:: View groups the current user belongs to
whoami /groups

:: View hostname
hostname

:: View system uptime
net statistics workstation

:: View shared resources on the system
net share

:: View open files over the network
net file
```

---

## 👤 CMD — Users & Groups

```cmd
:: List all local users
net user

:: View details of a specific user
net user username

:: Create a new user
net user newuser Password123! /add

:: Delete a user
net user username /delete

:: Disable a user account
net user username /active:no

:: Enable a user account
net user username /active:yes

:: List all local groups
net localgroup

:: View members of a group
net localgroup Administrators

:: Add a user to a group
net localgroup Administrators username /add

:: Remove a user from a group
net localgroup Administrators username /delete

:: View password policy
net accounts

:: View domain password policy
net accounts /domain
```

---

## 🌐 CMD — Network Analysis

```cmd
:: View IP configuration (full details)
ipconfig /all

:: Release and renew DHCP lease
ipconfig /release
ipconfig /renew

:: Flush DNS cache
ipconfig /flushdns

:: View DNS cache
ipconfig /displaydns

:: View active network connections with PIDs
netstat -ano

:: View listening ports with PIDs
netstat -ano | findstr LISTENING

:: View established connections
netstat -ano | findstr ESTABLISHED

:: View ARP table (IP to MAC mapping)
arp -a

:: View routing table
route print

:: Test connectivity
ping -n 4 8.8.8.8

:: Trace network path to a host
tracert google.com

:: DNS lookup
nslookup google.com

:: Test a specific port (requires PowerShell or telnet)
telnet host 80
```

---

## ⚙️ CMD — Processes & Services

```cmd
:: List all running processes
tasklist

:: Find a specific process by name
tasklist | findstr "chrome"

:: Kill a process by PID
taskkill /PID 1234 /F

:: Kill a process by name
taskkill /IM notepad.exe /F

:: List all services and their states
sc query type= all

:: List only running services
sc query type= all state= running

:: View details of a specific service
sc qc "wuauserv"

:: Start a service
net start "Windows Update"

:: Stop a service
net stop "Windows Update"

:: Disable a service (prevent from starting at boot)
sc config "servicename" start= disabled

:: Enable a service
sc config "servicename" start= auto

:: View startup programs (via registry — requires reg command)
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

---

## 📁 CMD — File System

```cmd
:: List files in a directory with details
dir /a /q C:\path\to\folder

:: List hidden files
dir /ah C:\

:: Search for a file by name
where /r C:\ filename.txt

:: Search for a string inside files
findstr /s /i "password" C:\path\*.txt
findstr /s /i "secret" C:\path\*.config

:: Copy a file
copy source.txt destination.txt

:: Delete a file securely (overwrite before delete)
cipher /w:C:\path\to\folder

:: View file attributes
attrib C:\path\to\file.txt

:: View disk usage
wmic logicaldisk get size,freespace,caption

:: Check file system integrity
sfc /scannow

:: Check and repair disk
chkdsk C: /f /r
```

---

## 👤 PowerShell — Users & Groups

> PowerShell provides richer output and filtering capabilities than CMD.

```powershell
# List all local users
Get-LocalUser

# View a specific user's details
Get-LocalUser -Name "username"

# List all local groups
Get-LocalGroup

# View members of a group
Get-LocalGroupMember -Group "Administrators"

# Create a new local user
New-LocalUser -Name "newuser" -Password (ConvertTo-SecureString "Password123!" -AsPlainText -Force)

# Add user to a group
Add-LocalGroupMember -Group "Administrators" -Member "username"

# Remove user from a group
Remove-LocalGroupMember -Group "Administrators" -Member "username"

# Disable a user account
Disable-LocalUser -Name "username"

# Enable a user account
Enable-LocalUser -Name "username"

# Check current user identity
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name

# Check if running as Administrator
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

---

## ⚙️ PowerShell — Processes & Services

```powershell
# List all running processes
Get-Process

# Find a specific process
Get-Process -Name "notepad"

# View processes sorted by CPU usage
Get-Process | Sort-Object CPU -Descending | Select-Object -First 20

# View processes with their file paths (detect suspicious locations)
Get-Process | Select-Object Name, Id, Path | Sort-Object Name

# Kill a process by name
Stop-Process -Name "notepad" -Force

# Kill a process by PID
Stop-Process -Id 1234 -Force

# List all services
Get-Service

# List only running services
Get-Service | Where-Object { $_.Status -eq "Running" }

# Find a specific service
Get-Service -Name "wuauserv"

# Start / stop / restart a service
Start-Service -Name "wuauserv"
Stop-Service -Name "wuauserv"
Restart-Service -Name "wuauserv"

# Disable a service
Set-Service -Name "Spooler" -StartupType Disabled

# View services set to start automatically
Get-Service | Where-Object { $_.StartType -eq "Automatic" }
```

---

## 🌐 PowerShell — Network & Firewall

```powershell
# View all network adapters and IP addresses
Get-NetIPAddress

# View active TCP connections
Get-NetTCPConnection

# View listening ports
Get-NetTCPConnection -State Listen

# View established connections (detect C2 or exfiltration)
Get-NetTCPConnection -State Established

# Resolve a process PID to name (from network connection)
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess |
  ForEach-Object { $_ | Add-Member -MemberType NoteProperty -Name ProcessName -Value (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name -PassThru }

# View DNS cache
Get-DnsClientCache

# Clear DNS cache
Clear-DnsClientCache

# View routing table
Get-NetRoute

# --- Firewall ---

# View all enabled firewall rules
Get-NetFirewallRule | Where-Object { $_.Enabled -eq "True" }

# View inbound rules only
Get-NetFirewallRule -Direction Inbound | Where-Object { $_.Enabled -eq "True" }

# View outbound rules only
Get-NetFirewallRule -Direction Outbound | Where-Object { $_.Enabled -eq "True" }

# Block inbound traffic on a specific port
New-NetFirewallRule -DisplayName "Block Telnet" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block

# Block outbound traffic to a specific IP
New-NetFirewallRule -DisplayName "Block Suspicious IP" -Direction Outbound -RemoteAddress 203.0.113.5 -Action Block

# Allow a specific application through the firewall
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow

# Remove a firewall rule
Remove-NetFirewallRule -DisplayName "Block Telnet"

# Enable / disable the firewall for all profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

---

## 📊 PowerShell — Event Logs & Auditing

> Windows Event Logs are the primary source of evidence during security investigations.

```powershell
# View the last 50 Security events
Get-EventLog -LogName Security -Newest 50

# View failed login attempts (Event ID 4625)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4625 }

# View successful logins (Event ID 4624)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4624 }

# View account creation events (Event ID 4720)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4720 }

# View user added to Administrators group (Event ID 4732)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4732 }

# View new service installations (Event ID 7045)
Get-EventLog -LogName System | Where-Object { $_.EventID -eq 7045 }

# View scheduled task creation (Event ID 4698)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4698 }

# Filter events by time range
Get-EventLog -LogName Security -After "2025-04-01" -Before "2025-04-30"

# Export security events to CSV (for analysis)
Get-EventLog -LogName Security -Newest 1000 | Export-Csv -Path "security_events.csv" -NoTypeInformation

# View audit policy configuration
auditpol /get /category:*

# Enable auditing for logon events
auditpol /set /subcategory:"Logon" /success:enable /failure:enable

# View System event log
Get-EventLog -LogName System -Newest 50

# View Application event log
Get-EventLog -LogName Application -Newest 50

# List all available event logs
Get-EventLog -List
```

---

## 📁 PowerShell — File Integrity & Scheduled Tasks

```powershell
# Calculate SHA256 hash of a file
Get-FileHash -Path "C:\path\to\file.txt" -Algorithm SHA256

# Calculate MD5 hash
Get-FileHash -Path "C:\path\to\file.txt" -Algorithm MD5

# Compare hash of a file against a known value
$expected = "abc123..."
$actual = (Get-FileHash "C:\file.txt" -Algorithm SHA256).Hash
if ($expected -eq $actual) { "File is intact" } else { "File has been modified!" }

# Find files modified in the last 24 hours
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-1) }

# Find files modified in the last hour
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.LastWriteTime -gt (Get-Date).AddHours(-1) }

# Search for files containing sensitive strings
Select-String -Path "C:\inetpub\*" -Pattern "password" -Recurse
Select-String -Path "C:\*" -Pattern "AWS_SECRET" -Recurse

# Find large files (potential data staging)
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue |
  Where-Object { $_.Length -gt 100MB }

# --- Scheduled Tasks ---

# List all scheduled tasks
Get-ScheduledTask

# List only enabled/ready tasks
Get-ScheduledTask | Where-Object { $_.State -eq "Ready" }

# View details of a specific task
Get-ScheduledTaskInfo -TaskName "TaskName"

# View task action (what it runs — look for suspicious scripts)
Get-ScheduledTask | Select-Object TaskName, TaskPath, @{Name="Action";Expression={$_.Actions.Execute}} | Format-List

# Disable a suspicious scheduled task
Disable-ScheduledTask -TaskName "SuspiciousTask"

# Delete a scheduled task
Unregister-ScheduledTask -TaskName "SuspiciousTask" -Confirm:$false
```

---

## 🔐 PowerShell — Execution Policy & Hardening

```powershell
# View current execution policy
Get-ExecutionPolicy

# View execution policy for all scopes
Get-ExecutionPolicy -List

# Set execution policy (recommended for security)
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine

# Block all scripts (most restrictive)
Set-ExecutionPolicy Restricted -Scope LocalMachine

# --- Registry (Startup & Persistence) ---

# View programs that run at startup (machine-wide)
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# View programs that run at startup (current user)
Get-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Remove a suspicious startup entry
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "SuspiciousApp"

# --- Windows Defender ---

# View Defender status
Get-MpComputerStatus

# Run a quick scan
Start-MpScan -ScanType QuickScan

# Run a full scan
Start-MpScan -ScanType FullScan

# Update Defender signatures
Update-MpSignature

# View recent threats detected
Get-MpThreatDetection
```

---

## 🔑 Windows Security Event IDs

> These Event IDs are essential for log analysis, SOC work, and security investigations.

| Event ID | Log | Description |
|---|---|---|
| **4624** | Security | Successful logon |
| **4625** | Security | Failed logon attempt |
| **4634** | Security | Account logoff |
| **4647** | Security | User-initiated logoff |
| **4648** | Security | Logon with explicit credentials (RunAs) |
| **4656** | Security | Handle to an object requested |
| **4663** | Security | Attempt to access an object |
| **4672** | Security | Special privileges assigned to new logon |
| **4688** | Security | New process created |
| **4689** | Security | Process exited |
| **4698** | Security | Scheduled task created |
| **4699** | Security | Scheduled task deleted |
| **4700** | Security | Scheduled task enabled |
| **4720** | Security | User account created |
| **4722** | Security | User account enabled |
| **4723** | Security | Password change attempt |
| **4724** | Security | Password reset attempt |
| **4725** | Security | User account disabled |
| **4726** | Security | User account deleted |
| **4728** | Security | User added to global group |
| **4732** | Security | User added to local Administrators group |
| **4740** | Security | User account locked out |
| **4756** | Security | User added to universal group |
| **4771** | Security | Kerberos pre-authentication failed |
| **4776** | Security | NTLM authentication attempt |
| **7034** | System | Service crashed unexpectedly |
| **7036** | System | Service started or stopped |
| **7045** | System | New service installed |
| **1102** | Security | Audit log cleared ⚠️ |
| **4719** | Security | Audit policy changed ⚠️ |

> ⚠️ **Events 1102 and 4719 are critical red flags** — they indicate an attacker may be covering their tracks.

---

## 🚑 Incident Response — Windows

> Act methodically. Preserve evidence before making changes.

```powershell
# 1. PRESERVE — capture system state immediately
$timestamp = Get-Date -Format "yyyyMMdd_HHmm"
$report = "incident_$timestamp.txt"

"=== INCIDENT REPORT - $timestamp ===" | Out-File $report
"--- Logged-in Users ---" | Out-File $report -Append
query user | Out-File $report -Append
"--- Running Processes ---" | Out-File $report -Append
Get-Process | Out-File $report -Append
"--- Network Connections ---" | Out-File $report -Append
Get-NetTCPConnection | Out-File $report -Append
"--- Listening Ports ---" | Out-File $report -Append
Get-NetTCPConnection -State Listen | Out-File $report -Append
"--- Local Administrators ---" | Out-File $report -Append
Get-LocalGroupMember -Group "Administrators" | Out-File $report -Append
"--- Scheduled Tasks ---" | Out-File $report -Append
Get-ScheduledTask | Where-Object { $_.State -eq "Ready" } | Out-File $report -Append
"--- Startup Programs ---" | Out-File $report -Append
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" | Out-File $report -Append

# 2. IDENTIFY — find suspicious activity

# Look for unknown administrators
Get-LocalGroupMember -Group "Administrators"

# Detect suspicious processes running from temp folders
Get-Process | Select-Object Name, Id, Path | Where-Object { $_.Path -like "*\Temp\*" -or $_.Path -like "*\AppData\*" }

# Find connections to unknown external IPs
Get-NetTCPConnection -State Established | Where-Object { $_.RemoteAddress -notlike "192.168.*" -and $_.RemoteAddress -ne "127.0.0.1" }

# Check for recently created user accounts
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4720 } | Select-Object -First 10

# Check for audit log clearing
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 1102 }

# 3. CONTAIN — isolate the threat

# Block suspicious IP via firewall
New-NetFirewallRule -DisplayName "BLOCK INCIDENT IP" -Direction Outbound -RemoteAddress "203.0.113.5" -Action Block
New-NetFirewallRule -DisplayName "BLOCK INCIDENT IP IN" -Direction Inbound -RemoteAddress "203.0.113.5" -Action Block

# Disable compromised user account
Disable-LocalUser -Name "compromised_user"

# Kill suspicious process
Stop-Process -Id <PID> -Force

# Disable suspicious scheduled task
Disable-ScheduledTask -TaskName "SuspiciousTask"

# 4. INVESTIGATE — trace the attack

# Search security logs for the attacker's activity
Get-EventLog -LogName Security | Where-Object { $_.Message -like "*203.0.113.5*" }

# Review recent failed and successful logins
Get-EventLog -LogName Security | Where-Object { $_.EventID -in @(4624, 4625) } | Select-Object TimeGenerated, EventID, Message | Format-List

# Look for lateral movement (logon type 3 = network logon)
Get-EventLog -LogName Security | Where-Object { $_.EventID -eq 4624 -and $_.Message -like "*Logon Type:*3*" }

# 5. RECOVER — clean and harden

# Remove unauthorized startup entries
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "MaliciousApp"

# Reset compromised user password
net user compromised_user NewSecurePassword!

# Re-enable account after cleanup
Enable-LocalUser -Name "compromised_user"

# Run Defender full scan
Start-MpScan -ScanType FullScan

# Apply pending Windows updates
Install-Module PSWindowsUpdate -Force
Get-WUInstall -AcceptAll -AutoReboot
```

---

## 📌 Quick Reference — Security Best Practices

| Practice | Command / Action |
|---|---|
| Audit failed logins | `Get-EventLog -LogName Security \| Where-Object { $_.EventID -eq 4625 }` |
| Check local admins | `Get-LocalGroupMember -Group "Administrators"` |
| Find listening ports | `Get-NetTCPConnection -State Listen` |
| Verify file integrity | `Get-FileHash -Algorithm SHA256` |
| Check startup entries | `Get-ItemProperty "HKLM:\...\CurrentVersion\Run"` |
| Monitor scheduled tasks | `Get-ScheduledTask \| Where-Object { $_.State -eq "Ready" }` |
| Block suspicious IP | `New-NetFirewallRule -RemoteAddress x.x.x.x -Action Block` |
| Disable compromised user | `Disable-LocalUser -Name "username"` |
| Check audit log cleared | `Get-EventLog -LogName Security \| Where-Object { $_.EventID -eq 1102 }` |
| Update Defender | `Update-MpSignature` |

---

## 🛠️ Recommended Tools

| Tool | Purpose |
|---|---|
| [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/) | Advanced process, network, and registry analysis (Microsoft) |
| [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) | Detailed process inspection (replaces Task Manager) |
| [Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns) | View all auto-start locations (persistence detection) |
| [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview) | Real-time network connections with process names |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security) | Built-in AV and EDR (manageable via PowerShell) |
| [Event Viewer](https://learn.microsoft.com/en-us/shows/inside/event-viewer) | GUI for Windows Event Logs |
| [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) | Advanced system activity logging (highly recommended) |
| [Wireshark](https://www.wireshark.org/) | Network packet capture and analysis |
| [Volatility](https://volatilityfoundation.org/) | Memory forensics framework |

---

<div align="center">

*Maintained as a personal reference for Cloud Security practices.*  
*Commands tested on Windows 10/11 and Windows Server environments.*

</div>
