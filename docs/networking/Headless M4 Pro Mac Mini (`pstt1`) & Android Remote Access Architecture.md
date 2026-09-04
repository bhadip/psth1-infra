# Headless M4 Pro Mac Mini (`pstt1`) & Android Remote Access Architecture

This document contains the definitive configuration guide for running an **M4 Pro Mac Mini (`pstt1`)** completely headless with **FileVault Enabled**, a **Static IP (`192.168.50.3`)**, and an **HDMI Dummy Plug**. It outlines how to bypass Apple's strict security barriers using a MacBook Pro or an Android Phone.

---

## 🏗️ Phase 1: Installation & Configuration (Run Locally on Mac Mini Once)

Perform these steps **locally** with a physical monitor, keyboard, and mouse attached to the Mac Mini before moving it to a headless location.

### 1. Tailscale Deployment (System Daemon Mode)
To ensure remote access from outside your home network, Tailscale must run as a persistent system service.
1. Open the Mac Mini Terminal and install Homebrew if you haven't already:
   ```bash
   /bin/bash -c "\$(curl -fsSL https://githubusercontent.com)"
   ```
2. Install Tailscale via Homebrew (do **not** use the Mac App Store version, as it will drop connections when logged out):
   ```bash
   brew install tailscale
   ```
3. Start the persistent background service daemon:
   ```bash
   sudo brew services start tailscale
   ```
4. Authenticate the machine to your Tailnet:
   ```bash
   sudo tailscale up --accept-routes --accept-dns
   ```
5. **Make it persist on restart:** 
   * Open your **Tailscale Admin Console** on a web browser.
   * Go to the **Machines** tab and locate `pstt1`.
   * Click the **three dots (...)** next to it and select **Disable key expiry**. This prevents random logouts every 180 days.

### 2. Configure Legacy VNC for Android Pre-Boot Login
This creates a permanent backdoor through macOS security, allowing your Android phone to view the account login window.
1. Open Terminal on the Mac Mini and execute this command to allow unencrypted VNC clients and set a permanent access password (`jfkdkd`):
   ```bash
   sudo defaults write /Library/Preferences/com.apple.RemoteManagement VNCAlwaysStartOnConsole -bool true && sudo defaults write /Library/Preferences/com.apple.RemoteManagement VNCPrivilegeSoftwareUpdate -bool true && sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart -configure -clientopts -setvnclegacy -vnclegacy yes -setvncpw -vncpw jfkdkd
   ```

### 3. Parsec Deployment (Ultra-Low Latency Workspace)
1. Download and install [Parsec for macOS](https://parsec.app).
2. Launch Parsec and log into your account.
3. Navigate to **Settings (Gear Icon) > Host** and change **Hosting** to **Enabled**.
4. **Grant Permissions:** Go to **System Settings > Privacy & Security**. 
   * Under **Accessibility**, toggle **Parsec** to **ON**.
   * Under **Screen & System Audio Recording**, toggle **Parsec** to **ON**.
5. **Make it persist on restart:** 
   * Go to **Parsec Settings > Client** and toggle **Run Parsec When My Computer Starts** to **ON**.
   * Go to **System Settings > General > Login Items**. Under the "Open at Login" list, ensure Parsec is listed. (If missing, click `+` and select Parsec from Applications).

### 4. Chrome Remote Desktop Deployment (Android Touch Friendly)
1. Open Google Chrome on the Mac Mini and visit [://google.com](https://://google.com/).
2. Follow the prompts to download and install the **Chrome Remote Desktop Host installer**.
3. Set a secure **PIN** and name the machine `pstt1`.
4. Go to **System Settings > Privacy & Security > Accessibility** and **Screen & System Audio Recording**, and ensure **Chrome Remote Desktop** is toggled **ON**.
5. **Make it persist on restart:** The host architecture natively installs its own system launch agents. It will run automatically in the background as long as the macOS desktop GUI is loaded.

***

## 💻 Phase 2: Client Setups (Your Remote Devices)

### On Your MacBook Pro
1. **Parsec Client:** Download and install [Parsec for macOS](https://parsec.app). Log into the same Parsec account.
2. **Native Screen Sharing:** No installations required; it is built cleanly into macOS.

### On Your Android Phone
1. **Termux / JuiceSSH:** Install any free SSH client app from the Google Play Store.
2. **RealVNC Viewer:** Install **VNC Viewer - Remote Desktop** by RealVNC from the Google Play Store.
3. **Chrome Remote Desktop:** Install **Chrome Remote Desktop** from the Google Play Store.

***

## 🔄 Phase 3: Headless Operational Blueprints

Once the configurations are saved, plug the **HDMI Dummy Plug** into the back of the Mac Mini, disconnect your physical monitor, and place the Mac Mini in its permanent location.

### Workflow A: Controlling from your MacBook Pro (Maximum Performance)

Follow this order every time the Mac Mini reboots:

1. **Decrypt the Disk:** Open Terminal on your MacBook Pro. Run your passwordless SSH script to connect to the Mac Mini and pass the FileVault disk-encryption block.
2. **Initialize the Desktop GUI:** 
   * Press `Cmd + Space`, type **Screen Sharing**, and open it.
   * Connect to `192.168.50.3` (or your Tailscale URL: `pstt1.gila-karat.ts.net`).
   * Type your Mac user password at the login screen.
   * **Crucial:** The absolute second your desktop icons load, **close Screen Sharing**.
3. **Launch the Workspace:** Open **Parsec** on your MacBook Pro and click **Connect**.
4. **Result:** You will enjoy a flawless, zero-latency 4K desktop experience running directly off the GPU dummy plug configuration.

### Workflow B: Controlling from your Android Phone (Mobile / Outside the House)

Follow this order when you are away from home with only your phone:

1. **Connect the Tunnel:** Open the **Tailscale app** on your Android phone and toggle it **ON**.
2. **Decrypt the Disk:** Open your mobile SSH app (e.g., JuiceSSH) and run your passwordless script to clear the pre-boot FileVault block.
3. **Bypass the Login Screen:**
   * Open **RealVNC Viewer** on your phone.
   * Connect to your Tailscale address: `pstt1.gila-karat.ts.net`.
   * When prompted, type your VNC bridge password: **`jfkdkd`**.
   * You will see the macOS login lock screen. Tap your user icon (`tom`) and type your main Mac system password.
   * Once your desktop dashboard loads, **close RealVNC Viewer**.
4. **Launch the Workspace:** Open the **Chrome Remote Desktop app** on your Android phone and select `pstt1`.
5. **Result:** Because Chrome Remote Desktop features a precision "Trackpad Mode" designed for small screens, you can seamlessly manage your processes with smooth touch tracking and zoom precision.
