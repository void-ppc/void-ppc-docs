# Booting

This project is intended to boot on a variety of machines. They might all have
a slightly different boot process, so describing that requires a somewhat more
comprehensive document than a section in the installation instructions.

## Supported environments

Generally, the live media support three types of environments primarily.

1) **Bare metal OpenPOWER (PowerNV)** - This includes machines such as Raptor
   Talos 2 (or Blackbird) and various IBM (and non-IBM) OpenPOWER hardware using
   the OPAL stack. These systems can boot `ppc64le` and `ppc64` media and use
   Petitboot+`kexec` as their bootloader of choice.
2) **Non-OpenPOWER IBM servers based of Slimline OpenFirmware (SLOF)** - This
   includes various IBM Power Systems (pSeries) that do not use the OpenPOWER
   stack. Their firmware of choice is SLOF, which is a variant of OpenFirmware.
   GRUB, present on the live media, is used as the bootloader. This ia also the
   default kind of firmware you will get in virtualized `qemu` environments by
   default, so it's important for KVM. They can run `ppc64le` for POWER8-and-newer
   based stuff as well as `ppc64` for all of them.
3) **NewWorld PowerPC Macs** - This includes various G3, G4 and G5 based 32-bit
   and 64-bit Apple hardware. Only NewWorld hardware is supported for the live
   media and installer (i.e. G3 Blue and White and newer, as well as all G4 and
   G5; the old Beige Macintoshes can't directly boot Linux). This hardware is
   based on Apple's variant of OpenFirmware. Therefore, GRUB from the live
   media is the bootloader. They can run `ppc64` images (for G5) and `ppc`
   images (for G3, G4).

There may be more environments that should be able to boot. For example, the
*IBM IntelliStation POWER 185* is an IBM workstation based on OpenFirmware and
the PowerPC 970MP (IBM's variant of the G5) and has all the prerequisites for
being able to boot Void. Therefore, it most likely does, but it's not tested,
and may require workarounds (such as manually booting the GRUB image from the
OpenFirmware console).

## Bare metal OpenPOWER

Since the firmware of these systems is based on a small Linux system and it
uses `kexec` as the boot mechanism, that makes booting the live media very simple.

All you need to do is put the contents of the live image onto any storage media
the firmware can read (optical media, USB sticks, etc.) and the Petitboot
bootloader takes care of the rest.

Remove the USB stick, then insert it into your OpenPOWER system, and wait for
Petitboot to come up. The entries for live OS boot should show up. Proceed with
installation or whatever else you need.

## SLOF (virtual machines, pSeries, etc)

### Virtual Machines

In a virtual machine, just specify the ISO image as a `cdrom` or a `drive` and
make `qemu` use it as the boot device. `GRUB` will automatically come up. Then
proceed with installation and so on.

### IBM pSeries

To boot on physical pSeries machine, you need to define a `cd` or `ud`
device alias, since those machines doesn't seem to define it automatically.

#### Using optical media

If booting from optical media, we need to get the CD drive's OF path through
the SMS menu:

First, boot to the SMS menu (please check your machine documentation on how
to do it).

Then, go through those menu items:
1. Boot Options
2. Install/Boot Device
3. CD/DVD
4. List All Devices
5. Pick your CD drive here, for example it might be shown as
  `SATA CD-ROM (loc=U78A0.001.DNWKAM4-P2-D2)`.
6. Information

In the Information page, there will be a string that looks like
`/pci@XXXXXX/pciYYYY/sata/disk@ZZZZZZ`.
That's the OF path of your CD drive. Copy it somewhere else, we'll use it to set
up the `cd` device alias.

Now, exit SMS and reboot to the OF prompt.

On the prompt, create a device alias for `cd` using the path we got from SMS.
```
0 > devalias cd /pci@XXXXXX/pciYYYY/sata/disk@ZZZZZZ
```

Print the alias listing again to check if it's correctly defined:
```
0 > devalias
...
...
cd          /pci@XXXXXX/pciYYYY/sata/disk@ZZZZZZ
```

Now that we've defined the `cd` device alias, it's time to boot into GRUB:
```
0 > boot cd:,\boot\grub.img
```

Now you can choose the menu item you want, boot into it and proceed with
installation.

#### Using a USB disk

If booting from USB disk, you can boot directly to the OF prompt since it's
possible get the USB disk's OF path from there.

Once you get into the OF prompt, list the whole device tree:
```
0 > dev / ls
```

It'll then list all the devices recognized by the firmware. For example here's
a simplified listing on a Power 750:
```
XXXXXXXXXXXX: ...
XXXXXXXXXXXX: ...
XXXXXXXXXXXX: ...
XXXXXXXXXXXX: /pci@800000020000201
XXXXXXXXXXXX:   /usb@1
XXXXXXXXXXXX:     /...
XXXXXXXXXXXX:   /usb@1,1
XXXXXXXXXXXX:     /hub@1
XXXXXXXXXXXX:       /usb-scsi@1
XXXXXXXXXXXX:         /disk
XXXXXXXXXXXX:         /tape
XXXXXXXXXXXX:   /usb@1,2
XXXXXXXXXXXX:     /...
XXXXXXXXXXXX: ...
XXXXXXXXXXXX: ...
XXXXXXXXXXXX: ...
```
We can see that there's a USB disk `/disk` under `/usb-scsi@1`.

Now, create a device alias for `ud` using that path. Of course, do adjust
the command using the path from your own machine.
```
0 > devalias ud /pci@800000020000201/usb@1,1/hub@1/usb-scsi@1/disk
```

Print the alias listing again to check if it's correctly defined:
```
0 > devalias
...
...
ud          /pci@800000020000201/usb@1,1/hub@1/usb-scsi@1/disk
```

Now that we've defined the `ud` device alias, it's time to boot into GRUB:
```
0 > boot cd:,\boot\grub.img
```

In GRUB, there's some adjustment to be made before we can boot successfully to
the live environment when booting through a USB disk. Select the menu item you
want, then press `E` to edit it. Now, do the following edits:
1. Delete the `insmod part_apple` line.
2. Delete the `search --label "VOID_LIVE"` line.
3. In the `linux` line, find the part that says `root=live:CDLABEL=VOID_LIVE`
   and change it into `root=live:LABEL=VOID_LIVE` (that is, remove the `CD`
   from `CDLABEL`).

When you're done with the edits, press Ctrl-x to boot into the live environment
and proceed with the installation.

#### Additional notes

Note that regardless of the method you choose, the live CD might take a long
time (up to twenty minutes) to fully boot, so don't worry if the boot process
appears to be stalling. Once installed, the slow booting problem seems to
disappear, though.

Also, on some machines, the main interactive console uses the `hvsi0` console,
so you may need to tell the kernel to use it; edit the menu item you want to
boot and append `console=hvsi0` to the `linux` line.

(See also the [Serial Console](#serial-console) section)

## NewWorld PowerPC Macs

If booting from optical media, this is straightforward; all you need to do is
insert the media and boot from it in the usual manner (for example, using the
boot device chooser).

Booting from USB is also possible on *any* NewWorld Mac, but may be slightly
more tricky.

So if you want to boot from USB, insert your USB stick in your Mac, then power
it on and **hold the Command + Option + O + F** combination. On standard
non-Apple keyboards, this is **Win + Alt + O + F**.
**Keep holding the combination until your display comes up.**

```
Release keys to continue!
```

This should be written on the screen, so release the combination. You will
get a prompt:

```
 ok
0 >
```

### G5 machines

The good news is, the G5s define the `ud` device alias, which matches the USB
storage media you have inserted, at least as long as only one USB stick is
inserted.

You can simply boot your Void like this:

```
boot ud:,\\:tbxi
```

For multiple USB media, it should be possible to use numbered `ud:N`. You can
list all the defined aliases with the `devalias` command.

If this doesn't work, you can try to boot the GRUB image directly.

```
boot ud:,\boot\grub.img
```

A GRUB menu should come up.

### G4/G3 or if it doesn't work

This is slightly trickier. Since the alias is not defined, we must create it.
First, list the whole device tree:

```
0 > dev / ls
```

This will present you with a long listing, most likely also telling you to press
Space for more, as the whole listing does not fit on the screen.

A simplified listing on my PowerBook G4 looks like this:

```
...
ffXXXXXX: ...
ffXXXXXX: ...
ffXXXXXX:  /pci@f2000000
ffXXXXXX:    /...
ffXXXXXX:      /...
ffXXXXXX:      /...
ffXXXXXX:    /usb@1a
ffXXXXXX:      /device@1
ffXXXXXX:        /keyboard@0
ffXXXXXX:        /mouse@1
ffXXXXXX:      /device@2
ffXXXXXX:        /keyboard@0
ffXXXXXX:        /mouse@1
ffXXXXXX:        /interface@2
ffXXXXXX:    /usb@1b
ffXXXXXX:      /disk@1
ffXXXXXX:    /...
ffXXXXXX:    /...
...
```

And so on. This basically represents the tree of all the devices attached in
your system. We are looking for USB, and within that, we are looking for a USB disk.

In this case, you can see it under `/usb@1b` as `/disk@1`. Depending on the
hardware as the USB port you use, this may look different.

Anyway, we've found the USB disk. Let's alias it as `ud`.

```
0 > devalias ud /pci@f2000000/usb@1b/disk@1
```

This should print `ok`. Obviously adjust the values for your own device tree,
you just need to join it all together.

Print the alias listing again:

```
0 > devalias
...
...
ud        /pci@f2000000/usb@1b/disk@1
```

Now that you can see it's there, proceed with booting:

```
boot ud:,\\:tbxi
```

If that doesn't work, try loading the GRUB image directly:

```
boot ud:,\boot\grub.img
```

A GRUB menu should come up.

### Post-GRUB

Once you have selected your option in the bootloader, loading Linux will
commence. You might see some messages like:

```
error: can't open device
Press any key to continue...
```

You don't need to press anything and these errors are harmless. Just wait (it
might take a minute or a few) and eventually Linux should load.

Proceed with installation and so on.

### Yaboot

If GRUB for some reason won't work and there are no available workarounds,
we also ship `yaboot` as a fallback. You can boot it from the OpenFirmware
console as well. Just replace this:

```
boot ud:,\\:tbxi
```

with:

```
boot ud:,\boot\yaboot conf=ud:,\etc\yaboot.conf
```

The `conf` argument may not always be necessary but generally is, as it will
otherwise often try to use incorrect media to look for the config file.

## Other OpenFirmware machines

These are not tested but should still work, as long as the CPU is good enough
to run your variant. The instructions will likely overlap with those for SLOF
or for Macs. Feel free to submit modifications for this chapter.

## Serial console

By default, the system will boot assuming output on your monitor. If you don't
have a monitor, or for some other reason need to access the system via the
serial port, you need to enable it.

Just proceed booting as usual, and once at the `GRUB` screen, edit the menu
item you want to boot and append something like this:

```
console=tty0 console=hvc0
```

This applies for machines such as the Talos 2, Blackbird or `qemu` `pSeries`
virtual machines. There is a special initramfs hook which makes sure to enable
the appropriate `agetty` service for the console. The supported values are
`ttyS0`, `hvc0` and `hvsi0`. If you don't need output on your monitor at all,
you can skip specifying `console=tty0`. But keep in mind that for the hook to
work, the `console` for the serial needs to be last! Otherwise the hook will
not pick it up.

## Other hardware

That's completely untested, so if you manage to get something to boot,
instructions to include here would be much appreciated.
