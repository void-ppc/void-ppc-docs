# Void Linux ppc64

This is the Void Linux porting project for 64-bit PowerPC/POWER systems. Keep in mind that **it is unofficial at this point**.

## Rationale

Void currently has no support for ppc64. Since ppc64 is becoming more relevant again with high performance systems such as Raptor Talos 2, it is desirable to have support for it in the distribution; this project's aim is to become a staging area for all porting work, with the intention of submitting a majority of the patches upstream.

There are several types of patches in the repository:

1) Patches intended for upstream submission. This makes up the majority of patches in the repository and it is assumed that they will be submitted upstream as soon as possible.
2) Patches adding functionality that is meant for eventual upstream submission, but not in its current form. These are typically things that there is some kind of disagreement about design-wise, or simply experimental patches or proof of concept patches.
3) Patches not meant for upstream submission. These will likely never make it into upstream repos and tend to include things such as updates to repo URLs, `xbps-src` itself or other things specific to `void-ppc64` but not the upstream project.

There is another goal to this project, and that is to enhance building of packages on native `ppc64` machines as well as improve cross compilation from native `ppc64` machines to other targets, such as `aarch64` or `x86_64`. While this is also intended for upstream submission, it is much less useful for upstream at this point and might be met with different concerns.

The project will continue to provide its own repos even after majority of the changes are upstreamed and official builds are made. The reason is that it is unlikely that the upstream project will be making native builds, and cross compilation is not always possible or results in features missing from the final builds; `void-ppc64` wants to provide a repository of the same quality as `x86_64` repositories (with the exception of unportable or proprietary software).

Additionally, smooth user experience is important for `void-ppc64`, so the default `xbps` repo locations as well as signing keys will be updated for the provided packages. Installer and `rootfs` builds will also be provided by the project, as well as static `xbps` and other pieces of infrastructure.

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
- [x] `go` (using gcc-go)
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
- [ ] `rust` (waiting for 1.32)
- [x] `go` (cross only)
- [ ] `java`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64 musl

Same as `ppc64le-musl`. Any potential differences will be mentioned.

## Project stages

**Current: initial support merged (good enough for base-voidstrap)**

The project will provide a binary repository during all stages. See the rationale section for how that is intended to be done.

### ~~Stage 1~~

~~This is the state where no platform specific work is held in upstream Void Linux repositories and therefore all changes are downstream in `void-ppc64`. The goal is to move on as soon as possible.~~

### Stage 2

At this point, changes will start making it into upstream `void-packages` and potentially other repositories. At this point, Void still won't provide any official packages. This project will act as a staging area for new changes, which will get submitted into upstream as soon as they reach mergeable status, or held downstream when not intended for merge.

### Stage 3

At this point Void Linux should start providing binary packages, which is the ultimate goal; it is yet unsure what path will be taken as cross-compilation is often impossible or results in missing features. It is unsure when the upstream project will do this, so `void-ppc64` is ready to provide continuously updated binary builds indefinitely, built on native hardware rather than through the means of cross-compilation (which means fully featured packages). The project will continue acting as a staging area for new commits to make it upstream.

### Stage 4

This is the point when `void-ppc64` stops being useful. It is unlikely when or if this will happen; at this point the binary repository will become a simple overlay of testing changes.

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

The primary area where porting happens is the `void-packages` repository, which mirrors the structure of the upstream template database. It consists of two primary branches:

- `master` - this branch has its consistency ensured, i.e. force pushes are forbidden and it's always pullable; it contains all of the changes, but not necessarily in the correct order or with a pretty `git` history; changes may be split between different commits. It is meant primarily for users and contributors.
- `staging` - same contents as `master`, but without ensuring consistency; this branch is rebased and force pushed into as much as is necessary, and commits are squashed, edited and reordered as is seen fit. All pull request branches are made from this one.
- any other branch is typically temporary, containing a pull request or experiments.

Both primary branches contain all types of changes as described in the Rationale section. Individual changes or commit ranges are cherry picked into temporary branches for submission.

## Contributing

If you wish to contribute into the project, you should typically want to base your changes on top of the `master` branch and then submit a pull request. That is because the `staging` branch changes history very often and it would mean breaking all pull requests made from it continuously; changes made over `master` can be easily rebased.

Once the change is good and has been merged into `master`, it will get cleaned up and cherry-picked (or otherwise merged) into `staging` and will go through the usual rebasing process, before being submitted as an upstream pull request.

Of course, once the project has reached stage 2 (i.e. at least initial porting work has made it upstream), you will also be able to submit your changes into Void upstream directly. This is the preferred way; in that case, it's best to mention me (`q66`) in the pull request so I can keep track and potentially review the changeset. Such changes will get pulled back into `void-ppc64` from upstream.

## FAQ

**Q:** Why is there no big endian `glibc` target?  
**A:** The project only aims to support targets supporting the ELFv2 ABI. While the ABI is endian independent, only `musl` uses it for big endian. Big endian `glibc` `ppc64` use the old ELFv1 ABI instead. Introducing an ELFv1 target would mean having to preserve backwards compatibility later, so it is explicitly omitted.

**Q:** Will multilib be supported?  
**A:** For now it is not planned; it might be investigated at some later point, but since there is fairly little practical binary only software for `ppc`/`ppc64`, it is not a priority. If you do have such software, you can likely easily use a `chroot`.

**Q:** What hardware is supported?
**A:** For little endian targets, at least POWER8 is necessary, while for big endian `musl`, the minimum requirement is PowerPC 970 aka G5, which is a derivative of POWER4. All packages are built with AltiVec enabled and tuned for POWER9.
