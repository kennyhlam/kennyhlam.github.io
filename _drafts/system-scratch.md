---
layout: post
title: Building a system from scratch
tags: build
---

This post is mostly aimed at some of the things I always end up doing whenever I'm resetting my system, and some issues I always encounter. I have a slightly unique setup build for my desktop, and every once in a while it encounters some obscure errors that are very hard to track down and often make it easier to just go ahead and re-install most things from scratch.

For example, my last system accumulated numerous errors associated with updating grub. It wasn't that much of an issue because I just ignored those errors and could still boot into my Ubuntu, but I would get system errors whenever I got updates from Ubuntu / apt update items.

Anyways, to start off with, here's some details on my system's hardware:

* Not too exciting to talk about: CPU, PS, case, fans, etc.
* Low-cost mobo with RAID support (specifically, RAID1, this will be important later)
* 2x Seagate 2TB drives (a third is hanging around waiting to be installed)
* 1x Samsung 128GB SSD
* 1x SanDisk 128GB SSD
* Relatively low-cost GPU (1050 Ti)

## RAID
So RAID is relatively important to me, since I'm one of those people who's paranoid about data loss. I'm getting to the point where things are getting a bit cozy in terms of storage and I'm getting close to the 2TB being full, and this will happen to anyone who's a bit of a data hoarder or works with large data-intensive files (photos, videos, etc).

Everyone will tell you RAID is for redundancy, not for backup. You should always have an off-site backup just in case a natural (or man-made) disaster hits and floods your machine or something of the sort. The issue with backups is that depending on the provider or how you did it, it could take some time to repair your data, so RAID just provides an easy uptime solution.

So it turns out, somewhat little (or maybe not so little if you're more into hardware) known fact is that the RAID that comes with lower-end motherboards is generally what's called `FakeRAID`. You can read more [here][fakeraid1] or [here][fakeraid2]. The tl;dr of both is that `FakeRAID` is generally bad, but a cheap solution if you want RAID. The big issue with hardware RAID is that the exact functioning is vendor-specific, so if your RAID controller fails, you have to find another solution that's a duplicate of your old one. Software has obvious issues that it's implemented at a higher level and hence is going to be slower.

That being said, I've been using `FakeRAID` for a while and haven't been too affected by the slowness. I've also already had one drive fail on me and have successfully repaired my array. I generally have a `RAID1` (mirrored) setup since it's the easiest, and I don't have exorbitant amounts of data. This, however, also means that I always have to have double the amount of space I'm planning to use.

Wanting to avoid vendor lock-in, the solution I'd like to go after is basically software RAID. After doing some browsing and some research, seems like there's two major solutions that will straight up offer you RAID: `DrivePool` or `FlexRAID`. The good people at [reddit][datahoarder-drivepool-snapraid] and [reddit][datahoarder-drivepool-snapraid2] seem to basically settle on `DrivePool` if you want something like a `RAID1` solution (use snapraid if you want something more like a RAID6 solution, this saves on disks you need to backup your data). I also looked at some comment threads on the `FlexRAID` forum, and the developer(s?) seem somewhat aloof and unstructured, not exactly what I want out of support for my data. I (unfortunately) have most of my data stored in Windows (NTFS) format since most of my photo editing is done in Windows, but people seem to want to steer you away from the Storage Spaces build into Win10 in general.


## Operating Systems & Remaining Disks

I run a dual system with both Windows10 and Ubuntu (now 18.04). At some point, I had both OS's on the same disk, but something (I don't remember what) happened, and grub and the bootloader for Windows kept interacting in undesirable ways, and at some point I decided it wasn't worth my time or effort to keep chasing down these bootloader conflicting issues (Windows doesn't play nice with anything, surprise). This does mean when I boot into the BIOS, I have to choose my startup disk appropriately, but because I rarely restart my desktop it's not a big deal (also, reading [this][bios] about BIOS really enlighted me to the firmware/lower level of things). Fortunately, if you just choose a disk to install Windows on, it generally just figures everything out for itself (it basically comandeers everything, so that's to be expected). Ubuntu is also rather easy to install, but a simple mistake I kept making this last time is that when you choose where to install your bootloader for Ubuntu, make sure you choose the **disk** to put the bootloader onto, not the hard drive partition where you're going to house Ubuntu (if you choose the partition, it simply won't boot from this disk). IMO, this is a general shortcoming of the installer, since most people will boot from a disk (I'm sure some people boot from a specific partition, but it's easier to boot from a whole disk).

## Software

I use Windows for mostly consumer-related stuff or gaming, so this block applies only to Ubuntu. I just generally install base suite of software that I like having on my machine (Python with virtualenv for general stuff/ML, Ruby with Jekyll for this site, shell configs, etc.). Generally, the sequence of steps:

1. Run the usual, `apt update`, `apt upgrade`, install the things you want like Chrome/Spotify/Steam
2. Ensure `git` is installed, generate your `SSH` key, ensure `vim` is on there, set up dot-files, change over to `zsh` and put my theme on and reboot to get your new shell: [steps here][config-sh]
3. Install your python-related goodies: [steps here][python-config]
4. Install your ruby-related goodies: [steps here][ruby-config]

## CUDA & CuDNN

The final thing (which is somewhat optional) is to install CUDA and CuDNN on your system. You'll basically want this if you want to use your GPU for ML-related stuff.

1. [Install nvidia drivers][nvidia-drivers], you should check on the site to see the latest version that your card supports
2. Basically you're going to follow the top answer [here][cuda-install], but you'll need to make some modifications:
  * Download the local runfile for the CUDA version you want, you'll also have to download the patches if they exist
  * Figure out how to get into a terminal session (I believe in Ubuntu it's now `ctrl`+`alt`+`F3` since 18.04 moved to Gnome)
  * Stop lightdm: `sudo service lightdm stop`
  * `echo "blacklist nouveau" > /etc/modprobe.d/blacklist-nouveau.conf; echo "options nouveau modeset=0" >> /etc/modprobe.d/blacklist-nouveau.conf`
  * `sudo update-initramfs -u`, if you're really curious, I believe Ubuntu starts up a ramdisk when originally loading, and this tells it to update the ramdisk with not running the graphics drivers (but I could be very wrong. read the `man` page)
  * `sudo sh cuda_<version>_linux.run --override`, and you'll subsequently have to run each patch as well. Be sure to decline installing the drivers since you've already done this. By decoupling the drivers from CUDA, you can freely update your drivers whenever you want
  * `sudo service lightdm start`
3. You're going to follow a hybrid of the top two answers' installation instructions for CuDNN [here][cudnn-install].
  * Log into your nvidia account and go to the [CuDNN download page][cudnn-download]
  * Download the `deb` files you need, `libcudnn`, `libcudnn-dev`, and `libcudnn-doc`
  * `sudo dpkg -i <file>.deb`
  * On other distributions, you can just download the `tar` and put the files into your CUDA directory, but it looks like Debian-based systems which use the `deb` files install things into a slightly different location.
  * To verify on Ubuntu, `cat /usr/include/x86_64-linux-gnu/cudnn_v*.h | grep CUDNN_MAJOR -A 2`, which was found [here][cudnn-verify]

[fakeraid1]: https://skrypuch.com/raid/
[fakeraid2]: https://superuser.com/questions/245928/does-fake-raid-offer-any-advantage-over-software-raid
[datahoarder-anti-flexraid]: https://www.reddit.com/r/DataHoarder/comments/6a1vay/drivepool_snapraid_vs_flexraid/
[datahoarder-drivepool-snapraid]: https://www.reddit.com/r/DataHoarder/comments/7b1wl1/drivepool_and_storage_space/
[datahoarder-mergefs]: https://www.reddit.com/r/DataHoarder/comments/96pq6s/possible_to_show_multiple_external_usb_hdds_as/
[datahoarder-drivepool-snapraid2]: https://www.reddit.com/r/DataHoarder/comments/8e7fuj/windows_10_software_raid_questions/
[bios]: https://www.happyassassin.net/2014/01/25/uefi-boot-how-does-that-actually-work-then/
[config-sh]: https://github.com/kennyhlam/config/blob/master/config.sh
[python-config]: https://github.com/kennyhlam/config/blob/master/python/python.sh
[ruby-config]: https://github.com/kennyhlam/config/blob/master/ruby/ruby.sh
[nvidia-drivers]: https://www.tecmint.com/install-nvidia-drivers-on-ubuntu/
[cuda-install]: https://askubuntu.com/questions/799184/how-can-i-install-cuda-on-ubuntu-16-04
[cudnn-install]: https://askubuntu.com/questions/767269/how-can-i-install-cudnn-on-ubuntu-16-04
[cudnn-download]: https://developer.nvidia.com/cudnn
[cudnn-verify]: https://stackoverflow.com/questions/31326015/how-to-verify-cudnn-installation
