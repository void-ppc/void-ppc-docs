# Partitioning Notes

This is generally simple, but will differ depending on the hardware/firmware.

When using the installer, it will provide you with instructions appropriate
for your machine.

## OpenPOWER

Since OpenPOWER systems use the Petitboot stack, which runs in a simple Linux
system, you're free to choose any partitioning model that environment supports.
The `/boot` partition must be mountable from it. Usually this means a GPT with
an `ext4` /boot on it will be safe.

## SLOF

On pSeries, virtual machines and so on, the default recommended choice is a
MBR. On newer versions of SLOF, GPT will also work (and in virtual machines
it does), but MBR is a safe choice.

In order to boot, you will need to create a `PowerPC PReP Boot` partition,
sized around 10MB. If using MBR, you will also need to mark it bootable.

Example:

```
# fdisk /dev/sdX
o # MBR, g for GPT
n # new partition for PReP boot; make it 10 megabytes
t # change partition type to PowerPC PReP boot
n # new partition for /
a # set first partition bootable
w # write changes and quit
```

The partition type number for `PowerPC PReP boot` should be `7` for GPT and
`41` for MBR.

## PowerPC Macs

Macs use APM (Apple Partition Map), at least for booting. The actual / partition
can be on anything Linux supports.

You can use `pmac-fdisk` for APM partitioning. Standard `fdisk` does not support it.

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
| /dev/sdX2 | Apple_Bootstrap     | bootstrap | ~10M | NewWorld bootblock |
| /dev/sdX3 | Apple_UNIX_SVR2     | rootfs    | any  | Linux native       |
| /dev/sdX4 | Apple_UNIX_SVR2     | swap      | any  | Linux swap         |

You can create it for example like this:

```
# pmac-fdisk /dev/sdX
i                                    # init partition table, wipes all data
C 2p 10M bootstrap Apple_Bootstrap   # bootstrap partition
c 3p 120G rootfs                     # root filesystem (/)
c 4p 4p swap                         # swap partition
w
q
```

**This is only for clean installations! It will wipe anything else present
on the drive. If you don't want that, read below.**

In an APM, the first partition is always automatic, being the APM itself.

**Keep in mind that the bootstrap partition is not the `/boot` partition!** Unless
you make a separate one, `/boot` will be in your `/` partition. The only thing that
resides in the bootstrap partition is the bootloader, which even with all modules
will have a few megabytes at most. The 10MB size is actually very generous and
generally will not be used up.

### Dual or multiboot

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
C 2p 10M bootstrap Apple_Bootstrap # bootstrap partition, could also be 4p
d 5p                               # delete the empty Apple_HFS
c 4p 45G rootfs                    # root filesystem (/)
c 5p 5p swap                       # remaining space as swap
```

Note how the rootfs is `4p`; this is because deleting a partition inbetween two
free spaces merges them all together, so `sdX4`, `sdX5` and `sdX6` will become
just `sdX4`.

Other configurations will need equivalent changes.
