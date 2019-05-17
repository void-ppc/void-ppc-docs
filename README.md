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

| Target       | Min. CPU requirement    | Notes                                       |
| ------------ | ----------------------- | ------------------------------------------- |
| ppc64le      | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64le-musl | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64        | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |
| ppc64-musl   | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |

The typical expected little endian target is the Raptor Talos 2, but any POWER8 or better system will work (but the NXP e6500 will not, as it lacks little endian altivec; on big endian it will work). The typical expected big endian target is older hardware such as Power Mac G5, but it will also work on modern POWER8 and newer hardware. Note that hardware without AltiVec support (e.g. the NXP `e5500` SoCs) is not supported and will not work. All targets are tuned for POWER9.

**All targets, including BE glibc and musl, use the ELFv2 ABI.** The `musl` libc always uses ELFv2 regardless of endianness, glibc uses ELFv2 by default on little endian on all distros but big endian distros typically default to the older ELFv1 ABI; this is for legacy and compatibility reasons, but ELFv2 has other benefits and we have no legacy support, therefore we're targeting ELFv2 on BE glibc as one of the first distributions to do so.

### Target status

#### ppc64le glibc

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-gnu`
- [x] `linux`
- [x] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [x] `rust`
- [x] `go` (`gcc-go` native bootstrap)
- [x] `java` (binary bootstrap from AdoptOpenJDK)
- [x] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64le musl

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-musl`
- [x] `linux`
- [x] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [x] `rust` (native and cross)
- [ ] `go` (cross-libc same-arch is broken)
- [ ] `java`
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64 glibc

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64-linux-gnu`
- [x] `linux`
- [x] graphical environment (only xorg + dependencies, wayland core, some minimal WMs)
- [x] `rust`
- [ ] `go` (unsupported on ELFv2 BE, also power8+ only)
- [ ] `java` (binary bootstrap from AdoptOpenJDK)
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc64 musl

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-musl`
- [x] `linux`
- [ ] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [x] `rust` (native and cross)
- [ ] `go` (unsupported on ELFv2 BE, also power8+ only)
- [ ] `java`
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

## Project stages

**Current: continuous merging, most changes upstream**

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

- xbps repository (`ppc64le` direct): https://repo.void-ppc64.octaforge.org/current
- xbps repository (`ppc64le-musl` direct): https://repo.void-ppc64.octaforge.org/current/musl
- xbps repository (`ppc64` direct): https://repo.void-ppc64.octaforge.org/current/be
- xbps repository (`ppc64-musl` direct): https://repo.void-ppc64.octaforge.org/current/be/musl
- xbps repository (`ppc64le` load balancing): https://auto.void-ppc64.octaforge.org/current
- xbps repository (`ppc64le-musl` load balancing): https://auto.void-ppc64.octaforge.org/current/musl
- xbps repository (`ppc64le` load balancing): https://auto.void-ppc64.octaforge.org/current/be
- xbps repository (`ppc64-musl` load balancing): https://auto.void-ppc64.octaforge.org/current/be/musl
- static xbps for both endians (built using musl): https://void-ppc64.octaforge.org/static
- rsync for mirroring: `rsync://octaforge.org/void-ppc64`

The load balancing variants will try to use different mirrors and are default for our `xbps` packages and so on.

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

**Q:** Will multilib be supported?  
**A:** No, it is not planned. However, the compiler is built as bi-arch, which means `-m32` works, and you can have it emit 32-bit code. This is useful for low level stuff (e.g. GRUB, which needs to emit 32-bit big endian code independent on a libc) while not burdening the higher level infrastructure. If you really need to build or use 32-bit software, use a 32-bit chroot, it should work just fine.

**Q:** I'm using big endian with an ASpeed VGA and colors are broken.  
**A:** This is a problem with the kernel `ast` driver. Since the fix appears to be non-trivial and there is no proper patch available, this is WONTFIX from our side. Either use a dedicated GPU (ideally PCIe, USB2 DisplayLink is known to work) or use little endian if you really need the `ast` to work properly.

**Q:** How do I install this?  
**A:** https://github.com/void-ppc64/documentation/blob/master/INSTALL.md

If you have any questions, suggestions or anything else, I'm on IRC (Freenode: `#talos-workstation`, `#voidlinux`, `#xbps` and others) as well as on Twitter (`@octaforge`).
