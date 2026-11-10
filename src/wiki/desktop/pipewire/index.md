title: Basix Wiki

# Prerequisites

1. ``XDG_RUNTIME_DIR`` must be set in the environment. See [seats](/wiki/desktop/seats).
2. dbus must be running.

# Pipewire

Install the `pipewire` and `wireplumber` packages.

You may start pipewire however you wish, likely from your window manager or perhaps as a user service.

I have this script at `/usr/local/bin/pipewire-launcher`, which my sway config executes on startup:

```
#!/bin/sh

# We need to kill any existing pipewire instance to restore sound
pkill -u "${USER}" -fx /usr/bin/pipewire-pulse 1>/dev/null 2>&1
pkill -u "${USER}" -fx /usr/bin/pipewire-media-session 1>/dev/null 2>&1
pkill -u "${USER}" -fx /usr/bin/wireplumber 1>/dev/null 2>&1
pkill -u "${USER}" -fx /usr/bin/pipewire 1>/dev/null 2>&1

exec /usr/bin/pipewire &

# wait for pipewire to start before attempting to start related daemons
while [ "$(pgrep -f /usr/bin/pipewire)" = "" ]; do
	sleep 1
done

if [ -x /usr/bin/wireplumber ]; then
	exec /usr/bin/wireplumber &
elif [ -x /usr/bin/pipewire-media-session ]; then
	exec /usr/bin/pipewire-media-session &
fi

[ -f "/usr/share/pipewire/pipewire-pulse.conf" ] && exec /usr/bin/pipewire-pulse &
```

This script was lifted from Alpine Linux [here](https://gitlab.alpinelinux.org/alpine/aports/-/blob/master/community/pipewire/pipewire-launcher.sh).

# Screen capture

If you are using a wayland-native screen capture solution such as [wl-screenrec](https://github.com/rosalyntg/wl-screenrec), you do not need a portal nor pipewire.

Assuming you're using a wlroots-based compositor, install the `xdg-desktop-portal` and `xdg-desktop-portal-wlr` packages, and start them with your window manager or potentially as a user service.

For example, from my sway config:

```
exec /usr/libexec/xdg-desktop-portal -r
```

This will start all of the relevant portals.

If you're using fuzzel as your app launcher, you should also get a fancy little window/display picker via that, though fuzzel isn't the only option for that.

# Tools

[wiremix](https://github.com/tsowell/wiremix) is in the repository for controlling audio devices.
