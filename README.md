## Installing macOS on the Google Pixelbook. 
It turns out that the latest verion(s) of macOS work quite well on the Google Pixelbook. 

--------------------------------------------------------------------------------------------------------------------------------------------------------
### ⚠️ READ ME!!
Pixelbook owners with an Samsung 512GB NVMe SSD should be aware that problems **will** arise during the installation process. This is primarily due to the fact that the Samsung SSD does not play well with macOS, which could result in degraded performance and prolonged boot times.

If you have an Pixelbook with eMMC, you may proceed without any issue.

--------------------------------------------------------------------------------------------------------------------------------------------------------

## Table of Contents

- [Specs](#some-notable-specs)
- [Current Status](#current-status)
- [Mandatory Disclaimer](#%EF%B8%8F-mandatory-disclaimer)
- [Requirements](#requirements)
- [**Installation**](#installation)
- [Post Install:](#post-install)
- [Misc. Information](#misc)
- [macOS Ventura](#macos-ventura) 
  - [Preparations](#preparations)
  - [Installing Ventura directly:](#preparations-for-installing-ventura-directly)
  - [Fixing WiFi](#fixing-wifi-on-ventura)

--------------------------------------------------------------------------------------------------------------------------------------------------------

| OpenCore Boot Menu | macOS Monterey |
|------------|-------------|
|<img src="https://github.com/olm3ca/PixelbookOSX/blob/36792715a2478665c79cab32f55d72ac2526062a/Pixelbook%20Opencore%20Boot.jpg?raw=true" width="300">|<img src="https://raw.githubusercontent.com/olm3ca/Pixel-Slate/main/NocturneMacOS.png" width="600">|

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Some notable specs:
-  Intel Core i5 7Y57 / i7 
-  Intel UHD 615
-  Intel WiFi Chip: 7265
-  Keyboard - PS/2
-  Touchpad: HID-over-I²C
-  Suspend: ACPI S3 sleep
-  Audio Codec: Uses unsupported codec `max98927` as speakers and `RT5514` as DMIC on SSP0 and RT5663 as headset on SSP1. Not currently working, and may never work. As a cheap and easy workaround, a [$10 USB sound adapter](https://www.amazon.com/Syba-external-Adapter-Windows-C-Media/dp/B001MSS6CS) or Bluetooth audio work perfectly well. 

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Current Status

| **Feature**        | **Status**           | **Notes**                                                                                     |
|--------------------|----------------------|-----------------------------------------------------------------------------------------------|
| WiFi               | Working              | With `itlwm.kext v2.1.0 stable` and `Heliport v1.4.1` `(Latest)`.                             |
| Bluetooth          | Working              | With `IntelBluetoothFirmware` and `BlueToolFixup.kext`.                                       |
| Suspend / Sleep    | Working partially    | Only on battery power, working with `EmeraldSDHC.kext`.                                       |
| Trackpad           | Working              | Install [Karabiner](https://karabiner-elements.pqrs.org) and go to Devices to enable ID 6353  |          
| Graphics Accel.    | Working              | With `-igfxnotelemetryload` in the `boot-args`.                                               |
| Internal Speakers  | Not working          | Unsupported codec.                                                                            |
| Keyboard backlight | Not Working          |                                                                                               |           
| Keyboard Remaps    | Working partially    | Install [Karabiner](https://karabiner-elements.pqrs.org) and use that to remap top row keys.  |
| eMMC Storage       | Working              | With `EmeraldSDHC.kext`.                                                                      |
| USB Ports          | Working              | Working OOTB, still advised to map USB ports.                                                 |
| Webcam             | Not Working          |                                                                                               |
| Internal Mic.      | Not working          | Same reason why internal speakers don't work; unsupported codec.                              |
| Logout / Lock      | Working              | Working OOTB.                                                                                 |
| Shutdown / Restart | Working              |                                                                                               |
| Touchscreen        | Working              | With `VoodooI2C.kext` and `VoodooI2CHID.kext`                                                 |
| Screen backlight   | Working partially    | Using BetterDisplay helps as a stopgap solution.                                              |

--------------------------------------------------------------------------------------------------------------------------------------------------------

### ⚠️ Mandatory Disclaimer

**By continuing, you acknowledge that you have read and understood the contents of the following disclaimer, and consent to their terms.**

**The process described in this document could cause irreversible damage to your laptop, and you should prepare yourself for that outcome before you begin. I accept absolutely no responsibility for the consequences of anyone choosing to follow or ignore any of the instructions in this document, and make no guarantees about the quality or effectiveness of the software in this repo.**

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Requirements

Before you start, you'll need to have the following things to complete the process:

- A SuzyQable CCD Debugging cable (or [make your own](https://www.reddit.com/r/chrultrabook/comments/uaiz1q/making_a_chromeos_suzy_q_cable_tutorial/)) if you haven't disabled write protect already.
- A USB-A to USB-C adapter
- 1 USB flash drive with USB-C connectors or adapters for your OpenCore USB.
- OpenCore version 0.8.8 or newer for proper boot device selection.
- **A willingness to accept that this is a potentially destructive process that may render your
  expensive Pixelbook inoperable or otherwise busted. See the [disclaimer](#disclaimer) below.**

--------------------------------------------------------------------------------------------------------------------------------------------------------

## Installation

Here are the steps to go from stock Pixelbook to a macOS 12 install using OpenCore:

1. If you haven't already, flash UEFI firmware. Read and follow [MrChromebox's instructions](https://mrchromebox.tech) on how to flash the UEFI firmware using MrChromebox's scripts. To do this, you will need to disable write protect with either the SuzyQable cable or by removing the battery. 

2. Setup your EFI folder using the [OpenCore Guide](https://dortania.github.io/OpenCore-Install-Guide/). Use [Laptop Kaby Lake & Amber Lake Y](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html#starting-point) for your `config.plist`. 

3. Re-visit this guide when you're done setting up your EFI. There are a few things we need to tweak to ensure our Pixelbook works with macOS. Skipping these steps will result in a **very** broken hack.
 
3. In your `config.plist`, under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`, add `-igfxnotelemetryload` to your `boot-args`. **Without it, you will NOT have iGPU acceleration**. translation: it will run horribly. 

4.  Under `DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0)`, make the following modifications:
 
     | AAPL,ig-platform-id  | data | 0000C087 |
     | -------------------- | ---- | -------- |
     | device-id            | data | C0870000 |
   
     _**These should be the only two items `in PciRoot(0x0)/Pci(0x2,0x0)`.**_

6. **Set your SMBIOS as MacBookAir8,1**. Ignore what the guide tells you to use, `MacBookAir8,1` works better with our laptop.
   - If you choose to use `MacBook10,1` which also works, you will NOT have Low Battery Mode.

7. Download [EmeraldSDHC](https://github.com/acidanthera/EmeraldSDHC/releases) for eMMC storage support. Put it in your Kexts folder. 

8. Download corpnewt's SSDTTime, open it, select the first option `FixHPET`, choose `C` for default, and drag the SSDT it makes (`SSDT-HPET.aml`) into your `ACPI` folder. Then, in the same folder, copy the patches from `oc_patches.plist` into your config.plist under `ACPI -> Patch`. Without it, eMMC won't be recognized by macOS.

9. Snapshot (cmd +r) or (ctrl + r) your `config.plist`. 

10. Now, boot from the OpenCore USB. In Disk Utility, go to Show All Devices in the top left, and then select the entire drive to format it as APFS. Begin the install after.
      - After some while, it will reboot. Go back into the boot menu and select your macOS Monterey install . In the OpeCore boot menu, you should now see "Install macOS Monterey" as a menu item. Select that to continue the installation. 
    - The second phase of the installation will continue for about ~25-30 minutes. If it appears to be stuck, **do NOT** shut it down. It is not stuck. It just takes a while.

11. After installing, make sure to copy your EFI from the USB Drive over to your EFI Partition in your eMMC drive. corpnewt's MountEFI is great for doing this.   


--------------------------------------------------------------------------------------------------------------------------------------------------------

### Post install: 
   - To fix sleep, you may want to follow these steps from the OC [guide fix here](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html#preparations)
   - Sound currently works via Bluetooth or a USB sound adapter. 
   - [Karabiner](https://karabiner-elements.pqrs.org), can make the touchpad functional, but not great. It's also helpful for remapping the keyboard to match what the Pixelbook F1-F10 keys do.

**Note:** Required for Monterey: Make your own `USBMap.kext`, by following the OpenCore [guide here](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html)

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Misc.
- Read the [OpenCore guide](https://dortania.github.io/OpenCore-Install-Guide/) on how to improve this hackintosh build and contribute here.
- *When formatting the eMMC drive in Disk Utility, make sure to toggle "Show all Drives" and **erase the WHOLE drive**, not just the current partition.
- Format the drive as `APFS`
- Map your USB ports prior to installing macOS for a painless install. You **will** reget it if you don't. You can use [USBToolBox](https://github.com/USBToolBox/tool) to do that. If you are using USBToolBox (Mainly Windows users), you need a second kext that goes along with it. [Github repo here](https://github.com/USBToolBox/kext). USBToolBox will not work without this kext. 
- `itlwm` is more stable & faster than `AirportItlwm`
- You might have DRM issues, there's no fix for this. :(
- To fix the battery life on Ventura, you can set Low Battery Mode to be always enabled on battery. It's not perfect, but it helps. You can also use CPUFriend to tweak power settings but it might break sleep.
- eMMC will come up as an external drive in the boot picker since eMMC is just an embedded SD card. Nothing you can do about it.
- To hide the drive picker, set `ShowPicker` to `False` in `Misc` ->` Boot` -> `ShowPicker`
- `AppleXcpmCfgLock` and `DisableIOMapper` can be enabled or disabled. Makes no difference.
- It's worth noting that while it's recommended, coreboot already includes mapped USB ports, meaning that USB mapping is not required. Proceed at your  own risk if you decide to skip USB mapping.
- Make sure your `ScanPolicy` is set to `0`. eMMC will not be recognized if it's some other value.
- Please report any broken links in issues. Half this guide was written while I was high. /s
- **USB ports will ONLY work with SSDT-USB-Reset** 
#### *Note: The hotkey to show drives **DOES NOT WORK**. Make a copy of your EFI with `ShowPicker` enabled if you need to boot from another drive.

--------------------------------------------------------------------------------------------------------------------------------------------------------

## macOS Ventura
Before beginning, it's important to keep the following in mind:

- Your battery life will decrease more quickly while using Ventura. To prevent this, it is recommended to stick with Monterey or an earlier version.
- AirportItlwm's Ventura variant does not play with our WiFi card, so you need to be using itlwm at all times.
With that, let's get started!

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Preparations 
Note: These steps can be completed after updating, however, you won't have WiFi during this time. For a smoother experience, it is recommended to perform the following steps before updating.

1. Mount your EFI using Corpnewt's MountEFI.
2. In the OC/Kexts folder, delete your previous `itlwm`/`AirportItlwm` kext and replace it with `itlwm v.2.2.0 alpha`.
3. If you haven't already, download and install Heliport, with the most recent stable release being sufficient.
4. Launch ProperTree and reload (by pressing `ctrl+r`) your `config.plist`.
5. Start the update process (if you haven't already).
6. [See below](#fixing-wifi-on-ventura) for fixing WiFi.

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Preparations for installing Ventura directly:
Note: For Windows only, not sure how it's like on Linux.
1. Under OC/Kexts, delete your old itlwm/AirportItlwm kext and replace it with `itlwm v.2.2.0 alpha`
2. Open up your kext folder, and locate `itlwm.kext`. 
3. Open it, and find `Info.plist`.
4. Open ProperTree, navigate to where your `Info.plist` is, and open it.
5. Under `IOKitPersonalities -> itlwm -> WiFiConfig`, enter your WiFI details. Save and close when done.
7. Launch ProperTree and reload (`ctrl+r`) your `config.plist`. 
8. Boot recovery. There will be no WiFi logo/symbol, but you will have WiFi. If you are able to install macOS, then you have not fucked up.

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Fixing WiFi on Ventura 
**Only for those that have macOS installed but haven't edited their `Info.plist`.**

1. Mount your EFI
2. Open up your kext folder, and locate `itlwm.kext`. 
3. Click it, and select `Show Package Contents` and open the Contents folder. Once inside, find the `Info.plist` and copy it via  `ctrl + c`.
4. Exit `itlwm.kext` and go back to your Kext folder. Paste your `Info.plist`.
5. Open ProperTree, select the `Info.plist` in your Kext folder and open it.
6. Under `IOKitPersonalities -> itlwm -> WiFiConfig`, enter your WiFI details. Save and close when done.
7. Replace the old `Info.plist` with the one you just edited. 


Do note that Heliport will report no WiFi upon logging in but keep in mind you actually do thanks to the edits we just made. :) 

--------------------------------------------------------------------------------------------------------------------------------------------------------

That's it. Feel free to share your ideas of how to improve this process and the hardware compatibility. 
