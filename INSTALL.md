# Installation

These instructions apply when using the installer. You can also install the distro manually if you wish to have more control over the process. You can find the instructions [here](https://github.com/void-power/documentation/blob/master/INSTALL-manual.md).

## Supported targets

The installer supports OpenPOWER (Petitboot stack), CHRP (SLOF stack) and NewWorld PowerPC Macs (OF). Everything else, as well as niche configurations, will need some amount of manual setup.

- `ppc64le` - This is the best supported target we have with the greatest number of binary packages. It supports only `POWER8` and newer hardware, either OpenPOWER (with Petitboot) or CHRP (with SLOF).
- `ppc64` - Same as above, plus any AltiVec-supporting ppc64 CPU starting with PowerPC 970.
- `ppc` - Generic 32-bit PowerPC, no AltiVec requirement.

For each target there's a matching `-musl` target as well, that lets you use the `musl` libc instead of `glibc`. The hardware support is the same for those.

## Preparation

You will need the following:

- A USB stick or an empty CD/DVD
- A Void live system. You can get images [here](https://void-power.octaforge.org/live/)

Simply `dd` the image onto your USB stick (recommended) or burn the image onto your optical media and you should be good to go.

## Booting

Once you're done, boot up your media on your target hardware. The way to boot will differ by hardware. In `qemu` it will start automatically (if selected the right boot drive), on OpenPOWER you will be presented with the boot options in Petitboot, on Macs you will have to drop into the OpenFirmware prompt (hold Cmd + Opt + O + F once you hear the initial bootup sound) and type something like `boot ud:,\\:tbxi`.

Either way, you will be presented with some kind of boot menu (typically Petitboot or `GRUB`), where you can pick the way to run your live OS. You can go with either way, the first one is better for low RAM machines.

![Live GRUB](https://i.imgur.com/GdY8T3c.png)

Once booted, you will be asked to log in. Once you do that, you will have a live system at your disposal, which can be used for recovery or installation or anything else.

![Live OS](https://i.imgur.com/8RrB9yU.png)

If you run into a message like `dracut: FATAL: Failed to mount block device of live image` after selecting your boot option in `GRUB`, it may be [this issue](https://github.com/void-power/void-mklive/issues/2). In that case, apply the mentioned workaround.

## Starting the installation

Start the installation by typing `void-installer`. The installer will automatically detect your hardware and present you with the appropriate options. You can exit the installer at any time and restart it later, the configuration is saved.

![Installer](https://i.imgur.com/2m1Csbq.png)

Most settings are obvious, so configure your keyboard, network, locale and so on. The only one worth pointing out at the beginning is the installation source. You should use a network installation, which will download packages from the Internet, giving you a clean system. Local installation just copies the live system into the hard drive.

![Installation source](https://i.imgur.com/EPUHboZ.png)

## Partitioning the target drive

Once you're done configuring the generic bits, you will need to partition your target drive, unless you have already done so beforehand. The installer does not partition the drive itself. It will merely execute the right tool for it.

This is what you will be presented with when installing on an OpenPOWER of SLOF system:

![SLOF partitioning](https://i.imgur.com/WJnFhrA.png)

And on a Mac system:

![Mac partitioning](https://i.imgur.com/p1so4it.png)

### OpenPOWER

OpenPOWER systems don't use an external bootloader, they have their Petitboot bootloader builtin in the firmware. There is no need for a boot partition, a boot sector, any flagging, or anything like this. You just need a single partition for the root filesystem, and a `swap` partition if you wish.

The partitioning tool of choice is `cfdisk`. It is recommended that you create a GUID Partition Table (GPT). Then set it up as required. It should be at least something like this:

![OpenPOWER cfdisk](https://i.imgur.com/Q83Fvht.png)

### SLOF

For SLOF systems, you want to install `GRUB`. You need a special partition (`PowerPC PReP Boot`) sized around 10MB to put `GRUB` onto. Then you need the root filesystem partition and maybe some `swap`.

The recommended partition table for most SLOF machines is `MBR` (`DOS`). It is also possible to use `GPT` for POWER8 and newer SLOF machines, but not recommended. Anyway, you want something like this:

![SLOF cfdisk](https://i.imgur.com/W3Gd6GH.png)

### NewWorld PowerPC Macs

These use their specific flavor of OpenFirmware. The partitioning tool of choice is `pmac-fdisk` (sometimes called `mac-fdisk`). The minimum requirement for booting Linux with `GRUB` on those is a roughly 10MB `Apple_Bootstrap` partition followed by the rootfs partition (`Apple_UNIX_SVR2`) and possibly a swap partition (also `Apple_UNIX_SVR2`).

The partitioning scheme is `APM` (Apple Partition Map). You will want something like this:

![pmac-fdisk](https://i.imgur.com/afs9kWx.png)

### OldWorld PowerPC Macs

These are not supported by `GRUB` or the installer. You will need to figure that out yourself. Fortunately, only the oldest G3 Power Macs are OldWorld.

### Other hardware

Anything else will have its own specific kind of setup.

## Filesystem creation

Once you have your drive partitioned, you will need to put filesystems on it.

### OpenPOWER

There it's straightforward. It will present you with one (or two with `swap`) partition.

![OpenPOWER filesystem](https://i.imgur.com/xZwB10n.png)

Format this partition with your preferred filesystem and give it the `/` mount point. Then if you have swap, mark it with `swap` and when asked whether to create a filesystem on it, answer `yes`. It does not actually create a real file system, but it will set up the `swap`. Then you're good to go.

### SLOF

You will have two partitions.

![SLOF filesystem](https://i.imgur.com/AaAU3vG.png)

Ignore the `PReP` partition for now, and format the root partition with your preferred filesystem, give it the `/` mount point and let the installer create a filesystem on it. If you have `swap`, mark it with `swap` and when asked whether to create a filesystem, answer `yes`. Then you're good to go.

### NewWorld PowerPC Macs

You will be given something like this:

![Mac filesystem](https://i.imgur.com/9TgYKTd.png)

The first partition is also the partition table itself, so leave it alone. The second partition is the bootstrap partition, also leave it alone. Format the root filesystem partition as you wish, and if you have `swap`, mark it with `swap` and when asked whether to create a filesystem, answer `yes`. That's pretty much it.

### OldWorld PowerPC Macs

These are not supported by the installer.

### Other hardware

These will have their own specific ways.

## Bootloader configuration

### OpenPOWER

As there is no bootloader to install on OpenPOWER, the installer will only ask you this:

![OpenPOWER GRUB](https://i.imgur.com/A5krLeH.png)

You will want to answer `yes`. That will make sure to create a `GRUB` configuration file (without actually installing `GRUB`) which the firmware bootloader can then parse to present you with options.

### SLOF

You will be asked something like this:

![SLOF GRUB](https://i.imgur.com/BYF8TwI.png)

Select the `PowerPC PReP boot` partition. Whether to use a graphical bootloader or not is up to you, so answer whichever you want. If you select `no`, you will be given a text menu (but still interactive).

### NewWorld PowerPC Macs

![Mac GRUB](https://i.imgur.com/yTk8Y7S.png)

Select the bootstrap partition you have created. The choice of whether to use a graphical bootloader or not is up to you, but keep in mind that it'll probably be slow (and if you're emulating a Mac in `qemu`, it does not work at all), so perhaps just choose text.

### OldWorld PowerPC Macs

You need to set these up manually.

### Other hardware

You need to set these up manually.

## Installing

Finally, you will be presented with something like this:

![Install](https://i.imgur.com/vMXUs9K.png)

Of course, the partitions formatted will differ. If you selected network installation, the installer will download some packages and unpack them, after which you will need to confirm it to resume installation.

![Downloaded](https://i.imgur.com/WfNcNnn.png)

### NewWorld PowerPC Macs

When installing on a Mac, it will ask you whether to update the boot device NVRAM record:

![NVRAM](https://i.imgur.com/goCBwPG.png)

You can choose to update it or not. If you update it, the installer will set the default boot device so that it will load `GRUB` by default. If you don't choose it, you will still get a blessed bootstrap partition, which means by holding the `Option` key during boot you should be able to choose the partition to boot from.

On `qemu`, this is known to fail, so you will receive an error dialog about that if you choose `Yes`. However, a failed NVRAM update will never make the whole installer fail.

## Finalizing

Once this is all done, you will be asked whether you want to enter the installed system's shell:

![Shell](https://i.imgur.com/LfF7V0A.png)

Considering you will only have the base command line system installed, this is a chance to do some more installations or finishing touches before rebooting. If you choose to enter the shell, you will be brought into a `chroot` of the system.

When you're done, simply type `exit` and then you will be asked whether to reboot the system:

![Reboot](https://i.imgur.com/Nkw5feY.png)

## Booting

If everything went well, you will get something like this after reboot:

![Final GRUB](https://i.imgur.com/gjQWTED.png)

Then you can boot and enjoy your new Void system:

![Final Void](https://i.imgur.com/xAPTsKR.png)
