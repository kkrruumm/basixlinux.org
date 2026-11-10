title: Basix Wiki

# Kernel choices

Basix has two kernel options- that being `linux-lts` and `linux-stable`. `linux-lts` tracks the latest LTS release (6.18 as of the time of writing) and `linux-stable` tracks the latest stable (not mainline!) release (7.3 as of the time of writing).

Which of these you choose, assuming you use a Basix-provided kernel option, is up to you. If you're running something out-of-tree such as `zfs`, you may wish to choose the LTS kernel instead of the latest stable release.

# Kernel configuration

The default kernel configuration included with Basix by default is a generalized "one size fits all" kernel configuration that was initially pulled from Void Linux and slightly tweaked (enabling `CONFIG_MODULE_SIG_ALL` and `CONFIG_INIT_ON_FREE_DEFAULT_ON`). This configuration is rather massive, and may take quite a long time to compile, so I would encourage you to strip out features you do not need.

A bit of configuration can be done with use flags on both kernel options in the form of a few "broad-stroke" toggles to disable pretty huge features.

If you would like to replace the kernel config with your own entirely, copy out one of the packages somewhere, register that directory as a bpm repository in `/etc/bpm/repos.conf`, and replace `dotconfig` with your own kernel configuration. At this point, you have your own kernel package, and may just ignore the officially provided ones.

You may also just take the traditional route of pulling a kernel source and compiling it and copying the files out manually, but I do not recommend doing this.

# Kernel hooks

Since the `bpm` package manager makes no assertion as to different types of hooks, you get to freestyle this quite a bit.

If you want to perform some form of operation related to the kernel (or any other package) on installation, removal, before installation or removal, simply create an executable in `/etc/bpm/hooks/<operation>/`. Hooks in general will be run in the order they come in from the filesystem.

All hooks on the system are always run by every package transaction that fits its category, so a kernel hook needs to check to make sure that it is indeed a kernel package that is invoking it. The first argument supplied to the hook is the package name and each hook will decide what to do with that.

Here is an example kernel hook, provided by the `tinyramfs` package:

```
(0) sagittarius ~ > cat /etc/bpm/hooks/post-install/20-initramfs
#!/bin/sh
# args: <pkg_name> <root> <version>
#
# this generates /boot/initramfs-<version>.img for every kernel the package that
# was just installed ships, a package is considered a kernel by this script if
# the manigest for it has a vmlinuz in it so it works for whatever kernel package name

root=${2%/}
in_root() { if [ -n "$root" ]; then chroot "$root" "$@"; else "$@"; fi; }

# an upgrade or revbump on a kernel is an install operation and the old vmlinuz
# and its modules are dropped without remove hooks so the image for the release that just
# got nuked is cleaned up here, only images that have a missing kernel and modules both are cleaned
prune() {
    for _img in "$root"/boot/initramfs-*.img; do
        [ -f "$_img" ] || continue
        _r=${_img##*/initramfs-}; _r=${_r%.img}
        [ -e "$root/boot/vmlinuz-$_r" ] && continue
        [ -d "$root/usr/lib/modules/$_r" ] && continue
        rm -f "$_img" && printf '%s\n' "tinyramfs: removed stale $_img"
    done
}

_m=$root/var/db/bpm/installed/$1/manifest
[ -f "$_m" ] || exit 0
_rels=$(sed -n 's|^/boot/vmlinuz-\(.*[^/]\)$|\1|p' "$_m")
[ -n "$_rels" ] || exit 0

[ -x "$root/usr/bin/tinyramfs" ] || exit 0
if [ ! -f "$root/etc/tinyramfs/config" ]; then
    printf '%s\n' "tinyramfs: no /etc/tinyramfs/config, not generating an initramfs for $1" >&2
    exit 1
fi

_rc=0
for _rel in $_rels; do
    if [ ! -d "$root/usr/lib/modules/$_rel" ] &&
       ! grep -q '^[[:space:]]*monolith=..*' "$root/etc/tinyramfs/config"; then
        printf '%s\n' "tinyramfs: $1 has no modules for $_rel, set monolith=1 in /etc/tinyramfs/config" >&2
        _rc=1; continue
    fi
    printf '%s\n' "tinyramfs: generating /boot/initramfs-$_rel.img"
    # the image can end up holding keyfiles from the luks hook
    ( umask 0077; in_root /usr/bin/tinyramfs -f -k "$_rel" "/boot/initramfs-$_rel.img" ) || _rc=1
done

prune
exit "$_rc"
```

In this case, we can see that the hook is making sure that the supplied manifest for the named package contains a `/boot/vmlinuz*` file, indicating that it is indeed a kernel. If this condition is not met, the hook will simply exit and bpm moves on.

# Loading kernel modules temporarily

Run `modprobe <module_name>` as root, same as any other system.

# Loading kernel modules on boot

Navigate to the `/etc/modules-load.d/` directory.

Here, you may create a `.conf` file to modprobe a specific kernel module during boot. Simply put the name of the kernel module you'd like to have load on boot in this file.

For example,

```
(1) sagittarius ~ > cat /etc/modules-load.d/zram.conf 
zram
(0) sagittarius ~ >
```

You may have as many of these as you want/need.

# sysctl

There is a default provided sysctl config at `/etc/sysctl.d/10-basix.conf`, which largely provides hardening options by default.

If you wish to overwrite or supply your own extra runtime kernel options, you should set them in `/etc/sysctl.conf`, which will take precedence over anything in `/etc/sysctl.d/`.
