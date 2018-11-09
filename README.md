### [Guide] HP Envy Haswell series J/K/Q/N using Clover UEFI**


##### Overview

The purpose of this guide is to provide a step-by-step guide to installing Mojave, High Sierra, Sierra, El Capitan, or Yosemite on the HP Envy J/K/Q/N series Haswell laptops. I no longer have the laptop, but original work was done when I had an HP Envy 15-J063CL.

Since I no longer own the laptop, I have not been able to personally test the scripts. Much of the work is based on my work with the Lenovo u430 and much of it carries over (in fact, to any similar laptop), so it should work, but there can always be small details that need adjusting. I'd appreciate accurate and detailed feedback to make the guide better. At this point, it has been tested and verified on several laptops with success.

Computer Specifics
HP Envy 15-J063CL (Costco)
i7-4700MQ @2.4Ghz, 12GB RAM
HM87 chipset
HD4600 graphics (1080p panel)
BCM4352 ac WiFi
RTL8111/8168/8411

Some early background/development is here: http://www.tonymacx86.com/mavericks...-locked-msrs-hp-envy-15-j063cl-i7-4700mq.html


##### What you need

- Haswell HP Envy J-series (now confirmed to work for K-series as well, Q-series too, N-series beta)
- macOS or OS X downloaded from the Mac App Store
- 8GB USB stick
- (optional) 32GB USB stick for HP OEM backup
- Broadcom BCM94352(HMB) for native WiFi


##### BIOS settings

To start, set BIOS to Windows 8 defaults.

Then insure:
- UEFI boot is enabled
- secure boot is disabled
- enable Legacy Boot (but UEFI first) and you may experience less boot time glitches

If you own the laptop, please help with any additional required BIOS settings that may be required.

Note: The DSDT/SSDT patching script will automatically disable the discrete nVidia card if you have it enabled in BIOS. It is best, therefore to keep it enabled in BIOS so you can still use it on Windows, but have DSDT/SSDT patched properly for OS X.


##### Preparing USB and initial Installation

Prior to installing OS X, it is a good idea to create an OEM recovery USB from Windows. If anything goes wrong and you want/need to get back to Windows, you can restore it via the USB. Use the utility provided by HP to accomplish this.

You can also leave Windows intact, but it can get tricky. Read here for more information: http://www.tonymacx86.com/multi-booting/133940-mavericks-windows-8-same-drive-without-erasing.html

This guide for creating USB and installing using Clover UEFI works well for this laptop: http://www.tonymacx86.com/el-capita...de-booting-os-x-installer-laptops-clover.html

##### Special notes:

- Use the latest RehabMan Clover build

- Definitely copy RealtekRTL8111.kext to Clover/kexts/Other as having network support during post-install is helpful. The rest of this guide depends on it. An alternate is to use AirportBrcmFixup.kext. This will enable WiFi, provided you have the BCM94352HMB WiFi card already installed.

- GenericUSBXHCI.kext is not necessary with this laptop. Do not use it.

- Use the 'createinstallmedia' approach. It works well, and there is little chance for pilot error. This method also gives you an OS X recovery partition.


##### Post Installation

Install Clover UEFI as described in the guide linked by the previous section (post #2). After installing Clover, and configuring it correctly (config.plist, kexts, etc) you should be able to boot from the HDD/SSD.

But there are still many issues and devices that won't work correctly. For that, we need to patch DSDT, provide a proper config.plist, and install the kexts that are required.

Since you have RealtekRTL8111.kext already injected by Clover, you should have internet access simply by using an Ethernet cable to your router. Plug it in and make sure you have internet access before continuing. Or if you're using AirportBrcmFixup.kext, you can connect to your WiFi router before continuing.

Installation of the tools and patching is easy provided the scripts and tools at the HP Envy repository: https://github.com/RehabMan/HP-Envy-DSDT-Patch.

To start, the developer tools must be installed. Run Terminal, and type:

```
xcode-select --install
```
You will be prompted to install the developer tools. Since you have internet working, you can choose to have it download and install them automatically. Do that before continuing.

After the developer tools are installed, we need a copy of the appropriate project.

In Terminal:
```
mkdir ~/Projects
cd ~/Projects
```
To clone the github project:
```
git clone https://github.com/RehabMan/HP-Envy-DSDT-Patch envy.git
```
Note: Previous versions of this guide used a separate github location for each of the separate models J/K/N/Q. For the current project, all have been merged into a single project.

Now it is time to install some more tools and all the kexts that are required...

In Terminal:
```
cd ~/Projects/envy.git
./download.sh
./install_downloads.sh
```
The download.sh script will automatically gather the latest version of all tools (patchmatic, iasl, MaciASL) and all the kexts (FakeSMC.kext, ACPIBatteryManager.kext, etc) from bitbucket. The install_downloads.sh will automatically install them to the proper locations.

With the current project, no patched DSDT/SSDTs are used. Instead I use Clover hotpatches and a small SSDT called SSDT-HACK-J, SSDT-HACK-K1 or SSDT-HACK-K2, etc, depending on your laptop model.

In Terminal:
```
cd ~/Projects/envy.git
make
make install_j
```
Similarly, there is a 'make install' script for each of the other models: install_k1, install_k2, install_n, install_q.

Note: Some Envy K models use a different xHCI controller (K2 = 8086:8c31 , instead of K1 = 8086:9xxx). Make sure you select the correct install kX variant depending on the the device-id of your XHC controller.

The 'make' causes the SSDT-HACK.aml files to be compiled (with iasl), the results placed in ./build.

Finally, 'make install_j' (or 'make install_k2', etc), mounts the EFI partition, and copies the built files where they can be loaded by Clover (to EFI/Clover/ACPI/patched).


##### Power Management

Everything required for CPU/IGPU power management is already installed with the steps above.
There is no longer any need to use the ssdtPRgen.sh script.

Also, be aware that hibernation (suspend to disk or S4 sleep) is not supported on hackintosh.

You should disable it:
```
sudo pmset -a hibernatemode 0
sudo rm /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage
```
Always check your hibernatemode after updates and disable it. System updates tend to re-enable it, although the trick above (making sleepimage a directory) tends to help.


##### Final config.plist

Up to now, you've been using the same config.plist we were using for installation. After all the APCI files are in place (previous two steps), you're ready to use the final config.plist from the Envy repo.

First, mount the EFI partition:
```Code:```
cd ~/Projects/envy.git
./mount_efi.sh
```
Then copy the file:
```
cd ~/Projects/envy.git
cp config_envyj.plist /Volumes/EFI/EFI/Clover/config.plist
```
There is a separate config.plist for each of the other models: config_envyk.plist, config_envyn.plist, config_envyq.plist.

You could also copy the file using Finder.

After copying the config.plist from the repo to EFI/Clover/config.plist, you should customize the SMBIOS so you have a unique serial. You can use Clover Configurator to do this (use google to find/download it). DO NOT use Clover Configurator to edit your actual config.plist. Instead edit a "dummy" config.plist to create the SMBIOS data and then use copy/paste with a plist editor (I use Xcode) to copy the SMBIOS section into my active config.plist. Clover Configurator is too buggy and cannot be trusted with edits to your real config.plist. This guide uses MacBookPro11,1. Do not use any other model identifier.


##### Do not stop reading

Although most of the post-install tasks are done, continue to read this guide. It it has important information you should know about.

In the case of a problem, don't bother asking about with without all files requested in "Problem Reporting".


##### Updates to the patch repositories

From time to time, updates may become available to the Envy or laptop patch repositories. In the event of such updates, you may want to update your copies, and re-patch DSDT/SSDT with the updates.

Since you're using git, it is easy...

In Terminal:
```
cd ~/Projects/envy.git
git pull
./download.sh
./install_downloads.sh
make
make install
```

##### What works

I have tested the following features:
- UEFI booting via Clover
- built-in keyboard (with special function keys)
- built-in trackpad (basic gestures)
- HDMI video/audio with hotplug (please verify, the patches came from the u430 repo and may need tweaking depending on the port used on the Envy)
- AirPlay mirroring to AppleTV
- native WiFi via BCM94352HMB
- Bluetooth (with handoff) using BCM94352HMB
- native USB3 with AppleUSBXHCI (USB2 works also)
- native audio with AppleHDA (using injector for easy updates), including headphone
- built-in mic
- built-in camera
- native power management
- battery status
- backlight controls with smooth transitions, save/restore across restart
- accelerated graphics for HD4400 including OpenCL
- wired Ethernet
- Mac App Store working
- screen works without flicker (contrary to HP ProBook)
- touchscreen (single touch only)

##### Not tested/not working

The following features have issues, or have not been tested:
- Messages/FaceTime (not tested, see guide: http://www.tonymacx86.com/general-help/110471-how-fix-imessage.html)
- some special Fkeys not working (??)
- card reader is not working


##### Known Problems

Find My Mac/Locking: Find My Mac does not work properly. Don't lock your mac because it's difficult (or impossible) to unlock again.

Slow WiFi after sleep/wake cycle: Disable "Wake for network access" in SysPrefs->Energy Saver.

Audio subwoofer: The subwoofer doesn't work, requires more work on AppleHDA. There were additional audio solutions created after I sold the laptop. More info here: http://www.insanelymac.com/forum/topic/290687-wip-hp-envy-17t-j000-quad-haswell-10851091010/

Audio (K-series): The internal mic does not work. It appears the ALC290 (provided by Mirone) does not quite match up with the K-series audio codec dump from Linux. A custom patch will be required. Those with the skills should look into it. Note: Maybe this is fixed with the transition to AppleALC?


##### Other post-install tasks

Trackpad: Be sure to visit the options in SysPrefs->Trackpad and change them to your liking.

Trackpad three finger support: You can configure three finger swipes in SysPrefs->Keyboard->Shortcuts. Instead of pressing keys for a given function, do the three finger swipe (up/down/left/right).

Disable trackpad when using an external mouse: The latest script installs the VoodooPS2Daemon. It allows you to disable the built-in trackpad when a USB mouse is plugged in. Just check the box in SysPrefs->Accessibility->"Mouse & Trackpad".

Bluetooth: If you get the Bluetooth Setup Assistant popup, go to SysPrefs->Bluetooth->Advanced, uncheck the boxes.


##### Keyboard Mapping

The mapping for Control, Option, and Command are according to the physical layout of the keys on an actual MacBook keyboard, not the labels on the keys. Control=Control, Windows=Option, and Alt=Command. If you want a more PC friendly keyboard layout, use Karabiner (formerly KeyRemap4MacBook).

Brightness up/down are implemented with DSDT patches and my VoodooPS2Controller. Mirror Toggle is also implemented. Since I don't have the laptop, I forget the physical keys used.

The function of Fn+F1..F12 and F1..F12 can be changed in SysPrefs->Keyboard.


##### System updates

First step should be to update to the latest repository.

To do so:
```
cd ~/Projects/envy.git
git stash
git pull
make
make install
./download.sh
./install_downloads.sh
```
Also update Clover to the latest using the Clover installer. Be sure to fix EFI/Clover/kexts, so that only EFI/Clover/kexts/Other is existing. All version specific directories under EFI/Clover/kexts should be removed.

Also update config.plist at EFI/Clover/config.plist to the latest content from the repo. Be sure to retain your own SMBIOS data at config.plist/SMBIOS.

Now you can update via the App Store. Just boot the installer/updater upon restart.

After updating, run ./install_downloads.sh again:
```
cd ~/Projects/envy.git
./install_downloads.sh
```

##### Updating to High Sierra

As you probably already know, High Sierra (and later) has a new file system called APFS. Boot drives on SSDs will automatically be converted to APFS if you start the installer in the default way (eg. running /Applications/Install macOS High Sierra.app).


##### Problem reporting

Problem reports should be accompanied by various files that allow your progress to be accounted for...

Read FAQ, "Problem Reporting". Carefully. Attach all requested files/output.
https://www.tonymacx86.com/threads/faq-read-first-laptop-frequent-questions.164990/
Use the gen_debug.sh tool mentioned in the FAQ, that way it is less likely you'll omit something.
