# Partitioning Notes

This is generally simple, but will differ depending on the hardware/firmware.

When using the installer, it will provide you with instructions appropriate
for your machine.

## OpenPOWER

OpenPOWER machines, such as the Talos 2 or IBM PowerNV servers, load into Linux
and feature a bootloader called Petitboot. The final kernel is then loaded via
the `kexec` mechanism. POWER8 and newer machines are in general OpenPOWER, but
not always (the other ones use SLOF, see below).

That's why there is no bootstrap partition necessary. However, the partition
that contains `/boot` must be mountable from the firmware environment. The
partition table does not matter either, as long as the firmware can see it;
the typical choice is either GPT or MBR.

**Necessary partitions:**

- Root partition (`/`)

**When using an unsupported root filesystem:**

- Root partition (`/`)
- `/boot` partition (with a supported filesystem)

Unsupported root file systems generally include out of tree ones such as ZFS,
and Btrfs is usually affected as well, since it requires page size and
endianness to match the machine where it was created, and Void uses 4 kB kernel
pages (a typical OpenPOWER firmware uses 64 kB).

## Open Firmware

Other machines the live images can boot on use Open Firmware. This comes in
several flavors:

- IBM PowerVM servers use SLOF (SlimLine Open Firmware)
- Virtual machines in `qemu` by default use SLOF
- Apple hardware uses Apple's version of OF

Open Firmware based machines always use some kind of bootstrap partition. The
bootstrap partition contains the first stage bootloader executable - this is
usually GRUB. The executable is just plain ELF.

The bootstrap partition is generally small, and **is not `/boot`**. The first
stage bootloader size is at most a few hundred kilobytes in general, unless
you manually install a full GRUB image, which may be larger (this is not
necessary in general, other GRUB modules can be loaded from a filesystem).

The rest of GRUB generally goes in `/boot/grub` which is either on your `/`
partition or your `/boot` partition if you have one.

**When using the installer, the bootstrap partition is what you should select
when it asks you where to install the bootloader.**

### SLOF

**Necessary partitions:**

- `PowerPC PReP Boot` (bootstrap)
- Root partition (`/`)

On pSeries, virtual machines and so on, the default recommended choice is a
MBR. On newer versions of SLOF, GPT will also work (and in virtual machines
it does), but MBR is a safe choice.

The bootstrap in SLOF is a special partition of type `PowerPC PReP Boot`.
Make it around 1 megabyte; this should be generous. This partition is never
mounted and does not contain a mountable filesystem.

Example:

```
# fdisk /dev/sdX
o # MBR, g for GPT
n # new partition for PReP boot; make it 1 megabyte
t # change partition type to PowerPC PReP boot
n # new partition for /
a # set first partition bootable
w # write changes and quit
```

The partition type number for `PowerPC PReP boot` should be `7` for GPT and
`41` for MBR.

### PowerPC Macs

**Necessary partitions:**

- `Apple_Bootstrap` (bootstrap)
- Root partition (`/`)

Macs use APM (Apple Partition Map), at least for booting. The actual `/`
partition can be on anything Linux supports.

You can use `pmac-fdisk` for APM partitioning. Standard `fdisk` does not
support it.

**Note that partitioning has nothing to do with filesystems present.**
Partitioning will only ensure that you will have the partitions with correct
types available. Do not get confused by partition types like `Apple_HFS`;
that is unrelated to the filesystem in the partition. You will format your
filesystems once everything is partitioned correctly.

You will need at least one partition on your APM, the bootstrap partition,
which will be used by OpenFirmware to invoke the bootloader. The actual OS
can be anywhere that GRUB can use. In an APM-only setup, your partitioning
will look for example like this:

| Device    | Type                | Name      | Size | System             |
| --------- | ------------------- | --------- | ---- | ------------------ |
| /dev/sdX1 | Apple_partition_map | Apple     | -    | Partition map      |
| /dev/sdX2 | Apple_Bootstrap     | bootstrap | 800k | NewWorld bootblock |
| /dev/sdX3 | Apple_UNIX_SVR2     | rootfs    | any  | Linux native       |
| /dev/sdX4 | Apple_UNIX_SVR2     | swap      | any  | Linux swap         |

You can create it for example like this:

```
# pmac-fdisk /dev/sdX
i                                    # init partition table, wipes all data
b 2p                                 # bootstrap partition
c 3p 120G rootfs                     # root filesystem (/)
c 4p 4p swap                         # swap partition, all unused space
w
q
```

**This is only for clean installations! It will wipe anything else present
on the drive. If you don't want that, read below.**

In an APM, the first partition is always automatic, being the APM itself.

The `b <x>` command in `pmac-fdisk` is functionally equivalent to something
like `C <x> 800k bootstrap Apple_Bootstrap`.

The bootstrap partition contains a legacy HFS filesystem when installed, but
the installer takes care of correctly formatting it and you should never worry
about doing that manually.

#### Dual or multiboot

If you want to preserve your existing system(s) and multi-boot the computer, you
will probably not want to reinitialize your partition layout.

In that case, you will need to look if you have free space (should be marked
`Apple_Free` when you print out the partition table in `pmac-fdisk`). If you do,
everything is good and you can just create a new bootstrap partition somewhere.
If you don't have available unused space, you will need to delete some other
partition, or shrink some existing filesystem to make more free space.

On installations with OS X, it seems to be a common occurence that there are
unused `Apple_Free` spaces sized about 128MB scattered around the disk. If that
is the case, that is a good place to make your bootstrap partition. OS X does not
need anything other than its own HFS+ partition, which is blessed and acts as its
own bootstrap.

Generally, it does not matter how the disk is layouted, as long as you have a
bootstrap partition somewhere and then another partition or a few for your rootfs
and possibly other things.

As an example, if you have an existing layout like this:

| Device    | Type                | Name      | Size | System             |
| --------- | ------------------- | --------- | ---- | ------------------ |
| /dev/sdX1 | Apple_partition_map | Apple     | -    | Partition map      |
| /dev/sdX2 | Apple_Free          |           | 128M | Free space         |
| /dev/sdX3 | Apple_HFS           | OS X      | 100G | HFS                |
| /dev/sdX4 | Apple_Free          |           | 128M | Free space         |
| /dev/sdX5 | Apple_HFS           | empty     | 50G  | HFS                |
| /dev/sdX6 | Apple_Free          |           | 8k   | Free space         |

In this context, `sdX3` is OS X, and `sdX5` is an empty HFS+ formatted partition
you want to install Void on. `sdX2` and `sdX4` are just unused gaps, as is `sdX6`.

You'd do something like this:

```
# pmac-fdisk /dev/sdX
b 2p                               # bootstrap partition, could also be 4p
d 5p                               # delete the empty Apple_HFS
c 4p 46G rootfs                    # root filesystem (/)
c 5p 5p swap                       # remaining (4G) unused space as swap
w
q
```

Note how the rootfs is `4p`; this is because deleting a partition inbetween two
free spaces merges them all together, so `sdX4`, `sdX5` and `sdX6` will become
just `sdX4`.

Other configurations will need equivalent changes.
