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

#### OpenFirmware

OpenFirmware is used on older pre-OpenPOWER hardware as well as some niche enterprise systems. This section is TBD, I don't own any such hardware myself.

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
$ xbps-install grub-utils  # to generate a grub.cfg
$ xbps-install grub-powerpc-ieee1275 # only for non-OpenPOWER systems without Petitboot if you want to use grub
```

At the point of writing this, the default kernel is 4.19. Our kernels are generic, i.e. for 64-bit big endian systems they're built for POWER4 and newer. They are known for panic on modern POWER8 and newer hardware; this seems to have gotten fixed with kernel 5.0. Therefore, if you are installing a big endian system, you will need to install a 5.0 or newer kernel (or recompile the 4.19 kernel for POWER8 and newer CPUs). On little endian, there is no problem and you don't have to do anything (they are compiled for POWER8 by default).

```
$ xbps-install linux5.0
```

Afterwards, create a location for `grub.cfg`. We will use `grub-utils` to generate a `grub.cfg` so that Petitboot can read kernel entries from it. For OpenFirmware systems, this is TBD.

```
$ mkdir /boot/grub
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

At last, update your GRUB options. For that, edit `/etc/default/grub`. It is a good idea to add the following line:

```
GRUB_DISABLE_OS_PROBER=true
```

At very least on OpenPOWER systems with Petitboot, every drive is scanned separately, and this drastically reduces the time needed to generate the configuration file.

You also want to remove all pre-set parameters from `GRUB_CMDLINE_LINUX_DEFAULT`. If you are on an OpenPOWER system and plan to have your default console go to a dedicated GPU instead of the onboard ASpeed VGA, add `modprobe.blacklist=ast` to it, but this is dependent on the particular hardware you are using and not specific to Void.

Finally, generate the configuration file

```
$ update-grub
```

## Booting

A system set up like this should be bootable. For usage, follow the Void Linux handbook: https://docs.voidlinux.org/
