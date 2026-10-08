title: basix wiki

# Basic Basix installation guide

To follow this guide, you should already largely be comfortable installing a Linux distribution manually.

The installation process for Basix is not unlike that of Kiss Linux or, to a lesser extent, Gentoo.

This guide will be assuming a UEFI target.

## Getting started

To start, you need to have another distribution to perform your bootstrap. Generally, I use the Void Linux live ISO, but largely this can be bootstrapped from pretty much whatever will boot.

At minimum, whatever you deploy Basix from needs to have a basic set of utilities, specifically a `tar` implementation and ideally a `chroot` wrapper, such as Void's `xchroot` or Kiss' `kiss-chroot`. You will also need the fsprogs for whatever filesystem you choose to deploy, as well as `cryptsetup` if you intend on encrypting your disk (you should).

## Partitioning the disk

First, one must partition the target disk- same as any other distribution. You may use `fdisk`, `cfdisk`, `parted`, whatever partitioning tool you want.

Generally, UEFI systems consist of two partitions- that being the `ESP` (EFI System Partition) and the root partition. For the most part, more than two partitions is rarely ever warranted.

The sizes of these are somewhat arbitrary, but I largely recommend defaulting to a minimum of `500M` for your ESP. This should provide you with enough room to grow.

The second partition, in general, consumes the rest of the disk.

## Formatting your partitions

Your ESP should generally be formatted as fat32. Some motherboards may have builtin drivers for things like ext4, but this is *rather* unlikely.

Using `sda` as an example disk here, you may do this with:

``mkfs.vfat -F32 /dev/sda1``

where `sda1` is the small ESP created in the section prior.

For your root partition, you have several choices.

You may, for example:

1. Encrypt the partition as a whole with cryptsetup (no, you do not need to split off /boot)
2. Rock it as is and just create the filesystem on the root partition with no encryption

I largely recommend encrypting the root filesytem regardless of what this machine is or what it's used for, it's pretty much free security short of a $5 wrench attack. I can't think of much reason to avoid protecting your data.

### Encrypted root

In the case of encryption, you may:

``cryptsetup luksFormat --type luks2 /dev/sda2``

and enter your passphrase.

Note that the version of GRUB currently in the Basix `core` repository does not support luks2, so you may replace the --type flag with `luks1`. This only applies if you don't intend on using something else, such as just booting a UKI directly.

Once encrypted, you must:

``cryptsetup luksOpen /dev/sda2 root``, where `root` may be any name you'd like to give this container. This is rather arbitrary.

Once unlocked, it's time to pick a filesystem. The currently supported filesystems by the Basix `core` repository are xfs and ext4. For this example, I will go with xfs as that is my preference.

Given this example, run:

``mkfs.xfs /dev/mapper/root``

and wait for it to finish.

### Unencrypted root

It's time to pick a filesystem. The currently supported filesystems by the Basix `core` repository are xfs and ext4. For this example, I will go with xfs as that is my preference.

Simply run:

``mkfs.xfs /dev/sda2``

and wait for it to finish.

## Mounting your partitions

Whether you chose to encrypt or not, the process here is largely the same- we need to mount the root partition, create a directory for the ESP mount, and mount that as well.

Using the encrypted setup as an example, run:

``mount /dev/mapper/root /mnt``

Your root partition should now be mounted. Now, creating the mountpoint for the ESP:

``mkdir -p /mnt/boot/efi``

You can largely put this kinda wherever, but `/boot/efi` is the recommended location for Basix.

Next, mount the ESP:

``mount /dev/sda1 /mnt/boot/efi``

## Extracting the Basix rootfs tarball

Once you've got your disk set up and ready, you may deploy the Basix rootfs.

This guide largely assumes you've downloaded this to your live environment one way or another, but basically:

``tar xvf basix-rootfs.tar.gz -C /mnt``

Note that `basix-rootfs.tar.gz` is not likely to be the exact name of the tarball you've downloaded.

## Configuring the fstab

If you're using Void, there is an included `xgenfstab` script that can set up your fstab for you. Arch has an equivalent to this, but it is untested by me.

If you don't have an fstab generation script, you may:

```
git clone https://github.com/glacion/genfstab
cd genfstab
./genfstab /mnt > /mnt/etc/fstab
```

Regardless of what you do here, make sure to look over this as opposed to blindly accepting that it's valid.

## Chrooting into the Basix system

Once you've done everything listed above, you should be ready to get a shell on your new Basix installation.

I largely test this just with Voids `xchroot`.

Following that example, run:

`xchroot /mnt`

and you should be greeted by a Basix shell.

Note that if not all mounts and otherwise are created properly, the `bpm` build system is unlikely to be functional.

## Configuring the build system and recompiling base

First, we need to fetch the Basix `core` repository.

This can be done by running ``bpm pull``.

### Editing your bpm.conf

Open up `/etc/bpm/bpm.conf` in the included `vi` text editor.

If there are any adjustments you'd like to make to build flags or otherwise global bpm behavior before building and installing packages, now's the time to do that.

For example, these are my own compiler settings:

```
: "${CFLAGS:=-O2 -pipe -march=x86-64-v3 -mtune=generic -fno-plt -fno-semantic-interposition}"
: "${CXXFLAGS:=$CFLAGS}"
: "${LDFLAGS:=-Wl,-O1,--as-needed,--sort-common,-z,pack-relative-relocs}"

: "${CGO_CFLAGS:=$CFLAGS}"
: "${CGO_CXXFLAGS:=$CXXFLAGS}"
: "${CGO_LDFLAGS:=$LDFLAGS}"

: "${RUSTFLAGS:=-C opt-level=3 -C target-cpu=x86-64-v3 -C target-feature=-crt-static -C link-arg=-Wl,-O1 -C link-arg=-Wl,--as-needed -C link-arg=-Wl,--sort-common}"
: "${GOFLAGS:=-trimpath -buildvcs=false}"
: "${GOAMD64:=v3}"
: "${CGO_ENABLED:=1}"
: "${ZIGFLAGS:=-Doptimize=ReleaseSafe -Dcpu=x86_64_v3}"
```

It's also perfectly valid to just not touch these. Up to you.

### Recompiling the Basix base system

Before building anything, the base of the buildroot needs to be compiled raw on the host.

The reason for this is that `bpm` builds packages in what is basically an isolated chroot and the packages to construct it need to exist prior to a build.

To do this, run:
```
BPM_BUILDROOT= bpm build baselayout musl busybox make binutils gcc linux-headers bzip2 pigz certs pkgconf zstd xz b3sum
```

Once you've done that and configured your `bpm.conf` to your liking, recompile and reinstall everything on the base system with:

```
cd /var/db/bpm/installed/
bpm build * && bpm install *
```

And wait for this to finish. Given Basix is a rather lean system by default, this shouldn't take too long on reasonable hardware.

## Installing the Linux kernel

You may have noticed that Basix didn't ship with a kernel by default. This is very much intentional. You may install the kernel as configured by me by simply running:

``bpm install linux``

and waiting. Given this is a full fat kernel config that supports a huge range of hardware, this may take a while.

If you would like to configure your own kernel, you must shadow the `linux` package. You may view directions for this in the ``/etc/bpm/repos.conf`` file.

To configure your own kernel, simply replace the `dotconfig` file in the `linux` packages `files/` directory with your own. This is very much not a required step.

## Installing fsprogs

Install whatever filesystem programs you need. For this example, the package name would be ``xfsprogs``.

## Installing network utilities

If you simply use dhcp on your network via ethernet, all you need to do is install the `dhcpcd` package.

Otherwise, if you're using wireless, you may additionally install the `wpa_supplicant` package.

Configure these the same way you would elsewhere.

## Setting up an initramfs

You probably need to do this.

If you've decided to roll with an encrypted setup, install the `cryptsetup` package **BEFORE** generating your UKI.

The currently supported option for initramfs generation on Basix is via [tinyramfs](https://github.com/illiliti/tinyramfs), documentation can be found on the linked Github repo.

A valid configuration for the setup I've described here today may be:

```
compress="gzip -9"
hostonly=true
hooks="mdevd,luks"
root_type=xfs
luks_root=UUID=<UUID of your luks container>
luks_name=root
root=UUID=<UUID of your root partition under the luks container>
```

and this config is expected to be created at `/etc/tinyramfs/config`.

Note the inclusion of the `mdevd` hook and the order the hooks are listed in.

## Setting up a boot chain

This is a required step. As is, your Basix installation will be unable to boot.

There are several things you can choose to do here:

1. Use a UKI to boot the system directly via the UEFI (recommended)
2. Use the GRUB bootloader (useful for BIOS installations, potentially also useful if you're Librebooted)
3. Set up your kernel to be its own bootloader via efistub

Assuming you choose to roll a UKI, install the following packages:

```
buki
systemd-boot-efistub
efibootmgr
```

After doing so, run `buki` for help on building a UKI.

If you've installed the `tinyramfs` initramfs generator, you may use that to generate your UKI instead of using buki. Consult their documentation on that.

Once you've built your UKI, place it somewhere on your ESP. A valid location is typically ``/boot/efi/bootx64.efi``.

Next, we need to generate a boot entry that points at this UKI. This can be done with the following, given the examples on this page:

``efibootmgr --create --label "Basix Linux" --disk /dev/sda --part 1 --loader "\bootx64.efi"``

Note the backslash in the loader path, we're in Windows land here. In addition, note that the loader path is relative to the root of your ESP.

The label can be whatever you want, get creative.

### Non-UKI boot chains

Consult other distributions instructions on setting up GRUB or efistub.

### Can this be automated once I've set it up once?

Yep. Create hooks in `/etc/bpm/hooks/` to automate the configuration of your boot chain setup on update for the Linux kernel.

## Finishing up

Don't forget to set your ``passwd`` for the root user at least.

I would **HIGHLY** recommend installing the `s6-frontend` package, just to make life less agonizing.

From this point on, you should be ready to reboot into your new system. You could go ahead and build and install more packages now if you want, though. Good examples may be a more fleshed out text editor such as `vim` or `emacs`, or potentially `openresolv`.

Have fun!
