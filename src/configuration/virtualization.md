# Virtualization

In general, virtualization works the same as on any other POWER distro. You
can use either `qemu` (with or without KVM) or some frontend such as `libvirt`.

This will not cover usage of the virtualization tools, as you can easily find
that elsewhere. However, it will cover quirks specific to Void, which are
generally in KVM.

## KVM

There are two methods to get KVM on POWER. These are:

1) `kvm_hv`
2) `kvm_pr`

They have matching modules in the kernel.

The `kvm_hv` method is the recommended way. It is the fastest, matching bare
metal performance; however, it can only virtualize at most one generation
older CPU. For example on POWER9, you can virtualize POWER9 and POWER8 using
this method. It is also only available on modern machines (POWER7 and later).
The endianness (and page size, to a degree) of the guest does not have to match.

The `kvm_pr` (PRoblem State) is the older method that runs entirely at user
level. This has the advantage that you can use it to emulate any POWER or
PowerPC CPU; it can emulate unsupported instructions and so on. However,
it is also quite a bit slower, and requires the host to be running in
HPT mode (Radix is not supported) - this has compatibility implications.
The page size of the guest does not have to match, but it will nearly always
be 4 KiB.

### Compatibility summary

This is a summary of everything written below, to give you an idea what
can and can't run. Read the detailed sections below for more information.
Bold are possible default Void configurations.

- **Radix host, 4 KiB pages** (Void default on POWER9 and newer):
  - Radix guest, 64 KiB pages (HV)
  - Radix guest, 4 KiB pages (HV)
  - HPT guest, 4 KiB pages (HV)
- **HPT host, 4 KiB pages** (Void default on POWER8 and older):
  - HPT guest, 4 KiB pages
  - KVM PR
- Radix host, 64 KiB pages:
  - Radix guest, 64 KiB pages (HV)
  - Radix guest, 4 KiB pages (HV)
  - HPT guest, 64 KiB pages (HV)
  - HPT guest, 4 KiB pages (HV)
- HPT host, 64 KiB pages:
  - HPT guest, 64 KiB pages (HV)
  - HPT guest, 4 KiB pages (HV)
  - KVM PR

Nested virtualization scenarios:

- **Radix guest, 4 KiB pages** or 64 KiB pages:
  - Radix guest, 64 KiB pages (nested HV)
  - Radix guest, 4 KiB pages (nested HV)
- **HPT guest, 4 KiB pages** or 64 KiB pages:
  - KVM PR

### Host system configuration

Void uses kernels with 4 KiB page size. Other distributions generally use
kernels with 64 KiB page size; the reason why is covered in our
[FAQ](../faq/index.md#why-use-4-kib-page-kernels-instead-of-64-kib-like-other-distros).

This, however, means there will be certain limitations, depending on your
system configuration.

#### POWER9 and newer (Radix, HPT)

POWER9 machines (and newer) with a recent enough kernel (all that are provided
by Void for `ppc64le`, and their big endian equivalents capable of booting
on modern machines, which generally means 5.4 and newer - 4.19 big endian
kernel can't boot on modern machines newer than POWER6) default to Radix MMU.
These systems have a multilevel page table similar to other architectures.

You can disable Radix MMU using the `disable_radix` kernel command line
parameter. The only reason to do this is usually to use `kvm_pr`. If you do
that, you will fall back to the traditional HPT (Hashed Page Table).

If you want to check what you're running, use this command:

```
$ grep MMU /proc/cpuinfo
```

You will get either `Radix` or `Hash`.

#### POWER8 and older (HPT)

These machines only support HPT. This has compatibility implications, some
of them with workarounds.

### KVM HV

On supported hardware, this will usually be the default. You can make it
explicit by specifying it as a part of your machine type in `qemu`, for
example like `-machine pseries,accel=kvm,kvm-type=HV`.

The `kvm_hv` module needs to be loaded, and `/dev/kvm` needs to have
permissions set so your user can access it. Usually this is done by adding
your user into the `kvm` group (and log out and in).

#### Radix guests

As long as your host system is also Radix, you can run Radix guests. Radix
guests do not have any limitations (other than those of KVM HV itself).
By default, KVM HV guests running on Radix hosts are always Radix capable,
and a compatible kernel will boot in Radix mode.

In general, this means it doesn't matter what page size the guest has. **You
can boot kernels with either 4 KiB or 64 KiB page size on 4 KiB Radix hosts.**

#### HPT guests

HPT guests will work on either HPT or Radix host. Unlike Radix guests, HPT
guests need to have their page size smaller or equal to the host page size.
**That means default Void kernels can only run HPT guests with 4 KiB pages.**

#### Configuring qemu for Radix guests

As a guest kernel can boot in either Radix or HPT mode, qemu cannot know ahead
of time in which mode it will run.

With the default `-machine pseries` (and any `pseries` of version `3.x` and
newer), you will get an error like this by default on Void:

```
qemu-system-ppc64: Can't support 64 kiB guest pages with 4 kiB host pages with this KVM implementation
```

This can be fixed easily. All you have to do is restrict the maximum page size
for HPT mode:

```
-machine pseries,cap-hpt-max-page-size=4096
```

Your Radix guests will then boot fine.

You can also use an older machine type. With something like `-machine pseries-2.11`,
these guests will boot out of box.

#### Configuring qemu for HPT guests

HPT guests need to have their page size be either smaller or matching with the
host. **That means Void will by default not be able to boot 64 KiB page kernels
with HPT.** There is no existing workaround for that. You can run other
distributions as containers, or compile your own alternative kernels for them.

Moreover, ever since [this patch](https://patchwork.kernel.org/patch/11393187)
(`qemu` 5.0), HPT guests will not boot out of box. You will get an error like
this instead:

```
qemu-system-ppc64: Unable to create 2048MiB RMA (VRMA only allows 512MiB)
```

The guest memory must be backed by larger than default pages. Fortunately, this
can be worked around rather easily, by using the hugepages feature of the Linux
kernel, which is one of the workarounds mentioned in the patch.

To manage hugepage mappings, you can use the `hugeadm` utility (`glibc` only,
package `libhugetlbfs-tools`). However, for simple configurations, you don't
need any external tools.

First check your default huge page size:

```
$ grep Hugepagesize /proc/meminfo
```

By default, it will usually be 2048 KiB. You can alter this by specifying
`hugepagesz=` on kernel command line, you can also specify multiple of them,
the first one will be the default (the choices are 2M, 16M, 1G, 16G). You can
also use the kernel command line to pre-allocate a specific number of hugepages
of each type.

Let's go with the easy default, in this case 2 MiB (2048 KiB). Allocate enough
hugepages to cover guest memory. Let's say, 2 GiB - this will be 1024 hugepages,
plus some extra, let's say 1100.

```
# sysctl vm.nr_hugepages=1100
```

Or alternatively:

```
# echo 1100 > /proc/sys/vm/nr_hugepages
```

If `grep HugePages_Total /proc/meminfo` says the number you want (should be
if you have enough memory), you can proceed to mount the backing and give it
correct permissions:

```
# mount -t hugetlbfs hugetlbfs /dev/hugepages
# chown root:kvm /dev/hugepages
# chmod 1770 /dev/hugepages
```

Later you can add this to `fstab`. The `fstab` line for that would be something
like `hugetlbfs /dev/hugepages hugetlbfs mode=01770,gid=<gid of kvm> 0 0`.

You can also use the `pagesize=` option to mount hugepages of a specific size.

Either way, once you have mounted your backing somehow, you can easily pass
it to `qemu`. On 4 KiB page hosts like Void, you will need to restrict the
guest HPT size, otherwise you will get an error like this, even on say
`pseries-2.11`:

```
qemu-system-ppc64: KVM can't supply 64kiB CI pages, which guest expects
```

Overall, the command will look like this, for example:

```
$ qemu-system-ppc64 -m 2048 -machine pseries,cap-hpt-max-page-size=4096 -mem-path /dev/hugepages ...
```

That's it - the guest should boot now.

#### Nested virtualization with KVM HV

It is possible to nest KVM HV virtual machines. However, this only works in
Radix mode, and you can only have nested Radix guests, i.e. HPT must not appear
anywhere in the chain. You also need to enable it using yet another capability
flag (`cap-nested-hv`), for example:

```
-machine pseries,cap-hpt-max-page-size=4096,cap-nested-hv=on
```

### KVM PR

To use KVM PR, your host must run as HPT. You can then use something like
`-machine pseries,accel=kvm,kvm-type=PR` to enable it. The `kvm_pr` kernel
module must be loaded. You can use PR in parallel with HV.

If you don't want to lose Radix and still want to use KVM PR, there's a trick
you can use. Simply start your host in Radix mode, then boot a HPT guest virtual
machine and run your KVM-PR virtual machine nested in that.
