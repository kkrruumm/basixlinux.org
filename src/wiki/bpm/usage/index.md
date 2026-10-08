title: Basix Wiki

# BPM usage

In general, most of your questions are probably answered by the help output:

```
(0) sagittarius ~ > bpm help
bpm 0.8.3 - basix package manager

usage: bpm <command> [argument ...]

  [b]uild    pkg...   build packages and any missing dependencies
  [i]nstall  pkg...   build if needed, then install into $BPM_ROOT
  [r]emove   pkg...   remove installed packages, -a also removes the orphans
                      that leaves behind, -f skips the reverse dependency check
  autoremove          remove every orphaned package, -n lists them instead,
                      -y skips the prompt
  [u]pdate            pull repositories, rebuild and install what changed
  pin        [pkg...] pin packages at their installed version, no args
                      lists what is pinned
  unpin      pkg...   release the pin
  [p]ull              clone or pull repositories only
  [s]earch   pattern  search repositories (globs allowed)
  [q]uery    pkg      show template metadata and resolved use flags
  [a]lternatives      list alternatives, or bpm a <pkg> <path> to swap
  verify     [pkg...] list files that no longer match what was packaged,
                      -c includes config files, --overwrite <pkg> <path>
                      restores the specified file from the archive
  [l]ist     [-r] [pkg]
                      list installed packages, -r appends the reason each package
                      was installed
  [f]iles    pkg      list files owned by an installed package
  [o]wns     path     show which package owns a path
  [c]hecksum [-w] pkg print blake3 sums of the dist_files (-w rewrites template)
  [d]ownload pkg...   fetch dist_files only
  clean [all]         remove build trees and staged destdirs, "all" also
                      removes the build root base
  [h]elp, [v]ersion

config: /etc/bpm/bpm.conf
repos:  /home/kris/basixpersonal /home/kris/basixstaging /var/db/bpm/repos/core
build:  /var/cache/bpm/buildroot
(0) sagittarius ~ >
```

If not, read on for specific details.

## Building packages

When one invokes `bpm [b]uild` on `[pkg...]`, it will be built in a chroot isolated via unshare.

Once built, packages are stored in, by default, `/var/cache/bpm/bin/`.

## Installing packages

When running `bpm [i]nstall` on `[pkg...]`, bpm will check the binary cache directory to see if the specified package at its current version as determined by the topmost repository providing a match in `/etc/bpm/repos.conf` has been built already.

If it has, then `bpm` will unpack the package onto the system and log it in, by default, `/var/db/bpm/installed/`.

## Removing packages

When running `bpm [r]emove` on `[pkg...]`, bpm will first check to see if there are other packages that depend on the thing you're attempting to remove. If none, then the package is removed from the system (the built package archive is kept in the bin cache).

If there is dependence on what you are attempting to remove, bpm will, by default, refuse to remove the package.

If you wish to override this, you may tack on `-f` to your invocation, e.g. `doas bpm remove -f [pkg]`.

If you wish to automatically remove orphaned packages (packages whose parent is no longer installed), you may tack on `-a` to your `remove` invocation in a similar fashion.

## Autoremoving packages

Related to the above, `bpm autoremove` will prompt the user for input on packages it believes are orphans, and if given permission, will remove these packages.

If one tacks on `-n` to the command, e.g. `bpm autoremove -n`, it will simply list packages it believes are orphans instead of attempting a removal.

When invoked with `-y`, it will skip the prompt for whether or not it should remove the packages it's found.

## Updating packages

When running `bpm [u]pdate`, bpm will refresh the repositories (pull them again from their upstreams), rebuild and install any packages that have had version or revision bumps, and install them.

## Pinning packages

This is useful if you don't want to let certain things update at the moment or potentially at all. This may also be useful if you don't want to be updating huge things like the kernel on a regular basis, though try not to be neglectful.

When running `bpm pin [pkg...]`, there will be an empty file called `pin` placed in the packages database (in /var/db/bpm/installed/ by default). The existence of this file tells bpm it shouldn't attempt to update it even when a new version is available.

When running `bpm pin` with no supplied arguments, it will list the packages that are currently pinned, if any.

## Unpinning packages

Run `bpm unpin [pkg...]` to remove the pin from a pinned package. On the next `[u]pdate`, bpm will now attempt to update this package if a newer version of it exists in the repos you have defined.

## Pulling package repositories

Running `bpm [p]ull` will sync all non-local package repositories listed in your `/etc/bpm/repos.conf` without attempting to install or update anything.

Open `/etc/bpm/repos.conf` in a text editor for information on adding repositories and package shadowing.

## Searching for packages in package repositories

Running `bpm [s]earch [query]` will search your synced package repositories for packages.

Example output:

```
(0) sagittarius ~ > bpm search linux
linux-stable	/home/kris/basixstaging
linux-firmware	/var/db/bpm/repos/core
linux-headers	/var/db/bpm/repos/core
linux-lts       /var/db/bpm/repos/core
linux-pam       /var/db/bpm/repos/core
linux-stable	/var/db/bpm/repos/core
s6-linux-init	/var/db/bpm/repos/core
s6-linux-utils	/var/db/bpm/repos/core
util-linux	    /var/db/bpm/repos/core
(0) sagittarius ~ > 
```

This will also list out the repository the matching package(s) exist in.

## Package querying

When running `bpm [q]uery [pkg]`, bpm will look for this package, choose the first match from your repositories, and provide various information about the package.

This information includes potential *and* resolved use flags, version info, the `build-style` of this package, resolved dependencies, etc.

Example output:

```
(0) sagittarius ~ > bpm q sway
pkg_name    sway
version     1.12
revision    2
build_style meson
short_desc  Tiling Wayland compositor compatible with i3
home_page   https://swaywm.org
license     MIT
depends     cairo dejavu-fonts-ttf json-c libdrm libevdev libinput libudev-zero
            libxkbcommon pango pcre2 pixman wlroots gdk-pixbuf basu
makedepends cairo json-c libdrm libevdev libinput libudev-zero libxkbcommon
            meson pango pcre2 pixman pkgconf wayland-protocols wlroots xorgproto
            scdoc gdk-pixbuf basu
use_flags   man swaybar swaynag tray -wallpapers pixbuf
use         man swaybar swaynag tray -wallpapers pixbuf
template    /var/db/bpm/repos/core/sway/template
installed   1.12-2 (auto)
required by nothing
(0) sagittarius ~ >
```

The information in the query output is rarely static. Listed depends/makedepends may change depending on what use flags you have set, and the query command will reflect the state of what features the package has enabled or disabled.

## Package alternatives

This is arguably the most powerful feature of bpm, and the concept was more or less stolen from the `kiss` package manager.

When a user installs two conflicting packages, this is totally fine and infact encouraged. Any overlapping binaries or files from the new package will simply be placed in the choices store (located in `/var/db/bpm/choices` by default) and bits of packages here and there can be swapped, mixed, and mangled at will/on the fly.

An entry in the choices store actually is is just the overlapping file that wasn't placed on the filesystem normally due to the conflict. An example of this may be something like `busybox` providing an `ip` binary and `iproute2` providing an `ip` binary as well.

When running `bpm [a]lternatives` without supplying an argument, potential alternatives will be listed.

Example:

```
(0) sagittarius ~ > bpm a
bzip2 /usr/bin/bzgrep
foot /usr/share/terminfo/f/foot
foot /usr/share/terminfo/f/foot-direct
iproute2 /usr/bin/tc
loksh /usr/bin/sh
loksh /usr/share/man/man1/sh.1
m4 /usr/bin/m4
m4 /usr/share/man/man1/m4.1
util-linux /usr/bin/addpart
util-linux /usr/bin/cal
```

This output lists the potential alternative package to supply a given path.

For example, if I wanted `loksh` to be the provider of my `/usr/bin/sh` on my system, I can simply swap this by running ``bpm a loksh /usr/bin/sh``. The binary/file/whatever that previously sat in that path will be moved into the choices store, so you may swap back just as easily.

Get creative, and try not to break shit as there aren't really any safeguards here.

This feature is also how may one may perform a clean initial migration of something like a coreutils provider. As of the time of writing, I am using `chimerautils` on my system as opposed to busybox, and once migrated, I simply removed the `busybox` package from my system. Note that busybox is still needed for build environments unless you choose to modify that too (which is very doable as well).

## Verifying package contents

Every file shipped by a bpm package has an attached blake3 hash, which would allow bpm to detect and potentially fix things like corruption.

Of course, this falls short in a couple ways, if the archive itself is corrupted then it just needs to be rebuilt. This is also **not** a way to detect system tampering, this in particular is a nongoal.

When you run `bpm verify` with no supplied arguments, `bpm` will do a system-wide walk over package provided files to see if any of them do not match the blake3 hash that was shipped by the package. If it finds any, you may have `bpm` "repair" these by reextracting the incorrect files from the archive.

If you instead run `bpm verify [pkg...]`, bpm will only do its "walk" for the specified packages instead of the whole system, as the whole system may take a little bit of time.

The usage is similar to the alternatives system, example:
```
(0) sagittarius ~ > bpm verify
foot /usr/share/terminfo/f/foot changed
foot /usr/share/terminfo/f/foot-direct changed
```

In this case, `changed` is telling us that there is another provider for these files (via the alternatives system), and that's why there's a mismatch.

Assuming these files were actually broken, you could reextract the file from the archive with ``bpm verify --overwrite foot /usr/share/terminfo/f/foot``, for example.

If you tack on `-c`, the "walk" will also include config files (off by default because, of course, these are likely to be user modified).

The hashing feature of bpm is also precisely how the "smart config handling" works. More on that later.

## Listing packages

When running `bpm [l]ist`, bpm will list out every installed package.

You may append `-r` as an argument to this to also list out the *reason* each package was installed, that being one of `explicit` or `auto`.

The "reasons" here are pretty simple, if something was installed by the user intentionally, that being explicitly listed in a `bpm install` command, this package is considered explicit. Anything pulled in as a dependency automatically as a result of explicit packages are considered `auto`. This is how the orphans system works. The reason is contained in the package database at, for example, `/var/db/bpm/installed/foot/reason`.

## Listing files owned by a package

When running `bpm [f]iles pkg`, bpm will list all of the paths that package owns. Example:

```
(0) sagittarius ~ > bpm f zlib
/usr/
/usr/include/
/usr/include/zconf.h
/usr/include/zlib.h
/usr/lib/
/usr/lib/libz.a
/usr/lib/libz.so
/usr/lib/libz.so.1
/usr/lib/libz.so.1.3.2
/usr/lib/pkgconfig/
/usr/lib/pkgconfig/zlib.pc
/usr/share/
/usr/share/man/
/usr/share/man/man3/
/usr/share/man/man3/zlib.3
/var/
/var/db/
/var/db/bpm/
/var/db/bpm/installed/
/var/db/bpm/installed/zlib/
/var/db/bpm/installed/zlib/depends
/var/db/bpm/installed/zlib/manifest
/var/db/bpm/installed/zlib/sums
/var/db/bpm/installed/zlib/template
/var/db/bpm/installed/zlib/use
/var/db/bpm/installed/zlib/version
(0) sagittarius ~ >
```

## Finding out what package owns a path

When running `bpm [o]wns [path]`, bpm will search the database and figure out what package currently owns that path on the filesystem. Example:

```
(0) sagittarius ~ > bpm o /boot/vmlinuz-7.2.9_1  
linux-stable
(0) sagittarius ~ > 
```

## Generating a checksum for a packages dist_files

This is for package maintainers.

When running `bpm [c]hecksum pkg`, bpm will output the blake3 hash of the `dist_files` for that source tarball or whatever it may be.

If you append `-w` to checksum, bpm will attempt to overwrite the config with the locally computed blake3 hash.

## Fetching dist_files

This is also for package maintainers.

When running `bpm [d]ownload [pkg...]`, bpm will only fetch the `dist_files` specified in the package templates, and do nothing else.

## Cleaning the build_root

Occasionally or potentially between every package build, your `build_root` should be cleaned up. 

This is a neat feature of bpm- all non base stuff for the build container is kept in an overlayfs which keeps everything that goes on top of this during builds disposable/ephemeral. This is so you don't have to regenerate the container every time you wish to clean it up.

When running `bpm clean`, everything in this overlayfs will be disposed of and the base of the container will be kept.

You may append `all` to your clean invocation to *also* dispose of the base of the container, if you wanted to do so.

# Configuration files for bpm

There are various configuration files stored in, by default, `/etc/bpm/`. These generally control global behavior, and these files are well commented.

Here's a quick rundown:

- bpm.conf - global bpm behavior, settings, and things like your compiler flags
- package.use - user overrides for package use flags, this is how you customize your builds of things
- repos.conf - the list of package repositories, resolved top to bottom and the first match for a package is kept

I'm not going to elaborate on all of this in this document, as I believe the package comments provide plenty of information.

Here's a brief example of my repos.conf, though:

```
# /etc/bpm/repos.conf - package repositories, highest priority first
#
#    <name> <git url or absolute path> [branch]
#
# git entries are cloned into $BPM_REPODIR/<name> by `bpm pull`, absolute
# paths are used where they are, which is useful for local "overlays"
#
# the first repository providing a template wins, so a local overlay listed
# above core lets you shadow any package

#local /home/me/overlay
personal /home/kris/basixpersonal
local /home/kris/basixstaging
core https://github.com/kkrruumm/basix-packages.git main
```

# Config handling

There is some degree of "smart" (i kinda dont like describing it that way) config handling.

Given bpm logs a blake3 hash for every file a package provides, this includes config files.

bpm will treat *every* file in /etc as a config.

When updating a package that provides a config, bpm will check to see if that configs hash matches the hash of the one it shipped last, meaning the user hasn't modified it. In this case, bpm will overwrite the config automatically.

If the hash doesn't match, this implies the user has modified it, and thus the new config will instead be stored next to the existing one as `config.new`.
