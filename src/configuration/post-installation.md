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

If there is a package manager upgrade (libxbps, xbps)
it will be done before upgrading any other packages,
because in case the transaction fails at least the package manager
needs to stay intact.
In that case, you’ll need to run `xbps-install -Su` again
to get all the upgrades.

## NTP (time syncing)

You might want to enable a time syncing daemon. This is especially important
if your hardware can't keep clock. For example, you can do:

```
# xbps-install -S openntpd
# ln -s /etc/sv/ntpd /var/service/
```

There are other NTP daemons to choose from as well.

Keep in mind that `openntpd` by default uses the https constraint feature. That
means that unless your time is set to a correct value in the first place, the
daemon will fail to set the date/time. Either use `date` to manually set a
close enough date/time, or remove/comment out the `constraints from` line in
`/etc/ntpd.conf`.

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

## Other things

The official handbook at <https://docs.voidlinux.org> should come in handy.
