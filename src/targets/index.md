# Supported Targets

The project supports three architectures, each with `glibc` and `musl`. A basic
table is below:

| Target       | Min. CPU requirement    | Notes                                       |
| ------------ | ----------------------- | ------------------------------------------- |
| ppc64le      | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64le-musl | `powerpc64le` (generic) | `-maltivec -mtune=power9`, POWER8 or better |
| ppc64        | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |
| ppc64-musl   | 970 / G5                | `-maltivec -mtune=power9`, POWER4 or better |
| ppc          | `powerpc` (generic)     | `-mno-altivec -mtune=G4`                    |
| ppc-musl     | `powerpc` (generic)     | `-mno-altivec -mtune=G4`                    |

The typical expected little endian target (`ppc64le`) is a Raptor Talos 2,
Blackbird or similar commonly accessible hardware. At very least, you will
need a POWER8, which means ISA 2.07 and little endian AltiVec/VSX. There may
be other hardware implementing these, but that is currently untested. Notably
the `e6500` systems are not supported, as they implement ISA 2.07 but do not
support little endian AltiVec/VSX.

For booting the little endian live media, a PowerNV (OpenPOWER) or PowerVM
environment is expected, so either Petitboot or SLOF can be used. In the
former case, it will directly load the kernel, in the latter case, GRUB
will be loaded by the firmware.

The 64-bit big endian requirements are more relaxed and require at least a
PowerPC 970 (G5) with AltiVec support. That means any 64-bit PowerPC Mac
will be able to boot the system, but not POWER4/POWER5, as they do not
support AltiVec (same with e.g. `e5500`; `e6500` should work). POWER6
and newer can boot it, this includes all targets supported by little
endian as well.

The 32-bit builds are completely generic and require no specific processor.
However, the live media expect an OpenFirmware environment (IBM or Apple
style), and the typical platform for those is therefore an Apple PowerPC
system of the NewWorld kind (any G4, plus G3 "Blue and White" and newer).
Other environments may need manual intervention.

This listing is not exhaustive. There may be other platforms capable of booting
the media. If you know of one, or know how to make one work, please let us know.

All 64-bit targets use the ELFv2 ABI for both kernel and userland. This includes
big endian `glibc`. This may have some compatibility implications, so be aware
of those (you will not be able to run legacy prebuilt binaries directly and
will need to set up a container with another distribution). Void aims to be
a legacy-free system and is the first and currently only distribution to use
ELFv2 on big endian `glibc`, as well as the first and only to use it for the
big endian kernel.

## Repository status

There is a separate [statistics](https://repo.voidlinux-ppc.org/stats.html) page
with a detailed matrix of what is built, what is buildable and what is not.

At the time of writing this, at least all major desktops and common applications
were available for all targets. Application-specific issues may exist, but not
more than on other distributions. Please report any issues you may come across
[here](https://github.com/void-ppc/void-packages/issues).
