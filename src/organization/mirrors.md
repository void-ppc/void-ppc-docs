# Sources And Mirrors

There are several mirrors to help lighten the load on the primary infrastructure.

Tier 1 mirrors sync directly from the primary and contain everything. They are
also required to provide https. Other mirrors may sync from somewhere else and
are allowed to host specific things only.

To change your mirrors to use a different set, you must create files in
`/etc/xbps.d` with the same names as those in `/usr/share/xbps.d`. Once you
have created such files, replace the URL with one of the servers below. Only
the files containing ‘repository’ in the filename need to be duplicated to
`/etc/xbps.d/`.

The default is <https://auto.voidlinux-ppc.org>. This is supposed to be
load-balancing between available Tier 1 mirrors, but that currently does not
work, so it’s equivalent to the primary for the time being.

## Tier 1 Mirrors

- <https://repo.voidlinux-ppc.org> (EU: Czech Republic)
- <https://mirrors.servercentral.com/void-ppc/> (USA: Chicago)
- <https://ppc.exqa.de>

## Tier 2 Mirrors

The are currently no Tier 2 mirrors. If you want to provide a mirror of either
kind, please contact us.

## Package repositories

The main project binary location is <https://repo.voidlinux-ppc.org>.

- xbps repository (`ppc64le` direct): <https://repo.voidlinux-ppc.org/current>
- xbps repository (`ppc64le-musl` direct): <https://repo.voidlinux-ppc.org/current/musl>
- xbps repository (`ppc64` direct): <https://repo.voidlinux-ppc.org/current/be>
- xbps repository (`ppc64-musl` direct): <https://repo.voidlinux-ppc.org/current/be/musl>
- xbps repository (`ppc` direct): <https://repo.voidlinux-ppc.org/current/ppc>
- xbps repository (`ppc-musl` direct): <https://repo.voidlinux-ppc.org/current/ppc/musl>
- xbps repository (`ppc64le` load balancing): <https://auto.voidlinux-ppc.org/current>
- xbps repository (`ppc64le-musl` load balancing): <https://auto.voidlinux-ppc.org/current/musl>
- xbps repository (`ppc64` load balancing): <https://auto.voidlinux-ppc.org/current/be>
- xbps repository (`ppc64-musl` load balancing): <https://auto.voidlinux-ppc.org/current/be/musl>
- xbps repository (`ppc` load balancing): <https://auto.voidlinux-ppc.org/current/ppc>
- xbps repository (`ppc-musl` load balancing): <https://auto.voidlinux-ppc.org/current/ppc/musl>
- static xbps for all (built using musl): <https://repo.voidlinux-ppc.org/static>
- rsync for mirroring: `rsync://voidlinux-ppc.org/void-ppc`

## Live images and rootfs tarballs

The primary location is <https://repo.voidlinux-ppc.org/live/> and contains
regularly updated ISO images (at very least for base system for every supported
target) and rootfs tarballs (`base-voidstrap` for each supported target, useful
for containers and so on). By extension, every mirror below also provides those
images.
