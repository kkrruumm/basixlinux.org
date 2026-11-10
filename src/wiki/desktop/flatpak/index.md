title: Basix Wiki

# Installation

Install the `flatpak` package.

You may then run the following as root to add the [flathub](https://flathub.org/) repository:

```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

# Usage

To install packages via flatpak, you will need to run the commands as root either directly or via something like `opendoas`.

More typical distributions use `polkit` to allow users in the `wheel` group to install things via flatpak without entering a password. This is not possible on Basix due to the lack of `polkit`. Not much can be done about this as `polkit` would send the distro down a bit of a rabbithole of arguably undesireable software such as `elogind` and `eudev`.

For example:

```
(0) sagittarius ~ > doas flatpak install flathub io.gitlab.librewolf-community
```

Outside of that one quirk, usage of flatpak here is pretty much identical to any other distribution.

# Potential Alternatives

[conty](https://github.com/Kron4ek/Conty)
[distrobox](https://distrobox.it/)
...more, I'm sure.

In particular, conty is a great pick that can be easily sandboxed with a tool like bubblewrap or maybe unshare, though the usage is not as polished up as it is with flatpak. Perhaps a bit of scripting could be done to wrap conty.
