# Post-Installation

Once you have a working system, there are things you might want to do.

## Page poisoning and SLUB debug

By default, Void enables `slub_debug=P page_poison=1`. These are hardening
options and have little effect on modern hardware, but on old PowerPC Macs,
the performance hit may be significant.

Therefore, you might want to remove these from `/etc/default/grub` and then
run `update-grub`.

## Updates

You will definitely want to update your system, especially if the live media
is old. Void is a rolling distribution and therefore updates frequently. Run:

```
# xbps-install -Su
```

## NTP (time syncing)

You might want to enable a time syncing daemon. This is especially important
if your hardware can't keep clock. For example, you can do:

```
# xbps-install -S openntpd
# ln -s /etc/sv/ntpd /var/service/
```

There are other NTP daemons to choose from as well.

## Logging

By default, Void comes with no logging daemon. There are different implementations
available, `socklog` is simplistic and easy to use:

```
# xbps-install -S socklog-void
# ln -s /etc/sv/socklog-unix /var/service/
# ln -s /etc/sv/nanoklogd /var/service/
```

## PopCorn

If you feel like helping us take over usage statistics in
<https://popcorn.voidlinux.org>, install and enable `PopCorn`:

```
# xbps-install PopCorn
# ln -s /etc/sv/popcorn /var/service/
```

## Apple hardware

### Media keys and keyboard backlight on laptops

Install and enable `pbbuttonsd`:

```
# xbps-install pbbuttonsd
# ln -s /etc/sv/pbbuttonsd /var/service/
```

### Right click emulation

Install and enable `mouseemu`:

```
# xbps-install mouseemu
# ln -s /etc/sv/mouseemu /var/service/
```

Middle click defaults to F10, right click to F11. Scrolling modifier
defaults to Alt.

### Wireless networking

The `b43` driver is usually used. Unfortunately, the firmware for that is
not redistributable, and we don't have a `restricted` template yet. But it
is not very difficult to set up. First install `b43-fwcutter`:

```
# xbps-install b43-fwcutter
```

Then fetch the firmware:

```
$ xbps-uhelper fetch http://www.lwfinger.com/b43-firmware/broadcom-wl-6.30.163.46.tar.bz2
```

You're free to use any fetching tool you want.

Extract it:

```
$ tar xf broadcom-wl-*.tar.bz2
```

And finally use the cutter to extract the firmware:

```
# b43-fwcutter -w /usr/lib/firmware broadcom-wl-*.wl_apsta.o
```

This will make sure to place the firmware in the appropriate location. After
that, just reboot and wireless network should just work, but don't expect it
to be fast :)

## Other things

The official handbook at <https://docs.voidlinux.org> should come in handy.
