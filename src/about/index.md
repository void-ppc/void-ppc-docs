# About

Void Linux for PowerPC/Power ISA is a currently unofficial staging fork of
the Void Linux distribution that is meant, as the name says, for PowerPC and
Power architecture devices. Its primary goal is upstreaming support for the
architecture into the upstream distribution. Its secondary goal is to provide
people with a complete, production ready distribution while there is still no
official repository.

Void currently has no official support for the architecture. While vast
majority of the actual source changes have been upstreamed (and therefore,
official `void-packages` can compile most things just fine), there is no
official binary repository. The reason for this is largely technical, as
the build infrastructure does not seem to be able to handle any more builders.
This may persist for a while, and that's where this fork comes in.

We put emphasis on wide hardware support. Therefore, you can run the distro
on a lot of different devices, including modern OpenPOWER hardware such as
the Raptor Talos 2 and Blackbird, various old PowerPC Macs (G3/G4/G5) and
even consoles like Nintendo Wii U. The distribution supports both 32-bit
and 64-bit hardware, and both little and big endian for 64-bit hardware.

Besides the things that set Void itself apart from the others, the PowerPC
fork has some unique aspects of its own. For example, it uses the modern
ELFv2 ABI not only on little endian targets but also on big endian for both
`glibc` and `musl` and even the kernel itself. We aim to be legacy free, and
being a new port, making that a reality becomes a lot easier.

In total we have six flavors of the distro, `ppc64le`, `ppc64le-musl`, `ppc64`,
`ppc64-musl`, `ppc` and `ppc-musl`. The requirements and hardware support of
each is described in other sections.

Last but not least, wide software support is also important. The project aims
for a complete repository coverage (where applicable) and therefore parity
with `x86_64` on all targets. This is a work in progress, and the repository
sizes may differ.

Since we provide custom repositories, our fork of `void-packages` is also
updated to use those. Therefore, as a user, you can build your own packages
effortlessly, just like if you were on `x86_64`.

