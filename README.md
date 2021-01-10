# Installing Mac OS Mojave on the Google Pixelbook 
We've managed to install GNU/Linux, Windows and a lot of other hacks on this beautiful laptop... so why not Mac OS? 

## This is a work-in-progress guide.
Not all of the functionality is working. This will be updated frequently as fixes are identified. At the moment this guide is focused on Mac OS 10.14.6 Mojave as it has the best support for our hardware.

## A note on Pixelbook hardware
For OS X purposes, what matters most in the [specs for the Pixelbook](https://support.google.com/pixelbook/answer/7504948?hl=en) is not so much about the i5-7Y57 or i7-7Y75 processor. The biggest difference for our purposes is the internal SSD. The 128GB and 256GB models have an EMMC SSD, and it is currently not working within OS X. The i7 with 512GB has an NVMe SSD, which is fully supported. So unless you have the 512GB hard drive, installing OS X to the internal drive is currently not possible. It does work on an external SSD, however. Other notable specs:
-  Intel wifi chip: 7265
-  keyboard - PS/2
-  touchpad: HID-over-I²C
-  Suspend: ACPI S3 sleep
-  Audio: Audio Codec: uses unsupported codecs MAX98927 as speakers and RT5514 as dmic on ssp0 and RT5663 as headset on ssp1... Not currently working, but also not impossible. As a cheap and easy workaround, a [$10 USB sound adapter](https://www.amazon.com/Syba-external-Adapter-Windows-C-Media/dp/B001MSS6CS) or bluetooth audio work perfectly well. 

## Current Status

Here's what's working at the moment:

| Feature            | Status               | Notes                                                             |
|--------------------|----------------------|-------------------------------------------------------------------|
| WiFi               | Working              | Working                                                           |
| Bluetooth          | Working              | Working                                                           |
| Suspend            | Not working          | Haven't started                                                   |
| Touchpad           | Working (partially)  | Install [Karabiner 12.10.0](https://github.com/pqrs-org/Karabiner-Elements/releases/download/v12.10.0/Karabiner-Elements-12.10.0.dmg) and go to Devices to enable ID 6353     |
| Graphics Accel.    | Working!             | On Mojave only, not Catalina or Big Sur.                          |
| Sound              | Not Working          | Partially working with bluetooth / USB sound adapter              |
| Keyboard backlight | Working (partially)  | 50% always on from latest MrChromebox firmware                    |
| Touchscreen        | Working! :-)         | With VoodooI2C.kext and VoodooI2CHID.kext                         |
| Mac OS 11 Big Sur  | Boots                | Same as Catalina, has issues with the iGPU. Not recommended atm.  |


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

7. Setting up MacOS. You'll need a working Mac for these steps, for now. 
    -  First, download both Catalina and Mojave installers from Apple - [gibMacOS](https://github.com/corpnewt/gibMacOS) is a great tool for that. 
    -  Run the BuildmacOSInstallApp tool within gibMacOS to create both of the installers.
    -  Right-click on Catalina and Mojave installers, then go to Show Package Contents, Contents, SharedSupport.
    -  Copy the InstallESD.dmg file from the Mojave install over to the Catalina installer in the same location, replacing it.
    -  Create the new Catalina installer (with Mojave's InstallESD.dmg inside) with `sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume` replacing "MyVolume" with the name of your install media USB.
    - When that has finished, you'll need to mount the EFI partition with the [MountEFI](https://github.com/corpnewt/MountEFI) utility and copy the contents of the latest EFI linked above into this partition.
    - Boot the Catalina installer. Make sure to use DiskUtility during the initial install steps to format your target drive as APFS. The whole volume.
    - The initial install will fail with 2 minutes remaining. Force power down and restart the Pixelbook.
    - At boot, select the install media again, but this time you will see "Mac OS Install" as a menu item. 
    - The installation will continue for about 15 minutes. 
    - At the end, it may fail with an error. Power down the Pixelbook.
    - Copy the EFI to your new Mojave drive using the same procedure from above 
    - Boot again, this time in the opencore menu, select Recovery. Let it boot and go to Disk Utility. Run First Aid on the Mojave volume. Reboot.
    - Reboot into Mojave and set it up! 

8. After install: 
    - If using itwlm, use [heliport](https://openintelwireless.github.io/HeliPort) to connect to Wifi.
    - Still need to identify sound, touchpad, and power management solutions.
    - Test and report back!





