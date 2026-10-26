# Samsung Galaxy A50 (SM-A505F) Complete Bootloader Unlock & Custom ROM Guide

Comprehensive step-by-step guide to unlock the bootloader, flash TWRP recovery, neutralize Samsung encryption/verity, and install custom ROMs on the Samsung Galaxy A50.

---

## 📱 Device Information & Current Live Status

Verified via active ADB connection (`RZ8M42HYX9T`):

| Property | Details |
| :--- | :--- |
| **Device Model** | Samsung Galaxy A50 (`SM-A505F` / `a50dd`) |
| **Chipset** | Samsung Exynos 9610 (Octa-core) |
| **Active Build ID** | `RP1A.200720.012.A505FDDS9CWA2` |
| **Android Version** | Android 11 (One UI 3.1) |
| **Binary / Bit Revision** | **Binary 9 (U9 / S9 / DDS9)** |
| **Security Patch** | 2023-01-01 |
| **Current Knox Warranty Bit**| `0x0` (Pristine, will trip to `0x1` upon unlocking) |
| **Battery Level** | ~58% (Safe to proceed, keep > 50%) |
| **Developer Options** | **USB Debugging:** ✅ Enabled<br>**OEM Unlocking:** ✅ Enabled |

---

## ⚠️ Important Warnings & Knox Consequences

Before proceeding, be aware of what happens when you unlock a Samsung bootloader:

* **Knox Permanent Trip (0x1):** Unlocking trips a physical eFuse inside the Exynos chip. This **cannot be reversed** even if you re-lock the bootloader.
* **Permanently Disabled Services:** Official Samsung Knox services will stop working permanently:
  * Samsung Pay / Samsung Wallet
  * Secure Folder
  * Samsung Pass / Samsung Health (Health can be patched via KnoxPatch)
* **Complete Data Wipe:** The device will undergo a factory reset during the bootloader unlock process and again during TWRP storage formatting. **Backup everything.**
* **Banking Apps & SafetyNet / Play Integrity:** Custom ROMs will require Magisk + Play Integrity Fix modules to pass device integrity and use banking apps (Google Pay, UPI, etc.).

---

## 🛠️ Required Files & PC Tools Checklist

Download these files onto your PC prior to starting:

1. **Samsung USB Drivers:** [Samsung Android USB Driver](https://developer.samsung.com/android-usb-driver)
2. **Odin Flash Tool:** Odin v3.14.4 (Official Samsung flashing utility)
3. **Custom Recovery:** TWRP for Galaxy A50 (`twrp-*-a505f.img.tar`) from [TWRP Official](https://twrp.me/samsung/samsunggalaxya50.html) or XDA.
4. **Multi-Disabler:** `multidisabler-samsung-*.zip` (critical to disable Knox VaultKeeper, dm-verity, and force-encryption).
5. **Custom ROM Zip:** (e.g. LineageOS 20 / crDroid / One UI modded port) + matching GApps (if ROM doesn't have GApps built-in).
6. **Magisk (Optional):** Latest `Magisk.apk` (renamed to `Magisk.zip` to flash via TWRP for root).
7. **External Storage (Recommended):** MicroSD Card or USB-OTG drive to store the ROM, GApps, and Multidisabler zips.

---

## 🚀 Phase 1: Bootloader Unlock Process

Because you already have **Developer Options**, **USB Debugging**, and **OEM Unlocking** enabled, follow these steps:

### Step 1.1: Boot into Download Mode
Connect your phone to the PC via USB cable.

**Method A: Fast ADB Command (Recommended)**
Open PowerShell or Command Prompt in your ADB directory and run:
```powershell
adb -s RZ8M42HYX9T reboot download
```

**Method B: Hardware Key Combo**
1. Power off the phone completely.
2. Press and hold **Volume Up + Volume Down** simultaneously.
3. While holding both buttons, plug the USB cable connected to your PC into the phone.
4. Release the buttons when the blue/teal warning screen appears.

---

### Step 1.2: Unlock the Bootloader
1. On the initial Download Mode warning screen, you will see an option:
   * `Long press Volume Up: Device unlock mode`
2. **Press and hold the Volume Up button** for ~5 seconds until the bootloader unlock confirmation screen appears.
3. Read the prompt: *"Unlock bootloader? All personal data will be erased."*
4. Press **Volume Up** once to confirm **"Yes"**.
5. The phone will format all internal user data and reboot automatically.

---

### Step 1.3: CRITICAL Post-Unlock Setup (Avoid KG / RMM Lock)
When the phone boots up for the first time after unlocking:
1. Complete the initial Samsung Setup Wizard quickly (you can skip Samsung account and Google account for now).
2. **MANDATORY:** Connect to **Wi-Fi or Mobile Data immediately**.
   * *Why?* Samsung's Knox Guard (KG) / RMM checks in with Samsung servers. If not connected to internet, the device enters `KG State: Prenormal` or `RMM: Prenormal`, which blocks flashing custom binaries in Odin.
3. Once connected to Wi-Fi, go to **Settings → About Phone → Software Information → Tap "Build Number" 7 times** to re-enable Developer Options.
4. Open **Settings → Developer Options**:
   * Confirm that **OEM Unlocking** is present and grayed out with the text *"Bootloader already unlocked"*.

---

## ⚡ Phase 2: Installing TWRP Custom Recovery

### Step 2.1: Reboot into Download Mode
Put the phone back into Download Mode:
```powershell
adb reboot download
```
*(Or power off -> hold Vol Up + Vol Down -> plug in USB cable -> press Vol Up once to proceed).*

---

### Step 2.2: Flash TWRP via Odin v3.14.4
1. Open **Odin3 v3.14.4.exe** on your PC as Administrator.
2. Check the **ID:COM** box in Odin — it should light up light blue (e.g., `0:[COMx]`) indicating the device is recognized.
3. Click the **Options** tab in Odin:
   * ⚠️ **UNCHECK "Auto Reboot"** *(Crucial! If the phone reboots automatically into stock One UI, the stock system will immediately overwrite TWRP with the stock recovery).*
   * Leave "F. Reset Time" checked.
4. Click the **AP** button and browse to your downloaded `twrp-*-a505f.img.tar`.
5. Click **Start**.
6. Wait a few seconds until the top-left box displays a green **PASS!**.

---

### Step 2.3: Boot Directly into TWRP (Manual Transition)
Since Auto Reboot was unchecked, the phone will remain in Download Mode. You must manually force it into TWRP:

1. Keep the phone plugged into the PC via USB cable.
2. Press and hold **Volume Down + Power (Side Key)** together.
3. As soon as the screen goes black (the exact second it powers off), **immediately release Volume Down and hold Volume Up + Power**.
4. Keep holding **Volume Up + Power** until the Samsung logo appears and then boots into the **Team Win Recovery Project (TWRP)** screen.
5. Swipe the slider at the bottom: **"Swipe to Allow Modifications"**.

---

## 🛡️ Phase 3: Decrypt Storage & Flash Multi-Disabler

Samsung's stock kernel enforces File-Based Encryption (FBE) and dm-verity. If not disabled, TWRP cannot read internal storage and custom ROMs will fail to boot.

1. In TWRP main menu, tap **Wipe** → **Format Data**.
2. Type **`yes`** on the keyboard and press the blue checkmark.
   * *This removes encryption and allows TWRP full read/write access.*
3. Go back to the TWRP main menu, tap **Reboot** → **Recovery**.
   * *The phone will reboot back into TWRP.*
4. Transfer `multidisabler-samsung-*.zip` to the phone (via MTP, MicroSD card, or `adb push`):
   ```powershell
   adb push multidisabler-samsung-3.x.zip /sdcard/
   ```
5. In TWRP, tap **Install** → Select `multidisabler-samsung-*.zip` → **Swipe to confirm Flash**.
6. (Optional) Run Wipe -> Format Data -> `yes` once more if prompted by multidisabler.

---

## 💿 Phase 4: Flashing a Custom ROM

### Recommended Custom ROM Options for SM-A505F (Binary 9 Base):

| ROM | Android Version | Best For | Notes |
| :--- | :--- | :--- | :--- |
| **LineageOS 20 (Unofficial)** | Android 13 | Maximum stability, clean AOSP, daily driver | High battery life, smooth performance |
| **crDroid 9 / 10** | Android 13 / 14 | Heavy customization, gaming tweaks | Based on LineageOS with extra UI controls |
| **Fresh ROM / Snake OS** | Android 11 (One UI 3.1) | Camera & Hardware compatibility | Keeps Samsung Camera quality, VoLTE, and optical in-display fingerprint working 100% |
| **Treble GSI (A/B ARM64)** | Android 13 / 14 | Testing newest Android versions | Requires Treble vendor fix / Mint Kernel |

---

### Step-by-Step ROM Installation in TWRP:

1. **Place ROM & Add-ons on Device:**
   * Copy your **ROM.zip**, **GApps.zip** (e.g. NikGApps / MindTheGapps ARM64 for Android 13), and **Magisk.zip** (optional) to an external MicroSD card or via ADB:
     ```powershell
     adb push custom-rom.zip /sdcard/
     adb push gapps.zip /sdcard/
     ```

2. **Clean Wipe:**
   * In TWRP, tap **Wipe** → **Advanced Wipe**.
   * Check the following partitions:
     * ✅ **Dalvik / ART Cache**
     * ✅ **Cache**
     * ✅ **System**
     * ✅ **Vendor**
     * ✅ **Data**
   * ⚠️ *Do NOT check Internal Storage or Micro SDCard if your installation zips are stored there.*
   * **Swipe to Wipe**.

3. **Install ROM & GApps:**
   * Tap **Install** from the main menu.
   * Select your **ROM.zip** file.
   * Tap **"Add more Zips"** and select your **GApps.zip** (if using a vanilla/non-GApps ROM build).
   * **Swipe to confirm Flash**.
   * Wait for the installation script to complete successfully.

4. **Install Magisk (Optional for Root):**
   * Tap **Install** → Select `Magisk.zip` → **Swipe to confirm Flash**.

5. **Final Wipe & Reboot:**
   * Tap **Wipe Cache/Dalvik**.
   * Tap **Reboot System**.
   * *Note: The first boot into a freshly installed custom ROM typically takes between 5 to 10 minutes. Do not interrupt the device.*

---

## 🔧 Troubleshooting & Common Issues

### 1. `KG State: Prenormal` or `RMM State: Prenormal` in Download Mode
* **Symptoms:** Odin shows `FAIL! (Auth)` or `Custom Binary blocked by OEM Lock / KG State`.
* **Fix:** Boot phone into stock OS, connect to Wi-Fi, go to Settings → Software Update → Check for Updates. Keep phone powered on with internet for a few minutes until KG state transitions from `Prenormal` to `Completed` / `Checking`.

### 2. Odin Stuck at `<ID:0/00x> Setup Connection...`
* **Fix:** Reinstall Samsung USB Drivers, use an original Samsung USB Type-C cable, and plug into a motherboard USB 2.0 port rather than a USB 3.0/hub port.

### 3. Phone Bootloops back to Stock Recovery instead of TWRP
* **Cause:** The phone rebooted into Samsung One UI before TWRP could be booted, causing Samsung's `recovery-from-boot.p` to reflash stock recovery.
* **Fix:** Re-flash TWRP in Odin with **Auto-Reboot UNCHECKED**. Perform the manual button transition (`Vol Down + Power` -> immediate switch to `Vol Up + Power` with cable connected) directly into TWRP.

### 4. Internal Storage Shows 0 MB or Unable to Mount in TWRP
* **Fix:** Tap **Wipe** → **Format Data** → type `yes`. Then tap **Reboot** → **Recovery**.

### 5. In-Display Fingerprint Sensor or VoLTE Issues on AOSP
* **Note:** Samsung uses proprietary VoLTE and optical fingerprint drivers (Goodix/Synaptics) tied to their One UI HAL. On pure AOSP/LineageOS, VoLTE may not work (carrier dependent), and the in-display fingerprint may require specific kernel patches. If native camera processing and VoLTE are mandatory for your usage, consider a debloated One UI port (like Fresh ROM or SnakeOS).

---

## 🔗 Useful Community & Download Links

* **XDA Galaxy A50 SM-A505F Forum:** [XDA Developers - Samsung Galaxy A50](https://xdaforums.com/c/samsung-galaxy-a50.8757/)
* **Official TWRP for Galaxy A50 (a505f):** [TWRP Official Devices](https://twrp.me/samsung/samsunggalaxya50.html)
* **Odin v3.14.4 Official Tool:** [Odin Download via Samsung Community](https://samsungodin.com/)
* **Multi-Disabler for Samsung Devices:** [GitHub - multidisabler-samsung](https://github.com/ianmacd/multidisabler-samsung)
* **LineageOS 20 for Galaxy A50:** [XDA Unofficial LineageOS 20 Thread](https://xdaforums.com/t/rom-13-0-unofficial-lineageos-20-for-galaxy-a50.4571991/)
* **crDroid for Galaxy A50:** [XDA crDroid Thread](https://xdaforums.com/c/samsung-galaxy-a50.8757/)

---

*Last Updated: 2026-10-26 | Generated for SM-A505F (Binary 9)*
