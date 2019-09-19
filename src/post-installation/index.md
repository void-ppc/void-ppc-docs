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

The packages are unfortunately only available for little endian right now,
as it's written in Go and we haven't yet set up Go `build-style` to build with
`gccgo` on platforms where official Go is not available.

## Other things

The official handbook at <https://docs.voidlinux.org> should come in handy.
