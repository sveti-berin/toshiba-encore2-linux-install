# Installing antiX Linux on Toshiba Encore 2 WT10-A32 (32-bit UEFI / 64-bit CPU)

This repository documents the successful installation of antiX Linux on a Toshiba Encore 2 WT10-A32 tablet. This device is notoriously difficult to install Linux on due to its hardware configuration: a 64-bit Intel Atom processor paired with a locked 32-bit UEFI firmware. 

Standard boot methods completely fail on this device. Below is the exact "EFI Hijack" workaround that finally allowed the system to boot and install antiX.

## The Problem
Standard installation media created via Rufus (using **GPT**, **UEFI (non CSM)**, and **FAT32** options) will not be recognized by the Toshiba BIOS. Attempts to use `rEFInd` as a standalone boot manager also failed to boot directly from the USB.

The ultimate solution involves manually copying the 32-bit EFI bootloader (`bootia32.efi`) into the internal Windows EFI partition and tricking the firmware into executing it by renaming it to the default Windows bootloader name (`bootmgfw.efi`).

## Prerequisites
* A USB flash drive.
* **Rufus** for creating the bootable media.
* The **antiX Linux** ISO file.
* A USB OTG cable and a USB keyboard.

---

## Step-by-Step Installation Guide

### Step 1: Prepare the USB Drive
1. Flash the antiX Linux ISO to your USB drive using Rufus. 
   * **Options to select:** `Partition scheme: GPT`, `Target system: UEFI (non CSM)`, `File system: FAT32`.
2. Extract `bootia32.efi` (from rEFInd or a generic 32-bit GRUB package) and place it inside the `EFI/BOOT/` directory on your USB drive.

### Step 2: The "EFI Hijack" (Command Prompt)
Insert the USB drive into the tablet. Boot into Windows and navigate to the Advanced Startup menu.
* Go to **Advanced Startup** -> **Troubleshoot** -> **Advanced Options** -> **Command Prompt**.

Once the Command Prompt opens, follow these exact commands to mount the internal EFI partition and back up the Windows bootloader:

```cmd
:: Mount the internal EFI partition as drive S:
mountvol S: /S

:: Navigate to the mounted partition to verify
S:
dir

:: Navigate to the Windows Boot directory
cd EFI\Microsoft\Boot
dir

:: Backup the original Windows bootloader
ren bootmgfw.efi bootmgfw_backup.efi
```
### Step 3: Copying the Linux Loader

Now, find your USB drive letter (usually C:, D:, or E: depending on the recovery environment) and copy the bootia32.efi file into the internal EFI partition, renaming it to masquerade as the Windows bootloader.
```cmd
:: Use diskpart to find your USB drive letter if you are unsure
diskpart
list volume
exit

:: Assuming your USB is drive E:, copy and rename the file
copy E:\EFI\BOOT\bootia32.efi S:\EFI\Microsoft\Boot\bootmgfw.efi

:: Verify the files are there
S:
cd EFI\Microsoft\Boot
dir
:: You should now see both bootmgfw_backup.efi and the new bootmgfw.efi
```
### Step 4: Restart and Boot GRUB

Restart the tablet:
```cmd
shutdown /r /t 0
```
The tablet firmware will attempt to boot "Windows", but will instead execute the 32-bit Linux loader, presenting you with the GRUB interface.

Note: In my experience, the initial GRUB loaded but failed to locate the necessary antiX installation files on the USB. To fix this, simply remove the USB, re-flash it with Rufus using the antiX ISO one more time, reinsert it, and restart. The system will automatically boot into the live antiX environment from the USB.

Step 5: Installation

Once booted into the live antiX desktop, run the Install antiX shortcut.

The installation process takes approximately 20 minutes.
This process completely wipes the internal Windows 8.1 installation, leaving a clean, lightweight Linux environment.

Post-Installation Performance & Notes

Overall, the performance is extremely satisfying, breathing new life into this limited hardware.

    RAM Usage: The system registers 888MB of available RAM (the rest is hardware-reserved for graphics). At idle, antiX uses an incredibly low 259MB, leaving plenty of headroom.

    Storage: While the tablet is advertised as a 32GB model, the internal storage registers as 14GB total available space. A fresh antiX installation occupies 6.66GB.

    Drivers & Updates: Running sudo apt update and sudo apt upgrade resolves most base issues. No major driver problems were encountered out of the box.

    Wi-Fi: The wireless connection works, though it can occasionally behave strangely (likely due to power management).

    Bluetooth: Still a work in progress. Requires manual configuration.

    Services: By design, antiX is extremely conservative with background processes. Many standard daemons and services do not start automatically at boot and must be manually enabled or started via the terminal.


Will update this repo if I get other things working again. 
