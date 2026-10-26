# Samsung Galaxy A50 (SM-A505F) Bootloader Unlock & Custom ROM Guide

A complete, battle-tested, end-to-end guide for unlocking the bootloader, flashing custom recoveries (TWRP / OrangeFox), fixing Android Verified Boot (AVB) errors, bypassing Samsung Knox encryption, and installing custom ROMs (Android 13 / LineageOS 20) with Google Apps (GApps).

---

## 📱 Device Specifications & Target Firmware

- **Model:** Samsung Galaxy A50 (`SM-A505F` / `a50dd` / `INS`)
- **Chipset:** Samsung Exynos 9610 (Octa-core)
- **Base OS Version:** Android 11 (One UI 3.1)
- **Base Build ID:** `RP1A.200720.012.A505FDDS9CWA2`
- **Bootloader / Binary (Bit) Version:** **Binary 9 (U9 / S9 / DDS9)**
- **Target OS:** LineageOS 20.0 (Android 13, Tiramisu) + MindTheGapps

---

## ⚠️ Important Warnings & Disclaimers

> [!CAUTION]
> - **Permanent Knox Trip (0x1):** Unlocking the bootloader blows an internal physical hardware eFuse. Knox warranty bit will change from `0x0` to `0x1`. This is permanent and hardware-level; it cannot be reversed even if you re-lock the bootloader.
> - **Knox Services Disabled:** Secure Folder, Samsung Pay, and Samsung Pass will no longer work.
> - **Full Data Wipe:** Unlocking the bootloader and formatting data will erase all personal files, media, and apps. Always back up critical data beforehand.

---

## 🛠️ Required Downloads & Tools Checklist

| Category | File / Tool | Purpose |
| :--- | :--- | :--- |
| **Flashing Utility** | [Samsung Odin v3.14.4](https://odindownloader.com/) | Official tool to flash partition archives to Samsung devices in Download Mode. |
| **PC Drivers** | [Samsung Android USB Driver v1.9.5+](https://developer.samsung.com/android-usb-driver) | Required for Windows to recognize COM/Modem ports in Odin mode. |
| **Custom Recovery** | `OrangeFox-R11.0-Stable-a50dd` / `TWRP 3.7.0` | Custom recovery for formatting partitions, flashing ZIPs, and ADB/MTP access. |
| **Security Patch** | `vbmeta.tar` (Disabled AVB) | Disables verification checks (`--flags 2`) to stop `ODIN MODE (AVB Fail)`. |
| **Custom ROM** | `lineage-20.0-*-UNOFFICIAL-a505f.zip` | Android 13 Custom ROM for Exynos 9610 / SM-A505F. |
| **Google Apps** | `MindTheGapps-13.0-arm64.zip` | Official LineageOS-recommended Google Play Store and Services package. |
| **Security Disabler** | `multidisabler-samsung-3.5-corsicanu.zip` | Disables File-Based Encryption (FBE), Vaultkeeper, and stock recovery restoration. |
| **Root (Optional)** | `Magisk-v27.0.zip` | Systemless root and Play Integrity bypass. |

---

## 🚀 Step-by-Step Installation Walkthrough

### Phase 1: Bootloader Unlock

1. **Enable Developer Options:**
   - Go to **Settings** → **About Phone** → **Software Information**.
   - Tap **Build Number** 7 times until you see *"Developer mode has been turned on"*.
2. **Enable OEM Unlocking:**
   - Go to **Settings** → **Developer Options**.
   - Toggle **OEM Unlocking** and **USB Debugging** to **ON**.
3. **Reboot to Warning Screen:**
   - Power off the device completely.
   - Hold **Volume Up + Volume Down** simultaneously.
   - While holding both buttons, connect the USB cable to your PC.
4. **Initiate Device Unlock Mode:**
   - On the teal Warning screen, **long-press Volume Up (5–7 seconds)** to enter Device Unlock Mode.
   - Press **Volume Up once** to confirm *"Unlock bootloader? All personal data will be erased"*.
   - The device will format data and reboot.
5. **Initial Boot & Wi-Fi Check (Critical):**
   - Quickly complete the setup wizard and **connect to Wi-Fi immediately**.
   - *Why?* This registers with Samsung's Knox Guard server to prevent your device from entering `KG State: Prenormal` (which blocks custom binaries in Odin).
   - Re-enable Developer Options and confirm that **OEM Unlocking** is grayed out with *"Bootloader already unlocked"*.

---

### Phase 2: Flashing Custom Recovery & Bypassing AVB

Samsung devices with Android 11 (Binary 9) enforce Android Verified Boot (AVB). Flashing recovery alone will trigger `ODIN MODE (AVB Fail) - recovery: Error verifying vbmeta image: Invalid vbmeta header`. You must flash a patched `vbmeta.tar` alongside your custom recovery.

1. **Create/Verify `vbmeta.tar`:**
   Generate a disabled vbmeta image using `avbtool.py`:
   ```bash
   python avbtool.py make_vbmeta_image --flags 2 --padding_size 4096 --output vbmeta.img
   python -c "import tarfile; tar = tarfile.open('vbmeta.tar', 'w', format=tarfile.USTAR_FORMAT); tar.add('vbmeta.img', arcname='vbmeta.img'); tar.close()"
   ```
2. **Package OrangeFox / TWRP:**
   Ensure `recovery.img` is packaged in an Odin-compatible USTAR format:
   ```bash
   python -c "import tarfile; tar = tarfile.open('OrangeFox_recovery.tar', 'w', format=tarfile.USTAR_FORMAT); tar.add('recovery.img', arcname='recovery.img'); tar.close()"
   ```
3. **Boot Device into Download Mode:**
   - Power off $\rightarrow$ hold **Volume Up + Volume Down** $\rightarrow$ plug in USB $\rightarrow$ press **Volume Up once**.
4. **Configure Odin v3.14.4:**
   - Verify `ID:COM` turns light blue (`0:[COMx]`).
   - Go to **Options** tab:
     - ❌ **Auto Reboot:** **UNCHECK** (Essential! Prevents stock ROM from overwriting recovery).
     - ❌ **Re-Partition:** **UNCHECK** (Never check this without a PIT file).
     - ✅ **F. Reset Time:** **CHECK**.
   - In **BL** slot: Load `vbmeta.tar`.
   - In **AP** slot: Load `OrangeFox_recovery.tar` (or `twrp-*-a505f.img.tar`).
   - Click **Start** and wait for **PASS!**.

---

### Phase 3: Transitioning into Custom Recovery

Since Auto-Reboot was unchecked, the phone remains on the Download screen:
1. Keep the USB cable connected to the PC.
2. Press and hold **Volume Down + Power** for ~7 seconds until the screen goes pitch black.
3. The instant it turns off:
   - **Immediately release Volume Down and hold Volume Up + Power**.
4. Keep holding **Volume Up + Power** until the OrangeFox/TWRP splash screen appears.
5. Release buttons.

---

### Phase 4: Formatting Encryption & Installing LineageOS 20

1. **Format Data (Remove Samsung FBE):**
   - In recovery, go to **Wipe** $\rightarrow$ **Format Data**.
   - Type **`yes`** and confirm.
   - Go to **Reboot** $\rightarrow$ **Recovery** to reboot recovery with cleanly mounted storage.
2. **Transfer ROM Files (MTP / USB / ADB):**
   - Connect to PC.
   - Copy the following files to internal storage or MicroSD:
     - `lineage-20.0-*-UNOFFICIAL-a505f.zip`
     - `MindTheGapps-13.0-arm64.zip`
     - `multidisabler-samsung-3.5-corsicanu.zip`
     - `Magisk-v27.0.zip` (Optional)
3. **Advanced Wipe:**
   - Go to **Wipe** $\rightarrow$ **Advanced Wipe**.
   - Check: `Dalvik / ART Cache`, `Cache`, `System`.
   - *(Do NOT wipe Internal Storage where your zips are located).*
   - Swipe to wipe.
4. **Flash Packages in Order (Critical Order):**
   - **Step 1:** Flash `lineage-20.0-*-UNOFFICIAL-a505f.zip`.
   - **Step 2:** Flash `MindTheGapps-13.0-arm64.zip` (Must be flashed immediately after ROM without rebooting).
   - **Step 3:** Flash `multidisabler-samsung-3.5-corsicanu.zip` (Disables Knox encryption and auto-recovery restore).
   - **Step 4 (Optional):** Flash `Magisk-v27.0.zip` for root access.
5. **Reboot to System:**
   - Tap **Reboot System**.
   - *First boot takes approximately 5–8 minutes while Android 13 initializes.*

---

## 🔧 Common Errors & Solutions

| Error Message | Cause | Solution |
| :--- | :--- | :--- |
| `ODIN MODE (AVB Fail) - recovery: Error verifying vbmeta image` | Stock `vbmeta` cryptographic check rejected custom recovery. | Flash `vbmeta.tar` (flags=2) in Odin's **BL** slot alongside recovery in **AP**. |
| `Can't open the specified file (Line: 2006)` | `Re-Partition` checkbox enabled in Odin without PIT file. | Uncheck **Re-Partition** under Odin Options tab. |
| `SetupConnection.. stuck in Odin` | Broken session from previously failed flash. | Close Odin, hold Vol Down + Power for 7s to restart Download Mode, reopen Odin. |
| `Error 70 in Recovery during GApps` | Insufficient space in dynamic system partition. | Use lightweight **NikGapps Core** or clean-flash LineageOS before GApps. |
| `Device not recognized in Odin` | Generic Microsoft serial driver (`usbser.inf`) bound instead of Samsung driver. | Install `SAMSUNG_USB_Driver.exe` and re-plug USB cable. |

---

## 📜 Credits & References

- [TeamWin (TWRP)](https://twrp.me/)
- [OrangeFox Recovery Project](https://orangefox.download/)
- [LineageOS Project](https://lineageos.org/)
- [MindTheGapps](https://gitlab.com/MindTheGapps/vendor_gapps)
- [Ianmacd & Corsicanu (multidisabler-samsung)](https://github.com/corsicanu/multidisabler-samsung)
- [XDA Developers Samsung Galaxy A50 Community](https://xdaforums.com/c/samsung-galaxy-a50.8757/)
