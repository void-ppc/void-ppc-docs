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
i                                    # init partition table
C 2p 10M bootstrap Apple_Bootstrap   # bootstrap partition
c 3p 120G rootfs                     # root filesystem (/)
c 4p 4p swap                         # swap partition
w
q
```

In an APM, the first partition is always automatic, being the APM itself.
