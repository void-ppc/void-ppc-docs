# Apple Hardware

## Media keys and keyboard backlight on laptops

Install and enable `pbbuttonsd`:

```
# xbps-install pbbuttonsd
# ln -s /etc/sv/pbbuttonsd /var/service/
```

## Right click emulation

Install and enable `mouseemu`:

```
# xbps-install mouseemu
# ln -s /etc/sv/mouseemu /var/service/
```

Middle click defaults to F10, right click to F11. Scrolling modifier
defaults to Alt.

## Wireless networking

The `b43` driver is usually used. Unfortunately, the firmware for that is
not redistributable. Our templates collection ships some templates which you
can use to build your own firmware packages.

### Using void-packages

You will need to set up `void-packages`. Follow the standard instructions,
using our `void-ppc` fork. The condensed version would be:

```
# xbps-install base-devel git
$ git clone https://github.com/void-ppc/void-packages.git
$ cd void-packages
$ ./xbps-src binary-bootstrap
```

Follow the official documentation for `xbps-src` usage for more information.

Enable `restricted` packages:

```
$ echo XBPS_ALLOW_RESTRICTED=yes >> etc/conf
```

Then build the appropriate firmware package:

```
$ ./xbps-src pkg b43-firmware
```

or:

```
$ ./xbps-src pkg b43-firmware-classic
```

Whether you should use `b43-firmware` (version `6.x.x.x`) or `b43-firmware-classic`
(version `5.x.x`) depends on the wireless card you have. First, find out which
one it is:

```
$ lspci | grep Wireless
```

The output may be something like (this is from a 2005 PowerBook G4 15"):

```
0001:10:12.0 Network controller: Broadcom Inc. and subsidiaries BCM4306 802.11b/g Wireless LAN Controller (rev 03)
```

If you have one of BCM4306 rev.3 (this is the above), BCM4311, BCM4312 or
BCM4318 rev.2, you should use `b43-firmware-classic`. If you have a BCM4331,
you should use `b43-firmware`. In other cases, you should probably be able to
use either.

Install the firmware:

```
# xbps-install -R hostdir/binpkgs/nonfree b43-firmware
```

or:

```
# xbps-install -R hostdir/binpkgs/nonfree b43-firmware-classic
```

If one doesn't work for you, try the other.

### Using b43-fwcutter manually

If you don't want to clone the `void-packages` repository for some reason,
you can always set it up manually. First, read the section above anyway; it
contains useful information about compatibility. Then install `b43-fwcutter`:

```
# xbps-install b43-fwcutter
```

Make a dedicated directory:

```
$ mkdir broadcom_fw && cd broadcom_fw
```

Then fetch the firmware. This is for `b43-firmware`:

```
$ xbps-uhelper fetch http://www.lwfinger.com/b43-firmware/broadcom-wl-6.30.163.46.tar.bz2
```

or for `b43-firmware-classic`:

```
$ xbps-uhelper fetch http://www.lwfinger.com/b43-firmware/broadcom-wl-5.100.138.tar.bz2
```

You're free to use any other tool you want to fetch it (`wget`, `curl`, etc).

Extract it:

```
$ tar xf broadcom-wl-*.tar.bz2
```

And finally use the cutter to extract the firmware. For `b43-firmware`:

```
# b43-fwcutter -w /usr/lib/firmware broadcom-wl-*.wl_apsta.o
```

Or for `b43-firmware-classic`:

```
# b43-fwcutter -w /usr/lib/firmware linux/wl_apsta.o
```

This will make sure to place the firmware in the appropriate location. After
that, just reboot and wireless network should just work, but don't expect it
to be fast :)

If you need to remove it later, just

```
# rm -rf /usr/lib/firmware/b43
```

Particularly you will need to do that when switching versions, as you should
not install two conflicting versions at the same time.

## Audio

By default, it might seem like audio "doesn't work". This is not actually true,
it's just that PCM is muted by default.

To remedy this, install `alsa-utils`:

```
# xbps-install alsa-utils
```

The open `alsamixer`. Press the F6 key to switch the card to something like
`SoundByLayout`; if you're using plain ALSA, you might not have to switch
anything, but PulseAudio will show its own mixer first.

Then once you see the `PCM` slider (you might have to scroll a little to the
right), up its level, it'll probably be at 0 by default. Don't up it too much,
or you will introduce distortion; it seems 80 is the maximum safe value.

Audio should work afterwards and you can change the volume using the `Master`
slider or using PulseAudio or whichever other solution you like. Don't get
confused by there being just one output instead of separate ones for
headphones and speakers; automatic jack sensing works and it will switch
depending on if there's anything plugged in.
