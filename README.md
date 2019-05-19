# Void Linux for Power architecture

This is the Void Linux porting project for the Power architecture and PowerPC, both 32 and 64-bit. Keep in mind that **it is unofficial at this point**.

## Rationale

Void currently has no official support for the architecture. There are two reasons for this project; one is general availability of high performance Power hardware (such as the Raptor Talos 2/Blackbird etc.), the other is enabling Void Linux to run on older PowerPC hardware, such as iBooks/PowerBooks, Power Macs and so on; this project's aim is to become a staging area for all porting work, with the intention of submitting a majority of the patches upstream.

There are several types of patches in the repository:

1) Patches intended for upstream submission. This makes up the majority of patches in the repository and it is assumed that they will be submitted upstream as soon as possible.
2) Patches adding functionality that is meant for eventual upstream submission, but not in its current form. These are typically things that there is some kind of disagreement about design-wise, or simply experimental patches or proof of concept patches.
3) Patches not meant for upstream submission. These will likely never make it into upstream repos and tend to include things such as updates to repo URLs, `xbps-src` itself or other things specific to `void-ppc64` but not the upstream project.

There is another goal to this project, and that is to enhance building of packages on native machines as well as improve cross compilation from native machines (particularly the high performance ones) to other targets, such as `aarch64` or `x86_64`. While this is also intended for upstream submission, it is much less useful for upstream at this point and might be met with different concerns.

The project will continue to provide its own repos even after majority of the changes are upstreamed. The longer-term goal is to have official, natively built (no cross compilation) packages in Void Linux itself; if this is not possible, the project will continue providing the repos indefinitely, as quality equivalent to the `x86_64` target is required.

Additionally, smooth user experience is important for `void-ppc64`, so the default `xbps` repo locations as well as signing keys will be updated for the provided packages. Installer and `rootfs` builds will also be provided by the project, as well as static `xbps` and other pieces of infrastructure.

## Supported targets

| Target       | Min. CPU requirement    | Notes                                       |
| ------------ | ----------------------- | ------------------------------------------- |
| ppc64le      | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64le-musl | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64        | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |
| ppc64-musl   | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |
| ppc          | `powerpc` (generic)     | `-mno-altivec -mtune=G4`                    |
| ppc-musl     | `powerpc` (generic)     | `-mno-altivec -mtune=G4`                    |

The typical expected 64-bit little endian target is the Raptor Talos 2, but it is also confirmed to work on POWER8 and possibly other Power ISA 2.07+ hardware. However, notably the NXP e6500 will not work, as it lacks little endian AltiVec support.

For 64-bit big endian, the PowerPC 970 (also known as the G5) is the minimum supported target. AltiVec support is required, so e.g. the e5500 will not work. Modern POWER8/POWER9 is however known to boot and work out of box.

For 32-bit there are no specific restrictions, there is no AltiVec requirement and no specific CPU requirement. The builds are tuned for the G4 though.

**All 64-bit targets, including BE glibc and musl, use the ELFv2 ABI.** The `musl` libc always uses ELFv2 regardless of endianness, glibc uses ELFv2 by default on little endian on all distros but big endian distros typically default to the older ELFv1 ABI; this is for legacy and compatibility reasons, but ELFv2 has other benefits and we have no legacy support, therefore we're targeting ELFv2 on BE glibc as one of the first distributions to do so.

### Target status

This is for the packages currently covered in the binary repository, it does not necessarily reflect what is *possible* or currently working if you build it yourself.

#### ppc64le glibc

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc64le-linux-gnu`
- [x] `linux`
- [x] graphical environment (gtk, qt, xorg, wayland, gnome, xfce4, etc.)
- [x] `rust`
- [x] `go` (native bootstrap)
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
- [x] graphical environment (a bit less than ppc64 glibc)
- [x] `rust` (native and cross)
- [ ] `go` (unsupported on ELFv2 BE, also power8+ only)
- [ ] `java`
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs


#### ppc glibc

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc-linux-gnu`
- [x] `linux`
- [ ] graphical environment
- [x] `rust`
- [ ] `go` (not supported upstream)
- [ ] `java`
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

#### ppc musl

- [x] `base-chroot`
- [x] `base-voidstrap`
- [x] `base-system`
- [x] `cross-powerpc-linux-musl`
- [x] `linux`
- [ ] graphical environment
- [x] `rust`
- [ ] `go` (not supported upstream)
- [ ] `java`
- [ ] `ghc`
- [ ] installer images
- [ ] rootfs tarballs

## Package/build mirrors

The main project binary location is https://void-power.octaforge.org.

- xbps repository (`ppc64le` direct): https://repo.void-power.octaforge.org/current
- xbps repository (`ppc64le-musl` direct): https://repo.void-power.octaforge.org/current/musl
- xbps repository (`ppc64` direct): https://repo.void-power.octaforge.org/current/be
- xbps repository (`ppc64-musl` direct): https://repo.void-power.octaforge.org/current/be/musl
- xbps repository (`ppc` direct): https://repo.void-power.octaforge.org/current/ppc
- xbps repository (`ppc-musl` direct): https://repo.void-power.octaforge.org/current/ppc/musl
- xbps repository (`ppc64le` load balancing): https://auto.void-power.octaforge.org/current
- xbps repository (`ppc64le-musl` load balancing): https://auto.void-power.octaforge.org/current/musl
- xbps repository (`ppc64` load balancing): https://auto.void-power.octaforge.org/current/be
- xbps repository (`ppc64-musl` load balancing): https://auto.void-power.octaforge.org/current/be/musl
- xbps repository (`ppc` load balancing): https://auto.void-power.octaforge.org/current/ppc
- xbps repository (`ppc-musl` load balancing): https://auto.void-power.octaforge.org/current/ppc/musl
- static xbps for all (built using musl): https://void-power.octaforge.org/static
- rsync for mirroring: `rsync://octaforge.org/void-power`

The load balancing variants will try to use different mirrors and are default for our `xbps` packages and so on.

### Mirror list

Since my server space is not free, mirrors are always appreciated! Let me know if you set up a mirror. Use `rsync`, the URL is provided above.

Current list:

- https://mirrors.servercentral.com/void-power/ (provided by `zdykstra` of the Void Linux as well as the Talos community, also hosts a Void mirror)
- https://ppc.exqa.de/ (provided by `Cogitri` of the Void Linux community)

Big thanks to all the mirror providers, as my bandwidth is limited and shared with my other projects.

## Repository structure

The primary area where porting happens is the `void-packages` repository, which mirrors the structure of the upstream template database. It consists of two primary branches:

- `master` - this branch has its consistency ensured, i.e. force pushes are forbidden and it's always pullable; it contains all of the changes, but not necessarily in the correct order or with a pretty `git` history; changes may be split between different commits. It is meant primarily for users and contributors.
- `staging` - same contents as `master`, but without ensuring consistency; this branch is rebased and force pushed into as much as is necessary, and commits are squashed, edited and reordered as is seen fit. All pull request branches are made from this one.
- any other branch is typically temporary, containing a pull request or experiments.

Both primary branches contain all types of changes as described in the Rationale section. Individual changes or commit ranges are cherry picked into temporary branches for submission.

## Contributing

If your changes are ready for upstream submission, please submit them upstream and mention the respective maintainers from this org for review and tracking.

Current maintainers:

- `q66` - mention always
- `pullmoll` - anything affecting 64-bit big endian
- `stenstorp` - anything affecting 32-bit

If you do not wish to submit it upstream yet, you can try submitting it here, by raising a pull request and/or opening an issue. Then it can receive feedback and potentially make it upstream.

## FAQ

**Q:** Will multilib be supported?  
**A:** No, it is not planned. However, the compiler is built as bi-arch, which means `-m32` works, and you can have it emit 32-bit code. This is useful for low level stuff (e.g. GRUB, which needs to emit 32-bit big endian code independent on a libc) while not burdening the higher level infrastructure. If you really need to build or use 32-bit software, use a 32-bit chroot, it should work just fine.

**Q:** I'm using big endian with an ASpeed VGA and colors are broken.  
**A:** This is a problem with the kernel `ast` driver. Since the fix appears to be non-trivial and there is no proper patch available, this is WONTFIX from our side. Either use a dedicated GPU (ideally PCIe, USB2 DisplayLink is known to work) or use little endian if you really need the `ast` to work properly.

**Q:** How do I install this?  
**A:** https://github.com/void-ppc64/documentation/blob/master/INSTALL.md

If you have any questions, suggestions or anything else, I'm (`q66`) available on IRC (Freenode: `#talos-workstation`, `#voidlinux`, `#xbps` and others) as well as on Twitter (`@octaforge`).
