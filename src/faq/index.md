# FAQ

This FAQ attempts to cover assorted questions not covered in the other chapters.

## Table of Contents

<!-- toc -->

## General

### Will multilib be supported?

No, it is not planned. However, the compiler is built as bi-arch, which
means `-m32` works, and you can have it emit 32-bit code. This is useful for
low level stuff (e.g. GRUB, which needs to emit 32-bit big endian code
independent on a libc) while not burdening the higher level infrastructure.
If you really need to build or use 32-bit software, use a 32-bit chroot, it
should work just fine.

## Boot

### Why is yaboot not supported?

The `yaboot` project is deprecated. Void uses `GRUB`, which covers pretty much
all of the use cases plus more (that is, on systems that don't use `petitboot`,
but there it uses at least the GRUB configuration file).

### Live USB boot fails in dracut (root mount)

If you get an issue like this (and GRUB has loaded fine):

```
mount: /run/initramfs/live: wrong fs type, bad option, bad superblock on /dev/sdb1, missing codepage or helper program, or other error.
dracut: FATAL: Failed to mount block device of live image
dracut: Refusing to continue
```

you might be dealing with a backup partition table of some system that was
previously on that USB stick overriding the new one. In that case, first do
the following before `dd`ing your ISO image onto the USB stick:

```
# wipefs -a /dev/sdN
```

where `sdN` is your USB stick.

### GRUB raises 'error: unrecognized number'

If your error looks like this:

```
Welcome to GRUB!

error: unrecognized number.

Decrementer exception at $SRR0: ...
```

You are most likely on an iBook/PowerBook with BootROM `4.8.7f1`. Since GRUB
will fall back to OpenFirmware console, you can see the model and BootROM
version before the console comes up.

The issue also seems to affect at least some iMacs G5 and possibly other
models (though you might not see a decrementer exception), so proceed anyway
if you see the error, even if you don't have the model listed here.

The problem is a firmware bug, which results in spurious data left in the MMU,
which confuses GRUB during number parsing (it doesn't encounter a trailing
zero like it should, continues parsing, finds junk and fails).

The solution is to type this in the OpenFirmware console that comes up:

```
dev /memory@0 100000 1000 do-unmap
```

and reboot. GRUB should come up afterwards.

It is possible that this issue may manifest in different ways as well, as there
are multiple places in GRUB where this could potentially be a problem. So far
this is the only one we've come across, though.

### SLOF machine (e.g. qemu/pseries) not booting after install (MBR)

You might have forgotten to mark the PReP boot partition as bootable when
partitioning. In that case, boot the live image again, open the drive with
`cfdisk`, set the bootable flag, and save the changes. Your system should
boot then.

## Installation

### Partition table does not refresh after pmac-fdisk

This generally happens when reinitializing the partition table. The `pmac-fdisk`
utility does not properly wipe the previous partition table. Therefore, run the
following:

```
# wipefs -a /dev/sdN
```

where `sdN` is your target drive. Then initialize and partition the drive from
scratch.

You should not need this when modifying an existing APM (e.g. resizing partitions
or creating new ones in free space).

## Kernel

### Why use 4 KiB page kernels instead of 64 KiB (like other distros)?

There are multiple reasons:

1) 64 KiB pages are only supported starting with POWER8 and older archs will
   emulate them
2) Software is generally more compatible with 4 KiB, since other architectures
   use 4 KiB page size as well
3) On desktop/workstation oriented systems, 4 KiB will generally perform better
   thanks to finer granularity (which leads to lower fragmentation etc.)
4) The systems that benefit from larger kernel pages are mostly single-purpose
   servers with huge amounts of RAM, which Void does not primarily target
   (other distros primarily aim at servers)
5) Guard pages become viable again, without wasting virtual memory
6) Transparent hugepages mostly allow addressing scenarios which would benefit
   from bigger pages than default

There are some drawbacks, which typically have workarounds, and have their own
FAQ entries.

### Booting from Btrfs volumes on OpenPOWER (Talos 2 etc.)

By default it is not possible to boot directly from a Btrfs volume on OpenPOWER
systems because Btrfs volumes are tied to the page size of the kernel they were
created on.

Since the Skiroot kernel on OpenPOWER systems typically uses 64 KiB pages and
Void kernels use 4kB pages, the firmware will not see the volume. Conversely,
Void will not see Btrfs volumes created on 64kB page hosts, just like e.g.
x86 systems won't see it.

The same thing applies to big endian vs little endian systems - Btrfs is also
tied to endianness.

There have also been reports about higher resource usage and lower reliability
with Btrfs volumes on 64 KiB page hosts.

Workarounds include:

1) Use a separate `/boot` partition with a filesystem other than Btrfs
2) Compile your own kernel with 64 KiB pages

## Virtual Machines

For things such as 4 KiB page virtualization troubles, we have an entire
page (no pun intended) dedicated to it [here](../configuration/virtualization.md).

## Graphics

Also see the [dedicated page](../configuration/graphics.md).

### Broken colors in ASpeed VGA on big endian

This is a problem with the kernel `ast` driver. Since the fix appears
to be non-trivial and there is no proper patch available, this is WONTFIX from
our side. Either use a dedicated GPU (ideally PCIe, USB2 DisplayLink is known
to work) or use little endian if you really need the `ast` to work properly.

### Rendering issues with Qt-based desktops (KDE, LXQt, ...) on big endian

This may manifest as the panel or menus behaving strangely and so on.

The workaround is to start the desktop with `QT_QUICK_BACKEND=software`
exported. The cause here is some endianness related bug(s) in the default
backend.

**Note:** This should no longer be needed since the latest build of `qt5-5.14.2_2`
in the repos, as the code has been patched to always use `software` on big endian
by default (you can still override it back to the old default via the environment
variable). This is not an upstream change, however.

### Rendering issues with WebKit on big endian

Drivers on big endian are more buggy than usual, so with some cards you might
be having rendering issues with accelerated compositing, which is on by default.
Whether you are affected or not depends on the specific card. Old cards that do
not support unified shaders are more likely to be affected.

To use WebKit without accelerated compositing, export the following:

```
export WEBKIT_DISABLE_COMPOSITING_MODE=1
```

Then start the web browser of your choice again.

### System hangs shortly after boot/Xorg on older kernels (at least 4.4) and Radeon AGP

You might have to append `radeon.agpmode=-1` to kernel command line to disable
AGP GART. This has always been unstable on PowerPC and has been disabled in
mainline kernel for at least 2 years, but older branches still have it.

### System hangs in complex OpenGL applications on late PowerBooks (Radeon 9600/9700)

This is a known issue and there is currently no workaround available. It doesn't
seem to be the AGP mode, and basic OpenGL works (e.g. `glxgears`, or menus in
video games). This will be updated if something is found.

### Console is 800x600 rectangle on Mac Mini G4

On Mac Mini G4, the console may by default display only on a portion of the
screen. This is seemingly because there is an S-Video port connected and
used by Linux by default.

To work around this, edit `/etc/default/grub` and add the following parameter
into `GRUB_CMDLINE_LINUX_DEFAULT`:

```
video=SVIDEO-1:d
```

If you run into the same problem on another machine, the output port name
might be different. You can check by listing the contents of `/sys/class/drm`.

You can also set the console video mode with something like:

```
video=DVI-I-1:1280x1024@60
```

You need to combine the two if you do.

## Networking

### WiFi does not work on Apple machines (b43)

See the [relevant section](../configuration/apple.md#wireless-networking).

### HTTPS does not work (e.g. repo sync)

Your date/time may be set wrong (common trouble on Apple machines). Use `date`
to check it. If that is the case, use this:

```
# date -s "YYYY-MM-DD HH:mm:ss"
```

Replace the letters with the current date and time. After that, it is recommended
that you set up [NTP](../configuration/post-installation.md#ntp-time-syncing).
