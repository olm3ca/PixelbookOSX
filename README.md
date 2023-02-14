# Installing macOS on the Google Pixelbook. 
It turns out macOS Monterey works quite well on this hardware. 


| OpenCore Boot Menu | macOS Monterey |
|------------|-------------|
|<img src="https://github.com/olm3ca/PixelbookOSX/blob/36792715a2478665c79cab32f55d72ac2526062a/Pixelbook%20Opencore%20Boot.jpg?raw=true" width="300">|<img src="https://raw.githubusercontent.com/olm3ca/Pixel-Slate/main/NocturneMacOS.png" width="600">|


## Some notable specs:
-  Intel Core i5 7Y57 / i7 
-  Intel UHD 615
-  Intel WiFi Chip: 7265
-  Keyboard - PS/2
-  Touchpad: HID-over-I²C
-  Suspend: ACPI S3 sleep
-  Audio Codec: Uses unsupported codec `max98927` as speakers and `RT5514` as DMIC on SSP0 and RT5663 as headset on SSP1. Not currently working, and may never work. As a cheap and easy workaround, a [$10 USB sound adapter](https://www.amazon.com/Syba-external-Adapter-Windows-C-Media/dp/B001MSS6CS) or Bluetooth audio work perfectly well. 

--------------------------------------------------------------------------------------------------------------------------------------------------------


## Current Status

| **Feature**        | **Status**           | **Notes**                                                                                     |
|--------------------|----------------------|-----------------------------------------------------------------------------------------------|
| WiFi               | Working              | With `itlwm.kext v2.1.0 stable` and `Heliport v1.4.1` `(Latest)`.                             |
| Bluetooth          | Working              | With `IntelBluetoothFirmware` and `BlueToolFixup.kext`.                                       |
| Suspend / Sleep    | Working partially    | Only on battery power, working with `EmeraldSDHC.kext`.                                       |
| Trackpad           | Working              | Install [Karabiner](https://karabiner-elements.pqrs.org) and go to Devices to enable ID 6353  |                                                                                           
| Graphics Accel.    | Working              | With `-igfxnotelemetryload` in the `boot-args`.                                               |
| Internal Speakers  | Not working          | Unsupported codec.                                                                            |
| Keyboard backlight | Not Working          |                                                                                               |                                           
| Keyboard Remaps    | Not Working          |                                                                                               |
| eMMC Storage       | Working              | With `EmeraldSDHC.kext`.                                                                      |
| USB Ports          | Working              | Make sure to map your USB ports with `USBMap.kext`(macOS) or `USBToolbox.kext` (Windows/Linux).|
| Webcam             | Not Working          |                                                                                                |
| Internal Mic.      | Not working          | Same reason why internal speakers don't work; unsupported codec.                              |
| Logout / Lock      | Working              | Working OOTB.                                                                                 |
| Shutdown / Restart | Working              |                                                                                               |
| Touchscreen        | Working              | With `VoodooI2C.kext` and `VoodooI2CHID.kext`                                                     |
| Screen backlight   | Not Working          |                                                                                               |

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

### ⚠️ Mandatory Disclaimer

**The process described in this document could cause irreversible damage to your laptop, and
you should prepare yourself for that outcome before you begin. I accept absolutely no responsibility for the consequences of anyone choosing to follow or ignore any of the instructions in this document, and make no guarantees about the quality or effectiveness of the
software in this repo.**


--------------------------------------------------------------------------------------------------------------------------------------------------------



## Installation

Here are the steps to go from stock Pixelbook to a macOS 12 install using OpenCore:

1. If you haven't already, flash UEFI firmware. Read and follow [MrChromebox's instructions](https://mrchromebox.tech) on how to flash the UEFI firmware using MrChromebox's scripts. To do this, you will need to disable write protect with either the SuzyQable cable or by removing the battery. 

2. Setup your EFI folder using the [OpenCore Guide](https://dortania.github.io/OpenCore-Install-Guide/). Use Kaby Lake Laptop for your `config.plist`.

3. In your `config.plist`, under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82`, add `-igfxnotelemetryload` to your `boot-args`. **Without it, you will NOT have iGPU acceleration**. translation: it will run horribly. 

4. Download corpnewt's SSDTTime, open it, select the first option `FixHPET`, choose `C` for default, and drag the SSDT it makes (`SSDT-HPET`) into your `ACPI` folder. Then, in the same folder, copy the patches from `oc_patches.plist` into your config.plist under `ACPI -> Patch`. Without it, eMMC won't be recognized by macOS.

5. Now, boot from the OpenCore USB. In Disk Utility, go to Show All Devices in the top left, and then select the entire drive to format it as APFS. Begin the install after.
      - After some while, it will reboot. Go back into the boot menu and select your macOS Monterey install . In the OpeCore boot menu, you should now see "Install macOS Monterey" as a menu item. Select that to continue the installation. 
    - The second phase of the installation will continue for about ~25-30 minutes. If it appears to be stuck, **do NOT** shut it down. It is not stuck. It just takes a while.

6. After installing, make sure to copy your EFI from the USB Drive over to your EFI Partition in your eMMC drive. corpnewt's MountEFI is great for doing this.   


--------------------------------------------------------------------------------------------------------------------------------------------------------

### Post install: 
   - To fix sleep, you may want to follow these steps from the OC [guide fix here](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html#preparations)
   - Sound currently works via Bluetooth or a USB sound adapter. 
   - [Karabiner](https://karabiner-elements.pqrs.org), can make the touchpad functional, but not great. It's also helpful for remapping the keyboard to match what the Pixelbook F1-F10 keys do.

**Note:** Required for Monterey: Make your own USBMap.kext, by following the OpenCore [guide here](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html)

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Misc.
- Read the [OpenCore guide](https://dortania.github.io/OpenCore-Install-Guide/) on how to improve this hackintosh build and contribute here.
- Despite what the OpenCore guide says, your SMBIOS should be `MacBookAir8,1`. `MacBook10,1` works too, but lacks some features.
- To hide the drive picker, set `ShowPicker` to `False` in `Misc` ->` Boot` -> `ShowPicker`
- `AppleXcpmCfgLock` and `DisableIOMapper` can be enabled or disabled. Makes no difference.
  
 #### *Note: The hotkey to show drives **DOES NOT WORK**. Make a copy of your EFI with `ShowPicker` enabled if you need to boot from another drive.

--------------------------------------------------------------------------------------------------------------------------------------------------------
### macOS Ventura
Before beginning, it's important to keep the following in mind:

- Your battery life will decrease more quickly while using Ventura. To prevent this, it is recommended to stick with Monterey or an earlier version.
- The Intel WiFi connection may be unstable at startup but should work after a few seconds (approximately 20 seconds) following login.
- To install Ventura, you must first install macOS 12 (Monterey), then perform an update through System Preferences as the WiFi card is not currently     compatible with `AirportItlwm`'s Ventura variant.
- **Has not been tested yet. Proceed at your OWN RISK.**

With that, let's get started!

### Preparation 
Note: These steps can be completed after updating, however, you won't have WiFi during this time. For a smoother experience, it is recommended to perform the following steps before updating.

1. Mount your EFI using Corpnewt's MountEFI.
2. In the OC/Kexts folder, delete your previous `itlwm`/`AirportItlwm` kext and replace it with `itlwm v.2.2.0 alpha`.
3. If you haven't already, download and install Heliport, with the most recent stable release being sufficient.
4. Launch ProperTree and reload (by pressing `ctrl+r`) your `config.plist`.
5. Start the update process (if you haven't already).

--------------------------------------------------------------------------------------------------------------------------------------------------------

That's it. Feel free to share your ideas of how to improve this process and the hardware compatibility. 




