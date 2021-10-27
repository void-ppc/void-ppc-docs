# Graphics

The goal of this page is to track compatibility with graphics hardware as well
as provide information on how to configure some specifics.

## ATI/AMD graphics cards

In general this comes down to generation and the port you are running.

On older kernels, all AGP Radeon cards are affected by random hangs caused by
AGP GART code. You will need to provide `radeon.agpmode=-1` on kernel command
line to work around the issue. This is not necessary at least on kernels 4.19
and newer.

### Rage, Rage Pro etc.

**Interface:** PCI, AGP  
**OpenGL:** 1.2  
**Works:** Console, X11 untested but may work  
**KMS:** No  
**3D acceleration:** No
**2D acceleration:** X11 (untested)
**X11 driver:** `xf86-video-mach64`  
**Wayland:** Only compositors that support `fbdev`

These models can generally be found in some old Macs. The framebuffer console
is served by an `fbdev` driver, there is no modern KMS driver.

In X11, these GPUs are served by the `xf86-video-mach64` driver. It will not
work out of box - you need to correctly set up your modelines in `xorg.conf.d`
to make it work.

### Rage 128 series

**Interface:** PCI, AGP  
**OpenGL:** 1.2  
**Works:** Console, X11 untested but may work  
**KMS:** No  
**3D acceleration:** No
**2D acceleration:** X11 (untested)
**X11 driver:** `xf86-video-r128`, `xf86-video-fbdev`
**Wayland:** Only compositors that support `fbdev`  

Similar things as above apply.

### Radeon 7000 series

**Interface:** PCI, AGP  
**OpenGL:** 1.3  
**Works:** Yes, AGP may have issues  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11  
**X11 driver:** `xf86-video-ati`  
**Wayland:** Only compositors that support `fbdev`

These cards come up, but experience freezes shortly after boot, as far as
has been confirmed. 3D acceleration is covered by Mesa. These cards only
support fixed-function OpenGL, which means you can't use the modesetting
driver or GLAMOR with them.

### Radeon 8000 series to 9250

**Interface:** AGP  
**OpenGL:** 1.4  
**Works:** Yes, issues on `musl`  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11  
**X11 driver:** `xf86-video-ati`  
**Wayland:** Only compositors that support `fbdev`

These cards work, including 3D acceleration. However, on `musl` systems,
OpenGL currently renders junk, and there is no known workaround other than
using a `glibc` system.

Since these cards do not support shaders, you can't use the modesetting driver
or GLAMOR with them.

### Radeon 9500 and newer, X300 - X600, X1050

**Interface:** AGP  
**OpenGL:** 2.0/2.1  
**Works:** With issues  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards seemingly work, but experience system hangs when running complex
OpenGL applications (the issue manifests e.g. when loading a map in a 3D
game).

These cards support shaders and generally render simple things (e.g. `glxgears`)
correctly, including on `musl` systems. Being OpenGL 2.1 capable hardware, they
can run accelerated Wayland and can use the `modesetting` driver and GLAMOR in
X11.

In general, things are good enough for desktop work and video, without stability
issues. Running games results in those hangs, though.

### Radeon X700 - X850, X12xx

**Interface:** AGP, PCI Express  
**OpenGL:** 2.0/2.1  
**Works:** Yes (untested)  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards should work but haven't been widely tested.

### Radeon X700 - X850, X12xx

**Interface:** AGP, PCI Express  
**OpenGL:** 2.0/2.1  
**Works:** Yes (untested)  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards should work but haven't been widely tested.

### Radeon X1300 - X19xx

**Interface:** AGP, PCI Express  
**OpenGL:** 2.0/2.1  
**Works:** Depending on model  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards should technically work.

However, on Macs, at least certain models use a reduced video BIOS, where a
part of it is loaded by the OS afterwards. This prevents function in Linux.
No workaround is currently known to the project.

PC versions of the cards should work at least on G4 Macs and so on once booted
into Linux.

### Radeon HD 2xxx - 4xxx

**Interface:** AGP, PCI Express  
**OpenGL:** 3.3 (3.2 on big endian)  
**Works:** Yes  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards work. You may experience driver bugs on big endian systems.

### Radeon HD 5xxx, 6xxx, 7450, 8450, R5 230/235 etc.

**Interface:** PCI Express  
**OpenGL:** 4.4/4.5 (3.2 on big endian)  
**Works:** Yes  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-ati`, `modesetting`  
**Wayland:** Yes

These cards work. You may experience driver bugs on big endian systems. You
will also not be able to utilize OpenGL beyond 3.2 on big endian systems.

These are the last cards that function on big endian systems in general.
GCN cards currently have broken kernel drivers in all kernels.

### Radeon GCN (HD 7000/8000 series, R/RX/WX series etc.)

**Interface:** PCI Express  
**OpenGL:** 4.6  
**Works:** Little endian  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-amdgpu`, `modesetting`  
**Wayland:** Yes

This includes everything up to Vega. These cards only work on little endian
systems due to `amdgpu` kernel driver requirement.

### Radeon RDNA (Navi)

**Interface:** PCI Express  
**OpenGL:** 4.6  
**Works:** Little endian and kernel 5.4+  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-amdgpu`, `modesetting`  
**Wayland:** Yes

In Void, AMD Navi cards (RX 5xxx etc.) work starting with kernel 5.4. In
upstream (vanilla) kernel you will need at least 5.6.

## NVIDIA graphics cards

Support for NVIDIA cards is generally significantly more limited and you will
need to use the `nouveau` driver.

### Pre-GeForce (RIVA etc.)

**Interface:** PCI, AGP  
**OpenGL:** 1.2  
**Works:** Console, X11 may work with `fbdev`  
**KMS:** No  
**3D acceleration:** No  
**2D acceleration:** No  
**X11 driver:** `xf86-video-fbdev`  
**Wayland:** Only compositors that support `fbdev`

The framebuffer console is served by an `fbdev` driver, there is no modern KMS
driver.

There is no 3D acceleration support, you may still be able to get X11 to work
with `xf86-video-fbdev`.

### GeForce 2 and older, 4 MX

**Interface:** AGP  
**OpenGL:** 1.3  
**Works:** Issues  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Only compositors that support `fbdev`

At least GeForce 2 MX only comes up in kernel 4.4, with newer kernels failing
to initialize the card. It is not known whether this affects GeForce 3 and 4 MX
series as well.

X11 will not come up on GeForce 2 MX. There is some issue with the video outputs
disappearing when scanning EDID.

### GeForce 3 and 4

**Interface:** AGP  
**OpenGL:** 1.3  
**Works:** Unknown  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Only compositors that support `fbdev`

It is currently unknown whether these cards work.

### GeForce FX

**Interface:** AGP  
**OpenGL:** 2.1  
**Works:** Unknown  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Yes

It is currently unknown whether these cards work.

### GeForce 6xxx, 7xxx

**Interface:** AGP, PCI Express  
**OpenGL:** 2.1  
**Works:** Yes  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Yes

These cards work, but suffer from `nouveau` bugs, which may cause broken
rendering in various applications. Some video pixel formats may also be
broken. In general it is good enough for video and accelerated desktop.

### GeForce 8xxx, 9xxx, 200 series

**Interface:** PCI Express  
**OpenGL:** 3.3  
**Works:** Little endian  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Yes

These cards should work with `nouveau` at least on little endian systems.

### GeForce 400 and newer series

**Interface:** PCI Express  
**OpenGL:** 4.6  
**Works:** Little endian, when supported by `nouveau`  
**KMS:** Yes  
**3D acceleration:** Yes  
**2D acceleration:** X11, GLAMOR  
**X11 driver:** `xf86-video-nouveau`  
**Wayland:** Yes

These cards should work with `nouveau` at least on little endian systems.
Exact hardware support is subject to `nouveau` support (e.g. reclocking on
new cards may not work and so on).
