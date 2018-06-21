---
title: "Installing Void Linux with full disk encryption"
description: "Walkthrough for installing Void Linux with LVM and LUKS encryption, including encrypted boot partition."
tags: [ "Void Linux", "GNU/Linux", "Installation" ]
lastmod: 2018-06-20
date: "2018-06-20T03:30:18"
categories:
  - "GNU/Linux"
  - "Installation"
disable_comments: true
---


Void Linux is one of the newer and less-known independent GNU/Linux distributions with a rolling release model. It is lightweight distribution that offers a ton of good stuff along with an 'old school' feel to the well-{made,polished} system. Few of the features that make Void Linux stand out are:

* Void Linux replaces openssl with a more secure implementation, [libressl][1].
* Void Linux does not use Systemd. Instead, it uses [runit][2]: an init system that is _extremely_ fast and really easy to use.
* Void Linux has its own package manager called [xbps][3]: a feature-packed, lightweight package management tool. Some people don't like it but trust me it is easy to use and "just werks".
* Void Linux provides multiple system images with two different implementations of the C Standard Library: 
	* Old and bulky [glibc][4]: almost all applications are compiled against glibc.
	* New and lightweight [musl][5]: although theoretically better and most of the things work, the application support isn't really _that_ great yet. 
* Void Linux supports all major architectures, and provides multiple images for lightweight desktop environments and also a barebones system image with no GUI.

For this guide, we will be using the barebones system image. You may choose to download an image with a desktop environment instead, and the tutorial will still remain same for most parts.

All downloads for Void Linux can be found [here][6]. I won't go into details on creating a bootable USB disk, you may use whatever tool you wish (dd, rufus) -- just make sure that it boots.

##### pre-installation faq

Here are some of the questions I would like to have cleared before getting you into reading this very lengthy post:

* __How sure are you that this works?__

	I used to run this setup on my hardware, and it worked really well. I assure you that your system will boot as long as you _carefully_ follow this guide.

* __Tell me more about \<insert stuff here\>?__

	First off, if you don't know much about encryption, or what LVM or LUKS is, this guide is __NOT__ for you. I will also not explain _every other thing_ I state in this guide, else for what I think is needed for you to get your system booting. If you are still interested, I suggest you to read more about \<insert stuff here\> and then decide for yourself.

* __But there already exists a guide for this on Void Linux wiki? Also why not contribute there instead?__

	Yes, and good luck on getting your system to boot with that guide. Why am I not contributing to the Void Linux wiki? I will, eventually. I just need some people (you) to be the guinea pigs (_not really_). It just takes some time to write a good wiki entry.

* __What can I expect to have working by the end of this guide?__

	If you follow this guide, you will end up with a barebones Void Linux installation with everything encrypted, including the `/boot` partition. You may then follow guides from the [wiki][7] to get your system running as you like.

* __How long will it take for installation?__

	The main system installation is really fast and easy. It is just that reading this guide and getting an encrypted system running is something that will require quite some time, entirely depending on how long you spend on reading stupid FAQs like these.


##### installation

Alright, enough of those questions. Assuming that you're ready to install Void Linux, and that you have made a bootable disk out of the system image and have booted into it, let's get started with the installation.

When you first boot into Void Linux, you will be greeted with a tty asking you to log into the system, or maybe even a desktop environment depending on the system image that you have downloaded.

Void Linux has a terminal-based ncurses installation helper, `void-installer`. Although unfortunately, this installer does not have an option for encrypted system installation. Instead, you may set your system up manually -- which is exactly what we're going to do. Now, the installation process may feel a bit similar to that of Arch Linux with an optional step to use the ncurses installer to save some time if you wish.

---

First off, the default shell for root on Void Linux is __sh__ (wise) and not __bash__. We will be switching to bash for convenience. Switching is as simple as typing 'bash' in the terminal.

We now need to format and create partitions for our new system. __parted__ is a good tool for formatting. We will also use it to create two basic partitions, one that serves as our ESP, and the other encrypted partition that holds our boot and root partitions.

```sh
parted /dev/sdX mklabel gpt
parted -a optimal /dev/sdX mkpart primary 2048s 100M
parted -a optimal /dev/sdX mkpart primary 100M 100%
parted /dev/sdX set 1 boot on
```

__(NOTE: Please replace X in sdX with your drive name. You won't be reminded again.)__

We need to setup LVM partitions on the second basic partition that we created and then encrypt these partitions with LUKS encryption. For that, we first need to install a couple of things using the Void Linux package manager:

```sh
xbps-install -S cryptsetup lvm2
```

* __lvm2__ is the LVM tool that we will use to create device mappings for our logical volumes
* __cryptsetup__ is the tool used to encrypt/decrypt the system using LUKS encryption.


I suggest making at least three logical partitions:

* a minimum of __200MB__ as a `/boot` partition -- to store your kernel and initramfs.
* a __2 * \<your RAM size\>__ (or just as big as your RAM) partition as a swap memory.
* a minimum of __30 GB__ space for `/root`.

You may also create additional partitions for `/home` or `/var`, but __do not__ skip any of the above mentioned ones.

Since our first partition (i.e: /dev/sdX1) is the ESP, the further partitions will end with numbers starting for 2.

__Make sure to enter a _really strong password_ when asked for it, and then also make sure that you write it down somewhere for safety.__

```sh
cryptsetup luksFormat -C aes-xts-plain64 -s 512 /dev/sdX2
cryptsetup luksOpen /dev/sdX2 crypt-pool
```

Here, we encrypt our disk and then open it as `crypt-pool`. We now need to create a logical volume for our system.

```sh
pvcreate /dev/mapper/crypt-pool
vgcreate vgpool /dev/mapper/crypt-pool
```

* __pvcreate__ generates a physical volume from __crypt-pool__
* __vgcreate__ creates a volume group from the generated physical volume.

[1]: libressl
[2]: http://smarden.org/runit/
[3]: https://wiki.voidlinux.eu/XBPS
[4]: glibc
[5]: musl
[6]: http://www.voidlinux.org/download/
[7]: https://wiki.voidlinux.eu/
