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

Unfortunately, I don't have a physical SLOF system available, so this section
could use expansion.

In a virtual machine, just specify the ISO image as a `cdrom` or a `drive` and
make `qemu` use it as the boot device. `GRUB` will automatically come up. Then
proceed with installation and so on.

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

## Other OpenFirmware machines

These are not tested but should still work, as long as the CPU is good enough
to run your variant. The instructions will likely overlap with those for SLOF
or for Macs. Feel free to submit modifications for this chapter.

## Other hardware

That's completely untested, so if you manage to get something to boot,
instructions to include here would be much appreciated.
