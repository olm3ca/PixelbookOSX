# Installing Mac OS on the Google Pixelbook - now with working VRAM! 
We've managed to install GNU/Linux, Windows and a lot of other hacks on this beautiful laptop... so why not Mac OS? 

## This is a work-in-progress guide.
Not all of the functionality is working. This will be updated frequently as fixes are identified. At the moment this guide is focused on Mac OS 10.14.6 Mojave as it has the best support for our hardware.

## A note on Pixelbook hardware
For OS X purposes, what matters most in the [specs for the Pixelbook](https://support.google.com/pixelbook/answer/7504948?hl=en) is not so much about the i5-7Y57 or i7-7Y75 processor. The biggest difference for our purposes is the internal SSD. The 128GB and 256GB models have an EMMC SSD, and it is currently not working within OS X. The i7 with 512GB has an NVMe SSD, which is fully supported. So unless you have the 512GB hard drive, installing OS X to the internal drive is currently not possible. It does work on an external SSD, however. Other notable specs:
-  Intel wifi chip: 7265
-  keyboard - PS/2
-  touchpad: HID-over-I²C
-  Suspend: ACPI S3 sleep
-  Audio: Audio Codec: Intel Sunrise Point-LP HD Audio. The speakers, mic and headphone jack are all connected to various codecs exposed via I²C. [yusefnapora explains in detail](https://github.com/yusefnapora/pixelbook-linux/blob/master/README.md#switching-audio-outputs--inputs) how complicated audio was in Ubuntu... But who knows, maybe we can figure it out.

## Current Status

Here's what's working at the moment:

| Feature            | Status               | Notes                                                             |
|--------------------|----------------------|-------------------------------------------------------------------|
| WiFi               | Working              | Working                                                           |
| Bluetooth          | Working              | Working                                                           |
| Suspend            | Not working          | Haven't started                                                   |
| Touchpad           | Not Working          |                                                                   |
| Graphics Accel.    | Working!             | On Mojave only, not Catalina or Big Sur.                          |
| Sound              | Not Working          | Partially working with bluetooth                                  |
| Keyboard backlight | Working (partially)  | 50% always on from latest MrChromebox firmware                    |
| Touchscreen        | Working! :-)         | With VoodooI2C.kext and VoodooI2CHID.kext                         |
| Mac OS 11 Big Sur  | Working (partially)  | Possible to upgrade from Catalina to Big Sur. Same functionality. |


### Requirements

Before you start, you'll need to have the following things to complete the process:

- A [SuzyQable CCD Debugging cable][suzyqable], ~$15 USD + shipping
- A USB-A to USB-C adapter
- 1 USB flash drives with USB-C connectors or adapters, preferably ~10GB or larger
- A willingness to accept that this is a potentially destructive process that may render your
  expensive Pixelbook inoperable or otherwise busted. See the [disclaimer](#disclaimer) below.

### Mandatory Disclaimer

The process described in this document could cause irreversible damage to your expensive laptop, and
you should prepare yourself mentally and emotionally for that outcome before you begin. I accept absolutely no responsibility for the consequences of anyone choosing to follow or ignore any of the instructions in this document, and make no guarantees about the quality or effectiveness of the
software in this repo.

## Installation

Here are the steps to go from stock Pixelbook to a Mac OS 10.14.6 Mojave install:

1. Flash UEFI firmware. Read and follow [yusefnapora's excellent guide](https://github.com/yusefnapora/pixelbook-linux) on how to flash the UEFI firmware using MrChromebox's scripts. To do this, you will need to disable write protect with either the SuzyQable cable or by removing the battery. 
2. Download and set up your Mac OS X Catalina and Mojave USB drives. 
3. Set up OpenCore on the EFI partition of the drive. [Read the OpenCore Install Guide.](https://dortania.github.io/) 
    - My EFI folder to use based on work completed so far (YMMV) is [here](https://www.dropbox.com/s/fbxfkh9t6ac8pav/EFI.zip?dl=0).
4. KEXTS: the following are suggestions, you can experiment with your own and report back!
    - For USB, follow Corpnewt's [USBMap](https://github.com/corpnewt/USBMap) procedure - this is instead of USBInjectAll, which is outdated.
    - AppleALC
    - IntelBluetoothFirmware
    - IntelBluetoothInjector
    - itlwm or AirportItlwm (use latest stable).
    - Lilu
    - SMCProcessor
    - VirtualSMC
    - VoodooI2C - touchscreen support
    - VoodooI2CHID
    - VoodooPS2Controller
    - WhateverGreen

5. Edit your config.plist with the following customizations:
    - SetupVirtualMap = No , rather than YES, as per OpenCore guide
    - Under DeviceProperties: AAPL,ig-platform-id is 00001E59, device-id is 591E0000. See full list of requirements in EFI or in the included config.plist. 
     - Under Kernel -> Quirks: AppleCpuPmCfgLock: True and AppleXcpmCfgLock: True
    
6. Test your config.plist for errors: https://opencore.slowgeek.com/

7. Setting up MacOS. This is currently a bit challenging for the non-512GB NVMe users. 
    -  First, make a Catalina and Mojave installer. 
    -  Install Mojave first, but it will fail with 2 minutes remaining. Note: if you are using an external drive, you must format it as APFS or the installer won't be able to install.
    - Next, copy the files from the MacOS Install Data folder to your computer. 
    - Now run the Catalina installer. It will also fail at 2 minutes remaining, but it will leave behind an important next step in the opencore bootloader.
    - Take the Catalina install target drive and copy the InstallESD.dmg file from the Mojave install over to the macOS Install Data folder, removing the one the Catalina installer made.
    - Plug the install target back into the Pixelbook. At boot, select the install media again, but this time you will see "Mac OS Install" as a menu item. 
    - The installation will continue for about 15 minutes. 
    - At the end, it will fail with an error. Reboot.
    - From opencore, select Recovery and let it boot. This may take 1-2 tries to fix the OS.
    - Once Recovery has done it's job, you'll be able to boot Mojave.

8. After install: 
    - If using itwlm, use [heliport](https://openintelwireless.github.io/HeliPort) to connect to Wifi.
    - Still need to identify sound, touchpad, and power management solutions.
    - Test and report back!





