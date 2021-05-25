# Pixelbook Project Campfire
Unofficial and fully working dual boot Windows 10 and ChromeOS for the Google Pixelbook 

## A fully-functional dual boot system.
Google developed drivers for the Pixelbook for Windows, and MrChromebox's firmware has enabled UEFI boot. 

## Current Status

Here's what's working on both operating systems:

| Feature            | Status               | Notes                                                             |
|--------------------|----------------------|-------------------------------------------------------------------|
| WiFi               | Working              | 	                                                                |
| Bluetooth          | Working              | 	                                                                |
| Suspend            | Working              | 	                                                                |
| Touchpad           | Working              | Working (on Windows tap-to-click is broken, but it works.)        |
| Tablet Mode	     | Working 		    | 								        |
| Sound              | Working              | See notes below - only works on kernel 4.4                        |
| Keyboard backlight | Working              |                                                                   |
| Touchscreen        | Working              |                                                                   |
| OS Updates         | Working	            | Yes - on Chrome OS, see notes below.  			        |


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

2. Install Windows 10. All drivers for the hardware will be installed using Windows Updates, no need for 3rd party tools.

3. In Disk Management, shrink the Windows 10 volume. Create a new NTFS partition with at least 30GB space. 

4. Instead of Brunch stable, we will download the latest version of [Brunch Unstable](https://github.com/sebanc/brunch-unstable/releases) with a few customizations. The Pixelbook can now use 5.4 kernel - see details below.

5. Read the official [Brunch github](https://github.com/sebanc/brunch) and understand the process of installing Brunch.

6. At this point, you should have the following ready:
	- A bootable USB drive with Linux Mint ready to go
	- All of the files as described in the Brunch github. 

7. Boot into Linux Mint, connect to wifi and run the chromeos-install.sh script. Note the partition name you formatted for the ChromeOS image. In may case it was `/dev/mmcblk0p5` so I entered `mmcblk0p5` at the prompt. Note: 512GB NVMe drives will have a differnt partition layout and naming format.

8. For grub to work properly, see below. We need to customize the grub entry with the following:
	- Replace "/kernel" in the grub configuration with "kernel-chromebook-5.4"
	- add "options=enable_updates,native_chromebook_image" to the kernel command line.

Optionally, copy the grub config added to this repo and use it in grub2win, just make sure to update the partition number if needed.

What this does is it tells Brunch to boot Chrome OS with the 5.4 kernel, and it also enables regular OS updates just like a normal install. My grub entry looks like this - yours may be different depending on where your NTFS partition is for the Chrome OS image: 

Grub2Win Config:

```
img_part=/dev/mmcblk0p5
	img_path=/chromos.img
	search --no-floppy --set=root --file $img_path
	loopback loop $img_path
	linux (loop,7)/kernel-chromebook-5.4 boot=local noresume noswap loglevel=7 disablevmx=off \
		cros_secure cros_debug options=enable_updates,native_chromebook_image loop.max_part=16 img_part=$img_part img_path=$img_path \
		console= vt.global_cursor_default=0 brunch_bootsplash=default 
	initrd (loop,7)/lib/firmware/amd-ucode.img (loop,7)/lib/firmware/intel-ucode.img (loop,7)/initramfs.img
```

Grub config (for Linux users, this would go in 40_Custom:
```
menuentry "ChromeOS" {
	img_part=/dev/mmcblk0p5
	img_path=/chromos.img
	search --no-floppy --set=root --file $img_path
	loopback loop $img_path
	linux (loop,7)/kernel-chromebook-5.4 boot=local noresume noswap loglevel=7 disablevmx=off \
		cros_secure cros_debug options=native_chromebook_image,enable_updates loop.max_part=16 img_part=$img_part img_path=$img_path \
		console= vt.global_cursor_default=0 brunch_bootsplash=default
	initrd (loop,7)/lib/firmware/amd-ucode.img (loop,7)/lib/firmware/intel-ucode.img (loop,7)/initramfs.img
}
```

9. Reboot into Windows, install [Grub2Win](https://sourceforge.net/projects/grub2win/) and add your grub config as Custom Code. Reboot and boot into Chrome OS to set it up. 

10. In Windows, make sure to disable fast startup or you will encounter problems from time to time booting into Chrome OS (you may see the bright screen with "We are repairing this device, please wait" but after several reboots, it still won't boot.) This procedure is explained in many places, here is [one tutorial](https://help.uaudio.com/hc/en-us/articles/213195423-How-To-Disable-Fast-Startup-in-Windows-10) to follow.. 
11. Back in Brunch, make sure to open crosh shell and issue this command to stop touchpad firmware updates from getting in the way: `sudo rm -r /opt/google/touch/scripts/*`





