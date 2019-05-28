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

Once you're done, boot up your media on your target hardware. You should be presented with `GRUB`, where you can pick the way to run your live OS. You can go with either way, the first one is better for low RAM machines.

<picture>

Once booted, you will be asked to log in. Once you do that, you will have a live system at your disposal, which can be used for recovery or installation or anything else.

<picture>

## Starting the installation

Start the installation by typing `void-installer`. The installer will automatically detect your hardware and present you with the appropriate options. You can exit the installer at any time and restart it later, the configuration is saved.

<picture>

Most settings are obvious, so configure your keyboard, network, locale and so on. The only one worth pointing out at the beginning is the installation source. You should use a network installation, which will download packages from the Internet, giving you a clean system. Local installation just copies the live system into the hard drive.

<picture>

## Partitioning the target drive

Once you're done configuring the generic bits, you will need to partition your target drive, unless you have already done so beforehand. The installer does not partition the drive itself. It will merely execute the right tool for it.

This is what you will be presented with when installing on an OpenPOWER of SLOF system:

<picture>

And on a Mac system:

<picture>

### OpenPOWER

OpenPOWER systems don't use an external bootloader, they have their Petitboot bootloader builtin in the firmware. There is no need for a boot partition, a boot sector, any flagging, or anything like this. You just need a single partition for the root filesystem, and a `swap` partition if you wish.

The partitioning tool of choice is `cfdisk`. It is recommended that you create a GUID Partition Table (GPT). Then set it up as required. It should be at least something like this:

<picture>

### SLOF

For SLOF systems, you want to install `GRUB`. You need a special partition (`PowerPC PReP Boot`) sized around 10MB to put `GRUB` onto. Then you need the root filesystem partition and maybe some `swap`.

The recommended partition table for most SLOF machines is `MBR` (`DOS`). It is also possible to use `GPT` for POWER8 and newer SLOF machines, but not recommended. Anyway, you want something like this:

<picture>

### NewWorld PowerPC Macs

These use their specific flavor of OpenFirmware. The partitioning tool of choice is `pmac-fdisk` (sometimes called `mac-fdisk`). The minimum requirement for booting Linux with `GRUB` on those is a roughly 10MB `Apple_Bootstrap` partition followed by the rootfs partition (`Apple_UNIX_SVR2`) and possibly a swap partition (also `Apple_UNIX_SVR2`).

The partitioning scheme is `APM` (Apple Partition Map). You will want something like this:

<picture>

### OldWorld PowerPC Macs

These are not supported by `GRUB` or the installer. You will need to figure that out yourself. Fortunately, only the oldest G3 Power Macs are OldWorld.

### Other hardware

Anything else will have its own specific kind of setup.

## Filesystem creation

Once you have your drive partitioned, you will need to put filesystems on it.

### OpenPOWER

There it's straightforward. It will present you with one (or two with `swap`) partition.

<picture>

Format this partition with your preferred filesystem and give it the `/` mount point. Then if you have swap, mark it with `swap` and when asked whether to create a filesystem on it, answer `yes`. It does not actually create a real file system, but it will set up the `swap`. Then you're good to go.

### SLOF

You will have two partitions.

<picture>

The `PowerPC PReP boot` partition needs to be marked as `ppc_prep` and when asked whether to create a filesystem on it, answer `yes`. It does not actually contain a real file system, but the installer will do some preparation on it.

<picture>

Then format the root partition with your preferred filesystem, give it the `/` mount point and let the installer create a filesystem on it. If you have `swap`, mark it with `swap` and when asked whether to create a filesystem, answer `yes`. Then you're good to go.

### NewWorld PowerPC Macs

You will be given something like this:

<picture>

The first partition is also the partition table itself, so leave it alone. You want to format the `bootstrap` partition as `hfs`:

<picture>

Have it mount at `/boot/apple_bootstrap`:

<picture>

Then format the root filesystem partition as you wish, and if you have `swap`, mark it with `swap` and when asked whether to create a filesystem, answer `yes`. That's pretty much it.

### OldWorld PowerPC Macs

These are not supported by the installer.

### Other hardware

These will have their own specific ways.

## Bootloader configuration

### OpenPOWER

As there is no bootloader to install on OpenPOWER, the installer will only ask you this:

<picture>

You will want to answer `yes`. That will make sure to create a `GRUB` configuration file (without actually installing `GRUB`) which the firmware bootloader can then parse to present you with options.

### SLOF

You will be asked something like this:

<picture>

Select the `PowerPC PReP boot` partition. Whether to use a graphical bootloader or not is up to you, so answer whichever you want. If you select `no`, you will be given a text menu (but still interactive).

### NewWorld PowerPC Macs

<picture>

You will probably want to select the drive you will be booting.


### OldWorld PowerPC Macs

You need to set these up manually.

### Other hardware

You need to set these up manually.

## Installing

Finally, you will be presented with one of the three:

<picture>
<picture>
<picture>

That depends on your hardware configuration. If you selected network installation, the installer will download some packages, after which you will need to confirm it to resume installation.

<picture>

## Finalizing

Once this is all done, you will be asked whether you want to enter the installed system's shell:

<picture>

Considering you will only have the base command line system installed, this is a chance to do some more installations or finishing touches before rebooting. If you choose to enter the shell, you will be brought into a `chroot` of the system.

When you're done, simply type `exit` and then you will be asked whether to reboot the system:

<picture>

## Booting

If everything went well, you will get something like this after reboot:

<picture>

Then you can boot and enjoy your new Void system:

<picture>

