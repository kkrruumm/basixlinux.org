title: Basix Linux

# About

Basix is a strictly non-corporate Linux(R) meta-distribution for the x86_64 architecture (though, there's little to no reason it can't be ported to others) that primarily focuses on *actual* modularity, conceptual simplicity, and user freedom.

The (primary) goal of Basix is not extreme minimalism, though minimalism tends to be a result of a conceptually simple and modular base system.

When a user installs Basix, they receive a complete copy of the distribution. Largely, this means that if one chooses to disconnect from Basix as an upstream and turn it into their own thing, pretty much nothing stands in the way of doing so. 
This is achieved by the utter lack of infrastructure, package repositories are just git repos.

At the moment, Basix should be considered incomplete and almost "beta software." With that said, its creator (me hi hello) is currently daily driving it.

# Default Software

Basix defaults to some potentially unusual software, but all of this can be changed by the user.

Most of these may be considered "replacements" for typical software.

init/service management/logging/etc: [s6](https://skarnet.org/software/s6/)

ssl: [libressl](https://www.libressl.org/)

device management: [mdevd](https://skarnet.org/software/mdevd/)

coreutils: [busybox](https://busybox.net/)

libc: [musl](https://musl.libc.org/)

package manager: our very own [bpm](https://github.com/kkrruumm/bpm)

service scripts: [execline](https://www.skarnet.org/software/execline/)

rsync: [openrsync](https://github.com/kristapsdz/openrsync)

cron: a wrapper around [snooze](https://github.com/leahneukirchen/snooze)

man: [mandoc](https://mandoc.bsd.lv/)

privilege escalation: [opendoas](https://github.com/Duncaen/OpenDoas) OR [s6-sudo](https://skarnet.org/software/s6/s6-sudo.html)

display server: [wayland](https://wayland.freedesktop.org/)

ntp: [openntpd](https://openntpd.org/)

seat management: [seatd](https://sr.ht/~kennylevinsen/seatd/) + [dumb_runtime_dir](https://github.com/ifreund/dumb_runtime_dir).

..and much more. If you're after an elaboration on why all of this stuff has been chosen, view the [philosophy](philosophy/index.html) page.

# Thank you

Basix was largely inspired by several things:

[Kiss](https://kisscommunity.bvnf.space/)
[Void](https://voidlinux.org/)
[Crux](https://crux.nu/)
[Gentoo](https://www.gentoo.org/)

In particular, Voids `xbps-src` package build system and especially kiss' `kiss` package manager / build system were used over the years as reference and inspiration for `bpm`.

If these didn't exist to ~~yoink ideas and code from~~ reference, `bpm` would probably not exist.

# This website, huh?

This website is a knockoff of [werc](https://werc.cat-v.org/), but generated with the SSG in the footer.

As for the box itself, this was the cheapest vps I could acquire from a somewhat reasonable hosting provider. It is currently running OpenBSD, and this website is served with OpenBSDs very own [httpd](https://man.openbsd.org/httpd.8).
