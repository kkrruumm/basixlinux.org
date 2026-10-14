title: Basix Wiki

By default, Basix is already quite hardened compared to the state of most other Linux distributions. This does not mean Basix is a super l33t hax0r OS.

This is largely achieved by what I consider to be "rational defaults," as well as a system that just does less stuff by default and gravitates towards a high quality and a well audited base system.

However, there are several ways the user may (and probably should) improve things themselves.

# Setting up a Network Firewall

Install the `nftables` package and enable its service.

Basix, by default, ships a relatively typical workstation/server type nftables configuration. This firewall config may be edited by opening `/etc/nftables.conf` in a text editor.

In particular, if you wish to allow certain ports into the machine, you will need to edit the `input` chain in this file.

# Kernel Parameters

Given the DIY nature of the Basix, I do not set kernel parameters by default in the kernel configuration.

However, there are a multitude of these you may wish to apply. A very brief few:

* `lockdown=confidentiality` - Attempts to separate userland and the kernel more. Userspace, including root, cannot modify the running kernel. When set to confidentiality as opposed to `integrity`, userspace also cannot *read* kernel memory nor extract kernel secrets.

* `module.sig_enforce=1` - Force all loaded kernel modules to be signed. This will likely break out-of-tree modules, so you may have to sign those yourself. During a kernel compile on Basix, all in-tree modules are signed automatically, so most people may just enable this.

* `page_alloc.shuffle=1` - Enables the kernel's page allocator shuffling feature. This randomizes the order of free pages in the buddy allocators free lists. Without this, the kernel will give out physical memory in a predictable/ascending order.

* `randomize_kstack_offset=1` - Enables the kernel's per-syscall kernel stack offset randomization. On every syscall entry, the kernel shifts where this syscall's stack frame begins by a random amount, so the layout of the kernel stack becomes more random.

* `init_on_free=1` - Explicitly zeroes memory that is freed. This one comes with a performance penalty, but is enabled on Basix kernels by default. This feature largely exists to prevent things like [cold boot attacks](https://en.wikipedia.org/wiki/Cold_boot_attack).

..and just, like, so many more things. You should do some reading around on [kernel.org](https://docs.kernel.org/admin-guide/kernel-parameters.html) for more information.

# sysctl

Basix ships a default sysctl configuration at `/etc/sysctl.d/10-basix.conf` with some moderately aggressive hardening options. This file may be edited by the user, but preferably one will "shadow" it with `/etc/sysctl.conf` or a config in `sysctl.d` that will be read out later, such as `/etc/sysctl.d/99-myconfig.conf`.

You may view this file and its many comments either on your filesystem, or [here](https://github.com/kkrruumm/basix-packages/blob/main/baselayout/files/sysctl.d/10-basix.conf).

You should read up on the possible sysctl options [here](https://docs.kernel.org/admin-guide/sysctl/index.html).

# Secure Boot

No, it's not an attack on the ecosystem. Secure boot is a wonderful bit of boot chain hardening when done correctly. Currently, [sbctl](https://github.com/Foxboron/sbctl) is provided in the repo to enable one to set up self-signed secure boot, and you should reference the documentation on the Github page for doing so.

Generally, on my own systems, I avoid Microsoft CA for secure boot as that kinda defeats the point. How this should be handled largely depends on your boot chain and your system- if your computer has a TPM, you may use it to read out checksums from the TPM eventlog for optionrom devices such as graphics cards (sbctl can do this for you).

Reasonable disclaimer here, don't soft-brick your system because your GPU firmware can no longer load. The effectiveness of this, of course, also depends on your motherboard vendor [not being stupid](https://kris.sh/posts/msi-motherboard-woes/). You should also set a BIOS passphrase if you choose to set this up.

# Boot Chains and You

This is precisely why the install guide recommends just booting a UKI directly with your UEFI, if you are using a UEFI system.

Assuming encryption, usage of just a "raw" UKI means you have precisely *one* file on the disk exposed- which may be protected with self-signed secure boot as explained above. This protects your initramfs, kernel parameters, the kernel itself, the whole boot chain.

If you're using [Libreboot](https://libreboot.org/) or maybe even your own custom coreboot by chance, all the better- Libreboot's GRUB payload bakes GRUB into the firmware and means absolutely nothing on the disk needs to be unencrypted as your firmware can unlock it, and solves the same problem secure boot otherwise tries to. At this point, hardening becomes following their guide to prevent internal flashing and whatnot, and maybe some [nail polish and glitter](https://trmm.net/Glitter/) on your screws for intrusion detection.

If you're using GRUB in a more traditional sense- good luck.

# Disk Encryption

Do it.

Generally, I recommend at least bumping `itertime` when you encrypt your drive- `10000`, or 10 seconds, is probably a fine target *in general*. As a TL;DR, the higher this value is, the longer brute forcing the drive should take. The value here is equivalent tot he amount of time it takes this LUKS container to unlock, or "iterate." Of course, the longer you're willing to wait here, the better. Keep in mind that this is calculated for the CPU it's run against, so if you're still rocking a Pentium 4, you may want to jack this up quite a lot. If the drive is unlocked by a system with a faster CPU, it will iterate quicker. The goal with this is to slow down the process of brute forcing.

Obviously, please use an actually good passphrase. I also recommend the usage of the `argon2id` KDF.

Depending on how your setup is, it's also possible to [detatch your luks headers](https://linuxconfig.org/how-to-use-luks-with-a-detached-header), which should net you plausible deniability that data exists on the drive at all as long as you keep the header hidden somewhere- DIY a "yubikey" by putting your header on a flash drive, whatever. Get creative.

# Miscellaneous bits

Try to adhere to the principle of least privilege. If a user doesn't need permissions for something, they shouldn't have them. This also applies to aplications- try to sandbox things where possible, [flatpak](https://flatpak.org/) is in the repo partially for this reason, though it's far from the best choice. You could also grab something like [conty](https://github.com/Kron4ek/Conty) to run applications out of, and sandbox it with bwrap. If you're using flatpak, please also grab [flatseal](https://flathub.org/en/apps/com.github.tchx84.Flatseal) and audit the permissions of everything you install from flathub, a lot of them are egregious.

Try to avoid running random software, and for fucks sake please stop blindly running `curl | bash` to install stuff. I would, of course, also recommend avoiding proprietary software as much as possible. If you must run proprietary software, please run it in a sandbox. You do not know what it may be doing to your computer.

OPSEC (Operations Security) is something a user should also keep in mind. If you have something to hide, don't talk about it. Practice good hygeine like using [ExifTool](https://exiftool.org/) to scrub EXIF data from images and such. ExifTool is in the Basix `core` repository.

Generally, I would suggest that it's not about giving away ZERO information to anything or anyone, but rather maintaining as much agency as possible over where that information goes.

Be wary of supply chain attacks. A lot of the modern programming world is unfortunately somewhat reliant on distribution methods largely out of the developers control. To be more specific, if you're going to install some garbage via npm or likewise, you should be very aware of what you're doing. Extra points if you put this stuff in a container, VM, or maybe even on a completely different machine on a different network or [VLAN](https://en.wikipedia.org/wiki/VLAN).

[Know your rights](https://www.eff.org/issues/know-your-rights).

# So is Basix a super hardened OS?

No. All of what is done by default and has been discussed on this page is largely unobtrusive day to day, but the bar is far from on the floor. Hardening a system is only worth it until it results in your computer being useless.

In addition, like you (unless you're a clanker), I am human and may make mistakes or miss things. Zero guarantees.
