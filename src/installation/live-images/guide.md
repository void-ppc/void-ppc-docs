# Installation Guide

Once you have [downloaded](./downloading.md) a Void image to install and
[prepared](./prep.md) your install media, you are ready to install Void Linux.

## Booting

Boot your machine from the install media you created. If you have enough RAM,
there is an option on the boot screen to load the entire image into ram, which
will take some time but speed up the rest of the install process.

Once the live image has booted, log in as `root` with password `voidlinux`.
Then, check if date/time is correct, with the `date` command. Especially
on old Macs with bad batteries and in some VMs it may be set to 1970. If
that is the case, fix it again with the `date` command.

Afterwards, run:

```
# void-installer
```

The following sections will detail each screen of the installer.

## Keyboard

Select the keymap for your keyboard; standard "qwerty" keyboards will generally
use the "us" keymap.

## Network

Select your primary network interface. If you do not choose to use DHCP, you
will be prompted to provide an IP address, gateway, and DNS servers.

If you intend to use a wireless connection during the installation, you may need
to configure it manually using wpa_supplicant and dhcpcd manually before running
`void-installer`.

## Source

To install packages provided on the install image, select `Local`. Otherwise,
you may select `Network` to download the latest packages from the Void
repository.

> Note: if you are installing a desktop environment from a ''flavor'' image, you
> MUST choose `Local` for the source!

## Hostname

Select a hostname for your computer (that is all lowercase, with no spaces.)

## Locale

Select your default locale settings. This option is for glibc only, as musl does
not currently support locales.

## Timezone

Select your timezone based on standard timezone options.

## Root password

Enter and confirm your `root` password for the new installation. The password
will not be shown on screen.

## User account

Choose a login (default `void`) and a descriptive name for that login. Then
enter and confirm the password for the new user. You will then be prompted to
verify the groups for this new user. They are added to the `wheel` group by
default and will have `sudo` access.

## Partition

Next, you will need to partition your disks. Void does not provide a preset
partition scheme, so you will need to create your partitions manually.
Depending on your platform, the installer will guide you with instructions
on how to partition your drive. You might need to use different partitioning
tools depending on the hardware (`cfdisk` for most, `pmac-fdisk` for Macs).

See the [Partitioning Notes](./partitions.md) for more details about
partitioning your disk.

## Filesystems

Create the filesystems for each partition you have created. For each partition
you will be prompted to choose a filesystem type, whether you want to create a
new filesystem on the partition, and a mount point, if applicable. When you are
finished, select `Done` to return to the main menu.

## Bootloader

### OpenPOWER systems

Since OpenPOWER systems use Petitboot, you will not be able to install a
bootloader. The installer will still ask you whether to at least use GRUB
for generating the configuration file. As Petitboot can read these configs,
it can generate a menu for you. You will probably want to say Yes.

### IBM OF systems

These use the `PReP boot` partition. You will need to point the installer to
it so that it can put GRUB in there. You will also be asked whether to use
a graphical terminal. That's generally up to you, in either case you will
get a boot menu.

### NewWorld PowerPC Macs

You will need to select your bootstrap partition. The choice of graphical
bootloader is again up to you, but keep in mind that it might be slow on
this hardware, so you will likely want to say No.

### Other hardware

Other hardware is generally unsupported by the installer when it comes to
bootloader setup, though you can still use it for other things and skip the
bootloader installation, then do it manually.

## Review settings

It is a good idea to review your settings before proceeding. Use the right arrow
key to select the settings button and hit `<enter>`. All your selections will be
shown for review.

## Install

Selecting `Install` from the menu will start the installer. The installer will
create all the filesystems selected, and install the base system packages. It
will then generate an initramfs and install a GRUB2 bootloader to the bootable
partition.

These steps will all run automatically, and after the installation is completed
successfully, you can reboot into your new Void Linux install!

## Post installation

See the [Post-Installation](../../configuration/post-installation/index.md)
guide for some tips on setting up your new system.
