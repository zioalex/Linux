# Install a custome kernel in Gentoo or Sabayon
Initial procedure written for Gentoo but the process to adapt it for Sabayon is ongoing.

https://bugs.gentoo.org/show_bug.cgi?id=338084

## STEP 0 - Copy the config from the previous kernel configured
cp /usr/src/linux/.config /usr/src/NEW_KERNEL
ln -sf /usr/src/NEW_KERNEL /usr/src/linux

## STEP 1 - Build all the component needed to boot
### use default kernel config in /etc/kernels/kernel-config-x86_64-3.12.13-gentoo

	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs all

### use default kernel config in /etc/kernels/kernel-config-x86_64-3.12.13-gentoo and run mrproper if you have problem building

	genkernel --symlink --save-config --luks --lvm --udev --integrated-initramfs --compress-initramfs  all

### if you want to change the config with menuconfig and /usr/src/linux kernel dir

	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs --menuconfig all
	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs --gconfig all

### or with the .config configuration and a custom kernel dir
	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs --kernel-config=.config --kerneldir=/usr/src/linux-3.17.7-ge
ntoo all

## STEP 2
### Only if the kernel version is configured

	cd /boot/grub
	grub-mkconfig > grub.cfg2
	# check the config file
	mv grub.cfg _grub.cfg.old
	mv grub.cfg2 grub.cfg

## STEP 3
### Only if the kernel version is configured
Reboot with the new kernel and an update the packages that depend on the kernel

**GENTOO**

	Rebuild the external module
	https://bugs.gentoo.org/show_bug.cgi?id=408611#c6 compiler.h
	cp /usr/src/linux/include/linux/compiler.h /usr/include/linux

	emerge -av @x11-module-rebuild @module-rebuild app-emulation/virtualbox-modules

**SABAYON**

    equo install -pv $(equo query  revdeps sys-kernel/linux-sabayon-4.20.16 -q)

## Try to use this to rebuild what you think is needed:

	emerge -av1 $(qlist -IC xorg x11 drm)
	# or better
	emerge -av1 $(qlist -IC x11-drivers drm media-libs/mesa)

## EXTRA STEP Rebuild the initramfs
Gentoo build new genkernel:
### To solve /var/tmp/genkernel/initramfs-3.12.13-gentoo.cpio missed file

	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs initramfs

### Specifying a kernel version

	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs --kernel-config=/etc/kernels/kernel-config-x86_64-3.14.14-ge
ntoo-updated initramfs

	genkernel --symlink --save-config --no-mrproper --luks --lvm --udev --integrated-initramfs --compress-initramfs --kernel-config=.config initramfs


## Awesome / Gnome Tip and Triks
### avoid the gnome on screen keyboard also know as caribou or antler
    Comment the Exec lines from the follow files:
    /usr/share/dbus-1/services/org.gnome.Caribou.Antler.service  /usr/share/dbus-1/services/org.gnome.Caribou.Daemon.service

For some reason gnome doesn't recognize the laptop keyboards and start it also if disabled

Or use the follow plugin:

    https://extensions.gnome.org/extension/1326/block-caribou/

### Error
./usr/gen_initramfs_list.sh: Cannot open '/var/tmp/genkernel/initramfs-4.20.0-sabayon.cpio
remove the CONFIG_INITRAMFS_SOURCE line from the .config

The kernel boot process hang?
---
Check the initrd link in the grub.cfg file.

    initrd  /intel-uc.img /initramfs-genkernel-x86_64-4.20.0-sabayon

grub-mkconfig is creating the new entry with an initrd reference although it is not needed. Remove it from grub.cfg.
Find a better way to handle it


### links
https://bbs.archlinux.org/viewtopic.php?id=188513
https://www.reddit.com/r/archlinux/comments/2jjmvi/disable_caribou_in_gnome_314/
