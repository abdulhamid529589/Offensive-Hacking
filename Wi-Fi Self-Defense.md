# Wi-Fi Self-Defense Reference Guide

## https://youtu.be/Wl3PHh6DaWU

### Based on Cody Kinsey's Wi-Fi Self-Defense Course

> **Legal & Ethical Notice:** All techniques in this document are for **educational and defensive purposes only**. Practice exclusively on your own devices — your own laptop, your own phone, your own network. Attacking devices without explicit permission is illegal under computer crime laws in virtually every country. Active Wi-Fi attacks are detectable and will attract attention in monitored environments (airports, government buildings, corporate networks). Never perform these against devices you do not own.

---

## Table of Contents

1. [Tools & Setup](#1-tools--setup)
2. [ESP8266 & Wi-Fi Nugget Overview](#2-esp8266--wi-fi-nugget-overview)
3. [Flashing Firmware via nugget.dev](#3-flashing-firmware-via-nuggetdev)
4. [Connecting via Serial Terminal](#4-connecting-via-serial-terminal)
5. [Lab 1 — Wi-Fi Probe Leaks (Passive)](#5-lab-1--wi-fi-probe-leaks-passive)
6. [Lab 2 — Beacon Swarms (Active)](#6-lab-2--beacon-swarms-active)
7. [Lab 3 — Evil QR Codes & Hidden Networks (Active)](#7-lab-3--evil-qr-codes--hidden-networks-active)
8. [Lab 4 — Wi-Fi Phishing / Evil Portal (Active)](#8-lab-4--wi-fi-phishing--evil-portal-active)
9. [Deauthentication Attacks — Awareness Only](#9-deauthentication-attacks--awareness-only)
10. [Hardening Your Devices — Full Checklist](#10-hardening-your-devices--full-checklist)
11. [Key Concepts Reference](#11-key-concepts-reference)
12. [Resources](#12-resources)

---

## 1. Tools & Setup

### Required Hardware

- **Wi-Fi Nugget** (ESP8266-based microcontroller) — or any board running the ESP8266 chip
- **USB cable** (data-capable, not charge-only)
- **Laptop** with Google Chrome installed

### Required Software / Websites

| Tool               | URL                 | Purpose                                   |
| ------------------ | ------------------- | ----------------------------------------- |
| Nugget Dev         | nugget.dev          | Flash firmware to the device              |
| Serial Terminal    | serialterminal.com  | Communicate with the device over USB      |
| Alternative Serial | serial.hoon.me      | Alternative serial terminal               |
| QiFi               | qifi.org            | Generate Wi-Fi QR codes                   |
| Course GitHub      | (provided in class) | Scripts, commands, examples to copy/paste |

### Why Google Chrome?

Chrome supports the **Web Serial API**, which allows a browser to communicate directly with a USB serial device. This replaces the need for command-line tools and makes flashing and controlling the device accessible to beginners.

---

## 2. ESP8266 & Wi-Fi Nugget Overview

### What is the ESP8266?

The **ESP8266** is a low-cost Wi-Fi microcontroller chip made by Espressif. It is approximately 7-8 years old and is widely used in IoT projects.

```
KEY CHARACTERISTICS:
- NOT a full computer (cannot run an OS like Linux)
- Runs custom compiled firmware loaded as a .bin file
- Has a built-in Wi-Fi radio
- Programmable in: Arduino (C++), MicroPython
- No built-in restrictions on Wi-Fi packet crafting
  → Can send arbitrary Wi-Fi frames
  → Can create fake access points
  → Can spoof MAC addresses
```

### Why is ESP8266 Useful for Wi-Fi Research?

When Espressif designed this chip, no restrictions were placed on the types of Wi-Fi frames it could send. This means it can:

- Create the **appearance of many Wi-Fi networks** simultaneously
- Send **arbitrary Wi-Fi packets** (probe responses, beacon frames, deauth frames)
- **Spoof MAC addresses** at the hardware level

This makes it an excellent educational tool for understanding Wi-Fi protocol vulnerabilities.

### Wi-Fi Nugget vs. Bare ESP8266

| Feature         | Bare ESP8266 (D1 Mini)        | Wi-Fi Nugget    |
| --------------- | ----------------------------- | --------------- |
| Screen          | No                            | Yes (OLED)      |
| Buttons         | No                            | Yes             |
| GPIO Pins       | Yes                           | Yes (on back)   |
| LED             | Yes                           | Yes             |
| Battery support | No                            | Via LiPo add-on |
| Ease of use     | Requires soldering/breadboard | Plug and play   |

The D1 Mini is itself a **breakout board** of the ESP8266 chip. The Wi-Fi Nugget is a breakout of the D1 Mini — adding screen, buttons, and accessibility features.

### Firmware Versions

| Version                    | Features                                                                | Screen | Use Case                      |
| -------------------------- | ----------------------------------------------------------------------- | ------ | ----------------------------- |
| **Deauthor V2**            | Web interface, display support, basic attacks                           | ✓ Yes  | Beginner-friendly, standalone |
| **Deauthor V3 (Advanced)** | Serial control, rogue AP, signal scanner, beacon swarm, Hunitor support | ✗ No   | This course — more powerful   |

> **Important:** When you flash V3, the screen will go dark. This is **expected and correct**. All resources are dedicated to Wi-Fi functions instead of display output.

---

## 3. Flashing Firmware via nugget.dev

### Step-by-Step

```
1. Open Google Chrome
2. Navigate to: nugget.dev
3. Plug in your Wi-Fi Nugget via USB
4. Click "Connect my Nugget"
5. Select "USB Serial Device" from the popup
   TIP: If unsure which device is yours → unplug, check list, replug, see which appeared
6. Select firmware: "Advanced V3 Deauthor" (bottom of list, marked "no screen")
7. Click "Erase" first (removes any old firmware cleanly)
8. Click "Program"
9. Wait ~30 seconds for flashing to complete
   WARNING: If it completes instantly → something went wrong (bad cable or browser issue)
10. Unplug and replug the device
11. Screen will NOT turn on → this means success
```

### Troubleshooting

| Symptom                          | Likely Cause                             | Fix                             |
| -------------------------------- | ---------------------------------------- | ------------------------------- |
| Flashing completes instantly     | USB cable is charge-only / browser error | Try different cable, try Chrome |
| Device not appearing in list     | Driver issue                             | Try different USB port          |
| Screen still on after V3 flash   | Flash didn't succeed                     | Re-erase and re-flash           |
| Can't connect in serial terminal | Wrong baud rate                          | Try 115200 baud                 |

---

## 4. Connecting via Serial Terminal

### Using serialterminal.com

```
1. Open Chrome → navigate to serialterminal.com
2. Click "Connect"
3. Select your Wi-Fi Nugget from the device list
4. Type: help  → press Enter
5. You should see a list of available commands
```

### Verify Everything Works

```bash
# Type these commands to confirm device is running correctly:
help          # Shows all command categories
chicken       # Fun test — confirms device is responsive
help scan     # Shows scan-specific options
help beacon   # Shows beacon-specific options
help ap       # Shows access point options
```

### Command Structure

Most commands follow this format:

```
command [positional_argument] [flags]

Examples:
scan                          # Basic scan with defaults
scan -m station -t 60         # Station scan, 60 seconds
beacon "MyNetwork"            # Create one fake AP called MyNetwork
beacon "Net1","Net2" -mon     # Create two APs with monitoring enabled
ap "Router_Name"              # Create open access point
stop                          # Stop any running attack/scan
```

---

## 5. Lab 1 — Wi-Fi Probe Leaks (Passive)

### What is This Attack?

Every Wi-Fi-enabled device (phone, laptop, tablet) constantly **broadcasts probe requests** — asking nearby access points "Are you there?" for every network it has ever connected to.

This is how automatic Wi-Fi reconnection works. The consequence is that **your device is constantly announcing its connection history** to anyone listening.

```
YOUR PHONE                          AIR
┌──────────────┐                     │
│ Trusted List │  ─── "Is Starbucks  │
│ - Starbucks  │       WiFi here?" ──►│ ← Anyone can hear
│ - Hotel123   │  ─── "Is Hotel123   │
│ - Work_WiFi  │       here?" ───────►│
│ - DeltaAir   │  ─── "Is DeltaAir  │
└──────────────┘       here?" ───────►│
```

### What Information Leaks?

- **Where you've been** (hotels, airports, restaurants, events)
- **Where you work** (corporate SSIDs)
- **Organizations you're affiliated with** (government, contractors, NGOs)
- **Your unique device identifier** (even with MAC randomization, pattern analysis reveals the device)

### Attack Type: Passive

- No signals transmitted
- Cannot be detected by the target
- Legal to perform (you're just listening)

### The Scan Command — Full Reference

```bash
# Basic scan (default — not optimized):
scan

# Optimized scan for station devices (phones, laptops):
scan -m station -t 60 -ch 1,6,11 -st 5000

# Flag breakdown:
# -m station   → Scan for client devices (phones, laptops), not routers
# -m ap        → Scan for access points (routers, hotspots)
# -t 60        → Run for 60 seconds (default is 20 seconds)
# -ch 1,6,11   → Only scan US non-overlapping channels (most efficient)
#               → Channel 14 is illegal in US — default scan wastes time on it
# -st 5000     → Spend 5000ms (5 seconds) per channel
#               → Longer = more likely to catch intermittently broadcasting devices
```

### Why Channels 1, 6, and 11?

Wi-Fi channels overlap. In the 2.4GHz band:

```
Channel: 1    2    3    4    5    6    7    8    9   10   11   12   13   14
         [===|=============================]
                        [===|=============================]
                                            [===|=================]
```

Channels 1, 6, and 11 are the **only non-overlapping channels** in the US. Devices gravitate toward these. Scanning only these three is both faster and more complete than scanning all 14.

### Reading Scan Results

```
Example output:
Station: AA:BB:CC:DD:EE:FF → Probing for: "Starbucks WiFi"
Station: AA:BB:CC:DD:EE:FF → Probing for: "DeltaAir"
Station: 11:22:33:44:55:66 → Probing for: "DHS_Secure"
Station: 11:22:33:44:55:66 → Probing for: "JPL_Internal"
```

Note: Multiple MAC addresses from the same device = **MAC address randomization** (the device changes its MAC to avoid being tracked, but probe requests for the same networks reveal it's the same device).

### Triggering More Probe Requests

**Unlocking your phone** causes a burst of probe requests. This is the fastest way to observe a device actively calling out.

### How to Protect Yourself

**1. Remove unused/open networks from your device:**

On **iPhone:**

```
Settings → Wi-Fi → tap (i) next to network → "Forget This Network"
For all networks: Settings → General → Transfer or Reset → Reset → Reset Network Settings
```

On **Android:**

```
Settings → Network & Internet → Wi-Fi → Saved Networks
→ tap each network → Forget
```

On **macOS:**

```
System Preferences → Network → Wi-Fi → Advanced
→ Select networks → click minus (−) → Remove
```

On **Windows:**

```
Settings → Network & Internet → Wi-Fi → Manage known networks
→ Click each → Forget
```

**2. Priority targets for deletion:**

- Open (no-password) Wi-Fi networks — anyone can impersonate these
- Old hotel networks
- Airline networks
- Event Wi-Fi networks (conferences, conventions)
- Networks from locations that reveal sensitive affiliations

**3. MAC Address Randomization:**
Enable on your device — it doesn't fully solve the probe request problem but adds a layer of protection.

```
iPhone: Settings → Wi-Fi → tap (i) → Private Wi-Fi Address → ON
Android: Settings → Wi-Fi → tap network → Privacy → Use randomized MAC
```

**4. Turn off Wi-Fi when not in use:**
Even airplane mode may not stop assisted GPS from using Wi-Fi for geolocation. Explicitly disable Wi-Fi from the settings, not just airplane mode.

---

## 6. Lab 2 — Beacon Swarms (Active)

### What is This Attack?

A **beacon swarm** creates a large number of fake Wi-Fi networks simultaneously. Devices nearby that have any of these networks in their trusted list will automatically attempt to connect, revealing their connection history.

```
BEACON SWARM                    NEARBY DEVICES
┌──────────────────────┐
│ Fake: "Starbucks"    │◄── Phone A tries to connect (was at Starbucks)
│ Fake: "DeltaAir"     │◄── Phone B tries to connect (flew Delta)
│ Fake: "Hotel Marriot"│◄── Phone C tries to connect (stayed at Marriott)
│ Fake: "Google Guest" │◄── Phone D tries to connect (visited Google office)
│ ... up to 100 nets   │
└──────────────────────┘
       [Wi-Fi Nugget]
```

### Attack Type: Active

- **Visible** to any Wi-Fi monitoring system
- Extremely obvious in monitored environments
- **Only perform against your own devices**

### The Beacon Command — Full Reference

```bash
# Create a single fake network:
beacon "NetworkName"

# Create multiple fake networks:
beacon "Network1","Network2","Network3"

# Create with monitoring (alerts when a device tries to connect):
beacon "Network1","Network2" -mon

# Stop the attack:
stop

# Available flags:
# -ch [number]     → Channel to broadcast on
# -t [seconds]     → How long to run
# -mon             → Monitor for connection attempts (required for alerts)
```

### Example: Common Network List

Common public networks to include in a beacon swarm (for testing against your own devices):

```bash
beacon "Starbucks WiFi","Google Starbucks","xfinitywifi","attwifi","Boingo Hotspot","Southwest WiFi","DeltaAir","United WiFi","Marriott WiFi","Hilton Honors","Best Western WiFi","IKEA WiFi","McDonald's Free WiFi","Burger King WiFi","Whopper WiFi" -mon
```

### Reading Beacon Swarm Results

```
Alert: Device AA:BB:CC:DD:EE:FF attempted to connect to "Starbucks WiFi"
Alert: Device 11:22:33:44:55:66 attempted to connect to "DeltaAir"
```

This tells you:

- The device has been to a Starbucks location before
- The device has flown Delta Airlines before
- The MAC address of the device (may be randomized but still useful)

### Reducing for Better Results

If you're seeing errors with too many access points, reduce the list:

```bash
# Fewer targets = better monitoring resolution
beacon "Network1","Network2","Network3" -mon
```

### Defensive Meaning

This attack shows why removing old stored networks is critical. A beacon swarm can:

1. Confirm someone has been to a location they deny visiting
2. Identify organizational affiliations (government, contractors)
3. Set up for a targeted phishing attack by knowing which network the victim's device will auto-join

### Modern Countermeasures

Some newer devices (iOS 14+, Android 11+) will warn users if a network they're connecting to is appearing in an unexpected location. If your device has been to a Starbucks in California and you're now in a foreign country, it may warn you before auto-connecting to a "Starbucks WiFi" beacon.

---

## 7. Lab 3 — Evil QR Codes & Hidden Networks (Active)

### What is a Hidden Wi-Fi Network?

A **hidden network** (also called a closed SSID) is a Wi-Fi access point that does **not** broadcast beacon frames. It doesn't announce its presence.

```
NORMAL NETWORK:
Router ──beacon─► "I am 'HomeWiFi', come connect to me"
                  Devices scan → see it → optionally join

HIDDEN NETWORK:
Router ──(silence)──
Device must actively ASK: "Is 'HiddenNet' nearby? Is 'HiddenNet' nearby?"
Router answers only when asked directly
```

**Consequence:** Any device that has a hidden network in its trusted list must **constantly broadcast that network name** via probe requests, forever. This is opposite of the intended privacy goal — hiding a network makes the _device_ more exposed, not less.

### What is an Evil QR Code?

A Wi-Fi QR code is a standardized format that stores Wi-Fi credentials so a phone can scan and auto-join a network. The format is:

```
WIFI:T:WPA;S:NetworkName;P:Password;H:false;;

Fields:
T: = Encryption type (WPA, WEP, nopass)
S: = SSID (network name)
P: = Password
H: = Hidden (true/false)  ← THIS IS THE EVIL FLAG
```

When `H:true` is set, the device joins the network as a hidden network — meaning it will now **permanently broadcast that network name** via probe requests.

### How the Attack Works

```
ATTACKER                              VICTIM
Creates QR code with H:true ──────► Scans QR code
                                      │
                              Device auto-joins silently
                                      │
                              Device now CONSTANTLY broadcasts:
                              "Is [hidden network name] nearby?"
                                      │
ATTACKER can now:                     │
- Track victim by signal strength ◄───┘
- Detect if victim enters an area
- Create matching network to take over data connection
- Perform MITM attack
```

### Generating a Wi-Fi QR Code

Go to **qifi.org**:

```
1. SSID:       Enter any network name (this is the "tag" placed on the device)
2. Encryption: WPA2/WPA3 (doesn't matter much)
3. Password:   Any password (even gibberish works — device still broadcasts the SSID)
4. Hidden:     ✓ CHECK THIS BOX ← This is what makes it malicious

Click Generate → QR code is created
```

### Using the Probe Reveal Scan to Confirm

After scanning the QR code with your test device, run:

```bash
scan -m station -t 60 -ch 1,6,11 -st 5000
```

You should see the device now probing for the network name you encoded in the QR code. This confirms the attack worked.

### Scanning QR Codes in a QR Reader (Seeing Raw Data)

Before scanning any QR code with your camera, use a QR reader that **shows the raw text** without auto-connecting:

- iOS: Use the built-in camera but read the preview URL before tapping
- Android: Use a dedicated QR app that shows the text first
- Look for `H:true` at the end of the decoded text

### Removing an Injected Hidden Network

**iPhone:**

```
Settings → Wi-Fi
→ Scroll down to find the hidden network (may show as just a name)
→ Tap (i) → Forget This Network
Note: You must be in range of the network to remove it on older iOS versions
iOS 17+: Settings → Wi-Fi → Edit → find the hidden network → delete
```

**Android:**

```
Settings → Network & Internet → Wi-Fi → Saved Networks
→ Find the network → Forget
```

**macOS:**

```
System Preferences → Network → Wi-Fi → Advanced
→ Find the network → select → click (−)
```

### Why This Attack is Significant

- **Defeats MAC randomization** — even if the device changes its MAC address, the unique hidden network name creates a fingerprint
- **Range:** Wi-Fi signal detectable up to ~1 mile with line-of-sight
- **Stalking potential:** Attacker can determine if victim is home, at work, or in a specific area
- **Platform for MITM:** Creating the matching network allows traffic interception

### Historical Note on iOS Vulnerability

Until approximately 2023-2024, it was **impossible to remove a hidden network from an iPhone** unless you were in physical proximity to the actual network. This meant that if someone scanned a malicious QR code, the hidden network was essentially permanent until Apple fixed the removal flow. This has since been resolved in recent iOS versions.

---

## 8. Lab 4 — Wi-Fi Phishing / Evil Portal (Active)

### What is Wi-Fi Phishing?

Wi-Fi phishing (also called an **evil portal** or **captive portal attack**) combines two techniques:

1. **Deauthentication** — Kick the victim off their real Wi-Fi
2. **Rogue Access Point** — Create a fake Wi-Fi with the same name but no password

When the victim connects to the fake network, they are presented with a captive portal — a webpage that appears to be their router requesting the Wi-Fi password for a firmware update or other plausible reason.

### Real-World Example (from the course instructor)

```
Scenario: Office in a mixed-use building

1. Attacker ran deauth attack → jammed office Wi-Fi
2. Created open network with same name as office Wi-Fi
3. Frustrated workers connected to open version
4. Captive portal appeared: "Router firmware updating — enter Wi-Fi password"
5. Instructor typed gibberish → password was rejected in real time
   (The portal was checking the password against the real router!)
6. Instructor typed real password → "update" appeared to succeed
7. Attacker had the real Wi-Fi password
8. Jamming stopped → office Wi-Fi returned to normal
9. Instructor thought they had fixed a router update — actually gave away the password
```

### Attack Anatomy

```
PHASE 1: Deauthentication
Attacker sends deauth frames → Victim's device disconnects from real AP
Real AP: [ACCESS BLOCKED]     Victim's device: "Disconnected!"

PHASE 2: Rogue AP Creation
Attacker creates: "HomeWiFi" (open, no password)
Victim sees: "HomeWiFi" (same name, but open)
Victim connects (open network, no password needed)

PHASE 3: Captive Portal
Victim's browser automatically opens captive portal
Portal displays: "Router Update Required — Enter Wi-Fi password to proceed"
Victim types password → attacker receives it

PHASE 4: Resolution
Attacker stops deauth → real network returns
Victim thinks: "I successfully updated my router!"
Reality: Victim gave away their Wi-Fi password
```

### The AP Command — Full Reference

```bash
# Create a basic open access point:
ap "NetworkName"

# Available flags:
# ap "Name"              → Create open network named "Name"
# ap "Name" -p "pass"    → Create password-protected network
# ap "Name" -h           → Create hidden access point
# ap "Name" -ch 6        → Specify channel
# ap "Name" -bssid XX:XX:XX:XX:XX:XX  → Spoof MAC address/manufacturer

# Stop the access point:
stop
```

### MAC Address Spoofing for Realism

The first 3 bytes (6 hex digits) of a MAC address identify the **manufacturer (OUI)**:

```
Example MAC: 00:1A:11:XX:XX:XX
             └──────┘
             This part = Google Inc.

Lookup: macaddress.io or wireshark OUI lookup
```

To make the rogue AP look like a legitimate router brand:

```bash
# 1. First scan to see what brand the real router is:
scan -m ap -t 30

# 2. Look up that MAC address prefix on macaddress.io
# 3. Find the OUI for that manufacturer
# 4. Create AP with matching OUI:
ap "HomeWiFi" -bssid A4:C3:F0:XX:XX:XX   # Linksys OUI example
```

### What the Portal Can Capture

When a victim connects and submits a password through the captive portal:

```
Serial terminal output:
Client connected: MAC=AA:BB:CC:DD:EE:FF IP=192.168.4.2
Password submitted: [the password they typed]
```

### Real-Time Password Verification

Sophisticated implementations of this attack check the submitted password against the real router in real time — if the password is wrong, the portal rejects it and asks again. This:

- Confirms to the attacker they have the correct password
- Convinces the victim that the portal "knows" the real password and is legitimate

### How to Spot This Attack

**Warning signs:**

- Your Wi-Fi disconnects suddenly (deauth)
- An open version of your network name appears
- A webpage appears asking for your Wi-Fi password
- Any captive portal claiming to be your home/office router

**Important rules:**

```
LEGITIMATE ROUTERS:
✓ Never ask for your Wi-Fi password via a captive portal
✓ Never ask for Wi-Fi password during firmware updates
✓ Firmware updates are done through router admin panel (192.168.1.1 etc.)
✓ Router admin panel uses a DIFFERENT password than Wi-Fi password

PHISHING PORTAL:
✗ Appears after unexpected disconnect
✗ Asks for Wi-Fi password (not router admin password)
✗ Operates on an open (no password) network with your network's name
```

### Defense Against Wi-Fi Phishing

**1. Use a VPN:**
A VPN creates an encrypted tunnel that:

- Prevents the rogue AP from seeing your traffic
- Blocks captive portal injections
- Prevents the attacker from seeing which apps/websites you're using

**2. Recognize the attack pattern:**
Unexpected disconnect + same-name open network = likely phishing attack

**3. Never enter Wi-Fi password into any captive portal:**
This is never a legitimate request from your router.

**4. Check the network:**
If disconnected, before reconnecting to any open network:

```
iPhone:  Settings → Wi-Fi → check for (!) warning icon
Android: Long-press network → see if it shows "No internet access"
```

**5. Remove open networks from trusted list:**
An open version of your network can only succeed as an attack if your device has open networks stored — remove them all.

---

## 9. Deauthentication Attacks — Awareness Only

### What is a Deauthentication Attack?

A deauthentication (deauth) attack sends forged 802.11 management frames that instruct a device to disconnect from its current access point. These frames are **unauthenticated in older Wi-Fi standards** — any device can send them and the target will comply.

```
ATTACKER sends:  "Dear [Victim's MAC], this is your router. Please disconnect."
VICTIM receives: Disconnect instruction (appears legitimate)
VICTIM does:     Disconnects from legitimate AP
```

### Why It Works

In Wi-Fi 4 (802.11n) and older, management frames are not authenticated or encrypted. There is no way for the receiving device to verify that a deauth frame was actually sent by the real AP.

**Wi-Fi 6 (802.11ax) with Protected Management Frames (PMF)** — also known as 802.11w — cryptographically signs management frames, making deauth attacks much less effective.

### Deauth Command (for awareness only)

```bash
# DO NOT USE AGAINST DEVICES YOU DON'T OWN

# View available scan results (to select target):
scan -m ap -t 30

# Deauth a specific access point (kicks all connected clients):
deauth ap2          # Kicks everyone off AP #2 from scan results

# Stop:
stop
```

### Why This is the Most Dangerous Command

- **Extremely loud** — immediately visible to any Wi-Fi monitoring system
- **Potentially criminal** — interfering with someone else's Wi-Fi is illegal in most jurisdictions (FCC regulations in US, similar laws elsewhere)
- **Unpredictable consequences** — you cannot know what device is doing something critical (medical device, VoIP emergency call, industrial control system)
- **Detectable** — Wi-Fi IDS (Intrusion Detection Systems) flag deauth floods immediately

### Defense Against Deauth Attacks

**Enable Protected Management Frames (PMF / 802.11w):**

On your router:

```
Router admin panel → Wireless settings → 802.11w / PMF → Required
Note: This requires all connecting devices to also support PMF (most modern devices do)
```

On modern routers supporting Wi-Fi 6, this may already be enabled by default.

**Upgrade to WPA3:**
WPA3 includes PMF as a mandatory feature.

---

## 10. Hardening Your Devices — Full Checklist

### Priority 1: Manage Your Trusted Network List

```
□ Open Settings → Wi-Fi → Manage saved/known networks
□ Delete ALL open (no password) networks
□ Delete old hotel networks
□ Delete old airline networks (Delta, United, Southwest, etc.)
□ Delete conference/event networks (DEF CON, Black Hat, etc.)
□ Delete networks from sensitive locations you don't want associated with you
□ Keep only: home network, work network (if necessary), networks you actively use
□ On iCloud/Google Account sync: be aware networks may sync across all your devices
  → Clearing one device may not clear others if sync is active
```

### Priority 2: Check for Hidden Networks

```
□ Run a probe reveal scan against your own phone
□ Look for any unknown hidden network SSIDs being broadcast
□ Go through saved networks and identify any hidden ones
□ Remove any hidden networks (especially ones you didn't intentionally add)
□ Never join a hidden network unless you explicitly need to
□ Tell IT departments: hiding SSIDs does not improve security and harms device privacy
```

### Priority 3: MAC Address Randomization

```
iPhone:
□ Settings → Wi-Fi → tap (i) on each network → Private Wi-Fi Address → Enabled

Android:
□ Settings → Wi-Fi → tap network → Privacy → Use randomized MAC

macOS (Monterey+):
□ System Preferences → Network → Wi-Fi → Advanced
  → Enable MAC address randomization per network
```

### Priority 4: Wi-Fi Radio Management

```
□ Turn off Wi-Fi when leaving a trusted area
□ Don't rely on Airplane Mode alone — explicitly toggle Wi-Fi off
  (Assisted GPS can still use Wi-Fi even in Airplane Mode)
□ In high-sensitivity locations (government buildings, secure facilities):
  → Keep Wi-Fi disabled
  → Use cellular data only
```

### Priority 5: QR Code Safety

```
□ Never scan Wi-Fi QR codes in untrusted locations
□ Before scanning: use a QR reader that shows raw text first
□ Check for H:true at end of decoded text (= hidden network = potentially malicious)
□ After scanning any QR code → check Wi-Fi settings for new unknown networks
□ Be suspicious of QR codes on public posters, stickers, menus
  (physical replacement of QR codes with malicious ones is a known attack)
```

### Priority 6: Phishing Defense

```
□ Never enter your Wi-Fi password into any captive portal
□ If your Wi-Fi disconnects unexpectedly → look for open version of network → be suspicious
□ Verify router admin requires admin password (not Wi-Fi password)
□ Use WPA3 on your home network (or WPA2 at minimum)
□ Use VPN on all public Wi-Fi connections
□ Enable PMF (Protected Management Frames) on router if available
□ Keep router firmware updated through the legitimate admin panel
```

### Priority 7: Router Hardening

```
□ Change default router admin password
□ Change default Wi-Fi network name (don't use ISP default that reveals ISP/router brand)
□ Enable WPA3 (or WPA2-AES at minimum — disable TKIP)
□ Enable PMF / 802.11w if router supports it
□ Disable WPS (Wi-Fi Protected Setup) — vulnerable to brute force
□ Disable UPnP if not needed
□ Keep router firmware updated
□ Use a long, random Wi-Fi password (20+ characters)
□ Guest network for IoT devices and visitors (isolated from main network)
```

### Priority 8: VPN Usage

```
□ Use a VPN on all public Wi-Fi (airports, hotels, coffee shops)
□ VPN prevents:
   → Traffic visibility to the AP operator
   → Captive portal injection (partially)
   → Man-in-the-middle traffic analysis
□ Choose a VPN with:
   → No-log policy
   → Kill switch feature
   → Based outside 5 Eyes intelligence alliance (optional, higher security)
□ Enable VPN kill switch — drops internet if VPN disconnects
   (prevents accidental exposure when VPN drops on public Wi-Fi)
```

---

## 11. Key Concepts Reference

### Wi-Fi Terminology

| Term                       | Definition                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| **SSID**                   | Service Set Identifier — the name of a Wi-Fi network                                       |
| **BSSID**                  | Basic Service Set Identifier — the MAC address of a Wi-Fi access point                     |
| **Access Point (AP)**      | A device creating a Wi-Fi network (e.g., a router)                                         |
| **Station (STA)**          | A client device connecting to a network (phone, laptop)                                    |
| **Probe Request**          | Frame sent by a station asking if a known network is nearby                                |
| **Beacon Frame**           | Frame broadcast by an AP announcing its presence                                           |
| **Hidden Network**         | AP that doesn't send beacon frames; SSID must be known in advance                          |
| **Captive Portal**         | Web page presented when joining a network that requires interaction before internet access |
| **Deauthentication Frame** | Management frame instructing a device to disconnect                                        |
| **PMF / 802.11w**          | Protected Management Frames — cryptographic signing of deauth and other management frames  |
| **WPA3**                   | Latest Wi-Fi security standard, includes mandatory PMF                                     |
| **OUI**                    | Organizationally Unique Identifier — first 3 bytes of MAC address, identifies manufacturer |
| **MAC Randomization**      | Feature that changes device's broadcast MAC address to prevent tracking                    |
| **MITM**                   | Man-in-the-Middle — attacker intercepts communication between two parties                  |

### Attack Type Summary

| Attack                 | Type                        | Detectable?        | Risk Level |
| ---------------------- | --------------------------- | ------------------ | ---------- |
| Probe reveal scan      | Passive                     | No                 | Low        |
| Beacon swarm           | Active                      | Yes — very obvious | Medium     |
| Evil QR code           | Active (requires proximity) | Yes if monitoring  | Medium     |
| Evil portal (phishing) | Active                      | Yes — very obvious | High       |
| Deauthentication       | Active                      | Yes — immediately  | Very High  |

### Legal Reference Points (United States)

| Law                                              | Relevance                                                                                   |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| **Computer Fraud and Abuse Act (CFAA)**          | Criminalizes unauthorized access to computer systems; intentional deauth of others' devices |
| **Electronic Communications Privacy Act (ECPA)** | Intercept of electronic communications without authorization                                |
| **FCC Part 15 / Section 333**                    | Prohibits deliberate interference with licensed radio communications                        |
| **State computer crime laws**                    | Vary by state; generally mirror federal law                                                 |

> These attacks are **legal** when performed only on your own devices, in your own environment, for defensive learning purposes.

---

## 12. Resources

### Online Tools

- **nugget.dev** — Flash firmware to Wi-Fi Nugget
- **serialterminal.com** — Web-based serial terminal (Chrome)
- **qifi.org** — Generate Wi-Fi QR codes
- **macaddress.io** — Look up manufacturer from MAC OUI
- **wigle.net** — Wi-Fi network mapping database

### Learning & Community

- **hack.gay** — Cody Kinsey's resource hub
- **retia.io** — Security Forward channel, Wi-Fi Nugget products
- **Null Byte YouTube** — Wi-Fi hacking tutorials (~1M subscribers)
- **Hack5 Channel** — Hardware hacking and Wi-Fi tools
- **Security Forward / Retia Livestreams** — Live Q&A, Wednesday and Friday

### Hardware

- **Wi-Fi Nugget** — retia.io store
- **D1 Mini (bare ESP8266)** — Amazon, AliExpress (~$3-5)
- **LiPo battery add-on** — retia.io store (for standalone operation)

### Firmware / Source Code

- **ESP8266 Deauthor V3** — GitHub: SpacehuhnTech/esp8266_deauther
- **GoPhish** — getgophish.com (for simulated phishing testing)

### Key Commands Quick Reference

```bash
# Scanning
scan                                          # Basic scan
scan -m station -t 60 -ch 1,6,11 -st 5000   # Optimized probe reveal

# Beacon Swarm
beacon "NetworkName"                          # Single beacon
beacon "Net1","Net2","Net3" -mon              # Multiple with monitoring
stop                                          # Stop beacons

# Access Point
ap "NetworkName"                              # Open AP
ap "NetworkName" -p "password"               # Password-protected AP
ap "NetworkName" -ch 6 -bssid XX:XX:XX:XX:XX:XX  # With channel and MAC spoof
stop                                          # Stop AP

# Help
help                                          # All commands
help scan                                     # Scan command flags
help beacon                                   # Beacon command flags
help ap                                       # AP command flags
chicken                                       # Morale boost / connectivity test
```

---

_Document based on Cody Kinsey's Wi-Fi Self-Defense course. All techniques should only be used against your own devices for defensive learning. Report Wi-Fi attacks you experience to your ISP and, if in a workplace, to your IT security team._
