title: Basix Wiki

# Desktop Environments

Full desktop environments will likely never be allowed in the `core` repository as they're **all** way too annoying to deal with, both usage and packaging wise. There is simply no point.

# Window Managers

At the moment, options for your window manager are extremely limited, but assuming someone wanted to contribute one, Wayland window managers are likely to be accepted in `core`, with a few exceptions.

Right now, the supported window manager is [sway](https://swaywm.org/), as this is the window manager the creator of Basix (me again) uses.

There are a handful of supplemental programs to go along with this, such as:

App launcher: [fuzzel](https://codeberg.org/dnkl/fuzzel)

Notification daemon: [fnott](https://codeberg.org/dnkl/fnott)

Terminal: [foot](https://codeberg.org/dnkl/foot/)

Stuff on swaybar: [i3blocks](https://github.com/vivien/i3blocks)

X11 window manager options will not be accepted, and belong in a personal package repository if you wish to use one.

In general, given we already have [wlroots](https://gitlab.freedesktop.org/wlroots/wlroots/) in the repo, any window managers based on this are generally acceptable. Send a PR.

# Login Managers

[greetd](https://sr.ht/~kennylevinsen/greetd/) is in the `core` repository. To use this, install the `greetd` package and prepare for a rust bootstrap. Once installed, configure it and enable its service.
