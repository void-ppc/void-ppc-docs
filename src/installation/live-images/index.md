# Live Installers

Like official Void, we provide a set of live images containing a bootable
system with an installer. These live images are also useful for rescue.

All 6 platforms have live images, including graphical flavors.

## Installer images

There are two kinds of images just like in the official distro; base images
and flavor images. Flavor images come with a graphical desktop environment.

### Base images

These provide only a minimal set of packages to install on the given platform.
This basically means `base-system` plus a few extras for partitioning and so on.

### Flavor images

These come with a desktop environment, a web browser and some other basic
applications. Please keep in mind that the installer will only copy over the
whole desktop if you use the local installation option. Network installation
only installs the base system always.

#### Comparison of flavor images

This differs slightly from the official distro. The desktops are the same but
the browsers and other applications might differ.

|                   | Enlightenment                                         | Cinnamon        | LXDE                                    | LXQT           | MATE                                                                                                                                                                | XFCE                                                                                                                                |
|-------------------|-------------------------------------------------------|-----------------|-----------------------------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Window Manager    | Enlightenment Window Manager                          | Mutter (Muffin) | Openbox                                 | Openbox        | Metacity (Marco)                                                                                                                                                    | xfwm4                                                                                                                               |
| File Manager      | Enlightenment File Manager                            | Nemo            | PCManFM                                 | PCManFM-Qt     | Caja                                                                                                                                                                | Thunar                                                                                                                              |
| Web Browser       | Firefox (ppc64le, ppc64), Epiphany (ppc)              | <=              | <=                                      | <=             | <=                                                                                                                                                                  | <=                                                                                                                                  |
| Terminal          | Terminology                                           | gnome-terminal  | LXTerminal                              | QTerminal      | MATE terminal                                                                                                                                                       | xfce4-Terminal                                                                                                                      |
| Document Viewer   | -                                                     | -               | -                                       | -              | Atril (PS/PDF)                                                                                                                                                      | -                                                                                                                                   |
| Plain text viewer | -                                                     | -               | -                                       | -              | Pluma                                                                                                                                                               | Mousepad                                                                                                                            |
| Image viewer      | -                                                     | -               | GPicView                                | LXImage        | Eye of MATE                                                                                                                                                         | Ristretto                                                                                                                           |
| Archive unpacker  | -                                                     | -               | -                                       | -              | Engrampa                                                                                                                                                            | -                                                                                                                                   |
| Other             | Mixer, EConnMan (connection manager), Elementary Test | -               | LXTask (task manager), MIME type editor | Screen grabber | Screen grabber, file finder, MATE color picker, MATE font viewer, Disk usage analyzer, Power statistics, System monitor (task manager), Dictionary, Log file viewer | Bulk rename, Orage Globaltime, Orage Calendar, Task Manager, Parole Media Player, Audio Mixer, MIME type editor, Application finder |
