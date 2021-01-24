# Pixelbook Project Campfire
Unofficial and fully working dual boot Windows 10 and ChromeOS for the Google Pixelbook 

## A fully-functional dual boot system.
Google developed drivers for the Pixelbook for Windows, and MrChromebox's firmware has enabled UEFI boot. 

## Current Status

Here's what's working on both operating systems:

| Feature            | Status               | Notes                                                             |
|--------------------|----------------------|-------------------------------------------------------------------|
| WiFi               | Working              | Working                                                           |
| Bluetooth          | Working              | Working                                                           |
| Suspend            | Working              | Working                                                           |
| Touchpad           | Working              | Working (on Windows tap-to-click is broken, but it works.)        |
| Graphics Accel.    | Working              |                                                                   |
| Sound              | Working              | See notes below on how to enable                                  |
| Keyboard backlight | Working              |                                                                   |
| Touchscreen        | Working              |                                                                   |
| OS Updates         | Partially            | Windows 100%. Chrome OS, manual updates are required w/ Brunch.   |


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

### Credit

This wouldn't be possible without the support from MrChromebox for the firmware, @sebanc for audio fixes, @shrikant2002 for the multi-install.sh script, and GetDroidTips for the internal dual-boot procedure. 

## Installation:
You will need to create USB flash drives: a Windows 10 install USB, a Linux Mint USB.

1. Flash UEFI firmware. Read and follow [yusefnapora's excellent guide](https://github.com/yusefnapora/pixelbook-linux) on how to flash the UEFI firmware using MrChromebox's scripts. To do this, you will need to disable write protect with either the SuzyQable cable or by removing the battery. Stop at the section before [installing Ubuntu](https://github.com/yusefnapora/pixelbook-linux#installing-stock-ubuntu). 

3. Install Windows 10. All drivers for the hardware will be installed using Windows Updates, no need for 3rd party tools.

4. In Disk Management, shrink the Windows 10 volume. Create a new NTFS partition with at least 30GB space. 

5. For now, instead of Brunch stable, download [Brunch Unstable](https://github.com/sebanc/brunch-unstable/releases) branch with a few customizations. This is in order to use the 4.4 kernel that will allow audio to work properly. If you use Brunch stable it will install kernel 5.4 and you will have no sound. 

6. Read this tutorial on [GetDroidTips](https://www.getdroidtips.com/install-chrome-os/) This is the method we will use, with a few customizations.  
	- Customization 1: No need for 60GB or 100GB partitions are described in the guide. See step 4 above - 30GB is plenty, but any additional disk space you allocate is entirely up to you.
	- Customization 2: Use Brunch Unstable, see step 4.
	- For this guide, we are using the [Recovery image from ChromeOS 86](https://dl.google.com/dl/edgedl/chromeos/recovery/chromeos_13421.99.0_eve_recovery_stable-channel_mp-v2.bin.zip).

7. At this point, you should have the following ready:
	- A bootable USB drive with Linux Mint ready to go
	- All of the files as described in the GetDroidTips tutorial in a ChromeOS folder on your Windows drive somewhere. 

8. Boot into Linux Mint, connect to wifi and run the multi-install.sh script. Note the partition name you formatted to NTFS for the ChromeOS image. In may case it was `/dev/mmcblk0p5` so I entered `mmcblk0p5` at the prompt. 

9. For grub to work properly, copy the text the script provides. You only need the part that starts from img_part. Make sure there is no } in the end. We need to customize the grub entry with the following:
	- Replace "/kernel" in the grub configuration with "//kernel-chromebook"
	- add "options=native_chromebook_image" to the kernel command line.

My grub entry looks like this - yours may be different depending on where your NTFS partition is for the Chrome OS image: 

```
img_part=/dev/mmcblk0p5
	img_path=/chromos.img
	search --no-floppy --set=root --file $img_path
	loopback loop $img_path
	linux (loop,7)/kernel-chromebook boot=local noresume noswap loglevel=7 disablevmx=off \
		cros_secure cros_debug loop.max_part=16 img_part=$img_part img_path=$img_path \
		console= vt.global_cursor_default=0 brunch_bootsplash=default options=native_chromebook_image
	initrd (loop,7)/lib/firmware/amd-ucode.img (loop,7)/lib/firmware/intel-ucode.img (loop,7)/initramfs.img
```
		

10. Reboot into Windows, install Grup2Win and add your grub config as Custom Code. Reboot and boot into Chrome OS to set it up. You're done!





