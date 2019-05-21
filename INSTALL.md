# Installation

As of 2019-03-30, there is no installation media provided, but it's possible to install the system manually without great difficulties.

`ppc64le` glibc is the best supported target right now and includes the majority of packages normally offered by Void. `ppc64le-musl` is catching up right now and should also provide a usable desktop system with several desktop environments. The big endian ports are installable but have much fewer packages built for now, same with the 32-bit ports. All targets are known to boot on their supported hardware.

## Preparation

You will need the following:

1. A USB stick of any Linux distribution for the target you want to install (64-bit little endian distro for 64-bit LE Void, 64-bit BE distro for 64-bit BE Void, for 32-bit you need any BE environment, either 64 or 32-bit)
2. A copy of the `xbps` package manager, available at https://void-power.octaforge.org/static/

Don't worry about the archives being marked `musl`, these work the same on `glibc` as well. The key point here is that the binaries are statically linked, so they will work on any distribution/environment regardless of the software packages you have. **You just need to get the right archive for the endianness you want.**

**It is advisable to get a modern distribution (for example, the latest version of Ubuntu or Fedora for little endian, and Adélie Linux for big endian as well as 32-bit) in order to not run into any issues with HTTPS certificates (all the links use Let's Encrypt).**

## Booting and setting up environment

Boot your Linux USB stick and fetch+extract the archive in a directory, like this:

```
$ # only for a 64-bit little endian system
$ wget https://void-power.octaforge.org/static/xbps-static-0.53_1.ppc64le-musl.tar.xz
$ # only for a 64-bit big endian system
$ wget https://void-power.octaforge.org/static/xbps-static-0.53_1.ppc64-musl.tar.xz
$ # only for a 32-bit system
$ wget https://void-power.octaforge.org/static/xbps-static-0.53_1.ppc-musl.tar.xz
$ mkdir sxbps && cd sxbps
$ tar xvf ../xbps-static*.tar.xz
```

You don't need most of the binaries, the one you are interested in is `xbps-install.static`, located in `usr/bin`.

**Also, everything mentioned in this guide should be run as root.**

### Setting up the target drive

#### OpenPOWER

On modern OpenPOWER hardware, such as Raptor Talos 2 or any IBM OpenPOWER system, you don't need to worry about much, as these systems use the Petitboot bootloader inside a Skiroot environment. You just need to select a target drive and create at least one partition on it. The way you do this depends on the distribution you currently have booted. As an example:

```
$ # our target drive is /dev/sda, we will use fdisk, which is commonly present
$ # do not type any of the comments in here (# foo) into the command line
$ fdisk /dev/sda # this brings up a prompt of a sort
> g # create a GUID partition table (GPT)
> n # create a single partition, confirm the pre-filled values to use entire disk
> w # write changes and quit
$ mkfs.ext4 /dev/sda1 # create an ext4 filesystem on the target drive
```

Now we have a target filesystem at `/dev/sda1`. Mount it:

```
$ mkdir -p /media/rootfs
$ mount /dev/sda1 /media/rootfs
```

And that's it.

#### PowerPC Macs

**NOTE:** this section is preliminary and may not actually work.

PowerPC Mac systems use OpenFirmware. You will need to create an Apple Partition Map on the drive, in this kind of layout, when using the `GRUB2` bootloader: 

| Device    | Type                | Name      | Size | System             |
|-----------|---------------------|-----------|------|--------------------|
| /dev/sda1 | Apple_partition_map | Apple     | -    | Partition map      |
| /dev/sda2 | Apple_Bootstrap     | bootstrap | ~10M | NewWorld bootblock |
| /dev/sda3 | Apple_UNIX_SVR2     | rootfs    | any  | Linux native       |
| /dev/sda4 | Apple_UNIX_SVR2     | swap      | any  | Linux swap         |

You also have a choice of using the `yaboot` bootloader, which wants a smaller bootstrap partition around ~`800k`. `GRUB2` is more powerful, however. For instructions on how to use `yaboot`, you will need to look it up elsewhere.

First partition is also the partition table. Swap is optional. You can use `mac-fdisk` to create the partition layout for `GRUB2`:

```
$ mac-fdisk
> i # initialize the partition map
> C # create bootstrap partition, say 10m for size and Apple_Bootstrap for type
> c # create rootfs partition, specify a size you want
> c # create swap partition, specify a size you want
> w # write changes
```

And for `yaboot`:

```
$ mac-fdisk
> i # initialize the partition map
> b # create an 800k bootstrap partition for yaboot
> c # create rootfs partition, specify a size you want
> c # create swap partition, specify a size you want
> w # write changes
```

`GRUB2` requires the bootstrap partition to be formatted with plain old HFS (**not** HFS+). This will then be used as `/boot/grub`.

```
$ mkfs.hfs -h /dev/sda2 # create legacy HFS partition using -h
$ mkfs.ext4 /dev/sda3   # create a filesystem for root
```

With the filesystems ready, mount the rootfs:

```
$ mkdir -p /media/rootfs
$ mount /dev/sda3 /media/rootfs
```

And that's it.

#### Other OpenFirmware

**NOTE:** this section is preliminary and may not actually work.

Non-Mac OpenFirmware is different. You will need a `PReP` partition and a root partition at least. You don't need a special partition for `/boot/grub`, that can go on the root partition just fine. You should also be able to use `GPT`.

```
$ # our target drive is /dev/sda, we will use fdisk, which is commonly present
$ # do not type any of the comments in here (# foo) into the command line
$ fdisk /dev/sda # this brings up a prompt of a sort
> g # create a GUID partition table (GPT)
> n # create a partition for PReP boot, primary, ~10M
> t # to change type to PReP Boot - select partition 1, hex code 41 for PPC PReP Boot
> n # create a second partition and fill the rest of the disk
> w # write changes and quit
$ mkfs.ext4 /dev/sda2 # create an ext4 filesystem on the target drive
```

Now we have a target filesystem at `/dev/sda2`. Mount it:

```
$ mkdir -p /media/rootfs
$ mount /dev/sda2 /media/rootfs
```

And that's it.

## Installing initial system

Let's assume we have a target rootfs partition mounted at `/media/rootfs`, as set up in the section above. Change your current directory to the `usr/bin` location where you extracted the static `xbps` archive, so that `xbps-install.static` is present in the current directory.

Then proceed to install a minimal system. Again, anything `# foo` is a comment.

```
$ export XBPS_ARCH=ppc64le # ppc64 for big endian, append -musl for musl
$ # install base-voidstrap, a minimal base system package for container environments
$ # -R == repository URL, -r == target directory, -S == sync
$ # add /be to URL for big endian, and /musl for musl (/be/musl for BE musl)
$ ./xbps-install.static -R https://repo.void-ppc64.octaforge.org/current -r /media/rootfs -S base-voidstrap
```

This will get a minimal system installed. We're not installing a full system yet, as we're not sure about the outside environment's software selection. The smaller the system is, the less likely it is to result in a failure.

Now we will need to set up the target so that we can chroot into it.

```
$ cp /etc/resolv.conf /media/rootfs/etc # for network access
$ cp /etc/hosts /media/rootfs/etc       # for network access
$ mount --bind /dev /media/rootfs/dev
$ mount --bind /proc /media/rootfs/proc
$ mount --bind /sys /media/rootfs/sys
```

If the `dev`, `proc` and `sys` directories do not exist in the target for some reason, create them. This is a sign of a failed configuration step, if the output of `xbps-install` doesn't show any errors, you should probably be fine.

Now is time to switch to the target system and install the rest of it.

```
$ chroot /media/rootfs
```

If the configuration failed for whatever reason, reconfigure the installed packages. You can most likely do this anyway, it will not do any harm, but it should not be necessary if the initial system installation did not fail.

```
$ xbps-reconfigure -f base-files # will populate the rootfs with necessary dirs if not done already
$ xbps-reconfigure -f -a         # reconfigure all packages
```

## Configuring the system

We can proceed to install everything else.

```
$ update-ca-certificates   # just in case, make sure the certs are all up to date
$ xbps-install -S          # fetch package database from repository
$ xbps-install base-system # this is a full base package with a kernel etc.
```

At the point of writing this, the default kernel is 4.19. Our kernels are generic, i.e. for 64-bit big endian systems they're built for POWER4 and newer. They are known for panic on modern POWER8 and newer hardware; this seems to have gotten fixed with kernel 5.0. Therefore, if you are installing a big endian system, you will need to install a 5.0 or newer kernel (or recompile the 4.19 kernel for POWER8 and newer CPUs). On little endian, there is no problem and you don't have to do anything (they are compiled for POWER8 by default).

```
$ xbps-install linux5.0
```

For glibc targets, it is necessary to enable a locale. The list is in `/etc/default/libc-locales`. You then need to reconfigure the appropriate package. For example:

```
$ sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/default/libc-locales
$ xbps-reconfigure -f glibc-locales
$ echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

You also need to set a timezone and a hostname.

```
$ echo 'TIMEZONE="Europe/Prague"' >> /etc/rc.conf
$ echo foo > /etc/hostname
```

You need to set the root password, otherwise you will not be able to log in in the target system.

```
$ passwd root
```

Finally, enable some services by default.

```
$ ln -s /etc/sv/dhcpcd /etc/runit/runsvdir/default/
$ ln -s /etc/sv/sshd /etc/runit/runsvdir/default/
```

You need `dhcpcd` for internet access (it's set up for DHCP out of box, can be configured for static IP) and `sshd` is optional. You can pre-enable any other services in `/etc/sv` the same way.

## Bootloader setup

### OpenPOWER

As OpenPOWER systems use Petitboot, which is embedded in the firmware, there is very little you have to do. Only a few things:

```
$ xbps-install grub-utils  # to generate a grub.cfg
$ mkdir -p /boot/grub
```

We only install the utils, as we'll be using those to generate a `GRUB` configuration file, which Petitboot can read and parse. Edit `/etc/default/grub` to update your kernel commandline. It is also a good idea to add the following line:

```
GRUB_DISABLE_OS_PROBER=true
```

This drastically reduces the time needed to generate the configuration file, and `os-prober` is kinda useless on Petitboot anyway as it scans every storage medium separately.

You also want to remove all pre-set parameters from `GRUB_CMDLINE_LINUX_DEFAULT`. If you are on an OpenPOWER system and plan to have your default console go to a dedicated GPU instead of the onboard ASpeed VGA, add `modprobe.blacklist=ast` to it, but this is dependent on the particular hardware you are using and not specific to Void.

Finally, generate the configuration file

```
$ update-grub
```

#### PowerPC Macs

**NOTE:** this section is preliminary and may not actually work.

We will need to install the OpenFirmware bootloader:

```
$ xbps-install grub-utils grub-powerpc-ieee1275
```

Also utilities to mount HFS:

```
$ xbps-install hfsprogs
```

Then create a mountpoint for the bootstrap partition and mount it:

```
$ mkdir -p /boot/grub
$ mount -t hfsplus /dev/sda2 /boot/grub
```

And proceed to install the bootloader, followed by configuration file generation:

```
$ grub-install /dev/sda
$ update-grub
```

#### Other OpenFirmware

**NOTE:** this section is preliminary and may not actually work.

We will need to install the OpenFirmware bootloader:

```
$ xbps-install grub-utils grub-powerpc-ieee1275
```

There is no partition to mount for `/boot/grub` like on Macs, as it uses `PReP Boot`. Just install the bootloader and generate the config:

```
$ mkdir -p /boot/grub
$ grub-install /dev/sda
$ update-grub
```

## Booting

A system set up like this should be bootable. For usage, follow the Void Linux handbook: https://docs.voidlinux.org/
