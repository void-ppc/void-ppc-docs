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

### Why use 4 kB page kernels instead of 64 kB (like other distros)?

One reason is that 64k is only supported starting with POWER8. New hardware
supports both 64k and 4k. On systems without a massive amount of RAM, 4k is
generally better because of its finer granularity (less fragmentation, guard
pages without wasting too much virtual memory etc.). Additionally, it is more
compatible with various software, which sometimes tends to assume 4k as is
standard on `x86` systems.

## Virtual Machines

### KVM not working on 4k kernels by default

The default `pseries` machine type will raise an error like this:

```
qemu-system-ppc64: KVM can't supply 64kiB CI pages, which guest expects
```

One workaround is using `-machine pseries-2.11` or other `2.x` series.

If you are on a system that has Radix MMU (POWER9 and newer), you can also work
around this by using something like `-machine pseries,cap-hpt-max-page-size=4096`.
This will prevent booting of 64k guest kernels with HPT though; Radix enabled
kernels will work fine.

On POWER8, there is no known workaround other than downgrading machine type.

## Graphics

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

## Networking

### WiFi does not work on Apple machines (b43)

See the [post-installation section](../post-installation/index.md#wireless-networking).
