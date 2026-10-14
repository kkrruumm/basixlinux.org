title: Basix Wiki

# Setting your timezone

Install the `tzdata` package. Upon doing so, you should have a bunch of files in `/usr/share/zoneinfo`.

Symlink the appropriate file to `/etc/localtime` to set your timezone. Example:

```
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

# NTP

To make sure your systems clock stays accurate, you should install an ntp daemon. Currently, the only option supported by Basix is OpenBSD's [OpenNTPD](https://openntpd.org/).

To set this up, install the `openntpd` package and enable its service.
