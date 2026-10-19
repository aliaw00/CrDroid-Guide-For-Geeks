# CrDroid-Guide-For-Geeks

# How to Install a Custom Android ROM: A Complete Step-by-Step Guide

**Last checked:** October 19, 2026  
**Example device:** Xiaomi POCO X3 Pro  
**Example ROM:** crDroid 13 (Android 17)  
**Example device codename:** `vayu`  
**Example installation method:** crDroid Recovery + ADB sideload  
**Computer used in this guide:** Debian/Linux; Windows is needed for Xiaomi's official Mi Unlock tool in the POCO example.

> **Important:** This article uses the POCO X3 Pro and crDroid as a worked example, but also explains how to adapt the process to other phones and ROMs. Do not copy device-specific flashing commands, recovery images, firmware files, or partition instructions to another device. Always follow the official installation guide for your exact phone model, device codename, and ROM version.

## 1. Introduction

The POCO X3 Pro is an Android smartphone, but its official Android software is old. I wanted to try a newer Android version and give this old phone a second life. One way to do that is to replace the original operating system with a community-developed Android ROM, such as crDroid.

crDroid is a custom Android ROM based on the Android Open Source Project (AOSP) and the LineageOS project. It provides additional customization options, system controls, and a different Android experience from the manufacturer's original software. In simple terms, crDroid builds on work from LineageOS, which is itself based on AOSP—a little like how one Linux distribution can build on another, although the projects are not exactly equivalent to Debian and Ubuntu.

As of **October 19, 2026**, the [official crDroid device page](https://crdroid.net/vayu/13/) lists crDroid 13, based on Android 17, for the POCO X3 Pro (`vayu`). The listed build is dated October 8, 2026. These details can change, so check the official page before downloading anything.

### What you will accomplish

By following this guide, you will:

1. Prepare your phone and computer.
2. Back up your data and unlock the bootloader, if needed.
3. Install the recovery or other support images required by your device and ROM.
4. Format data if the ROM's clean-install instructions require it.
5. Install the ROM using the method supported by the device-specific guide.
6. Optionally install Google Apps, if compatible and desired.
7. Complete the first boot and verify that the system works.

**Important:** Installing a custom ROM involves risk. Unlocking the bootloader and formatting data usually erase personal information. Incorrect firmware, recovery images, or flashing commands can prevent a phone from booting. Read each step before acting, and stop if a command or installation step fails. It can be manageable if you prepare carefully, but do not rush it.

## 2. Before You Begin: Understand the Important Terms

### Bootloader

The bootloader is the software responsible for starting the operating system. Many phones ship with a locked bootloader.

An **unlocked bootloader** permits compatible custom software to be installed. Unlocking it is normally required for this type of installation, unless the device-specific guide says otherwise or the bootloader is already unlocked.

Unlocking commonly erases the phone's data and reduces some of the device's security protections. The method and eligibility requirements vary by manufacturer and model.

### Recovery

Recovery is a separate environment used for maintenance tasks such as installing system updates, formatting data, and resetting the phone.

For the POCO X3 Pro example, we will use **crDroid Recovery**, linked from the official device page. Other phones and ROMs may require a different recovery—or may use recovery images stored in partitions such as `boot` or `vendor_boot` rather than a separate `recovery` partition.

Do not automatically substitute TWRP or OrangeFox. A recovery suitable for another device or an older ROM is not necessarily compatible with the current release. Use a third-party recovery only if the ROM or device maintainer explicitly supports it.

### ADB and Fastboot

These are command-line tools for communicating with Android devices.

- **ADB (Android Debug Bridge):** Communicates with Android and can transfer installation packages to recovery.
- **Fastboot:** Communicates with the bootloader and can flash supported images to specified partitions.

You will use both tools from your Debian terminal in this example. On another computer, follow the installation instructions for that operating system.

### ROM and Google Apps

The **ROM** is the custom Android operating system.

Google Apps, often called **GApps**, is an optional package that provides Google services and applications. You do not need GApps if you prefer a Google-free system. If you want Google Play Store and related services, use only a package compatible with your specific ROM release, Android version, and device architecture. Some ROMs have different requirements or provide Google Apps separately from the ROM.

### Firmware

Firmware includes device-specific low-level components needed for the hardware to function correctly. It is distinct from the Android ROM.

The crDroid installation page for the POCO X3 Pro requires the latest available firmware for that device. Firmware requirements vary by phone, region, ROM, and the software currently installed. Do not flash an arbitrary package just because its date looks newer; use the firmware and source specified in the official device guide.

## 3. Requirements and Preparation

Prepare everything before modifying the phone.

### Hardware requirements

You need:

- A phone in stable working condition (this guide uses the POCO X3 Pro as an example).
- A reliable USB data cable, not a charge-only cable.
- A computer with a USB port.
- A working internet connection.
- Enough computer storage for the ROM, recovery/support images, and optional Google Apps.
- A sufficiently charged phone. Aim for at least 60–70% battery.

Avoid this procedure if the phone randomly shuts down, overheats severely, reboots unexpectedly, or repeatedly disconnects from USB. A software installation will not fix a hardware fault.

### Software and files

Download files only from the ROM project's official site or from links provided by the official device maintainer. The exact files needed differ by device and ROM.

#### 1. ROM package

For the POCO X3 Pro example:

- Official device page: <https://crdroid.net/vayu/13/>
- ROM directory: <https://sourceforge.net/projects/crdroid/files/vayu/13.x/>

The listed build at the time this article was checked was approximately 1.8 GB. If the official page lists a newer build when you download the ROM, follow that page and use the matching current instructions and checksum.

#### 2. Recovery or other support images

For the POCO X3 Pro example, the official recovery directory is:

<https://sourceforge.net/projects/crdroid/files/vayu/13.x/recovery/>

Use the image linked from the current crDroid device page. Depending on the phone, the files supplied by a ROM maintainer may include `recovery.img`, `boot.img`, `vendor_boot.img`, `init_boot.img`, `dtbo.img`, or other support files. **Do not assume every device needs all these files—or that the same command works for every file.**

> Download `TWRP` or `OrangeFox` just for backup and having another recovery.

#### 3. Google Apps (optional)

For the POCO X3 Pro example, start from the GApps link on the official crDroid device page or the [crDroid GApps directory](https://sourceforge.net/projects/crdroid/files/gapps/).

Choose a package explicitly compatible with the ROM version, Android version, and device architecture. For the example in this article, that means crDroid 13 / Android 17 / ARM64. Package names and availability can change, so do not rely on an old filename from a tutorial. If the ROM maintainer recommends a particular GApps package, follow that recommendation.

#### 4. ADB and Fastboot

On Debian, install the tools with:

```bash
sudo apt update
sudo apt install adb fastboot android-sdk-platform-tools-common
```

Confirm that they are available:

```bash
adb version
fastboot --version
```

For other operating systems, use the official Android platform-tools instructions or the package recommended by your distribution.

### Verify the download

Open a terminal in the directory containing the downloaded ROM and calculate its SHA-256 checksum. Replace the example filename with the **exact name** of the file you downloaded:

```bash
sha256sum "exact-ROM-filename.zip"
```

Compare the result with the checksum on the official download page. Do the same for other images if their official checksums are provided. Do not continue if a checksum does not match; redownload the file from the official source and check again.

Keep the files together in a directory such as:

```bash
mkdir -p ~/crdroid
cd ~/crdroid
```

Save or move the downloaded files into this directory. If you choose a different directory, adjust the commands below accordingly.

## 4. Identify Your Device and Protect Your Data

### Confirm the device codename

Every ROM build supports specific devices and often uses a device codename. Confirm both the exact commercial model and the codename before downloading or flashing anything.

For this example, the official crDroid page lists the POCO X3 Pro as `vayu`. The related hardware family also uses the `bhima` designation in some contexts. Do not assume a ROM build supports both identifiers merely because another project does; if your device reports `bhima`, confirm compatibility with the crDroid maintainer before flashing.

If Android still boots and USB debugging is enabled, you can inspect the device property with:

```bash
adb shell getprop ro.product.device
```

You can also check the model information in Settings > About phone. Menu names and reported properties can vary, so use the official device guide to resolve any uncertainty.

Do not confuse the POCO X3 Pro with the POCO X3 NFC, whose codename is `surya`. Their installation files are not interchangeable. For other phones, look up the exact model and codename in the ROM project's supported-device list.

### Back up your data

**Unlocking the bootloader erases personal data on many devices.** A clean ROM installation may also format internal storage.

Back up anything you cannot afford to lose:

- Photos and videos.
- Downloads and documents.
- WhatsApp and other messaging backups.
- Contacts, notes, and calendar information.
- Authenticator recovery codes and two-factor authentication methods.
- Password-manager recovery information.
- Any application data that is not automatically synchronized to your account.

Copy important files to your computer or another storage device and verify that they open correctly. Do not keep your only backup on the phone you are about to format.

### Remove accounts before a clean installation

The official crDroid guide recommends removing Google accounts from the POCO X3 Pro before a fresh installation. This helps prevent Factory Reset Protection (FRP) from blocking setup after the reset.

For other devices, follow the ROM and manufacturer instructions. In every case, make sure you know the credentials for any accounts linked to the phone.

### Check the phone's hardware condition

The POCO X3 Pro has documented power-on and motherboard failure concerns in some units. That does not mean every unit will fail, or that flashing a ROM causes these faults. It does mean you should avoid starting an invasive software procedure on a phone that is already unstable.

If the phone repeatedly reboots, occasionally goes blank, overheats, or has intermittent USB connectivity, investigate the hardware condition first. For any other phone, consider known device-specific hardware issues before proceeding.

POCO India reference: <https://www.poco.in/extended-warranty>

## 5. Unlock the Bootloader

> **Skip this section only if your bootloader is already unlocked.** The method below is specific to Xiaomi's POCO X3 Pro. If you have another phone, follow the bootloader-unlocking instructions for that manufacturer and model. Do not assume that the same tool or Fastboot command works on every device.

### POCO X3 Pro: Xiaomi Mi Unlock

The official Xiaomi Mi Unlock application runs on Windows. You therefore need access to a Windows PC for this step, even if you perform the rest of the installation on Debian/Linux.

**My suggestion:** If you only have a Debian computer, you can try running Windows in VirtualBox, but USB passthrough introduces extra setup and potential connection problems. A physical Windows PC is usually the simpler, more reliable option. Do not proceed until the unlock tool can detect the phone consistently.

If you choose VirtualBox:

1. [Download and install VirtualBox](https://www.virtualbox.org/wiki/Downloads).
2. Check the VirtualBox documentation to determine whether you need the Extension Pack for USB 2.0/3.0 passthrough with your version and configuration.
3. Download the Windows ISO from [Microsoft's official Windows download page](https://www.microsoft.com/software-download/windows11).
4. Create a Windows virtual machine and complete Windows setup.
5. Configure USB passthrough so the guest Windows system can access the phone. The exact steps depend on your VirtualBox version and host setup.
6. Install and run the official Mi Unlock tool inside Windows.

If USB passthrough does not work reliably, use a physical Windows PC instead of experimenting while the phone is in a sensitive flashing state.

### Step 1: Enable Developer Options

On the POCO X3 Pro:

1. Open Settings.
2. Select About phone.
3. Tap the MIUI version entry repeatedly until Developer options are enabled.
4. Return to Settings and open Additional settings > Developer options.

Menu wording may vary by MIUI version. On another phone, follow that manufacturer's instructions for enabling Developer options and OEM unlocking, if applicable.

### Step 2: Link your Xiaomi account

For the POCO X3 Pro:

1. Insert a working SIM card.
2. Sign in to your Xiaomi account.
3. Connect the phone to the internet, preferably using mobile data if the unlocking process requires it.
4. Open Developer options > Mi Unlock status.
5. Use the option to add or bind the account and device.

Follow the prompts shown on the phone. Do not unlink the account while waiting for permission. These account-binding steps are Xiaomi-specific; other manufacturers use different procedures.

### Step 3: Use the official Mi Unlock tool

1. Visit Xiaomi's [official bootloader-unlocking website](https://en.miui.com/unlock/).
2. Download the official Mi Unlock utility for Windows.
3. Sign in with the same Xiaomi account linked to the phone.
4. Turn off the phone.
5. Hold Volume Down + Power until the FASTBOOT screen appears.
6. Connect the phone to the Windows computer.
7. Follow the utility's instructions.

Xiaomi may impose a waiting period before unlocking is permitted. The exact requirement depends on the account, device, and current Xiaomi policy. Follow the waiting period displayed by the official tool rather than relying on a fixed number of days quoted by an old tutorial.

Avoid unofficial unlock services and modified unlocking tools.

For other manufacturers, use their official unlock process. Some devices use a documented Fastboot command, while others require a manufacturer utility or may not permit bootloader unlocking at all. **Do not run a generic unlock command unless the official instructions for your exact device tell you to.**

### Step 4: Confirm unlocking

After the tool reports success, the phone will usually reset.

Complete any required basic setup and re-enable USB debugging if you want to return to using ADB. Check the unlock status in Developer options or by following the device's official instructions. If the tool reports an error or the bootloader remains locked, stop here and resolve it before flashing recovery or other images.

**Security note:** An unlocked bootloader weakens some protections against unauthorized software. Use a strong device PIN, keep sensitive backups protected, and understand the consequences before continuing.

## 6. Prepare Debian and Establish an ADB Connection

This section uses Debian/Linux. On Windows or macOS, use the equivalent terminal or command prompt and install the appropriate platform tools.

### Step 1: Enable USB debugging

On the phone, open:

`Settings > Additional settings > Developer options > USB debugging`

Enable it, connect the phone to your computer, and approve the computer's debugging authorization prompt.

### Step 2: Verify the ADB connection

Run:

```bash
adb devices
```

An authorized device should appear in the output.

If the status is `unauthorized`, unlock the phone and accept the authorization prompt. If the device does not appear, troubleshoot the cable, USB port, permissions, or USB connection before proceeding.

### Step 3: Prepare your files

Enter your download directory:

```bash
cd ~/crdroid
```

Confirm that the required files exist:

```bash
ls -lh
```

For this example, you should have the ROM ZIP, `recovery.img`, and—if you intend to install Google Apps—the compatible GApps ZIP. Rename a file only if the official instructions permit it and you understand which file it is. Otherwise, keep the downloaded filename and use that exact filename in commands.

## 7. Install the Required Recovery or Support Images

This is a critical, device-specific step. Use only the image files and flashing commands linked or specified by the official guide for your exact device and ROM release.

### Step 1: Enter Fastboot mode

If Android is running and ADB is authorized, run:

```bash
adb reboot bootloader
```

Alternatively, power off the POCO X3 Pro and hold Volume Down + Power until FASTBOOT appears. Button combinations differ by phone.

### Step 2: Confirm that Fastboot detects the phone

Run:

```bash
fastboot devices
```

A device identifier should appear. If no device appears, stop. Do not attempt to flash anything until the connection is working reliably.

### Step 3: Flash the recovery image

**For the POCO X3 Pro + crDroid 13 example only**, the official instructions use:

```bash
fastboot flash recovery recovery.img
```

Wait for the command to finish and check its output. It should report that the operation completed successfully.

**Do not copy this command to another phone unless that phone's official guide explicitly uses it.** Some modern devices do not have a separate recovery partition. Their instructions may require flashing a recovery image to another partition, flashing multiple support images, or booting an image temporarily. Partition names and commands are not interchangeable. Do not substitute `boot.img`, `vendor_boot.img`, `init_boot.img`, `dtbo.img`, or any other image unless the device-specific instructions tell you to.

### Step 4: Boot directly into recovery

For the POCO X3 Pro example, the official guide warns against booting the existing operating system immediately after flashing recovery, because the recovery may be overwritten.

With the phone powered off, hold Volume Up + Power until the Xiaomi/POCO splash appears, then release Power while continuing to hold Volume Up until recovery loads. Button behavior can vary; follow the official steps for your phone.

For this example, recovery should display the crDroid branding. If a different recovery appears, stop and investigate why before installing the ROM.

## 8. Perform a Clean Installation of the ROM

**Warning: Formatting data erases internal storage.** Confirm that your backup is complete and that all required files are available on your computer.

The POCO X3 Pro crDroid 13 guide specifically requires formatting data once with crDroid Recovery because of its configured secure FUSE passthrough. **This requirement is not universal.** For another phone or ROM, follow that project's clean-install instructions exactly.

### Step 1: Format data, if required

For the POCO X3 Pro example, in crDroid Recovery:

1. Open Factory Reset.
2. Choose Format data.
3. Confirm the operation as requested by recovery.
4. Wait until formatting finishes.
5. Return to the main recovery menu.

If touch input does not work in recovery, check the recovery documentation; some recoveries can be navigated with the volume and power buttons.

Formatting data is not merely a cache wipe. It removes the contents of internal storage and clears the existing data-encryption state. If recovery reports a formatting error, do not ignore it. Investigate the problem before continuing.

### Step 2: Start ADB sideload mode

For the POCO X3 Pro crDroid Recovery:

1. Select Apply Update.
2. Select Apply from ADB.

The recovery will wait for the computer to send an installation package. The names and location of these menu items can differ in other recoveries.

### Step 3: Sideload the ROM

On Debian, from the directory containing the ZIP, run the command below after replacing the sample filename with the exact name of your downloaded ROM:

```bash
adb sideload "exact-ROM-filename.zip"
```

For the POCO X3 Pro example, use the latest compatible crDroid ZIP downloaded from the official `vayu` page. Do not use a package intended for another device.

Wait for the operation to finish. Watch the messages on the recovery screen, not only the percentage shown in the terminal; ADB sideload progress can sometimes behave differently from a normal file copy.

If recovery reports a failed installation, an incompatible package, or a verification error, do not reboot blindly. Read the error and verify that the file, device, firmware, recovery, and installation instructions match.

### Step 4: Install Google Apps, if desired

If you want Google Play Store and Google services, install a compatible GApps package before the first boot **if the ROM's guide supports or requires this process**. Some ROMs handle this differently, so follow the official instructions.

For the POCO X3 Pro example, after the ROM installation, return to or restart recovery as prompted, then select Apply Update > Apply from ADB again.

Run the following command after replacing the placeholder with the exact GApps filename:

```bash
adb sideload "exact-compatible-GApps-filename.zip"
```

Do not reboot into Android between installing the ROM and GApps if the official guide says to install both in the same session. If you prefer a Google-free system, skip this step.

### Step 5: Reboot into the new system

Once all required packages have installed successfully:

1. Return to the main recovery menu.
2. Select Reboot system.
3. Wait for Android to start.

The first boot can take several minutes. Do not interrupt it simply because the boot animation remains visible longer than usual.

If the device remains stuck for an unusually long time, consult the troubleshooting section below before performing further wipes or flashing unrelated files.

## 9. First Boot and Initial Setup

After the ROM starts successfully, complete the Android setup process.

### Configure the basics

Select your language, connect to Wi-Fi or mobile data, and configure your screen lock.

If you installed Google Apps, sign in to your Google account and allow the necessary components to initialize. If you did not install GApps, configure the system according to your preferred privacy and application setup.

### Test essential hardware

Before restoring all your data, verify the following where applicable:

- SIM card detection and mobile calls.
- Mobile data and Wi-Fi.
- Bluetooth.
- Front and rear cameras.
- Microphone and speakers.
- Fingerprint reader or other biometric features.
- Charging and battery reporting.
- Screen brightness, touch input, and refresh-rate behavior.
- GPS and location services.

The results can depend on the exact device variant, firmware, and ROM release. Do not assume that every feature is functioning correctly simply because Android has booted.

### Restore your data gradually

Restore files and application data from your backups. Confirm that important accounts and messaging services work.

Avoid immediately installing Magisk, custom kernels, modules, or system modifications. First establish that the clean ROM works normally; otherwise, troubleshooting becomes much harder.

## 10. Troubleshooting Common Problems

### Problem: `adb devices` shows no device

Try another USB data cable or port, unlock the screen, check USB debugging, and confirm that the debugging authorization prompt was accepted.

On Debian, the relevant Android USB permissions and device rules may also need to be installed.

### Problem: `fastboot devices` shows nothing

Check that the phone is really in Fastboot mode. Try a different USB port or cable. If you are using a virtual machine, make sure the USB device is attached to the guest operating system.

Do not continue with flashing until Fastboot reliably detects the phone.

### Problem: Recovery does not look like the expected recovery

You may have flashed the wrong image, used an incompatible file, or had the recovery overwritten.

Return to the bootloader only if it is safe and possible to do so. Verify the correct image and repeat the recovery procedure using the current official instructions. Do not try random images or partition commands.

### Problem: ROM installation fails

Check the following:

1. The ZIP is the official package for the supported device.
2. The download completed successfully and the checksum matches.
3. The recovery is the one specified for the current ROM release.
4. The required firmware is installed.
5. The error message does not indicate an incompatible package or failed verification.

Do not try random firmware files or partition images to force the installation to proceed.

### Problem: The phone stays on the boot animation

Give the first boot sufficient time to complete.

If the device remains stuck, re-enter the correct recovery if possible and review the installation messages. Confirm the device codename, firmware requirements, ROM version, and whether all clean-install steps were completed.

Do not repeatedly wipe the phone or flash unrelated images without identifying the cause.

### Problem: Wi-Fi, camera, or sensors do not work

Confirm that the problem exists on a clean installation and that the correct firmware is installed. Custom kernels and modifications can introduce compatibility problems, so remove variables before reporting a bug.

Follow the ROM's official device-support instructions to report reproducible issues.

### Problem: The phone becomes unresponsive or repeatedly powers off

A software installation cannot repair a physical motherboard or power fault. If the phone develops intermittent power-on symptoms, investigate the hardware rather than continuing to flash software.

## 11. Important Advice After Installation

### Keep the bootloader unlocked

Do not relock the bootloader while a custom ROM is installed unless the ROM and device maintainers explicitly document a supported procedure.

Relocking with incompatible firmware can prevent the phone from booting and may make recovery more difficult.

### Update carefully

Use the built-in system updater when a compatible OTA update is offered, or follow the official recovery-update procedure.

For a normal update to a newer build of the same crDroid version, the POCO X3 Pro guide allows installing the newer ROM without formatting data again. A major Android-version upgrade or a change to a different ROM can have different requirements, so read the instructions for that specific transition.

Back up important data before updates. Do not assume that GApps must—or must not—be reflashed after every update; follow the update notes for your exact ROM and GApps package.

### Understand the security trade-offs

An unlocked bootloader and a custom ROM change your device's security configuration. Some banking applications, corporate applications, DRM-protected streaming features, and device-integrity checks may behave differently.

Google Play certification or integrity checks are not guaranteed. Do not assume that root or integrity workarounds will be available, stable, or safe.

### Avoid unnecessary modifications

Start with the official ROM, recovery, and firmware requirements. Test battery life, performance, camera operation, and connectivity before adding modules or kernels.

This provides a reliable baseline and makes future updates and troubleshooting easier.

## 12. How to Return to the Original Operating System

If you decide that the custom ROM is not suitable, you can investigate restoring the manufacturer's original operating system for your exact device and region. For the POCO X3 Pro, this means the appropriate official MIUI/stock firmware package.

Use a trusted official stock ROM and the appropriate restoration procedure. Read the instructions carefully before starting because restoring stock firmware can also erase data.

**Never choose an option that erases the phone and relocks the bootloader unless you have verified that the complete official firmware is correct for your precise model and region and that relocking is appropriate.**

If the phone is not detected in Fastboot or has a hardware failure, the usual software-restoration procedure may not work.

## 13. Final Pre-Installation Checklist

Before starting, make sure you have checked every item that applies:

- [ ] The exact phone model and device codename are confirmed.
- [ ] The ROM is an official or trusted, explicitly compatible build for that device.
- [ ] The required recovery and support images were downloaded from the matching device page.
- [ ] The required firmware matches the device, region, and ROM instructions.
- [ ] The bootloader is unlocked.
- [ ] All important personal data has been backed up elsewhere.
- [ ] Accounts have been removed if the device-specific clean-install guide recommends it.
- [ ] ADB and Fastboot work on the computer.
- [ ] The computer recognizes the phone in both ADB and Fastboot mode, if both are needed.
- [ ] The ROM checksum has been verified when one is provided.
- [ ] A compatible GApps package is available, if needed.
- [ ] You understand whether formatting data is required and that it erases internal storage.
- [ ] You have read the full device-specific installation instructions.
- [ ] You will stop if an installation command or recovery step fails.

## 14. Official Resources

### POCO X3 Pro + crDroid example

- **crDroid 13 device page:** <https://crdroid.net/vayu/13/>
- **Official device-specific installation guide:** <https://crdroid.net/vayu/13/install>
- **Official recovery files:** <https://sourceforge.net/projects/crdroid/files/vayu/13.x/recovery/>
- **Official ROM files:** <https://sourceforge.net/projects/crdroid/files/vayu/13.x/>
- **crDroid GApps directory:** <https://sourceforge.net/projects/crdroid/files/gapps/>
- **Xiaomi bootloader unlocking:** <https://en.miui.com/unlock/>
- **Xiaomi bootloader FAQ:** <https://www.mi.com/uk/support/faq/details/KA-07238/>
- **Debian Android tools:** <https://wiki.debian.org/AndroidTools>

### For another phone or ROM

Start from the ROM project's official download page and device-specific installation guide. Check the exact model and codename, the required stock firmware version, supported recovery, partition-flashing commands, clean-install steps, optional Google Apps, and update instructions. If the project does not explicitly support your device, do not attempt to install a build intended for a different phone.

## Conclusion

The safest approach is to follow the official device-specific sequence: unlock the bootloader, install the required recovery or support images, format data if the clean-install instructions require it, install the ROM using the supported method, optionally install compatible Google Apps, and then reboot.

The most important rules are to verify your exact device and downloaded files, meet the firmware requirements, keep a separate backup, and never continue after an unexplained failure. A successful installation is not just about getting Android to boot; it also means confirming that the phone's essential hardware works correctly and that you can maintain and update the system safely.
