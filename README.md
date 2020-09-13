# pixelbookgohardware
hardinfo report on pixelbook go with all kinds of useful info.

As of right now, September 2020, the audio in Linux on Kaby Lake based devices (chromebooks) doesn't work. Here I'm trying to figure it out, since I just bought one and want to run linux on it, without chromeos.

The hardware inside the Pixelbook Go (ATLAS) is KBL_DA7219_MAX98373.
Decodes as KabyLake, with DA7219 and MAX98373 audio chips. 

What I've tried so far:
I installed stock ubuntu 20.04. No sound and no UHD graphics, but the system was stable/performing correctly with screen and keyboard backlight controls under the stock kernel. 
I manually upgraded the kernel (dpkg method) to 5.8.9 (https://kernel.ubuntu.com/~kernel-ppa/mainline/v5.8.9/) and the UHD screen started working perfectly.
But still no sound.

From a hardware perspective, it looks like both of these are already in upstream kernels since 5.6 (https://www.phoronix.com/scan.php?page=news_item&px=Linux-5.6-Sound), also seems to be the default hardware in Jasper Lake configs.

So it seems that the issue is either: 1) a driver config problem or 2) a sound routing config problem.

Based on the fact that no sound card shows up in Ubuntu, I'm guessing it's certainly 1 and probably 2 as well.

Solutions people have come up with that I either haven't tried or didn't work for me:
Gallium OS issue, solution for a different KBL configuration, the same approach might work, need to copy out the files.
https://github.com/GalliumOS/galliumos-distro/issues/536 

Github user @Flantel has also come up with a solution for the EVE chromebook (full on Pixelbook, not the Go variety):
https://github.com/flantel/pixelbook-linux/blob/master/implementation-details.md#audio-support
Should be possible to mod this by changing the various paths/files from EVE firmware to ATLAS. Being lazy I tried it without mods, and unsurprisingly it didn't work.

Here's a relevant patch from Intel, https://patchwork.kernel.org/patch/10774385/ which I guess is now included?

I also have an Acer Chromebook 14 running Ubuntu 20.04, got the sound fixed on that by swapping in this asound.state file 
https://pastebin.com/c4qU9hdg
However I haven't found a similar file lurking anywhere for the pixelbook go.

In terms of drivers, following from the gallium os issue, I've been exploring my Chromebook /lib/firmware/intel folder. It contains:
dsp_fw_578A29B8-0890-446F-A80A-2FE202AE0DBA.bin
dsp_fw_B489C2DE-0F96-42E1-8A2D-C25B5091EE49.bin
dsp_fw_C75061F3-F2B2-4DCC-8F9F-82ABB4131E66.bin
dsp_fw_kbl.bin
dsp_fw_kbl_v1037.bin
dsp_fw_kbl_v2630.bin
dsp_fw_kbl_v3266.bin
dsp_fw_kbl_v3420.bin
ibt-18-16-1.ddc
ibt-18-16-1.sfi
ipu3-fw.bin
irci_irci_ecr-master_20161208_0213_20170112_1500.bin

Within this folder, dsp_fw_kbl.bin is a symlink to dsp_fw_kbl_v3420.bin. This file shows up in stretch backports for https://packages.debian.org/stretch-backports/all/firmware-intel-sound/filelist

Something else interesting I found on my journey - a bunch of config files for the intel drivers, but the DA7219 & MAX98373 combo isn't included.
https://github.com/torvalds/linux/blob/master/sound/soc/intel/boards/Kconfig
Could this be the missing piece? Maybe one of them. It might explain why the default intel sound driver doesn't show any devices. 

Moving on to the config side, it sounds like UCM (Use Case Manager) for ALSA has evolved from version 1 to version 2 (UCM dir -> UCM2 dir), and presumably changed. This is an issue because any files exported from the chrome system (Kernel 4.4.223-17591-g3f222006e533) will also need to be updated to use UCM2, or a legacy UCM something magically installed.

Finally, tangental to the above, seems like Mediatek are using a v.similar combo, but not identical here on their MT8183 SoC: https://cateee.net/lkddb/web-lkddb/SND_SOC_MT8183_DA7219_MAX98357A.html 
