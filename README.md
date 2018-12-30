# Void Linux ppc64

This is the Void Linux porting project for 64-bit PowerPC/POWER systems. Keep in mind that **it is unofficial at this point**.

## Rationale

Void currently has no support for ppc64. Since ppc64 is becoming more relevant again with high performance systems such as Raptor Talos 2, it is desirable to have support for it in the distribution; this project's aim is to become a staging area for all porting work before it makes it upstream in Void Linux.

Until there are official builds, the project will continue to provide certain non-mergeable modifications into the repositories for smoothest user experience, such as builds of `xbps` packages with the default repo location pointing to the unofficial one (as well as providing the correct unofficial signing key out of box) in order to avoid having to manually configure things.

## Supported targets

This project primarily aims to support modern, high performance systems, without a particular focus on old hardware. That does not mean it is unsupported, however. There are three targets the project aims to support.

- Little endian glibc (`ppc64le`)
- Little endian musl (`ppc64le-musl`)
- Big endian musl (`ppc64-musl`)

The first two, being little endian, are only meant to work on POWER8 and higher, and are the primary targets, because of smoothest software support (least likely to encounter issues getting it to build or run). Big endian `musl` target should work on all POWER4 and newer systems, such as PowerPC G5/970 present on Power Macs. 32-bit PowerPC is out of scope of the project, and is already being done elsewhere.

### Target status

#### ppc64le glibc

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [ ] `cross-powerpc64le-linux-gnu` (made, pending testing on an x86 setup)
- [x] `linux`
- [x] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [x] `rust`
- [x] `go`
- [ ] `java`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64le musl

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-musl`
- [x] `linux`
- [ ] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [ ] `rust`
- [ ] `go`
- [ ] `java`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64 musl

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-musl`
- [x] `linux`
- [ ] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [ ] `rust`
- [ ] `go`
- [ ] `java`
- [ ] installer images
- [ ] rootfs tarballs

## Project stages

**Current: stage1** (initial PR being prepared)

### Stage 1

This is the state where no platform specific work is held in upstream Void Linux repositories and therefore all changes are downstream in `void-ppc64`. The goal is to move on as soon as possible.

At this point, the project maintains an unofficial binary repository and eventually ISO and rootfs builds. The unofficial repository is fully working and best covers the `ppc64le` target but others are also being built.

### Stage 2

At this point, changes will start making it into upstream `void-packages` and potentially other repositories. At this point, Void still won't provide any official packages. This project will act as a staging area for new changes, which will get submitted into upstream as soon as they reach mergeable status.

This project will still provide a binary repository of packages, snapshots etc. for anyone's use.

### Stage 3

At this point Void Linux should start providing binary packages, which is the ultimate goal; it is yet unsure what path will be taken as cross-compilation is often impossible or results in missing features. It is unsure when the upstream project will do this, so `void-ppc64` is ready to provide continuously updated binary builds indefinitely, built on native hardware rather than through the means of cross-compilation (which means fully featured packages). The project will continue acting as a staging area for new commits to make it upstream.

The binary repository will transform into an overlay repository containing testing packages.

## Package/build mirrors

The main project binary location is https://void-ppc64.octaforge.org.

- xbps repository (`ppc64le`): https://void-ppc64.octaforge.org/current
- xbps repository (`ppc64le-musl`, `ppc64-musl`): https://void-ppc64.octaforge.org/current/musl
- static xbps for both endians (built using musl): https://void-ppc64.octaforge.org/static
- rsync for mirroring: `rsync://octaforge.org/void-ppc64`

### Mirror list

Since my server space is not free, mirrors are always appreciated! Let me know if you set up a mirror. Use `rsync`, the URL is provided above.

Current list:

- https://mirrors.servercentral.com/voidlinux-ppc64/ (provided by `zdykstra` of the Void Linux as well as the Talos community, also hosts a Void mirror)
- https://ppc.exqa.de/ (provided by `Cogitri` of the Void Linux community)

Big thanks to all the mirror providers, as my bandwidth is limited and shared with my other projects.

## Repository structure

_Note: this is not yet in place, is more of a proposal that is yet to be implemented_

The primary area where porting happens is the `void-packages` repository, which mirrors the structure of the upstream template database. It consists of several branches:

- `master` - this branch has its consistency ensured, i.e. force pushes are forbidden and it's always pullable; it consists of the upstream `master` branch plus non-mergeable changes of the `void-ppc64` project, e.g. the `xbps` updates; it is not useful until the project has entered stage 2, since it may not contain all of the necessary porting work
- `staging` - this contains the latest porting work meant for submission into upstream; it does not contain any non-mergeable changes and does not ensure consistency, i.e. it may get rebased at any time
- `testing` - this is essentially a combination of the `master` and `staging` branches, it contains everything from `staging` but it also ensures consistency, i.e. it's always pullable, and also contains the non-mergeable changes from `master`
- any other branch is typically temporary, containing a pull request or experiments

## Contributing

If you wish to contribute into the project, you should typically want to base your changes on top of the `testing` branch and then submit a pull request.

Once the change is good and has been merged into `testing`, it will great cleaned up and cherry-picked (or otherwise merged) into `staging` and will go through the usual rebasing process, before being submitted as an upstream pull request.

Of course, once the project has reached stage 2 (i.e. at least initial porting work has made it upstream), you will also be able to submit your changes into Void upstream directly. This is the preferred way; in that case, it's best to mention me (`q66`) in the pull request so I can keep track and potentially review the changeset. Such changes will get pulled back into `void-ppc64` from upstream.
