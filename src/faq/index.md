# FAQ

This FAQ attempts to cover assorted questions not covered in the other chapters.

## Table of Contents

<!-- toc -->

## Will multilib be supported?  

No, it is not planned. However, the compiler is built as bi-arch, which
means `-m32` works, and you can have it emit 32-bit code. This is useful for
low level stuff (e.g. GRUB, which needs to emit 32-bit big endian code
independent on a libc) while not burdening the higher level infrastructure.
If you really need to build or use 32-bit software, use a 32-bit chroot, it
should work just fine.

## I'm using big endian with an ASpeed VGA and colors are broken.  

This is a problem with the kernel `ast` driver. Since the fix appears
to be non-trivial and there is no proper patch available, this is WONTFIX from
our side. Either use a dedicated GPU (ideally PCIe, USB2 DisplayLink is known
to work) or use little endian if you really need the `ast` to work properly.

## Why is yaboot not supported as a bootloader?

The `yaboot` project is deprecated. Void uses `GRUB`, which covers pretty much
all of the use cases plus more (that is, on systems that don't use `petitboot`,
but there it uses at least the GRUB configuration file).

## Why are you using 4k page size, when most distros use 64k?

One reason is that 64k is only supported starting with POWER8. New hardware
supports both 64k and 4k. On systems without a massive amount of RAM, 4k is
generally better because of its finer granularity (less fragmentation, guard
pages without wasting too much virtual memory etc.). Additionally, it is more
compatible with various software, which sometimes tends to assume 4k as is
standard on `x86` systems.

## How do i fix qemu/KVM not working on 4k kernels with machine newer than pseries-2.x?

If you really need to use something newer than `pseries-2.x` (which works fine),
you will likely get an error like this:

```
qemu-system-ppc64: KVM can't supply 64kiB CI pages, which guest expects
```

If you are on a system that has Radix MMU (POWER9 and newer), you can work
around this by using something like `-machine pseries,cap-hpt-max-page-size=4096`.
This will prevent booting of 64k guest kernels with HPT though; Radix enabled
kernels will work fine.

On POWER8, there is no known workaround.

## A live USB fails in dracut when mounting root filesystem image

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

## When partitioning an APM with pmac-fdisk, the partition table does not refresh afterwards

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

## I can boot the installation media on a Mac, but after installation the bootloader doesn't come up

Since several people complained about this, it probably deserves its own FAQ
entry.

You need to verify that your bootstrap partition has the correct partition
type. Like noted in the Partitioning Notes section, `pmac-fdisk` will happily
let you reverse the parameter order and create a partition of type `bootstrap`
named `Apple_Bootstrap` rather than a partition of type `Apple_Bootstrap` named
`bootstrap`. The bootloader will install on such partition, but Open Firmware
will not boot from it.

You can use the `p` command in `pmac-fdisk` to print out the partition table
and check.

## I get 'error: unrecognized number' when starting GRUB

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

## I have strange rendering issues with Qt-based desktops (KDE, LXQt, ...) on big endian systems

This may manifest as the panel or menus behaving strangely and so on.

The workaround is to start the desktop with `QT_QUICK_BACKEND=software`
exported. The cause here is some endianness related bug(s) in the default
backend.
