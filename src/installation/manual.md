# Manual installation

If the installer doesn't suffice for you, you can also install the system
manually. For that, you will need a working Linux system on the hardware,
which means a Void live image, a live image of any other distro, or some
other environment.

## Preparation

You will need the following:

1. A Linux environment
2. A static binary copy of the `xbps` package manager, available at
   <https://repo.voidlinux-ppc.org/static/> - not needed on a Void system

Don't worry about the archives being marked `musl`, these work the same on
`glibc` as well. The key point here is that the binaries are statically linked,
so they will work on any distribution/environment regardless of the software
packages you have. **You just need to get the right archive for the endianness
you want.**

## Booting and setting up environment

Boot your Linux removable media. If you don't know how to, follow the
[boot instructions](./live-images/booting.md).

If the environment is not Void, grab your static `xbps` copy.

```
# wget https://repo.voidlinux-ppc.org/static/xbps-static-0.59_5.ppc64le-musl.tar.xz
```

or:

```
# wget https://repo.voidlinux-ppc.org/static/xbps-static-0.59_5.ppc64-musl.tar.xz
```

or:

```
# wget https://repo.voidlinux-ppc.org/static/xbps-static-0.59_5.ppc-musl.tar.xz
```

and then:

```
# mkdir sxbps && cd sxbps
# tar xvf ../xbps-static*.tar.xz
```

You don't need most of the binaries, the one you are interested in is
`xbps-install.static`, located in `usr/bin`.

**Also, everything mentioned in this guide should be run as root.**

### Time

The time may be set incorrectly, especially on old Macs with bad batteries or
in VMs. Use the `date` command to set it to at least the correct day.

### Setting up the target drive

See the [Partitioning Notes](./live-images/partitions.md) for more details about
partitioning your disk.

Once you have partitioned your disk, format the `/` and mount it:

```
# mkfs.ext4 /dev/sdXN
```

Now we have a target filesystem at `/dev/sdXN`. Mount it:

```
$ mkdir -p /media/rootfs
$ mount /dev/sdXN /media/rootfs
```

## Installing initial system

Let's assume we have a target rootfs partition mounted at `/media/rootfs`, as
set up in the section above. Change your current directory to the `usr/bin`
location where you extracted the static `xbps` archive, so that
`xbps-install.static` is present in the current directory.

Then proceed to install a minimal system. Again, anything `# foo` is a comment.

```
# export XBPS_ARCH=ppc64le
# ./xbps-install.static -R https://repo.voidlinux-ppc.org/current -r /media/rootfs -S base-voidstrap
```

This will get a minimal system installed. Alter `XBPS_ARCH` as needed, for
example for 32-bit `musl` it will be `ppc-musl`. We're not installing a full
system yet, as we're not sure about the outside environment's software selection.
The smaller the system is, the less likely it is to result in a failure.

Now we will need to set up the target so that we can chroot into it.

```
# cp /etc/resolv.conf /media/rootfs/etc
# cp /etc/hosts /media/rootfs/etc
# mount -t devtmpfs none /media/rootfs/dev
# mount -t proc none /media/rootfs/proc
# mount -t sysfs none /media/rootfs/sys
```

If the `dev`, `proc` and `sys` directories do not exist in the target for some
reason, create them. This is a sign of a failed configuration step, if the
output of `xbps-install` doesn't show any errors, you should probably be fine.

Now is time to switch to the target system and install the rest of it.

```
# chroot /media/rootfs
```

If the configuration failed for whatever reason, reconfigure the installed
packages. You can most likely do this anyway, it will not do any harm, but it
should not be necessary if the initial system installation did not fail.

```
# xbps-reconfigure -f base-files
# xbps-reconfigure -f -a
```

## Configuring the system

We can proceed to install everything else.

```
# update-ca-certificates
# xbps-install -S
# xbps-install base-system
```

For glibc targets, it is necessary to enable a locale. The list is in
`/etc/default/libc-locales`. You then need to reconfigure the appropriate
package. For example:

```
# sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/default/libc-locales
# xbps-reconfigure -f glibc-locales
# echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

You also need to set a timezone and a hostname.

```
# echo 'TIMEZONE="Europe/Prague"' >> /etc/rc.conf
# echo foo > /etc/hostname
```

You need to set the root password, otherwise you will not be able to log in in
the target system.

```
# passwd root
```

Finally, enable some services by default.

```
# ln -s /etc/sv/dhcpcd /etc/runit/runsvdir/default/
# ln -s /etc/sv/sshd /etc/runit/runsvdir/default/
```

You need `dhcpcd` for internet access (it's set up for DHCP out of box, can be
configured for static IP) and `sshd` is optional. You can pre-enable any other
services in `/etc/sv` the same way.

## Bootloader setup

### OpenPOWER

As OpenPOWER systems use Petitboot, which is embedded in the firmware, there is
very little you have to do. Only a few things:

```
# xbps-install grub-utils
# mkdir -p /boot/grub
```

We only install the utils, as we'll be using those to generate a `GRUB`
configuration file, which Petitboot can read and parse. Edit `/etc/default/grub`
to update your kernel commandline. It is also a good idea to add the following line:

```
GRUB_DISABLE_OS_PROBER=true
```

This drastically reduces the time needed to generate the configuration file,
and `os-prober` is kinda useless on Petitboot anyway as it scans every storage
medium separately.

```
# update-grub
```

### PowerPC Macs

We will need to install the OpenFirmware bootloader:

```
# xbps-install grub-utils grub-powerpc-ieee1275
```

Also utilities to deal with HFS:

```
# xbps-install hfsutils hfsprogs
```

Let's assume the bootstrap partition is `/dev/sdXM`. Create a mountpoint for
the bootstrap partition, format it and mount it:

```
# mkdir -p /media/bootstrap
# dd if=/dev/zero of=/dev/sdXM bs=512
# hformat -l bootstrap /dev/sdXM
# mount -t hfs /dev/sdXM /media/bootstrap
```

And proceed to install the bootloader, then unmount the bootstrap partition:

```
# grub-install --macppc-directory=/media/bootstrap /dev/sdXM
# umount /media/bootstrap
# rmdir /media/bootstrap
```

Unfortunately, that's not all you need to do. You still need to bless the
directory with the bootloader and set up the file type for the boot script:

```
# hmount /dev/sdXM
# hattrib -t tbxi -c UNIX :System:Library:CoreServices:BootX
# hattrib -b :System:Library:CoreServices
# humount
```

Finally, generate the configuration file:

```
# update-grub
```

It is also recommended to uncomment the following line in `/etc/default/grub`:

```
GRUB_TERMINAL_OUTPUT=console
```

and then run `update-grub` again. This is necessary because graphical GRUB
is very slow on Macs and in virtual machines it will not work at all. You can
also add `GRUB_DISABLE_OS_PROBER=true` to prevent `update-grub` from scanning
other drives, which speeds it up considerably.

### Other OpenFirmware

We will need to install the OpenFirmware bootloader:

```
# xbps-install grub-utils grub-powerpc-ieee1275
```

Then we will need to install the bootloader into the `PowerPC PReP boot`
partition, with some files going to `/boot/grub`. That's easy:

```
# grub-install --boot-directory=/boot /dev/sda1 # must point it to the PReP partition
# update-grub
```

Before doing `update-grub`, maybe tweak your `/etc/default/grub`, see the
OpenPOWER section.

## Booting

A system set up like this should be bootable. For usage, follow the Void Linux
handbook: https://docs.voidlinux.org/
